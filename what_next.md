
# What's Next

The following steps are suggested after you successfully installed a server with one-click-django-server, a development environment with one-click-django-dev, and had several successful deployment cycles.

[Staging Environment](#staging-environment)    
[jQuery, Bootstrap & other external libraries](#jquery-bootstrap--other-external-libraries)    
[Add Developers](#add-developers)    
[Security](#security)    
[Save Secrets Outside the Repository](#save-secrets-outside-the-repository)    
[MySQL Dumps](#mysql-dumps)    
[Backup](#backup)    
[Separate the Single Server to a Webserver & DB Server](#separate-the-single-server-to-a-webserver--db-server)    
[Https](#https)    
[CDN](#cdn)    


## Staging Environment

You can use the [one-click-django-server](#https://github.com/Aviah/one-click-django-server) scripts  to install a staging environment.    

The repositories chain would be dev >> staging >> production. If you push to production only from staging, then only tested code is deployed. 

However, staging can be used for testing and experiments with non-production code, so you may want another staging machine, or still use dev >> staging, and then deploy with dev>> production. 

A few tips:

+ Adjust the git workflow.
+ Add to staging a `robots.txt` file, to blocks search engines from indexing the staging site
+ Password protect the staging site, or limit access by IP
+ Selenium is a great tool to test the stage website before pushing the code to production. See [django-selenium-page-objects](https://github.com/Aviah/django-selenium-page-objects)
+ Add a specific settings file for staging, like `settings_stage.py`, similar to `settings_dev.py` and `settings_production.py` 


## jQuery, Bootstrap & other external libraries

Most of the popular libraries have a free CDN, which is faster, free, and probably available in the browser cache already. To use these libraries in your site, add the library links to the base template, where they will be available to every template that `extends` the base template.

Here in an example for jQuery and Bootstrap:

    <head>
    
        <script type="text/javascript" src="https://code.jquery.com/jquery-2.2.0.min.js"></script>
        <script type="text/javascript" src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/js/bootstrap.min.js"></script>
        <link type="text/css" rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css">
   
        ...        
    </head>
 
Many javascript libraries, including Bootstrap, require jQuery, so it's a good idea to put jQuery first.
    
*Note: You can serve a copy of external libraries with your site static files. But there are many advantages for using the the public CDN: it's faster, you don't pay for the bandwidth, and the latest version is probably already cached in the user's browser, so your site will load faster*

### Add Developers


The project is saved in the `django` user home directory on the server. Any additional developer can access the website with the django account.
   
When you let users access the django account, they will be able to access the site directories, work with the git repository, change everything on the site, reload the site  (with `touch wsgi.py`), and access the django logs. They will not have a sudo permissions, and not the sudo access to Apache, Nginx, and their logs.

To add developers, add their public key to the django user `authorized_keys` file:

    you@dev-machine$ scp john_smith_id_rsa.pub django@PUB.IP.IP.IP:~/
    you@dev-machine$ ssh django@PUB.IP.IP.IP
    django@my-django-server$ cat john_smith_id_rsa.pub >> .ssh/authorized_keys
    
    
    
Now John Smith can ssh to the server and work on the site:
 
    john@dev-machine$ ssh django@PUB.IP.IP.IP
 	
 	
If you add a **new** user with ssh access, you need to add this user to the `sshgroup`: 

    django@my-django-server$ sudo usermod -a -G sshgroup newuser
    
*Note sshgroup is a shell group, created with the setup script. Replace newuser with the actual new username*
    

### Security

Security in this project is really basic, and there are lot of good resources, books, websites, checklists, videos and tools, about how to improve the website security. This is an ongoing effort.

A few ideas where to start:

* The firewall is really basic, add some advanced and more specific rules, both for iptables and ip6tables
* Strong passwords
* Move the django code secrets (settings `SECRET_KEY`, database passwords etc) to the secrets.py outside the repository
* Change the django admin url
* Update the OS regulary
* Use additional tools



### Save Secrets Outside the Repository

It's a good practice to save secrets (e.g. django secret key or the database password) outside the repository. Especially if you are using the public repos in github.    

To move secrets from the repository, save them in `mysite/site_config/secrets.py`. This file is **not** maintained in the repository, but imported to `settings.py`.
Make sure that both development environment and production have an updated secrets file, because it is outside the repository, and is **not** be pushed deployment.

Even when you move a secret outside the repository, the info is still saved in previous versions. It's not trivial to remove it completely from the repository(although possible).    
The best, and simplest way is to **create new secrets** when moving secrets from the repository to another file. If this is a database password, you will have to change the password in MySQL and `FLUSH PRIVILEGES`.



### MySQL Dumps
`mysqldump` saves one or more database into text files. You can actually backup the entire server.
This feature is useful as a backups, or to save a database fixtures. 

To save the database into a file:
    
    you@my-django-server$ mysqldump -u django -p --databases django_db --add-drop-database > django_db.bak.sql
    
*Use the actual django-mysql password (on your settings file)*

To load a backup:

    you@my-django-server$ mysql -u root -p  < django_db.bak.sql
    
*Use the actual mysql root password (provided when you installed MySQL)*


Once the db is dumped to a file, it's easy to tar, zip, scp to another machine, download a production db etc.

*Note: django provides `manage.py dumpdata` which dumps the django site database. This command is also available as `fab backup` from your dev machine. However, the mysqldump is more robust, and can backup data even when manage.py doesn't work because of bugs or migration errors in the python code*


### Backup 

VPS providers usually offer a backup with additional costs. Typically, the backup is scheduled daily, hourly or other interval.  

To make sure that the database is in a known state when the VPS backup runs, you should dump the db just before the backup starts. Run a `mysqldump` with a crontab task a few minutes before the scheduled backup.


### Advanced Deployment and Managment Commands with Fabric

Add your own commands with fabric: A fabric command is a python functions, available in your fabfile, at your home directory. 

By default, the setup script names the file `fabfile_mysite.py` (with your site name instead of `mysite`). The settings to call that file are in `~/.fabricrc`.

Fabric has more ways to find a fabfile in the current directory, or call other fabfile.

What is nice about fabric, is  that it can run on multiple servers, and you can really add what you want: run or restart services, load fixtures, move code etc. 
Fabric really lets you streamline the whole development-staging-production cycle, accross multiple servers, and in standard, plain & simple Python.

See the [Fabric docmentation](http://docs.fabfile.org/en/1.10/).


### Separate the Single Server to a Webserver & DB Server
Idealy, you should separate the database from the webserver. But don't do it before it's really necessary! If you have good traffic, you can simply upgrade the same single VPS with more CPU and more RAM. Doubling the VPS capacity will cost you less than an hour of work. You can wait with the more complex deployment until you absolutly, positively must. 

When you do absolutly positively need to separate servers, the easiest way would be to clone the current single server.

1. Shutdown the VPS
2. Clone it to another VPS, which will serve as the database server
3. In the original VPS, remove MySQL server (using apt-get). This will be the web server. You will still need the MySQL client.
4. In the cloned VPS, remove Nginx and Apache (using apt-get). This will be the database server.
5. Edit `/etc/hostname`,`/etc/network/interfaces` and `/etc/hosts` on the cloned server with the new IP and hostname
6. Edit `/etc/hosts` on both servers, so each one has an entry  of the other
7. Edit the firewall on both servers. The web server (the original VPS), should be able to access the DB server via port 3306. The database server (the cloned VPS) does not need to listen on port 80 (or 443) anymore.
8.  On the database server (the cloned VPS) set the correct MySQL permissions, to allow a django user to access it from the web server machine:  
        
        you@cloned-vps-db-server: mysql -uroot -p
        mysql> CREATE USER 'django'@'webserver' IDENTIFIED BY 'djangomysqlpassword';  
        mysql> GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,ALTER,INDEX,CREATE TEMPORARY TABLES,LOCK TABLES, EXECUTE ON django_db.* TO 'django'@'webserver';  
        mysql> FLUSH PRIVILEGES;
             
9. In `settings_production.py`, change the `DATABASE` settings for django. Since now the server does not use localhost, you will need to move the `DATABASE` config to `settings_dev.py` and `setting_production.py`, each one with another host and possibly another passowrd.



### Https

Https is handled by Nginx.  When Nginx receives an https request, as a proxy, Nginx decrypts it and sends the unencrypted request to Apache-mod_wsgi.

So when you move a site to https, it does not require many changes on the django side: Your django code continues to get the same unencrypted traffic. However, it's still important to get into the details in order to implement https properly.

The recommended practice is to move entire site to https, and avoid mixed http/https websites. 

Here is a general overview of the steps to move the site to https:

1. Buy the SSL certificate. The cheapest will do, unless you really need the fancy certificates with the company name in the browser's bar  

2. Create a certificate signing request (CSR) with a matching key, using openssl, from the command line. You can save the files in `/etc/ssl`. Openssl asks you a few questions to create the CSR. The "Common Name" question is where you provide the domain. See your specific certificate issuer instructions about what exactly you should type here, depends on the certificate type and what you need (subdomains, for both www.exapmle.com and example.com etc). 

3. Send the CSR to the certificate issuer (usually with a web form). If you purchased the simplest domain validation certificate, you should receive the certificate by mail in minutes. You will recieve two files: your own certificate and the issuer intermediate certificates.

4. Save the certificate files on the server, and create one file with both (`cat your_certificate >> intermidiate_certificate`).

5.  Move the current Nginx server to https: edit `/etc/nginx/sites-enabled/django`. Change the server to listen on 443, and add ssl configs: `ssl_on`, `ssl_certificate` for the certificate+intermedate file, `ssl_certificate_key` of the key file. see Nginx docs.

6. In `/etc/nginx/sites-enabled/django`, add another Nginx server that listens to the same domain on port 80, and rewrites everything to https: `rewrite ^ https://www.example.com$request_uri? permanent;`

7. Restart Nginx. If everything works, `curl www.yourdomain.com` should return a 301 redirect.

8. Now you will need a few fixes to the code:
	* Secure cookies, see `SESSION_COOKIE_SECURE`, `CSRF_COOKIE_SECURE`
	* Make sure that when a full url path is used the protocol is always https (e.g. email links, redirects etc)
	* Set `SECURE_PROXY_SSL_HEADER` to https. Django is behind the Nginx proxy, so it sees **http**. It needs to know that the **proxy** received https.
	* If you use external libraries, CDNs or other resources, make sure everything is https
	* Use `//` as the url base: `//path/to/page` instead of `/path/to/page`. This tells the browsers to follow whatever protocol called the url. You can use https in production and http in development, with the same code, as long as the url base is `//`
	* Save the specific django https settings in `settings_production.py`. 
	* On the dev local site you can still work with http without any certificate.
	* Add the django security middleware to the site settings MIDDLEWARE, and configure it's options. Note that many of the middleware options are available also directly with Nginx, which will provide better preformance.
	* Check the site with `manage.py check --deploy`
	* Read carefully the django docs about https

9. Test your site in Qualysis free online SSL checking tool.
10. To make sure the cookies are secure, you can see them in Chrome developer tools.

A few things to consider:

+ Configure HSTS (recommended in Nginx, also possible with django security middleware), which tells the browsers to always request https from your site. This is a more secure option, provided you will not want to use http.
+ Tell Google that your new https site is actually the **same** previous http site. Google has ton of staff about this topic, and it's important to do it right tto keep the Google rank when moving to https (use canonical links, webmaster tools etc, see Google's resources).  

*Note: A simple domain validation SSL certificate is cheap. But if you use a CDN, adding your own certificate to receive https CDN may be expensive*

### CDN
   
The simplest CDN just pulls a file from the original url, and distribute it with anohter url.    
In a typical deployment, the browser requests `cdn.example.com/static/main.js`. The CDN goes to `www.example.com/static/main.js`, pulls `main.js` to the CDN's cache, and then sends it to the browser.    
The next time someone asks for `cdn.example.com/static/main.js`, it's already in the CDN's cache and served quickly.

The configuration is easy. Here is an overview how to implement it:

1. Sign up for a CDN. Follow the CDN's provider instructions of how to add a subdomain such as `cdn.example.com`, and allow the CDN to use it.

2. Point the cdn to your site at `www.yourdomain.com`. The CDN will request the files from your website, in the same way that a client browser requests these files. The CDN pulls the files with the site production urls, directly from  Nginx.
3. The CDN then caches the files. It has to call your Nginx once to get the file, afterwards it will use it's cache.
4. Only when the file is missing from the CDN cache, it will ask for it from your website Nginx. This could happen, because CDNs clear files from their cache if a file was not requested for a given period.
5. **The CDN calls Nginx to get the files, but the browsers that visit your site don't**. The browser does not use requests like `www.yourdomain.com/media/logo.png`, since this will lead it to your site's Nginx. Rather, the browser calls something like `cdn.yourdomain.com/media/logo.png`, which goes to the CDN.
4. To point the browser to the CDN and **not** to the site, add the following settings to `settings_production.py`:

        STATIC_URL = '//cdn.yourdomain.com/static/'
        MEDIA_RES_URL = '//cdn.yourdomain.com/media/'
        MEDIA_URL = '//cdn.yourdomain.com/uploads/'
       
      *Use the actual CDN url* 

7. With the CDN, the website Nginx is busy most of the time serving as a proxy to Apache-mod-wsgi. The CDN offloads the media files, and the user will get these images and media much faster from the CDN's cache.

Development and deployment are the same. Whenever you change a static file, deploy with `fab deploy:True` to run `collectstatic`. This will update the static files and reload the site. When the user requests for one of those new static files (`collectstatic` renamed the files with the new hash), the CDN gets this new file from your site's Nginx, and saves it to the CDN's cache.

Some CDN do not pull from your current site, but rather use their own storage. You  have to upload the files to this CDN's storage. Django allows to customize `collectstatic` to push the files to the correct CDN cache, see the django docs. 

 Support this project with my affiliate link| 
--------------------------------------------|
https://www.linode.com/?r=cc1175deb6f3ad2f2cd6285f8f82cefe1f0b3f46|




