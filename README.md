# Netbox 

## Overview

This documentation describes how to install NetBox, configure NetBox with Let's encrypt and enable SAML2 for Zitadel.


### Prerequisite:

* Ubunt-22.0.4
* All updates/upgrade completed (sudo apt update && upgrade)
* Nginx (apt install nginx)
* Hosts file is configured
* Hostname file is configured
* Date/Time corrected (sudo timedatectl set-timezone America/Chicago)
* Static IP Address
* Install locate (apt install mlocate)
* Net-tools (apt install net-tools)

### PostgreSQL Database Installation

```
apt install -y postgresql redis-server
```

### login Postgres

```
sudo -u postgres psql
```

### Database Create and configure

```
CREATE DATABASE netbox;
```
```
CREATE USER netbox WITH PASSWORD 'J5brHrAXFLQSif0K';
```
```
ALTER DATABASE netbox OWNER TO netbox;
```

### Installation Dependencies 

```
apt install -y python3 python3-pip python3-venv python3-dev build-essential libxml2-dev libxslt1-dev  libpq-dev libssl-dev zlib1g-dev
```

### Check Python Version

```
python3 -V
```

### Download NetBox

```
wget https://github.com/netbox-community/netbox/archive/refs/tags/v3.6.5.tar.gz
```

### Extract Netbox package

Extract Netbox and create a soft link.

```
sudo tar -xzf v3.6.5.tar.gz -C /opt
```
```
sudo ln -s /opt/netbox-3.6.5/ /opt/netbox
```

### Create System user

```
sudo adduser --system --group netbox
```
```
sudo chown --recursive netbox /opt/netbox/netbox/media/
```
```
sudo chown --recursive netbox /opt/netbox/netbox/reports/
```
```
sudo chown --recursive netbox /opt/netbox/netbox/scripts/
```

### Generate a key for SECRET_KEY

Save the output for the configuration step below. This will be added to Netbox configuration file.

```
python3 /opt/netbox-3.6.5/netbox/generate_secret_key.py/generate_secret_key.py
```

## NetBox Configuration file (configuration.py)

navigate to the following directory.
```
cd /opt/netbox-3.6.5/netbox/netbox
```
Copy the configuration_example.py file and rename it.
```
cp configuration_example.py configuration.py
```

Edit the configuration.py file.

```
vi /opt/netbox-3.6.5/netbox/netbox/configuration.py
```

In this section of the configuration.py file add the secert key.

```
SECRET_KEY = '!3#qSHaL&(7mw38....................j^mKZG@$Ax'
```

NOTE: ALLOWED_HOST will be configured to [] till the certificates are made from CertBot below.

```
ALLOWED_HOSTS = ['localhost']
DATABASE = {
  'NAME': 'netbox', # Database name
  'USER': 'netbox', # PostgreSQL username
  'PASSWORD': 'J5brHrAXFLQSif0K', # PostgreSQL password
  'HOST': 'localhost', # Database server
  'PORT': '', # Database port (leave blank for default)
  'CONN_MAX_AGE': 300, # Max database connection age (seconds)
}
```
```
REDIS = {
  'tasks': {
  'HOST': 'localhost', # Redis server
  'PORT': 6379, # Redis port
  'PASSWORD': '', # Redis password (optional)
  'DATABASE': 0, # Database ID
  'SSL': False, # Use SSL (optional)
},
'caching': {
  'HOST': 'localhost',
  'PORT': 6379,
  'PASSWORD': '',
  'DATABASE': 1, # Unique ID for second database
  'SSL': False,
  }
}
```
### Remote File Storage

Disabled in configuration.py file.

```
# 'ENGINE': 'django.db.backends.postgresql', # Database engine
```
Set DEBUG to true
```
DEBUG = True
```
Save and close file.

### Logging configuration

```
sudo mkdir /var/log/netbox
```
```
sudo touch /var/log/netbox/netbox.log
```
```
sudo chown -R netbox.netbox /var/log/netbox
```

NetBox configuration.py file. 

