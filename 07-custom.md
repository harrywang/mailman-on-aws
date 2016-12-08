
### Update email templates

To replace default email templates, generate custom templates in /opt/mailman/mailman-bundler/var/templates/site/en/ as follows:

list:member:generic:footer.txt

```
_______________________________________________
$display_name mailing list
$listname
https://[DOMAIN]/mailman3/lists/$list_id/
list:user:notice:welcome.txt
```

list:user:notice:welcome.txt

```
To post to this list, send your email to:

  $listname

General information about the mailing list is at:

  https://[DOMAIN]/mailman3/lists/$list_id/

If you ever want to unsubscribe or change your options (eg, switch to or
from digest mode, change your password, etc.), visit your subscription
page at:

  https://[DOMAIN]/mailman3/accounts/mailmansettings/

```
Replace [DOMAIN] with your domain name.

