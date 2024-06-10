# deploying-with-nginx

```
sudo apt update
sudo apt upgrade
```

clone your project

```
git clone <url>
```

# install nginx

```
sudo apt install nginx
```

Now, a demo html will be served.





# Install supervisor and gunicorn

```
sudo apt install supervisor
```

Create a venv and activate it

```
pip3 install gunicorn
```

```
cd /etc/supervisor/conf.d/
```

create a gunicorn conf file

```
sudo touch gunicorn.conf
sudo nano gunicorn.conf
```



Enter this:

```
[program:gunicorn]
directory=/home/YOUR_USERNAME/YOUR_PROJECT_NAME 
command=/home/YOUR_USERNAME/YOUR_ENV_NAME/bin/gunicorn --workers 3 --bind unix:/home/YOUR_USERNAME/YOUR_PROJECT_NAME/app.sock elevate.wsgi:application  
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
sudo nano /etc/sites-available
sudo touch django.conf
sudo nano django.conf
```

enter this:

``` 
server{

	listen 80;
	server_name ;

	location / {
		include proxy_params;
		proxy_pass http://unix:/home/<b>username</b>/elevate/app.sock;
	}

}
```

Test

```
sudo nginx -t
```

Run

```
sudo ln django.conf /etc/nginx/sites-enabled
sudo service nginx restart
````

## Additional

Restart supervisor

```
sudo systemctl restart supervisor
```

Restart nginx

```
sudo systemctk restart nginx
```


 




