# mailman-on-aws
Way to deploy Mailman 3 on AWS (SES and EC2)

List of steps:

1. [Initialize EC2 Virtual Machine](00-EC2-init.md)

2. [Set Up SES, Simple Email Service](01-SES.md)

3. [Set up a testing Mailman](02-mailman-testing.md)

4. [Set Up RDS, Relational Database Service](03-rds.md)

5. [Put Mailman into Production](04-mailman-production.md)

6. [Upgrade packages](05-nginx-proxy.md)

7. [Use Nginx to proxy Mailman](06-nginx-proxy.md)

References:
* [Integrating Amazon SES with Postfix] (http://docs.aws.amazon.com/ses/latest/DeveloperGuide/postfix.html)
* [Mailman bundler - GNU Mailman, Postorius and HyperKitty](https://gitlab.com/mailman/mailman-bundler)
* [Mailman installation for production](https://wiki.list.org/DOC/Mailman%203%20installation%20experience)
