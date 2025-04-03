

### A. Report Purpose and Business Context

The goal of this report is to answer the question: *How many DVD rentals were sold by each employee during a specific month?* This information will help management assess the efficiency and performance of each employee, enabling them to reward top performers and identify areas for improvement among lower performers. By incentivizing high performers, the company can motivate all employees to increase sales, ultimately boosting overall revenue. Additionally, the report is flexible and can be easily modified to track sales for any given month, providing regular insights.

### A1. Detailed and Summary Tables

The detailed table will contain the following fields:
- **staff_id** (integer): Unique identifier for each staff member.
- **employee_name** (text): Combines the first and last names of the employee.
- **rental_date** (date): The date of the rental transaction, which will be used to filter sales by month.
- **customer_id** (integer): Unique identifier for each customer.

The summary table will contain:
- **employee_name** (text): The full name of each employee.
- **total_rentals** (integer): The total number of rentals handled by each employee, based on the count of rental transactions in the detailed table.

### A2. Explanation of Data Types

**Detailed Table Data Types**:
- **staff_id** (integer): Numeric ID to identify the staff member.
- **first_name** (varchar): Employee's first name (used in the transformation to create employee_name).
- **last_name** (varchar): Employee's last name (used in the transformation to create employee_name).
- **employee_name** (text): Concatenation of the first and last names into a single string for better readability.
- **rental_date** (date): The date of the rental.
- **customer_id** (integer): A unique identifier for each customer.

**Summary Table Data Types**:
- **employee_name** (text): The employee's full name, derived from the detailed table.
- **total_rentals** (integer): The total number of rentals for the employee, calculated from the rental dates.

### A3. Data Sources and Transformation

The data for both tables comes from the **Staff** and **Rental** tables. I will extract the staff ID, first name, and last name from the **Staff** table, and rental date and customer ID from the **Rental** table. The **employee_name** field will combine the first and last names for improved readability. The summary table will be populated based on aggregated data from the detailed table, using the **employee_name** and **total_rentals** columns.

### A4. Transformation Function

A function will be used to transform the employee’s first and last names into a single **employee_name** field. This function will enhance the readability of the detailed table and streamline the process of reporting employee names.

### A5. Use of Detailed and Summary Tables

The **detailed table** will allow tracking of each individual rental transaction, making it possible to verify sales and evaluate employee productivity for a specific month. Additionally, it includes the **customer_id**, which could help identify frequent customers for each employee. The **summary table** provides a high-level overview of employee performance, summarizing the total rentals for each employee. This table is particularly useful for identifying high performers and areas for improvement. It can also assist in scheduling, ensuring top performers are available during peak times.

### A6. Report Frequency

I recommend refreshing this report on a monthly basis to ensure that the data is up-to-date for employee reviews. This will also allow managers to track performance over time and make informed decisions regarding training or incentives.

---

### B. Employee Name Transformation Code

```sql
-- Function to combine first and last name into a full employee name
CREATE OR REPLACE FUNCTION employee_name(first_name VARCHAR(45), last_name VARCHAR(45))
	RETURNS text
	LANGUAGE plpgsql
AS
$$
DECLARE employee_name text;
BEGIN
    -- Concatenate first and last name into a full employee name
    SELECT CONCAT(first_name, ' ', last_name) INTO employee_name;
    RETURN employee_name;
END;
$$
;
```

### C. Code for Creating Detailed and Summary Tables

```sql
-- Create Detailed Table
DROP TABLE IF EXISTS rentals_per_employee_detailed;
CREATE TABLE rentals_per_employee_detailed (
    staff_id integer,
    employee_name text,
    rental_date date,
    customer_id int
);

-- Create Summary Table
DROP TABLE IF EXISTS rentals_per_employee_summary;
CREATE TABLE rentals_per_employee_summary (
    employee_name text,
    total_rentals integer
);
```

### D. Code for Inserting Data into the Detailed Table

```sql
-- Insert data from the Staff and Rental tables into the detailed table
INSERT INTO rentals_per_employee_detailed (
    SELECT s.staff_id, employee_name(s.first_name, s.last_name), r.rental_date, r.customer_id
    FROM staff s
    JOIN rental r
    ON s.staff_id = r.staff_id
    WHERE rental_date BETWEEN '2005-05-01' AND '2005-05-31'
);
```

### E. Trigger and Function to Update Summary Table

```sql
-- Function to update the summary table after an insert into the detailed table
CREATE OR REPLACE FUNCTION update_summary_table() RETURNS TRIGGER AS
$$
BEGIN
    -- Delete old data in the summary table
    DELETE FROM rentals_per_employee_summary;

    -- Insert new summary data: employee name and total rentals
    INSERT INTO rentals_per_employee_summary (
        SELECT d.employee_name, COUNT(d.rental_date) AS total_rentals
        FROM rentals_per_employee_detailed d
        GROUP BY employee_name
    );

    -- Return the new row
    RETURN new;
END;
$$
LANGUAGE plpgsql;

-- Trigger to execute function after each insert into the detailed table
CREATE TRIGGER trig_update
    AFTER INSERT ON rentals_per_employee_detailed
    FOR EACH ROW
    EXECUTE PROCEDURE update_summary_table();
```

### F. Code for Refreshing Data in Both Tables

```sql
-- Procedure to clear and refresh the data in both the detailed and summary tables
CREATE OR REPLACE PROCEDURE refresh_tables()
LANGUAGE plpgsql
AS $$
BEGIN
    -- Delete old data from both tables
    DELETE FROM rentals_per_employee_detailed;
    DELETE FROM rentals_per_employee_summary;

    -- Re-insert data into the detailed table
    INSERT INTO rentals_per_employee_detailed (
        SELECT s.staff_id, employee_name(s.first_name, s.last_name), r.rental_date, r.customer_id
        FROM staff s
        JOIN rental r
        ON s.staff_id = r.staff_id
        WHERE rental_date BETWEEN '2005-05-01' AND '2005-05-31'
    );

    RETURN;
END
$$;
```

### F1. Job Scheduling with pgAgent

To automate this procedure every month, I recommend using **pgAgent**, a job scheduling tool for PostgreSQL. It allows you to automate stored procedures and SQL functions. While pgAgent requires a separate installation, it offers extensive functionality for automating tasks, such as refreshing monthly data, and provides flexibility to adjust the date range dynamically for different months.

### H. Sources

No sources were used in the creation of this report.
