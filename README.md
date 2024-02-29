# netbox
Netbo lab install

NetBox Installation Final
Overview
This documentation describes how to install NetBox and configure NetBox with Let's encrypt and
SAML for SSO.
I remove the step for self-signed certificate, Let's Encrypt is the replacement.
WARNING: if you copy and paste be careful of white spaces in configuration files

## Prerequisite:

Ubunt-22.0.4
All updates/upgrade completed (sudo apt update && upgrade)
Nginx (apt install nginx)
Hosts file is configured
Hostname file is configured
Date/Time corrected (sudo timedatectl set-timezone America/Chicago)
Static IP Address
Install locate (apt install mlocate)
Net-tools (apt install net-tools)

## PostgreSQL Database Installation
sudo apt install -y postgresql

## Check Version

psql -V

## Database Creation

sudo -u postgres psql

## Database Create

CREATE DATABASE netbox;
CREATE USER netbox WITH PASSWORD 'J5brHrAXFLQSif0K';
ALTER DATABASE netbox OWNER TO netbox;

-- the next two commands are needed on PostgreSQL 15 and later
\connect netbox;
GRANT CREATE ON SCHEMA public TO netbox;

Verify Service Status

redis-cli ping

## NetBox Installation

sudo apt install -y python3 python3-pip python3-venv python3-dev build-essential libxml2-dev libxslt1-dev libffidev
libpq-dev libssl-dev zlib1g-dev

## Check Version

python3 -V

## Download NetBox

sudo wget https://github.com/netbox-community/netbox/archive/refs/tags/v3.6.5.tar.gz

## Extract package

sudo tar -xzf v3.6.5.tar.gz -C /opt
sudo ln -s /opt/netbox-3.6.5/ /opt/netbox


## Create System user

sudo adduser --system --group netbox
sudo chown --recursive netbox /opt/netbox/netbox/media/
sudo chown --recursive netbox /opt/netbox/netbox/reports/
sudo chown --recursive netbox /opt/netbox/netbox/scripts/

Configuration

cd /opt/netbox/netbox/netbox/
sudo cp configuration_example.py configuration.py

## Generate a key for SECRET_KEY

python3 ../generate_secret_key.py

NetBox Configuration file ( configuration.py)
ALLOWED_HOST will be configured to localhost till the certificates are made from CertBot below.

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

SECRET_KEY = '!3#qSHaL&(7mw38....................j^mKZG@$Ax'

## Logging configuration
```
sudo mkdir /var/log/netbox
sudo touch /var/log/netbox/netbox.log
sudo chown -R netbox.netbox /var/log/netbox
```
Add this section to NetBox configuration.py file
NOTE: If you copy and Paste ensure there are no white spaces.

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
## Remote File Storage
This setting has been disabled in configuration.py file
```
# 'ENGINE': 'django.db.backends.postgresql', # Database engine
```
Upgrade Script
NOTE: This depends on the version of Python installed.

```
/opt/netbox/netbox/netbox
```
## Execute upgrade script

```
sudo PYTHON=/usr/bin/python3.10 /opt/netbox/upgrade.sh
```
## Create a Super User
```
source /opt/netbox/venv/bin/activate
```
```
cd /opt/netbox/netbox
python3 manage.py createsuperuser
```
## Schedule the Housekeeping Task

```
sudo ln -s /opt/netbox/contrib/netbox-housekeeping.sh /etc/cron.daily/netbox-housekeeping
```
## Test the Application
```
python3 manage.py runserver 0.0.0.0:8000 --insecure
```
Warning: If the test service does not run, or does not complete checks that show "OK",
something has gone wrong. Call for help. Do not proceed with the rest of this guide until the
installation has been corrected.
## NetBox Service

Gunicorn

```
sudo cp /opt/netbox/contrib/gunicorn.py /opt/netbox/gunicorn.py
```
Systemd setup
```
sudo cp -v /opt/netbox/contrib/*.service /etc/systemd/system/
sudo systemctl daemon-reload
```
```
sudo systemctl start netbox netbox-rq
sudo systemctl enable netbox netbox-rq
```
## Status
```
systemctl status netbox.service
```

