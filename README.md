# FlaskAppDeploy step by step

# Update your local packages 

1. sudo apt-get update

# Install Python3.8.0

1. sudo apt update
2. sudo apt install software-properties-common
3. sudo apt install python3.8
4. python3.8 --version
5. sudo apt-get install nginx

# Install Postgres

1. sudo apt install postgresql postgresql-contrib
2. sudo -i -u postgres
3. psql -U postgres
4. ALTER USER postgres PASSWORD '<new-password>';
5. CREATE DATABASE database_name
 
### To access database after step-2 :
    
1. psql -U postgres -d database_name

# Run migraion

### Note : make .env file as referenced from dev.env
 
1. export PYTHONPATH=$PYTHONPATH:{full_path_of_project}
2. alembic upgrade head
 
# Create the WSGI Entry Point

### Folder Structure:
```
src
  |____ app.py
  |____ wsgi.py
```

1. sudo nano wsgi.py

```
from app import app

if __name__ == "__main__":
    app.run()
 ```
 
# Install Dependencies

1. sudo apt install virtualenv
2. virtualenv -p python3.8 env
3. source env/bin/activate
4. pip3 install gunicorn
5. gunicorn --bind 0.0.0.0:5000 wsgi:app
6. deactivate

#  Create a systemd Unit File
1. sudo nano /etc/systemd/system/app.service

```
[Unit]

Description=Gunicorn instance to serve myproject
After=network.target

[Service]

User=yourusername

Group=www-data

WorkingDirectory=/home/ubuntu/work/deployment/src
Environment="PATH=/home/ubuntu/work/deployment/src/myprojectvenv/bin"

ExecStart=/home/ubuntu/work/deployment/src/myprojectvenv/bin/gunicorn --workers 3 --bind unix:app.sock -m 007 wsgi:app

[Install]
WantedBy=multi-user.target
```

# We can now start the Gunicorn service we created and enable it so that it starts at boot:
1. sudo systemctl start app
2. sudo systemctl enable app

# Configuring Nginx
sudo nano /etc/nginx/sites-available/app

```
server {
    listen 80;
    server_name server_domain_or_IP;

location / {
  include proxy_params;
  proxy_pass http://unix:/home/ubuntu/work/deployment/src/app.sock;
    }
}
```

# Enable Nginx server block:
1. sudo ln -s /etc/nginx/sites-available/app /etc/nginx/sites-enabled
2. sudo systemctl restart nginx
3. sudo ufw allow 'Nginx Full'

# Ec2 Security Groups (Inbound Rules)
    
Set Type HTTP, Protocol TCP, Port range 80, and Source to ???0.0.0.0/0???.

Set Type HTTP, Protocol TCP, Port range 80, and Source to ???::/0???.

Set Type Custom TCP, Protocol TCP, Port range 8080, and Source to ???0.0.0.0/0???.

Set Type SSH, Protocol TCP, Port range 22, and Source to ???0.0.0.0/0???.

Set Type HTTPS, Protocol TCP, Port range 443, and Source to ???0.0.0.0/0???.
