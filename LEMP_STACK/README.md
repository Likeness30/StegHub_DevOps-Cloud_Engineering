## WEB STACK IMPLEMENTATION (LEMP STACK) IN AWS
---

## What is a LEMP Stack?

Introduction:

__The LEMP stack is a popular open-source web development platform that consists of four main components: Linux, Nginx, MySQL, and PHP (or sometimes Perl or Python). This documentation outlines the setup, configuration, and usage of the LEMP stack.__

- **Linux**: The operating system that runs the server. In this guide, we'll use **Ubuntu** (a popular Linux distribution) to manage software installations and configurations.
- **Nginx**: A high-performance web server. Nginx is used to serve static content (like HTML, CSS, and images) and pass dynamic requests (PHP) to the application server.
- **MySQL**: The database management system (DBMS). MySQL stores and retrieves all the data used by your dynamic applications.
- **PHP**: A server-side scripting language used to generate dynamic web content. PHP works with MySQL to retrieve data from the database and output it on the web page.

---

## Why Use a LEMP Stack?
The LEMP stack is popular because it offers:
- **Performance**: Nginx is known for handling high traffic with low memory usage.
- **Flexibility**: PHP is widely used for dynamic websites, and MySQL offers a reliable and scalable database solution.
- **Compatibility**: These components work seamlessly together and are open-source, meaning they are free and well-supported by the community.

---

## Prerequisites

- AWS account
- Basic knowledge of EC2 and SSH
- SSH key pair for connecting to your EC2 instance
- Optional: A registered domain (for SSL configuration)

---

## Step 1: Launch an EC2 Instance

