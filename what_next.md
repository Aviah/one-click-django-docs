
# What's Next

The following steps are suggested after you successfully installed a server with one-click-django-server, a development environment with one-click-django-dev, and had several successful deployment cycles.

[Staging Environment]()    
[Jquery, bootstrap & other external libraries]()    
[Add developers]()    
[Improve the firewall]()    
[Save Secrets Outside the Repository]()    
[MySQL Dumps]()    
[Backup]()    
[Separate the Single Server to a Webserver & DB Server]()    
[Https]()    
[CDN]()    


### Staging environment

You can use one-click-django-server to install a staging environment. Then repository chain would be dev >> staging >> production. If you push to production only from staging, then only tested code is deployed to production. However, staging can be used for testing and experiments with non production code, so you may want another staging machine, or still deploy to stage and then to production from the dev machine.    

A few tips:

+ Adjust the git workflow.
+ Add robots.txt that blocks search engines from indexing the stage site
+ Password protect the stage site, or limit access to your ip
+ Selenium is a great tool to test the stage website before pushing the code to production
+ Add another settings file, like settings_stage.py, similar to settings_dev.py and settings_production.py 


### Jquery, bootstrap & other external libraries

Most common libraries have a free CDN, which is faster, free, and probably available in the browser cache already. To use these libraries in your site, add the links to the base template, where they will be available to every template that extends the base template. Here in an example for jQuery and Bootstrap:

    <head>
    
        <script type="text/javascript" src="https://code.jquery.com/jquery-2.2.0.min.js"></script>
        <script type="text/javascript" src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/js/bootstrap.min.js"></script>
        <link type="text/css" rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css">
   
        ...        
    </head>
 
Many javascript libraries, including bootstrap, require jQuery, so it's a good idea to put jQuery first.
    
*Note: You can serve a copy of most external libraries with your site static files. But there are many advantages for using the public CDN: it's faster, you don't pay for the bandwidth, and for  most common libraries like bootstrap and jQuery, the latest version is probably already cached in the user's browser, so your site will load faster*

### Add developers


The project is saved in the django user home directory on the server, so any additional developer can access it with the django account.   
When you let users access the django account, they will be able to access the site directories, work with the git repository, change everything on the site, reload the site  (with touch wsgi.py) and access the logs. They will not have a sudo permissions.

To add developers, add their public key to the django user authorized_keys file:

    you@dev-machine$ scp john_smith_id_rsa.pub django@PUB.IP.IP.IP:~/
    you@dev-machine$ ssh django@PUB.IP.IP.IP
    django@my-django-server$ cat john_smith_id_rsa.pub >> .ssh/authorized_keys
    
    
    
Now John Smith can ssh to the server and work on the site:
 
    john@dev-machine$ ssh django@PUB.IP.IP.IP
 	
 	
If you add a **new** user with ssh access, you need to add this user to the sshgroup: 

    django@my-django-server$ sudo usermod -a -G sshgroup newuser
    
*sshgroup is a group, created with the setup script. Replace newuser with the actual new username*
    

### Securiy

Security in this project is really basic, and there are lot of good resources, books, websites, checklists, videos and tools, about how to improve the website security. This is an ongoing effort.

A few ideas where to start:

* The firewall is really basic, add some advanced and more specific rules, both for iptables and ip6tables
* Strong passwords
* Move the django code secrets (settings SECRET_KEY, database passwords etc) to the secrets.py outside the repository
* Change the django admin url
* Update the OS regulary
* Use additional tools



### Save Secrets Outside the Repository

It's a good practice to save secrets (e.g. django secret key or the database password) outside the repository. Especially if you are using the public repos in github.    
To move secrets from the repository, save them in **mysite/site_config/secrets.py**. This file is not in the repository, but imported to settings.py. Make sure that both development environment and production have an updated secrets file because it is outside the repository, and will not be pushed deployment.


*Note: Even when you move a secret outside the repository, the info is still saved in previous versions. It's not trivial to remove it completely (although possible). The best, and simplest way is to create new secrets when moving from the repository to another file. If this is a database password, you will have to change the password in MySQL and FLUSH PRIVILEGES*



### MySQL Dumps
mysqldump can be used to save and load database into text files. This feature is useful as a backups, or to save database fixtures. 

To save the database into a file:
    
    you@my-django-server$ mysqldump -u django -p --databases django_db --add-drop-database > django_db.bak.sql
    
