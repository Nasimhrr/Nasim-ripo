#!/usr/bin/env python3 """ Smart File Organizer

One-shot, dependency-free file organizer for messy folders.

Groups files into categories by extension (configurable)

Creates date buckets (YYYY/MM) if you enable --by-date

Dry-run mode to preview actions

Undo last run from a JSON log

Collision-safe moves (keeps both files)


Usage examples:

Preview what would happen

python smart_file_organizer.py ~/Downloads --dry-run

Actually organize by type only

python smart_file_organizer.py ~/Downloads

Organize by type and date 
python smart_file_organizer.py ~/Downloads --by-date

Custom output root (default is the same folder)

python smart_file_organizer.py ~/Downloads --out ~/Downloads/Sorted

Undo the last run out (uses the log stored next to -- out)

python smart_file_organizer.py --undo --out ~/Downloads/sorted

Notes:

Only regular files are moved; directories are ignored.

Hidden files are ignored by default (use --include-hidden to include them).

The log file is stored at <out>/.organizer_log.json


Author: ruthlessly practical. """ import argparse import hashlib import json import os from dataclasses import dataclass from datetime import datetime from pathlib import Path from typing import Dict, List, Optional, Tuple

LOG_NAME = ".organizer_log.json"

DEFAULT_CATEGORIES: Dict[str, str] = { # Images ".jpg": "Pictures", ".jpeg": "Pictures", ".png": "Pictures", ".gif": "Pictures", ".webp": "Pictures", ".heic": "Pictures", ".svg": "Pictures", ".bmp": "Pictures", # Video ".mp4": "Videos", ".mov": "Videos", ".avi": "Videos", ".mkv": "Videos", ".webm": "Videos", # Audio ".mp3": "Audio", ".wav": "Audio", ".m4a": "Audio", ".flac": "Audio", ".ogg": "Audio", # Documents ".pdf": "Documents", ".doc": "Documents", ".docx": "Documents", ".xls": "Documents", ".xlsx": "Documents", ".ppt": "Documents", ".pptx": "Documents", ".txt": "Documents", ".md": "Documents", # Archives ".zip": "Archives", ".rar": "Archives", ".7z": "Archives", ".tar": "Archives", ".gz": "Archives", # Code ".py": "Code", ".js": "Code", ".ts": "Code", ".json": "Code", ".html": "Code", ".css": "Code", ".java": "Code", ".c": "Code", ".cpp": "Code", ".cs": "Code", ".go": "Code", ".rs": "Code", # Misc ".dmg": "Installers", ".pkg": "Installers", ".exe": "Installers", ".msi": "Installers", }

@dataclass class Move: src: str dst: str

def human_size(num: int) -> str: for unit in ["B", "KB", "MB", "GB", "TB"]: if num < : return f"{num:.1f}{unit}" num /=1024.0  return f"{num:.1f}PB"

def file_hash(path: Path, nbytes: int = 256 * 1024) -> str: """Fast-ish content hash for duplicate detection (reads first/last chunks).""" h = hashlib.sha256() size = path.stat().st_size with path.open("rb") as f: chunk = f.read(min(nbytes, size)) h.update(chunk) if size > nbytes: f.seek(max(0, size - nbytes)) h.update(f.read(nbytes)) return h.hexdigest()

def load_log(out_root: Path) -> Dict: log_path = out_root / LOG_NAME if log_path.exists(): with log_path.open("r", encoding="utf-8") as f: return json.load(f) return {"runs": []}

def save_log(out_root: Path, log: Dict) -> None: log_path = out_root / LOG_NAME out_root.mkdir(parents=True, exist_ok=True) with log_path.open("w", encoding="utf-8") as f: json.dump(log, f, indent=2)

def pick_category(path: Path, categories: Dict[str, str]) -> str: ext = path.suffix.lower() return categories.get(ext, "Other")

def date_bucket(path: Path) -> Tuple[str, str]: ts = path.stat().st_mtime d = datetime.fromtimestamp(ts) return d.strftime("%Y"), d.strftime("%m")

def collision_safe_dst(dst: Path) -> Path: if not dst.exists(): return dst stem, suffix = dst.stem, dst.suffix i = 2 while True: candidate = dst.with_name(f"{stem} ({i}){suffix}") if not candidate.exists(): return candidate i += 1

def plan_moves(src_root: Path, out_root: Path, by_date: bool, include_hidden: bool, categories: Dict[str, str]) -> List[Move]: moves: List[Move] = [] seen_hashes = {} for p in src_root.iterdir(): if p.is_dir(): continue if not include_hidden and p.name.startswith('.'): continue cat = pick_category(p, categories) parts = [out_root, Path(cat)] if by_date: y, m = date_bucket(p) parts += [Path(y), Path(m)] target_dir = Path(*parts) target_dir.mkdir(parents=True, exist_ok=True) dst = target_dir / p.name # If a file with same name exists but identical content, skip. if dst.exists() and dst.is_file(): try: if file_hash(p) == file_hash(dst): continue  # exact duplicate except Exception: pass dst = collision_safe_dst(dst) moves.append(Move(str(p), str(dst))) return moves

def perform_moves(moves: List[Move], dry_run: bool) -> None: for mv in moves: if dry_run: size = Path(mv.src).stat().st_size print(f"DRY  {mv.src}  ->  {mv.dst}  ({human_size(size)})") else: os.replace(mv.src, mv.dst) print(f"MOVE {mv.src}  ->  {mv.dst}")

def record_run(out_root: Path, src_root: Path, moves: List[Move]) -> None: log = load_log(out_root) log["runs"].append({ "ts": datetime.now().isoformat(timespec="seconds"), "src_root": str(src_root), "moves": [mv.dict for mv in moves], }) save_log(out_root, log)

def undo_last(out_root: Path) -> None: log = load_log(out_root) if not log.get("runs"): print("Nothing to undo.") return run = log["runs"].pop() # Reverse moves in reverse order for mv in reversed(run["moves"]): src = Path(mv["dst"])  # where we moved to dst = Path(mv["src"])  # original location dst.parent.mkdir(parents=True, exist_ok=True) if src.exists(): os.replace(src, dst) print(f"UNDO {src} -> {dst}") else: print(f"SKIP (missing) {src}") save_log(out_root, log)

def main(): parser = argparse.ArgumentParser(description="Organize files by type (and optionally by date) with undo support.") parser.add_argument("src", nargs="?", default=None, help="Source folder to organize (e.g., ~/Downloads)") parser.add_argument("--out", default=None, help="Destination root (default: same as src)") parser.add_argument("--by-date", action="store_true", help="Nest into YYYY/MM buckets based on file modified time") parser.add_argument("--include-hidden", action="store_true", help="Include dotfiles") parser.add_argument("--dry-run", action="store_true", help="Only print the plan, do not move anything") parser.add_argument("--undo", action="store_true", help="Undo the last run (reads the log under --out)")

args = 

if args.undo:
    out_root =Path(args.out or ".") .expanduser().resolve()
    undo_last(out_root)
    return

if not args.src:
    parser.error("Provide a source folder or use --undo.")

src_root = Path(args.src).expanduser().resolve()
if not src_root.exists() or not src_root.is_dir():
    parser.error("Source folder does not exist or is not a directory.")

out_root = Path(args.out).expanduser().resolve() if args.out else src_root

moves = plan_moves(src_root, out_root, args.by_date, args.include_hidden, DEFAULT_CATEGORIES)
if not moves:
    print("Nothing to do. Folder already clean or filters excluded everything.")
    return

perform_moves(moves, args.dry_run)
if not args.dry_run:
    record_run(out_root, src_root, moves)
    print(f"\n✅ Organized {len(moves)} file(s). Log saved to {out_root / LOG_NAME}")

if name == "main": main()

