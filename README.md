# Prerequisites
#
- JDK 1.8 or later
- Maven 3 or later
- MySQL 5.6 or later
- Vagrant


# PROJECT 1

# WE WILL SETUP WEB APPLICATION STACK.
We will setup multi-tier web application stack on our local machine. This will help you to be able to setup stack locally for your projects

# ARCHITECTURE

- NGINX
- TOMCAT
- RABBITMQ
- MEMCACHED
- MYSQL server

There is manual and automanual setup

# Let's do manual first


* We will setup a website/web app which is a social networking site written in Java.

*Stack is collection of services or technology to create an experience

* We will use Nginx as a web server and as a load balancer

*Apache Tomcat is a java web application service

* RabbitMQ is dummy in this project, not functional. it is a message broker or queueing agent


* So when users try to login to access our application which in apache tomcat, our app will run sql query to access user info stored in mysql server. But it will first go to memcached service if the user has login in before and it cached there.

- TOOLS TO INSTALL

Install chocolatey in your local machine 
Then Install all these via chocolatey


choco install virtualbox --version=6.1.40
choco install vagrant
choco install git
choco install jdk8
choco install maven
choco install awscli
choco install intellijidea-community
choco install sublimetext3.app

Note: You must not use chocolatey, you can install them directyl from browser.

Next STEP

From gitbash run

git clone https://github.com/Helen-Chukwukelu/Multi-tier-Web-Application-Stack.git

cd Multi-tier-Web-Application-Stack/
switch to local setup branch if that is not your current branch

git checkout local-setup

- Install vagrant host manager
vagrant plugin install vagrant hostmanager
vagrant up

Open your virtualbox, if all are successfully set up and running. go back to terminal and run

vagrant ssh web01
cat /etc/hosts

ping app01

logout and try to ssh into app01
vagrant ssh app01
cat /etc/hosts

ping mc01

this is to ping back end services that the app will try to connect to


- Next step

Open the setup document in this path

Multi-tier-Web-Application-Stack/vagrant/Manual_provisioning/VprofileProjectSetup.pdf


-Setup sql service

sudo -i && yum update -y

Now setup a variable for database

DATABASE_PASS='admin123'
echo $DATABASE_PASS

Note this is temporary, you can mke it permanent
RUN
vi /etc/profile
and then add the file just beneath the last code/line 

DATABASE_PASS='admin123' 

save and exit


- Install and setup a repository which is epel-release

yum install epel-release -y

this will set the repo access and then run

# Install Maria DB Package
yum install git mariadb-server -y

# Starting & enabling mariadb-server
systemctl start mariadb
systemctl enable mariadb
systemctl status mariadb

# RUN mysql secure installation script.
mysql_secure_installation

press enter
until it ask to enter new password, use admin 123

- The below is to help you with the responses 

Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] n
Remove test database and access to it? [Y/n] y
Reload privilege tables now? [Y/n] y


NOTE: Set db root password, I will be using admin123 as pasPsword

IT IS SETUP!
Now test password and see if you can login

mysql -u root -p
exit

Now clone the source code


- Download Source code & Initialize Database.
 git clone -b local-setup https://github.com/devopshydclub/vprofile-project.git](https://github.com/Helen-Chukwukelu/Multi-tier-Web-Application-Stack.git

reason of cloning is because our database file is there
$cd Multi-tier-Web-Application-Stack.git
$cd src/
$cd main/
$ls
$cd resources/
$ls
$pwd

we will initialiswe our database with that
mysql -u root -p"$DATABASE_PASS" -e "create database accounts"
mysql -u root -p"$DATABASE_PASS" -e "grant all privileges on accounts.* TO 'admin'@'app01' identified by 'admin123' "

Now run db backup sql file

$cd ../../..pwd
$pwd

Are you in vprofile-project? good!
$ls

Then run the commands below

$mysql -u root -p"$DATABASE_PASS" accounts < src/main/resources/db_backup.sql
$mysql -u root -p"$DATABASE_PASS" -e "FLUSH PRIVILEGES"
$mysql -u root -p"$DATABASE_PASS" 

- Test

$show databases;
$use accounts;
$show tables;

$exit
$logout

If you want to setup firewall, use the below

Restart mariadb-server
$systemctl restart mariadb

Starting the firewall and allowing the mariadb to access from port no. 3306
$systemctl start firewalld
$systemctl enable firewalld
$firewall-cmd --get-active-zones
$firewall-cmd --zone=public --add-port=3306/tcp --permanent
$firewall-cmd --reload
$systemctl restart mariadb



# MEMCATCH SETUP

$vagrant ssh mc01
$sudo -i
$yum update -y
$yum install epel-release -y
$yum install memcached -y
$systemctl start memcached
$systemctl enable memcached
$systemctl status memcached
$memcached -p 11211 -U 11111 -u memcached -d

- lets validate if it is running in the right port, run
$sudo netstat -nap | grep memcached

GOOD!
$exit


# LETS SETUP RABBIT MQ

$vagrant ssh rmq01

- Login to the RabbitMQ vm
$vagrant ssh rmq01
$sudo -i
$yum update -y

- Verify Hosts entry, if entries missing update the it with IP and hostnames
$cat /etc/hosts

