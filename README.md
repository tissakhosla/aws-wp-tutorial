# mre-wp

[Tony Teaches Tech Tutorial](https://www.youtube.com/watch?v=_nU4OrQ68Us&ab_channel=TonyTeachesTech)

1. Create AWS t2.micro freee tier
2. Select Key Pair/Create New Pair
3. Add the following to .ssh/config, replace IP as needed

```
Host mre
	Hostname 3.80.168.97
	IdentityFile /home/ubuntu/.ssh/mresite.pem
	User ubuntu
```
4. Execute the following
```
chmod 400 mrekey.pem
ssh mre
<<now on AWS session>>
sudo apt update
sudo apt upgrade
sudo apt install nginx mariadb-server php-fpm php-mysql
cd /var/www/html/
sudo wget https://wordpress.org/latest.tar.gz
sudo tar -xzvf latest.tar.gz 
sudo rm -rf latest.tar.gz 
sudo chown -R www-data:www-data wordpress
sudo find wordpress/ -type d -exec chmod 755 {} \;
sudo find wordpress/ -type f -exec chmod 644 {} \;
cd wordpress/
mysql_secure_installation
### Create Password for Root if needed using
sudo su -
passwd ubuntu
ENTER NEW PWD: Etc...
```
4. Handle DB with
```
sudo mysql -u root -p
MariaDB [(none)]> create database example_db default character set utf8 collate utf8_unicode_ci;
MariaDB [(none)]> create user 'mre_wp'@'localhost' identified by 'piano';
MariaDB [(none)]> grant all privileges on example_db.* TO 'mre_wp'@'localhost';
MariaDB [(none)]> flush privileges;
MariaDB [(none)]> exit
```
5. Handle web server
```
cd /etc/nginx/sites-available
sudo vi wordpress.conf
```
6. In it:
```
upstream php-handler {
        server unix:/var/run/php/php8.1-fpm.sock;
}
server {
        listen 80;
        server_name 3.80.168.97;
        root /var/www/wordpress;
        index index.php;

        location / {
                try_files $uri $uri/ /index.php?$args;
        }

        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass php-handler;
        }
}

```
7. symlink `sudo ln -s /etc/nginx/sites-available/wordpress.conf /etc/nginx/sites-enabled/`
8. test `  49  sudo nginx -t`
9. restart nginx `sudo systemctl restart nginx`
10. Go to the IP and install wordpress!
11. Go to Site Health and see the critical issues? Run `sudo apt install php-curl php-dom php-mbstring php-imagick php-zip php-gd`
12. Deal with SSL using CertBot and LetsEncrypt once we have a Domain or and Elastic IP
```
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
PUT THE DOMAIN
```
