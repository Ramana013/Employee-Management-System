# Employee-Management-System
import mysql.connector

# Database Connection (Modify with your credentials)
conn = mysql.connector.connect(host="127.0.0.1", user="root", password="Ramya@013", database="flm_db1")
cursor = conn.cursor()

# Task Management Functions
import datetime

def insert_task():
    print("\nPlease fill in the Task details. Fields marked with * are mandatory.\n")
    while True:
        task_id = input("Task ID *: ").strip()
        
        # Check if task_id already exists
        cursor.execute("SELECT task_id FROM tasks WHERE task_id = %s", (task_id,))
        if cursor.fetchone():
            print("❌ A task with this ID already exists! Please enter a unique Task ID.")
            choice = input("Press 'R' to re-enter Task ID or 'B' to go back to Task Module: ").strip().upper()
            if choice == 'B':
                return  # Exit function
            continue  # Retry entering task_id

        task_name = input("Task Name *: ").strip()

        # Check if task with the same name exists
        cursor.execute("SELECT task_name FROM tasks WHERE task_name = %s", (task_name,))
        if cursor.fetchone():
            print("❌ A task with this name already exists! Please enter a unique task name.")
            choice = input("Press 'R' to re-enter Task Name or 'B' to go back to Task Module: ").strip().upper()
            if choice == 'B':
                return  # Exit function
            continue  # Retry entering task_name

        task_description = input("Task Description: ").strip()
        assigned_employee = input("Assigned Employee ID *: ").strip()
        priority = input("Priority (Low/Medium/High) *: ").strip().capitalize()
        deadline = input("Deadline (YYYY-MM-DD) *: ").strip()

        # Validate required fields
        if not task_id or not task_name or not assigned_employee or not deadline or not priority:
            print("❌ Please fill in all details in all required fields.")
            continue  # Restart loop to re-enter data

        # Validate Task ID as an integer
        if not task_id.isdigit():
            print("❌ Task ID must be a numeric value.")
            continue

        # Validate Priority input
        if priority not in ["Low", "Medium", "High"]:
            print("❌ Invalid priority! Enter 'Low', 'Medium', or 'High'.")
            continue

        # Check if assigned employee exists
        cursor.execute("SELECT emp_id FROM employees WHERE emp_id = %s", (assigned_employee,))
        if cursor.fetchone() is None:
            print("❌ No employee found with this ID! Please enter a valid Employee ID.")
            continue

        # Validate deadline format and prevent past dates
        try:
            deadline_date = datetime.datetime.strptime(deadline, "%Y-%m-%d")
            if deadline_date < datetime.datetime.today():
                print("❌ Deadline cannot be in the past! Enter a future date.")
                continue
        except ValueError:
            print("❌ Invalid date format! Please enter in YYYY-MM-DD format.")
            continue

        # Insert task into the database
        sql = """INSERT INTO tasks (task_id, task_name, task_description, assigned_employee, priority, deadline) 
                 VALUES (%s, %s, %s, %s, %s, %s)"""
        values = (task_id, task_name, task_description, assigned_employee, priority, deadline)
        
        cursor.execute(sql, values)
        conn.commit()
        
        print("✅ Task added successfully!")
        break  # Exit loop after successful insertion


def list_tasks():
    cursor.execute("SELECT * FROM tasks")
    tasks = cursor.fetchall()
    for task in tasks:
        print(task)

def update_task():
    task_id = input("Enter Task ID to update: ")
    new_description = input("Enter new description: ")
    new_priority = input("Enter new priority: ")
    new_deadline = input("Enter new deadline (YYYY-MM-DD): ")
    sql = "UPDATE tasks SET task_description=%s, priority=%s, deadline=%s WHERE task_id=%s"
    values = (new_description, new_priority, new_deadline, task_id)
    cursor.execute(sql, values)
    conn.commit()
    print("Task updated successfully!")

def delete_task():
    task_id = input("Enter Task ID to delete: ")
    cursor.execute("DELETE FROM tasks WHERE task_id = %s", (task_id,))
    conn.commit()
    print("Task deleted successfully!")

def search_task():
    task_id = input("Enter Task ID to search: ")
    cursor.execute("SELECT * FROM tasks WHERE task_id = %s", (task_id,))
    task = cursor.fetchone()
    if task:
        print(task)
    else:
        print("Task not found.")

