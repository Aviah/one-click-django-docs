#One-Click-Django Docs

## Intro


### The one-click-django project goal

The goal is to build as simply as possible a working django enviroment:

* Single server
* Single local development site
* Simple Deployment
* For one developer.     

If you are new to django, this simple setup can save you a lot of hussle, and yet lets you develop and deploy to a real website ASAP.    
Once your are comfortable with django and the environment, you can extend and scale (see [What's Next](whats-next.md)).

You need to install the server, and the local development site. If you haven't installed it already, it's easy, the setup scripts auto-install (almost) everything:    

* Production website:[ Django site on Ubuntu 14.04 Server](https://github.com/Aviah/one-click-django-server)
* Develop & Deploy:  on [OSX El-Capitan](https://github.com/Aviah/one-click-django-dev-osx-el-capitan), or [Ubuntu 14.04 Trusty desktop](https://github.com/Aviah/one-click-django-dev-ubuntu-14-04-trusty)

### Before you continue with the project docs:

Please read:

* The one-click-django-server [README](https://github.com/aviah/one-click-django-server/blob/master/readme.md)
*  The one-click-djang-dev README: [OSX](https://github.com/Aviah/one-click-django-dev-osx-el-capitan/blob/master/README.md) or [Ubuntu desktop](https://github.com/Aviah/one-click-django-dev-ubuntu-14-04-trusty/blob/master/README.md) 
* The [Playground](playground.md), which let's you play and experiment a bit with the one-click-django project


If you are new to django, why not take our version to the official django polls tutorial. It implments the polls app in a real development-deployment-production environment, with git.    
When you finish this tutorial, the polls app will run on the real website at `www.yourdommain.com/polls`.    
Start here [Django Tutorial with Deployment](https://github.com/Aviah/one-click-django-polls-tutorial) 


## Docs at a Glance


###Reference
[Coding Refence](coding_ref.md)    
[Deployment](deployment.md)    
[Command Line Aliases Reference](command_line_aliases_ref.md)    
[Project Reference](project_ref.md), provides a lot of details   

###Advanced:
When you want https, multiple servers, CDN, add more developers to the project, etc.

[What's Next](what_next.md)    
[Multiple Sites](multiple_sites.md)



## Table of Contents


###Project Reference


[Coding Reference](coding_ref.md)   
A quick reference how to use the project when you code: imports, settings, templates, etc.

[Deployment](deployment.md)   
Simple deployment recipes with the fabric commands.    
Deploy Python, HTML, javascript, css, and migrations.    
Also some additional fabric commands for common website admin.

[Command Line Aliases Reference](command_line_aliases_ref.md)  
The project includes a few useful command line aliases for the server and the dev machine

[Project Reference](project_ref.md)    
More details about the project's directories & files layout, git repositories, production & development settings, static files (css,javascript,images, media), django admin, cache and the web servers

[What's Next](what_next.md)  
The goal of this project is to provide a complete yet simple production, development and deplyoment environment. This document provides an overview of some more advanced topics: staging, external libraries, add more developers, save secrets outside the repository, MySQL dumps, backup, separate webserver & database server, https, CDN and so on.

[Multiple Site](multiple_sites.md)   
How to use the one-click-django scripts to handle multiple websites from the same development machine


Support this project with my affiliate link| 
-------------------------------------------|
https://www.linode.com/?r=cc1175deb6f3ad2f2cd6285f8f82cefe1f0b3f46|




