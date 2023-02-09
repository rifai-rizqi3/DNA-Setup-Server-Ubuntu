# Setup-Server-Ubuntu
Nginx, Php, PhpMyAdmin, MariaDB, Postgres, PhpPgAdmin Installation] Installation of nginx, php, phpmyadmin, phppdadmin, mysql and postgres in ubuntu 22.04 #public #loclhost #ubuntu #php #mariadb #installation #phppgadmin #phpmyadmin #postgres #nginx

## Installation and Configuration

1. Update System

	```bash
    sudo apt install update && sudo apt upgrade -y
    ```


2. Install Nginx

	```bash
    sudo apt install nginx -y
    ```


3. Test on the browser [Click Me](http://localhost/)


5. Install Php7.0
    ```bash
    sudo apt install php7.0 php7.0-fpm -y
    ```

6. Install Mariadb
    ```bash
    sudo apt install mariadb-server mariadb-client php7.0-mysql -y
    ```
7. Configure php.ini which is located on `/etc/php/7.0/fpm/php.ini`
    ```bash
    sudo sed -i "s/display_errors = .*/display_errors = On/" /etc/php/7.0/fpm/php.ini
    #sudo sed -i "s/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/" /etc/php/7.0/fpm/php.ini
    ```
8. Setup Nginx Configuration
    * Open File
      ```
      sudo vim /etc/nginx/sites-available/default
      ```
    * Replace with the folowing Code
      ```
      server {
          listen 80;
          server_name localhost;
          root /extra/localhost/php;
          index index.html index.htm index.php;
          location / {
                  try_files $uri $uri/ =404;
          }

          location ~ \.php$ {
                  include snippets/fastcgi-php.conf;
                  fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
          }
          location /pma {
                  root /var/www/html;

                  index index.php;
                  try_files $uri $uri/ =404;

                  location ~ ^/pma/(doc|sql|setup)/ {
                          deny all;
                  }

                  location ~ ^/pma/(.+\.php)$ {
                          include snippets/fastcgi-php.conf;
                          fastcgi_pass unix:/run/php/php7.0-fpm.sock;
                          fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                          include fastcgi_params;
                  }
          }
          location /ppa {
                  root /var/www/html;
                  index index.php;
                  try_files $uri $uri/ =404;

                  location ~ ^/ppa/(doc|sql|setup)/ {
                          deny all;
                  }

                  location ~ ^/ppa/(.+\.php)$ {
                          include snippets/fastcgi-php.conf;
                          fastcgi_pass unix:/run/php/php7.0-fpm.sock;
                          fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                          include fastcgi_params;
                  }
           }
          location ~ /\.ht {
              deny all;
          }

      }
      ```
    * Remove previous smlink and add new one
      ```
      sudo rm -f /etc/nginx/sites-enabled/default
      sudo ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/
      ```
9. Add www-data to username group:
    ```
    sudo gpasswd -a www-data rajan
    ```
10. Make sure that username group can enter all directories along the path:
    ```
    sudo chmod g+x /home && chmod g+x /home/rajan && chmod g+x /home/rajan/Dropbox && chmod g+x /home/rajan/Dropbox/yipl && chmod g+x /home/rajan/Dropbox/yipl/php
    ```
11. Install Phpmyadmin (During Installation You are asked to enter password, and choose yes in other case)
    ```
    sudo apt-get install phpmyadmin php-mbstring php-gettext -y
    ```
12. Install Postgres and dependencies
    ```
    sudo apt install postgresql postgresql-contrib -y
    ```
13. Using Cli to login
    ```
    sudo -i -u postgres
    psql
    ```
14. Install Prerequisites and phpPgAdmin
    ```
    sudo apt install php-pgsql phppgadmin -y
    ```
15. Resolve these kind of errors:
    > ERROR 1045: Access denied for user: 'root@localhost' (Using password: NO)
    
    or
    
    > ERROR 1045: Access denied for user: 'root@localhost' (Using password: YES)
    ```
    sudo mysql
    use mysql
    UPDATE user SET Password=PASSWORD('root') WHERE User='root';
    UPDATE user SET plugin='' WHERE User='root';
    flush privileges;
    exit
    ```
16. Setup password for user postgres
    ```
    sudo -u postgres psql postgres
    \password postgres
    \q
    ```
17. Set `extra_login_security = false` in `/etc/phppgadmin/config.inc.php`
    ```
    sudo sed -i "s/$conf\['extra_login_security'\] = true;/$conf\['extra_login_security'\] = false;/" /etc/phppgadmin/config.inc.php
    ```
18. Set `peer` to `MD5` in `/etc/postgresql/9.5/main/pg_hba.conf`
    ```
    sudo sed -i "s/postgres                                peer/postgres                                md5/" /etc/postgresql/9.5/main/pg_hba.conf
    ```
19. Simlink of PhpMyAdmin and PhpPgAdmin
```
sudo ln -s /usr/share/phpmyadmin /var/www/html/pma
sudo ln -s /usr/share/phppgadmin /var/www/html/ppa
```
20. Restart Every thing
    ```
    sudo service nginx restart
    sudo service mysql restart
    sudo service php7.0-fpm restart
    sudo service postgresql restart
    ```


1. Making Postgres accessable in network

    Change listen_addresses from localhost to * in

    ```
     sudo vim /etc/postgresql/10/main/postgresql.conf
     listen_addresses = '*'
    ```

    ```
    sudo vim /etc/postgresql/10/main/pg_hba.conf 
    host   all              postgres      0.0.0.0/0               md5
    ```

