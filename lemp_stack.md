**WARNING**: This tutorial is incomplete! You will have to figure some things out for yourself. When you do, please edit the tutorial to include that information.



In this tutorial, we will install all of the needed requirements for a LEMP stack (Linux, Nginx, MySQL, and PHP or Python). This will all take place on a virtual machine running Ubuntu 14.04 created for us by [IITS](mailto:prodesk@haverford.edu). While written with Ubuntu 14.04 in mind, most of the information here is also correct for Ubuntu 16.04.

The tutorial assumes basic knowledge of the Unix command line. If you need a refresher on common Unix commands, see Appendix B.  Note that wherever you see text in `<angle-brackets>`, substitute it (without the brackets) with your own value. For example, if you see `ssh <username>@haverford.edu`, you might write `ssh kbenston@haverford.edu` instead. If a file name has spaces, make sure to put quotes around it. `rm foo bar` removes two files, `foo` and `bar`, while `rm "foo bar"` removes a single file called `foo bar`.

Whenever there is a command you should run, it will be set on its own line and preceded by a dollar sign; don't type the dollar sign. So if you see

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
$ ssh <username>@<server-name>
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

Now visit the IP address or hostname of your project in your browser. You should see an Nginx welcome page. If you don't, try running

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

For a helpful tutorial on MySQL, see .<br>
<br>

To create a new database and user for your project, log in to the MySQL shell by typing `$ mysql -u root -p`.  In the shell:

```
> CREATE DATABASE <database-name>;
> CREATE USER '<user_name>'@'localhost' IDENTIFIED BY '<password>';
> GRANT ALL PRIVILEGES ON * . * TO '<user_name>'@'localhost';
> FLUSH PRIVILEGES;
```

Finally, if you are using Python, you'll need some MySQL headers:

```
$ sudo apt-get install libmysqlclient-dev
```

### PostgreSQL

Install PostgreSQL with

```
$ sudo apt-get install postgresql postgresql-contrib
```

Some setup needs to be done. Type

```
$ sudo -u postgres psql postgres
```

This should open up a new prompt, something like `postgres=#`. You should make a Postgres account before you create your database. The username should reflect the project, so for example the user for a project Foo might be called `fooadmin`, and the database could be called `foodb`. Still at the Postgres prompt, type (make sure to put single quotes around the password, and a semicolon at the end of the line)

```
$ CREATE USER <database-username> WITH PASSWORD '<password>';
$ CREATE DATABASE <database-name>;
$ ALTER ROLE <database-username> SET client_encoding TO 'utf8';
$ ALTER ROLE <database-username> SET default_transaction_isolation TO 'read committed';
$ ALTER ROLE <database-username> SET timezone TO 'UTC';
$ GRANT ALL PRIVILEGES ON DATABASE <database-name> TO <database-username>;
```

Finally, type `\q` to exit.

## Step 4: Install PHP or Python

If you're using PHP, follow [this tutorial](https://github.com/HCDigitalScholarship/documentation/blob/master/php.md) to get it installed.

Python already comes preinstalled on Ubuntu systems. You will most likely need to install either the `python-dev` or `python3-dev` package, depending on which version of Python your website uses. Do

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

Some packages need additional setup. You should only follow these steps if your project requires these packages.

### Additional optional packages
These packages may or may not be required for your package. If in doubt, consult a DS librarian.

#### psycopg2

The `psycopg2` is a Python interface for communicating with PostgreSQL databases. It has several prerequisites that need to be installed:

```
$ sudo apt-get install libpq-dev build-essential postgresql-server-dev-all
```

#### lxml

The lxml package is used for XML parsing. Installing it on Ubuntu is tricky. You'll need the following packages:

```
$ sudo apt-get install libxml2-dev libxslt1-dev zlib1g-dev lib32z1-dev
$ sudo apt-get build-dep python3-lxml
```

Some of those packages might not be necessary. If you have some free time, you can experiment with the minimal number of packages needed to successfully compile lxml.

