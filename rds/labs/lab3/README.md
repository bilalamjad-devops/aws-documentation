# Connecting Python Flask to a MySQL Docker Container: A Complete From-Scratch Guide

In my previous guide, we successfully built a CLI bridge between a basic Python script and an isolated MySQL Docker container.

Today, we are taking our architecture to the next level. We are upgrading our project from a simple CLI script into a fully functional 2-Tier Web Application using Python Flask as our frontend/backend application layer, and Docker for our persistent database layer.

### Step 1: Establish the Database 

First, let's spin up our isolated database. If you already have your container running from the previous lab, you can reuse it. If not, launch a fresh one with this terminal command:

```docker
docker run --name mysql-container -e MYSQL_ROOT_PASSWORD=password123 -p 3306:3306 -d mysql:latest
```

### app.py

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

### templates/index.html

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

### Step 5: Resolve Environments & Deploy

If you are running on a clean control plane or modern Linux environment, verify that your dependencies are mapped explicitly to your runtime using the PEP 668 bypass configurations:

```python
python3 -m pip install Flask mysql-connector-python --break-system-packages
```

Once installed, fire up your application server:

```python
python3 app.py
```

Open your browser and navigate to http://localhost:5000 (or use your cloud platform's web preview port option). Type your custom strings (e.g., Bilal Amjad

### Proof

```docker
docker exec -it mysql-container mysql -u root -p
```

(Enter password: password123 when prompted)

```mysql
USE web_db;
SELECT * FROM web_table;
```

+----+------------------------------------+
| id | content                            |
+----+------------------------------------+
|  1 | Bilal Amjad                        |
+----+------------------------------------+
1 row in set (0.00 sec)


Conclusion
Congratulations! 🚀 You have just built, verified, and successfully tested a multi-tier containerized application workflow.