# Employee Management Functions
def create_emp():
    print("\nPlease fill in the Employee details. Fields marked with * are mandatory.\n")
    while True:
        emp_id = input("Employee ID *: ").strip()
        
        # Check if Employee ID already exists
        cursor.execute("SELECT emp_id FROM employees WHERE emp_id = %s", (emp_id,))
        if cursor.fetchone():
            print("❌ Employee with this ID already exists! Please enter a unique ID.")
            choice = input("Press 'R' to re-enter a new ID or 'B' to go back to Employee Module: ").strip().upper()
            if choice == 'B':
                return  # Exit function
            continue  # Retry entering ID
        
        emp_name = input("Employee Name: ").strip()
        emp_no = input("Employee Phone *: ").strip()
        emp_email = input("Employee Email: ").strip()
        emp_job_role = input("Employee Job Role: ").strip()
        emp_salary = input("Employee Salary: ").strip()
        
        # Validate required fields
        if not emp_name or not emp_no or not emp_email or not emp_job_role:
            print("❌ All fields are required. Please fill in all details.")
            continue  # Restart loop to re-enter data
        
        # Validate emp_no (phone) and emp_salary (salary) as numbers
        if not emp_no.isdigit():
            print("❌ Invalid phone number. Please enter digits only.")
            continue
        
        if not emp_salary.isdigit():
            print("❌ Invalid salary. Please enter a numeric value.")
            continue
        
        sql = """INSERT INTO employees (emp_id, emp_name, emp_no, emp_email, emp_job_role, emp_salary) 
                 VALUES (%s, %s, %s, %s, %s, %s)"""
        values = (emp_id, emp_name, emp_no, emp_email, emp_job_role, emp_salary)
        
        cursor.execute(sql, values)
        conn.commit()
        
        print("✅ Employee added successfully!")
        break  # Exit loop after successful insertion



def list_employees():
    cursor.execute("SELECT * FROM employees")
    employees = cursor.fetchall()
    for emp in employees:
        print(emp)

def update_employee():
    emp_id = input("Enter Employee ID to update: ")
    field = input("Which field to update? (name/phone/email/role/salary): ").strip().lower()
    new_value = input("Enter new value: ")

    field_mapping = {
        "name": "emp_name",
        "phone": "emp_no",
        "email": "emp_email",
        "role": "emp_job_role",
        "salary": "emp_salary"
    }

    if field in field_mapping:
        sql = f"UPDATE employees SET {field_mapping[field]} = %s WHERE emp_id = %s"
        cursor.execute(sql, (new_value, emp_id))
        conn.commit()
        print("Employee updated successfully!")
    else:
        print("Invalid field selected.")

def delete_employee():
    emp_id = input("Enter Employee ID to delete: ")
    cursor.execute("DELETE FROM employees WHERE emp_id = %s", (emp_id,))
    conn.commit()
    print("Employee deleted successfully!")

def search_employee():
    emp_id = input("Enter Employee ID to search: ")
    cursor.execute("SELECT * FROM employees WHERE emp_id = %s", (emp_id,))
    emp = cursor.fetchone()
    if emp:
        print(emp)
    else:
        print("Employee not found.")
#Client Management Functions

def create_client():
    print("\nPlease fill in the client details. Fields marked with * are mandatory.\n")
    client_id = input("Enter Client ID *: ")
    client_name = input("Enter Client Name *: ")
    client_type = input("Enter Client Type (Individual/Business) *: ")
    primary_contact = input("Enter Primary Person Contact *: ")
    phone_number = input("Enter Phone Number *: ")
    email = input("Enter Email Address *: ")
    address = input("Enter Address: ")
    city = input("Enter City: ")
    state = input("Enter State: ")
    client_status = input("Enter Client Status (Active/Inactive/Prospective) *: ")

    sql = """INSERT INTO clients (
        client_id, client_name, client_type, primary_contact, phone_number, email, 
        address, city, state, client_status
    ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s)"""

    values = (client_id, client_name, client_type, primary_contact, phone_number, email, 
              address, city, state, client_status)

    try:
        cursor.execute(sql, values)
        conn.commit()
        print("\n✅ Client added successfully!")
    except Exception as e:
        print("\n❌ Error while adding client:", e)



def list_clients():
    try:
        cursor.execute("SELECT * FROM clients")
        clients = cursor.fetchall()
        
        if not clients:
            print("\nNo clients found.")
            return
        
        print("\nList of Clients:")
        print("-" * 80)
        for client in clients:
            print(client)
        print("-" * 80)

    except Exception as e:
        print("\n❌ Error while fetching clients:", e)