## Create SSL Certificate with Let's Encrypt

There are a few prerequisites to use Let’s Encrypts. You have to accept the ToS of Let’s Encrypt to
register an account.
Port 80 of the node needs to be reachable from the internet. There must be no other listener on
port 80.
The requested (sub)domain needs to resolve to a public IP of the Node.
CertBot Installation command for Nginx
Nginx Configuration
Install CertBot
sudo apt install certbot python3-certbot-nginx

## Create Certificates
```
sudo certbot --nginx
```
root@keycloak:/home/ubuntu# sudo certbot --nginx

```
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address (used for urgent renewal and security notices)
(Enter 'c' to cancel): greg.smith@enseva.com
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
space separated) (Enter 'c' to cancel): netbox3.hungry-howard.com
Requesting a certificate for netbox3.hungry-howard.com
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/netbox3.hungry-howard.com/fullchain.pem
Key is saved at: /etc/letsencrypt/live/netbox3.hungry-howard.com/privkey.pem
This certificate expires on 2024-03-13.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.
Deploying certificate
Successfully deployed certificate for netbox3.hungry-howard.com to /etc/nginx/sites-enabled/default
Congratulations! You have successfully enabled HTTPS on https://netbox3.hungry-howard.com
```
## Add plugin's to local requirement text file

```
sudo sh -c "echo 'django_saml2_auth' >> /opt/netbox/local_requirements.txt"
sudo sh -c "echo 'netbox-plugin-auth-saml2' >> "/opt/netbox/local_requirements.txt""
```
## Netbox Plugin for SSO using SAML2

```
vi /opt/netbox/netbox/netbox/configuration.py
```

Adjust ALLOWED_HOST to FQDN.
```
ALLOWED_HOSTS = ['netbox3.hungry-howard.com']
```
Zitadel XML file

Retrieve the metadata from Zitadel. Add /saml/v2/metadata end point on Zitadel instance. Copy
the Metadata and Paste it in the file called DCIM.xml
```
vi /opt/netbox/DCIM.xml
```
Adjust the configuration.py file as shown below
```
PLUGINS = ['django3_saml2_nbplugin']
PLUGINS_CONFIG = {
'django3_saml2_nbplugin':{
'AUTHENTICATION_BACKEND': 'netbox.authentication.RemoteUserBackend',
'ASSERTION_URL': 'https://netbox.hungry-howard.com/',
'ENTITY_ID':'https://netbox.hungry-howard.com/',
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
```
Remote Authentication configurations
NOTE: Debugging issues add DEBUG = False to DEBUG = True
```
REMOTE_AUTH_ENABLED = True
REMOTE_AUTH_BACKEND = 'netbox.authentication.RemoteUserBackend'
REMOTE_AUTH_AUTO_CREATE_USER = True
```
### This is option create a button for SSO is the auto redirect is NOT configured in Nginx ###
BANNER_LOGIN = '<a href="/api/plugins/sso/login" class="btn btn-primary btn-block">Login with SSO</a>'

Nginx Configuration

There are two ways to configure Nginx. For this demo I choose step #2. I tried to add the
configurations needed but failed so I created a new file /etc/nginx/sites-available/test then
copy and pasted #2 configuration below to that file. Save and close. Execute this command to
write over netbox site file.

