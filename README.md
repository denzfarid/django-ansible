# Repo setup
* Generate Django project using the liip/django-template
* Run the install script:
```
curl -sS https://raw.githubusercontent.com/liip/django-ansible/master/install.sh | /bin/bash
```

# Server setup
* Install base server with Ubuntu 16.04
* Check if python is installed, when not install it with sudo apt-get install python-minimal
* Make sure the server can clone the configured git repo

# Deploy
Go to the deployment directory, there are two scripts:

```setup.sh```: Installs all software and setups the app
```deploy.sh```: Updates the code only

# License
MIT
