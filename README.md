# FlaskAppDeploy

# update your local packages 
1.sudo apt-get update

# install python3.8.0

sudo apt update
sudo apt install software-properties-common
sudo apt install python3.8
python3.8 --version

# install dependencies
2. sudo apt-get install python3-pip python3-dev nginx
3. python3 -m venv env
4. sudo pip3 install virtualenv
5. source env/bin/activate
6. pip3 install gunicorn
7. gunicorn --bind 0.0.0.0:5000 wsgi:app
8. deactivate

#  Create a systemd Unit File
sudo nano /etc/systemd/system/app.service

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

# We can now start the Gunicorn service we created and enable it so that it starts at boot:
sudo systemctl start app
sudo systemctl enable app

# Configuring Nginx
sudo nano /etc/nginx/sites-available/app

server {
    listen 80;
    server_name server_domain_or_IP;

location / {
  include proxy_params;
  proxy_pass http://unix:/home/ubuntu/work/deployment/src/app.sock;
    }
}

# enable Nginx server block:
sudo ln -s /etc/nginx/sites-available/app /etc/nginx/sites-enabled
sudo systemctl restart nginx
sudo ufw allow 'Nginx Full'

# Ec2 Security Groups (Inbound Rules)
Set Type HTTP, Protocol TCP, Port range 80, and Source to “0.0.0.0/0”.

Set Type HTTP, Protocol TCP, Port range 80, and Source to “::/0”.

Set Type Custom TCP, Protocol TCP, Port range 8080, and Source to “0.0.0.0/0”.

Set Type SSH, Protocol TCP, Port range 22, and Source to “0.0.0.0/0”.

Set Type HTTPS, Protocol TCP, Port range 443, and Source to “0.0.0.0/0”.
