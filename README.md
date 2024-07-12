# HNGdevops-stage3

Here's a guide to help you complete the DevOps Stage 3 Task:

### Local Setup

1. **Install RabbitMQ and Celery**:
    - Install RabbitMQ: [RabbitMQ installation guide](https://www.rabbitmq.com/download.html).
    - Install Celery: `pip install celery`.

2. **Set up Python Application**:
    - Create a new Python project and set up a virtual environment.
    - Install necessary packages: `pip install Flask smtplib`.

3. **Python Application Functionality**:
    - Create a `app.py` file:
        ```python
        from flask import Flask, request
        import smtplib
        from celery import Celery
        import logging
        from datetime import datetime

        app = Flask(__name__)
        app.config['CELERY_BROKER_URL'] = 'pyamqp://guest@localhost//'
        app.config['CELERY_RESULT_BACKEND'] = 'rpc://'

        celery = Celery(app.name, broker=app.config['CELERY_BROKER_URL'])
        celery.conf.update(app.config)

        @celery.task
        def send_email(to_email):
            server = smtplib.SMTP('smtp.your-email.com', 587)
            server.starttls()
            server.login("youremail@domain.com", "password")
            message = "Subject: Test\n\nThis is a test email"
            server.sendmail("youremail@domain.com", to_email, message)
            server.quit()

        @app.route('/')
        def index():
            sendmail = request.args.get('sendmail')
            talktome = request.args.get('talktome')

            if sendmail:
                send_email.delay(sendmail)
                return f'Email will be sent to {sendmail}', 200

            if talktome:
                with open("/var/log/messaging_system.log", "a") as log_file:
                    log_file.write(f'{datetime.now()}\n')
                return 'Logged the current time.', 200

            return 'Invalid parameters', 400

        if __name__ == '__main__':
            app.run(debug=True)
        ```

### Nginx Configuration

1. **Install Nginx**:
    - Install Nginx: `sudo apt-get install nginx`.

2. **Configure Nginx**:
    - Create a new configuration file in `/etc/nginx/sites-available/`:
        ```
        server {
            listen 80;

            location / {
                proxy_pass http://127.0.0.1:5000;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
            }
        }
        ```
    - Enable the configuration: `sudo ln -s /etc/nginx/sites-available/your_config /etc/nginx/sites-enabled/`.
    - Restart Nginx: `sudo systemctl restart nginx`.

### Expose Endpoint using Ngrok

1. **Install Ngrok**:
    - Download and install Ngrok from [ngrok.com](https://ngrok.com/).

2. **Expose Local Server**:
    - Run Ngrok: `ngrok http 80`.
    - Note the provided forwarding URL for external access.

### Documentation and Walk-through

1. **Record Screen-captured Walk-through**:
    - Use a tool like OBS Studio or any screen recording software.
    - Ensure you cover the entire setup and deployment process:
        - RabbitMQ/Celery setup
        - Python application development
        - Nginx configuration
        - Sending email via SMTP
        - Logging current time
        - Exposing the endpoint using Ngrok

2. **Submission**:
    - Provide the Ngrok endpoint for testing.
    - Submit the screen recording through the [submission form](https://forms.gle/okhAFDnXQ5Dw6Ls39).

Ensure all specified features work correctly, document your code well, and make the screen recording clear and comprehensive.
