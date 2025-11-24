## 📄 Project Report: Employee Data Manager (C Language)

### 1. Project Description: Employee Data Manager

The **Employee Data Manager** is a command-line interface (CLI) application built in **C** to manage basic records for employees and departments. The core data structure used is a **singly linked list** for both `Employee` records and `Department` records. This system simulates a simple relational database management function by allowing the user to create, display, update, and delete records, as well as **"JOIN"** the employee and department data based on the matching `dept_id`. The application also supports **file I/O** to persist the data between sessions by saving and loading the linked lists to/from separate CSV files (`employees.csv` and `departments.csv`).

### 2. Detailed Explanation of Functionalities

The application provides the following key functional groups:

| Functionality Group | Description |
| :--- | :--- |
| **Data Persistence** | The program can **Load** all employee and department data from CSV files at startup and **Save** all current data back to the files upon request or exit. This ensures data is not lost between runs. |
| **Employee Management** | Allows the user to **Add** new employees, **Display** the full list of employees, **Update** an employee's salary by ID, and **Delete** an employee by ID. |
| **Department Management** | Allows the user to **Add** new departments, **Display** the department list, and **Delete** a department by ID. Deleting a department automatically deletes all associated employee records for data integrity. |
| **Data Query (JOIN)** | The key feature is the **`perform_join`** function, which mimics a database **INNER JOIN**. It links employees with their corresponding department names and displays the combined result set. This result is stored in a temporary `JoinedRecord` linked list. |
| **Memory Management** | Comprehensive functions are implemented for **dynamic memory allocation** (for strings and nodes) and **deallocation** (`free`) to prevent memory leaks, including functions to free individual nodes and entire lists. |

---

### 3. Step-by-Step Implementation of Functionalities

#### A. Data Structures (Structs)

1.  **Definition:** Three primary structs (`Employee`, `Department`, `JoinedRecord`) were defined. All structs include a `*next` pointer, enabling them to form a **singly linked list**.
2.  **Dynamic Strings:** The `name` and `dept_name` fields are defined as `char *`. This requires the use of `malloc`/`strdup` for memory allocation when creating nodes, which is essential for handling string data of varying lengths and for proper memory freeing.

#### B. Node and List Management (CRUD Operations)

1.  **Creation:** Functions like `create_employee` and `create_department` use `malloc` to allocate memory for the node, `strdup` to safely copy the input strings (name/dept\_name), and initialize the fields. They handle failed memory allocations.
2.  **Insertion:** `add_employee` and `add_department` implement a **head-insertion** method, adding new nodes to the beginning of the list for $O(1)$ complexity.
3.  **Display:** `display_employee_list` and `display_department_list` traverse their respective linked lists, printing the data in a formatted table.
4.  **Update/Deletion:** `update_employee_salary` traverses the list until the matching ID is found. `delete_employee` and `delete_department` traverse the list, requiring a `prev` pointer to correctly bypass and free the target node. `delete_department` leverages `delete_employees_by_dept` to enforce relational integrity. **Crucially, all deletion and list-clearing functions call `free` on the allocated strings (`name`, `dept_name`) *before* freeing the node itself.**

#### C. The JOIN Operation (`perform_join`)

1.  **Mechanism:** The `perform_join` function iterates through the **Employee List** (outer loop). For **each employee**, it iterates through the entire **Department List** (inner loop).
2.  **Matching:** An **INNER JOIN** condition is checked: `if (current_emp->dept_id == current_dept->dept_id)`.
3.  **Record Creation:** If a match is found, a new `JoinedRecord` node is created using `create_joined_record`, copying data from both the employee and the department node.
4.  **List Building:** The new `JoinedRecord` is appended to the `join_head` list. The inner loop breaks after the first match is found for the employee (assuming unique Department IDs), and the outer loop proceeds to the next employee.
5.  **Cleanup:** After displaying the results with `display_joined_results`, the entire temporary **`JoinedRecord` list is freed** using `free_joined_record_list` to release memory.

#### D. File I/O (`load_...` and `save_...`)

1.  **Saving:** `save_employees` and `save_departments` open their respective files in **write mode (`"w"`)**, traverse the list, and write each record as a comma-separated line (CSV format) using `fprintf`.
2.  **Loading:** `load_employees` and `load_departments` open the files in **read mode (`"r"`)**. Before loading, the existing list is cleared with `free_employee_list` or `free_department_list` to prevent memory leaks and duplication. It uses `fgets` to read lines and string manipulation (`strchr`, `strrchr`, `strncpy`) along with `sscanf` to parse the structured data, including names enclosed in quotes. Each parsed line is then used to call the appropriate `create_` function, and the resulting node is added to the list.