1. **Log in to AWS**: Go to the [AWS Management Console](https://aws.amazon.com).
2. **Navigate to EC2**: From the services menu, select EC2.
3. **Launch a new instance**:
   - **Amazon Machine Image (AMI)**: Select `Ubuntu 22.04 LTS`.
   - **Instance type**: Choose `t2.micro` (free-tier eligible).
   - **Key Pair**: Select or create a key pair for SSH access.
   - **Security Group**: Open the following ports:
     - SSH (22)
     - HTTP (80)
     - HTTPS (443)

4. **Launch** the instance and note the public IP address.

---

## Step 2: Connect to the EC2 Instance

Once the instance is running, connect to it via SSH.

```
ssh -i /path/to/your-key.pem ubuntu@<your-ec2-public-ip>
```
update your server
![alt text](images/apt_update.PNG "apt update")

___ The private ssh key that got downloaded was located, permission was changed for the private key file and then used to connect to the instance by running
```
chmod 400 my-ec2-key.pem
```
```
ssh -i "my-ec2-key.pem" ubuntu@13.60.59.105
```
Where __username=ubuntu__ and __public ip address=13.60.59.105__
## Step 1 - Install nginx web server

__1.__ __Update and upgrade the server’s package index__

```
sudo apt update
sudo apt upgrade -y
```
![alt text](images/apt_update.PNG "apt update")


__2.__ __Verify that nginx is active and running__

```
sudo systemctl status nginx
```
![alt text](images/nginx.1.PNG "Install nginx")
![alt text](images/nginx.2.PNG "nginx status")

__3.__ __Access nginx locally on the Ubuntu shell__

```
curl http://13.60.59.105
```
![alt text](images/nginx.3.PNG "nginx landing page")


__4.__ __Test with the public IP address if the Nginx server can respond to request from the internet using the url on a browser.__

__5.__ __Now, you can visit your instance's public IP to verify that Nginx is working:
http://13.60.59.105
You should see the default Nginx welcome page.

## Step 2 - Install MySQL
__1.__ __Install a relational database (RDB)__

MySQL was installed in this project. It is a popular relational database management system used within PHP environments.
```
sudo apt install mysql-server
```
![alt text](images/mysql.1.PNG "Install mysql-server")

Configure the user and key password
![alt text](images/mysql_set.PNG "mysql configuration")
![alt text](images/mysql_t.PNG "mysql login")

## Step 4 - Configure nginx to use PHP processor

__1.__ __Create a root web directory for your_domain__

```
sudo mkdir /var/www/projectLEMP
```
__2.__ __Assign the directory ownership with $USER which will reference the current system user__

```
sudo chown -R $USER:$USER /var/www/projectLEMP
```

__3.__ __Create a new configuration file in Nginx’s “sites-available” directory__.

```
sudo nano /etc/nginx/sites-available/projectLEMP
```
Paste in the following bare-bones configuration:

```
server {
  listen 80;
  server_name projectLEMP www.projectLEMP;
  root /var/www/projectLEMP;

  index index.html index.htm index.php;

  location / {
    try_files $uri $uri/ =404;
  }

  location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
  }

  location ~ /\.ht {
    deny all;
  }
}
```

### Here’s what each directives and location blocks does:

- __listen__ - Defines what port nginx listens on. In this case it will listen on port 80, the default port for HTTP.

- __root__ - Defines the document root where the files served by this website are stored.

- __index__ - Defines in which order Nginx will prioritize the index files for this website. It is a common practice to list index.html files with a higher precedence than index.php files to allow for quickly setting up a maintenance landing page for PHP applications. You can adjust these settings to better suit your application needs.

- __server_name__ - Defines which domain name and/or IP addresses the server block should respond for. Point this directive to your domain name or public IP address.

- __location /__ - The first location block includes the try_files directive, which checks for the existence of files or directories matching a URI request. If Nginx cannot find the appropriate result, it will return a 404 error.

- __location ~ \.php$__ - This location handles the actual PHP processing by pointing Nginx to the fastcgi-php.conf configuration file and the php7.4-fpm.sock file, which declares what socket is associated with php-fpm.

- __location ~ /\.ht__ - The last location block deals with .htaccess files, which Nginx does not process. By adding the deny all directive, if any .htaccess files happen to find their way into the document root, they will not be served to visitors.

__4.__ __Activate the configuration by linking to the config file from Nginx’s sites-enabled directory__

```
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
```
__5.__ __Test the configuration for syntax error__

```
sudo nginx -t
```

__6.__ __Disable the default Nginx host that currently configured to listen on port 80__

```
sudo unlink /etc/nginx/sites-enabled/default
```
__7.__ __Reload Nginx to apply the changes__
```
sudo systemctl reload nginx
```
__8.__ __The new website is now active but the web root /var/www/projectLEMP is still empty. Create an index.html file in this location so to test the virtual host work as expected.__

```
sudo echo ‘Hello LEMP from hostname’ $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) ‘with public IP’ $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
```

#### Open it with public dns name (port is optional)
```
http://<public-DNS-name>:80
```
![alt text](images/dns_test.PNG "test with public dns")
![alt text](images/test.ip.PNG "test with public ip")



This file can be left in place as a temporary landing page for the application until an index.php file is set up to replace it. Once this is done, remove or rename the index.html file from the document root as it will take precedence over index.php file by default.

The LEMP stack is now fully configured.
At this point, the LEMP stack is completely installed and fully operational.


## Step 5 - Test PHP with Nginx

Test the LEMP stack to validate that Nginx can handle the .php files off to the PHP processor.

__1.__ __Create a test PHP file in the document root. Open a new file called info.php within the document root.__

```
sudo nano /var/www/projectLEMP/info.php
```

```
<?php
phpinfo();
```
![alt text](images/phpt.PNG "test php with nginx")

 It’s best to remove the file created as it contains sensitive information about the PHP environment and the ubuntu server. after checking it,It can always be recreated if the information is needed later.
```
sudo rm /var/www/projectLEMP/info.php
```
## Step 6 - Retrieve Data from MySQL database with PHP

### Create a new user with the mysql_native_password authentication method in order to be able to connect to MySQL database from PHP.

Create a database named todo_database and a user named todo_user

__1.__ __First, connect to the MySQL console using the root account.__
```
sudo mysql -p
```
```
CREATE DATABASE todo_database;
```
![alt text](images/db.PNG "Create database todo")

__3.__ __Create a new user and grant the user full privileges on the new database.__
```
CREATE USER 'todo_user'@'%' IDENTIFIED WITH mysql_native_password BY 'Admin123$';

GRANT ALL ON todo_database.* TO 'todo_user'@'%';
```
```
exit
```
![alt text](images/table.PNG "Create user")

From MySQL console, run the following:
```
CREATE TABLE todo_database.todo_list (
  item_id INT AUTO_INCREMENT,
  content VARCHAR(255),
  PRIMARY KEY(item_id)
);
```
__6.__ __Insert a few rows of content to the test table__.
```
INSERT INTO todo_database.todo_list (content) VALUES ("My first important item");

INSERT INTO todo_database.todo_list (content) VALUES ("My second important item");

INSERT INTO todo_database.todo_list (content) VALUES ("My third important item");

INSERT INTO todo_database.todo_list (content) VALUES ("and this one more thing");
```
![alt text](images/table3.PNG "Insertions into table")
![alt text](images/table4.PNG "Insertions")



__7.__ __To confirm that the data was successfully saved to the table run:__
```
SELECT * FROM todo_database.todo_list;
exit
```
the select command acalso be used for several commands, removing, adding, verfying etc.
![alt text](images/Tableinfo.PNG "Database infomation")


### Create a PHP script that will connect to MySQL and query the content.

__Create a new PHP file in the custom web root directory__
```
sudo nano /var/www/projectLEMP/todo_list.php
```
The PHP script connects to MySQL database and queries for the content of the todo_list table, displays the results in a list. If there’s a problem with the database connection, it will throw an exception.

Copy the content below into the todo_list.php script.
```
<?php
$user = "todo_user";
$password = "Admin123$";
$database = "todo_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
?>
```

__When the PHP script queried the database there was an error "502 Bad Gateway" displayed on the browser. This is because the PHP version specified in the Nginx server configuration is php8.1 while the version of the PHP installed is php8.3__
you can go back into you php config file and check for any errors you must have made and use the righ syntax.
__This was troubleshooted by updating the Nginx server configuration with PHP version php8.3__
then update the nginx and php servers
```
sudo systemctl restart nginx
http://your_instance_public_ip/todo_list.php
```
__You should see the PHP information page, confirming that your LEMP stack is working.

Ater updating the Nginx server, the URL was tested again on the browser and there no error.
```
http://your_instance_public_ip/todo_list.php
```
![alt text](images/todo.PNG "Todo info")