# Install Mailman Bundler

Reference: https://gitlab.com/mailman/mailman-bundler

## Install all necessary packages

For Ubuntu or Debian users, run

    sudo apt-get install ruby-sass python python3 python-pip python3-pip git python-virtualenv libpq-dev python-dev python3-dev gcc
    
Centos:

    sudo yum install rubygem-sass python python3 python-pip python3-pip git python-virtualenv python-devel python3-devel gcc

## Setting up for testing first

Prepare an empty folder for mailman:

    sudo mkdir /opt/mailman
    cd /opt/mailman
    
Install Mailman, HyperKitty, Postorius, MailmanClient, HyperKitty - Mailman plugin, django-mailman3 separately.

    sudo mkdir git
    cd git
    sudo git clone https://gitlab.com/alex1007/mailman.git
    sudo git clone https://gitlab.com/alex1007/hyperkitty.git
    sudo git clone https://gitlab.com/alex1007/postorius.git
    sudo git clone https://gitlab.com/alex1007/mailmanclient.git
    sudo git clone https://gitlab.com/alex1007/django-mailman3.git
    sudo git clone https://gitlab.com/alex1007/mailman-hyperkitty.git
    
Enter each repo to install the package

    cd XXX/
    python3.4 setup.py develop # for mailman, mailman-hyperkitty
or

    python setup.py develop # for hyperkitty, postorius, mailmanclient, django-mailman3
    
