**Step 1: Run your MySQL Database**

```mysql
docker run --name my-local-mysql -e MYSQL_ROOT_PASSWORD=password123 -p 3306:3306 -d mysql:latest
```

**Step 2: Bypass the Python Error & Install Driver**

Your Python app does not know how to speak "MySQL language" yet. We must install the translator driver. Since you are on a Linux control plane, run this command to bypass that error you saw earlier

```mysql
pip install mysql-connector-python --break-system-packages
```

**Step 3: Write the Input Application Code**

`vi app.py`


```python
import mysql.connector

# 1. Connect to your running Docker MySQL container
try:
    db = mysql.connector.connect(
        host="127.0.0.1",       # This points to your local machine / port
        user="root",
        password="password123" # Must match your Docker password flag
    )
    cursor = db.cursor()

    # 2. Setup Database & Table structures automatically
    cursor.execute("CREATE DATABASE IF NOT EXISTS my_db")
    cursor.execute("USE my_db")
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS my_table (
            id INT AUTO_INCREMENT PRIMARY KEY,
            student_name VARCHAR(100)
        )
    """)

    # 3. GET DATA FROM USER (This takes your input)
    print("\n=====================================")
    user_input = input("Enter data to save in DB: ")
    print("=====================================\n")

    # 4. Insert your input data safely into MySQL
    query = "INSERT INTO my_table (student_name) VALUES (%s)"
    cursor.execute(query, (user_input,))
    db.commit() # This saves it permanently!
    print(f"🎉 Success! '{user_input}' has been saved to MySQL.")


    cursor.close()
    db.close()
```



**Step 4: Run your App and Test Inputs**

Execute your script using python3:

```python
python3 app.py
```


What will happen next:

- The terminal will pause and ask: Enter student name to save in DB:
- Type any name (e.g., Ali Khan) and hit Enter.
- The script will inject that data into the Docker container, pull the list back out, and display your entry on the screen.



Step 1: Log into the MySQL Container

```docker
docker exec -it my-local-mysql mysql -u root -p
```

- What happens next: The terminal will ask for a Enter password:.
- Type your password: Type password123 and press Enter.

When successful, your prompt will change to: mysql>


Step 2: Look at the Databases

List all available databases to find the one your Python app created (my_db):


```mysql
SHOW DATABASES;
```

Step 3: Open your Database

Tell MySQL you want to work inside your specific database:

```mysql
USE my_db;
```

Step 4: Check the Tables

Verify that the my_table table exists inside the database:

```mysql
SHOW TABLES;
```

Step 5: View Your Stored Data (The Final Proof)

```mysql
SELECT * FROM my_table;
```

You will see a clean table output matching exactly what your script inserted:

```text
+----+--------------+

| id | student_name |
+----+--------------+

|  1 | Ali Khan     |
+----+--------------+
1 row in set (0.00 sec)
```

Step 6: Exit the Container

```mysql
exit;
```

