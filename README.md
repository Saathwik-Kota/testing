Project Report: Employee Data Manager (C Language)

1. Project Overview: Employee Data Manager

The Employee Data Manager is a command-line app written in C. It helps you keep track of employees and departments—basically, it’s a simple record system. At its core, it uses singly linked lists to store both employee and department info. You can add, view, update, and delete records, just like a basic database. There’s also a JOIN feature that lets you see which employee belongs to which department, matching them up by dept_id. And so you don’t lose your data, it saves everything to CSV files (employees.csv and departments.csv) when you quit, and loads them back up when you start.

2. What It Does

Here’s what you can actually do with it:

Data Persistence  
The app loads all your employee and department data from CSV files when you start it up. When you’re done, you can save everything back to those files. So, nothing disappears between sessions.

Employee Management  
You can add new employees, see a list of everyone, update someone’s salary by their ID, or delete an employee by their ID.

Department Management  
Add new departments, view all departments, or delete a department by ID. If you delete a department, the app automatically deletes all employees in that department too, so your data stays consistent.

Data Query (JOIN)  
The big feature is the JOIN. With this, you can see employees and their department names together, matched up by dept_id. It works like an INNER JOIN in a database, combining info from both lists.

Memory Management  
Since it’s C, you have to watch your memory. The app uses dynamic allocation for strings and nodes, and it’s careful to free up memory when you delete things, so you don’t get leaks.

3. How It’s Built

A. Data Structures

There are three main structs: Employee, Department, and JoinedRecord. Each one has a *next pointer, so you can chain them into linked lists. For names (and dept names), you use char pointers and allocate memory with malloc and strdup, because you don’t know in advance how long the names will be.

B. Managing the Lists (CRUD)

Creation  
Functions like create_employee and create_department use malloc to make room for a new node. They copy the name strings safely and fill in the data. If there’s not enough memory, they handle it.

Insertion  
Employees and departments are added at the head of the list (head-insertion), so it’s fast—O(1) time.

Display  
You can print the employee or department lists in a nice table by traversing the links.

Update/Deletion  
When you want to change someone’s salary, update_employee_salary walks the list to find the right ID. For deletions, both delete_employee and delete_department look for the target node, using a prev pointer to keep the chain intact. If you delete a department, delete_employees_by_dept takes care of removing all its employees. And whenever a node or list is deleted, the app frees up the strings before freeing the node.

C. JOIN Operation

Mechanism  
perform_join loops through each employee. For every employee, it checks every department to find a matching dept_id.

Matching  
If there’s a match (employee’s dept_id equals department’s dept_id), it creates a new JoinedRecord node with data from both.

List Building  
It adds this joined record to a temporary joined list. The inner loop breaks after finding a match (assuming department IDs are unique), and then moves on to the next employee.

Cleanup  
After showing the joined results, it frees the whole temporary joined list so memory isn’t wasted.

D. File I/O

Saving  
save_employees and save_departments open their files in write mode, walk through the list, and write each record as a CSV line using fprintf.

Loading:  
When load_employees or load_departments runs, it opens the file in read mode (“r”). First, it wipes out any existing list by calling free_employee_list or free_department_list—this stops memory leaks and keeps us from loading the same data twice. Then, it grabs each line with fgets and uses a mix of string functions (strchr, strrchr, strncpy) and sscanf to break down the data. Names are wrapped in quotes, so it has to handle those carefully. For every line it parses, it calls the right create_ function and adds the new node to the list.

Breakdown of Contributions  
This project splits into three big parts, with each team member owning one:

Core Data Structures & Memory Management  
Team Member A set up the Employee, Department, and JoinedRecord structs, plus all the functions for walking through and freeing the lists—free_employee_list, free_department_list, free_joined_record_list, and the single-node free functions. They also wrote create_employee and create_department.

Core CRUD & Relational Functions  
Team Member B built the main logic for the app: adding employees and departments, displaying lists, updating salaries, deleting employees (by id or by department), deleting departments, and the key perform_join function.

File I/O and CLI Interface  
Team Member C handled saving and loading for employees and departments (save_employees, load_employees, save_departments, load_departments). They also set up the user interface functions: display_menu, the main program loop (with the switch for user choices, loading at start, saving at exit), and helper functions for input like add_department_cli.

Function Explanations

free_employee_node(Employee *emp)  
Frees up memory for a single Employee node, including the name string. Takes an Employee pointer, returns nothing. It’s essential for cleaning up after deleting a single employee.

free_department_list(Department **head)  
Walks through the whole Department list, freeing each node and its string. Input is a pointer to the head of the list; it returns nothing. Keeps memory leaks away when clearing the list (before loading or on exit).

create_employee(int id, const char *name, float salary, int dept_id)  
Allocates memory for a new Employee node and its name, sets up all the fields. Takes id, name, salary, and department id. Returns a pointer to the new Employee node (or NULL if it fails). This is the go-to for creating new employee records.

add_employee(Employee **head, Employee *new_node)  
Drops a new Employee node at the head of the employee list. Takes a pointer to the list head and the new node. Returns nothing. Adds records to your in-memory database.

display_employee_list(Employee *head)  
Walks through the Employee list and prints everyone out in a table. Input: pointer to list head. Output: nothing (just prints). Lets users see all the data.

update_employee_salary(Employee *head, int id, float new_salary)  
Finds an employee by id and updates their salary. Takes the list head, employee id, and new salary. Returns nothing. Changes the salary for an existing employee.

delete_department(Department **dept_head, Employee **emp_head, int id)  
Finds the department by id and removes it from the list, then calls delete_employees_by_dept to clear out all employees in that department. Inputs: pointers to the department and employee list heads, and the department id. Returns nothing. Keeps data consistent by cleaning up related employees.

save_employees(Employee *head)  
Opens employees.csv for writing, goes through the list, and writes every employee record in CSV format. Takes the list head, returns nothing. Saves your employee data to disk.

load_departments(Department **head)  
Clears the current department list, opens departments.csv to read, grabs each line, parses the CSV, and rebuilds the Department list. Input: pointer to list head. Output: nothing. Loads department data from disk.

perform_join(Employee *emp_head, Department *dept_head)  
Does an inner join: for each employee, finds the matching department using dept_id, creates a JoinedRecord, and links it into a new result list. Inputs: employee and department list heads. Output: pointer to the head of the JoinedRecord list. This is the main query function, linking employee data with department info.

display_joined_results(JoinedRecord *head)  
Walks through the JoinedRecord list and prints out the combined employee and department info. Input: head of the JoinedRecord list. Output: nothing (just prints). Shows the results of the JOIN.

main()  
Sets up the employee and department list heads, loads data from disk, runs the main application loop (using display_menu and a switch for user actions), and saves everything plus cleans up on exit. Input: none. Output: int (exit status). This is where everything starts and gets managed.
