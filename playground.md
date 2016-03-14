#Playground


**The playground let's you play and and experiment a bit with the one-click-django environment** 

## Environment

Make sure you installed the server, and the local development site.    
If you haven't already, it's easy, the setup scripts auto-install (almost) everything:    

* Production website:[ Django site on Ubuntu 14.04 Server](https://github.com/Aviah/one-click-django-server)
* Develop & Deploy:  on [OSX El-Capitan](https://github.com/Aviah/one-click-django-dev-osx-el-capitan), or [Ubuntu 14.04 Trusty desktop](https://github.com/Aviah/one-click-django-dev-ubuntu-14-04-trusty)


*A note about the IDE: This is obviously a personal choice, what you feel that works for you. If you haven't selected an IDE yet, I recommend Wing IDE Pro which I use for years. It has the "Python Zen", just fits to the Python programmer, integrates with django, git, excellent debugging and great support* 


## Play...

Ok! The site is up, development local site is up, time to play.    

To see what commands you can run on **production** from your **local** machine: 

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


Woot! Move the site to maintenance mode:

	you@dev-machine: fab site_maintenance
	
Fabric runs the site_maintenance command, and show you each step. It will ask you for your production user password. When done, the site should be in maintenance mode. Check it in the browser `www.yourdomain.com`

Now bring the site up again, with:

	you@dev-machine: fab site_up
	
The production site should be up and running. Check it in the browser.   


Continue with some code changes.    
Say that we want to add a few exclamation marks to the "Hello World" title. To make the message important!!!!

Enter to the `site_repo` directory, and with your favourite Python IDE, or other editor:

	you@dev-machine: nano home/views.py
	
Then add some !!!!!!!

	 context = {'page_title':'It Works',
                 'intro':'Hello World!!!!!!'}
Save and exit.        

Test this change on the local site with the django development server.    
Assuming you are in the site_repo dir:

	you@dev-machine: cd ..
	you@dev-machine: ./manage.py runserver
	
Django starts the development server, which should output:

	January 20, 2016 - 11:09:35
	Django version 1.8.7, using settings 'site_repo.settings'
	Starting development server at http://127.0.0.1:8000/
	Quit the server with CONTROL-C.
	
	
Browse to the local site, at `127.0.0.1:8000`. The new message should appear with the !!!!!

You can stop the django runserver with Ctrl+C (otherwise it runs and reloads with any code change)

Time to commit and push this change.
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

The `home/views.py` file is obviously modified.    
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
	To django@45.33.93.118:/home/django/site_repo.git
    00bfc74..5a04cb6  master -> master
    
The push goes to the **central project repository**. The central repo saves the gradual progress of your work. You can push often to this repo,  it will not change the production website.  

So now you have made a change, tested it localy, and pushed it to the the main project repo.
It seems like deployment time.   
Easy enough:

	you@dev-machine: fab deploy
	
*Note: to use the fab commands use ssh key without a passphrase. When you create the key, enter an empty passphrase*

Fabric tells you exactly what it runs:

	[django@45.33.93.118] Executing task 'deploy'
	[localhost] local: git --git-dir=/home/usename/my-django-projects/my-site/site_repo/.git push production master
	Counting objects: 7, done.
	Compressing objects: 100% (4/4), done.
	Writing objects: 100% (4/4), 357 bytes, done.
	Total 4 (delta 3), reused 0 (delta 0)
	To django@45.33.93.118:/home/django/my-site/site_repo/
   	00bfc74..5a04cb6  master -> master
	[django@45.33.93.118] run: git reset --hard
	[django@45.33.93.118] out: HEAD is now at 5a04cb6 home page welcome message

	[django@45.33.93.118] run: mv secrets.py secrets.txt
	[django@45.33.93.118] run: rm *.py
	[django@45.33.93.118] run: mv secrets.txt secrets.py
	[django@45.33.93.118] run: touch __init__.py
	[django@45.33.93.118] run: cp ~/my_site/site_repo/settings_production.py ~/my_site/site_config/
	[django@45.33.93.118] run: touch ~/my_site/site_repo/wsgi.py

	Done.
	Disconnecting from django@45.33.93.118... done.
	
	
Check the site (actually, "check the site!!!!!").    
It should now have a few more exclamation marks.

Great. This deployment was actually simple: since we changed only Python code, deployment is just pushing the new code and reload mod_wsgi.     

These are the deployment steps that fabric runs:

1. Push master to `remote production master`
2. Update the production working directory with the latest update, with `git reset --hard`
3. Update the `site_config` directory: fabric copies the `settings_production.py` file from the repository. All the settings files are maintained in the repository, but mod_wsgi loads the settings from `site_config`, which should include only the specific environment settings. See [Project Reference](project_ref.md) )
4. Finally, to tell mod_wsgi to reload the new code, `touch wsgi.py`.

*Note: fabric keeps the `site_config/secrets.py` where you can save sensitive settings like django SECRET_KEY or passwords outside the repository. This option is important especially for public repositories*



Next, we will update a static file, `main.css`. We will change the button fonts to the blue.         
From the `site_repo` directory:

	you@dev-machine: nano static/main.css
	
Edit this line as follows:

	.info {font-weight:bold;color:blue;}
	
Save and exit, then run the django development server:

	you@dev-machine: cd ..
	you@dev-machine: ./manage.py runserver
	
Browse to the local site at `127.0.0.1:8000`.
Do the button fonts appear in shining blue? Good.    
Exit the development server with Ctrl+C.
   
Push this change to the project's main repository:

	you@dev-machine: cd site_repo
	you@dev-machine: git status
	you@dev-machine: git add .
	you@dev-machine: git status
	you@dev-machine: git commit -m 'the color is blue'
	you@dev-machine: git push
	

Now the code is ready for deployment.

The static file deployment is a bit different: reload mod_wsgi is not enough, because it will only handle new Python code. We have to update css as well, and run `collectstatic`.    
Still easy:

	you@dev-machine: fab deploy:True
	
Fabric will run the same deployment as before, but with one more step,  `collectstatic`.   
Django, via fabric, will ask you:


	[django@45.33.93.118] out: This will overwrite existing files!
	[django@45.33.93.118] out: Are you sure you want to do this?

	[django@45.33.93.118] out: Type 'yes' to continue, or 'no' to cancel: yes
 
Enter 'yes'.

Then, in the fabric output, you should see a line like this:

	[django@45.33.93.118] out: Post-processed 'main.css' as 'main.8339f0693044.css'
	
This lines tells you that django just hashed the new `main.css` and used the hash for the file name.    
When a browser hits  the site, the site will tell the browser to use this new css file - and not the old `main.css` file from cache.
Check it and browse to your production site. The button fonts should appear in blue.


*Note: `collectstatic` lets you safely provide static files with long expiry dates. When the file changes, the browser will always use the newer version. If nothing changes, the cache with long expiry dates is much faster.*


**Congrats! you have just completed a full develop-deploy cycle for a django website.**

## Continue to the Tutorial

The tutorial is based on the official polls tutorial, here with git & deployment of the polls app to a real website.

To continue to the tutorial: [Django Tutorial Part 1: Create the Polls App
](https://github.com/Aviah/one-click-django-polls-tutorial/blob/master/tutorial_part1.md)


## What's Next

Take a look at the project files, especially the repository `site_repo`. These files include a few examples of the basic django concepts.

If you are new to django, why not take our version to the official django polls tutorial. It implments the polls app in in this real development-deployment-production environment, with git. When you finish this tutorial, the polls app will run on the real website at `www.yourdommain.com/polls`.    
Start here [Django Tutorial with Deployment](https://github.com/Aviah/one-click-django-polls-tutorial)

For the complete project reference: layout, files, directories, settings, deployment, media files, logging, coding reference etc, see the [Project reference docs](https://github.com/aviah/one-click-django-docs/)

To see how a complete django website is built with this project layout, see the [django-website](https://github.com/Aviah/django-website) repository.

Finally, of course, the awesome, comprehensive and easy to understand [django documentation](https://docs.djangoproject.com). 

Support this project with my affiliate link| 
-------------------------------------------|
https://www.linode.com/?r=cc1175deb6f3ad2f2cd6285f8f82cefe1f0b3f46|




	