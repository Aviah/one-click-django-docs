# Command Line Aliases Reference

A few useful command line aliases for the server and the dev machine

[Server Aliases](#server-aliases)
[Dev Machine Aliases](#dev-machine-aliases)

## Server Aliases

**Load Firewall**

      you@vps-machine$ firewall-up

**Reset Firewall**    
Reset chains and clear all rules, actually no firewall at all. Sometimes useful for debugging, but not recommended!

      you@vps-machine$ firewall-down

**Site Mainenance**    
Replace the django site with a static "Maintenance" page. The maintenance file is at `/usr/share/nginx/html/index.html`:

      you@vps-machine$ site-maintenance

**Site Up**    
Restart Apache and  Nginx with the django site. Use to load the site after maintenance, or when an issue requires a full restart.

      you@vps-machine$ site-up

**Site Reload**    
Touch the `wsgi.py` file, so mod_wsgi reloads the python code (without the full Apache restart). Use after the python code changed, or after the static files changed with `collectstatic`. A reload is required after `collectstatic` creates new files, with the new files hashes:

      you@vps-machine$ site-reload

**Password Protect Site**    
Site password protection with Apache auth. Similar to `site-up`, and configures Apache to require password to access the site. The username and password credentials are saved at `site_config/django_auth.py`:

      you@vps-machine$ site-auth-on

**Remove Site Password**    
Similar to `site-up`, clears Apache password if any. Use to remove the site password after `site-auth-on`:

      you@vps-machine$ site-auth-off

**Tail Logs**    
A quick look at the logs, for debugging. Tails the following logs: `main.log`, `debug.log`, Apache `error.log`, Nginx `error.log`:

      you@vps-machine$ tail-logs
      
      
## Dev Machine Aliases

**Site Up**    
Restarts Apache &  Nginx with the django site. Use to load the site after maintenance, or when an issue requires a full restart.

      you@dev-machine$ site-up

**Site Reload**    
Touch the `wsgi.py` file, so mod_wsgi reloads the python code (without the full Apache restart). Use after the python code changed, or after the static files changed with `collectstatic`. A reload is required after `collectstatic` creates new files, with the new files hashes:

      you@vps-machine$ site-reload
      
      
**Tail Logs**    
A quick look at the logs, for debugging. Tails the following logs: `main.log`, `debug.log`, Apache `error.log`, Nginx `error.log`:

      you@vps-machine$ tail-logs
      
        
## Django Shell Commands
 
Here is an example for the `check` command.    
The `manage.py` has A LOT of commands, see the django docs.    

On the dev machine:

	you@dev-machine: ~/myprojects/mysite/./manage.py check
	
*Note: On the dev machine, you may work a lot from the `site_repo` directory, where the main project repository and code is. So you can run it from `site_repo` with: .././manage.py runserver*
	
On the server:

	you@my-django-server: /home/django/mysite/./manage.py check --deploy
	

Django has it's own shell, which provides the necessary settings to work with django and the django objects - from the python shell. So you can run, in the shell, lines like these :

	you@dev-machine: ./manage.py shell
	>>> from django.contrib.auth.models import User
	>>> u = User.objects.get(pk=1)
	>>> u
	<User: root>
	
And work with your own models, API etc from this shell.

*Note: to work with Wing IDE Pro (recomended!), and the Wing IDE django integration, copy `manange.py` file to the site_repo directory, and create the wing project*.

Manage.py actually runs the commands of `django-admin`, the django shell utility, . The `manage.py` script just adds the environment variable for the django project. When you need a django-admin command that does not depends on these environment variable, simply run `django-admin`.     
Most of the time, however, manage.py is more useful.
       
## Dev Fabric Commands

You run a fabric command on your dev machine, and then fabric runs the command on **the server**.

So running:

	you@dev-machine: fab site_maintenance
	
On your dev machine, will move the **server** to maintenance.

The project includes some useful fabric commands in a fabfile, located in your home directory.    
To see these commands:

		you@dev-machine: fab -l
		
		
These are the fabric commands:
		
		backup
    	deploy
    	migrate
    	site_auth_off
    	site_auth_on
    	site_maintenance
    	site_up
    	
    	
 Load the site password-protected by Apache auth:
	
     you@dev-machine$ fab site_auth_on


Load the site, and clear Apache auth password if any:
	
     you@dev-machine$ fab site_auth_off
     
     
Maintenance:

    you@dev-machine$ fab site_maintenance
    
    
Restart Apache & Nginx with the django site:

	you@dev-machine fab site_up
	
Backup the server database to `@vps-machine:/home/django/mysite/db_backup/`.    
Name the backup file with a time stamp:

	you@dev-machine fab backup
	
Provide a backup file name:

	you@dev-machne: fab backup:"before_migration_12345"
	
For details about `deploy` and `migrate` fabric command, see [Deployment](deployment.md). 

*Note: The fabric commands are available in the fabfile, a python file saved by the setup script. You can always add your own commands. A fabric command is a python functions, in the fabfile. See [What's Next](what_next.md)*
				

Support this project with my affiliate link| 
-------------------------------------------|
https://www.linode.com/?r=cc1175deb6f3ad2f2cd6285f8f82cefe1f0b3f46|