```
vi /opt/netbox-3.6.5/netbox/netbox/configuration.py
```
Add/Configure this section. 

```
LOGGING = {
  'version': 1,
  'disable_existing_loggers': False,
  'formatters': {
  'normal': {
  'format': '%(asctime)s %(name)s %(levelname)s: %(message)s'
  },
},
'handlers': {
    'file': {
    'level': 'DEBUG',
    'class': 'logging.handlers.WatchedFileHandler',
    'filename': '/var/log/netbox/netbox.log',
    'formatter': 'normal',
  },
},
'loggers': {
    'django': {
    'handlers': ['file'],
    'level': 'INFO',
},
'netbox': {
    'handlers': ['file'],
    'level': 'INFO',
    },
  },
}
```

### Create a Super User

Apply Data migration

```
python manage.py migrate
```

```
source /opt/netbox/venv/bin/activate
```

Change directory.

```
cd /opt/netbox/netbox
```

Create super user and follow the promt.

```
python3 manage.py createsuperuser
```

### Schedule the Housekeeping Task

```
sudo ln -s /opt/netbox/contrib/netbox-housekeeping.sh /etc/cron.daily/netbox-housekeeping
```
### Test the Application

Change directory

```
cd /opt/netbox-3.6.5/netbox
```

```
python3 manage.py runserver 0.0.0.0:8000 --insecure
```
###  Verify Service Status

```
redis-cli ping
```
Warning: If the test service does not run, or does not complete checks that show "OK",
something has gone wrong. Call for help. Do not proceed with the rest of this guide until the
installation has been corrected.

## NetBox Service

Gunicorn Setup

```
sudo cp /opt/netbox/contrib/gunicorn.py /opt/netbox/gunicorn.py
```
Systemd setup
```
sudo cp -v /opt/netbox/contrib/*.service /etc/systemd/system/
```
```
sudo systemctl daemon-reload
```
```
sudo systemctl start netbox netbox-rq
```
```
sudo systemctl enable netbox netbox-rq
```
Check  Status
```
systemctl status netbox.service
```

## Create SSL Certificate with Let's Encrypt

There are a few prerequisites to use Let’s Encrypts. You have to accept the ToS of Let’s Encrypt to register an account.
Port 80 of the node needs to be reachable from the internet. There must be no other listener on port 80.
The requested (sub)domain needs to resolve to a public IP of the Node.

Install CertBot

```
sudo apt install certbot python3-certbot-nginx
```

Create Certificates

```
sudo certbot --nginx
```

Example:

```
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address (used for urgent renewal and security notices)
(Enter 'c' to cancel): email.domain.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.3-September-21-2022.pdf. You must
agree in order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: y
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: n
Account registered.
Please enter the domain name(s) you would like on your certificate (comma and/or
space separated) (Enter 'c' to cancel): netbox.domain.com
Requesting a certificate for netbox.domain.com
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/netbox.domain.com/fullchain.pem
Key is saved at: /etc/letsencrypt/live/netbox.domain.com/privkey.pem
This certificate expires on 2024-03-13.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.
Deploying certificate
Successfully deployed certificate for netbox.domain.com to /etc/nginx/sites-enabled/default
Congratulations! You have successfully enabled HTTPS on https://netbox.domain.com
```

### Netbox SAML Plugin Install

Prepping for NetBox plugin to be enabled. Configuring the following files before starting services.

```
apt install pkg-config libxml2-dev libxmlsec1-dev libxmlsec1-openssl xmlsec1
```
```
source /opt/netbox/venv/bin/activate
```
```
cd /opt/netbox
```

Using Pip3 install packages needed.

```
pip3 install xmlsec
```
```
pip3 install django3-auth-saml2
```
```
pip3 install netbox-plugin-auth-saml2
```

Add plugin's to local requirement text file.

```
sudo sh -c "echo 'django_saml2_auth' >> /opt/netbox/local_requirements.txt"
```
```
sudo sh -c "echo 'django3-auth-saml2' >> /opt/netbox/local_requirements.txt"
```

