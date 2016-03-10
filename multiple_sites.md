#Using one-click-django server for multiple sites 

**All developed from the same dev machine**


## Overview

For simplicity, the one-click-django project is built and configured as one website, with one matching development environment. The one-server/one-site  saves some hustle, especially if you are new django.

However, it's fairly easy to adjust your setup to auto install and develop multiple sites with the same scripts.

If you want to use the scripts from multiple websites, you can build a dedicated VPS server for each website. Wehn a VPS costs $10 a months, if you have 2-3 website it's not worth it to config a one server for multiple websites.

Then configure the dev machine to hold multiple sites. It's possible to install virtualenv, create separate environment for each site, and a separate virtual host in the webservers. I suggest here a solution which to me seems simpler: save the config files of each site, and when you switch sites just run a script that copies the config files to the correct place, and restart the webservers.

### Why Multiple VPS?

The easiest solution is dedicate one VPS per site. Off course, you can configure the same VPS to serve multiple sites, but when a decent VPS costs $10 a month, it just doesn't worth the time and effort, unless you serve really many sites, and the savings from using one VPS for all multiple websites is substantial.


### The Simplest Solution
1. A separate VPS for each website with one-click-django-server
2. A separate dev virtual machine for each website with one-click-django-dev
3. Done


## Configure the Same Dev Machine for Multiple Websites

** The following doc uses mysite1, mysite2, as the names of the two sites**


If you already have a development environment for another django site on your dev machine, run the following steps


### Save Config Files and DB Dump of mysite1

**Config Files**

Add "etc" directory to the repository at ~/myprojects/mysite1/site_repo.   
To this directory, copy the following files:

On a Mac:

1. /Library/Python/2.7/site-package/django_projects.pth
2. /usr/local/etc/nginx/nginx.conf
3. /usr/local/etc/nginx/extra/django-site-nginx
4. /etc/apapche2/httpd.conf
5. /etc/apache2/extra/django-site-apache
6. ~/.django_site_aliases
7. ~/.fabricrc

On Ubuntu:

1. /usr/lib/python2.7/dist-packages/django_projects.pth
2. /etc/nginx/nginx.conf
3. /etc/nginx/sites-enabled/django
4. /etc/apache2/apache2.conf
5. /etc/apache2/ports.conf
6. /etc/apache2/sites-enabled/django
7. ~/.django_site_aliases
8. ~/.fabricrc


When all the files copied to the repository /etc directory, add the files and commit.

*Note: If you don't want to keep these files in a repo, copy to another directory, e.g. ~/myrpojects/mysite1/etc/*

*Note: If you made other configuration changes for this site, to other files, save these file to the repository etc directory as well*


**Database Dump**

Make a dump of mysite1 databases:

		you@dev-machine$ mysqldump -uroot -p --all-databases --add-drop-database > ~/myprojects/mysite1/mysite1_dump.sql

### Install Development Environment for mysite2

At this point you have:
 
1. Two separate VPS, each with a one-click-django-server website: one for mysite1, the other for mysite2.
2. A development machine that is configured to the 1st one-click-django-server mysite1. 
3. All the config files for mysite1 are saved and commited, and you made a database dump.

Now continue and install the development environment for the second site mysite2, on the same development machine, with one-click-django-dev.    
When the dev environment is complete, the local django site at 127.0.0.1 (or 127.0.0.1:8000 with django development server) should show mysite2.

### Save Config Files and DB Dump of mysite2

Now add etc directory to the 2nd site repository, and copy the files, similarily to what you saved to the first site.

Add the file and commit.

Make a dump of the database:

	you@dev-machine$ mysqldump -uroot -p --all-databases --add-drop-database > ~/myprojects/mysite2/mysite2_dump.sql


### Write a switch script between mysite1 and mysite2

Write a bash script that accepts a site name, and copies it's config files to the correct directories, and restart webservers. You can call the script switch_site.sh.

Here are suggested templates.

On a Mac:

		#!/bin/bash
		cd ~/myprojects/$1/site_repo/etc/
		sudo cp  django_projects.pth /Library/Python/2.7/site-packages/
		cp nginx.conf /usr/local/etc/nginx/
 		cp django-site-nginx /usr/local/etc/nginx/extra/
 		sudo cp httpd.conf /etc/apapche2/
 		sudo cp django-site-apache /etc/apache2/extra/
 		cp .django_site_aliases ~/
 		cp .fabricrc ~/
 		nginx -s stop
 		nginx
 		sudo apachectl restart
 		mysql.server restart
 		
On Ubuntu:

		#!/bin/bash
		cd ~/myprojects/$1/site_repo/etc/
		sudo cp django_projects.pth /usr/lib/python2.7/dist-packages/
		sudo cp nginx.conf /etc/nginx/
		sudo cp django /etc/nginx/sites-enabled/
		sudo cp apache2.conf /etc/apache2/
		sudo cp ports.conf /etc/apache2/
		sudo cp django /etc/apache2/sites-enabled/
		cp .django_site_aliases ~/
		cp  .fabricrc ~/
		sudo service nginx restart
		sudo service apache2 restart
		sudo service mysql restart


## Switch Sites



To switch from mysite1 to mysite2:

	you@dev-machine$ mysqldump -uroot -p --all-databases --add-drop-database > ~/myprojects/mysite1/mysite1_dump.sql
	you@dev-machine$ mysql -uroot -p < ~/myprojects/mysite2/mysite2_dump.sql
	you@dev-machine: ./switch_site.sh mysite2
		
		
To switch from mysite2 to mysite1:

	you@dev-machine$ mysqldump -uroot -p --all-databases --add-drop-database > ~/myprojects/mysite2/mysite2_dump.sql
	you@dev-machine$ mysql -uroot -p < ~/myprojects/mysite1/mysite1_dump.sql
	you@dev-machine: ./switch_site.sh mysite1
		
		
After you switch, restart the terminal (or use source).

Support this project with my affiliate link| 
-------------------------------------------|
https://www.linode.com/?r=cc1175deb6f3ad2f2cd6285f8f82cefe1f0b3f46|








       
 
 



		

 