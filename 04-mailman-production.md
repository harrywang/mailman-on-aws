# Put Mailman into Production

## Install PostgreSQL

In this instance, we use PostgreSQL to store data from Mailman.

We have already set up our RDS server, let's review it.

    Endpoint: mailman-postgres.abcd.ap-northeast-1.rds.amazonaws.com:5432
    Username: mailman
    Password: [mailman-pw]
    DB Name: mailmandb
    
First, we need to install libpq-dev in your system using default user. Run 

    sudo apt-get install libpq-dev 

Then, change back to mailman user and activate the virtual environment.
    
    sudo su mailman
    cd /opt/mailman
    
Install PostgreSQL library of python

    pip install psycopg2
    pip3 install psycopg2
    
## Change status to production

For a production setup, you first need to change the deployment parameter to production in buildout.cfg and run buildout again. It will regenerate the scripts in bin and the contents of the deployment directory. Run

    nano mailman-bundler/buildout.cfg
    # Replace "deployment = testing" with "deployment = production"
    cd mailman-bundler
    buildout
    
In the bundler directory, open the mailman_web/production.py file, look for the SECRET_KEY parameter and set something random.

    nano mailman_web/production.py
    # set SECRET_KEY parameter to something random.
    # here, you may also want to do some modification. Check the things you will need to change here: https://docs.djangoproject.com/en/1.9/ref/settings/
    
## Set Database

### Mailman

Run:

    nano deployment/mailman.cfg 
    
Uncomment the following lines (delete the # character in the beginning):
    
    #[database]
    #class: mailman.database.postgresql.PostgreSQLDatabase
    #url: postgres://mailman:mailman-db-password@localhost/mailman
    
and replace them with

    [database]
    class: mailman.database.postgresql.PostgreSQLDatabase
    url: postgres://[DB USERNAME]:[DB PASSWORD]@[ENDPOINT]/[DB NAME]

Save.

### Mailman-web

Create `mailman_web/settings_local.py`. Fill domain name in ALLOWED_HOSTS and BROWSERID_AUDIENCES. Update the database information.

```
# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = 'elided'

MAILMAN_ARCHIVER_KEY = 'elided'

# The SITE_ID is the 1-based index of the Django site.  It's normally 1
# but if you have, or had in the past, multiple sites, it may be different.
SITE_ID = 1

ADMINS = (
     ('Mailman Admin', 'mailman@localhost'),
)

# Hosts/domain names that are valid for this site; required if DEBUG is False
# See https://docs.djangoproject.com/en/1.9/ref/settings/#allowed-hosts
#ALLOWED_HOSTS = ['*']
ALLOWED_HOSTS = ['localhost',
                 '127.0.0.1',
                ]

# And for BrowserID too, see
# http://django-browserid.rtfd.org/page/user/settings.html#django.conf.settings.BROWSERID_AUDIENCES
BROWSERID_AUDIENCES = [ "http://localhost",
                        "http://localhost:8000",
                        "http://127.0.0.1:8000",
                      ]


## Email confirmation / address activation
# Add a from-address for email-confirmation:
# EMAIL_CONFIRMATION_FROM = 'postmaster@example.org'
EMAIL_CONFIRMATION_FROM = 'mailman@mailman3.org'

MAILMAN_ARCHIVER_FROM = ('127.0.0.1',
                         '::1',
                         '::ffff:127.0.0.1',
                         '104.239.228.201',
                        )

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2', # Last part is one of 'postgresql_psycopg2', 'mysql', 'sqlite3' or 'oracle'.
        'NAME': 'mailmanweb',  # Example, change as needed
        'USER': 'mailman',
        'PASSWORD': 'elided',
        'HOST': '127.0.0.1',   # Empty for localhost through domain sockets or '127.0.0.1' for localhost through TCP.
        'PORT': '5432',            # Set to empty string for default.
    }
}

# If you're behind a proxy, use the X-Forwarded-Host header
# See https://docs.djangoproject.com/en/1.8/ref/settings/#use-x-forwarded-host
USE_X_FORWARDED_HOST = True
# And if your proxy does your SSL encoding for you, set SECURE_PROXY_SSL_HEADER
# see https://docs.djangoproject.com/en/1.5/ref/settings/#secure-proxy-ssl-header
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')

# Use SSL when logged in. You need to enable the SSLRedirect middleware for
# this feature to work.
USE_SSL = False

ACCOUNT_DEFAULT_HTTP_PROTOCOL = "http"

TIME_ZONE = 'UTC'

SOCIALACCOUNT_PROVIDERS = {
    #'openid': {
    #    'SERVERS': [
    #        dict(id='yahoo',
    #             name='Yahoo',
    #             openid_url='http://me.yahoo.com'),
    #    ],
    #},
    'google': {
        'SCOPE': ['profile', 'email'],
        'AUTH_PARAMS': {'access_type': 'online'},
    },
    'facebook': {
       'METHOD': 'oauth2',
       'SCOPE': ['email'],
       'FIELDS': [
           'email',
           'name',
           'first_name',
           'last_name',
           'locale',
           'timezone',
           ],
       'VERSION': 'v2.4',
    },
}

# From Address for emails sent to users
DEFAULT_FROM_EMAIL = 'mailman@mailman3.org'
# From Address for emails sent to admins
SERVER_EMAIL = 'mailman@mailman3.org'
# Compatibility with Bootstrap 3
from django.contrib.messages import constants as messages
MESSAGE_TAGS = {
    messages.ERROR: 'danger'
}

```

## Make Folders for cache

Go back to default user.
    
    exit
    sudo mkdir /var/spool
    sudo mkdir /var/spool/mailman-web
    sudo mkdir /var/spool/mailman-web/static
    sudo chmod 777 /var/spool/mailman-web/ -R
    sudo chown mailman /var/spool/mailman-web -R
    sudo mkdir /var/log/mailman-web/
    sudo chmod 777 /var/log/mailman-web/


## Other settings

There are some other variables that need to be set in production.py for production serving. You must set the **hostname** your website will respond to in **ALLOWED_HOSTS** and **BROWSERID_AUDIENCES**.

Also, you need to append **USE_X_FORWARDED_HOST = True ** to the production.py file.

## Reboot mailman

Run:

    sudo su mailman
    cd mailman-bundler
    ./bin/mailman-post-update

## Running web interface on Gunicorn

To add Gunicorn support, just run the following command:

    buildout install gunicorn
    
You have to edit scripts in `mailman-bundler/bin/` again, just like what you did last time.

Create a superuser here:

    ./bin/mailman-web-django-admin createsuperuser

You can then serve the Mailman web interfaces with Gunicorn by running the following command:

    ./bin/gunicorn mailman_web.wsgi:application -b 0.0.0.0:8000

No img and css now -- it is normal. 

## Start when Boot

You need to run as root or use sudo. So go back to default user. Run:
    
    exit
    sudo nano /etc/rc.local

Add the following lines before exit 0:

    su -c "/opt/mailman/mailman-bundler/bin/mailman start &" mailman
    su -c "/opt/mailman/mailman-bundler/bin/gunicorn -c /opt/mailman/mailman-bundler/deployment/gunicorn.conf mailman_web.wsgi:application &" mailman

Now you are done. Go to configure nginx.
