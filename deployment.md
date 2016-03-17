#Deployment


[Overview](#overview)    
[Before You Deploy](#before-you-deploy)    
[Deployment of Python, HTML Changes](#deployment-of-python-html-changes)   
[Deployment of javascript, CSS Changes](#deployment-of-js-css-changes)    
[Deployment with Images, Media](#deployment-with-images--or-ther-media-resources)   
[Deployment of Database Schema Changes](#deployment-of-database-schema-changes)    



## Overview

Deployment is fairly simple, since the whole site is stored on a single server.   
All the deployment actions run on this one server: code updates, `collectstatic` for static files, and db migrations. 
   
To deploy, you push the master branch to the production remote at `@server:/home/django/mysite/site_repo`, and reload the code to mod_wsgi.    
The master branch should always be (closing) stable.     
As you add or update your code, work in a separate branch, and merge to master when something is stable and ready for production. 

The on-going development progress is pushed to the central project repository at `@server:/home/django/mysite.git`.    
This is the `origin` remote of the development repository. This repo is separated from the production website, so you can push often to save your progress to this repo. Simply use `git push` or `git push --all`.
    
After you pushed a new master version that is ready for production, deploy  it to the website.


*Note: Deployment commands are provided with a fabric fabfile. The fabfile is saved at your home directory. To use fabric, you need a public ssh key without a passphrase.*


## Before You Deploy

Check that everything works on the local Nginx & Aapache mod_wsgi site. If js or css changed, you should run:

     you@dev-machine$ python ~/myprojects/mysite/manage.py collectstatic
     
If the database schema changed (models or custom schema sql) you should run:

     you@dev-machine$ python ~/myprojects/mysite/manage.py makemigrations
     you@dev-machine$ python ~/myprojects/mysite/manage.py migrate
     
     
Then reload the site:


    you@dev-machine$ site-reload (or site-up)
    
    
Test the site.    
If everything works, and all changes are commited, push to the main repository:
 
 	you@dev-machine: git push
 		
Now you are ready to deploy the production website.
 
*Note: In a more advanced stage, you would run a tests suite before deployment*

      

## Deployment of Python, HTML Changes

Changes to Deploy| Status
--------|-------
Python, HTML| Yes
js, css|No
Images, Media|No
DB Schema|No

If the only commited changes from the last deployment are changes in Python or HTML:


    you@dev-machine$ fab deploy
    
    

This will push to production/master, and reload mod_wsgi.


## Deployment of js, css Changes

Changes to Deploy| Status
--------|-------
Python, HTML| Any
js, css|Yes
Images, Media|No
DB Schema|No

If you commited changes to javascript or css since the last deplyoment: 

	you@dev-machine$ fab deploy:True 

This will run the same as `fab deploy`, but will also run `collectstatic`.

*Note: It's safe to deploy with `collectstatic` even if you didn't change any js or css file. `collectstatic` checks the static files, and if nothing changed it will keep the current files.*



## Deployment with Images  (or ther media resources)

Changes to Deploy| Status
--------|-------
Python, HTML| Any
js, css|Any
Images, Media|Yes
DB Schema|No



Images, and other non js,css media resources, are not saved in the repository.    

Upload the new images, or the changed images, to the server:


    you@dev-machine: scp ~/myprojects/mysite/media_resources/new_img.png django@PUB.IP.IP.IP:~/mysite/media_resources/
    
    
   *Replace PUB.IP.IP.IP with the actual server public IP, and `myprojects`, `mysite` with the actual name of the project directories.*
   

After all the site images were copied to the server:

	you@dev-machine$ fab deploy:True
	
Similarily to the deployment of js,css, this will run `collectstatic` on the server, and update the static files with the new images.

*Note: If you find yourself uploading resources frequently, you can add a fabric command to upload and deploy. Edit the fabfile in your home directory.*

*Note: If you want to save images in the repository, and upload them with git, see the project refernce [Static Files: Images](project_ref.md#staic-files-images-or-other-media)*

***Managing Images in the repository: **
By default, the project images are managed **outside** the repository. This is really a personal preference. If you want to save images **in** the repository, just add an images directory to the repository to `mysite/site_repo/static/images/`, copy the images to this directory, push and deploy with `fab deploy:True`.
Since `collectstatic` picks the files in `mysite/site_repo/static/`, it will also pick the `images`.


## Deployment of Database Schema Changes

Changes to Deploy| Status
--------|-------
Python, HTML| Any
js, css|Any
Images, Media|Any
DB Schema|Yes

After any changes in the models, or a custom sql, run:

    you@dev-machine$ python ~/myprojects/mysite/manage.py makekigrations
    you@dev-machine$ python ~/myprojects/mysite/manage.py migrate
    
  
 Check that everything works on the local site, then commit.

Before any database change, it's better to backup.    
Run the backup **before** you deploy the code with the new migrations, otherwise django will report an error.   
Run:

	you@dev-machine$ fab site_maintenance
	
Check that the site is actually in maintenance, then: 	

	you@dev-machine: fab backup
	
This will backup the database with `manage.py dumpdata` to the `/home/django/mysite/db_backup` directory, on the server. The file is a json created by django.    
By default, the file name is a timestamp.    
To provide another name:

	you@dev-machine: fab backup:Before_Migs_01234
	
*Note: To load a backup, you will have to ssh to the server and use `manage.py loaddata`. See the django docs. Loading data will revert all data to the date and time of the backup, so be careful (and load it in maintenance mode, and only after backing up the current database properly)* 

*Note: To backup with `mysqldump` see [What's Next](what_next.md). The mysqldump utility runs SQL, and unlike `manage.py`, obviously, regardless of the django code. So it will backup any data in the database, even when manage.py fails (migrations issues, missing models etc). See the django docs about dumpdata and loaddata*



**Now for the deployment**:

	you@dev-machine$ fab maintenance // if already in maintenance after backup, not required    
    you@dev-machine$ fab deploy  // or fab deploy:True if the changes contain js,css)
    you@dev-machine$ fab migrate
    you@dev-machine$ fab site_up // or fab site_auth_on to password protect the site
    
   
This will go maintenance, deploy code, migrate the server's database, and reload the site.
    

Support this project with my affiliate link| 
-------------------------------------------|
https://www.linode.com/?r=cc1175deb6f3ad2f2cd6285f8f82cefe1f0b3f46|




     
  


   
   
   
    

