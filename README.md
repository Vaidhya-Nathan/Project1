# Project1
# Student Placement Selection 
#Isntall the libraries before starting
    !pip install faker
    !pip install streamlit
#Execute these two commands and make sure no errots

#Import modules required
    import mysql.connector
    import faker
    import random
    import streamlit as st
    import pandas as pd

#Check at this point whether no errors reported
#Initialize fake for further use
        fake = faker.Faker()

#connect with MySQL here we are using the parameters and varaibles as dictornay

        DB_CONFIG = {
            "host": "localhost",
            "user": "root",
            "password": "12345678",
            "database": "placement_db"
        }

#Define the connection string. This can be called appropriately


    def get_db_connection():
        return mysql.connector.connect(**DB_CONFIG)

###########CReate databased and tables written as function code
    def create_database():
        conn = mysql.connector.connect(host=DB_CONFIG["host"], user=DB_CONFIG["user"], password=DB_CONFIG["password"])
        cursor = conn.cursor()
        cursor.execute("CREATE DATABASE IF NOT EXISTS placement_db")
        conn.close()

    def create_tables():
        conn = get_db_connection()
        cursor = conn.cursor()
    
    # Create Students table
        cursor.execute('''
        CREATE TABLE IF NOT EXISTS Students (
            student_id INT AUTO_INCREMENT PRIMARY KEY,
            name VARCHAR(100),
            age INT,
            gender VARCHAR(10),
            email VARCHAR(100),
            phone VARCHAR(10),
            enrollment_year INT,
            course_batch VARCHAR(50),
            city VARCHAR(50),
            graduation_year INT
        )''')
    
    # Create Programming table
    cursor.execute('''
    CREATE TABLE IF NOT EXISTS Programming (
        programming_id INT AUTO_INCREMENT PRIMARY KEY,
        student_id INT,
        language VARCHAR(50),
        problems_solved INT,
        assessments_completed INT,
        mini_projects INT,
        certifications_earned INT,
        latest_project_score INT,
        FOREIGN KEY (student_id) REFERENCES Students(student_id)
    )''')
    
    # Create Soft Skills table
    cursor.execute('''
    CREATE TABLE IF NOT EXISTS SoftSkills (
        soft_skill_id INT AUTO_INCREMENT PRIMARY KEY,
        student_id INT,
        communication INT,
        teamwork INT,
        presentation INT,
        leadership INT,
        critical_thinking INT,
        interpersonal_skills INT,
        FOREIGN KEY (student_id) REFERENCES Students(student_id)
    )''')
    
    # Create Placements table
    cursor.execute('''
    CREATE TABLE IF NOT EXISTS Placements (
        placement_id INT AUTO_INCREMENT PRIMARY KEY,
        student_id INT,
        mock_interview_score INT,
        internships_completed INT,
        placement_status VARCHAR(20),
        company_name VARCHAR(100),
        placement_package INT,
        interview_rounds_cleared INT,
        placement_date DATE,
        FOREIGN KEY (student_id) REFERENCES Students(student_id)
    )''')
    
    conn.commit()
    conn.close()

    #Create function fetch_eligible_student with two parameters problem solved and soft skills -------Can be called later
    def fetch_eligible_students(problems_solved_threshold, soft_skills_threshold):
    conn = get_db_connection()
    cursor = conn.cursor(dictionary=True)
    query = '''
        SELECT s.student_id, s.name, p.problems_solved, ss.communication, ss.teamwork, ss.presentation
        FROM Students s
        JOIN Programming p ON s.student_id = p.student_id
        JOIN SoftSkills ss ON s.student_id = ss.student_id
        WHERE p.problems_solved > %s AND ss.communication > %s
    '''
    cursor.execute(query, (problems_solved_threshold, soft_skills_threshold))
    students = cursor.fetchall()
    conn.close()
    return students

#### define function to insert fake data  use the fake initialized at first step to insert the dummy data
#### Probems faced
#####1. Phone number length was problem so used numerifying the text and limiting the numbers to 10 digits
#####2. When writing database the database error with timeout so commit after every iteration
    def insert_fake_data(num_students=100, batch_size=10):
    conn = get_db_connection()
    cursor = conn.cursor()
    
    students = []
    for i in range(num_students):
        student = (
            fake.name(),
            random.randint(18, 25),
            random.choice(["Male", "Female", "Other"]),
            fake.email(),
            fake.numerify(text="##########"),  # Fix phone number length
            random.randint(2018, 2023),
            fake.word().capitalize() + " Batch",
            fake.city(),
            random.randint(2022, 2026)
        )
        students.append(student)

## functions were called

    create_database()
    create_tables()
    insert_fake_data(100)
    print("MySQL database created and populated with fake data including programming, soft skills, and placements!")
            if (i + 1) % batch_size == 0:  # Insert in batches
                cursor.executemany('''
                INSERT INTO Students (name, age, gender, email, phone, enrollment_year, course_batch, city, graduation_year)
                VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s)''', students)
                conn.commit()
                students = []  # Clear batch

    conn.commit() 
    conn.close()

  # struck at streamlit application
