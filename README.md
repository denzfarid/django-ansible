This project allows you to easily deploy your Django project on a Debian/Ubuntu Linux server.
The following services/software will be installed on the server:
* nginx
* uwsgi
* PostgreSQL
* (optional) SSL with Lets Encrypt/certbot or your own certificate

# Server setup
* Setup a server with Ubuntu 16.04 or Debian 8
* Create a user with passwordless sudo privileges
* As the root user (`sudo su`), generate a passwordless ssh key with `ssh-keygen`
* Add the *root* SSH public key to your git repository, e.g. as deploy key on GitLab/GitHub
* Check if python is installed, if not install it with `sudo apt-get install python-minimal -y`
  or `sudo yum install python`

# Project setup
* Generate Django project using the liip/django-template
* Run the install script in your project root (same directory as `manage.py`):
```
curl -sS https://raw.githubusercontent.com/liip/django-ansible/master/install.sh | /bin/bash
```
* The script added the directory `deployment/` to your project, you should add that to git
* Open the following files and update them to your requirements:
    * `deployment/hosts`: Update the host for your servers hostname/ip address and set the user
    you have created above
    * `deployment/group_vars/all`: Update all placeholder values

# Deploy
Go to the deployment directory, there are two scripts:

* ```setup.sh```: Installs all software and setups the app, you have to run this the first time and any time you
update one of the config files inside `deployment/`
* ```deploy.sh```: Deploys the code only and reloads the app server (uwsgi)

# Multiple environments/systems
To deploy to multiple different environments (e.g. test/staging/prod) you have to do the following:
Make a copy of the files `deployment/group_vars/production` and `deployment/production` and adjust
them for your new environment. Add the name of you new environment to the `hosts:` entry in
`deployment/site.yml`.

# Email
django-ansible uses dj-email-url to configure the email settings. Add this to your
`settings.py`/`base.py` to make it work:
```
import dj_email_url
EMAIL_URL = get_env_variable('EMAIL_URL', 'console://')
email_config = dj_email_url.parse(EMAIL_URL)
EMAIL_FILE_PATH = email_config['EMAIL_FILE_PATH']
EMAIL_HOST_USER = email_config['EMAIL_HOST_USER']
EMAIL_HOST_PASSWORD = email_config['EMAIL_HOST_PASSWORD']
EMAIL_HOST = email_config['EMAIL_HOST']
EMAIL_PORT = email_config['EMAIL_PORT']
EMAIL_BACKEND = email_config['EMAIL_BACKEND']
EMAIL_USE_TLS = email_config['EMAIL_USE_TLS']
EMAIL_USE_SSL = email_config['EMAIL_USE_SSL']
DEFAULT_FROM_EMAIL = get_env_variable('EMAIL_FROM', 'webmaster@localhost')
```

Then add the configuration to your `group_vars/<environment>` file:
```
email_url: smtp://user:password@smtp.example.com:465/?ssl=True
email_from: webmaster@example.com
```

The default EMAIL_URL variable generated by django-ansible is the same configuration as the Django default,
which is sending emails to an smtp server running on port 25 on localhost without authentication.

With the configuration above emails will be printed to the console when running
locally/inside drifter without the `envdir/EMAIL_URL` file.

# Customization
You can add additional installation steps by adding Ansible Tasks to your project.
To do this, create a directory `custom` in the `deployment` directory. This is the root directory
for your custom Ansible role, you can add tasks, handlers, templates and other Ansible artifacts here.
You also have to add your custom role to your `deployment/site.yml` under the `roles` entry.

To e.g. add a cronjob to your project, create a `tasks` directory inside `custom`, copy the file
from `examples/tasks/cronjob.yml` to `<your-project>/deployment/custom/tasks/cronjob.yml` and adjust the configuration
in this file for your needs. You also have to add a `<your-project>/deployment/custom/tasks/main.yml` file which
includes the new task file:

    - include: cronjob.yml

Example task files:

* [Cronjob](examples/tasks/cronjob.yml)
* [Additional Software](examples/tasks/additional_software.yml)
* [wkhtmltopdf](examples/tasks/wkhtmltopdf.yml)

# python-rq task queue
There is an additional role "taskqueue" which setups python-rq with redis
as broker. To use it, add django-rq to your INSTALLED_APPS and define queues
with RQ_QUEUES in your Django settings file. (see example in
`tests/django_app/settings/base.py`)

Then activate the role by adding/uncommenting `- taskqueue` in your
`deployment/site.yml` file.

# License
MIT