You will likely also need to set up a swapfile to give the compiler enough memory to finish compilation. Follow the instructions in [this Stack Overflow post](https://stackoverflow.com/questions/24455238/lxml-installation-error-ubuntu-14-04-internal-compiler-error) (and make sure to turn the swapfile off when you're done).

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
$ sudo virtualenv <projectname> --python=<your Python version>
```

Make sure to supply `virtualenv` with the version of Python you want to be using. Usually this will be `--python=python2.7` for Python 2 or `--python=python3.4` for Python 3.

**Notes: If the last command line does not work, try this:**  
*On macOS and Linux:*
```
$ sudo python3 -m virtualenv <projectname>
```
*On Windows:*
```
$ sudo py -m virtualenv <projectname>
```
***
Then you may activate your virtualenv. 

*On macOS and Linux:*
```
$ source <projectname>/bin/activate
```
*On Windows:*
```
$ .\<projectname>\Scripts\activate
```

**Notes:** Unfortunately, virtualenv does not play nicely with the Unix permissions system. If you have your virtualenv activated and you run a command with `sudo`, the virtualenv won't apply for that command. Usually the only time you need to activate your virtualenv is to run `manage.py` (you can edit files and restart the server without needing the virtualenv) and to install new Python packages. Here's how you can do that:

```
$ sudo su
$ source /usr/local/lib/python-virtualenv/<your-project-name>/bin/activate
do whatever you need to
$ exit
```

The `sudo su` command makes you the root user. This is as dangerous as running every command with `sudo`, so only do what you absolutely need to and type `exit` (this exits from the root user, not from the SSH connection) as soon as you're done.
***

Activate your virtualenv as explained above, and then navigate to the root of your project directory (that you just cloned from GitHub). 
```
$ cd /srv/<your-project-name>/
```
There should be a file named `requirements.txt` that contains a list of all packages that your website needs. Do

```
$ pip install -r requirements.txt
OR, if using Python 3
$ pip3 install -r requirements.txt
```

If you get an error, it is most likely because you forgot to switch to root. Don't run it again with `sudo`; follow the steps above to make sure that your virtualenv is properly activated in root mode.<br/>
If you get wired errors, also try to upgrade your `pip` by running `pip install --upgrade pip`

The git repo you cloned probably didn't contain a `settings.py` file, for security reasons. You'll need to copy this from wherever you were developing previously. The exact location depends on your project's setup, but, for a project `foo`, it would typically go in either `foo/foo/settings.py` or `foo/settings.py`. You also may need to change the `DATABASES` variable in the settings file. If you are setting up a production server, make sure `DEBUG` is set to `False`.

Try running

```
$ ./manage.py runserver
```

in the root directory of your project with the virtualenv activated. If it runs without errors, you've successfully set up your project! Make sure to kill the local server with Control+C.

## Step 6: Install uWSGI (Python only)

This step only needs to be done if you're deploying a Python app (e.g., Django or Flask). The Nginx server knows how to serve webpages to the outside world, but it doesn't know how to communicate with Python applications. That's where uWSGI comes in: it's the bridge between your Python framework and the Nginx server.

Install uWSGI with

```
$ sudo apt-get install uwsgi uwsgi-plugin-python
OR, for Python 3
$ sudo apt-get install uwsgi uwsgi-plugin-python3
```

First, create a uWSGI configuration file at `/etc/uwsgi/apps-available/<projectname>.ini` following this template. Again, if your project is already in production, copy and edit the production config file.

```
[uwsgi]

uid = www-data
gid = www-data
socket = /run/uwsgi/app/<projectname>/<projectname>.socket  // you define a socket here

plugins = python # or python3
chdir = /path/to/your/project/root/directory
virtualenv = /usr/local/lib/python-virtualenv/<projectname>
module = <projectname>.wsgi:application  // look at settings.py and see how WSGI_APPLICATION is defined

processes = 4
threads = 2
```

Create a symlink to this file in the `apps-enabled` directory.

```
$ sudo ln -s /etc/uwsgi/apps-available/<projectname>.ini /etc/uwsgi/apps-enabled/<projectname>.ini
```

Then, copy this file to `/etc/nginx/sites-available/<projectname>`, editing it to insert your project-specific information. If you're setting up a new server for a project that is already in production, you're probably better off copying the configuration file from the production server and making any necessary changes.

```
server {
    listen 80;

    location /static {
        alias /path/to/your/static/directory;
    }

    location / {
        uwsgi_pass  unix:/run/uwsgi/app/<projectname>/<projectname>.socket; 
                # should be the same as the socket you just defined in the previous file
        include     /etc/nginx/uwsgi_params;
    }
}
```

Now you want to symlink this file to the `sites-enabled` directory.

```
$ sudo ln -s /etc/nginx/sites-available/<projectname>  /etc/nginx/sites-enabled
```


Make sure that root is the owner of all the configuration files:

```
$ sudo chown root:root /etc/nginx/sites-available/<projectname> /etc/nginx/sites-enabled/<projectname> /etc/uwsgi/apps-available/<projectname>.ini /etc/uwsgi/apps-enabled/<projectname>.ini
```

Make sure that `www-data` is the owner of your project files:

```
$ sudo chown -R www-data:www-data /srv/<projectname>
```

Optionally you can then grant write access to all members of `www-data` without requiring them to use sudo:

```
$ sudo chmod -R g+w /srv/<projectname>
```

Restart Nginx and uWSGI:

```
$ sudo service nginx restart
$ sudo service uwsgi restart
```

Your website should now be up and running! If it isn't (and it probably isn't), check the uWSGI and Nginx logs at `/var/log/uwsgi/` and `/var/log/nginx/`, respectively. There are a couple of things that you can try.

- Delete the `/etc/nginx/sites-enabled/default` file.
- Change `ALLOWED_HOSTS` in your Django `settings.py` file to match the URL or IP address of your server. This is only necessary if `DEBUG` is set to `False`.

## Appendix A - Other Resources

Many of these resources were used to prepare this tutorial and can be consulted for further reference.

- [How To Install Linux, nginx, MySQL, PHP (LEMP) stack on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-14-04) (note Ubuntu 14.04)
- [Setting up Django and your web server with uWSGI and nginx](http://uwsgi-docs.readthedocs.io/en/latest/tutorials/Django_and_nginx.html)
- [PostgreSQL on Ubuntu](https://help.ubuntu.com/community/PostgreSQL)
- [Setting up PostgreSQL with Python 3 and psycopg on Ubuntu 16.04](https://www.fullstackpython.com/blog/postgresql-python-3-psycopg2-ubuntu-1604.html)
- [How To Use PostgreSQL with your Django Application on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-use-postgresql-with-your-django-application-on-ubuntu-14-04)
- [lxml Installation](http://lxml.de/installation.html)
- [How to Create a New User and Grant Permissions in MySQL](https://www.digitalocean.com/community/tutorials/how-to-create-a-new-user-and-grant-permissions-in-mysql)

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
