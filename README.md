# Deploy a Flask web app on an AWS Ubuntu server

Step by step instruction.

## Create an Instance

1. Login in to your AWS console and go the EC2 Dashboard.
2. Head over to Launch Instance and choose for Ubuntu Server.
3. You can leave all default settings and press on Review and Launch.
4. Create a Key Pair, download the Key Pair file and store it locally on
`~/.ssh/example_key.pem`

## Connect to the Instance by SSH

Open a terminal on your local computer.

Your key must not be publicly viewable for SSH to work. Use this command if
needed:

```bash
$ chmod 400 ~/.ssh/example_key.pem
```


```bash
$ ssh -i "~/.ssh/example_key.pem" ubuntu@ec0-00-00-000-000.eu-central-1.compute.amazonaws.com
```

Change to string `ubuntu@ec0-00-00-000-000.eu-central-1.compute.amazonaws.com`
the IP address of your Instance. You can find this IP address under Public DNS
(IPv4) at the EC2 Dashboard on the Instance tab.

## Connect to Github

In my case, I have a web application developed locally on my computer and made
it available on the Github. So when I can clone the repository on my instance,
I can deploy the web application on the Ubuntu server.

To clone a repository from the Github, you first need to setup up a SSH protocol
to your Github account.

```bash
$ ssh-keygen -t rsa -b 4096 -C "example@gmail.com"
```

Press enter a few times if you would like to continue with de the default
settings. After generating your rsa key pair, print your public key in the
console by the following command and copy the whole string.

```bash
$ cat .ssh/id_rsa.pub
```

Open your Github account, go to settings, click on SSH and GPG keys and click
New SSH key. Give the key a name and paste the copied string into the Key field.

Now you should be able to pull and push commits from your Github account within
your instance. To test the connect, enter the following command into your
console:

```bash
$ ssh -T git@github.com
```

## Setting up directories

Now it is time to setup the main directories for your web application.

First go to the highest folder of the instance:

```bash
$ ../..
```

And change directory to:

```bash
$ var/
```

Create a directory called `www`:

```bash
$ sudo mkdir www
```

Although it is first all the most important thing to get your web application
deployed, it is already a good idea to make a distinction between a production
and a development environment. If you would like to do this as well, type in
the following commands:

```bash
$ sudo mkdir dev
$ sudo mkdir prod
```

## Set correct permissions to directory

In order to deploy your web app in a save way to you instance, use the following
steps in order to set the right permissions.

Add yourself to the www-data group and set the setgid bit on the /var/www
directory such that all newly created files inherit this group as well.

```bash
$ sudo gpasswd -a "$USER" www-data
```

Set ownership of www directory to www-data and set correct write, read and
execute permission to this user.

```bash
$ sudo chown -R "$USER":www-data /var/www
$ find /var/www -type f -exec chmod 0660 {} \;
$ sudo find /var/www -type d -exec chmod 2770 {} \;
```

## Clone your web application

Now, move to the `dev` folder and clone your web application.

```bash
$ cd /var/www/dev
$ git clone git@github.com:<user-name>/<repository-name>.git
```

Move to the repository and create a file where logs can be stores.

```bash
$ cd repository-name
$ sudo mkdir logs
```

## Installing pip, python3 and virtualenv

Now you've cloned your web application, it is time to install some other
software in order to run the application properly.

```bash
$ sudo apt-get update
$ curl -O https://bootstrap.pypa.io/get-pip.py
$ sudo apt-get install python3-distutils
$ sudo python3 get-pip.py
$ sudo pip3 install virtualenv
$ sudo apt-get install libapache2-mod-wsgi-py3
$ sudo apt-get install apache2
```

## Setup virtual environment

Python3, pip3 and virtualenv are now available on your instance.

Change directory to your web application:

```bash
$ cd /var/www/dev/repository-name
```

Create a virtual environment for your web application and activate it:

```bash
$ virtualenv -p python3 venv
$ source venv/bin/activate
```

Install Flask:

```bash
$ pip3 install flask
```

## Debug


## Deploy the web application

With the other packages `libapache2-mod-wsgi-py3` and `apache2` (already
installed), we can deploy our web app.

Change directory to the following and create a `webapp.conf` file. It is
recommended to change the `webapp` part in the file name to your own domain
name, for example `example.com`. In this case your file name will be
`example.com.conf`:

```bash
$ cd /etc/apache2/sites-available
$ sudo nano example.com.conf
```

Paste in the following text to the `example.com.conf` file:

```bash
<VirtualHost *:80>

    ServerName example.com
    ServerAlias www.example.com
    ServerAdmin example@example.com

    WSGIScriptAlias / /var/www/dev/repository-name/web/app.wsgi
    DocumentRoot /var/www/dev/repository-name/web

    ErrorLog /var/www/dev/repository-name/logs/error.log
    CustomLog /var/www/dev/repository-name/logs/access.log combined

    Alias /static /var/www/dev/repository-name/web/static
    Alias /templates /var/www/dev/repository-name/web/templates

</VirtualHost>
```

Change content to your IP address, your repository name and your repository
directory.

Hit `control+X`, `y` and `enter` to save changes.

## Enable configuration

Enable your custom configuration settings with the following commands.

```bash
$ sudo a2ensite example.com.conf
$ sudo a2enmod wsgi
```

And at the same time, disable the apache2 default page.

```bash
$ sudo a2dissite 000-default.conf
```

## Test

Enter the following commands to check you syntax. If something in your conf file
is not correct, you will get this error(s) by an echo in your terminal.

## Start Server

When all configurations seems to be fine, it is time to start up your server!

```bash
$ sudo service apache2 start
```

Type `ec0-00-00-000-000.eu-central-1.compute.amazonaws.com` in your url on the
browser. You should see that your web application is deployed!

## From HTTP to HTTPS (optional)

If you want to encrypt your web application by using an SSL certificate,
then use the following steps. This steps will encrypt your web application with
a free SSL certificate by 'Let's Encrypt'.

Install some dependencies with the following commands.

```bash
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository universe
```

Add the repository `certbot` to your instance.

```bash
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
```

Install a SSL certificate.

```bash
$ sudo apt-get install --reinstall ca-certificates
$ sudo update-ca-certificates
```

Install certbot.

```bash
$ sudo apt-get install certbot python-certbot-apache
```

Set up the certificate for your domain.

```bash
sudo certbot --apache -d example.com
```

Just to best sure, restart the apache server.

```bash
$ sudo service apache2 restart
```

Enter your emailadress and answer some questions from certbot to continue.

Your web application is now secured!

Your SSL certificate is valid for 90 days. To automate renewal, enter the
following command.

```bash
$ sudo certbot renew --dry-run
```

If you would like to secure an extra alias of your domain, for example
www.example.com, while this one is not already redirected, you can do this by
editing your `example.com.conf` file.

## redirect Alias to secured later on.

When you had not filled in a ServerAlias (for example: www.example.com) in the
`example.com.conf` file, this alias will not automatticly redirect to HTTPS
secured domain when you add this alias later on. To create a correct redirect
for an Alias later on, follow the steps below.

Certbot has added some content in the `example.com.conf` file to redirect your
original domain to the secured one. This content will look like the following
example.

```bash
RewriteEngine on
RewriteCond %{SERVER_NAME} =example.com [OR]
RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
```

Add a `RewriteCond` for your domain alias so that the rewrite engine will look
like the second example.

```bash
$ sudo nano /etc/apache2/sites-available/example.com.conf
```

```bash
RewriteEngine on
RewriteCond %{SERVER_NAME} =example.com
RewriteCond %{SERVER_NAME} =www.example.com [OR]
RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
```

Now your alias will be redirected to the domain that is secured by an HTTPS
connection. Note: don't forget to add the ServerAlias `example.com.conf` as
well.

## Used sources

- https://www.jakowicz.com/flask-apache-wsgi/
- https://medium.com/@satyavinay456/deploying-flask-application-using-wsgi-server-with-apache2-on-aws-cloud-ubuntu-machine-b7a15ca25cff
- https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts
- https://www.inmotionhosting.com/support/website/git/how-to-add-ssh-keys-to-your-github-account
- https://askubuntu.com/questions/46331/how-to-avoid-using-sudo-when-working-in-var-www/46371#46371
- https://askubuntu.com/questions/629995/apache-not-able-to-restart

- https://www.youtube.com/watch?v=7Bo78eDEy7g
- https://certbot.eff.org/lets-encrypt/ubuntubionic-apache
- https://stackoverflow.com/questions/43308552/filenotfounderror-add-apt-repository-ppacertbot-certbot
