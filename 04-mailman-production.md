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

    vim mailman-bundler/buildout.cfg
    # Replace "deployment = testing" with "deployment = production"
    cd mailman-bundler
    buildout
    
You have to edit scripts in `mailman-bundler/bin/` again, just like what you did last time.

Also, update mailman and mailman-hyperkitty in virtual environment
     
     source /opt/mailman/mailman-bundler/venv-3.4/bin/activate
     pip3 install --upgrade -e /opt/mailman/git/mailman-hyperkitty/
     pip3 install --upgrade -e /opt/mailman/git/mailman/

In the bundler directory, open the mailman_web/production.py file, look for the SECRET_KEY parameter and set something random.

```
    #-*- coding: utf-8 -*-
"""
Django settings for HyperKitty + Postorius
"""

import os
BASE_DIR = os.path.dirname(os.path.abspath(__file__))
VAR_DIR = "/var/spool"

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = 'change-that-at-install-time'

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = False

ADMINS = (
     ('Mailman Admin', 'root@localhost'),
)

# Hosts/domain names that are valid for this site; required if DEBUG is False
# See https://docs.djangoproject.com/en/1.5/ref/settings/#allowed-hosts
ALLOWED_HOSTS = ["localhost"]
# And for BrowserID too, see
# http://django-browserid.rtfd.org/page/user/settings.html#django.conf.settings.BROWSERID_AUDIENCES
BROWSERID_AUDIENCES = [ "http://localhost", "http://localhost:8000" ]

# Mailman API credentials
MAILMAN_REST_API_URL = 'http://localhost:8001'
MAILMAN_REST_API_USER = 'restadmin'
MAILMAN_REST_API_PASS = 'restpass'
MAILMAN_ARCHIVER_KEY = 'SecretArchiverAPIKey'
MAILMAN_ARCHIVER_FROM = ('127.0.0.1', '::1', '::ffff:127.0.0.1')

# Application definition

INSTALLED_APPS = (
    'hyperkitty',
    'postorius',
    'django_mailman3',
    'django.contrib.admin',
    # 'django.contrib.admindocs',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.sites',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'django_gravatar',
    'paintstore',
    'compressor',
    'haystack',
    'django_extensions',
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
    'allauth.socialaccount.providers.facebook',
    'allauth.socialaccount.providers.github',
    'allauth.socialaccount.providers.gitlab',
    'allauth.socialaccount.providers.google',
    #'allauth.socialaccount.providers.openid',
    #'allauth.socialaccount.providers.stackexchange',
    #'allauth.socialaccount.providers.twitter',
    #'django_mailman3.lib.auth.fedora',
)


MIDDLEWARE_CLASSES = (
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.middleware.locale.LocaleMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.auth.middleware.SessionAuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'django.middleware.security.SecurityMiddleware',
    'django_mailman3.middleware.TimezoneMiddleware',
    'postorius.middleware.PostoriusMiddleware',
)

ROOT_URLCONF = 'mailman_web.urls'


#
# Postorius
#

## Email confirmation / address activation
# Add a from-address for email-confirmation:
# EMAIL_CONFIRMATION_FROM = 'postmaster@example.org'

# These can be set to override the defaults but are not mandatory:
# EMAIL_CONFIRMATION_TEMPLATE = 'postorius/address_confirmation_message.txt'
# EMAIL_CONFIRMATION_SUBJECT = 'Confirmation needed'



# Database
# https://docs.djangoproject.com/en/1.6/ref/settings/#databases

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2', # Last part is one of 'postgresql_psycopg2', 'mysql', 'sqlite3' or 'oracle'.
        'NAME': 'mailmanweb',  # Example, change as needed
        'USER': 'mailmanweb',  # Example, change as needed
        'PASSWORD': 'change-this-password',  # Example, obviously
        'HOST': '127.0.0.1',   # Empty for localhost through domain sockets or '127.0.0.1' for localhost through TCP.
        'PORT': '',            # Set to empty string for default.
    }
}


# If you're behind a proxy, use the X-Forwarded-Host header
# See https://docs.djangoproject.com/en/1.5/ref/settings/#use-x-forwarded-host
#USE_X_FORWARDED_HOST = True
# And if your proxy does your SSL encoding for you, set SECURE_PROXY_SSL_HEADER
# see https://docs.djangoproject.com/en/1.5/ref/settings/#secure-proxy-ssl-header
#SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')

# Internationalization
# https://docs.djangoproject.com/en/1.6/topics/i18n/

LANGUAGE_CODE = 'en-us'

TIME_ZONE = 'America/Chicago'

USE_I18N = True

USE_L10N = True

USE_TZ = True


# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/1.6/howto/static-files/

# Absolute filesystem path to the directory that will hold user-uploaded files.
# Example: "/var/www/example.com/media/"
MEDIA_ROOT = ''

# URL that handles the media served from MEDIA_ROOT. Make sure to use a
# trailing slash.
# Examples: "http://example.com/media/", "http://media.example.com/"
MEDIA_URL = ''

# Absolute path to the directory static files should be collected to.
# Don't put anything in this directory yourself; store your static files
# in apps' "static/" subdirectories and in STATICFILES_DIRS.
# Example: "/var/www/example.com/static/"
#STATIC_ROOT = ''
STATIC_ROOT = os.path.join(VAR_DIR, "mailman-web", "static")

# URL prefix for static files.
# Example: "http://example.com/static/", "http://static.example.com/"
STATIC_URL = '/static/'

# Additional locations of static files
STATICFILES_DIRS = (
    # Put strings here, like "/home/html/static" or "C:/www/django/static".
    # Always use forward slashes, even on Windows.
    # Don't forget to use absolute paths, not relative paths.
)

# List of finder classes that know how to find static files in
# various locations.
STATICFILES_FINDERS = (
    'django.contrib.staticfiles.finders.FileSystemFinder',
    'django.contrib.staticfiles.finders.AppDirectoriesFinder',
#    'django.contrib.staticfiles.finders.DefaultStorageFinder',
    'compressor.finders.CompressorFinder',
)


TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.i18n',
                'django.template.context_processors.media',
                'django.template.context_processors.static',
                'django.template.context_processors.tz',
                'django.template.context_processors.csrf',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
                'django_mailman3.context_processors.common',
                'hyperkitty.context_processors.common',
                'postorius.context_processors.postorius',
            ],
        },
    },
]


# Django 1.6+ defaults to a JSON serializer, but it won't work with django-openid, see
# https://bugs.launchpad.net/django-openid-auth/+bug/1252826
SESSION_SERIALIZER = 'django.contrib.sessions.serializers.PickleSerializer'


LOGIN_URL = 'account_login'
LOGIN_REDIRECT_URL = 'hk_root'
LOGOUT_URL = 'account_logout'

# Use the email username as identifier, but truncate it because
# the User.username field is only 30 chars long.
def username(email):
    return email.rsplit('@', 1)[0][:30]
BROWSERID_USERNAME_ALGO = username
BROWSERID_VERIFY_CLASS = "django_browserid.views.Verify"

# Password validation
# https://docs.djangoproject.com/en/1.9/ref/settings/#auth-password-validators

AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]


#
# Social auth
#

AUTHENTICATION_BACKENDS = (
    'django.contrib.auth.backends.ModelBackend',
    'allauth.account.auth_backends.AuthenticationBackend',
)

# Django Allauth
ACCOUNT_AUTHENTICATION_METHOD = "username_email"
ACCOUNT_EMAIL_REQUIRED = True
ACCOUNT_EMAIL_VERIFICATION = "mandatory"
ACCOUNT_DEFAULT_HTTP_PROTOCOL = "https"
ACCOUNT_UNIQUE_EMAIL  = True

SOCIALACCOUNT_PROVIDERS = {
    'openid': {
        'SERVERS': [
            dict(id='yahoo',
                 name='Yahoo',
                 openid_url='http://me.yahoo.com'),
        ],
    },
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



#
# Gravatar
# https://github.com/twaddington/django-gravatar
#
# Gravatar base url.
#GRAVATAR_URL = 'http://cdn.libravatar.org/'
# Gravatar base secure https url.
#GRAVATAR_SECURE_URL = 'https://seccdn.libravatar.org/'
# Gravatar size in pixels.
#GRAVATAR_DEFAULT_SIZE = '80'
# An image url or one of the following: 'mm', 'identicon', 'monsterid', 'wavatar', 'retro'.
#GRAVATAR_DEFAULT_IMAGE = 'mm'
# One of the following: 'g', 'pg', 'r', 'x'.
#GRAVATAR_DEFAULT_RATING = 'g'
# True to use https by default, False for plain http.
#GRAVATAR_DEFAULT_SECURE = True

#
# django-compressor
# https://pypi.python.org/pypi/django_compressor
#
COMPRESS_PRECOMPILERS = (
   ('text/less', 'lessc {infile} {outfile}'),
   ('text/x-scss', 'sass {infile} {outfile}'),
   ('text/x-sass', 'sass {infile} {outfile}'),
)
COMPRESS_OFFLINE = True
# needed for debug mode
#INTERNAL_IPS = ('127.0.0.1',)

# Django Crispy Forms
#CRISPY_TEMPLATE_PACK = 'bootstrap3'
#CRISPY_FAIL_SILENTLY = not DEBUG

# Compatibility with Bootstrap 3
from django.contrib.messages import constants as messages
MESSAGE_TAGS = {
    messages.ERROR: 'danger'
}


#
# Full-text search engine
#
HAYSTACK_CONNECTIONS = {
    'default': {
        'ENGINE': 'haystack.backends.whoosh_backend.WhooshEngine',
        'PATH': os.path.join(VAR_DIR, "mailman-web", "fulltext_index"),
    },
}


# A sample logging configuration. The only tangible logging
# performed by this configuration is to send an email to
# the site admins on every HTTP 500 error when DEBUG=False.
# See http://docs.djangoproject.com/en/dev/topics/logging for
# more details on how to customize your logging configuration.
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'filters': {
        'require_debug_false': {
            '()': 'django.utils.log.RequireDebugFalse'
        }
    },
    'handlers': {
        'mail_admins': {
            'level': 'ERROR',
            'filters': ['require_debug_false'],
            'class': 'django.utils.log.AdminEmailHandler'
        },
        'file':{
            'level': 'INFO',
            #'class': 'logging.handlers.RotatingFileHandler',
            'class': 'logging.handlers.WatchedFileHandler',
            'filename': '/var/log/mailman-web/mailman-web.log',
            'formatter': 'verbose',
        },
    },
    'loggers': {
        #'django.request': {
        #    'handlers': ['mail_admins'],
        #    'level': 'ERROR',
        #    'propagate': True,
        #},
        'django.request': {
            'handlers': ['file'],
            'level': 'ERROR',
            'propagate': True,
        },
        'django': {
            'handlers': ['file'],
            'level': 'ERROR',
            'propagate': True,
        },
        'hyperkitty': {
            'handlers': ['file'],
            'level': 'INFO',
            'propagate': True,
        },
    },
    'formatters': {
        'verbose': {
            'format': '%(levelname)s %(asctime)s %(module)s %(process)d %(thread)d %(message)s'
        },
        'simple': {
            'format': '%(levelname)s %(message)s'
        },
    },
    'root': {
        'handlers': ['file'],
        'level': 'INFO',
    },
}


## Cache: use the local memcached server
#CACHES = {
#    'default': {
#        'BACKEND': 'django.core.cache.backends.memcached.PyLibMCCache',
#        'LOCATION': '127.0.0.1:11211',
#    }
#}



#
# HyperKitty-specific
#

APP_NAME = 'Mailing-list archives'

# Allow authentication with the internal user database?
# By default, only a login through Persona or your email provider is allowed.
USE_INTERNAL_AUTH = False

# Use SSL when logged in. You need to enable the SSLRedirect middleware for
# this feature to work.
#USE_SSL = True

# Only display mailing-lists from the same virtual host as the webserver
FILTER_VHOST = False

# This is for development purposes
USE_MOCKUPS = False


try:
    from settings_local import *
except ImportError:
    pass
```   
https://docs.djangoproject.com/en/1.9/ref/settings/

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
