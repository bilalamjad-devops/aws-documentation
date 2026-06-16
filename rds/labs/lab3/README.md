# How to Connect Python Flask to a MySQL Docker Container (Step-by-Step)


In my previous guide, we successfully connected a basic Python script to an isolated MySQL database running inside a Docker container.


Today, we are taking our architecture to the next level. We are upgrading our project from a simple terminal script into a fully functional 2-Tier Web Application. We will use Python Flask to build a clean web interface, and Docker to run our background database layer.

Users will be able to type data into a web form, and Flask will store that data directly inside MySQL.

### Step 1: Start the MySQL Container

Instead of installing a heavy database package directly onto your computer, launch a MySQL container using Docker. This container will act as the storage layer for our web application.

Run the following command in your terminal:

```docker
docker run --name mysql-container -e MYSQL_ROOT_PASSWORD=password123 -p 3306:3306 -d mysql:latest
```
<img width="1112" height="358" alt="lab2 1" src="https://github.com/user-attachments/assets/d98deb80-3e6f-4ad0-8b1e-f6f9e6c52fb0" />

### Step 2: Install the Required Packages

Our web application needs both the Flask framework and a communication driver to speak to MySQL.

If you are using a modern Linux distribution that restricts global installations, run this command to install the required packages cleanly:


```python
python3 -m pip install Flask mysql-connector-python --break-system-packages --ignore-installed
```

<img width="1115" height="617" alt="lab2 3" src="https://github.com/user-attachments/assets/9d171784-fb67-41ee-9180-26a5d24c24c5" />


### Step 3: Create the Python Flask Application Files

We need two files to build this application: a backend Python script (app.py) and a frontend web layout page (index.html).

vi `app.py`

```phthon
from flask import Flask, render_template, request
import mysql.connector

app = Flask(__name__)

# Helper configuration function to manage database handshakes
def get_db_connection():
    db = mysql.connector.connect(
        host="127.0.0.1",       # Targets the localhost port forwarded from Docker
        user="root",
        password="password123" # Must align with your container credentials
    )
    cursor = db.cursor()
    cursor.execute("CREATE DATABASE IF NOT EXISTS web_db")
    cursor.execute("USE web_db")
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS web_table (
            id INT AUTO_INCREMENT PRIMARY KEY,
            content VARCHAR(255)
        )
    """)
    return db, cursor

# Route 1: Serves the initial home page web layout
@app.route('/')
def index():
    return render_template('index.html')

# Route 2: Catches incoming form text payloads and commits them to disk
@app.route('/submit', methods=['POST'])
def submit_data():
    if request.method == 'POST':
        form_input = request.form['user_data']
        
        try:
            db, cursor = get_db_connection()
            
            # Sanitized SQL parameterization to secure the database layer
            query = "INSERT INTO web_table (content) VALUES (%s)"
            cursor.execute(query, (form_input,))
            db.commit() # Safely commits data permanently to disk
            
            success_msg = f"🎉 Successfully saved '{form_input}' directly inside MySQL Container!"
            cursor.close()
            db.close()
            
        except Exception as e:
            success_msg = f"❌ Database connectivity failure: {e}"
            
        return render_template('index.html', message=success_msg)

if __name__ == '__main__':
    # Deploys development server on port 5000 across all interfaces
    app.run(host='0.0.0.0', port=5000, debug=True)
```

**templates/index.html**

```bash
mkdir templates
vi templates/index.html
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flask to MySQL Docker GUI</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background-color: #f0f2f5; margin: 50px; text-align: center; }
        .container { background: white; padding: 40px; border-radius: 12px; box-shadow: 0px 4px 15px rgba(0,0,0,0.05); display: inline-block; }
        h2 { color: #333; margin-bottom: 20px; }
        input[type="text"] { padding: 12px; width: 280px; border: 1px solid #ccd0d5; border-radius: 6px; font-size: 16px; }
        input[type="submit"] { padding: 12px 24px; background-color: #007bff; color: white; border: none; border-radius: 6px; font-size: 16px; cursor: pointer; margin-left: 10px; transition: 0.2s; }
        input[type="submit"]:hover { background-color: #0056b3; }
        .message { margin-top: 20px; color: #28a745; font-weight: bold; font-size: 15px; }
    </style>
</head>
<body>
    <div class="container">
        <h2>Submit Data to MySQL Docker Container</h2>
        <form action="/submit" method="POST">
            <input type="text" name="user_data" placeholder="Enter custom content here..." required>
            <input type="submit" value="Deploy to DB">
        </form>

        {% if message %}
            <p class="message">{{ message }}</p>
        {% endif %}
    </div>
</body>
</html>
```

### Step 4: Run the Application and Test Inputs

```python
python3 app.py
```

<img width="1110" height="253" alt="lab2 4" src="https://github.com/user-attachments/assets/d554a355-15b1-4d13-8561-38d42a545353" />


Open your web browser and go to: http://localhost:5000

You will see a clean user interface. Type a custom message or name into the text field (for example: Bilal Amjad) and click the Deploy to DB button. A green success text message will display indicating that the application successfully talked to the database container.


<img width="1598" height="754" alt="lab2 5" src="https://github.com/user-attachments/assets/e2feab35-de29-45bc-b827-3c491528d361" />
### Step 5: View Your Stored Data (The Final Proof)

Let's log directly inside our background Docker container to verify the data is sitting safely in our database.


**1. Open the Container Shell**

```docker
docker exec -it mysql-container mysql -u root -p
```



<img width="1111" height="311" alt="lab2 6" src="https://github.com/user-attachments/assets/36258166-32f7-4e01-97f6-687bf220589d" />



(Enter password: password123 when prompted)


**2. Select the App Database**

```mysql
USE web_db;
```



**3. View the Data Table**

```mysql
SELECT * FROM web_table;
```

<img width="1113" height="116" alt="lab2 7" src="https://github.com/user-attachments/assets/c0c9f5f2-e2d9-467c-88c8-aff74df6bbd1" />


You will see your clean data pop up on the screen:

<img width="1113" height="170" alt="lab2 8" src="https://github.com/user-attachments/assets/78e6667e-1719-4383-b965-f5b92e8896c4" />


```text
+----+------------------------------------+
| id | content                            |
+----+------------------------------------+
|  1 | Bilal Amjad                        |
+----+------------------------------------+
1 row in set (0.00 sec)
```

**4. Exit the Database Container**
```mysql
exit;
```


<img width="1114" height="52" alt="lab2 9" src="https://github.com/user-attachments/assets/212efee1-5202-4182-951f-10f2c6d09e52" />


Conclusion

Congratulations! 🚀 You have just built, verified, and successfully tested a multi-tier containerized web application workflow. You now understand how a frontend web server passes information down to a separate database service using custom ports.