```
sudo sh -c "echo 'netbox-plugin-auth-saml2' >> "/opt/netbox/local_requirements.txt""
```

Configure Netbox for SSO using the SAML2 Plugin

```
vi /opt/netbox/netbox/netbox/configuration.py
```

Adjust ALLOWED_HOST to FQDN.

```
ALLOWED_HOSTS = ['netbox.domain.com']
```

### Zitadel XML file

Retrieve the metadata from Zitadel. Add /saml/v2/metadata end point on Zitadel instance. Copy
the Metadata and Paste it in the file called DCIM.xml

```
vi /opt/netbox/DCIM.xml
```
Save and close file

Adjust the configuration.py file as shown below

```
vi /opt/netbox/netbox/netbox/configuration.py
```
Plugin Configurations
```
PLUGINS = ['django3_saml2_nbplugin']
PLUGINS_CONFIG = {
        'django3_saml2_nbplugin':{
        'AUTHENTICATION_BACKEND': 'django3_saml2_nbplugin.backends.SAML2CustomAttrUserBackend',
        'ASSERTION_URL': 'https://netbox.hungry-howard.com/',
        'ENTITY_ID':'https://netbox.hungry-howard.com/',
        'METADATA_LOCAL_FILE_PATH': '/opt/netbox/DCIM.xml',
        'CUSTOM_ATTR_BACKEND': {
            "USERNAME_ATTR": "emailaddress",
            "MAIL_ATTR": "emailaddress",
            "FIRST_NAME_ATTR": "givenname",
            "LAST_NAME_ATTR": "surname",
        '   ALWAYS_UPDATE_USER': True,
    }
  }
}

```
# Remote Authentication configurations

NOTE: Debugging issues, add DEBUG = False to DEBUG = True

Configure the following lines as shown below.

```
REMOTE_AUTH_ENABLED = True
```
```
REMOTE_AUTH_BACKEND = 'netbox.authentication.RemoteUserBackend'
```
```
REMOTE_AUTH_AUTO_CREATE_USER = True
```
Create/Name the button for SSO.  

```
BANNER_LOGIN = '<a href="/api/plugins/sso/login" class="btn btn-primary btn-block">Login with SSO</a>'
```
Save and close file.

### Nginx Configuration

Edit nginx site file.

```
vi  /etc/nginx/sites-available/netbox
```

Configure site file.

```
map $http_x_forwarded_proto $thescheme {
  default $scheme;
  https https;
}
server {
  listen 80 default_server;
  server_name netbox.domain.com;
  return 301 https://$host$request_uri;
}
server {
    listen 443 ssl;
    server_name netbox.domain.com;
    ssl_certificate /etc/letsencrypt/live/netbox.domain.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/netbox.domian.com/privkey.pem; # managed by Certbot
    client_max_body_size 25m;
    proxy_set_header X-Forwarded-Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Proto $thescheme;
    add_header P3P 'CP="ALL DSP COR PSAa PSDa OUR NOR ONL UNI COM NAV"';
    location /static/ {
    alias /opt/netbox/netbox/static/;
}
location / {
    proxy_pass http://127.0.0.1:8001;
}
  location /login/ {
    proxy_pass http://127.0.0.1:8001/api/plugins/sso/login/;
}
    location /sso/ {
    proxy_pass http://127.0.0.1:8001/api/plugins/sso/; # Must have a trailing slash to strip the original path
  }
}
```
Disable default site.

```
cd /etc/nginx/sites-enabled
```
```
rm default
```

Enable netbox site.

```
ln -s /etc/nginx/sites-available/netbox  /etc/nginx/sites-enabled/netbox
```


Configure the setting.py file.

```
vi /opt/netbox/netbox/netbox/settings.py
```

Add the following to the bottom of the file.

```
SAML2_AUTH_CONFIG = {
  'AUTHENTICATION_BACKEND': 'django.contrib.auth.backends.RemoteUserBackend',
  'METADATA_AUTO_CONF_URL': "https://zitadel.domain.com/saml/v2/metadata"
}
```
### Netbox Security 

