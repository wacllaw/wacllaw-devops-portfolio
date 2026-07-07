# LAMP Stack Implementation on AWS EC2

*A step-by-step deployment guide and project report*

---

## Overview

In this project, I deployed a complete LAMP stack — Linux, Apache, MySQL, and PHP — on an AWS EC2 instance running Ubuntu Server 24.04 LTS. My goal was to provision a cloud server from scratch, install and configure each layer of the stack, and verify that the environment could serve a dynamic PHP application. Along the way, I documented the exact commands I used, the configuration files I created, and the issues I ran into, along with how I resolved them.

### What Is a LAMP Stack?

The **LAMP stack** is a widely used open-source web development platform made up of four core components:

| Layer | Component | Role |
|---|---|---|
| **L** | Linux | The operating system |
| **A** | Apache | The web server |
| **M** | MySQL | The relational database management system |
| **P** | PHP | The server-side scripting language |

Together, these components provide everything needed to build and host dynamic websites and web applications.

### My Environment

| Item | Details |
|---|---|
| Local machine | Windows 11 with WSL (Windows Subsystem for Linux) + Ubuntu terminal |
| Cloud provider | AWS EC2 (Ubuntu Server 24.04 LTS) |
| Instance type | t3.micro (Free Tier) |

I used WSL because I don't have a native Linux laptop, so I ran all commands from the Ubuntu terminal within WSL.

![AWS EC2 Dashboard](screenshots/01-aws-console.png)
*Figure 1: AWS EC2 console dashboard*

---

## Step 1: Launch the EC2 Instance

1. Log in to the AWS Management Console and navigate to **EC2**.
2. Click **Launch instance**.
3. Configure the instance as follows:
   - **Name:** `LAMP`
   - **Software Image (AMI):** Ubuntu Server 24.04 LTS

     ![Ubuntu Server AMI selection](screenshots/03-aws-ubuntu-image.png)
     *Figure 2: Selecting the Ubuntu Server 24.04 LTS AMI*

   - **Instance type:** `t3.micro`
   - **Key pair:** Create a new key pair named `LAMP.pem`
   - **Security group:** Open port 22 (SSH) and port 80 (HTTP)

     ![Security group configuration](screenshots/02-aws-secgroup-lamp.png)
     *Figure 3: Security group settings for the LAMP instance*

4. Launch the instance.

   ![Running EC2 instance](screenshots/04-aws-running-instance.png)
   *Figure 4: Confirmation that the instance is running*

---

## Step 2: Connect to EC2 from WSL

1. Download the `.pem` key file and move it to your WSL Ubuntu home directory.
2. Copy the public IP address of your EC2 instance and connect via SSH:

   ```bash
   ssh -i LAMP.pem ubuntu@54.175.226.119
   ```

   You may encounter a permissions error at this stage:

   ![SSH permission error](screenshots/05-aws-ssh-error-message.png)
   *Figure 5: Permissions error when connecting via SSH*

3. Set the correct permissions on the key file and reconnect:

   ```bash
   chmod 400 LAMP.pem
   ssh -i LAMP.pem ubuntu@54.175.226.119
   ```

   ![Successful SSH connection](screenshots/06-aws-chmod-400.png)
   *Figure 6: Connecting successfully after fixing key permissions*

---

## Step 3: Update System Packages

Update your system's package index and upgrade installed packages:

```bash
sudo apt update -y && sudo apt upgrade -y
```

![System package update](screenshots/07-aws-sudo-apt-update.png)
*Figure 7: Updating and upgrading system packages*

---

## Step 4: Install Apache

Install Apache and start the service:

```bash
sudo apt install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2
sudo systemctl status apache2
```

![Installing Apache](screenshots/08-aws-sudo-install-apache.png)
*Figure 8: Installing the Apache package*

![Apache service status](screenshots/09-aws-sudo-update-apache.png)
*Figure 9: Verifying the Apache service is running*

Visit `http://<your-ip-address>` in a browser to confirm the installation. You should see the default Apache landing page.

![Default Apache landing page](screenshots/10-aws-apache-default.png)
*Figure 10: Default Apache page confirming a successful installation*

---

