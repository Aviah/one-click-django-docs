#Project Reference


[Project Layout](#project-layout)    
[Git Repositories](#git-repositories)    
[Production & Development Settings](#production--development-settings)    
[Staic Files: javascript,css](#staic-files-javascriptcss)    
[Staic Files: Images (or other media)](#staic-files-images-or-other-media)    
[Django Admin](#django-admin)    
[Cache](#cache)    
[Web Servers](#web-servers)    




## Project Layout

The project layout is a mixture of common practices and personal preferences.   
However, django is flexible, and everything can be easily changed.


**Production**

    /home/django/
    |
    |- site_repo.git  (bare repository, dev pushes to this repo as the "origin" remote)
    |
    |- mysite (the project directory)
        |
        |- manage.py (the django utility for the project)
        |
        |- db_backup (database dumps directory)        
        |
        |- logs (logs directory)
        |   |
        |   |- main.log (production log, to log use logging.getLogger("main").info("production msg"), logs everything from log level info)
        |   |- debug.log (to log use logging.debug("debug msg"), logs when DEBUG_LOG=True)
        |   |- debug_db.log (django log, all db interactions, logs when DEBUG_DB_LOG=True)  
        |
        |- media_resources (pre-prepared site media directory, e.g. logo, icons, etc. Source directory for collectstatic, allows to keep images outside the repository)
        |
        |- media_uploads (media directory, for user uploaded resources, e.g. avatars)
        |
        |- site_config (environment specific settings directory on the PYTHONPATH)
        |   |
        |   |- settings_production.py (settings.py will try to import this file, make sure it exists in production)
        |   |- settings_tmp.py (settings.py will try to import this file, useful for ad-hoc settings during dev)
        |   |- site_auth.py (credentials for password-protected site via Apache auth, if used)
        |   |- secrets.py (to save passwords, SECRET_KEY, outside the repository. Important when the repository is public)
        |
        |- site_repo (the website git repo. The dev "production" remote. This is the actual website code that is loaded to mod_wsgi)
        |   |
        |   |- settings.py
        |   |- urls.py
        |   |- wsgi.py
        |   etc...
        |
        |- static_root (static js & css files, created from the repository with manage.py collectstatic)
        
**Development Machine**

Similar to production, but without the bare repository.    
The site is saved in you home directory.

On a Mac:

    /Users/username/myprojects/
    |
    |- mysite (the project directory)
        |
        |- manage.py (the django utility for the project)
        |
        
    ...




On Ubuntu:

    /home/username/myprojects/
    |
    |- mysite (the project directory)
        |
        |- manage.py (the django utility for the project)
        |
        
    ...



## Git Repositories

###Origin Remote 
On the server, a bare git repository `@server:/home/django/site_repo.git`. This is the project central repo, which is separated from the website code, so you can push often to save the progress of your work.     To push everything, from your local `site_repo`:

	you@dev-machine: git push --all remote origin

###Production Remote  
On the server, the production website repository. This is the code that mod_wsgi loads.     
The repo is `@server:/home/django/mysite/site_repo`.   
Do not push directly to this repository, but rather use [Deployment](deployment.md)


###Development Local Website 
A clone of the server bare repository, at your user directory, at `@dev-machine:~/myprojectsd/mysite/site_repo`.


*Note: "myprojects" or "mysite" are the actual names of the project and site you provided when installing the server and the dev environment.*

**Summary:** When you work on the code on your local machine, you push to the origin remote, and deploy to the site production repository. Keep the master branch stable, and work on temporary or other branches for development. See [Deployment](deployment.md)

    
    
## Production & Development Settings  


### A Separate settings file for development and production

The common practice is to create several environment specific settings files, in addition to the django main settings file.    
The project includes the following environment settings files: 

* `settings_production.py`

* `settings_dev.py`

When django loads, the main `settings.py` tries to import both. On production, only `settings_production.py` is available, and on the dev machine, only `settings_dev.py` is available. 
Thus, the final settings for each environemnt will be different.  

The most important change between production and the dev machine is `DEBUG`: In `settings_production.py`, the `DEBUG=False`, and in `settings_dev.py` it's `DEBUG=True`.


Note that **both** settings files are maintained in the repository. This is why  settings.py tries to import the specific environment settings files from a **non-repository** directory.    
The correct environment file should be copied on each machine from the repository (the `site_repo` directory), to the `site_config` directory. 

*Note: `site_config` is on the PYTHONPATH and you can import from it*

So the settings on production looks as follows:

    /home/django/
    |
    |- site_repo.git  (bare repository, dev pushes to this repo)
    |
    |- mysite (the project directory)
        |
        |- manage.py (the django utility for the project)
        |
        |- site_config
        |	|
        |	|- settings_production.py (imported when the site loads)
        |	|- secrets.py (optional)        
        |
        |- site_repo
        |   |
        |   |- settings.py
        |   |- settings_production.py (just saved in repo)
        |   |- settings_dev.py (just saved in repo)               
        |   |- urls.py
        |   |- wsgi.py
        ...
        
And on the development machine:

    ~/myprojects/mysite/
    |   
    |- site_repo.git  (bare repository, dev pushes to this repo)
    |
    |- mysite (the project directory)
        |
        |- manage.py (the django utility for the project)
        |
        |- site_config
        |	|
        |	|- settings_dev.py (imported when the site loads)
        |
        |- site_repo
        |   |
        |   |- settings.py
        |   |- settings_production.py (maintained in repo)
        |   |- settings_dev.py (just saved in repo)               
        |   |- urls.py
        |   |- wsgi.py
        ...

When you commit a change to `settings_dev.py` or `settings_production.py` in the repository, you have to copy the changed file to `site_config/` and reload the site.    
The deployment script runs this copy, and on the dev machine you will have to manaualy copy `settings_dev.py` after you changed it, from the repository to `site_config`.

To use the optional `secrets.py` to save passwords and the SECRET_KEY outside the repository, which is important if your repository is public, see [Save Secrets Outside the Repository](https://github.com/Aviah/one-click-django-docs/blob/master/what_next.md#save-secrets-outside-the-repository)


### Ad-hoc settings

Sometimes you just need an ad-hoc settings change to test or try something.

For this purpose, there is a another settings file in `site_config`: `settings_tmp.py`. Use it for these ad-hoc settings.

The `settings_tmp.py` is imported after `settings_dev`, but **before** `settings_production.py`, so production settings always have a priority.
  


## Staic Files: javascript, css

Out of the box, django separates external resources into two categories: site resources, and user uploads. Each is served from it's own url and root directory.  
However, it's often handy to have **three** categories of static files, as follows:

1. **Scripts:** javascript & css files, that change often during development.
1. **Non js/css**: Pre-prepared, optimized assets, such as the logo, icons, images etc.
1. **User uploads**: e.g. avatars


### Where the files are saved?

Scripts files (js, css) are saved and maintained in the repository `@dev-machine:~/mysite/site_repo/static` directory. The local development site will serve them, by django, from the repository. 

On production, django's `collectstatic` copies the scripts to external media directory, `@server:/home/django/mysite/static_root`. The files from this directory are served by the Nginx directly (and not by django).


### Serving Scripts: js,css

**Development:**    

Webserver | django development server   
----------|--------------------------  
**Settings**|**DEBUG=True**   

To see the changes after you change the static files:

	you@dev-machine: python ~/myprojects/mysite/manage.py runserver

Django development server will serve the scripts directly from the repository. This is great for development, since every change is immediately reflected in the  website, at `127.0.0.1:8000`.

Django development server will **not** serve static files when `DEBUG=False`.

**Production:**
  

Webserver | Nginx + Apache   
----------|--------------------------  
**Settings**|**DEBUG=False**  


Static files are served directly by Nginx. Nginx serves the files from the `STATIC_ROOT` directory (`@server:/home/django/mysite/static_root`), and does **not** pass the js & css requests to django. 

Since the files are **not** served from the repository, every change to the static files in the repository requires an update to the files in `mysite/static_root`: 

    you@production-machine: python ~/myprojects/mysite/manage.py collectstatic
    you@production-machine: site-reload
    
You don't have to run the above lines, since this update with `collectstatic`  is a part of the deployment, see [Deployment](deployment.md).
    
Django identifies the correct file to serve with the `static` template tag, so you must use this tag in the templates:

	<link href="{% static 'foo.css' %}" type="text/css" rel="stylesheet">
	<script src="{% static 'bar.js' %}" type="text/javascript">
	
See the coding reference for [Templates](https://github.com/Aviah/one-click-django-docs/blob/master/coding_ref.md#templates)*


**Testing**

Webserver | Nginx + Apache   
----------|--------------------------  
**Settings**|**DEBUG=True**  



Before you push code to production, it's better to make sure it works with Apache/Nginx on the dev machine (and not only on the django development server).     
If you changed any static file, you run `collectstatic` and reload the site, and then browse to the local Apache/Nginx site at `127.0.0.1`:

    you@dev-machine: python ~/myprojects/mysite/manage.py collectstatic
    you@dev-machine: site-reload

 
**Summary:**    
1. During **development**, use the django development server, and the files are served directly from the repository when DEBUG=True.   
2. On **production**, static files are served with Nginx, and you have to deploy with `collectstatic`, using `fab deploy:True`.


*Note: django also supports serving static files from the application's  directory, see the django docs*

## Staic Files: Images (or other media)

### Where the website images are saved?

1. Images are saved outsite the repository at `mysite/media_resources/`.
2. `collectstatic` looks for new or changed images, because the images directory is included in `STATICFILES_DIRS`
3. References to images in templates use the `static` template tag:

		<img src="{% static 'foo.png' %}">
		
3. The images files are served similarly to js,css static files, by Nginx or the django development server.

Since images are saved outside the repository, you will need to scp the files to production:
Example:

        you@dev-machine: scp logo.png django@PUB.IP.IP.IP:~/mysite/media_resources/
        
Then run `collectstatic` on the server. When you use `fab deploy:True` the fabric deployment script will run it as well, see [Deployment with Images](https://github.com/Aviah/one-click-django-docs/blob/master/deployment.md#deployment-with-images--or-ther-media-resources).

### Managing Images in the repository
By default, the project images are managed **outside** the repository. This is really a personal preference. If you want to save images **in** the repository, just add an images directory to the repository to `mysite/site_repo/static/images/`, copy the images to this directory, push and deploy with `fab deploy:True`.
Since `collectstatic` picks the files in `mysite/site_repo/static/`, it will also pick the `images`.

*Note: The favicon is also saved as an image in `mysite/media_resources/favicon.ico`, see the project's `base.html` template.*


### Users' Uploads

1. Images are served
2. Files are served from **mysite/media_uploads/** directory, both for runserver , or via Nginx
2. Uploads are configured with MEDIA_URL and MEDIA_ROOT, django's settings for uploaded files.
3. Refernces in templates should use {{ MEDIA_URL }}:

		<img src="{{MEDIA_URL}}bar.png">		
4. Uploads Limit:
You can limit the upload file size in django, with a custom file uploader (see the django docs).    
The project limits requests in Apache conf file, with LimitRequestBody, to 10MB. You can change it according to the typical uploads of your site:

    	you@vps-machine$ nano /etc/apache2/apache2.conf.django
    	you@vps-machine$ nano /etc/apache2/apache2.conf.django.auth
    	you@vps-machine$ site_auth_off (or site_auth_on)    
   

*Note: For external files, expire date, and browser cache, see [Web Servers]()*


## Django Admin

**To Access the production website admin:**

www.yourdomain.com/admin

**To Access admin on the local development site:**

Django dev server: 127.0.0.1:80000/admin   
Nginx+Apache: 127.0.0.1/admin



Then login with the admin username and password, which you provided during the installation. You can change the admin url, see django docs and urls.py in the project.


## Cache

The site is configured with the simple django basic file-based cache, at **mysite/django_cache**.  

Use the cache:

    from django.core.cache import cache
    ...
    cache.set('key','foo')
    cache.get('key')
    
  See an example in home/views.py.

*Note: django provides many other caching options, multiple caches, etc, see the django docs.*


## Web Servers

**Django Development Server**: Serves the site at port 8000. This is the best way to develop and test. The site is served at 127.0.0.1:8000,  and when DEBUG=True, everything is served with this server, including js, css (and images), so every change in the code is immidiatly reflected in the site. However, this server is not secured and not suitable for production.

**Nginx**: Nginx serves requests on port 80. Requests for static files are served directly by Nginx. For all the other requests, Nginx serves as a proxy and passes the requests to Apache. This configuration has many advantages, django docs and other places have a lot of details as well as the pros and cons compared to other configurations.  
Since static files are not served from the repo, collectstatic and site reload is required when the static files change. See External files.

**Apache**: Apache listens on 127.0.0.1:8001. So it doesn't listen to external requests, only to requests that Nginx passes to it. These are the non static/media requests that go to Apache, which sends them to django via mod_wsgi.

**Browsers Cache:** In Nginx, the static, media, and uploads, are configured to 180d expiry. This expiry date will not affect js,css, since each time you change the source of js,css, you should run collectstatic. Then, when you reload the site, django will not request the old cached files. With collectstatic, any change in the js,css scripts is reflected with requests to the newer versions.


**Replace cached images** Images do not have "collectstatic", so during development, in order to see new images instead of the cached, just clear the browser's cache, or use select "ignore cache" from the developer's tools.    
Once images are ready for production, the best solution is to change the image name in the site code, so the browsers will not use an older cached image.   
Otherwise change the 180d config in /etc/nginx/sites-enabled/django. Later, when images change, the best solution is to save the new image with a new name

Support this project with my affiliate link| 
-------------------------------------------|
https://www.linode.com/?r=cc1175deb6f3ad2f2cd6285f8f82cefe1f0b3f46|