Install Dependencies
$sudo yum install wget -y
$cd /tmp/
$wget http://packages.erlang-solutions.com/erlang-solutions-2.0-1.noarch.rpm
$sudo rpm -Uvh erlang-solutions-2.0-1.noarch.rpm
$sudo yum -y install erlang socat

# Install Rabbitmq Server

$curl -s https://packagecloud.io/install/repositories/rabbitmq/rabbitmq-server/script.rpm.sh | sudo bash
$sudo yum install rabbitmq-server -y

- Start & Enable RabbitMQ

$sudo systemctl start rabbitmq-server
$sudo systemctl enable rabbitmq-server
$sudo systemctl status rabbitmq-server

- Config Change

$sudo sh -c 'echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config'
$sudo rabbitmqctl add_user test test

Note: name is user and password is test

Give user administrative privilege
$sudo rabbitmqctl set_user_tags test administrator

- Restart and see if everything is working well

$systemctl restart rabbitmq-server
$systemctl status rabbitmq-server

-**RUNNING! AND BACKEND IS SETUP!**

Dababase, Memcatch and rabbitmq will now login to the main app01 where we will setup our tomcat service

run $exit and $exit again


# TOMCAT SETUP

Login to the tomcat vm
$ vagrant ssh app01

Verify Hosts entry, if entries missing update the it with IP and hostnames
$cat /etc/hosts

Update OS with latest patches
$yum update -y

Set Repository

$yum install epel-release -y

Install Dependencies
$yum install java-1.8.0-openjdk -y
$yum install git maven wget -y

Change dir to /tmp
$cd /tmp/

Download & Tomcat Package
$wget https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.37/bin/apache-tomcat-8.5.37.tar.gz
$tar xzvf apache-tomcat-8.5.37.tar.gz

- Add tomcat user which will create tomcat home direction for us
$useradd --home-dir /usr/local/tomcat8 --shell /sbin/nologin tomcat
id tomcat

Copy data to tomcat home dir
$cp -r /tmp/apache-tomcat-8.5.37/* /usr/local/tomcat8/

Make tomcat user owner of tomcat home dir
$chown -R tomcat.tomcat /usr/local/tomcat8

Setup systemd for tomcat

vi /etc/systemd/system/tomcat.service


Update file with following content.

[Unit
Description=Tomcat
After=network.target
[Service]
User=tomcat
WorkingDirectory=/usr/local/tomcat8
Environment=JRE_HOME=/usr/lib/jvm/jre
Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment=CATALINA_HOME=/usr/local/tomcat8
Environment=CATALINE_BASE=/usr/local/tomcat8
ExecStart=/usr/local/tomcat8/bin/catalina.sh run
ExecStop=/usr/local/tomcat8/bin/shutdown.sh
SyslogIdentifier=tomcat-%i
[Install]
WantedBy=multi-user.target

Save it and run the commands

$systemctl daemon-reload
$systemctl start tomcat
$systemctl enable tomcat
$systemctl status tomcat

**RUNNING!

# NOW lets build artifact from our source code and deploy to tomcat server


CODE BUILD & DEPLOY (app01)

Download Source code
$git clone -b local-setup https://github.com/Helen-Chukwukelu/Multi-tier-Web-Application-Stack.git

Update configuration
$cd Multi-tier-Web-Application-Stack.git
$git status

Ensure you are on local-setup branch

Before we build, we have to update the configuration file that willl be used by our app to connect backedend services, ie database, memcached, rabbitmq
$vim src/main/resources/application.properties

Update file with backend server details

in my case i didnt update anything because my details are correct

- Now Build code
Run below command inside the repository (Multi-tier-Web-Application-Stack.git)

$mvn install
$cd target/

- Deploy artifact
$systemctl stop tomcat
$sleep 120
$rm -rf /usr/local/tomcat8/webapps/ROOT*
cd ..
$cp target/vprofile-v2.war /usr/local/tomcat8/webapps/ROOT.war
$systemctl start tomcat
$exit
$exit


Login to the Nginx vm
$vagrant ssh web01

Verify Hosts entry, if entries missing update the it with IP and hostnames
$cat /etc/hosts

Update OS with latest patches

$apt update
$apt upgrade

Install nginx
$apt install nginx -y

Create Nginx conf file with below content which will be used to redirect request from NGINX SERVER TO TOMCAT SERVER ON PORT 8080. Nginx will serve as a loadbalancer

first run
# vi /etc/nginx/sites-available/vproapp

Then update with this below


upstream vproapp {
server app01:8080;
}
server {
listen 80;
location / {
proxy_pass http://vproapp;
}
}

save!

Remove default nginx conf
$rm -rf /etc/nginx/sites-enabled/default

Create link to activate website
$ln -s /etc/nginx/sites-available/vproapp /etc/nginx/sites-enabled/vproapp

Restart Nginx
$systemctl restart nginx

# GOOD! NOW LETS VALIDATE WHAT WE HAVE DONE

Run the command

$ifconfig

Copy the inet addr which is ip address and access it from browser

It wil show a login page

sign in with 
username: admin_vp
passsword: admin_vp

The sign is successfull meaning database is connected

You also validate other backend services


**CONGRATULATIONS!



