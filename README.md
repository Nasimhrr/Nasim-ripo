
import json
import os
from datetime import datetime

FILE_NAME = "tasks.json"


def load_tasks():
    if not os.path.exists(FILE_NAME):
        return []
    with open(FILE_NAME, "r", encoding="utf-8") as file:
        return json.load(file)


def save_tasks(tasks):
    with open(FILE_NAME, "w", encoding="utf-8") as file:
        json.dump(tasks, file, indent=4, ensure_ascii=False)


def add_task(title):
    tasks = load_tasks()
    task = {
        "id": len(tasks) + 1,
        "title": title,
        "created_at": datetime.now().strftime("%Y-%m-%d %H:%M"),
        "done": False
    }
    tasks.append(task)
    save_tasks(tasks)
    print("✅ ")


def list_tasks():
    tasks = load_tasks()
    if not tasks:
        print("📭 No tasks found")
        return

    for task in tasks:
        status = "✔️" if task["done"] else "❌"
        print(f'{task["id"]}. {task["title"]} | {status} | {task["created_at"]}')


def complete_task(task_id):
    tasks = load_tasks()
    for task in tasks:
        if task["id"] == task_id:
            task["done"] = True
            save_tasks(tasks)
            print("🎉 Task marked as completed")
            return
    print("⚠️ Task not found")


def delete_task(task_id):
    tasks = load_tasks()
    tasks = [task for task in tasks if task["id"] != task_id]
    save_tasks(tasks)
    print("🗑️ Task deleted")


def show_menu():
    print("\n--- Task Manager ---")
    print("1. Add task")
    print("2. List tasks")
    print("3. Complete task")
    print("4. Delete task")
    print("5. Exit")


def main():
    while True:
        show_menu()
        choice = input("Choose an option: ")

        if choice == "1":
            title = input("Task title: ")
            add_task(title)

        elif choice == "2":
            list_tasks()

        elif choice == "3":
            task_id = int(input("Task ID to complete: "))
            complete_task(task_id)

        elif choice == "4":
            task_id = int(input("Task ID to delete: "))
            delete_task(task_id)

        elif choice == "5":
            print("👋 Goodbye")
            break

        else:
            print("❌ Invalid choice")


if __name__ == "__main__":
    main((
