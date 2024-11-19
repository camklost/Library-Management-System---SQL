# **Library Management System: SQL Project Documentation**

## **Project Overview**  
This project demonstrates a comprehensive library management system implemented using SQL. It includes tasks such as managing book records, issuing and returning books, tracking overdue items, and generating performance reports. The project highlights key SQL techniques like `INSERT`, `UPDATE`, `DELETE`, `JOIN`, `GROUP BY`, `HAVING`, `CTAS`, stored procedures, and aggregate functions.

---

## **Database Tables**

The following tables were used in the project:

- `books`: Contains book details like ISBN, title, category, rental price, availability status, author, and publisher.

- `branch`: Stores branch information, including manager details and contact info.

- `employees`: Maintains employee records with branch associations.

- `issued_status`: Tracks the books issued by employees to members.

- `members`: Holds member details, including registration dates and addresses.

- `return_status`: Records information about returned books.

- `book_counts`: A summary table created to track the number of times each book has been issued.


## **Database Setup**

The following schema was used to create and structure the library management database. This design supports key functionalities such as member management, book issuance and returns, and branch-level operations.

```sql
-- Library Management System Project

-- Branch Table
DROP TABLE IF EXISTS branch;
CREATE TABLE branch(
    branch_id VARCHAR(10) PRIMARY KEY,	
    manager_id VARCHAR(10),	
    branch_address VARCHAR(55),	
    contact_no VARCHAR(15),
    UNIQUE (branch_id)
);

-- Employees Table
DROP TABLE IF EXISTS employees;
CREATE TABLE employees(
    emp_id VARCHAR(10) PRIMARY KEY,	
    emp_name VARCHAR(25),	
    position VARCHAR(15),	
    salary FLOAT,	
    branch_id VARCHAR(10) -- FK
);

-- Books Table
DROP TABLE IF EXISTS books;
CREATE TABLE books(
    isbn VARCHAR(20) PRIMARY KEY,	
    book_title VARCHAR(75),	
    category VARCHAR(25),	
    rental_price FLOAT,	
    status VARCHAR(15),	
    author VARCHAR(35),	
    publisher VARCHAR(55),
    UNIQUE (isbn)
);

-- Members Table
DROP TABLE IF EXISTS members;
CREATE TABLE members(
    member_id VARCHAR(10) PRIMARY KEY,	
    member_name VARCHAR(25),	
    member_address VARCHAR(75),	
    reg_date DATE
);

-- Issued Status Table
DROP TABLE IF EXISTS issued_status;
CREATE TABLE issued_status(
    issued_id VARCHAR(10) PRIMARY KEY,	
    issued_member_id VARCHAR(10), -- FK	
    issued_book_name VARCHAR(75),	
    issued_date DATE,	
    issued_book_isbn VARCHAR(25), -- FK	
    issued_emp_id VARCHAR(10) -- FK
);

-- Return Status Table
DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status(
    return_id VARCHAR(10) PRIMARY KEY,	
    issued_id VARCHAR(10), -- FK	
    return_book_name VARCHAR(75),	
    return_date DATE,	
    return_book_isbn VARCHAR(20)
);

-- Foreign Keys
ALTER TABLE issued_status
ADD CONSTRAINT fk_members
FOREIGN KEY (issued_member_id)
REFERENCES members(member_id);

ALTER TABLE issued_status
ADD CONSTRAINT fk_books
FOREIGN KEY (issued_book_isbn)
REFERENCES books(isbn);

ALTER TABLE issued_status
ADD CONSTRAINT fk_employees
FOREIGN KEY (issued_emp_id)
REFERENCES employees(emp_id);

ALTER TABLE employees
ADD CONSTRAINT fk_branch
FOREIGN KEY (branch_id)
REFERENCES branch(branch_id);

ALTER TABLE return_status
ADD CONSTRAINT fk_issued_status
FOREIGN KEY (issued_id)
REFERENCES issued_status(issued_id);
```
## Entity Relationship Diagram (ERD)
Below is the Entity Relationship Diagram (ERD) of the library database in a snowflake design:

![Library ERD](https://github.com/camklost/Library-Management-System---SQL/blob/main/library_erd.png)


---

## **Accomplished Tasks**

Here are the SQL queries used to achieve various tasks in the library management system:

### **Task 1: Verify All Tables**  

Objective: Ensure all tables are working correctly  

```sql
-- Display contents of all tables for verification
SELECT * FROM books;
SELECT * FROM branch;
SELECT * FROM employees;
SELECT * FROM issued_status;
SELECT * FROM members;
SELECT * FROM return_status;
SELECT * FROM book_counts;

```

### **Task 2: Add a New Book Record**  

Objective: Add a new book to the library's catalog.  

```sql
-- Add a new book with complete details
INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
VALUES
('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');

-- Verify the addition
SELECT * FROM books;

```
### **Task 3: Update Member Address**

```sql
-- Update the address of member C101
UPDATE members
SET member_address = '125 Main St'
WHERE member_id = 'C101';
```

### **Task 4: Delete a Record from Issued Status**

Objective: Remove a specific issuance record from the `issued_status` table.

```sql
-- Delete the record with issued_id = 'IS121'
DELETE FROM issued_status
WHERE issued_id = 'IS121';
```

### **Task 5: Retrieve Books Issued by Specific Employees**

Objective: Select all books issued by the employee with `emp_id = 'E101'`.

```sql
-- Retrieve all books issued by employee E101
SELECT issued_book_name
FROM issued_status
WHERE issued_emp_id = 'E101';
```

### **Task 6: Identify Employees Who Issued More Than One Book**

Objective: Use `GROUP BY` and `HAVING` to identify such employees.

```sql
-- Count the number of books issued by each employee and filter
SELECT 
    issued_emp_id, 
    COUNT(issued_id) AS total_books_issued
FROM issued_status
GROUP BY issued_emp_id
HAVING COUNT(issued_id) > 1;
```

### **Task 7: Create a Summary Table Using CTAS**

Objective: Create a summary table for books issued.

```sql
-- Create a table summarizing book issuance counts
CREATE TABLE book_counts AS
SELECT 
    issued_book_isbn AS isbn,
    issued_book_name AS book_title,
    COUNT(issued_id) AS no_issued
FROM issued_status
GROUP BY isbn, book_title;
```

### **Task 8: Retrieve Books in a Specific Category**

Objective: Retrieve all books in the "Classic" category.

```sql
-- Fetch books in the "Classic" category
SELECT * FROM books
WHERE category = 'Classic';
```

### **Task 9: Calculate Total Rental Income by Category**

Objective: Calculate rental income grouped by book categories.

```sql
-- Sum rental prices for books issued by category
SELECT 
    b.category,
    SUM(b.rental_price) AS total_income
FROM books AS b
JOIN issued_status AS i
ON b.isbn = i.issued_book_isbn
GROUP BY b.category;
```

### **Task 10: List Recently Registered Members**

Objective: Retrieve members registered in the last 180 days.

```sql
-- Use current date to find recent registrations

-- The last 180 days from now:
SELECT * FROM members
WHERE reg_date >= CURRENT_DATE - INTERVAL '180 days';

-- The last 180 days from the last date of the dataset:
SELECT *
FROM members
WHERE reg_date >= (
    SELECT MAX(reg_date) 
    FROM members
) - INTERVAL '180 days'; 
```

### **Task 11: List Employees With Branch Details and Manager Names**

Objective: Fetch branch details along with employee and manager names.

```sql
-- Fetch employees with their branch and manager details
SELECT 
    e.emp_name AS employee_name,
    m.emp_name AS manager_name,
    b.branch_address,
    b.contact_no
FROM employees AS e
JOIN branch AS b
    ON e.branch_id = b.branch_id
LEFT JOIN employees AS m
    ON b.manager_id = m.emp_id;
```

### **Task 12: Create a Table for High-Priced Books**

Objective: Create a table of books with rental prices above $7.

```sql
-- Create a new table for high-priced books
CREATE TABLE high_priced_books AS
SELECT *
FROM books
WHERE rental_price > 7;

-- Verify the table
SELECT * FROM high_priced_books;
```

### **Task 13: List Books Not Yet Returned**

Objective: Identify books currently issued but not returned.

```sql
-- Use a LEFT JOIN to find books with no return record
SELECT * FROM issued_status AS ist
LEFT JOIN return_status AS rs
ON rs.issued_id = ist.issued_id
WHERE rs.return_id IS NULL;
```

### **Task 14: Identify Overdue Members and Books**

Objective: Find members with books overdue by more than 30 days.

```sql
-- Fetch overdue members with book details
SELECT
    i.issued_member_id,
    m.member_name,
    b.book_title,
    i.issued_date,
    CURRENT_DATE - i.issued_date AS days_overdue
FROM issued_status AS i
JOIN members AS m
    ON m.member_id = i.issued_member_id
JOIN books AS b
    ON b.isbn = i.issued_book_isbn
LEFT JOIN return_status AS r
    ON r.issued_id = i.issued_id
WHERE r.return_date IS NULL
  AND (CURRENT_DATE - i.issued_date) > 30;
```

### **Task 15: Most Popular Books**

Objective: Identify the top three most-issued books.

```sql
-- Retrieve top 3 most-issued books
SELECT 
    issued_book_name AS book_title,
    COUNT(issued_id) AS times_issued
FROM issued_status
GROUP BY issued_book_name
ORDER BY COUNT(issued_id) DESC
LIMIT 3;
```

### **Task 16: Total Books Issued per Member**

Objective: Count the number of books issued by each member.

```sql
-- Count books issued by each member
SELECT 
    issued_member_id AS member_id,
    m.member_name,
    COUNT(issued_id) AS total_books_issued
FROM issued_status AS i
JOIN members AS m
    ON m.member_id = i.issued_member_id
GROUP BY issued_member_id, m.member_name
ORDER BY total_books_issued DESC;
```

### **Task 17: Books Never Issued**

Objective: Identify books in the catalog that have never been issued.

```sql
-- Use an anti-join to find books never issued
SELECT * 
FROM books AS b
LEFT JOIN issued_status AS i
    ON b.isbn = i.issued_book_isbn
WHERE i.issued_id IS NULL;
```

### **Task 18: Automate Late Fee Calculation Using a Stored Procedure**

Objective: Create a stored procedure to calculate late fees.

```sql
-- Create a stored procedure for late fee calculation
CREATE OR REPLACE PROCEDURE calculate_late_fees()
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE issued_status AS i
    SET late_fee = CASE 
        WHEN CURRENT_DATE - i.issued_date > 30 THEN (CURRENT_DATE - i.issued_date - 30) * 0.50
        ELSE 0
    END
    WHERE NOT EXISTS (
        SELECT 1
        FROM return_status AS r
        WHERE r.issued_id = i.issued_id
    );
END;
$$;

-- Execute the procedure
CALL calculate_late_fees();
```

### **Task 19: Branch Performance Report**

Objective: Generate a report showing branch performance, including issued books, returned books, and revenue.

```sql
-- Create a table summarizing branch performance
CREATE TABLE branch_reports
AS
SELECT 
    b.branch_id,
    COUNT(i.issued_id) AS number_book_issued, -- Total books issued
    COUNT(r.return_id) AS number_of_book_return, -- Total books returned
    SUM(bk.rental_price) AS total_revenue -- Revenue from book rentals
FROM issued_status AS i
JOIN employees AS e
    ON e.emp_id = i.issued_emp_id
JOIN branch AS b
    ON e.branch_id = b.branch_id
LEFT JOIN return_status AS r
    ON r.issued_id = i.issued_id
JOIN books AS bk
    ON i.issued_book_isbn = bk.isbn
GROUP BY b.branch_id;

-- View the performance report
SELECT * FROM branch_reports;
```

### **Task 20: Update Book Status on Return**

Objective: Update the status of books in the `books` table to "yes" when they are returned.

```sql
-- Stored procedure to handle book returns and update book status
CREATE OR REPLACE PROCEDURE add_return_records(p_return_id VARCHAR(10), p_issued_id VARCHAR(10))
LANGUAGE plpgsql
AS $$

DECLARE
    v_isbn VARCHAR(50); -- Variable to store the book ISBN
    v_book_name VARCHAR(80); -- Variable to store the book name

BEGIN
    -- Insert a return record into the return_status table
    INSERT INTO return_status(return_id, issued_id, return_date)
    VALUES
    (p_return_id, p_issued_id, CURRENT_DATE);

    -- Retrieve the book ISBN and name from the issued_status table
    SELECT
        issued_book_isbn,
        issued_book_name
    INTO 
        v_isbn,
        v_book_name
    FROM issued_status
    WHERE issued_id = p_issued_id;

    -- Update the book's status in the books table
    UPDATE books
    SET status = 'yes'
    WHERE isbn = v_isbn;

    -- Notify that the status has been updated
    RAISE NOTICE 'Status updated for: %', v_book_name;

END;
$$;

-- Test the procedure
CALL add_return_records('RS138', 'IS135');
```

### **Task 21: Active Members**

Objective: Create a table of members who issued at least one book in the last 8 months.
```sql
-- Create a table to store active members
CREATE TABLE active_members
AS
SELECT * FROM members
WHERE member_id IN (
    -- Subquery to identify members who issued books recently
    SELECT DISTINCT issued_member_id
    FROM issued_status
    WHERE issued_date >= CURRENT_DATE - INTERVAL '8 month'
);

-- View the active members table
SELECT * FROM active_members;
```

### **Task 22: Employees with the Most Book Issues Processed**

Objective: Identify the top 3 employees who processed the most book issues.

```sql
-- Query to find the top employees by books issued
SELECT
    e.emp_name, -- Employee name
    b.branch_address, -- Branch details
    COUNT(i.issued_id) AS no_book_issued -- Total books issued by the employee
FROM issued_status AS i
JOIN employees AS e
    ON e.emp_id = i.issued_emp_id
JOIN branch AS b
    ON e.branch_id = b.branch_id
GROUP BY e.emp_name, b.branch_address
ORDER BY no_book_issued DESC
LIMIT 3;
```

### **Task 24: Stored Procedure for Issuing Books**

Objective: Automate the process of issuing books and updating their status.

```sql
-- Stored procedure to manage book issuance
CREATE OR REPLACE PROCEDURE issue_book(p_issued_id VARCHAR(10), p_issued_member_id VARCHAR(30), p_issued_book_isbn VARCHAR(50), p_issued_emp_id VARCHAR(10))
LANGUAGE plpgsql
AS $$

DECLARE
    v_status VARCHAR(10); -- Variable to store the book's availability status

BEGIN
    -- Check the book's current status
    SELECT 
        status
    INTO
        v_status
    FROM books
    WHERE isbn = p_issued_book_isbn;

    -- If the book is available, process the issuance
    IF v_status = 'yes' THEN
        -- Insert a record into the issued_status table
        INSERT INTO issued_status(issued_id, issued_member_id, issued_date, issued_book_isbn, issued_emp_id)
        VALUES
        (p_issued_id, p_issued_member_id, CURRENT_DATE, p_issued_book_isbn, p_issued_emp_id);

        -- Update the book's status to unavailable
        UPDATE books
        SET status = 'no'
        WHERE isbn = p_issued_book_isbn;

        -- Notify the user of successful issuance
        RAISE NOTICE 'Book records added successfully for book ISBN: %', p_issued_book_isbn;

    ELSE
        -- Notify the user that the book is unavailable
        RAISE NOTICE 'Sorry, the book you have requested is unavailable. ISBN: %', p_issued_book_isbn;

    END IF;

END;
$$;

-- Test the procedure with available and unavailable books
CALL issue_book('IS155', 'C108', '978-0-553-29698-2', 'E104');
CALL issue_book('IS155', 'C108', '978-0-375-41398', 'E104');
```

---

## **Insights and Conclusions**

### **Key Learnings**

- Joins: Efficiently combined data across tables to extract meaningful insights.

- Aggregate Functions: Calculated totals, averages, and counts to summarize data.

- Stored Procedures: Automated late fee calculations, demonstrating SQL scripting capabilities.

- Table Creation: Used CTAS for summarizing data and creating specific subsets for reporting.

- Anti-Join: Identified unused resources, such as books never issued.

- Automation: Stored procedures simplify common library tasks like issuing and returning books.

- Branch Analysis: Reports help identify high-performing branches and optimize resources.

- Membership Insights: Active members table highlights member engagement.

### **Suggested Improvements**

- Normalization: Further normalize tables to reduce redundancy.

- Indexes: Add indexes to frequently queried columns to improve performance.

- User Roles: Implement stricter role-based access control to protect sensitive data.

### **Future Enhancements**

- Integrate the database with a front-end application for library staff and members.

- Develop additional stored procedures for overdue reminders and book availability notifications.

- Incorporate analytics for forecasting popular book categories and inventory needs.