*Use the actual django-mysql password (on your settings file)*

To load a backup:

    you@my-django-server$ mysql -u root -p  < django_db.bak.sql
    
*Use the actual mysql root password (provided when you installed MySQL)*


Once the db is dumped to a file, it's easy to tar,zip, and scp, to backup to another machine, download production db to work on it loacly etc.


### Backup 

VPS providers usually offer a backup with additional costs. Typically, the backup is scheduled daily,hourly or another time period.  
To make sure that the database is in a known state when this VPS backup runs, you should dump the db just before the backup starts. Run a mysqldump with a crontab task a few minutes before the scheduled backup.

*Note: manage.py provides it's own dumpdata and loaddata commands for the database provided in the settings. For backup from development , see: fab backup in [Deployment](deployment.md)*

### Advanced Deployment and Managment Commands with Fabric

Add your own commands with fabric. Fabric commands are python functions, available in your fabfile, at your home directory. By default, the setup script named thei file fabfile_mysite.py (with your site name instead of mysite). The settings to call that file are in ~/.fabricrc. Fabric has more ways to find a fabfile in the current directory an options to call other files as well.
What nice about fabric that it can run on multiple servers, and you can really add what you want: run or restart services, load fixtures, move code etc, and streamline the whole development-staging-production cycle accross multiple servers. All in standard, plain & simple Python.
See the Fabric docs.


### Separate the Single Server to a Webserver & DB Server
Idealy, you should separate the database from the webserver. But don't do it before it's really necessary! If you have good traffic, you can simply upgrade the same single VPS with more CPU and more RAM. Doubling the VPS capacity will cost you less than an hour of work. You can Wait with this deployment until you absolutly, positivly must. 

When you do absolutly positivly need to separate servers, the easiest way would be to clone the single server.

1. Shutdown the webserver
2. Clone it to another VPS, which will serve as the database server
3. 1. In the original VPS, one remove MySQL server (using apt-get). This will be the webserver. You will still need the MySQL client.
4. In the cloned VPS, remove Nginx and Apache (using apt-get). This will be the database server.
5. Edit hostname,network/interfaces and hosts files on the cloned server with the new ip and hostname
6. Edit /etc/hosts file on both servers, so each one has an entry  of the other
7. Edit the firewall on both servers. The webserver (the original VPS), should be able to access the DB server via port 3306. The database server (the cloned VPS) does not need to listen on port 80 anymore
8.  On the database server (the cloned VPS) set the correct MySQL permissions, to allow a django user to access it from the webserver:  
        
        you@cloned-vps-db-server: mysql -uroot -p
        mysql> CREATE USER 'django'@'webserver' IDENTIFIED BY 'djangomysqlpassword';  
        mysql> GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,ALTER,INDEX,CREATE TEMPORARY TABLES,LOCK TABLES, EXECUTE ON django_db.* TO 'django'@'webserver';  
        mysql> FLUSH PRIVILEGES;
         
    
9. In settings_production.py change the DATABASE settings for django. Since now the server does not use localhost, like the development machine, you will need to move the DATABASE config to settings_dev.py and setting_production.py, each one with another host and possibly passowrd



### Https

Https is handled by Nginx.  When receiving https request as a proxy, Nginx decrypts it and sends the unencrypted request to Apache/mod_wsgi. So moving to https does not require many changes on the django side: Your django code continues to get the same unencrypted traffic. However, it's still important to get into the details in order to implement it properly.

The recommended practice is to move everything to https, and avoid mixed http/https websites. 

Here is a general overview of the steps required to move the site to https:

1. Buy the SSL certificate. The cheapest will do, unless you really need the fancy certificates with the company name in the browsers bar  

2. Create a certificate signing request (CSR) with a matching key, using openssl, from the command line. You can save the files in /etc/ssl. Openssl asks you a few questions to create the CSR. The "Common Name" question is where you provide the domain. See your specific certificate issuer instructions what exactly you should type here, depends on the certificate type and what you need (subdomains, for both www.exapmle.com and example.com etc). 

3. Send the CSR to the certificate issuer (usually with a web form). If you purchased the simplest domain validation certificate, you should recieve the certificate by mail in minutes. You recieve two files: your own certificate and the issuer intermediate certificates.

4. Save the certificate files on the server, and create one file with both (cat your_certificate >> intermidiate_certificate).

