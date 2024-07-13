### Step-by-Step Guide for Setting Up Messaging System with RabbitMQ/Celery and Python Application Behind Nginx

#### Objective:
Deploy a Python application behind Nginx that interacts with RabbitMQ/Celery for email sending and logging functionality.

---

### 1. Local Setup

#### 1.1 Install RabbitMQ
- **Install RabbitMQ**:
  ```bash
  sudo apt-get update
  sudo apt-get install rabbitmq-server
  sudo systemctl enable rabbitmq-server
  sudo systemctl start rabbitmq-server
  ```

- **Verify Installation**:
  ```bash
  sudo rabbitmqctl status
  ```

#### 1.2 Install Celery
- **Install Celery using pip**:
  ```bash
  pip install celery
  ```

#### 1.3 Install Flask
- **Install Flask using pip**:
  ```bash
  pip install flask
  ```

#### 1.4 Install python-dotenv
- **Install python-dotenv to manage environment variables**:
  ```bash
  pip install python-dotenv
  ```

### 2. Set Up Python Application

#### 2.1 Create Application Structure
```bash
mkdir messaging_system
cd messaging_system
touch app.py tasks.py .env
```

#### 2.2 Define the Flask Application (app.py)
Create and edit `app.py`:
```python
from flask import Flask, request
from tasks import send_email_task
import logging
import datetime
from dotenv import load_dotenv
import os

# Load environment variables
load_dotenv()

app = Flask(__name__)

# Set up logging
logging.basicConfig(filename='/var/log/messaging_system.log', level=logging.INFO)

@app.route('/')
def index():
    if 'sendmail' in request.args:
        email = request.args.get('sendmail')
        send_email_task.delay(email)
        return f"Email to {email} has been queued."
    
    if 'talktome' in request.args:
        current_time = datetime.datetime.now().isoformat()
        logging.info(f"Current time logged: {current_time}")
        return f"Logged current time: {current_time}"
    
    return "Please provide a valid parameter (?sendmail or ?talktome)."

if __name__ == "__main__":
    app.run(debug=True)
```

#### 2.3 Define Celery Tasks (tasks.py)
Create and edit `tasks.py`:
```python
from celery import Celery
import smtplib
import os
import logging

# Initialize Celery
app = Celery('tasks', broker='pyamqp://guest@localhost//')

# Configure logging
logging.basicConfig(filename='/var/log/messaging_system.log', level=logging.INFO)

@app.task
def send_email_task(email):
    try:
        # Retrieve email credentials from environment variables
        email_user = os.getenv('EMAIL_USER')
        email_password = os.getenv('EMAIL_PASSWORD')

        if not email_user or not email_password:
            raise ValueError("Email credentials are not set in the environment variables.")

        # Set up the SMTP server
        server = smtplib.SMTP('smtp.gmail.com', 587)
        server.starttls()
        server.login(email_user, email_password)

        # Compose the email
        subject = 'Test Email'
        body = 'This is a test email.'
        message = f'Subject: {subject}\n\n{body}'

        # Send the email
        server.sendmail(email_user, email, message)
        server.quit()

        logging.info(f"Email sent successfully to {email}")
        return 'Email sent successfully!'
    except Exception as e:
        logging.error(f"Failed to send email to {email}: {e}")
        return str(e)
```

#### 2.4 Set Up Environment Variables (.env)
Create and edit the `.env` file:
```
EMAIL_USER=ougabriel@gmail.com
EMAIL_PASSWORD=H@ppy097
```

### 3. Configure Nginx

#### 3.1 Install Nginx
```bash
sudo apt-get install nginx
```

#### 3.2 Configure Nginx to Serve Flask Application
- **Create an Nginx configuration file**:
  ```bash
  sudo nano /etc/nginx/sites-available/messaging_system
  ```

- **Add the following configuration**:
  ```
  server {
      listen 80;
      server_name localhost;

      location / {
          proxy_pass http://127.0.0.1:5000;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
      }
  }
  ```

- **Enable the configuration**:
  ```bash
  sudo ln -s /etc/nginx/sites-available/messaging_system /etc/nginx/sites-enabled/
  sudo nginx -t
  sudo systemctl restart nginx
  ```

### 4. Expose the Application with ngrok

#### 4.1 Install ngrok
```bash
wget https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip
unzip ngrok-stable-linux-amd64.zip
sudo mv ngrok /usr/local/bin
```

#### 4.2 Expose Localhost
```bash
ngrok http 80
```

### 5. Documentation and Walk-through

#### 5.1 Record the Walk-through
- Use a screen recording tool like OBS Studio or any other screen capture software.
- Ensure the recording covers:
  - RabbitMQ/Celery setup
  - Python application development
  - Nginx configuration
  - Sending email via SMTP
  - Logging current time
  - Exposing the endpoint using ngrok

#### 5.2 Submission
- **Provide the ngrok endpoint**.
- **Submit the screen recording**.
- **Ensure all requirements are met**.

### 6. Submission Details
- **Submission Link**: [Google Form](https://forms.gle/okhAFDnXQ5Dw6Ls39)
- **Deadline**: Saturday, 12th July 2024, by 11:59 PM GMT

---

### Ports Required to be Open
- **RabbitMQ**:
  - Management Interface: `15672` (for accessing the RabbitMQ management UI)
  - AMQP Port: `5672` (for communication between RabbitMQ and Celery)
- **Flask**:
  - Development Server: `5000` (used for running the Flask development server)
- **Nginx**:
  - HTTP Port: `80` (for serving the application via HTTP)
- **ngrok**:
  - No specific port, but it tunnels your local Nginx server on port `80`.

#### Ensure Ports are Open

##### For Ubuntu/Debian:
- **Open Ports Using UFW (Uncomplicated Firewall)**
  ```bash
  sudo ufw allow 15672/tcp  # RabbitMQ Management Interface
  sudo ufw allow 5672/tcp   # RabbitMQ AMQP
  sudo ufw allow 80/tcp     # Nginx HTTP
  sudo ufw allow 5000/tcp   # Flask (only if accessing directly without Nginx)
  ```

##### Enable UFW if it's not already enabled:
```bash
sudo ufw enable
```

##### Check UFW Status:
```bash
sudo ufw status
```

---

### Final Notes
- Ensure that your email credentials and server details in `tasks.py` are correct and secure.
- Test the application thoroughly to confirm all functionalities before submitting.
- Keep the ngrok endpoint active for testing until you receive confirmation of submission.