## Step 5: Install MySQL

1. Install the MySQL server:

   ```bash
   sudo apt install mysql-server
   ```

   ![Installing MySQL](screenshots/11-aws-mysql-install.png)
   *Figure 11: Installing the MySQL server package*

2. Start and enable the MySQL service:

   ```bash
   sudo systemctl start mysql
   sudo systemctl enable mysql
   ```

3. Check that the service is running:

   ```bash
   sudo systemctl status mysql
   ```

   ![MySQL service status](screenshots/12-aws-mysql-status.png)
   *Figure 12: Confirming that MySQL is active and running*

4. Run the secure installation script to harden your MySQL setup:

   ```bash
   sudo mysql_secure_installation
   ```

   During this step, set a strong password validation policy, set a strong root password, and remove anonymous users.

   ![MySQL secure installation](screenshots/13-aws-mysql-set-password.png)
   *Figure 13: Running the MySQL secure installation script*

5. Test that you can log in:

   ```bash
   mysql -u root -p
   ```

---

## Step 6: Install PHP

1. Install PHP along with the modules needed for Apache and MySQL integration:

   ```bash
   sudo apt install php libapache2-mod-php php-mysql
   php -v
   ```

   ![Installing PHP](screenshots/14-aws-sudo-install-php.png)
   *Figure 14: Installing PHP and required modules*

   ![PHP version check](screenshots/15-aws-php-version.png)
   *Figure 15: Verifying the installed PHP version*

2. Create a directory for your project:

   ```bash
   sudo mkdir /var/www/html/projectlamp
   cd /var/www/html/projectlamp
   ```

3. Create a test PHP file to confirm PHP is working:

   ```bash
   sudo nano info.php
   ```

   Add the following content:

   ```php
   <?php
   phpinfo();
   ?>
   ```

   ![Creating info.php](screenshots/18-aws-sudo-nano-info-93.png)
   *Figure 16: Creating the test PHP file*

4. Create a virtual host configuration file for your project:

   ```bash
   sudo nano /etc/apache2/sites-available/projectlamp.conf
   ```

   Add the following configuration:

   ```apache
   <VirtualHost *:80>
       ServerName projectlamp
       ServerAlias www.projectlamp
       ServerAdmin webmaster@localhost
       DocumentRoot /var/www/html/projectlamp
       ErrorLog ${APACHE_LOG_DIR}/error.log
       CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   ```

5. Enable the new site and reload Apache:

   ```bash
   sudo a2ensite projectlamp.conf
   sudo systemctl reload apache2
   ```

   ![Enabling the virtual host](screenshots/20-aws-enable-site.png)
   *Figure 17: Enabling the new virtual host*

---

## Step 7: Test the LAMP Stack

Visit `http://<your-ip-address>/info.php` in your browser. You should see the PHP information page, confirming that Apache, MySQL, and PHP are all working together correctly.

![PHP info page](screenshots/21-aws-info-php.png)
*Figure 18: The PHP info page confirming the stack is fully functional*

---

## Challenges and Solutions

| Issue | Solution |
|---|---|
| SSH permission denied | Resolved by running `chmod 400 LAMP.pem` before connecting |
| MySQL authentication errors | Resolved by running `ALTER USER ... WITH mysql_native_password` |
| "Not Found" error when visiting the site | Resolved by disabling the default site (`000-default.conf`) and enabling the custom virtual host |
| `systemctl` commands required authentication | Resolved by prefixing commands with `sudo` |

---

## Conclusion

By completing this project, I successfully deployed a fully functional LAMP stack on AWS EC2 using Ubuntu 24.04 LTS. I provisioned the cloud instance, configured secure SSH access, and installed and integrated Apache, MySQL, and PHP into a working environment. Along the way, I ran into real configuration issues — including SSH permission errors, MySQL authentication problems, and Apache routing conflicts — and worked through each one methodically. This project strengthened my understanding of Linux server administration, virtual host configuration, and the troubleshooting process behind standing up a production-style web stack.

### Skills Demonstrated

1. AWS EC2 instance provisioning
2. Linux server administration
3. LAMP stack configuration
4. Virtual host setup
5. Troubleshooting and technical documentation