If you need to install python3.4, go check [Building Python 3.4 from source](http://devmartin.com/blog/2016/04/creating-a-virtual-environment-with-python3.4-on-ubuntu-16.04-xenial-xerus/)
    
Clone [Mailman Bundler](https://gitlab.com/mailman/mailman-bundler):
    
    sudo git clone https://gitlab.com/alex1007/mailman-bundler.git

Now switch to a dedicated Mailman user.

    sudo useradd mailman --no-user-group --create-home
    sudo usermod mailman --append --groups sudo
    sudo chown mailman ./ -R
    sudo su mailman

We choose not to install virtual environment.
    
In the bundler directory, open the mailman_web/testing.py file, look for the SECRET_KEY parameter and set something random.
    
    SECRET_KEY = '[SET IT TO SOMETHING RANDOM]'

Go into mailman-bundler folder, install and run buildout:

    cd mailman-bundler
    pip install zc.buildout
    buildout
    
Edit every script in `/mailman-bundler/bin/`. Remove `/opt/mailman/mailman-bundler/eggs/Django-1.8.16-py2.7.egg` and `/opt/mailman/mailman-bundler/eggs/postorius-1.0.3-py2.7.egg` in the path. Update path of mailman in `mailman` script(you probably can find your excutable mailman in `/usr/local/bin/`).

Now initialize Django's database:

    ./bin/mailman-post-update

Now create an initial superuser to login as, this is the user you'll use to login to Mailman's web interface (Postorius):
  
    ./bin/mailman-web-django-admin createsuperuser

Install falcon 0.3 if version of your current package is < 0.3 Note: [Postorius error if you don't update falcon](https://gitlab.com/mailman/postorius/issues/122)

    source /opt/mailman/mailman-bundler/venv-3.4/bin/activate
    pip install --upgrade falcon==0.3
    
## Update configuration files
    
update `mailman-bundler/mailman_web/urls.py`.
```
# mailman-bundler/mailman_web/urls.py

# Changes are converting url patterns to a list for Django 1.9 and
# removing '{"SSL": True\} from a couple of URLs as it caused problems.

from django.conf.urls import include, url
from django.core.urlresolvers import reverse_lazy
from django.views.generic import RedirectView

# Comment the next two lines to disable the admin:
from django.contrib import admin
admin.autodiscover()

urlpatterns = [url(r'^$', RedirectView.as_view(url=reverse_lazy('hyperkitty.views.index.index'))),
               url(r'^mailman3/', include('postorius.urls')),
               url(r'^archives/', include('hyperkitty.urls')),
               url(r'', include('social.apps.django_app.urls', namespace='social')),
               url(r'', include('django_browserid.urls')),
              ]
```

update `mailman-bundler/deployment/mailman.cfg`
```
# mailman-bundler/deployment/mailman.cfg 

# changes are site_owner, [database] and [shell]

# This is the absolute bare minimum base configuration file.  User supplied
# configurations are pushed onto this.

[mailman]
# This address is the "site owner" address.  Certain messages which must be
# delivered to a human, but which can't be delivered to a list owner (e.g. a
# bounce from a list owner), will be sent to this address.  It should point to
# a human.
site_owner: mailman@mailman3.org
layout: here

[paths.here]
# Everything in the same directory
var_dir: /opt/mailman/mailman-bundler/var

[database]
class: mailman.database.postgresql.PostgreSQLDatabase
#url: postgres://mailman:mailman-db-password@localhost/mailman
url: postgres://mailman:db_passwd@localhost/mailmanweb

[archiver.hyperkitty]
class: mailman_hyperkitty.Archiver
enable: yes
configuration: /opt/mailman/mailman-bundler/deployment/mailman-hyperkitty.cfg

[archiver.prototype]
enable: yes

[shell]
history_file: $var_dir/history.py

#[logging.database]
#level: debug
```

update `mailman-bundler/deployment/mailman-hyperkitty.cfg`
```
# mailman-bundler/deployment/mailman-hyperkitty.cfg

# changes are base_url and api_key

# This is the mailman extension configuration file to enable HyperKitty as an
# archiver. Remember to add the following lines in the mailman.cfg file:
#
# [archiver.hyperkitty]
# class: mailman_hyperkitty.Archiver
# enable: yes
# configuration: /path/to/here/hyperkitty.cfg
#

[general]
base_url: https://lists.mailman3.org/archives
api_key: arch_secret
```

## Configure Postfix

The deployment/postfix-main.cf contains a few lines that you must add to your main Postfix configuration file (usually /etc/postfix/main.cf).

Exit mailman user and return to default user first.

    exit
    cat mailman-bundler/deployment/postfix-main.cf # Copy all lines shown

The return should look like

    # Support the default VERP delimiter.
    recipient_delimiter = +
    unknown_local_recipient_reject_code = 550
    owner_request_special = no
    # Maps
    transport_maps =
        hash:/opt/mailman/mailman-bundler/var/data/postfix_lmtp
    local_recipient_maps =
        hash:/opt/mailman/mailman-bundler/var/data/postfix_lmtp
    relay_domains =
        hash:/opt/mailman/mailman-bundler/var/data/postfix_domains

Copy them. Then paste it to the postfix config file:

    sudo nano /etc/postfix/main.cf
    sudo postfix reload
    
Make sure postfix has read access to the *.db files in var/data (it won't have access by default). You can change the permissions of those files to world-readable:
    
    sudo chmod 777 /opt/mailman/mailman-bundler/var/data/

## Start your test server

Change back to mailman user:

    sudo su mailman

Start mailman:

    cd mailman-bundler/
    ./bin/mailman start

Run Django (the web interfaces server):

    ./bin/mailman-web-django-admin runserver 0.0.0.0:8000

Now, the web interface are available at:
- http://[IP OR DOMAIN}:8000/mailman3 for Postorius
- http://[IP OR DOMAIN}:8000/archives for HyperKitty

You should be able to see something but no error. Do some test here.

## Test your server

### Create a domain

- Open your browser and navigate to Postorius (http://127.0.0.1:8000/mailman3)
- Go to Settings -> "New domain"
- Enter a mail host: lists.example.com
- Enter a web host: http://lists.example.com

### Create a mailing-list

- In Postorius, go to "Lists" and click "New list"
- Enter list name, e.g. test
- Choose the mailhost
- Define the list owner (defaults to domain contact)
- Choose whether to advertise list
- Enter description
- Save list.

On the lists' front page, add a moderator in the corresponding category.

### Subscribe to your mailing-list

To subscribe, send an email to test-request@lists.example.com with subject subscribe.
To unsubscribe, send an email to test-request@lists.example.com with subject unsubscribe.
