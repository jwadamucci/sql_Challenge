# Challenge 9

## Intitial Data Evaluation

Using Excel, the data was evaluated and initial relationships were determined.  

The tables and their relationships were then drafted using a website called quickdatabasediagrams.com.  That is a tool for sketching out relational databases, their tables and columns, and the various keys that aid in relating the tables to each other.  It is a text-based tool that creates an easily digestible diagram of the database.  
  
Both the text used to create the diagram and a screenshot of the diagram are included in this Challenge's solution:
* QuickDBDiagrams.txt
* QuickDBDiagrams.png

## Database Schema Creation

Using PostgreSQL's PGAdmin4, the employeeDB database was created and populated.  Initial data for the tables was provided in 6 CSV files.  
  
This document outlines the following executed SQL code:
* creating the tables (Table Creation)
* setting up their relationships for the employeeDB application (Establishing Foriegn Keys)
* querying the tables to extract data for analysis (Data Analysis)

## Table Creation

The following tables were created in the *public* schema:

### Employees Table

CREATE TABLE public.employees
(  
    emp_no serial NOT NULL,  
    emp_title_id character(256) NOT NULL,  
    birth_date date NOT NULL,  
    first_name character(256) NOT NULL,  
    last_name character(256) NOT NULL,  
    sex character NOT NULL,  
    hire_date date NOT NULL,  
    PRIMARY KEY (emp_no)  
  
);  
  
ALTER TABLE IF EXISTS public.employees  
    OWNER to postgres;  

### Titles Table

CREATE TABLE public.titles  
(  
        title_id character(256) NOT NULL,  
        title character(256) NOT NULL,  
        PRIMARY KEY (title_id)  
);

ALTER TABLE IF EXISTS public.titles  
    OWNER to postgres;  

### Departments Table

CREATE TABLE public.departments  
(  
    dept_no character(256) NOT NULL,  
    dept_name character(256) NOT NULL,  
    PRIMARY KEY (dept_no)  
);  

ALTER TABLE IF EXISTS public.departments  
    OWNER to postgres;  
  
### Salaries Table
  
CREATE TABLE public.salaries  
(  
    emp_no integer NOT NULL,  
    salary integer NOT NULL,  
    PRIMARY KEY (emp_no)  
);

ALTER TABLE IF EXISTS public.salaries  
    OWNER to postgres;  

### Department Employees Table  
  
CREATE TABLE public.dept_emp  
(  
    emp_no integer NOT NULL,  
    dept_no character(256) NOT NULL  
);  

ALTER TABLE IF EXISTS public.dept_emp  
    OWNER to postgres;  
  
### Department Manager Table  
  
CREATE TABLE public.dept_manager  
(  
    dept_no character(256) NOT NULL,  
    emp_no integer NOT NULL  
);  
  
ALTER TABLE IF EXISTS public.dept_manager  
    OWNER to postgres;  
  
## Table creation complete.  
  
## Creating Foreign Keys (Establishing Relationships)  
The following foreign keys were added to establish relationships between the tables:  
  
### Department Employees Table  
  
ALTER TABLE IF EXISTS public.dept_emp  
    ADD CONSTRAINT "emp_no_FK" FOREIGN KEY (emp_no)  
    REFERENCES public.employees (emp_no) MATCH SIMPLE  
    ON UPDATE NO ACTION  
    ON DELETE NO ACTION  
    NOT VALID;  
  
ALTER TABLE IF EXISTS public.dept_emp  
    ADD CONSTRAINT "dept_no_FK" FOREIGN KEY (dept_no)  
    REFERENCES public.departments (dept_no) MATCH SIMPLE  
    ON UPDATE NO ACTION  
    ON DELETE NO ACTION  
    NOT VALID;  
  
### Department Manager Table  
  
ALTER TABLE IF EXISTS public.dept_manager  
    ADD CONSTRAINT "dept_no_FK" FOREIGN KEY (dept_no)  
    REFERENCES public.departments (dept_no) MATCH SIMPLE  
    ON UPDATE NO ACTION  
    ON DELETE NO ACTION  
    NOT VALID;  
  
### Employees Table  
  
ALTER TABLE IF EXISTS public.employees  
    ADD CONSTRAINT "emp_title_id_FK" FOREIGN KEY (emp_title_id)  
    REFERENCES public.titles (title_id) MATCH SIMPLE  
    ON UPDATE NO ACTION  
    ON DELETE NO ACTION  
    NOT VALID;  
  
