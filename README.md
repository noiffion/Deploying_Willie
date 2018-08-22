## Deploying_Willie

This README lists a series of steps needed to deploy a web application on Amazon Lightsail and 
some other essential information about the website.


#### Info about the webpage:

  The URI of the site:           http://www.54.188.224.166.xip.io/
  The IP address of the site:    54.188.224.166
  The SSH port:                  2200
  
The private SSH key was included in the 'Notes to the Reviewer' section on submission.
The public key was in the '.ssh' folder of user 'grader'.


#### Steps of deployment

1) The application is hosted on [Amazon Lightsail] (https://lightsail.aws.amazon.com/)
2) The operating system instance used was Ubuntu 16.04
3) The hostname of the instance is: 'Ubuntu-Oregon'

4) After initializing the instance and connecting to it via SSH using the pop-up Lightsail
   terminal (and the default public-private SSH keypair provided by Amazon) the software packages
   of the instance were updated and upgraded using:
```
$ sudo apt update
$ sudo apt upgrade
```


5) The following packages were installed using:
```
$ sudo apt install <package_name>
```
   - python 3
   - sqlite 3
   - ntp
   - postgresql
   - finger
   - git
   - apache2-dev

(libapache2-mod-wsgi-py3 was NOT installed via 'apt')

6) In the development machine an SSH keypair was generated using:
```
$ ssh-keygen
``` 
  and the public key was uploaded to the SSH keys section of the Account menu of Lightsail 
  (https://lightsail.aws.amazon.com/ls/webapp/account/keys). 
  In the server the 'sshd_config' file had to be modified:

```
$ sudo vim /etc/ssh/sshd_config 
```
  With these modifications the instance became accessible via a different SSH keypair that was 
  originally provided.


7) The firewall of the instance (iptables) was calibrated using the 'ufw' program:
```
$ sudo ufw default deny incoming
$ sudo ufw allow ssh  
$ sudo ufw allow 2222/tcp  
$ sudo ufw allow www
$ sudo ufw allow 123/udp
$ sudo ufw deny 22
$ sudo ufw enable  
$ sudo ufw status 
```
  With the commands above the firewall was set up. Logging out of the SSH session now the firewall
  of Lightsail could be set up (these two are different) to make it possible to reach the server 
  using SSH, HTTP, and NTP  protocols via the 2222, 80, and 123 ports respectively 
  ('Networking' tab of the instance --> 'Firewall' --> 'Edit rules' 
   https://lightsail.aws.amazon.com/ls/webapp/us-west-2/instances/Ubuntu-Oregon/networking).
  In the user's home directory .vimrc and .bashrc files were also modified a bit for convenience.
  
  After that the server was accessible only via the the terminal SSH of the development machine
  and the browser based pop-up SSH terminal became disabled.


8) A new user 'grader' was created with the password: grader and also permission for that user to
   be able to 'sudo' was set up in '90-cloud-init-users' (with the same parameters as 'ubuntu' user).
```
$ sudo adduser grader
$ sudo vim /etc/sudoers.d/90-cloud-init-users
``` 


9) A new SSH keypair was generated on the development machine and the public key was put into the
   'grader' user's 'authorized_keys' file in the 'home/.ssh' directory. 
   The private key (grader_ub_1604) was retained on the development machine. 
   After logging out of the SSH session the server became accessible for the new user with:
```
$ ssh grader@54.188.224.166 -p 2200 -i grader_ub_1604 
```


10) A new user in Postgresql was created (although postgres was not used by the application as the
    database server.)


11) With pip3 'virtualenv' was installed and a new 'virt_hamnet' folder was created in:
    '/var/www/html/'. The virtual environment was then activated. 

```
$ pip3 install virtualenv
$ virtualenv /var/www/html/virt_hamnet
$ source virt_hamnet/bin/activate
``` 

12) After that the following python libraries were installed in the virtual environment:

```
$ pip install
              Flask
              google-auth
              google-auth-oauthlib
              google-oauth
              SQLAlchemy
              pycodestyle
              urllib3
              mod-wsgi
```

13) The application 'Shake-speare' from [https://github.com/noiffion/hamnet.git] (https://github.com/noiffion/hamnet.git)
    was cloned in this directory. The hamnet.wsgi file here and the wsgi.conf, wsgi.load 
    files in /etc/apache2/mods-available and the hamnet.conf file in /etc/apache2/sites-available
    were set up.

14) After checking the settings of the server and the wsgi and also the error log with:
```
$ apache2ctl configtest
$ apache2ctl -M|grep -i wsgi
$ sudo tail -f /var/log/apache2/error.log
```

  and then apache was ready to be restarted and run:
```
$ sudo service apache2 restart
```

(The site currently has only limited functionality due to the fact that only 'http' protocol based
 connections are allowed (following the requirements of the project) on port 80 and 'https' is not. 
 Therefore logging in to the site is not possible for users: for that an SSL certificate would be 
 needed an less restrictive firewall settings.)

#### Information sources:
http://terokarvinen.com/2017/write-python-3-web-apps-with-apache2-mod_wsgi-install-ubuntu-16-04-xenial-every-tiny-part-tested-separately
https://askubuntu.com/questions/569550/assertionerror-using-apache2-and-libapache2-mod-wsgi-py3-on-ubuntu-14-04-python
https://hk.saowen.com/a/0a0048ca7141440d0553425e8df46b16cdf4c13f50df4c5888256393d34bb1b9
http://terokarvinen.com/2016/deploy-flask-python3-on-apache2-ubuntu
