# Coding Reference

This document is a quick reference. For more details about the project layout, production and development settings, static files, etc., see Project_Ref.

[Python/Django]()    
[Settings]()    
[Logging]()     
[Cache]()     
[Command Line]()



## Python/Django

**Import module**

	
    from site_repo import foo

**Import from a module**

    from site_repo.foo import bar

  
**Import settings**

The conventional django settings import:

    from django.conf import settings
    
  
**Templates**

Templates are saved in the site_repo/templates directory.

The base template (mysite/site_repo/template/base.html) should include all the site global resources, e.g. basic page structure, favicon, css, references to external resources like bootstrap, jquery, fonts etc.

Then, a new template can include only the few lines required for it's specific purpse. Django "extends" the base template, so another  template inherits all the base template code, except where it changes the base template defaults.

To extend the base template, a new template should start with:

	{% extends "base.html" %}
	
If the template uses static resources (js, css, images) it should load the static template tag (once, at the beginning of the template):

	{% load static from staticfiles %}
	
Django uses the static template tag to call the correct file: the static file in production, and the original file during dev. See "Static Files" in the [Project Reference](project_ref.md)
	

*Note: django also supports serving templates from the application's  directory, see the django docs*

*Note: "Extends" is not limited, of course, to the base template. Django allows quiet complex and hierarchical "template inheritence". See the django docs*


**Css & javascript Files in Templates**

The template should load the static template tag (once, at the beginning of the template):

	{% load static from staticfiles %}


To use a css file in a template:

	<link href="{% static 'foo.css' %}" type="text/css" rel="stylesheet">
	
To use a javascript file in a template:

	<script src="{% static 'bar.js' %}" type="text/javascript">

    
**Images in Templates**

The template should load the static template tag (once, at the beginning of the template):

	{% load static from staticfiles %}

An image from the site pre-prepared media which you save to mysite/media_resources (logo, icons etc):

	<img src="{% static 'foo.png' %}">
	
An image from the uploads directory, that should be used to images uploaded by users:

	<img src="{{MEDIA_URL}}bar.png">

The image name should follow the }} brackets, **without** any spaces.

*Note: the MEDIA_URL is used for uploads, but was not renamed to something like "MEDIA_UPLOADS" to respect django defaults*

## Settings

### Settings in the repository

The django docs recommend to use in your settings files only settings you want to change, and let django use it's default settings for everything else.

Put your **production** settings in the repository:

	 site_repo/settings_production.py
	 
Put your **development** settings in the repository:

	 site_repo/settings_dev.py
	 
Put all other settings that are **the same for production & development**, but different from the django defaults, in the repository:

	 site_repo/settings.py
	 
	 
### Settings when the Website Runs

The actual settings when the website runs are, in that order:

1. The default django settings at django/conf/global_settings.py
2. Your main settings file from site_repo/settings.py
3. If exist, settings from site_config/settings_dev.py
4. If exists, settings from site_config/settings_tmp.py
5. If exists, settings from site_config/settings_production.py


The things to remember:

+ Permanenet settings changes go to the **repository**, and should be commited as the rest of the code.   
+ You adjust the environment with the **site_config** directory: on production, it should contain just settings_production.py (and maybe secrets.py).  On the development machine, the settings_dev.py and settings_tmp.py. 
+ If you want to see changes you made in the repository to settings_production.py or settings_dev.py, you should copy the changed file to site_config, and reload the site. The deployment script makes this copy when you deploy, on dev machine you will have to do that manually:

		you@dev-machine: cd ~/myprojects/mysite/
		you@dev-machine: cp site_repo/settings_dev.py site_config/
		you@dev-machine: site-reload
		
+ For **ad-hoc** changes, e.g. turning DEBUG on/off, turning logging on/off, try to play with and test a default django setting, just edit the site_config settings files:
		
		you@dev-machine: cd ~/myprojects/mysite/
		
	The edit the files:

			you@dev-machine: nano site_config/settings_dev.py
			
	Or, to change some settings and keep the original settings_dev as is:

			you@dev-machine: nano site_config/settings_tmp.py
			
	And reload:
		

			you@dev-machine: site-reload

For details, see Project Reference, Deployment

*Note: There is also an option: the site actually tries to load another setting file, if exists: site_config/secrets.py. This is useful if you want to keep secrets outside the repository (e.g. if you open source the project on github).
See What's Next

	 	 
	 
## Logging
           
**Production logging** 

Log to mysite/logs/main.log:

    logging.getLogger('main').info('some production msg')
    
  If you use getLogger a lot, define it in the beggining of the file:

    main_log = logging.getLogger('main')

  Then for each message: 
	
    main_log.info('some production msg')

  *Note: The main logger will log only when logging level >= info*

    
**Debug logging**

To enable debug logging, set djagno settings:

	DEBUG = True
	DEBUG_LOG = True
	
It's handy to set these keys in /mysite/site_config/settings_dev.py, so you can change the settings on and off without any change to the repository source code.

To logs to myprojects/mysite/logs/deubg.log, use:

    logging.debug('some debug msg')
    

Since debug.log is configured as the root logger, there is no need to use getLogger to use it.

   *Note: debug.log captures also the production logging messages, so during dev you can just use the debug.log to see all logging messages, both production and debug*
    
  
**Django database logging**
   
Django has a built in log of all the interactions with the database. 
This is a heavy log, and usually handy when you need a specific database debugging, or  to check if your code does not call the database multiple times for the same data (see django documentation on querysets).

To enable debug logging, set djagno settings:

	DEBUG = True
	DEBUG_DB_LOG = True
	
It's handy to set these keys in /mysite/site_config/settings_dev.py, so you can change the settings on and off without any change to the repository source code.

With these settings, django will auto log all DB interactions to myprojects/mysite/logs/deubg_db.log


*Note: django has many other built in logs, including a db schema changes log. To use them, you will have to add these logs to the settings LOGGING. See django docs, and the LOGGING config in the settings file. For DB logging you can enable the full MySQL logging, which is useful, but significantly slows down the DB*



## Cache

The project is configured to use django basic file cache:


	# import cache
	from django.core.cache import cache
    
    # then use it
    cache.set('key','foo')
    cache.get('key')
    
    
## Command Line

When using the command line to run and test code from the project, you will need to set the environment variable DJANGO_SETTINGS_MODULE:

	you@dev-machine$ python
	you@dev-machine>>> import os
	you@dev-machine>>> os.environ.setdefault("DJANGO_SETTINGS_MODULE","site_repo.settings")
	
	
And you will probably need:

	
	you@dev-machine>>> import django
	you@dev-machine>>> django.setup()
	
	
*Note: If you use an IDE, it's probably let you configure default environment variables that the IDE auto add to a new python shell. In Wing IDE (which I use, and it's awesome), add this environment variable to the project properties.*

For more details, see the [project reference](Project_Ref)

Support this project with my affiliate link| 
-------------------------------------------|
https://www.linode.com/?r=cc1175deb6f3ad2f2cd6285f8f82cefe1f0b3f46|




	

