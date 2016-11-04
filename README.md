# mailman-on-aws
Way to deploy Mailman 3 on AWS (SES and EC2)

Reference:
* [Integrating Amazon SES with Postfix] (http://docs.aws.amazon.com/ses/latest/DeveloperGuide/postfix.html)
* [Mailman bundler - GNU Mailman, Postorius and HyperKitty](https://gitlab.com/mailman/mailman-bundler)
* [Mailman installation for production](https://wiki.list.org/DOC/Mailman%203%20installation%20experience)

List of article:
- [Initialize EC2 Virtual Machine](00-EC2-init.md)
- [Set Up SES, Simple Email Service](01-SES.md)
- [Set up a testing Mailman](02-mailman-testing.md)
- [Set Up RDS, Relational Database Service](03-rds.md)
- [Put Mailman into Production](04-mailman-production.md)
- [Use Nginx to proxy Mailma](05-nginx-proxy.md)
