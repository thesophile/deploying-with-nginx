# Deploying with nginx

## Basic setup


Update and upgrade  
```
sudo apt update
sudo apt upgrade
```

> if you don't have git installed:
> ```
> sudo apt install git
> ```
> If that's not working, check your outbound rules and permit outbound traffic 

clone your project

```
git clone <repository_url>
```
> [!TIP]
> to clone from a particular branch, use
> ```
> git clone -b <branch_name> <repository_url>
> ```


### install nginx

```
sudo apt install nginx
```

>[!Tip]
>Now, a demo html will be served.



### install supervisor

```
sudo apt install supervisor
```

### install and configure gunicorn

Create a venv and activate it
```
python3 -m venv myenv
```
```
source myenv/bin/activate
```

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

### configure nginx

<!--
```
sudo nano /etc/nginx/nginx.conf
```

change user from www-data to root
-->

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

> [!TIP]
> YOUR_SERVER_NAME is your IP address (like `192.168.1.1`) or your registered domain name (like `www.example.com`)

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

Now, your website should be served by nginx at `http://<Server_IP_address>/`

## Static files
If you have static files, you have to collect them to a convenient location for nginx to access and define that path in django.conf

Usually static files are stored in /var/www/static. Create it using mkdir

```
sudo mkdir /var/www/static
```

Now since we are going to write to this directory as a user, we have to grant necessary permissions:

```
sudo chown -R USERNAME:root /var/www/static
```

Collect static:
Usually static files are dispersed in different locations across your django project. You have to collect all of them to a single location for nginx to serve.

First tell django which are your static 
files. Then specify to which folder your files should be collected to

Your django project's settings.py should contain:
```
STATIC_URL = 'static/'

STATICFILES_DIRS = [
    BASE_DIR / "manim" / "static" / "css",
]

STATIC_ROOT = '/var/www/static/'
```
Then collect it:

```
python3 manage.py collectstatic
```
Now, the static files are moved to /var/www/static
Next, tell django to serve these by editing the django.conf

```
sudo nano /etc/nginx/sites-available/django.conf
```
Add this:
```
location /static/ {
        alias /var/www/static/;  # Adjust to your STATIC_ROOT
    }
```

Now you can restart nginx

```
sudo systemctl restart nginx
```
## Media Files
To Display media files, for example files contained in a folder called 'media' directly inside the project directory, do as follows:

1. Add this to your settings.py of your project:
```
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```   
2. Add this to your django.conf:
```
        location /media/ {
        alias /home/admin/CloudPy/media/;
        }

``` 

## Domain Name

1. Register for a Domain at somewhere like [Hostinger.in](https://www.hostinger.in/)

2. Go to the panel and add an A record pointing to the IP address of your server

![image](https://github.com/thesophile/deploying-with-nginx/assets/90457368/fb6f605a-e2ec-4d4b-8e5a-57f684d41b3b)

![image](https://github.com/thesophile/deploying-with-nginx/assets/90457368/5c06df3a-3801-4cd5-9c82-4d51f34dd35d)

If you have to redirect www requests, add a CNAME record too:

![image](https://github.com/thesophile/deploying-with-nginx/assets/90457368/dff56bd0-0747-43e8-98d8-a3ce180b7d82)

 


Edit django.conf:

``` 
server{

	listen 80;
	server_name DOMAIN.COM WWW.DOMAIN.COM ;

	location / {
		include proxy_params;
		proxy_pass http://unix:/home/YOUR_USERNAME/YOUR_PROJECT_NAME/app.sock;
	}

}
```

## To acquire SSL certificate (https)

Use [certbot](https://certbot.eff.org/instructions?ws=nginx&os=debianbuster)

>[!WARNING]
> Delete the default file (Next to django.conf) before running the `sudo certbot --nginx` command because, it can mistakenly write to default instead of to django.conf.  

Make sure your django.conf looks something like this:
 
```
server{
        server_name cloudpy.online www.cloudpy.online;

        listen 443 ssl;
        ssl_certificate /etc/letsencrypt/live/cloudpy.online/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/cloudpy.online/privkey.pem;
        include /etc/letsencrypt/options-ssl-nginx.conf;
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

        location / {
                include proxy_params;
                proxy_pass http://unix:/home/admin/CloudPy/app.sock;
        }

        location /static/ {
                alias /var/www/static/;
                expires 7d;
        }
}

server {
    listen 80;
    server_name cloudpy.online www.cloudpy.online;
    return 301 https://$host$request_uri;
}
```


