## Project-1: Vprofile Project:Multi Tier Web Application Stack Setup Locally

[*Project Source*](https://medium.com/r/?url=https%3A%2F%2Fwww.udemy.com%2Fcourse%2Fdecodingdevops%2Flearn%2Flecture%2F27976104%23overview)
![Project-Architecture](Images/Architecture%20setup%20of%20Vprofile%20project.png)

## PreRequisites Installed:
  * Oracle VM VirtualBox Manager (Hypervisor)
  * Vagrant
  * Vagrant plugins
  * Gitbash or equivalent editor
  * IDE (VScode, SubmlimeText, etc)

## STEP 1: VIRTUAL MACHINE  Setup

- First clone the repository
```sh
git clone https://github.com/hkhcoder/vprofile-project.git
```

- I navigated to the directory that has my Vagrantfile. Before starting my Virtual Box, I executed the command below to install the Vagrant host manager.
```sh
vagrant plugin install vagrant-hostmanager
```

- After the installation of plugin , I ran the command below to Bring up my VMs.
```sh
vagrant up
```
PS: Bringing up VMs takes time . If VM setup stops in the middle, run vagrant up command again.

- i checked the VMs from `Oracle VM VirtualBox Manager`.
![](images/VMs%20running%20in%20Virtualbox.png)

- Next, I checked the current machine state of the VMs using `vagrant status` command , then proceed to validate my VMs one after the other with command  `vagrant ssh <name_of_VM_given_in_Vagrantfile>`
```sh
vagrant ssh web01
```

![](Images/Vagrant%20plugin%20ensuring%20the%20machines%20connect%20to%20each%20other%20through%20the%20Hostname.png)

- I ran the command `cat/etc/hosts` file. Since  have plugin has been installed, file will be updated automatically.
```sh
cat /etc/hosts
```
Afterwards I ping `web01` to test if the hostname resolves to the IP address or not
![](Images/Pinging%20web01.png)

- Web01 connection was successful
`logout`

- Now we will try to ping `app01` from `web01` vbox.
```sh
ping app01
```

- I Connected to web01 vbox and check connectivity of `app01` with `rmq01`, `db01` and `mc01` for basic network test.

```sh
cat /etc/hosts
ping rmq01
ping mc01
ping app01
ping db01
logout
```

## STEP 2: Provisioning

- Here are 6 different services for the application.
```sh
Services
1. Nginx:
Web Service
2. Tomcat
Application Server
3. RabbitMQ
Broker/Queuing Agent
4. Memcache
DB Caching
5. ElasticSearch
Indexing/Search service
6. MySQL
SQL Database
```
![](Images/Detailed%20Architecture%20of%20VprofileProject.png)

- The services need to be setup in the order listed below:
```sh
1. MySQL     (Database SVC)
2. Memcache  (DB Caching SVC)
3. RabbitMQ  (Broker/Queue SVC)
4. Tomcat    (Application SVC)
5. Nginx     (Web SVC)
```
PS: This is an ideal practice to setup any stack


### Provisioning MySQL 

- Let's start by setting up the  MySQL Database.
```sh
vagrant ssh db01
```

- Switch to root user and update all packages. 
  It's a good practice to update OS with latest patches, when logged into the VM for the first time. 
```sh
sudo -i
yum update -y
```
- Next, I Installed the EPEL(Extra Packages for Enterprise Linux) repository.
```sh
yum install epel-release -y
```
Install Maria DB Package
```sh
yum install git mariadb-server -y
```
After installing Mariadb, start and enable Mariadb service. Also, check the status of Mariadb service to make sure it is `actively(running)`.
```sh
systemctl start mariadb
systemctl enable mariadb
systemctl status mariadb
```
- RUN mysql secure installation script.
Ps: Set db root password, we will be using `admin123` as password

```sh
mysql_secure_installation
```
```sh
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] Y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] Y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] Y
 ... Success!
```
- Check connectivity to db with command blow: Once it asks password, enter admin 123 password. If connection is successful `exit` from DB.
![](Images/connected%20to%20maria%20db.png)

- Run below commands to set DB name and users before initializing .

```sh
mysql -u root -p
mysql> create database accounts;
mysql> grant all privileges on accounts.* TO 'admin'@'%' identified by 'password' ;
mysql> FLUSH PRIVILEGES;
mysql> exit;
```
Ps: For mariadb, commands must end with a semi colon (;) or \g


- Next, clone source code to db VM then change directory to `src/main/resources/` to get the `sql queries.
```
git clone https://github.com/devopshydclub/vprofile-project.git
cd vprofile-project/src/main/resources/
```
![](Images/git%20clone.png)

- Login to DB for a quick verification to see if SQL queries start a database with role ,user and user_role tables.
```sh
mysql -u root -p'DB PASS' accounts < src/main/resources/db_backup.sql
mysql -u root -p'DB PASS' accounts
MariaDB [(accounts)]> Show tables;
MariaDB [(accounts)]> exit;
```

- Restart the `mariadb server` , check the status and `logout`.
```sh
systemctl restart mariadb
systemctl status mariadb
logout
```

### Provisioning Memcache

-  login to memcached server first, and switch to root user.
```sh
vagrant ssh mc01
sudo -i
```

- Similar to MySQL provisioning, start with updating OS with latest patches and download epel repository.
```sh
yum update -y
dnf install epel-release -y
```

- Install `memcached` package.
```sh
dnf install memcached -y
```

- Start/enable the memcached service and check the status of service.
```sh
systemctl start memcached
systemctl enable memcached
systemctl status memcache
```

- Run command below to change the Memcached configuration from `127.0.0.1` to `0.0.0.0` .
```sh
sed -i 's/127.0.0.1/0.0.0.0/g' /etc/sysconfig/memcached
```

- Restart the service with command below and also check the the status (active or not )
```sh
systemctl restart memcached
systemctl status memcached
``` 
![](Images/Memcached%20active%20and%20running.png)
- All good, exit from server with `exit` command.

### Provisioning RabbitMQ

- Login to Rabbit MQ server first, and switch to root user.
```sh
vagrant ssh rmq01
sudo -i
```

- Update OS with latest patches and download epel repository.
```sh
yum update -y
yum install epel-release -y
```

- Install, enable repository and install RabbitMQ server with the commands below:
```sh
 dnf -y install centos-release-rabbitmq-38
 dnf --enablerepo=centos-rabbitmq-38 -y install rabbitmq-server
```

- Lets start/enable the rabbitmq service and check the status of service.
```sh
systemctl start rabbitmq-server
systemctl enable rabbitmq-server
systemctl status rabbitmq-server
```

- Create a test user with password test
- create user_tag for test user as administrator.
- Restart service after complete configuration
```sh
sh -c 'echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config'
rabbitmqctl add_user test test
rabbitmqctl set_user_tags test administrator
systemctl restart rabbitmq-server
systemctl status rabbitmq-server
```

![](Images/RabbitMq%20Active%20and%20running.png)


### Provisioning Tomcat 

- login to `app01` server first, and switch to root user.
```sh
vagrant ssh app01
sudo -i
```

- We will start with updating OS with latest patches and download epel repository.
```sh
yum update -y
yum install epel-release -y
```

- Update OS with latest patches and download epel repository.
```sh
yum update -y
yum install epel-release -y
```

- Install dependencies for Tomcat server.
```sh
 dnf -y install java-11-openjdk java-11-openjdk-devel
 dnf install git maven wget -y
```

- Download and extract binary of tarball for Tomcat using the below commands, but before downloading i switched to `/tmp/ directory`.
```sh
cd /tmp
wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.75/bin/apache-tomcat-9.0.75.tar.gz
ls
tar xzvf apache-tomcat-9.0.75.tar.gz
```

![](Images/Extracted%20Tomcat.png)

- Add tomcat user and copy data to tomcat home directory.
  Check the new user tomcat with `id tomcat` command.
```sh
useradd --home-dir /usr/local/tomcat --shell /sbin/nologin tomcat
```

- Copy the data to `/usr/local/tomcat/` directory which is the home-directory for ``tomcat` user.
```sh
cp -r apache-tomcat-9.0.75/* /usr/local/tomcat/
ls -ls /usr/local/tomcat/
``` 

- Currently root user has ownership of all files under `/usr/local/tomcat/ directory`. change it to `tomcat` user

```sh
ls -ld /usr/local/tomcat/
chown -R tomcat.tomcat /usr/local/tomcat/
ls -ls /usr/local/tomcat/
```
![](Images/Tomcat%20User%20directory.png)


- Setup systemd for tomcat, create a file with below content. After creating this file, start tomcat service with `systemctl start tomcat` and stop tomcat with `systemctl stop tomcat` commands.
- Any changes made to file under `/etc/systemd/system/` directory, we need to run below command to be effective:

```sh
[Unit]
Description=Tomcat
After=network.target
[Service]
User=tomcat
WorkingDirectory=/usr/local/tomcat
Environment=JRE_HOME=/usr/lib/jvm/jre
Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment=CATALINA_HOME=/usr/local/tomcat
Environment=CATALINE_BASE=/usr/local/tomcat
ExecStart=/usr/local/tomcat/bin/catalina.sh run
ExecStop=/usr/local/tomcat/bin/shutdown.sh
SyslogIdentifier=tomcat-%i
[Install]
WantedBy=multi-user.target
```

- Enable tomcat service. The service name tomcat has to be same as given `/etc/systemd/system/tomcat.service` directory.
```sh
systemctl daemon-reload
```
```sh
systemctl enable tomcat
systemctl start tomcat
systemctl status tomcat
```

- Our Tomcat server is active running, now we will build our source code and deploy it to Tomcat server.

#### Code Build & Deploy to Tomcat Server

- The source code will be clone in /tmp directory,
```sh
git clone -b main https://github.com/hkhcoder/vprofile-project.git
ls
cd vprofile-project/
```

- Before building the artifact, we need to update our configuration file that will be connect to our backend services db, memcached and rabbitmq service.
```sh
vim src/main/resources/application.properties
```

- In application.properties file: Ensure the settings are correct, check DB configuration. The db server is` db01` , and we have` admin` user with password `admin123` as we setup. For memcached service, hostname is `mc01`.For rabbitMQ, hostname is `rmq01` and we have created user `test` with pwd `test`.

```sh
#JDBC Configutation for Database Connection
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://db01:3306/accounts?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull
jdbc.username=admin
jdbc.password=admin123

#Memcached Configuration For Active and StandBy Host
#For Active Host
memcached.active.host=mc01
memcached.active.port=11211
#For StandBy Host
memcached.standBy.host=127.0.0.2
memcached.standBy.port=11211

#RabbitMq Configuration
rabbitmq.address=rmq01
rabbitmq.port=5672
rabbitmq.username=test
rabbitmq.password=test
```

- Run mvn install command to create our artifact. Artifact will be created in `/tmp/vprofile-project/target/vprofile-v2.war`
```sh
cd target/
ls
```

- Deploy artifact `vprofile-v2.war` to Tomcat server. But before that, remove default app from our server. For that reason first we will shutdown server. The default app will be in `/usr/local/tomcat/webapps/ROOT` directory.
```sh
systemctl stop tomcat
ls /usr/local/tomcat/webapps/
rm -rf /usr/local/tomcat/webapps/ROOT
```

- The artifact is under vprofile-project/target directory. Now copy the artifact to `/usr/local/tomcat/webapps/` directory as `ROOT.war` and start tomcat server. Once we start the server, it will extract the artifact `ROOT.war` under `ROOT` directory.. 
```sh
ls /usr/local/tomcat/webapps/
cp target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
ls /usr/local/tomcat/webapps/
systemctl start tomcat
systemctl status tomcat
```

- While the application is coming up we can provision our Nginx server.

### Provisioning Nginx 

- Nginx server is Ubuntu, despite our servers are RedHat. To update OS with latest patches run below command:
```sh
apt update && sudo apt upgrade
```

- Install nginx.
```sh
sudo -i
apt install nginx -y
```

- Create a Nginx configuration file under directory `/etc/nginx/sites-available/` with below content:
```sh
vi /etc/nginx/sites-available/vproapp
Content to add:
upstream vproapp {
server app01:8080;
}
server {
listen 80;
location / {
proxy_pass http://vproapp;
}
}
```

- Remove default nginx config file:
```sh
rm -rf /etc/nginx/sites-enabled/default
```

- Create a symbolic link for our configuration file using default config location below to enable our site. Then restart nginx server.
```sh
ln -s /etc/nginx/sites-available/vproapp /etc/nginx/sites-enabled/vproapp
ls -i /etc/nginx/sites-enabled/
systemctl restart nginx
systemctl status nginx
```

![](Images/Nginx%20running.png)


### STEP 3: Validate Application from Browser

- In web01 server, run `ip addr show` to get its IP address. So the IP address of our web01 is : `192.168.56.11`
```sh
192.168.56.11
```
- Validate Nginx is running on browser `http://<IP_of_Nginx_server>`.
![](Images/App%20accessible%20through%20nginx.png)

- Validate Db connection using credentials `admin_vp` for both username and password.


- Validate app is running from Tomcat server

![](Images/DB%20connection%20succesful.png)

- Validate RabbitMQ connection by clicking RabbitMQ
  
![](Images/RabbitMq%20connection%20succesful.png)

- Validate Memcache connection by clicking MemCache

![](Images/Memcache%20connection%20succesful..png)

- Validate data is coming from Database when user first time requests it.

![](Images/Data%20gotten%20from%20Memcache.png)

- Validate data is coming from Memcached when user second time requests it.

![](Images/Data%20gotten%20from%20DB%20when%20user%20request%20second%20time.png)

### STEP 4: Destroy VMs
- Run the command below to destroy all virtual machines.
```sh
vagrant destroy --force
```
![](Images/Vagrant%20Destroy.png)

- Double check Oracle VM virtual box to be sure VMs are destroyed.

![](Images/VMs%20are%20gone%20after%20destroying%20vagrants.png)



