---

### 4. Breakdown of Contributions

This project is divided into three major components, with contributions assigned to three team members.

| Component | Description | Contributor |
| :--- | :--- | :--- |
| **Core Data Structures & Memory Management** | Defining the `Employee`, `Department`, and `JoinedRecord` structs. Implementing all list traversal and freeing functions: `free_employee_list`, `free_department_list`, `free_joined_record_list`, and the single-node free functions. Implemented `create_employee` and `create_department`. | **Team Member A** |
| **Core CRUD & Relational Functions** | Implementing the primary application logic: `add_employee`, `add_department`, `display_employee_list`, `display_department_list`, `update_employee_salary`, `delete_employee`, `delete_employees_by_dept`, `delete_department`, and the crucial **`perform_join`** logic. | **Team Member B** |
| **File I/O and CLI Interface** | Implementing data persistence: `save_employees`, `load_employees`, `save_departments`, `load_departments`. Implementing the user interface functions: `display_menu`, `main` function structure (including `switch` case for choices, initial load, and exit save), and helper functions for CLI input like `add_department_cli`. | **Team Member C** |

---

### 5. Function Explanations

| Function Name | Description, Inputs, Outputs, and Role |
| :--- | :--- |
| `free_employee_node(Employee *emp)` | Frees memory for a single `Employee` node's string (`emp->name`) and the node itself. **Input:** `Employee *`. **Output:** `void`. **Role:** Essential for proper memory cleanup during single-node deletions. |
| `free_department_list(Department **head)` | Iterates through the entire `Department` list, freeing each node's string and the node. **Input:** `Department **`. **Output:** `void`. **Role:** Prevents memory leaks by clearing the list (used before loading and on exit). |
| `create_employee(int id, const char *name, float salary, int dept_id)` | Allocates memory for a new `Employee` node and its name string, initializing fields. **Input:** ID, name (string), salary (float), Dept ID. **Output:** `Employee *` (pointer to the new node or `NULL` on failure). **Role:** Factory function for new employee records. |
| `add_employee(Employee **head, Employee *new_node)` | Inserts a new `Employee` node at the head of the employee linked list. **Input:** `Employee **` (list head), `Employee *` (new node). **Output:** `void`. **Role:** Adds new records to the in-memory database. |
| `display_employee_list(Employee *head)` | Traverses the `Employee` list and prints all employee records in a formatted table. **Input:** `Employee *` (list head). **Output:** `void` (prints to stdout). **Role:** Allows the user to view the data. |
| `update_employee_salary(Employee *head, int id, float new_salary)` | Finds an employee by `id` and updates their `salary` field. **Input:** `Employee *` (list head), ID, new salary (float). **Output:** `void`. **Role:** Modifies existing data. |
| `delete_department(Department **dept_head, Employee **emp_head, int id)` | Finds a department by `id`, deletes it from the department list, and then calls `delete_employees_by_dept` to remove all associated employees. **Input:** `Department **`, `Employee **`, Dept ID. **Output:** `void`. **Role:** Enforces referential integrity by deleting related employee records. |
| `save_employees(Employee *head)` | Opens `employees.csv` in write mode, traverses the list, and writes all employee records to the file in CSV format. **Input:** `Employee *` (list head). **Output:** `void`. **Role:** Persists employee data to disk. |
| `load_departments(Department **head)` | Clears the existing department list, opens `departments.csv` in read mode, reads lines, parses the CSV data, and rebuilds the `Department` linked list. **Input:** `Department **` (list head pointer). **Output:** `void`. **Role:** Loads department data from disk into memory. |
| `perform_join(Employee *emp_head, Department *dept_head)` | Performs an inner join: iterates through employees, finds the matching department based on `dept_id`, creates a `JoinedRecord`, and links it to a new result list. **Input:** `Employee *`, `Department *` (list heads). **Output:** `JoinedRecord *` (head of the temporary join result list). **Role:** The core query functionality, linking relational data. |
| `display_joined_results(JoinedRecord *head)` | Traverses the temporary `JoinedRecord` list and prints the combined employee/department data. **Input:** `JoinedRecord *` (list head). **Output:** `void` (prints to stdout). **Role:** Displays the result of the JOIN operation. |
| `main()` | Initializes the two list heads (`employee_head`, `department_head`), loads initial data, manages the application loop using `display_menu` and a `switch` statement for user choices, and performs final cleanup/save on exit. **Input:** None. **Output:** `int` (program exit status). **Role:** The application's entry point and control center. |