### Salaries Table  
  
ALTER TABLE IF EXISTS public.salaries  
    ADD CONSTRAINT "emp_no_FK" FOREIGN KEY (emp_no)  
    REFERENCES public.employees (emp_no) MATCH SIMPLE  
    ON UPDATE NO ACTION  
    ON DELETE NO ACTION  
    NOT VALID;  
  
## End of foreign keys.  
  
## Data Analysis  
The following SQL queries were executed for data analysis:  
  
1. List the employee number, last name, first name, sex, and salary of each employee.  
  
    SELECT e.emp_no, last_name, first_name, sex, s.salary FROM employees AS e  
        JOIN salaries AS s ON e.emp_no = s.emp_no;  
      
    _Saved in /Data Analysis Results/DA 1 Results.csv_  
  
2. List the first name, last name, and hire date for the employees who were hired in 1986.  
  
    SELECT first_name, last_name, hire_date FROM employees  
        WHERE hire_date BETWEEN '1986-01-01' AND '1986-12-31' ORDER BY hire_date ASC;  
  
    _Saved in /Data Analysis Results/DA 2 Results.csv_  
  
3. List the manager of each department along with their department number, department name, employee number, last name, and first name.  
  
    SELECT d.dept_no, d.dept_name, e.emp_no, e.first_name, e.last_name FROM employees AS e  
        INNER JOIN dept_manager AS dm ON e.emp_no = dm.emp_no  
        INNER JOIN departments AS d ON dm.dept_no = d.dept_no  
    WHERE e.emp_no IN (SELECT emp_no FROM dept_manager)  
    ORDER BY d.dept_name asc, e.last_name asc;  
  
    _Saved in /Data Analysis Results/DA 3 Results.csv_  
  
4. List the department number for each employee along with that employee’s employee number, last name, first name, and department name.  
  
    SELECT d.dept_no, e.emp_no, e.first_name, e.last_name, d.dept_name FROM employees AS e  
        INNER JOIN dept_emp AS de ON e.emp_no = de.emp_no  
        INNER JOIN departments AS d ON de.dept_no = d.dept_no  
    WHERE e.emp_no IN (SELECT emp_no FROM employees)  
    ORDER BY d.dept_name asc, e.last_name asc;  
  
    _Saved in /Data Analysis Results/DA 4 Results.csv_  
  
5. List first name, last name, and sex of each employee whose first name is Hercules and whose last name begins with the letter B.  
  
    SELECT first_name, last_name, sex FROM employees  
    WHERE first_name = 'Hercules' AND last_name LIKE 'B%'  
    ORDER BY last_name ASC;  
  
    _Saved in /Data Analysis Results/DA 5 Results.csv_  
  
6. List each employee in the Sales department, including their employee number, last name, and first name.  
  
    SELECT e.emp_no, e.first_name, e.last_name FROM employees AS e  
    WHERE e.emp_no IN   
        (SELECT emp_no from dept_emp WHERE dept_no =  
            (SELECT dept_no FROM departments WHERE dept_name = 'Sales')  
    )  
    ORDER BY last_name asc;  
  
    _Saved in /Data Analysis Results/DA 6 Results.csv_  
  
7. List each employee in the Sales and Development departments, including their employee number, last name, first name, and department name.  
  
    SELECT e.emp_no, e.first_name, e.last_name, d.dept_name FROM employees AS e  
        INNER JOIN dept_emp AS de ON e.emp_no = de.emp_no  
        INNER JOIN departments as d ON de.dept_no = d.dept_no  
    WHERE de.dept_no IN   
        (SELECT dept_no from departments WHERE (dept_name = 'Sales' OR dept_name = 'Development'))  
    ORDER BY last_name, dept_name asc;  
  
    _Saved in /Data Analysis Results/DA 7 Results.csv_  
  
8. List the frequency counts, in descending order, of all the employee last names (that is, how many employees share each last name).  
  
    SELECT last_name, count(last_name) as "# Sharing Last Name" FROM employees  
        GROUP BY last_name ORDER BY "# Sharing Last Name" DESC, last_name ASC;  
  
    _Saved in /Data Analysis Results/DA 8 Results.csv_  
  
## End of data analysis.  
# End of Challenge 9