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
sudp apt install nginx
```
Now, a demo html will be served.





# Install supervisor 
```
sudo apt install supervisor
```

# Install gunicorn
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
command
```

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
content
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


 




