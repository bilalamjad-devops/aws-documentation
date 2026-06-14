# Connecting Python to a MySQL Docker Container: A Complete From-Scratch Guide

<img width="1110" height="358" alt="lab1 - 1" src="https://github.com/user-attachments/assets/c2480f94-34d7-4773-ae0d-734fa2ba9b38" />



When starting out in DevOps or backend development, one of the most important milestones is learning how to make an application talk to an isolated database.

In this guide, we are going to do exactly that. We will launch a MySQL database inside a background Docker container, write a simple Python script to send custom input data straight into it, and manually step inside the database to verify our data is safely stored.

We will also tackle a common, modern Linux roadblock along the way: Python’s PEP 668 environment error. Let’s build it from scratch.


### Step 1: Run your MySQL Database

Instead of downloading heavy installers onto your local machine, we can use Docker to spin up a completely isolated MySQL database engine in seconds.

Run this command in your terminal:

```docker
docker run --name mysql-container -e MYSQL_ROOT_PASSWORD=password123 -p 3306:3306 -d mysql:latest
```

<img width="1110" height="358" alt="lab1 - 1" src="https://github.com/user-attachments/assets/c8d651ce-546f-4fa0-9a93-1c0cfefe73cd" />


**What this flag configuration means:**

- `--name mysql-container`: Gives our running container a friendly name.
- `-e MYSQL_ROOT_PASSWORD=password123`: Sets the master login password inside the database.
- `-p 3306:3306`: Forwards port 3306 from inside the container to our local machine so our script can reach it.
- `-d`: Runs the container in "detached" mode (silently in the background).


### Step 2: Bypass the Python Error & Install Driver

Our Python app cannot speak "MySQL language" natively. We need to install an official communication driver called `mysql-connector-python`.


If you are working on a modern Linux server or environment as a root user, running standard pip installs will trigger a frustrating block: `error: externally-managed-environment` (PEP 668). The OS does this to prevent you from breaking system packages.

To bypass this safely in your testing environment, use the `--break-system-packages` flag:

```python
pip install mysql-connector-python --break-system-packages
```

<img width="1111" height="207" alt="lab1 - 2" src="https://github.com/user-attachments/assets/79b98035-b832-4428-bd92-b4109395cdf8" />


### Step 3: Write the Input Application Code

Now, let's create our application file. Open a clean script file using the terminal text editor:

`vi app.py`

Paste the following simplified Python code inside it. This script handles the network connection, automatically sets up a generic database and table structure, and prompts you to type whatever data you want to save.

```python
import mysql.connector

try:
    # 1. Connect to your running Docker MySQL container
    db = mysql.connector.connect(
        host="127.0.0.1",       # Forces a network connection to localhost
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
            data_content VARCHAR(100)
        )
    """)

    # 3. Get raw data from the user terminal input
    print("\n=====================================")
    user_input = input("Enter data to save in DB: ")
    print("=====================================\n")

    # 4. Insert your input data safely using a parameterized query
    query = "INSERT INTO my_table (data_content) VALUES (%s)"
    cursor.execute(query, (user_input,))
    db.commit() # Saves changes permanently to the database disk
    print(f"🎉 Success! '{user_input}' has been safely saved to MySQL.")

    cursor.close()
    db.close()

except Exception as e:
    print(f"❌ Connection Failed! Error details: {e}")
```


<img width="1113" height="224" alt="lab1 - 3" src="https://github.com/user-attachments/assets/c4334207-2547-404a-b396-a21659ab0577" />


### Step 4: Run your App and Test Inputs

Execute your script directly from your terminal:

```python
python3 app.py
```

**What to expect next:**


1. The terminal execution will pause and display: Enter data to save in DB:
2. Type any string or name (e.g., Bilal Amjad) and press Enter.
3. The script will establish a connection over port 3306, inject your text into the background container, and output a confirmation success message.


### Step 5: Peek Inside the Container to Verify (The Final Proof)

To prove that the data didn't just disappear into thin air, we can log directly inside the isolated Docker container and inspect the raw SQL tables.

**1. Log into the MySQL Container shell**

```docker
docker exec -it mysql-container mysql -u root -p
```

<img width="1111" height="348" alt="lab1 - 4" src="https://github.com/user-attachments/assets/2222bacc-6acf-48ba-8abc-9d244de244f5" />


Note: The terminal will prompt you for `Enter password:`. Type `password123` and press Enter. For security reasons, the cursor will not show characters or asterisks while typing.

**2. Find your Database**

List all active databases to find the one our script automatically generated:

```mysql
SHOW DATABASES;
```
"lab1 - 5" src="https://github.com/user-attachments/assets/d648e8ba-13d9-4645-ad9d-46c55d306b97" />

**3. Open the Database**

```mysql
USE my_db;
```
<img width="1112" height="250" alt=<img width="1113" height="141" alt="lab1 - 6" src="https://github.com/user-attachments/assets/a505b34f-6019-4ac4-a066-f3647e017582" />

**4. Check the Tables**

Verify that our generic table was created:

```mysql
SHOW TABLES;
```



<img width="1113" height="204" alt="lab1 - 7" src="https://github.com/user-attachments/assets/6b609d7a-cf3e-4e54-9bfe-9fe237d72941" />



**5. Query the Row Results**

Run a standard selection query to print out the final proof:

```mysql
SELECT * FROM my_table;
```

<img width="1113" height="173" alt="lab1 - 8" src="https://github.com/user-attachments/assets/3344e5fe-426d-409a-8ba0-f87aeaef034b" />


You will see a clean SQL output table containing your exact custom input data:

```text
+----+--------------+

| id | data_content |
+----+--------------+

|  1 | Bilal Amjad     |
+----+--------------+
1 row in set (0.00 sec)
```

**6. Exit the Container**

Once verified, type this to exit the container shell and return safely to your local machine terminal prompt:

```docker
exit;
```
<img width="1113" height="49" alt="lab1 - 9" src="https://github.com/user-attachments/assets/db710606-15d4-4589-bcaa-12aaf3143b25" />

## Conclusion

Congratulations! You have successfully built a bridge between a frontend application script and an isolated database infrastructure component. Understanding how ports connect (3306), handling driver requirements, and overcoming Linux environment blocks are vital foundational building blocks for a career in DevOps.





















