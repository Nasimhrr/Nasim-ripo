# contact_book.py
# A simple Contact Book application

contacts = {}

def add_contact(name,phone ):
    """Add a new contact"""
    if name in contacts:
        print("This contact already exists!")
    else:
        contacts[name] = phone
        print(f"{name} with number {phone} has been added.")

def delete_contact(name):
    """Delete a contact by name"""
    if name in contacts:
        del contacts[name]
        print(f"{name} has been deleted.")
    else:
        print("Contact not found!")

def search_contact(name):
    """ Search for a contact """
    if name in contacts:
        print(f"{name}: {contacts[name]}")
    else:
        print("Contact not found!")

def show_all():
    """ """
    if contacts:
        print("📒 Contact List:")
        for name, phone in contacts.items():
            print(f"- {name}: {phone}")
    else:
        print("The contact book is empty.")

def menu():
    while True:
        print("\n=== Simple Contact book ===")
        print("1. Add a contact")
        print("2. Delete a contact")
        print("3. Search for a contact")
        print("4. Show all contacts")
        print("5.exite ")

        choice = input("Enter your choice: ")

        if choice == "1":
            name = input("Enter name: ")
            phone = input("Enter phone number: ")
            add_contact(name, phone)

        elif choice == "2":
            name = input("Enter name to delete: ")
            delete_contact(name)

        elif choice == "3":
            name = input("Enter name to search: ")
            search_contact(name)

        elif choice == "4":
            show_all()

        elif choice == "5":
            print("Exiting program...")
            break

        else:
            print("Invalid option!")

if __name__ == "__main__":
    menu()