def update_client():
    client_id = input("Enter Client ID to update: ").strip()
    
    # Check if the client exists
    cursor.execute("SELECT * FROM clients WHERE client_id = %s", (client_id,))
    client = cursor.fetchone()
    
    if not client:
        print("\n❌ Client not found.")
        return

    print("\nLeave a field blank to keep the existing value.")

    # Prompt user for updates
    client_name = input("Enter New Client Name (*): ").strip() or client[1]
    client_type = input("Enter New Client Type (Individual/Business) (*): ").strip() or client[2]
    primary_contact = input("Enter New Primary Contact (*): ").strip() or client[3]
    phone_number = input("Enter New Phone Number (*): ").strip() or client[4]
    email = input("Enter New Email (*): ").strip() or client[5]
    address = input("Enter New Address: ").strip() or client[6]
    city = input("Enter New City: ").strip() or client[7]
    state = input("Enter New State: ").strip() or client[8]
    client_status = input("Enter New Client Status (Active/Inactive/Prospective) (*): ").strip() or client[9]

    # Update query
    sql = """UPDATE clients 
             SET client_name = %s, client_type = %s, primary_contact = %s, phone_number = %s, 
                 email = %s, address = %s, city = %s, state = %s, client_status = %s
             WHERE client_id = %s"""

    values = (client_name, client_type, primary_contact, phone_number, email, address, city, state, client_status, client_id)

    try:
        cursor.execute(sql, values)
        conn.commit()
        print("\n✅ Client details updated successfully!")
    except Exception as e:
        print("\n❌ Error while updating client:", e)



def delete_client():
    client_id = input("Enter Client ID to delete: ")

    # Check if the client exists
    cursor.execute("SELECT * FROM clients WHERE client_id = %s", (client_id,))
    record = cursor.fetchone()

    if not record:
        print("No client found with the given ID.")
        return

    confirm = input(f"Are you sure you want to delete Client ID {client_id}? (yes/no): ").strip().lower()

    if confirm == "yes":
        try:
            sql = "DELETE FROM clients WHERE client_id = %s"
            cursor.execute(sql, (client_id,))
            conn.commit()
            print("Client record deleted successfully!")
        except mysql.connector.Error as err:
            print(f"Error: {err}")
    else:
        print("Operation canceled.")

def search_client():
    client_id = input("Enter Client ID to search: ").strip()
    
    cursor.execute("SELECT * FROM clients WHERE client_id = %s", (client_id,))
    client = cursor.fetchone()
    
    if client:
        print("\n🔍 Client Details:")
        print(f"Client ID: {client[0]}")
        print(f"Client Name: {client[1]}")
        print(f"Client Type: {client[2]}")
        print(f"Primary Contact: {client[3]}")
        print(f"Phone Number: {client[4]}")
        print(f"Email: {client[5]}")
        print(f"Address: {client[6] or 'N/A'}")
        print(f"City: {client[7] or 'N/A'}")
        print(f"State: {client[8] or 'N/A'}")
        print(f"Client Status: {client[9]}")
    else:
        print("\n❌ Client not found.")



    # Main Menu Loop
while True:
    print("\n1. Task Management\n2. Employee Management\n3. Client Management\n4. Exit")
    num = int(input("Enter your choice: "))

    if num == 1:
        while True:
            print("\n1. Create Task\n2. List Tasks\n3. Update Task\n4. Delete Task\n5. Search Task\n6. Back")
            choice = int(input("Enter your choice: "))
            if choice == 1:
                insert_task()
            elif choice == 2:
                list_tasks()
            elif choice == 3:
                update_task()
            elif choice == 4:
                delete_task()
            elif choice == 5:
                search_task()
            elif choice == 6:
                break
            else:
                print("Invalid choice, try again!")

    elif num == 2:
        while True:
            print("\n1. Create Employee\n2. List Employees\n3. Update Employee\n4. Delete Employee\n5. Search Employee\n6. Back")
            choice = int(input("Enter your choice: "))
            if choice == 1:
                create_emp()
            elif choice == 2:
                list_employees()
            elif choice == 3:
                update_employee()
            elif choice == 4:
                delete_employee()
            elif choice == 5:
                search_employee()
            elif choice == 6:
                break
            else:
                print("Invalid choice, try again!")
    
    elif num == 3:
        print("\n1. Create Client\n2. List Clients\n3. Update Client\n4. Delete Client\n5. Search Client\n6. Back")
        choice = int(input("Enter your choice: "))
        
        if choice == 1:
            create_client()
        elif choice == 2:
            list_clients()
        elif choice == 3:
            update_client()
        elif choice == 4:
            delete_client()
        elif choice == 5:
            search_client()
        elif choice == 6:
            break
        else:
            print("Invalid choice, try again!")


    elif num == 4:
        print("Exiting program...")
        break
    else:
        print("Invalid choice, try again!")

# Close the connection at the end
cursor.close()
conn.close()
