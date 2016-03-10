#Playground


The playground let's you play and experiment a bit in the one-click-django environment. Make sure you installed the server, and the local development site.

If you haven't already - it's easy, the setup script auto installs (almost) everything:    
[Production website and Ubuntu 14.04 Server](https://github.com/aviah/one-click-django-server/master/readme.md)    
[Development local site, OSX El-Capitan](https://github.com/aviah/one-click-django-dev-osx-el-capitan/master/readme.md)    
[Development local site, Ubuntu 14.04 Trusty](https://github.com/aviah/one-click-django-dev-ubuntu-14-04-trusty/master/readme.md)


Ok! The site is up, development local site is up, time to play. This doc will guide you how the basic develop-deploy cycle works.

*A note about an IDE: This is obviously a personal choice, what you feels works for you. If you haven't selected an IDE yet, I can recommend Wing IDE Pro which I use for years. It has the "Python Zen", just fits to the Python programer,  integrated with django, git, excellent debugging and great support* 

We start with a few very simple experiments.

See what commands are ready for you **localy**, to run on the **production** site:

	you@dev-machine: fab -l
	

Fabric should show the list:

	Available commands:

	 backup
    deploy
    migrate
    site_auth_off
    site_auth_on
    site_maintenance
    site_up


Woot! Let's move the site to maintenance mode:

	you@dev-machine: fab site_maintenance
	
Fabric will run this command, and show you each step. It will ask you for your production user password.    
When done, the site should be in maintenance mode. Check it in the browser.

Now bring the site up again, with, well, site_up:

	you@dev-machine: fab site_up
	
The production site should be up and running. Check it in the browser.   
Great!


Let's continue with some code changes.    
Say that we want to add a few exclamation marks to the "Hello World" title. To make the message important!!!!

Enter to the **site_repo directory** of your site, and with your favourite Python IDE or any editor:

	you@dev-machine: nano home/views.py
	
Then add some !!!!!!!

	 context = {'page_title':'It Works',
                 'intro':'Hello World!!!!!!'}
Save and exit.        

Test this change on the local site with the django development server.    
Assuming you are in the site_repo dir:

	you@dev-machine: cd ..
	you@dev-machine: ./manage.py runserver
	
This starts the development server, which should output:

	January 20, 2016 - 11:09:35
	Django version 1.8.7, using settings 'site_repo.settings'
	Starting development server at http://127.0.0.1:8000/
	Quit the server with CONTROL-C.
	
	
With a browser on your local machine, browse to the local site, at 127.0.0.1:8000.
The new message, with the !!!!!, should appear.
Stop the django runserver with Ctrl+C.

Now it's time to commit and push this change.
Check the local repository:

	you@dev-machine: cd site_repo
	you@dev-machine: git status
	

This is what has changed:

	# On branch master
	
	# Changes not staged for commit:
	#   (use "git add <file>..." to update what will be committed)
	#   (use "git checkout -- <file>..." to discard changes in working directory)
	#
	#	modified:   home/views.py
	#
	no changes added to commit (use "git add" and/or "git commit -a")   

The home/views.py file is obviously modified.    
Stage the change:

	you@dev-machine: git add .
	you@dev-machine: git status
	
Just to make sure that the modified file is ready to commit:

	# On branch master
	# Changes to be committed:
	#   (use "git reset HEAD <file>..." to unstage)
	#
	#	modified:   home/views.py
	#


Commit and push:

	you@dev-machine: git commit -m 'home page message'
	you@dev-machine: git push
	
This pushes the change to the site bare repository, on the server:

	Counting objects: 7, done.
	Compressing objects: 100% (4/4), done.
	Writing objects: 100% (4/4), 357 bytes, done.
	Total 4 (delta 3), reused 0 (delta 0)
	To django@92.53.193.10:/home/django/site_repo.git
    00bfc74..5a04cb6  master -> master
    
The push goes to the central project repository. You can push often,  it will not change the production website, just saves the gradual progress of your work to the main project git repo.
    
So now you have made a change, tested it localy, and pushed it to the the main project repo.
It seems like deployment time.   
Easy enough:

	you@dev-machine: fab deploy

Fabric tells you exactly what it runs:

	[django@92.53.193.10] Executing task 'deploy'
	[localhost] local: git --git-dir=/home/usename/my-django-projects/my-site/site_repo/.git push production master
	Counting objects: 7, done.
	Compressing objects: 100% (4/4), done.
	Writing objects: 100% (4/4), 357 bytes, done.
	Total 4 (delta 3), reused 0 (delta 0)
	To django@92.53.193.10:/home/django/my-site/site_repo/
   	00bfc74..5a04cb6  master -> master
	[django@45.33.93.118] run: git reset --hard
	[django@45.33.93.118] out: HEAD is now at 5a04cb6 home page welcome message

	[django@92.53.193.10] run: rm ~/my-site/site_config/*.py
	[django@92.53.193.10] run: touch ~/my-site/site_config/__init__.py
	[django@92.53.193.10] run: cp ~/my-site/site_repo/settings_production.py ~/my-site/site_config/
	[django@92.53.193.10] run: touch ~/my-site/site_repo/wsgi.py

	Done.
	Disconnecting from django@45.33.93.118... done.
	
	
Check the site! Or actually, "Check the site!!!!!".
It should have a few more exclamation marks.

Great. This deployment was actually simple: since we changed only Python code, deployment is just pushing the new code and reloading mod_wsgi with the new code.     
These are the deployment steps that fabric runs:

1. Push master to remote production master
2. Update the production working directory with the latest update, with git reset --hard
3. Update the site_config directory, by copying the settings_production.py from the repository. Settings files are maintained in the repository, but the the specific environment settings file is loaded to mod_wsgi from **outside** the repository, from the site_config directory (this allows to run specific environemnt adjustments, see [Project Reference](project_ref.md) )
4. Finally, to tell mod_wsgi to reload the code, you need to touch the wsgi.py file.


For the next step, we will update a static file, main.css. We will change the button fonts color ("static files info"), to the color blue.        
From the site_repo directory:

	you@dev-machine: nano static/main.css
	
Edit this line as follows:

	.info {font-weight:bold;color:blue;}
	
Save and exit, then run the django development server:

	you@dev-machine: cd ..
	you@dev-machine: ./manage.py runserver
	
Browse to the local site at 127.0.0.1:8000.
The button fonts appear in shining blue? Good.    
Exit the development server with Ctrl+C.
   
Push this change to the project's main repository:

	you@dev-machine: cd site_repo
	you@dev-machine: git status
	you@dev-machine: git add .
	you@dev-machine: git status
	you@dev-machine: git commit -m 'the color is blue'
	you@dev-machine: git push
	

Ready for deployment.
The static file deployment is a bit different: loading mod_wsgi is not enough, because it only handles Python code. We have to run collectstatic as well.    
Still easy:

	you@dev-machine: fab deploy:True
	
This will run the same deployment as before, but with one more step, run collectstatic.   
Django, via fabric,  will ask you:


	[django@92.53.193.10] out: This will overwrite existing files!
	[django@92.53.193.10] out: Are you sure you want to do this?

	[django@92.53.193.10] out: Type 'yes' to continue, or 'no' to cancel: yes
 
Enter 'yes'.

Then, in the fabric output, you should see a line like this:

	[django@92.53.193.10] out: Post-processed 'main.css' as 'main.8339f0693044.css'
	
This lines tells you that django just hashed the new main.css and used the hash for the file name. When a browser hits  the site, the site will tell the browser to use this new css file - and not the old main.css file from the browser's cache. Thus, collectstatic lets you safely provide static files with long expiry dates: when the file changes, the browser will always use the newer version. But if nothing changes, the cache is much faster.

Check it and browse to your production site. The button fonts should appear in blue.

**This is it:  you have the complete working development cycle for a django web application, with development, test localy, and deploy.**


**Next:**

Check the project files, especially the repository (site_repo). They include a few examples of the basic django concepts.

For a more complete tutorial, continue with [Part 1: Create the Polls app](tutorial_part1.md). This is a tutorial that based on the official django tutorial, implemented in the one-click-django-environment, with git, deployment, and a real develop-deploy cycle.

For more details see the full [One Click django Docs](readme_docs.md), and, of course, the awesome, comprehensive and easy to understand django documentation. 

Support this project with my affiliate link| 
-------------------------------------------|
https://www.linode.com/?r=cc1175deb6f3ad2f2cd6285f8f82cefe1f0b3f46|




	