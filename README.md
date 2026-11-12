"""
main.py
Main menu for the Student Registration & Contact Intelligence System.
Run this file to start the application.
"""

from pathlib import Path
import csv

from loader import load_contacts, save_contacts, CLEAN_FILE
from validator import validate_contact
from analytics import generate_report

BASE_DIR = Path(__file__).resolve().parent


def read_clean_contacts():
    """Read the saved student records from students_clean.csv."""
    if not CLEAN_FILE.exists():
        return []

    with open(CLEAN_FILE, "r", newline="", encoding="utf-8-sig") as csv_file:
        reader = csv.DictReader(csv_file)
        return [
            {
                "name": (row.get("name") or "").strip(),
                "phone": (row.get("phone") or "").strip(),
                "email": (row.get("email") or "").strip()
            }
            for row in reader
        ]


def display_students(students):
    if not students:
        print("\nNo student records found.")
        return

    print("\n{:<5} {:<25} {:<15} {:<30}".format(
        "No.", "Name", "Phone", "Email"
    ))
    print("-" * 78)

    for number, student in enumerate(students, start=1):
        print("{:<5} {:<25} {:<15} {:<30}".format(
            number,
            student["name"][:24],
            student["phone"],
            student["email"][:29]
        ))


def find_student_index(students, search_value):
    search_value = search_value.strip().lower()
    for index, student in enumerate(students):
        if search_value in (
            student["name"].lower(),
            student["phone"].lower(),
            student["email"].lower()
        ):
            return index
    return None


def register_student(students):
    print("\n--- REGISTER NEW STUDENT ---")
    name = input("Enter student name: ").strip()
    phone = input("Enter 10-digit phone number: ").strip()
    email = input("Enter email address: ").strip()

    valid, reason = validate_contact(name, phone, email)
    if not valid:
        print("Registration rejected:", reason)
        return

    if any(s["phone"] == phone for s in students):
        print("Registration rejected: Duplicate phone number.")
        return

    if any(s["email"].lower() == email.lower() for s in students):
        print("Registration rejected: Duplicate email address.")
        return

    students.append({"name": name, "phone": phone, "email": email})
    save_contacts(students)
    print("Student registered successfully!")


def search_student(students):
    query = input("\nEnter exact name, phone or email to search: ").strip()
    index = find_student_index(students, query)

    if index is None:
        print("Student not found.")
    else:
        print("\nStudent found:")
        display_students([students[index]])


def update_student(students):
    query = input("\nEnter exact phone number of student to update: ").strip()
    index = next(
        (i for i, student in enumerate(students)
         if student["phone"] == query),
        None
    )

    if index is None:
        print("Student not found.")
        return

    current = students[index]
    print("Press Enter to keep the existing value.")

    name = input(f"New name [{current['name']}]: ").strip() or current["name"]
    phone = input(f"New phone [{current['phone']}]: ").strip() or current["phone"]
    email = input(f"New email [{current['email']}]: ").strip() or current["email"]

    valid, reason = validate_contact(name, phone, email)
    if not valid:
        print("Update rejected:", reason)
        return

    for other_index, student in enumerate(students):
        if other_index != index:
            if student["phone"] == phone:
                print("Update rejected: Phone number already belongs to another student.")
                return
            if student["email"].lower() == email.lower():
                print("Update rejected: Email already belongs to another student.")
                return

    students[index] = {"name": name, "phone": phone, "email": email}
    save_contacts(students)
    print("Student details updated successfully!")


def delete_student(students):
    query = input("\nEnter exact phone number of student to delete: ").strip()
    index = next(
        (i for i, student in enumerate(students)
         if student["phone"] == query),
        None
    )

    if index is None:
        print("Student not found.")
        return

    print("Selected student:", students[index]["name"])
    confirmation = input("Are you sure you want to delete? (y/n): ").strip().lower()

    if confirmation == "y":
        removed = students.pop(index)
        save_contacts(students)
        print(f"{removed['name']} deleted successfully.")
    else:
        print("Delete cancelled.")


def main():
    print("=" * 52)
    print(" STUDENT REGISTRATION & CONTACT INTELLIGENCE SYSTEM")
    print("=" * 52)

    # First run: create the cleaned file from the original CSV.
    # Later runs: load the saved clean file so changes are preserved.
    if CLEAN_FILE.exists():
        students = read_clean_contacts()
        print(f"Loaded {len(students)} saved student record(s).")
    else:
        print("First run: processing original contacts.csv...")
        students = load_contacts()

    while True:
        print("\n--------------- MAIN MENU ---------------")
        print("1. View All Students")
        print("2. Search Student")
        print("3. Register Student")
        print("4. Update Student")
        print("5. Delete Student")
        print("6. Analytics Report")
        print("7. Exit")
        print("-----------------------------------------")

        choice = input("Enter your choice (1-7): ").strip()

        if choice == "1":
            display_students(students)
        elif choice == "2":
            search_student(students)
        elif choice == "3":
            register_student(students)
        elif choice == "4":
            update_student(students)
        elif choice == "5":
            delete_student(students)
        elif choice == "6":
            generate_report(students)
        elif choice == "7":
            save_contacts(students)
            print("Student records saved. Thank you for using the system!")
            break
        else:
            print("Invalid choice. Please enter a number from 1 to 7.")


if __name__ == "__main__":
    main()
