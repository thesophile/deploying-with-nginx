# deploying-with-nginx

```
sudo apt update
sudo apt upgrade
```

clone your project

```
git clone <url>
```

## install nginx

```
sudo apt install nginx
```

Now, a demo html will be served.





## Install supervisor and gunicorn

install supervisor

```
sudo apt install supervisor
```

Create a venv and activate it

Install django

```
pip3 install django
```

install gunicorn

```
pip3 install gunicorn
```

create a gunicorn conf file

```
cd /etc/supervisor/conf.d/
```

```
sudo touch gunicorn.conf
sudo nano gunicorn.conf
```



Enter this:

```
[program:gunicorn]
directory=/home/YOUR_USERNAME/YOUR_PROJECT_NAME 
command=/home/YOUR_USERNAME/YOUR_ENV_NAME/bin/gunicorn --workers 3 --bind unix:/home/YOUR_USERNAME/YOUR_PROJECT_NAME/app.sock YOUR_PROJECT_NAME.wsgi:application  
autostart=true
autorestart=true
stderr_logfile=/var/log/gunicorn/gunicorn.err.log
stdout_logfile=/var/log/gunicorn/gunicorn.out.log

[group:guni]
programs:gunicorn
```
> [!IMPORTANT]
> Replace YOUR_USERNAME and YOUR_ENV_NAME with your actual username and virtual env name

Make log directory

```
sudo mkdir /var/log/gunicorn
```

Apply changes

```
sudo supervisorctl reread
sudo supervisorctl update
```

check status

```
sudo supervisorctl status
```

change nginx conf

```
sudo nano /etc/nginx/nginx.conf
```

change user from www-data to root

make a django.conf folder in sites-available

```
cd /etc/nginx/sites-available
sudo touch django.conf
sudo nano django.conf
```

enter this:

``` 
server{

	listen 80;
	server_name YOUR_SERVER_NAME;

	location / {
		include proxy_params;
		proxy_pass http://unix:/home/YOUR_USERNAME/YOUR_PROJECT_NAME/app.sock;
	}

}
```

Test

```
sudo nginx -t
```

copy django.conf to sites-enabled

```
sudo ln django.conf /etc/nginx/sites-enabled
```

Restart nginx

```
sudo systemctl restart nginx
```

Restart supervisor

```
sudo systemctl restart supervisor
```

Now, your website should be served by nginx

## Static files
If you have static files, you have to collect them to a convenient location for nginx to access and define that path in django.conf

Usually static files are stored in /var/www/static. Create it using mkdir

```
sudo mkdir /var/www/static
```

Now since we are going to write to this directory as a user, we have to grant necessary permissions:



 




