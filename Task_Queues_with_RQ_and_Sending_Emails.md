# Task Queues with RQ and Sending Emails

This guide covers how to set up task queues using RQ (Redis Queue) and send emails using Python and Mailgun. It includes setting up Redis, processing background tasks, and deploying a background worker on Render.com.

## 1. How to Send Emails with Python and Mailgun

1. Sign up for a [Mailgun](https://www.mailgun.com/) account and get your API key and domain.
2. Install the `requests` library, which is used to send HTTP requests:
   ```bash
   pip install requests
   ```
3. Create a function in Python to send an email using the Mailgun API:
```python
import requests

def send_email(to, subject, body):
    return requests.post(
        "https://api.mailgun.net/v3/YOUR_DOMAIN_NAME/messages",
        auth=("api", "YOUR_API_KEY"),
        data={"from": "Your Name <mailgun@YOUR_DOMAIN_NAME>",
              "to": [to],
              "subject": subject,
              "text": body})
```

4. Replace `YOUR_DOMAIN_NAME` and `YOUR_API_KEY` with the values from your Mailgun account.

## 2. How to Send Emails When Users Register

After a user registers, you might want to send a welcome email. This can be done by calling the send_email function after the registration logic:
```python
def register_user(email, username, password):
    # Registration logic (e.g., save the user in the database)
    send_email(email, "Welcome to Our Service", "Thank you for registering!")
```
This ensures that an email is automatically sent whenever a new user registers.

## 3. What is a Task Queue and Setting Up a Redis Database

1. Task Queue: A task queue is used to handle background tasks outside of the main application flow, allowing the main application to remain responsive.
2. Redis Database: Redis is an in-memory data structure store used as a database, cache, and message broker. RQ uses Redis to manage task queues.
3. To set up Redis locally, you can use Docker:
```bash
docker run -d -p 6379:6379 redis
```
Alternatively, you can install Redis directly on your machine. Follow the installation instructions for your operating system from the Redis website.

## 4. How to Populate and Consume the Task Queue with RQ

1. Install RQ:
```bash
pip install rq
```

2. Create a task function that you want to run in the background:
```python
from rq import Queue
from redis import Redis
from your_email_module import send_email

redis_conn = Redis()
q = Queue(connection=redis_conn)

def send_email_task(to, subject, body):
    # Your email sending logic
    send_email(to, subject, body)
```

3. To add a task to the queue:
```python
job = q.enqueue(send_email_task, "user@example.com", "Welcome", "Thank you for joining!")
```
4. The task is now queued and will be processed by an RQ worker.

## How to Process Background Tasks with the RQ Worker

1. To start processing tasks, run an RQ worker:
```bash
rq worker --with-scheduler
```
2. The worker will continuously monitor the Redis queue and execute tasks as they come in.

## 6. How to Send HTML Emails Using Mailgun and Python

1. To send HTML emails, modify the send_email function to accept and send HTML content:
```python
def send_html_email(to, subject, html_body):
    return requests.post(
        "https://api.mailgun.net/v3/YOUR_DOMAIN_NAME/messages",
        auth=("api", "YOUR_API_KEY"),
        data={"from": "Your Name <mailgun@YOUR_DOMAIN_NAME>",
              "to": [to],
              "subject": subject,
              "html": html_body})
```
2. Call this function when you need to send an HTML email:
```python
html_content = "<h1>Welcome</h1><p>Thank you for registering!</p>"
send_html_email("user@example.com", "Welcome to Our Service", html_content)
```

## 7. How to Deploy a Background Worker to Render.com

1. Create a Procfile in your project directory to specify the RQ worker command:
```plaintext
worker: rq worker --with-scheduler
```
2. Commit the `Procfile` to your repository and push it to your version control system (e.g., GitHub).
3. On Render.com:
* Create a new Background Worker from the dashboard.
* Connect it to your repository and select the branch to deploy.
* In the Start Command field, enter:
```bash
rq worker --with-scheduler
```
4. Deploy the worker, and it will start processing tasks in the background using Redis.
5. 
This setup allows you to handle tasks asynchronously, improve your app's performance, and ensure emails are sent reliably without blocking the main application.
