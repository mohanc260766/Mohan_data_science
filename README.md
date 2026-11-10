# Mohan_data_science
import csv
import re
import io

# --- 1. Generate Synthetic Raw contacts.csv File ---
synthetic_csv_data = """name,phone,email,course
Alice Smith,123-456-7890,alice@email.com,IIT-JEE
Bob ,987654321,bob@email,NEET
Charlie,555-666-7777,charlie@email.com,NDA
Alice Smith,123-456-7890,alice@email.com,IIT-JEE
David O'Connor,444-555-1111,david@domain.co.in,Foundation
Eve,1112223333,invalid_email,NEET
,9998887777,no_name@email.com,IIT-JEE
Frank Miller,999-888-7777,frank@email.com,NDA
"""

# Writing to an in-memory string file buffer to simulate reading a CSV file directly
csv_file_buffer = io.StringIO(synthetic_csv_data.strip())
# --- 2. Registration & Contact Intelligence Engine ---

def clean_phone(phone_str):
    # Remove common formatting characters like hyphens, spaces, and parentheses
    cleaned = re.sub(r'[\s\-\(\)]', '', phone_str)
    return cleaned

def validate_registration(row, seen_emails):
    name = row.get('name', '').strip()
    phone = row.get('phone', '').strip()
    email = row.get('email', '').strip()
    course = row.get('course', '').strip()
    
    errors = []
    
    # Name Validation
    if not name:
        errors.append("Incomplete/Missing Name")
    elif not re.match(r"^[a-zA-Z\s']+$", name):
        errors.append("Malformed Name (Contains invalid characters)")
        
    # Phone Validation (Should be exactly 10 digits after cleaning)
    cleaned_phone = clean_phone(phone)
    if not cleaned_phone.isdigit() or len(cleaned_phone) != 10:
        errors.append(f"Malformed Phone Number ({phone})")
        
    # Email Validation
    email_regex = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    if not re.match(email_regex, email):
        errors.append(f"Invalid Email Format ({email})")
    elif email.lower() in seen_emails:
        errors.append(f"Duplicate Registration Email ({email})")
        
    return errors, {
        'name': name,
        'phone': cleaned_phone if not errors else phone,
        'email': email.lower(),
        'course': course
    }

clean_students = []
faulty_records = []
seen_emails = set()

# Process the CSV data
reader = csv.DictReader(csv_file_buffer)
for row in reader:
    errors, record = validate_registration(row, seen_emails)
    if not errors:
        clean_students.append(record)
        seen_emails.add(record['email'])
    else:
        record['errors'] = errors
        faulty_records.append(record)

print(f"Processing Complete: {len(clean_students)} clean records, {len(faulty_records)} faulty records stored.")
# --- 3. Business Intelligence (BI) Summary Report ---

print("="*60)
print("       COACHING CENTRE BI REGISTRATION SUMMARY REPORT       ")
print("="*60)
print(f"Total Registrations Received : {len(clean_students) + len(faulty_records)}")
print(f"Clean & Valid Enrolments     : {len(clean_students)}")
print(f"Faulty / Flagged Records     : {len(faulty_records)}")
print("-"*60)

# Count courses in clean records
course_counts = {}
for s in clean_students:
    course_counts[s['course']] = course_counts.get(s['course'], 0) + 1

print("Successful Enrolments by Course:")
for course, count in course_counts.items():
    print(f" - {course}: {count} student(s)")

print("-"*60)
print("Faulty Records Details:")
for idx, f_rec in enumerate(faulty_records, 1):
    print(f" {idx}. Name: {f_rec['name'] or '<Missing>'} | Errors: {', '.join(f_rec['errors'])}")
print("="*60)
# --- 4. Interactive Query Interface ---

def search_student(query):
    query = query.lower().strip()
    results = []
    for s in clean_students:
        if (query in s['name'].lower() or 
            query in s['email'] or 
            query in s['phone']):
            results.append(s)
    return results

# Demo Query
search_term = "Alice"
print(f"Searching for '{search_term}':")
search_results = search_student(search_term)
for res in search_results:
    print(f"Found: {res['name']} | Phone: {res['phone']} | Email: {res['email']} | Course: {res['course']}")
