# REST API with Flask and Python

This guide provides a step-by-step tutorial on how to create a simple REST API using Flask, a lightweight web application framework in Python.
Course for side-by-side project: https://www.udemy.com/course/rest-api-flask-and-python/

## Prerequisites

- Python 3.x installed on your system
- Basic understanding of Python programming
- Familiarity with RESTful principles
- A text editor or IDE (e.g., VS Code, PyCharm)

## Setup

### 1. Install Flask

First, you'll need to install Flask. You can do this using pip:

```bash
pip install Flask
```

### 2. Create a Project Directory
Create a directory for your project and navigate into it:

```bash
mkdir flask_rest_api
cd flask_rest_api
```

### 3. Create a Virtual Environment (Optional but Recommended)
It's a good practice to use a virtual environment for your projects:

```bash
python -m venv venv
```
Activate the virtual environment:

On Windows:
```bash
venv\Scripts\activate
```
On macOS/Linux:
```bash
source venv/bin/activate
```

### 4. Install Flask in the Virtual Environment
Once the virtual environment is activated, install Flask:
```bash
pip install Flask
```

## Building the API

### 1. Create the Flask App
In the project directory, create a file named app.py and add the following code:

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

@app.route('/')
def home():
    return "Welcome to the Flask REST API!"

if __name__ == '__main__':
    app.run(debug=True)
```

### 2. Define API Endpoints
Let's define a simple API to manage a list of items.

Update app.py:

```python
items = []

@app.route('/items', methods=['GET'])
def get_items():
    return jsonify(items)

@app.route('/items', methods=['POST'])
def add_item():
    item = request.json.get('item')
    items.append({'item': item})
    return jsonify({'item': item}), 201

@app.route('/items/<int:item_id>', methods=['GET'])
def get_item(item_id):
    if 0 <= item_id < len(items):
        return jsonify(items[item_id])
    else:
        return jsonify({'error': 'Item not found'}), 404

@app.route('/items/<int:item_id>', methods=['PUT'])
def update_item(item_id):
    if 0 <= item_id < len(items):
        item = request.json.get('item')
        items[item_id] = {'item': item}
        return jsonify(items[item_id])
    else:
        return jsonify({'error': 'Item not found'}), 404

@app.route('/items/<int:item_id>', methods=['DELETE'])
def delete_item(item_id):
    if 0 <= item_id < len(items):
        deleted_item = items.pop(item_id)
        return jsonify(deleted_item)
    else:
        return jsonify({'error': 'Item not found'}), 404

if __name__ == '__main__':
    app.run(debug=True)
```

### 3. Run the API
Run the Flask app by executing the following command:
```bash
python app.py
```
Your API will be running at `http://127.0.0.1:5000/`.

### 4. Test the API
You can use tools like **Insomnia** to test the API endpoints.
```
GET /items: Retrieve the list of items.
POST /items: Add a new item. Example JSON body: {"item": "Sample Item"}.
GET /items/<item_id>: Retrieve a specific item by its ID.
PUT /items/<item_id>: Update a specific item by its ID. Example JSON body: {"item": "Updated Item"}.
DELETE /items/<item_id>: Delete a specific item by its ID.
```

## Additional Notes

### 1. Using Insomnia Instead of Postman
Insomnia is an alternative to Postman for testing your API endpoints. You can create requests in Insomnia by specifying the method (GET, POST, etc.), the URL, and the body (if applicable). Insomnia provides a clean and intuitive interface for organizing and testing your API requests.

### 2. Docker
Docker allows you to containerize your Flask application, ensuring that it runs consistently across different environments. Here's how to set it up:

#### Create a Dockerfile in your project directory:
```Dockerfile
# Use an official Python runtime as a parent image
FROM python:3.9-slim

# Set the working directory
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Make port 5000 available to the world outside this container
EXPOSE 5000

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

#### Build and run your Docker container:
```bash
docker build -t flask-rest-api .
docker run -p 5000:5000 flask-rest-api
```
Your Flask application will now be running inside a Docker container.

### 3. Flask-Smorest
Flask-Smorest is an extension that adds a layer of convenience to Flask for building REST APIs with detailed documentation and input/output validation. To use it:

Install Flask-Smorest:
```bash
pip install flask-smorest
Update your app.py:
python
Copy code
from flask_smorest import Api, Blueprint