5.  Move the current Nginx server to https: edit /etc/nginx/sites-enabled/django. Change the server to listen on 443, and add to it the Nginx ssl configs,  ssl_on, ssl_certificate for the certificate+intermedate file, ssl_certificate_key to the key file etc. see Nginx docs.

6. In /etc/nginx/sites-enabled/django, add another Nginx server that listens to the same domain on port 80, and rewrites everything to https.

7. Restart Nginx. If everything works, curl www.yourdomain.com should get a 301 redirect.

8. Now you will need a few fixes to the code:
	* Secure cookies, see SESSION_COOKIE_SECURE, CSRF_COOKIE_SECURE
	* Make sure that when a full url is used the protocol is always https (e.g. email links, redirects etc)
	* Set SECURE_PROXY_SSL_HEADER to https. Django is behind the Nginx proxy, so it sees http. It needs to know that the proxy received https.
	* If you use external libraries, CDNs or resources, make sure everything is https
	* Using // as the url base: //path/to/page instead of /path/to/page. This tells the browsers to follow whatever protocol called the url. You can use https in production and http in development, with the same code, as long as the url base is //
	* Save the specific django https settings in settings_production.py. 
	* On the dev machine local site you can still work with http without any certificate.
	* Add the django security middleware to the site settings MIDDLEWARE, and configure it's options. Note that many of the middleware options are available also directly with Nginx, which will provide better preformance.
	* Check the site with manage.py check --deploy
	* Read carefully the django docs about https

9. Test your site in Qualysis free online SSL checking tool.
10. To make sure the cookies are secure, you can see them in Chrome developer tools.

A few things to consider:

+ Configure HSTS (in Nginx), which tells browsers to always request https from your site. This is a more secure option, provided you will not want to use http.
+ Tell Google that your new https site is actually the previous http site. Google has ton of staff on this issue, and it's important to do it right, and keep the Google rank when moving to https (use canonical links, webmaster tools etc, see Google resources).  

*Note: A simple domain validation SSL certificate is cheap. But if you use a CDN, adding your own certificate to recieve https CDN may be quiet expensive*

### CDN
   
The simplest CDN just pulls the files from one url, and distribute them to another. In a typical deployment, the browser requests cdn.example.com/static/main.js. The CDN goes to www.example.com/static/main.js, pulls the file to it's cache, and then sends it to the browser. The next time someone asks for cdn.example.com/static/main.js, it's already in the cache and served quickly.

The configuration is easy. Here is an overview how to implement it:

1. Sign up for a cdn. Follow the cdn provider instructions of how to add a subdomain such as cdn.example.com, and allow the CDN to use it.

2. Point the cdn to your site at www.yourdomain.com. The CDN will request the files from Nginx, in the same way that the browsers requested these files before. In production, Nginx serves the static files, so the CDN will get it from Nginx directly.
3. The CDN caches the files, so it will call your site's Nginx only when the file is missing from the CDN cache (or by a specific config which tells it to refresh the cahce). Otherwise the CDN uses it's own cache, and the site Nginx serves most of the time as the Apache/mod_wsgi proxy.

3. **The CDN calls Nginx to get the files, but the browsers that visit your site don't**. The browser does not use reuests like **www.yourdomain.com/media/logo.png**, since this will lead it to your site's Nginx.Rather, the browser calls something like **cdn.yourdomain.com/media/logo.png**, which goes to the CDN.
4. In order to point the browser to the CDN and not to the site, add the following settings to settings_production.py:

       STATIC_URL = '//cdn.yourdomain.com/static/'
       MEDIA_RES_URL = '//cdn.yourdomain.com/media/'
       MEDIA_URL = '//cdn.yourdomain.com/uploads/'
       
      Commit, deploy, and reload the site. 

5. Development and deployment did not change. Whenever you change a static file, the deployment runs collectstatic and reloads the site. The site will request the CDN for a new file (collectstatic hashed the changes), the CDN will request this new file from your site's Nginx and save it to the CDN's cache.

Other CDN use their own storage, and you have to upload the files to this storage. Django allows to customize collectstatic to push the files to the caching or CDN correct storage, see the django docs. 

 Support this project with my affiliate link| 
--------------------------------------------|
https://www.linode.com/?r=cc1175deb6f3ad2f2cd6285f8f82cefe1f0b3f46|




