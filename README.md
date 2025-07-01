# Host a Website on EC2 with RDS (MySQL) and HTTPS via ACM


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

### **STEP 5: Connect EC2 to RDS**

```
  sudo yum install mysql -y
```

```
  mysql -h <RDS-ENDPOINT> -u admin -p
```

````
  CREATE DATABASE mydb;
  USE mydb;
  CREATE TABLE users ( id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(100));
````

### **STEP 6: Create Load Balancer (for HTTPS with ACM)**

1. Go to EC2 > Load Balancers > Create Load Balancer

2. Choose: Application Load Balancer

3. Name: my-website-lb

4. Scheme: Internet-facing

5. Listener: Port 80 (HTTP)

6. Add Listener: Port 443 (HTTPS)

7. Select VPC + Subnets (same as EC2)

8. Create a new Target Group:
 
   * Name: mytg

   * Target type: instance

   * Protocol: HTTP
  
10. Register your EC2 instance to the target group

11. Create

### **STEP 7: Request SSL Certificate (ACM)**

1. Go to ACM > Request certificate

2. Select: Public Certificate

3. Domain name: yourdomain.com

4. Validation: DNS (add CNAME to your domain provider)

5. Wait until status = Issued

### **STEP 8: Attach ACM to Load Balancer**

1. Go to your Load Balancer

2. Listener (443) → Edit → Add certificate:

    * Select the ACM certificate

3. Save and apply changes

### **STEP 9: Point Domain to Load Balancer**

If you own a domain (e.g. Hostinger, GoDaddy):

1. Go to your domain provider

2. Add A Record or CNAME:

   
 | Name | Type  | Value                                   |
| ---- | ----- | --------------------------------------- |
| `@`  | CNAME | `my-website-lb-xxxxx.elb.amazonaws.com(Your-load-balancers-DNS)` |

### **STEP 10: Test Your Website**

````
  https://yourdomain.com
````