app = Flask(__name__)
api = Api(app)

blp = Blueprint('items', __name__, url_prefix='/items')

@blp.route('/')
def get_items():
    return jsonify(items)

api.register_blueprint(blp)
```
This sets up a blueprint for your API, allowing you to better organize your routes and documentation.

### 4. SQLAlchemy
SQLAlchemy is an ORM (Object-Relational Mapping) library that makes it easier to work with databases in Python. To integrate it into your Flask app:

#### Install SQLAlchemy:
```bash
pip install flask-sqlalchemy
```
#### Update app.py to include a database model:
```python
from flask_sqlalchemy import SQLAlchemy

app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///items.db'
db = SQLAlchemy(app)

class Item(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(80), nullable=False)

db.create_all()
```
#### Modify your endpoints to interact with the database:
```python
@app.route('/items', methods=['POST'])
def add_item():
    item_name = request.json.get('item')
    new_item = Item(name=item_name)
    db.session.add(new_item)
    db.session.commit()
    return jsonify({'id': new_item.id, 'item': new_item.name}), 201
```

### 5. JWT (JSON Web Tokens)
JWT is commonly used for securely transmitting information between parties as a JSON object. To add JWT authentication to your API:

#### Install Flask-JWT-Extended:
```bash
pip install flask-jwt-extended
```

#### Update app.py to include JWT authentication:
```python
from flask_jwt_extended import JWTManager, create_access_token, jwt_required

app.config['JWT_SECRET_KEY'] = 'your_jwt_secret_key'
jwt = JWTManager(app)

@app.route('/login', methods=['POST'])
def login():
    username = request.json.get('username')
    access_token = create_access_token(identity=username)
    return jsonify(access_token=access_token)

@app.route('/protected', methods=['GET'])
@jwt_required()
def protected():
    return jsonify(message="This is a protected route")
```
This will protect certain routes with JWT, requiring users to be authenticated to access them.

## Database Migrations with Alembic and Flask-Migrate
Alembic is a lightweight database migration tool for SQLAlchemy, and Flask-Migrate is an extension that integrates Alembic with Flask to handle database migrations smoothly. Migrations are necessary when you need to modify your database schema (e.g., adding a new column, changing a data type) while keeping your existing data intact.

### Steps to Incorporate Database Migrations

#### 1. Install Flask-Migrate:
First, install Flask-Migrate along with Flask-SQLAlchemy and Alembic:
```bash
pip install flask-migrate
```

#### 2. Initialize Flask-Migrate:
In your app.py, after setting up SQLAlchemy, initialize Flask-Migrate:
```python
from flask_migrate import Migrate

app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///items.db'
db = SQLAlchemy(app)
migrate = Migrate(app, db)
```

#### 3. Set Up the Migration Directory:
Once Flask-Migrate is initialized, you need to create the migration repository:
```bash
flask db init
```
This command creates a migrations directory in your project, which will store the migration files.

#### 4. Create a Migration Script:
Whenever you make changes to your models, create a new migration script to reflect those changes in the database:
```bash
flask db migrate -m "Your migration message"
```
This generates a migration script in the migrations/versions directory, which includes the necessary steps to apply the changes.

#### 5. Apply the Migration:
To apply the migration and update the database schema, run:
```bash
flask db upgrade
```
This command runs the migration scripts to apply the changes to your database.

#### 6. Rollback a Migration (Optional):
If you need to undo a migration, you can rollback to the previous state:
```bash
flask db downgrade
```
##### Benefits of Using Alembic and Flask-Migrate

* Version Control: Track changes to your database schema over time.
* Ease of Use: Simplifies the process of applying complex schema changes, even across different environments.
* Seamless Integration: Works directly with Flask and SQLAlchemy, allowing you to focus on application logic without worrying about manual database updates.

Incorporating Alembic and Flask-Migrate into your Flask project ensures that your database schema evolves smoothly and consistently with your application code.

## Conclusion
You've successfully created a basic REST API using Flask, along with additional tools like Insomnia, Docker, Flask-Smorest, SQLAlchemy, and JWT. These tools and libraries provide a robust foundation for building scalable and secure APIs.

