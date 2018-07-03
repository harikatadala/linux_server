# Linux Server Configuration Project
### By Tadala Durga Harika

### About
  This is the Udacity project which is configuring linux server.

### Server Details
Server IP Address 13.232.41.220

Hosted site Url [http://13.232.41.220.xip.io/](http://13.232.41.220.xip.io/)

### Configuring Linux Server

#### Updating all packages
```
sudo apt-get update
sudo apt-get upgrade
```

#### Creating grader User:
  ```
  sudo adduser grader
  ```
  Password for grader user
  harika31
  
  This will add new user
  ```
  sudo nano /etc/sudoers
  ```
  Below the Root user append the following line
  ```
  grader  ALL=(ALL:ALL) ALL
  ```
  This will grant sudo permission to grader
  #### Creating a ssh key pair for grader
   On your local machine in terminal/command prompt
   ```
   ssh-keygen
   ```
   This will generate public and private ssh keys which is saved to .ssh folder
   
   Then in your virtual machine change to newly created user
   ```
   su - grader
   ```
   Create a new directory .ssh and new file authorized_keys in that directory
   ```
   mkdir .ssh
   sudo nano .ssh/authorized_keys
   ```
   Copy the public key with .pub extension to authorized_keys and save the file
   ```
   chmod 700 .ssh
   chmod 644 .ssh/authorized_keys
   ```
   - 700 will give read write and execute permission to user.
   - 644 prevent other user from writting in to file.
   Then restart ssh server
   ```
   service ssh restart
   ```
   
   Now from your log in to grader with private key generated 
   ```
   ssh -i .ssh/id_rsa grader@13.232.41.220
   ```
  #### Changing the ssh port to 2200
   ```
   sudo nano /etc/ssh/sshd_config
   ```
   Change port 22 to port 2200
    
   Restart the ssh server
   
   ```
   service ssh restart
   ```
   
   >Note: Before Logging using ssh add custom TCP port 2200 under lightsaail firewall in networking tab in lightsail instance console  
   
   Now Login using command like this
   ```
   ssh -i .ssh/id_rsa -p 2200 grader@13.232.41.220
   ```
   
  #### Disabling ssh login as root
  `sudo nano /etc/ssh/sshd_config`
  
  make change `PermitRootLogin no`
  
  #### Configurating  Ufw firewall

  Goto amazon aws lightsail account 

  Goto networking then change firewall as follows
  ```
  Custom        UDP        123
  
  Custom        TCP        2200
  ```
  
  #### Installing Apache2 
  In terminal 
  
  ```sudo apt-get install apache2```
  
  Now mod_wsgi
  
  ```sudo apt-get install python-setuptools libapache2-mod-wsgi```
  
  Enable mod_wsgi
  
  ```sudo a2enmod wsgi ```
  ##### Setting up your flask application to work with apache2
   Creating a flask app
   
   In /var/www directory create a new folder
   `sudo mkdir FlaskApp`
   
   Install git 
   
   `sudo apt-get install git`
   
   move to the FlaskApp `cd FlaskApp`
   
   In that direcory clone your github repository
   
   `sudo git clone 'https://github.com/harikatadala/catalog1231.git'`
   
   Rename the repository to FlaskApp
   
   Then rename your project file to `__init__.py`
   
   Make Following changes in __init__.py
   ```
   PROJECT_ROOT = os.path.realpath(os.path.dirname(__file__))
   json_url = os.path.join(PROJECT_ROOT, 'client_secrets.json')
   CLIENT_ID = json.load(open(json_url))['web']['client_id']
   ```
   Use json_url instead client_secrets.json in script
   
   
  ##### Install and configuring postgresql for project
   Install Postgres `sudo apt-get install postgresql`
   
   login to postgres `sudo su - postgres`
   
   postgres shell `psql`
   
   create user `CREATE USER catalog WITH PASSWORD 'password';`
   
   permit user to createdb `ALTER USER catalog CREATEDB;`
   
   Create a db name  catalog with user catalog `CREATE DATABASE catalog WITH OWNER catalog;`
   
   connect to db `\c catalog`
   
   revoke all permission to public `REVOKE ALL ON SCHEMA public FROM public;`
   
   Give schema permission to user catalog `GRANT ALL ON SCHEMA public TO catalog;`
   
   exit from db and postgres `\q and exit`
   
   Change the database connection in both db_setup.py and `__init__.py` as `engine =       create_engine('postgresql://catalog:password@localhost/catalog')`
   
   Now you are ready with your applicatiom
  #### Configure and Enable a New Virtual Host
   `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
   
   In this add the following code
   ```
   <VirtualHost *:80>
		ServerName 13.232.41.220
		ServerAdmin admin@mywebsite.com
		WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
		<Directory /var/www/FlaskApp/FlaskApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/FlaskApp/FlaskApp/static
		<Directory /var/www/FlaskApp/FlaskApp/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
   ```
   Enable the virtual host 
   `sudo a2ensite FlaskApp`
   
   Disabling the default apache2 page
   `sudo a2dissite 000-default.conf`
   
  #### Create the .wsgi File
    ```
    cd /var/www/FlaskApp
    sudo nano flaskapp.wsgi 
    ```
   Add the following code
   
   ```
   #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/FlaskApp/")

    from FlaskApp import app as application
    application.secret_key = 'w-edVSchXN099u_DtXhaqoyG'
   ```
   save and exit
   
   Deploying flask app with apache2 is referred from [Digital ocean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
   
   #### Installing require modules
   You can either install all modules on your machine or create a virtual environment for the project and install the modules
   ` pip install flask sqlalchemy psycopg2 requests oauth2client`
   
   #### Setting up your Google Oauth2
   
   - Go to [Google Cloud Platform](https://console.cloud.google.com/).
  
   - Click `APIs & services` on left menu.
   
   - Click `Credentials`.
   
   - Create an OAuth Client ID (under the Credentials tab), and add http://13.232.41.220.xip.io 
     as authorized JavaScript origins.
   
   - Add the below three links as authorized javascript origins
     
     1. http://13.232.41.220.xip.io/login

     2. http://13.232.41.220.xip.io/gconnect

     3. http://13.232.41.220.xip.io/callback 

   - Download the corresponding JSON file, open it and  copy the contents.

   - Open `/var/www/FlaskApp/FlaskApp/client_secret.json` and paste the previous contents into the this file.

   - Replace the client ID to line 25 of the `templates/login.html` file in the project directory.
   
   #### Final Step
   
   Restart your apache2 server
  
   `sudo service apache2 restart`

    ## Login to the grader
   
   Open puttygen and load the `test.pem` to genarate private key and click on `save privatekey`
   
   It is saved as privatekey.ppk 
   
   Now open putty by selecting auth in ssh and load `privatekey` with static IP: 13.232.41.220 port:2200
   
   login as : ubuntu
   
   `su - grader`
   
   password for grader:harika31
   
  #### Making .ssh folder
  
  ` mkdir .ssh`
   
   creates .ssh directory
   
   `touch .ssh/authorized_keys`
   
   create authorized_keys in .ssh folder
   
   `nano .ssh/authorized_keys`
   
   In git bash 

   `ssh-keygen`
   
   .ssh file will be created with id_rsa files
   
   now copy the contents of id_rsa.pub file (microsoft office file) and paste it in the grader. 
   
   ## Test.pem file
   
   -----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAwBQAKY1XXBdlba92c4vvIzeC6mj7qtqQRjCFyRm7SU7McTe+
bLw1rCO9abxm+mPD3gB6dEGru4RmuZpvEjj/JhsXk9nq+PCtrRzvuKyPHF5AqUK3
Ju0WLovN3F8fRbRMwbg80aF6YLwDBJu1Xml6MztiaKazhsN8UWsJgwM05L7IvqBV
8tC6qfCOPjgq/ya6VovuQc+Y9xp0jOUws/XWNK/6f585UfXRzZOxdsiZyOIagkqp
3wsNhsuboerthzlsIWiFyOZ9f3DgztdtVt10cNfhC6bxAYFhJr25xokZhOnvnjoy
rmiFDtsr/YgPa7ltzJCtDhQYszlB8IIa0ZSoGQIDAQABAoIBACUKNlXM+dG6eUbD
lVYG4CCsbcSCZjW2XCgM51+2ZJVoaqhSlZgmWztM0RP4zuruHjFLalHM8C8DA0Q7
cbvT3fAdPVi3p0ZGM1e0ws2cCSTxUArT4LnS8nobX6FlfoRUlpCs1J2gRBcvam2T
EVCZioUiqIGB1BDi1bBlsbnf/m01v4Xm66tyUXmRh97u27jSBrG6YyIe+IKEC9Fk
Lg8D/gzwIHTt+YUhjsdSzbr1GONw5q0jmKWmD9PdVCO8ZXv8N03TyNcd0EzY5sSN
s4BycPBUmfoDP2rjy4YOHo8bYf4ZsSbjbSiD9lqFlYQ2tEaJPt0rhROgBas4z2d1
34zMVeECgYEA7kBO3YPkaNsozENJbP088K9ZVKf3wx2MGI8ri63+sh8RO2pE3Z9U
9zubcqjK7PVZJfHDIFUXjZEZM8kSy/+r/95k37xOAVaKNT+KZXNywoGuKUf/YLfT
WdKUwJ+3hJwXobW1WkY16fu+JR9K7gYYHDGUwjFhmjveOQwB7L1aioUCgYEAzmMg
Dbcd+DIjPzIYnn688y8D1LLgvEGU4AiVJ158FQkOvGNfK4vV98MUYV3qeaXXQ5qm
8GAgrqHFhw+Q7OVOE0t/9/z8KHzRsT4al2Gn11o0+WHpu4MaMHPEaH8OfZWnvZcT
nH1nVfM7KuNWw95Tyh9U1oBjEBWWsoORf701PYUCgYBAHBfcrZfxyz9gL577b+1N
CrIsAILAAxxmo2fhTzGg9pEpfsAHLs+rM2Px54+rUZ3qgvKxqZQL6QZyE+I1+Jds
44gbWE1ZONM53t47zGQOCN03iIMkoHKD0hFq/89fJK2LOx0QrKHnU3FoBdKg2Az2
0TSpSKZt3Tw/94YxEQbjDQKBgQCzC82n+F94jU6EqZowDfUv526kXJaY2zAjd26m
G9L7kMMG7hKHPaXfbo7EtWwQIq5wSL9gs5RGy7MIK5nn2jp0hMA8zG2ZVke4Qw9g
mui369sfKjFSajcTJ6uRmABjNKyzzlfGIAjAyOVgnJ8OB1ebdrjr6a+HKaN1tKxK
LEP+3QKBgFPc+9JUVqYX4kB/NoGtuwPsLv97zmWNqtbzNbt8DRpMmrBtumwMWvle
DKB3h3kC6FcatRmoEJ1QMQezpK+ZOOjemRNTbmffzBJlSVZvcaWUhU1lOS3a69C/
SZPAW6OWnoEif0eqrAK8n/GJ5l4QdoW62cMfYuwJP9Mw+tZcpiew
-----END RSA PRIVATE KEY-----

      ## Privatekey.ppk file
   
   PuTTY-User-Key-File-2: ssh-rsa
Encryption: none
Comment: imported-openssh-key
Public-Lines: 6
AAAAB3NzaC1yc2EAAAADAQABAAABAQDAFAApjVdcF2Vtr3Zzi+8jN4LqaPuq2pBG
MIXJGbtJTsxxN75svDWsI71pvGb6Y8PeAHp0Qau7hGa5mm8SOP8mGxeT2er48K2t
HO+4rI8cXkCpQrcm7RYui83cXx9FtEzBuDzRoXpgvAMEm7VeaXozO2JoprOGw3xR
awmDAzTkvsi+oFXy0Lqp8I4+OCr/JrpWi+5Bz5j3GnSM5TCz9dY0r/p/nzlR9dHN
k7F2yJnI4hqCSqnfCw2Gy5uh6u2HOWwhaIXI5n1/cODO121W3XRw1+ELpvEBgWEm
vbnGiRmE6e+eOjKuaIUO2yv9iA9ruW3MkK0OFBizOUHwghrRlKgZ
Private-Lines: 14
AAABACUKNlXM+dG6eUbDlVYG4CCsbcSCZjW2XCgM51+2ZJVoaqhSlZgmWztM0RP4
zuruHjFLalHM8C8DA0Q7cbvT3fAdPVi3p0ZGM1e0ws2cCSTxUArT4LnS8nobX6Fl
foRUlpCs1J2gRBcvam2TEVCZioUiqIGB1BDi1bBlsbnf/m01v4Xm66tyUXmRh97u
27jSBrG6YyIe+IKEC9FkLg8D/gzwIHTt+YUhjsdSzbr1GONw5q0jmKWmD9PdVCO8
ZXv8N03TyNcd0EzY5sSNs4BycPBUmfoDP2rjy4YOHo8bYf4ZsSbjbSiD9lqFlYQ2
tEaJPt0rhROgBas4z2d134zMVeEAAACBAO5ATt2D5GjbKMxDSWz9PPCvWVSn98Md
jBiPK4ut/rIfETtqRN2fVPc7m3Koyuz1WSXxwyBVF42RGTPJEsv/q//eZN+8TgFW
ijU/imVzcsKBrilH/2C301nSlMCft4ScF6G1tVpGNen7viUfSu4GGBwxlMIxYZo7
3jkMAey9WoqFAAAAgQDOYyANtx34MiM/MhiefrzzLwPUsuC8QZTgCJUnXnwVCQ68
Y18ri9X3wxRhXep5pddDmqbwYCCuocWHD5Ds5U4TS3/3/PwofNGxPhqXYafXWjT5
Yem7gxowc8Rofw59lae9lxOcfWdV8zsq41bD3lPKH1TWgGMQFZayg5F/vTU9hQAA
AIBT3PvSVFamF+JAfzaBrbsD7C7/e85ljarW8zW7fA0aTJqwbbpsDFr5Xgygd4d5
AuhXGrUZqBCdUDEHs6SvmTjo3pkTU25n38wSZUlWb3GllIVNZTkt2uvQv0mTwFuj
lp6BIn9HqqwCvJ/xieZeEHaFutnDH2LsCT/TMPrWXKYnsA==
Private-MAC: 0d1c91e8b7ace4358510a40814ac15085f72bb52

   
   ## id_rsa file
   
   -----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAuDJIskKEx3E4m53381UvHUK2VOxOSvkQT++HjWpvNjSiN0qB
XeFOU4oHR5/iBYvTvXDWeAyFriGzDFoLfmiDEzKBcfrotB9WH4wlozP/W5k6Oq5Y
nXrKqS2pLwpuGT7CJG7Cv4cW+N6Rmc7AKhKptuvCWByqoBWtqX1rBI9z40WY4Hnc
by4+m+EF9kfVSjqDWkafz+N1EHrLzP2WRLsWbxtZS9y8Hvg9UNxIY/x5dC85P91/
zDd9AysFsjK9T7NZhwr4zxkwn1QjtHWBKPEnOP8D2P57XipBfSAO9mIChsUwo+RC
yNplxTeOR8T0bb57tze2idtU6m000oS26xOYIQIDAQABAoIBAAyHjx5hG/sdHoan
ZjhtMPlCkniQYnsDGjL1zji2Zr4/MlGkhcxHCTbxirE6lPtfmZWnWU9tOqKvtChA
78s3fMMPinKgaLfn8cnPlLfXT6WBdM6NEYYU1jhP3NRQV1DsTfSAfEre+5cOPMVD
pRq3jkfdHak0uf6NuvRujZsPzwhEk/9DucHUEa6BrBKR3Lj/SQnSKSTM6aHN9lJw
VDE4SkPw7vqSu3GIAmunUlnW+zp8QBV367x/xPGy7iTgSZzdXLLtyOQLJhROBGdl
KyLOpO3F9Y5XJ7Z5pnXi7kglYxQXBWj4ipGT6oDYwn+4dRc0jmeElPDyYuif7fvM
5mX7qgECgYEA8RlCDueswcXRSTOTrc+NHRv7o5KyE5uFsycT7apiBojZVvPJphWf
CO8dniDc0+t3+qGEYLsGQ64P/8PGtqMPkSHQzHJDEA9IGgdCeoaRx02TqgfmbLIZ
SbVECu8ceSa5pt7w/wfPgwxrW/YicRCDz9Eu9Jb5j6+M+IbHVsV4KtECgYEAw5Sz
NSY881P5TvNJ9XhhQIJUdmJJPp3AmkpXEvrLQJUa6E0uMI06Mq2s3PWGB/2pqN7k
tZxrI+LWWiCWJdjqcL+WEDv8tatCx8lcr5RfnpqDn2YDYZ1m78SQUfMTvINpvl73
bPpwheEm9yTJ3CmRLPxCJ/Zds5mUK4sWBzPYTFECgYB9zQQe6tvFsAZn/7WK98VV
7wf6bczlVj0JIdbQKBtF9ZoSONp7AsUc0EfP/q27Fh5gdG6yWmQGZIiMmAbTvXmd
7NsaEnTTqGWizy7s59Kg/+WQztZ42OVBBIggfGuK+53baHCvYc/xPUW3klKeVFOy
2B98GFrXkm4oL4tLx3XQQQKBgA2N+rWORt8eTML5/YAzrZnI7o2d1jGuwTBwQoSg
OQHq04iSAMeAoi3s93QA2B8pp156xCMKCirch7qLU5yf4e+ad438elEk42XxQKky
gUX+s2ySp11WiVHIr6/eRtnjyapV7pPw72d36TlrpUnK37LccMFQQ8I0vJ4f75Dk
OPNhAoGAMyfzaUYZC/fBnoeRt/3mzimyNHdhajG9LAzGWj6l6VLsKZe8YtfsZZdy
POcCAwqtQeJkLOMpBbTMKDV9wv60RBau6aUwWeWBA0KGahQ4CFawDt8eB9k6PjMQ
cLXK4i7P0a1jT3JYo82iT4GXYAJOsCk3NVsTMNXhr0LAZRLmKuw=
-----END RSA PRIVATE KEY-----

   
   ## id_rsa.pub file
   
  ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC4MkiyQoTHcTibnffzVS8dQrZU7E5K+RBP74eNam82NKI3SoFd4U5TigdHn+IFi9O9cNZ4DIWuIbMMWgt+aIMTMoFx+ui0H1YfjCWjM/9bmTo6rlidesqpLakvCm4ZPsIkbsK/hxb43pGZzsAqEqm268JYHKqgFa2pfWsEj3PjRZjgedxvLj6b4QX2R9VKOoNaRp/P43UQesvM/ZZEuxZvG1lL3Lwe+D1Q3Ehj/Hl0Lzk/3X/MN30DKwWyMr1Ps1mHCvjPGTCfVCO0dYEo8Sc4/wPY/nteKkF9IA72YgKGxTCj5ELI2mXFN45HxPRtvnu3N7aJ21TqbTTShLbrE5gh Tarun Tadala@TarunTadala

 ## References

   1. https://github.com/rrjoson/udacity-linux-server-configuration

   2. https://github.com/boisalai/udacity-linux-server-configuration

   3. https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

   4. stackoverflow website 
