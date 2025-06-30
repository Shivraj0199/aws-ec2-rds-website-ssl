## Host a Website on EC2 with RDS (MySQL) and HTTPS via ACM


### **🎯 Objective:**
**Create a secure web application where:**

* A user visits a public website
* Fills a form (e.g., login or register)
* Data gets stored in an Amazon RDS (MySQL) database
* Website is served over HTTPS


### **STEP 1: VPC & Subnets (Auto or Manual)**

For a basic project, you can use the default VPC (skip custom VPC setup)

### **STEP 2: Create RDS MySQL Database**

* Go to RDS > Create Database

* Select:

  * Engine: MySQL

  * Template: Free Tier

* Settings:

  * DB instance identifier: mydb

  * Master username/password

* Connectivity:

  * Public Access: NO

  * VPC Security Group: create new (e.g., rds-sg)

  * Add inbound rule: port 3306 from EC2 SG (later)

* Create DB and save RDS Endpoint

### **STEP 3: Launch EC2 (Web Server)**

* Go to EC2 > Launch Instance

* AMI: Amazon Linux 2 or Ubuntu

* Instance type: t2.micro

* Security Group (e.g., web-sg):

  * Inbound: 22, 80
 
### **STEP 4: Create Website with Form (on EC2)**

* SSH into EC2:

* Install LAMP Stack

```
      sudo yum update -y
      sudo yum install httpd -y
      sudo systemctl start httpd
      sudo systemctl enable httpd
      sudo dnf install php php-mysqlnd php-cli php-gd php-xml php-mbstring -y
      sudo systemctl restart httpd
```

* Create a simple login/register form: ( /var/www/html/index.html )
    
* index.html
 
```
  <form action="save.php" method="POST">
  <input type="text" name="username" placeholder="Username" required><br>
  <input type="password" name="password" placeholder="Password" required><br>
  <button type="submit">Register</button>
  </form>
```

* save.html

```
   <?php
   $conn = new mysqli("<RDS-End-Point>", "<master-user-name>", "<db-password>", "<db-name>");
   if ($conn->connect_error) {
   die("Connection failed");
   }
   $username = $_POST['username'];
   $password = $_POST['password'];
   $conn->query("INSERT INTO users (username, password) VALUES ('$username', '$password')");
   echo "Registered successfully!";
   ?>
```

