# WordPress Project
I created a WordPress site on a Ubuntu Server 20.04 LTS running in the us-east-1 (N.Virgina) region on AWS Cloud. I installed an Apache2 webserver on my virtual server with MySQL, Nginx and PHP. I allocated an Elastic IP address for my instance and registered a domain name using Route 53 (AWS) for the WordPress site. You can access it using - 3.220.165.196 or http://techwithbashiir.com/.  

I undertook this personal project to demonstrate my AWS and Linux system administration skills. The key goals of this project were that:  

- The WordPress site had to be highly available and operational at all times. 

- It must meet the minimal performance requirements for reliability purposes.


## AWS Architecture: 

Below is the architecture of the WordPress application which was developed using Lucidchart. It demonstrates how the different AWS services interact and connect to provide the service to the user. 

![WordPress App](https://user-images.githubusercontent.com/89197223/146795202-f5d72b74-9e88-42a4-b766-764f9b96e9b7.png)
  


## 1.Linux Server 
### 1.a: Provisioning an Ubuntu Server on AWS
The first step of the project was to provision an Ubuntu server using EC2 Compute service by using my free-tier AWS account. I selected the Ubuntu 20.4 LTS AMI t.2 micro isntance type and deployed the server in the us-east-1 (N.Virgina) Region. I kept the default instance settings for the storage and network configuration ( 8GB SSD gp2 and default VPC). I then added a simple tag in order to identify my resource.   

I also configured the security group of the instance by configuring inbound rules in order to allow SSH, HTTP & HTTPS traffic into my server. This kept port 22, 80 and 443 open for incoming traffic from all hosts (0.0.0.0/0). Once this was complete I launched the ubuntu server and it was up and running very quickly. 

<img width="1157" alt="Screenshot 2021-12-13 at 10 47 28" src="https://user-images.githubusercontent.com/89197223/145798935-c119fc55-abc4-4727-b1d6-ea67c79360b5.png">

### 1.b: Associating an Elastic IP Address with the Ubuntu server 
I went to Network & security under EC2 compute service and selected Elastic IP. This was to allocate a fixed public IPV4 address that I can attach to my running Ubuntu server. It provided me the 3.220.165.196 IP address which will be strictly for my instance, and it will not change in the event of failure or if the instance is stopped until I release the address. 

This also simplifies management for me as I don't need to update any of my sessions with a new IP address when connecting to the remote server.

<img width="1424" alt="Screenshot 2021-12-14 at 12 51 15" src="https://user-images.githubusercontent.com/89197223/146001711-f3ba37f3-2baf-458b-bced-dc2e45b43243.png">


### 1.c: Connecting to the remote server using SSH 
I downloaded the corresponding private key .pem file to my local machine to use when connecting to the remote server using SSH. The key was stored in my Downloads directory with 644 permission levels that were not secure. I modified the private key file to make groups and others have no permissions by running chmod 0700 against it. This ensured that only the authorised owner of the private key had read and write access to the SSH key. It was also important to run sudo apt-get update && apt-get upgrade once connected to the server to download the latest packages for it. 

To connect to my Ubuntu server I started a terminal session and executed the command: ssh -i privatekeyfile.pem@3.220.165.196 whith sudo privileges. This connected me to my remote server on AWS and I was able to begin performing configurations on it.

### 1.d: User creation and authentication  
This was very straightforward: 
- Ran adduser techwithbashiir  
- Added the user account to the sudoers group using usermod -aG sudo techwithbashiir; 

At this point, my newly-created user had administrative privileges whenever it used sudo and I could use this account to access my remote server that had the SSH Public key installed. 


 ## 2. Apache Webserver
The next step was to install and configure the Apache webserver onto the running Ubuntu Sever. I connected to the Ubuntu server on AWS using SSH. I then assumed root permissions by running sudo su at the shell prompt.

### 2.a: Installation
To install the Apache webserver I ran sudo apt-get install apache2. I accepted the installation terms and the webserver was successfully installed. To confirm this I ran - systemctl status apache2 and it showed that the Apache webserver service was running and active. 

## 3. Nginx
To improve my WordPress site performance I integrated the Nginx server into my application setup. This will improve performance by providing web caching functionality. It will store regular requested content on the cache server and reduce the load on the webserver.       

### 3.a: Nginx installation
To install the Nginx service I ran sudo apt-get install nginx. I accepted the installation terms and the webserver was successfully installed. To confirm this I ran - systemctl status nginx and it showed that the nginx service was running and active. 

### 3.b: Nginx configuration
To configure the ngnix server I went to the /etc/nginx directory and pasted the below configuration into the nginx.conf file: 

    user  www-data;
    worker_processes  auto;

    pid /run/nginx.pid;

    events {
    worker_connections  1024;
    }

    http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
    error_log  /var/log/nginx_error.log error;
    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    # SSL
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # no sslv3 (poodle etc.)
    ssl_prefer_server_ciphers on;

    # Gzip Settings
    gzip on;
    gzip_disable "msie6";
    gzip_vary on;
    gzip_min_length 512;
    gzip_types text/plain text/html application/x-javascript text/javascript application/javascript text/xml text/css application/font-sfnt;

    fastcgi_cache_path /usr/share/nginx/cache/fcgi levels=1:2 keys_zone=microcache:10m max_size=1024m inactive=1h;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
    }
    
    
