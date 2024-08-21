## Deployments with Render.com
This guide outlines the steps to deploy a Flask application using Render.com, Docker, and PostgreSQL. It covers setting up the web service, running Flask with Gunicorn, managing the database, and testing the deployment.

### 1. Creating a Render.com Web Service

1. Sign in to [Render.com](https://render.com/) and create a new account if you haven't already.
2. Navigate to the **Dashboard** and click on **New Web Service**.
3. Connect your GitHub/GitLab repository or manually enter the repository URL.
4. Select the branch you want to deploy, choose the runtime (Python 3), and specify the build and start commands:
   - **Build Command**: `pip install -r requirements.txt`
   - **Start Command**: `gunicorn -w 4 -b 0.0.0.0:8000 app:app`

5. Click on **Create Web Service** to start the deployment process.

### 2. How to Run Flask with Gunicorn in Docker

1. Create a `Dockerfile` in your project directory with the following content:

   ```Dockerfile
   # Use an official Python runtime as a parent image
   FROM python:3.9-slim

   # Set the working directory
   WORKDIR /app

   # Copy the current directory contents into the container at /app
   COPY . /app

   # Install dependencies
   RUN pip install --no-cache-dir -r requirements.txt

   # Expose port 8000 for Gunicorn
   EXPOSE 8000

   # Start the Flask app with Gunicorn
   CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:8000", "app:app"]
   ```
2. Build the Docker image:
```bash
docker build -t flask-gunicorn-app .
```

3. Run the container locally to test:
```bash
docker run -p 8000:8000 flask-gunicorn-app
```

### 3. Get a Deployed PostgreSQL Database

On Render.com, create a new **PostgreSQL Database** from the dashboard.
Save the connection string provided by Render, which will be used in your Flask app for connecting to the database.

### 4. Run the App and Database Locally with Docker Compose

1. Create a `docker-compose.yml` file in your project directory:
   
```yaml
version: '3.8'

services:
  web:
    build: .
    command: gunicorn -w 4 -b 0.0.0.0:8000 app:app
    ports:
      - "8000:8000"
    depends_on:
      - db

  db:
    image: postgres:13
    environment:
      POSTGRES_DB: your_db_name
      POSTGRES_USER: your_db_user
      POSTGRES_PASSWORD: your_db_password
    ports:
      - "5432:5432"
```

2. Run the containers:
```bash
docker-compose up --build
```

3. Your Flask app should now be running locally at `http://localhost:8000` with the PostgreSQL database accessible on port `5432`.

### 5. How to Run the Database Migrations in Your Compose Container

1. Access the web container:

```bash
docker-compose exec web bash
```

2. Run the migration commands inside the container:
```bash
flask db upgrade
```
This will apply the migrations to your local PostgreSQL database.

### 6. Connecting to Your Docker Compose Database with a Database Client

1. Use a database client (e.g., pgAdmin, DBeaver) and connect to your Docker Compose PostgreSQL database using:
Host: `localhost`
Port: `5432`
Database: `your_db_name`
Username: `your_db_user`
Password: `your_db_password`

2. This allows you to view and manage your database tables, schemas, and data.

### 7. Use PostgreSQL Locally and in Production

1. For local development, configure your Flask app to connect to the PostgreSQL database provided by Docker Compose:
```python
app.config['SQLALCHEMY_DATABASE_URI'] = 'postgresql://your_db_user:your_db_password@db/your_db_name'
```

2. For production, use the connection string provided by Render.com:
```python
app.config['SQLALCHEMY_DATABASE_URI'] = 'your_render_postgresql_connection_string'
```

3. Ensure your app dynamically switches between the local and production databases based on the environment.

### 8. Test the Finished Production App

1. Once your web service is deployed on Render.com, test the application by visiting the provided URL.
2. Check the logs on Render.com to monitor the application’s performance and troubleshoot any issues.
3. Test the API endpoints, database connections, and other functionality to ensure everything works as expected in the production environment.
