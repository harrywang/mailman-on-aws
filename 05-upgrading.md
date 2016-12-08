# Upgrade some components to the newest version

Mailman-bundler is a convenient tool -- but some of the shipped components are out-of-date. We need to upgrade them.

Stop all gunicorn process first.
  
    sudo killall gunicorn
    
Install Mailman, HyperKitty, Postorius, MailmanClient, HyperKitty - Mailman plugin, django-mailman3 separately.

    sudo su mailman
    cd /opt/mailman
    mkdir git
    cd git
    git clone https://gitlab.com/alex1007/mailman.git
    git clone https://gitlab.com/alex1007/hyperkitty.git
    git clone https://gitlab.com/alex1007/postorius.git
    git clone https://gitlab.com/alex1007/mailmanclient.git
    git clone https://gitlab.com/alex1007/django-mailman3.git
    git clone https://gitlab.com/alex1007/mailman-hyperkitty.git
    
    pip install -e /opt/mailman/git/postorius
    pip install -e /opt/mailman/git/hyperkitty
    pip install -e /opt/mailman/git/mailmanclient
    pip install -e /opt/mailman/git/django-mailman3
    
    source mailman-bundler/venv-3.4/bin/activate
    pip3 install --upgrade -e /opt/mailman/git/mailman-hyperkitty/
    pip3 install --upgrade -e /opt/mailman/git/mailman/
    pip3 install psycopg2
    
Check every file in folder `/opt/mailman/mailman-bundler/bin`.

add `'/opt/mailman/venv/lib/python2.7/site-packages'` into the sys.path[0:0] list if it's not in it.
comment out `'/opt/mailman/mailman-bundler/eggs/Django-1.8.16-py2.7.egg`, `'/opt/mailman/mailman-bundler/eggs/postorius-1.0.3-py2.7.egg'`,`'/opt/mailman/mailman-bundler/eggs/HyperKitty-1.0.3-py2.7.egg'`,`'/opt/mailman/mailman-bundler/eggs/mailmanclient-1.0.1-py2.7.egg'`.

Then give run permission to it.

    chmod 777 /opt/mailman/mailman-bundler/bin/gunicorn /opt/mailman/mailman-bundler/bin/mailman-web-django-admin 
    chmod -R 7755 /opt/mailman/venv/lib/python2.7/site-packages/ /opt/mailman/venv/local/lib/python2.7/site-packages/

Upgrade the database and cache

    /opt/mailman/mailman-bundler/bin/mailman-web-django-admin makemigrations
    /opt/mailman/mailman-bundler/bin/mailman-web-django-admin migrate
    /opt/mailman/mailman-bundler/bin/mailman-web-django-admin collectstatic # press enter to continue

Fix some permission error

    exit
    sudo chown -R mailman:users /opt/mailman/mailman-bundler/var/mailman-web/
    sudo su mailman
    
Restart gunicorn

    /opt/mailman/mailman-bundler/bin/gunicorn -c /opt/mailman/mailman-bundler/deployment/gunicorn.conf mailman_web.wsgi:application &
