# WEEK-7
Question 1: Build a Complete Database Management System Objective:
-- Create Members Table
CREATE TABLE members (
    member_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    join_date DATE NOT NULL,
    is_active BOOLEAN DEFAULT TRUE
);

-- Create Books Table
CREATE TABLE books (
    book_id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    author VARCHAR(100) NOT NULL,
    isbn VARCHAR(20) NOT NULL UNIQUE,
    published_year INT,
    copies_available INT NOT NULL DEFAULT 1
);

-- Create Loans Table (Many-to-Many: Members <-> Books)
CREATE TABLE loans (
    loan_id INT AUTO_INCREMENT PRIMARY KEY,
    member_id INT NOT NULL,
    book_id INT NOT NULL,
    loan_date DATE NOT NULL,
    due_date DATE NOT NULL,
    return_date DATE,
    CONSTRAINT fk_member FOREIGN KEY (member_id) REFERENCES members(member_id),
    CONSTRAINT fk_book FOREIGN KEY (book_id) REFERENCES books(book_id),
    CONSTRAINT uc_member_book UNIQUE (member_id, book_id, loan_date)
);

-- Create Fines Table (1-1: Each Loan can have one Fine)
CREATE TABLE fines (
    fine_id INT AUTO_INCREMENT PRIMARY KEY,
    loan_id INT NOT NULL UNIQUE,
    amount DECIMAL(6,2) NOT NULL,
    paid BOOLEAN DEFAULT FALSE,
    CONSTRAINT fk_loan FOREIGN KEY (loan_id) REFERENCES loans(loan_id)
);

-- Sample Data for Members
INSERT INTO members (name, email, join_date) VALUES
('Alice Smith', 'alice@example.com', '2023-01-10'),
('Bob Jones', 'bob@example.com', '2023-02-15');

-- Sample Data for Books
INSERT INTO books (title, author, isbn, published_year, copies_available) VALUES
('The Great Gatsby', 'F. Scott Fitzgerald', '9780743273565', 1925, 3),
('1984', 'George Orwell', '9780451524935', 1949, 2);

-- Sample Data for Loans
INSERT INTO loans (member_id, book_id, loan_date, due_date) VALUES
(1, 1, '2024-04-01', '2024-04-15'),
(2, 2, '2024-04-03', '2024-04-17');

-- Sample Data for Fines
INSERT INTO fines (loan_id, amount, paid) VALUES
(1, 5.00, FALSE);

2.SIMPLE CRUD API USING MY SQL + PROGRAMMING
-- Users Table
CREATE TABLE users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE
);

-- Tasks Table
CREATE TABLE tasks (
    task_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    due_date DATE,
    status ENUM('pending', 'in_progress', 'completed') DEFAULT 'pending',
    CONSTRAINT fk_user FOREIGN KEY (user_id) REFERENCES users(user_id)
);

3. CRUD API EXAMPLE (PYTHON + FAST API)
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import mysql.connector

app = FastAPI()

# Database connection setup
def get_db():
    return mysql.connector.connect(
        host="localhost",
        user="root",
        password="yourpassword",
        database="task_manager"
    )

# Pydantic models
class User(BaseModel):
    username: str
    email: str

class Task(BaseModel):
    user_id: int
    title: str
    description: str = None
    due_date: str = None
    status: str = "pending"

# CRUD for Users
@app.post("/users/")
def create_user(user: User):
    db = get_db()
    cursor = db.cursor()
    cursor.execute("INSERT INTO users (username, email) VALUES (%s, %s)", (user.username, user.email))
    db.commit()
    return {"message": "User created"}

@app.get("/users/{user_id}")
def get_user(user_id: int):
    db = get_db()
    cursor = db.cursor(dictionary=True)
    cursor.execute("SELECT * FROM users WHERE user_id = %s", (user_id,))
    result = cursor.fetchone()
    if not result:
        raise HTTPException(status_code=404, detail="User not found")
    return result

# CRUD for Tasks
@app.post("/tasks/")
def create_task(task: Task):
    db = get_db()
    cursor = db.cursor()
    cursor.execute(
        "INSERT INTO tasks (user_id, title, description, due_date, status) VALUES (%s, %s, %s, %s, %s)",
        (task.user_id, task.title, task.description, task.due_date, task.status)
    )
    db.commit()
    return {"message": "Task created"}

@app.get("/tasks/{task_id}")
def get_task(task_id: int):
    db = get_db()
    cursor = db.cursor(dictionary=True)
    cursor.execute("SELECT * FROM tasks WHERE task_id = %s", (task_id,))
    result = cursor.fetchone()
    if not result:
        raise HTTPException(status_code=404, detail="Task not found")
    return result

@app.put("/tasks/{task_id}")
def update_task(task_id: int, task: Task):
    db = get_db()
    cursor = db.cursor()
    cursor.execute(
        "UPDATE tasks SET title=%s, description=%s, due_date=%s, status=%s WHERE task_id=%s",
        (task.title, task.description, task.due_date, task.status, task_id)
    )
    db.commit()
    return {"message": "Task updated"}

@app.delete("/tasks/{task_id}")
def delete_task(task_id: int):
    db = get_db()
    cursor = db.cursor()
    cursor.execute("DELETE FROM tasks WHERE task_id=%s", (task_id,))
    db.commit()
    return {"message": "Task deleted"}