Enable Login required  by setting this  to TRUE to permit only authenticated users to access any part of NetBox.

Edit the following file
```
vi /opt/netbox/netbox/netbox/configuration.py
```

```
LOGIN_REQUIRED = True
```

### Restart service 

Restart nginx services.

```
systemctl restart nginx
```

Restart Netbox service.

```
systemctl restart netbox
```

Conclusion:

### Configuration.py file

NetBox configuration.py file should look something like this when completed.


```
ALLOWED_HOSTS = ['netbox.domain.com']
DATABASE = {
  'NAME': 'netbox', # Database name
  'USER': 'netbox', # PostgreSQL username
  'PASSWORD': 'J5brHrAXFLQSif0K', # PostgreSQL password
  'HOST': 'localhost', # Database server
  'PORT': '', # Database port (leave blank for default)
  'CONN_MAX_AGE': 300, # Max database connection age
}
REDIS = {
  'tasks': {
    'HOST': 'localhost',
    'PORT': 6379,
    'USERNAME': '',
    'PASSWORD': '',
    'DATABASE': 0,
    'SSL': False,
},
'caching': {
    'HOST': 'localhost',
    'PORT': 6379,
    'USERNAME': '',
    'PASSWORD': '',
    'DATABASE': 1,
    'SSL': False,
  }
}
SECRET_KEY = '!3#qSHaL&(7mw38hdF0kHi98w+6x%N&34t3vh$(Oj^mKZG@$Ax'

ADMINS = [
  ]
ALLOW_TOKEN_RETRIEVAL = False
AUTH_PASSWORD_VALIDATORS = [
  BASE_PATH = ''
  CORS_ORIGIN_ALLOW_ALL = False
  CORS_ORIGIN_WHITELIST = []
  CORS_ORIGIN_REGEX_WHITELIST = [
]
CSRF_COOKIE_NAME = 'csrftoken'
DEBUG = False
DEFAULT_LANGUAGE = 'en-us'
EMAIL = {
  'SERVER': 'localhost',
  'PORT': 25,
  'USERNAME': '',
  'PASSWORD': '',
  'USE_SSL': False,
  'USE_TLS': False,
  'TIMEOUT': 10, # seconds
  'FROM_EMAIL': '',
}
ENABLE_LOCALIZATION = False
EXEMPT_VIEW_PERMISSIONS = [
]
  INTERNAL_IPS = ('127.0.0.1', '::1')
  LOGGING = {
  'version': 1,
  'disable_existing_loggers': False,
  'formatters': {
    'normal': {
    'format': '%(asctime)s %(name)s %(levelname)s: %(message)s'
  },
},
'handlers': {
  'file': {
    'level': 'DEBUG',
    'class': 'logging.handlers.WatchedFileHandler',
    'filename': '/var/log/netbox/netbox.log',
    'formatter': 'normal',
  },
},
'loggers': {
    'django': {
    'handlers': ['file'],
    'level': 'INFO',
  },
    'netbox': {
    'handlers': ['file'],
    'level': 'INFO',
    },
  },
}
LOGIN_PERSISTENCE = False
LOGIN_REQUIRED = True
LOGIN_TIMEOUT = None
LOGOUT_REDIRECT_URL = 'home'
METRICS_ENABLED = False
PLUGINS = ['django3_saml2_nbplugin']
PLUGINS_CONFIG = {
    'django3_saml2_nbplugin': {
    'AUTHENTICATION_BACKEND': 'netbox.authentication.RemoteUserBackend',
    'ASSERTION_URL': 'https://netbox.domain.com/',
    'ENTITY_ID':'https://netbox.domain.com',
    'METADATA_LOCAL_FILE_PATH': '/opt/netbox/DCIM.xml',
    'CUSTOM_ATTR_BACKEND': {
    "USERNAME_ATTR": "emailaddress",
    "MAIL_ATTR": "emailaddress",
    "FIRST_NAME_ATTR": "givenname",
    "LAST_NAME_ATTR": "surname",
    'ALWAYS_UPDATE_USER': True,
    }
  }
}
REMOTE_AUTH_ENABLED = True
REMOTE_AUTH_BACKEND = 'netbox.authentication.RemoteUserBackend'
BANNER_LOGIN = '<a href="/api/plugins/sso/login" class="btn btn-primary btn-block">Zitadel</a>'
REMOTE_AUTH_AUTO_CREATE_USER = True
RELEASE_CHECK_URL = None
RQ_DEFAULT_TIMEOUT = 300
SESSION_COOKIE_NAME = 'sessionid'
SESSION_FILE_PATH = None
TIME_ZONE = 'America/Chicago'
DEBUG= True
DATE_FORMAT = 'N j, Y'
SHORT_DATE_FORMAT = 'Y-m-d'
TIME_FORMAT = 'g:i a'
SHORT_TIME_FORMAT = 'H:i:s'
DATETIME_FORMAT = 'N j, Y g:i a'
SHORT_DATETIME_FORMAT = 'Y-m-d H:i'
```


