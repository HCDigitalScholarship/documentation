**WARNING:** This tutorial is incomplete! You will have to figure some things out for yourself. When you do, please edit the tutorial to include that information.



In this tutorial, we will install all of the needed requirements for a LEMP stack (Linux, Nginx, MySQL, and PHP or Python). This will all take place on a virtual machine running Ubuntu 14.04 created for us by [IITS](mailto:prodesk@haverford.edu). 

The tutorial assumes basic knowledge of the Unix command line. Note that wherever you see text in `<angle brackets>`, you are supposed to substitute it (including the brackets) with your own value. For example, if you see `ssh <username>@haverford.edu`, you might write `ssh kbenston@haverford.edu` instead. If you need a refresher on common Unix commands, see Appendix B. 

Whenever there is a command, you should run, it will be set on its own line and preceded by a dollar sign; don't type the dollar sign. So if you see

```
$ echo "Hello, world"
```

You should type `echo "Hello, world"` (again, without the dollar sign) in your terminal and press enter.

This tutorial is structured as follows:

1. Log onto the server and do basic setup
2. Install and configure Nginx
3. Install your database software (MySQL or PostgreSQL)
4. Install your programming language (PHP or Python)
5. Clone your app
6. Install and configure uWSGI (if you're using Python)

If you get stuck at any part of this tutorial, check out the additional resources in Appendix A.

## Step 1: Basic setup

The first step is to log onto the server. Your information should have been provided to you by one of the DS librarians.

```
$ ssh <username>@<server name>
```

This will open a remote session with the server. When you're done, type `exit` or press Control+D.

Next, we want to make sure all the packages on the server are up to date. `apt-get` is the command for updating and installing new software. You'll be using it a lot in this tutorial.

```
$ sudo apt-get update
```

If you want to change the password that the librarians gave you, type

```
$ passwd
```

You'll be prompted to enter your old password and then a new one.

## Step 2: Install Nginx

Nginx is the server software that you'll be using. It can be installed with

```
$ sudo apt-get install nginx
```

Now visit the IP address or hostname of your project in your browser. You should see an Nginx welcome page. If you don't try, running

```
$ sudo /etc/init.d/nginx start
```

## Step 3: Install MySQL or PostgreSQL

After your server software is installed, you'll need a database to manage persistent data for your website. Two popular options are MySQL and PostgreSQL. If you're unsure about which database your website is using, ask a DS librarian.

### MySQL

MySQL can be installed with:

```
$ sudo apt-get install mysql-server
```

This will prompt you to create a root password. Choose whatever you like, but make sure to write it down somewhere. After installation, you'll need to run some configuration scripts.

```
$ sudo mysql_install_db
$ sudo mysql_secure_installation
```

The second script will give you several prompts to remove unsafe defaults: press enter for all of them.

Your MySQL database is now set up!

### PostgreSQL

Install PostgreSQL with

```
$ sudo apt-get install postgresql postgresql-contrib
```

Some setup needs to be done. Type

```
$ sudo -u postgres psql postgres
```

This should open up a new prompt, something like `postgres=#`. Type `\password postgres` to set the main Postgres password. Make sure to write it down somewhere.

You should make a Postgres account before you create your database. The username should reflect the project, so for example the database user for a project Foo might be called `foodb`. Still at the Postgres prompt, type (make sure to put single quotes around the password, and a semicolon at the end of the line)

```
$ CREATE USER <some descriptive name> WITH PASSWORD '<password>';
$ CREATE DATABASE <your database name>;
$ GRANT ALL PRIVILEGES ON DATABASE <your database name> TO <the username you created>;
```

Finally, type `\q` to exit.

You'll probably need to do some additional configuration to get Postgres to communicate with your application. Check out [the Postgres documentation](https://help.ubuntu.com/community/PostgreSQL) for more details. If you're using Python, you probably need the `psycopg2` package. This will be installed in step 5, but take a look at [this Stack Overflow post](https://stackoverflow.com/questions/5420789/how-to-install-psycopg2-with-pip-on-python) if you get any installation errors.

## Step 4: Install PHP or Python

### Python

If you're using Python you should be good to go—Python comes preinstalled on Ubuntu 14.04 systems. You will most likely need to install either the `python-dev` or `python3-dev` package, depending on which version of Python your website uses. Do

```
$ sudo apt-get install python-dev
OR, for Python 3
$ sudo apt-get install python3-dev
```

You will also need pip, the Python package manager.

```
$ sudo apt-get install python-pip
OR, for Python 3
$ sudo apt-get install python3-pip
```

### PHP

PHP is an open source web scripting language that is widely use to build dynamic webpages. You only need to install PHP if you're using Omeka, Drupal or another PHP-bases content management system. Otherwise, there's no need and you can skip to the next step.

To install PHP, open terminal and type in this command.

```
$ sudo apt-get install php5-fpm
```

Next, we need to make one small change in the php configuration. Open up `php.ini`:

```
$ sudo vim /etc/php5/fpm/php.ini
```

Find the line, `cgi.fix_pathinfo=1`, and change the 1 to 0.

If this number is kept as 1, the PHP interpreter will do its best to process the file that is as near to the requested file as possible. This is a possible security risk. If this number is set to 0, conversely, the interpreter will only process the exact file path—a much safer alternative.

[This next step may be optional, it was already set when I tried it] We need to make another small change in the php5-fpm configuration. Open up `www.conf`:

```
$ sudo vim /etc/php5/fpm/pool.d/www.conf
```

Find the line, `listen = 127.0.0.1:9000`, and change the `127.0.0.1:9000` to `/var/run/php5-fpm.sock`. 

Restart php-fpm:

```
sudo service php5-fpm restart
```

## Step 5: Clone your app

Currently this section only has information about deploying Django apps. Other frameworks may be added in the future.

### Django

To clone your app from GitHub, you'll need to install git first (it may already be installed).

```
$ sudo apt-get install git
```

Navigate to the `/srv/` directory and clone your project there. Make sure you type the leading forward slash!

```
$ cd /srv/
$ sudo git clone https://github.com/HCDigitalScholarship/<your project name>.git
```

Next, you need to step up virtualenv, which lets you install packages for a single project instead of systemwide.

```
$ sudo apt-get install python-virtualenv
```

Now, make a virtual environment for your project in the `/usr/local/lib/python-virtualenv/` folder.

```
$ sudo mkdir /usr/local/lib/python-virtualenv
$ cd /usr/local/lib/python-virtualenv
$ sudo virtualenv <your project name> --python=<your Python version>
```

Make sure to supply `virtualenv` with the version of Python you want to be using. Usually this will be `--python=python2.7` for Python 2 or `--python=python3.4` for Python 3.

Now you can activate your virtualenv with

```
$ source /usr/local/lib/python-virtualenv/<your-project-name>/bin/activate
```

Unfortunately, virtualenv does not play nicely with the Unix permissions system. If you have your virtualenv activated and you run a command with `sudo`, the virtualenv won't apply for that command. Usually the only time you need to activate your virtualenv is to run `manage.py` (you can edit files and restart the server without needing the virtualenv) and to install new Python packages. Here's how you can do that:

```
$ sudo su
$ source /usr/local/lib/python-virtualenv/<your-project-name>/bin/activate
do whatever you need to
$ exit
```

The `sudo su` command makes you the root user. This is as dangerous as running every command with `sudo`, so only do what you absolutely need to and type `exit` (this exits from the root user, not from the SSH connection) as soon as you're done.

Activate your virtualenv as explained above, and then navigate to the root of your project directory (that you just cloned from GitHub). There should be a file named `requirements.txt` that contains a list of all packages that your website needs. Do

```
$ pip install -r requirements.txt
OR, if using Python 3
$ pip3 install -r requirements.txt
```

If you get an error, it is most likely because you forgot to switch to root. Don't run it again with `sudo`; follow the steps above to make sure that your virtualenv is properly activated in root mode.

The git repo you cloned probably didn't contain a `settings.py` file, for security reasons. You'll need to copy this from wherever you were developing previously. The exact location depends on your project's setup, but, for a project `foo`, it would typically go in either `foo/foo/settings.py` or `foo/settings.py`.

Try running

```
$ ./manage.py runserver
```

in the root directory of your project with the virtualenv activated. If it runs without errors, you've successfully set up your project! Make sure to kill the local server with Control+C.

## Step 6: Install uWSGI (Python only)

This step only needs to be done if you're deploying a Python app (e.g., Django or Flask). The Nginx server knows how to serve webpages to the outside world, but it doesn't know how to communicate with Python applications. That's where uWSGI comes in: it's the bridge between your Python framework and the Nginx server.

Make sure your virtualenv is NOT activated. Install uWSGI with

```
$ sudo pip install uwsgi
OR, if using Python 3
$ sudo pip3 install uwsgi
```

Copy [this file](https://raw.githubusercontent.com/nginx/nginx/master/conf/uwsgi_params) into your root project directory and name is `uwsgi_params`. If you save it on your local computer you can copy it like this (running this command locally):

```
$ scp uwsgi_params <username>@<server name>:/tmp/
```

And then back on the server:

```
$ sudo mv /tmp/uwsgi_params <your project directory>
```

Now, using the same process, copy this file to `/etc/nginx/sites-available/<project name>` (lightly modified from the example [on the uWSGI docs](http://uwsgi-docs.readthedocs.io/en/latest/tutorials/Django_and_nginx.html)).

```
# configuration of the server
server {
    # the port your site will be served on
    listen      80;
    # the domain name it will serve for
    server_name .example.com; # substitute your machine's IP address or FQDN
    charset     utf-8;

    # max upload size
    client_max_body_size 75M;   # adjust to taste

    # Django media
    location /media  {
        alias /path/to/your/mysite/media;  # your Django project's media files - amend as required
    }

    location /static {
        alias /path/to/your/mysite/static; # your Django project's static files - amend as required
    }

    # Finally, send all non-media requests to the Django server.
    location / {
        uwsgi_pass  django;
        include     /path/to/your/mysite/uwsgi_params; # the uwsgi_params file you installed
    }
}
```

Now you want to symlink this file to the `sites-enabled` directory.

```
$ ln -s /etc/nginx/sites-available/<projectname>  /etc/nginx/sites-enabled
```

Next, create a uWSGI configuration file at `/etc/uwsgi/apps-available/<projectname>.ini` following this template (you may need to create the `uwsgi` and `apps-available` folders first):

```
[uwsgi]

uid = www-data
gid = www-data
socket = /run/uwsgi/app/<projectname>/<projectname>.socket

plugins = python # or python3
chdir = <your project directory>
virtualenv = /usr/local/lib/python-virtualenv/<projectname>
module = <projectname>.wsgi:application

processes = 4
threads = 2
```

Create a symlink to this file in the `apps-enabled` directory.

```
$ sudo mkdir /etc/uwsgi/apps-enabled
$ ln -s /etc/uwsgi/apps-available/<projectname>.ini /etc/uwsgi/apps-enabled
```



## Appendix A - Other Resources

Many of these resources were used to prepare this tutorial and can be consulted for further reference.

- [How To Install Linux, nginx, MySQL, PHP (LEMP) stack on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-14-04)
- [Setting up Django and your web server with uWSGI and nginx](http://uwsgi-docs.readthedocs.io/en/latest/tutorials/Django_and_nginx.html)
- [PostgreSQL on Ubuntu](https://help.ubuntu.com/community/PostgreSQL)
- [Setting up PostgreSQL with Python 3 and psycopg on Ubuntu 16.04](https://www.fullstackpython.com/blog/postgresql-python-3-psycopg2-ubuntu-1604.html) (note Ubuntu 16.04)

## Appendix B - Common Unix Commands

Your user permissions won't allow you to do much on the server, so most of the commands below need to be run with `sudo`, which lets you execute a command with elevated privileges (elevated privileges, root permissions, and superuser permissions all mean the same thing). For example, you would type `sudo mv foo bar` to execute `mv foo bar` with root permissions. Always try commands without `sudo` first; the command line is unforgiving, and it's possible to do serious damage to the server when you have root permissions. If you get a permissions error when trying to run a command, *make sure it is really what you want to do*, and then do it again with `sudo`.

One exception to this rule is editing text files: always do `sudo vim`, because vim will let you open and edit a file you don't have write permissions for, but when you try to save it you won't be able to, and all your work will be lost.

#### Navigating the filesystem

- `ls` list files in current directory
- `pwd` print the name of the current directory (stands for "print working directory")
- `locate` show the location of a file or directory
- `cd <directory-name>` change directory (you can supply a relative or absolute path)

#### Moving and copying files

- `mv <file-path> <destination>` move a file. This is also used to rename files: `mv foo bar` renames the file `foo` to `bar`.
- `cp <file-path> <destination>` copy a file.
- `scp` copy from one machine/server to another (ex. `$ scp /source/path/image.jpg user@server:/destination/path)` Note: when moving files onto a server, you'll typically need to copy to `/tmp/` first then to the desired destination.

Note that `mv` and `cp` will by default silently overwrite destination files if they already exist. To prevent this behavior, use the `-n` flag (e.g., `mv -n foo bar`).

#### Editing text files

Remote servers don't usually provide graphics, so you can't use text editors like Sublime or Atom (or even gedit). The best text editor you have available is vim, which runs entirely in the terminal.

- `sudo vim <file-path>` open a file in the vim text editor (within vim press the `i` key to edit text, and `esc` then `:w` to save changes.  Type `esc` then `:q` to quit). For more information, take a look at [this tutorial](https://danielmiessler.com/study/vim/).

As noted in the introduction to this Appendix, you need to run vim with `sudo` or else you won't be able to save your changes.

#### Bash shortcuts

Bash, the shell that you are using, has a number of keyboard shortcuts to make entering commands easier. When you're typing a command, you can use the left and right arrow keys to move your cursor and edit different parts of it. You can use the up and down arrow keys to scroll through previous commands. If you accidentally typed a command that you don't want to execute, press Control+C to abort it.