### 4. MySQL Database
The database server for the WordPress site is a MySQL database server which will provide reliable data and backend services to to the WordPress application. I installed and configured the database server by perfroming MySQL queries. 

## 4.a: MySQL Database Download
To download the MySQL database I ran the sudo apt-get install mysql-server to start the process. Once completed, I ran sudo mysql_secure_installation to configure the database using MySQL's security script: 

- For the VALIDATE PASSWORD PLUGIN, I set a MySQL root password as STRONG length  
- I disabled root account access from a remote client (outside of the local host); 
- I removed anonymous user pre-set accounts; 
- I removed the test database, and privileges that allow access to the database 
- I reloaded the privilege tables

### 4.b: Creating a user and database for the WordPress site

Firstly I created a new MySQL password for my WordPress site by running the script: 

- echo -n @ && cat /dev/urandom | env LC_CTYPE=C tr -dc [:alnum:] | head -c 15 && echo 

I then logged into my database with the provided password using root privileges by running the command: mysql –u root –p. 

I followed this by performing the below MySQL queries on the database: 

- Created the WordPress database with: CREATE DATABASE techwithbashiir; 
- Created the websiteuser user with:  CREATE USER techwithbashiir@localhost IDENTIFIED BY ('password') 
- Granted full privileges on Wordpress to techwithbashiir with: GRANT ALL PRIVILEGES ON techwithbashiir.* TO techwithbashiir@localhost;   
- Updated MySQL privileges with: FLUSH PRIVILEGES;


## 5. PHP
This section covers the PHP installation and configuration for the WordPress site:

### 5.a  Installing PHP and extensions for WordPress
To install the PHP packages on my Ubuntu server I executed the command: sudo apt-get install php7.2 on the shell prompt. I also downloaded all the extension packages by running - sudo apt-get install php-json php-xmlrpc php-curl php-gd php-xml php-mbstring. 

At this point, I had a production server running in my AWS environment with database and webserver services performing effectively. 


## 6. Service management using Systemd

### 6.a: Restarting and Checking system services 

Now that I had all the required services on the Ubuntu server It was important that I now restarted and checked that the services were running correctly. I made sure of this by using the systemd service and running sudo systemctl restart mysql-server, ngninx, apache2, php7.2-fpm. I then checked the status of the services by running sudo systemctl status mysql-server, ngninx, apache2, php7.2-fpm. All the services were in the active and running state which is what I wanted. 


## 7. WordPress Application Setup
This is the section where I installed the WordPress application on the Ubuntu server. I created a system user where I congiured the nginx server and the php-fpm on. 

### 7.a: Creating a systemuser - techwithbashiir
Ensuring correct permissions on the home directory

- adduser techwithbashiir
- mkdir -p /home/techwithbashiir/logs
- chown techwithbashiir:www-data /home/techwithbashiir/logs/

I then applied the necessary permissions levels on the home directory. This is to ensure that the nginx and PHP processes can read all website related files in the users home directory. 

- chown techwithbashiir:www-data /home/techwithbashiir 
- chmod 755 /home/techwithbashiir 


### 7.b: Creating a Nginx configuration file 

I put the below configuration in the Nginx configuration file:  

    server {
    listen       80;
    server_name  www.techwithbashiir.com;

    client_max_body_size 20m;

    index index.php index.html index.htm;
    root   /home/techwithbashiir/public_html;

    location / {
        try_files $uri $uri/ /index.php?q=$uri&$args;
    }

    # pass the PHP scripts to FastCGI server
    location ~ \.php$ {
            # Basic
            try_files $uri =404;
            fastcgi_index index.php;

            # Create a no cache flag
            set $no_cache "";

            # Don't ever cache POSTs
            if ($request_method = POST) {
              set $no_cache 1;
            }

            # Admin stuff should not be cached
            if ($request_uri ~* "/(wp-admin/|wp-login.php)") {
              set $no_cache 1;
            }

            # WooCommerce stuff should not be cached
            if ($request_uri ~* "/store.*|/cart.*|/my-account.*|/checkout.*|/addons.*") {
              set $no_cache 1;
            }

            # If we are the admin, make sure nothing
            # gets cached, so no weird stuff will happen
            if ($http_cookie ~* "wordpress_logged_in_") {
              set $no_cache 1;
            }

            # Cache and cache bypass handling
            fastcgi_no_cache $no_cache;
            fastcgi_cache_bypass $no_cache;
            fastcgi_cache microcache;
            fastcgi_cache_key $scheme$request_method$server_name$request_uri$args;
            fastcgi_cache_valid 200 60m;
            fastcgi_cache_valid 404 10m;
            fastcgi_cache_use_stale updating;


            # General FastCGI handling
            fastcgi_pass unix:/var/run/php/techwithbashiir.sock;
            fastcgi_pass_header Set-Cookie;
            fastcgi_pass_header Cookie;
            fastcgi_ignore_headers Cache-Control Expires Set-Cookie;
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_param SCRIPT_FILENAME $request_filename;
            fastcgi_intercept_errors on;
            include fastcgi_params;         
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|woff|ttf|svg|otf)$ {
            expires 30d;
            add_header Pragma public;
            add_header Cache-Control "public";
            access_log off;
    }

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
    }

    server {
    listen       80;
    server_name  techwithbashiir.com;
    rewrite ^/(.*)$ http://www.techwithbashiir.com/$1 permanent;
    } 