```
cp /etc/nginx/sites-available/test /etc/nginx/sites-available/netbox
```
NOTE: Adjust the server_name and fullpath to certificates in each one of these files.
1. In this section you can configure Nginx to bypass the login screen. The auto redirect is enabled
goes right to IDP no login button.
```
map $http_x_forwarded_proto $thescheme {
default $scheme;
https https;
}
server {
listen 80 default_server;
server_name netbox.hungry-howard.com;
return 301 https://$host$request_uri;
}
server {
listen 443 ssl;
server_name netbox.hungry-howard.com;
ssl_certificate /etc/letsencrypt/live/netbox.hungry-howard.com/fullchain.pem; # managed by Certbot
ssl_certificate_key /etc/letsencrypt/live/netbox.hungry-howard.com/privkey.pem; # managed by Certbot
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
### Comment out the following lines there will be login Web UI with the SSO button. This is good for
testing and to login with the superuser created.
```
map $http_x_forwarded_proto $thescheme {
default $scheme;
https https;
}
server {
listen 80 default_server;
server_name netbox3.hungry-howard.com;
return 301 https://$host$request_uri;
}
{
listen 443 ssl;
server_name netbox3.hungry-howard.com;
ssl_certificate /etc/letsencrypt/live/netbox3.hungry-howard.com/fullchain.pem; # managed by Certbot
ssl_certificate_key /etc/letsencrypt/live/netbox3.hungry-howard.com/privkey.pem; # managed by Certbot
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
########### This section is for controlling auto redirect ###################
# location /login/ {
# proxy_pass http://127.0.0.1:8001/api/plugins/sso/login/;
# }
# location /sso/ {
# proxy_pass http://127.0.0.1:8001/api/plugins/sso/; # Must have a trailing slash to strip the original path
# }
}
```
Configure the setting.py file
Add the following to the bottom of the file.
```
SAML2_AUTH_CONFIG = {
'AUTHENTICATION_BACKEND': 'django.contrib.auth.backends.RemoteUserBackend',
'METADATA_AUTO_CONF_URL': "https://global-edjak2.zitadel.cloud/saml/v2/metadata"
}
```
Restart service
```
systemctl restart nginx
systemctl restart netbox
```
Conclusion:
NetBox configruation.py file should look something like this.
```
ALLOWED_HOSTS = ['netbox.hungry-howard.com']
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
CORS_ORIGIN_WHITELIST = [
]
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
LOGIN_REQUIRED = False
LOGIN_TIMEOUT = None
LOGOUT_REDIRECT_URL = 'home'
METRICS_ENABLED = False
PLUGINS = ['django3_saml2_nbplugin']
PLUGINS_CONFIG = {
'django3_saml2_nbplugin': {
'AUTHENTICATION_BACKEND': 'netbox.authentication.RemoteUserBackend',
'ASSERTION_URL': 'https://netbox.hungry-howard.com/',
'ENTITY_ID':'https://netbox.hungry-howard.com',
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
BANNER_LOGIN = '<a href="/api/plugins/sso/login" class="btn btn-primary btn-block">Login with SSO</a>'
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
### Nginx configuration file
```
map $http_x_forwarded_proto $thescheme {
default $scheme;
https https;
}
server {
listen 80 default_server;
server_name netbox.hungry-howard.com;
return 301 https://$host$request_uri;
}
server {
listen 443 ssl;
server_name netbox.hungry-howard.com;
ssl_certificate /etc/letsencrypt/live/netbox.hungry-howard.com/fullchain.pem; # managed by Certbot
ssl_certificate_key /etc/letsencrypt/live/netbox.hungry-howard.com/privkey.pem; # managed by Certbot
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
}
```
Settings.py file
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
'METADATA_AUTO_CONF_URL': "https://global-edjak2.zitadel.cloud/saml/v2/metadata"
}
```
### Zitadel XML file
```
<?xml version="1.0"?>
  <md:EntityDescriptor xmlns:md="urn:oasis:names:tc:SAML:2.0:metadata"
    entityID="https://netbox3.hungry-howard.com">
  <md:SPSSODescriptor AuthnRequestsSigned="false" WantAssertionsSigned="false"
    protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
      <md:SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://netbox3.hungry-howard.com/" />
      <md:NameIDFormat>urn:oasis:names:tc:SAML:2.0:nameid-format:unspecified</md:NameIDFormat>
  <md:AssertionConsumerService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST"Location="https://netbox3.hungry-howard.com/api/plugins/sso/acs/" index="1" />
    </md:SPSSODescriptor>
</md:EntityDescriptor>
```























