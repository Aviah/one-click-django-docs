# Coding Reference

This document is a quick reference. For more details about the project layout, production and development settings, static files, etc., see Project_Ref.

[Imports](#imports)    
[Templates](#templates)    
[Settings](#settings)    
[Logging](#logging)     
[Cache](#cache)     
[Command Line](#command-line)



## Imports

Import a project module:
	
    from site_repo import foo

Import from a project module:

    from site_repo.foo import bar

  
Import settings (conventional django-settings import):

    from django.conf import settings
    
Relative Imports (conventional relative imports):


	from . import foo
	from ..baz import bar
    
  
##Templates

Templates are saved in the `site_repo/templates` directory.

The base template, `mysite/site_repo/template/base.html`, should include all the site global resources, e.g. the basic page structure, favicon, css, references to external resources like bootstrap, jQuery, fonts etc.

Then, a new template can include only the few lines required for it's specific purpse.    
Django "extends" the base template, and any template can inherit all the base template code, and then change what it needs.

To extend the base template, a new template should start with:

	{% extends "base.html" %}
	

*Note: django also supports templates from the apps'  directory, see the django docs*

*Note: The `extends` template tag is not limited, of course, to the base template. Django allows to build a complex hierarchical "template inheritence". See the django docs*


###Css & javascript Files in Templates

If the template uses static resources - like js, css, images - it should load the static template tag.    
Load the tag once, at the beginning of the template:

	{% load static from staticfiles %}
	
Django uses the `static` template tag to load the correct file:

* The static file in production
* The original file from the repository, during development.

See the project reference for [Static Files](project_ref.md#staic-files-javascriptcss)
	

Use a css file in a template:

	<link href="{% static 'foo.css' %}" type="text/css" rel="stylesheet">
	
Use a javascript file in a template:

	<script src="{% static 'bar.js' %}" type="text/javascript">

    
###Images in Templates

Load the static template tag.    
Load the tag once, at the beginning of the template:

	{% load static from staticfiles %}

An image from the site pre-prepared media directory, which you savde to `mysite/media_resources` (logo, icons etc):

	<img src="{% static 'foo.png' %}">
	
An image from the `mysite/media_uploads` directory, that should be used for the images that users uploaded:

	<img src="{{MEDIA_URL}}bar.png">

The image name should follow the `}}` brackets, **without** any spaces.


## Settings

### Settings in the repository

The django docs recommend to use in your settings files only the settings that you want to change, and let django use it's default settings for everything else.

Keep the specific **production** settings in:

	 site_repo/settings_production.py
	 
Keep the specific **development** settings in:

	 site_repo/settings_dev.py
	 
All other settings that are shared and **the same for both production & development** (but different from the django defaults) in:

	 site_repo/settings.py
	 
	 
### Settings when the Website Runs

The actual settings when the website runs are loaded, in the following order:

1. The default django settings at `django/conf/global_settings.py`
2. Your main settings file  `site_repo/settings.py`
3. If exist, settings from `site_config/settings_dev.py`
4. If exists, settings from `site_config/settings_tmp.py`
5. If exists, settings from `site_config/settings_production.py`


Things to remember:

* A permanent settings change goes to the **repository**, and should be commited as the rest of the code.   
* Adjust the environment with the `site_config` directory.
* `site_config` on production: should contain just `settings_production.py` (and maybe `secrets.py`).
* `site_config` on a dev machine: `settings_dev.py` and `settings_tmp.py`. 

### Permanent settings change, in the repository
   
If you want to update the website with changes that you made in the repository to `settings_production.py` or `settings_dev.py`, you should copy the changed file to site_config, and reload the site. 
The deployment fabric script makes this copy when you deploy. On the dev machine you will have to do that manually:

	you@dev-machine: cd ~/myprojects/mysite/
	you@dev-machine: cp site_repo/settings_dev.py site_config/
	you@dev-machine: site-reload

### Ad-hoc settings change
		
For **ad-hoc** changes, e.g. turning `DEBUG` on/off, turning `logging` on/off, and other tests and experiments,  just edit the `site_config settings` files:
		
	you@dev-machine: cd ~/myprojects/mysite/
		
Then edit the files:

	you@dev-machine: nano site_config/settings_dev.py
			
Or, to change some settings and keep the original `settings_dev.py` as is:

	you@dev-machine: nano site_config/settings_tmp.py
			
And reload:
		
	you@dev-machine: site-reload

For more details see [Deployment](deployment.md), and the project reference for [Production & Development Settings](project_ref.md#production--development-settings)

*Note: The site actually tries to load another setting file, if exists: `site_config/secrets.py`. This is useful if you want to keep secrets outside the repository (e.g. if you open source the project on github).
See [What's Next](what_next.md)*

	 	 
	 
## Logging
           
**Production Logging** 

Log to `mysite/logs/main.log`:

    logging.getLogger('main').info('some production msg')
    
If you use `getLogger` a lot, define it in the beginning of the file:

    main_log = logging.getLogger('main')

Then for each message: 
	
    main_log.info('some production msg')

*Note: The main logger will log only when the logging level >= info*

    
**Debug Logging**

To enable debug logging, set djagno settings:

	DEBUG = True
	DEBUG_LOG = True
	
It's handy to set these keys in `/mysite/site_config/settings_dev.py`, so you can change the settings on/off without any change to the repository source code.

To log to `myprojects/mysite/logs/deubg.log`, use:

    logging.debug('some debug msg')
    

Since `debug.log` is configured as the **root logger**, there is no need to call `getLogger` to use it.

Tip: the `debug.log` captures also the production logging messages, so during dev you can just use `debug.log` to see **all** the logging messages, both production and debug.
    
  
**Django Database Logging**
   
Django has a built in log for all the interactions with the database. 

This is a heavy log, and usually used when you need a specific database debugging, or to check if your code does not call the database multiple times for the same data (see django documentation on querysets).

To enable debug logging, set:

	DEBUG = True
	DEBUG_DB_LOG = True
	
It's handy to add these keys to `/mysite/site_config/settings_dev.py`, so you can change the settings on/off without any change to the repository source code.

With these settings, django will auto log all the interactions with the database to `myprojects/mysite/logs/deubg_db.log`.


*Note: django has many other built in logs, including a database schema changes log. To use these logs, you will have to update the `LOGGING` settings. See the django docs, and the `LOGGING` config in the settings file. For additional database logging  logging you can enable the full MySQL logging, which is useful, but significantly slows down preformance*



## Cache

The project is configured to use the basic django's file cache:


	# import cache
	from django.core.cache import cache
    
    # then use it
    cache.set('key','foo')
    cache.get('key')
    
    
## Command Line

When using the command line to run and test project code, you will need to set the environment variable `DJANGO_SETTINGS_MODULE`:

	you@dev-machine$ python
	you@dev-machine>>> import os
	you@dev-machine>>> os.environ.setdefault("DJANGO_SETTINGS_MODULE","site_repo.settings")
	
	
And probably also:

	
	you@dev-machine>>> import django
	you@dev-machine>>> django.setup()
	
	
*Note: If you use an IDE, it's usually lets you configure the default environment variables for the python shell. In Wing IDE (which I use, and it's awesome), add this environment variable to the project properties.*

For more details, see the [Project Reference](project_ref.md)

Support this project with my affiliate link| 
-------------------------------------------|
https://www.linode.com/?r=cc1175deb6f3ad2f2cd6285f8f82cefe1f0b3f46|




	