### Settings.py file

Note: This is a large file so just copied the bottom for example.

```
django_apps = plugin_config.django_apps
if plugin_name in django_apps:
django_apps.pop(plugin_name)
if plugin_module not in django_apps:
django_apps.append(plugin_module)
for app in django_apps:
if "." in app:
parts = app.split(".")
spec = importlib.util.find_spec(".".join(parts[:-1]))
else:
spec = importlib.util.find_spec(app)
if spec is None:
raise ImproperlyConfigured(
    f"Failed to load django_apps specified by plugin {plugin_name}: {django_apps} "
    f"The module {app} cannot be imported. Check that the necessary package has been "
    "installed within the correct Python environment."
)
INSTALLED_APPS.extend(django_apps)
sorted_apps = reversed(list(dict.fromkeys(reversed(INSTALLED_APPS))))
INSTALLED_APPS = list(sorted_apps)
if plugin_name not in PLUGINS_CONFIG:
PLUGINS_CONFIG[plugin_name] = {}
plugin_config.validate(PLUGINS_CONFIG[plugin_name], VERSION)
plugin_middleware = plugin_config.middleware
if plugin_middleware and type(plugin_middleware) in (list, tuple):
MIDDLEWARE.extend(plugin_middleware)
if type(plugin_config.queues) is not list:
raise ImproperlyConfigured(
    "Plugin {} queues must be a list.".format(plugin_name)
  )
RQ_QUEUES.update({
    f"{plugin_name}.{queue}": RQ_PARAMS for queue in plugin_config.queues
})
SAML2_AUTH_CONFIG = {
    'AUTHENTICATION_BACKEND': 'django.contrib.auth.backends.RemoteUserBackend',
'METADATA_AUTO_CONF_URL': "https://zitadel.domain.com/saml/v2/metadata"
}
```

### Zitadel XML file

```
<?xml version="1.0"?>
  <md:EntityDescriptor xmlns:md="urn:oasis:names:tc:SAML:2.0:metadata"
    entityID="https://netbox3.hungry-howard.com">
  <md:SPSSODescriptor AuthnRequestsSigned="false" WantAssertionsSigned="false"
    protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
      <md:SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://netbox.domain.com/" />
      <md:NameIDFormat>urn:oasis:names:tc:SAML:2.0:nameid-format:unspecified</md:NameIDFormat>
  <md:AssertionConsumerService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST"Location="https://netbox.domain.com/api/plugins/sso/acs/" index="1" />
    </md:SPSSODescriptor>
</md:EntityDescriptor>
```
![image](https://github.com/HungryHowies/netbox/assets/22652276/d97b540b-fbc8-42ea-ad58-dbc0ca9013ff)

### Using the upgrade Script

To ensure the Latest Netbox verion is install.

Naviaagte to Netbox home directory.

```
cd /opt/netbox
```
Execute upgrade script

```
./upgrade.sh
```

Restart serivces

```
sudo systemctl restart netbox netbox-rq
```






