### 7.c: Creating a php-fpm pool configuration file

I put the below configuration in the php-fpm configuration file: 

    [techwithbashiir]
    listen = /var/run/php/techwithbashiir.sock
    listen.owner = techwithbashiir
    listen.group = www-data
    listen.mode = 0660
    user = techwithbashiir
    group = www-data
    pm = dynamic
    pm.max_children = 75
    pm.start_servers = 8
    pm.min_spare_servers = 5
    pm.max_spare_servers = 20
    pm.max_requests = 500

    php_admin_value[upload_max_filesize] = 25M
    php_admin_value[error_log] = /home/techwithbashiir/logs/phpfpm_error.log
    php_admin_value[open_basedir] = /home/techwithbashiir:/tmp


### 7.d: Downloading & Intalling Wordpress  

In order to download Wordpress I executed wget https://wordpress.org/latest.tar.gz. This transferred and downloaded all the WordPress application files onto my Ubuntu server. It was orginally in a compressed file and I extracted it by using the tar zxf utility. 

The wordpress application was now hosted on my Ubuntu server and fully accessible to be used. 


## 8. Using Route 53 (AWS) to create a domain name for the WordPress site

I decided to use AWS DNS registar Route53 to register my WordPress site domain name. This cost me a nominal fee of £10.90 which I was satisfied to pay and it reserved the domain name for me to use for 1 whole year. 

### 8.a: Registering a domain name using Route 53 service

To make my WordPress site more user friendly I registered a domain using AWS Route 53 service. I chose the address - techwithbashiir.com as this was available for me to reserve. I provided some key detail about myself such as Name, Location and Contact information. I made a payment of £10.90 for the domain name for 1 year and it was active in under 1 hour.   

### 8.b: Created a DNS A record on Route 53 Hosted zone   

Once the domain name was successfully registered I created a DNS A record to match the public IPV4 address of the ubuntu server - 3.220.165.196 to the newly created domain name – techwithbashiir.com. To do this I provided the record type, the IP address of the server, TTL (300s) and routing policy – simple routing.  

### 8.c: Tested the DNS server using DIG and NSLOOKUP queries

Once I had the domain name record created I tested its functionality by performing dns queries against it. I ran the dig and nslookup commands against – techwithbashiir.com, and it returned my server information successfully. The domain name was good to be used now.   

<img width="1423" alt="Screenshot 2021-12-14 at 12 10 31" src="https://user-images.githubusercontent.com/89197223/145995828-d47d014b-f7b3-4a6b-9fd1-fd9eaf3b429c.png">





 ### 9 Final WordPress Site
 

<img width="1074" alt="Screenshot 2021-12-15 at 15 24 09" src="https://user-images.githubusercontent.com/89197223/146214345-484bf872-8832-4926-bd44-2d821e5ba8f5.png">



 ### I want to give a special thanks to: 
 
 [Amazon Web Services](https://aws.amazon.com/free/?trk=ps_a131L0000085EJeQAM&trkCampaign=acq_paid_search_brand&sc_channel=ps&sc_campaign=acquisition_UK&sc_publisher=google&sc_category=core-main&sc_country=UK&sc_geo=EMEA&sc_outcome=Acquisition&sc_detail=amazon%20web%20services&sc_content=Brand_amazon_web_services_e&sc_matchtype=e&sc_segment=433803620861&sc_medium=ACQ-P|PS-GO|Brand|Desktop|SU|Core-Main|Core|UK|EN|Text&s_kwcid=AL!4422!3!433803620861!e!!g!!amazon%20web%20services&ef_id=EAIaIQobChMIs5PagYnm9AIVd4BQBh0uBQN4EAAYASAAEgI2kvD_BwE:G:s&s_kwcid=AL!4422!3!433803620861!e!!g!!amazon%20web%20services) - For the amazing cloud resources & documentation. 
 
[David Cohen](https://www.linkedin.com/in/david-cohen-3a704525/) for his awesome Udemy course and other content on Youtube

 

 
