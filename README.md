In this tutorial, we will install all of the needed requirements for a LEMP stack (Linux, Nginx, MySQL, and PHP). This will all take place on a virtual machine created for us by IITS.  They can be contacted at [Prodesk](mailto:prodesk@haverford.edu). In this tutorial we will: 

#### Part I
1. Install Nginx
2. Install mySQL
3. Install PHP
4. Install and configure uWsgi

#### Part II
5. Install virtualenv
6. Create an virtual environment for the project
7. Install requirements in the virtualenv
8. Step X: Install Git and Clone Your Project

## Step 1: Connect to the Virtual Machine
You can connect to the virtual machine using [SSH](https://en.wikipedia.org/wiki/Secure_Shell).

    $ ssh yourusername@virtualmachinename

If you do not have a username on the server, one can be created by the DS librarians using `$ adduser <username>` and `$ sudo usermod -a -G sudo <username>` Use `control + D` to disconnect from SSH and return to your local machine. 

### Common Unix commands
- `ls` list files in current directory
- `pwd` print the name of the current directory (stands for "print working directory")
- `locate` show the location of a file or directory
- `cd <directory-name>` change directory (you can supply a relative or absolute path)
- `mv <file-path> <destination>` move a file
- `vi <file-path>` open a file in the vi text editor (within vi press the `i` key to edit text, and `esc` then `:w` to save changes.  Type `esc` then `:q` to quit)
- `scp` copy from one machine/server to another (ex. `$ scp /source/path/image.jpg user@server:/destination/path)` Note: when moving files onto a server, you'll typically need to copy to /tmp/ first then to the desired destination.
- `sudo <command>` run a command that requires security privileges.

## Step 2: Install virtualenv

Since this is our first interaction with the apt packaging system in this session, we should update our local package index before we begin so that we are using the most up-to-date information:

    $ sudo apt-get update

Enter the following commands to install virtualenv:

    $ sudo apt-get install python-virtualenv
    $ cd
    $ sudo mkdir /usr/local/lib/python-virtualenv
    $ cd /usr/local/lib/python-virtualenv

If you are using Python 3, make sure to specify the `--python=python3.X` flag (where X is the proper version number) in the following command. Otherwise, the virtual environment will be created with the default version of Python, which is typically Python 2.

    $ sudo virtualenv <project name>
    $ cd <project name>

Now you can activate your virtual environment with this command: `$ source bin/activate`

Note how the command line has changed to show that you are running in a virtual environment: `(bridge) ajanco@bridge:/usr/local/lib/python-virtualenv/bridge$`

## Step 3: Install Git and Clone Your Project
[Git](https://en.wikipedia.org/wiki/Git) is version-control software that is used to keep track of changes on collaborative projects. It allows you to save successive revisions of your work and then view them at a later time (for example, if you made a horrible mistake and just want to return to a working version), and it has strong support for multiple developers working on a single project. You can install Git like this:

    $ sudo apt-get install git

All of our projects live on [Github](https://github.com/HCDigitalScholarship). Find the project that you're working on and click on the title.  This will open the repository (usually abbreviated as "repo"). Check the name of the branch and be sure that you're in the branch that you need.  Next, click on the "Clone or download" button.  A URL will appear.  Select and copy that address.

Now, switch to a terminal window open on the server and type in the commands below. You can also `cd` into `/srv/www/`, `/srv/html/`, or `/var/www/` if you'd like. Whatever you choose, Git is going to create a folder there with the name of the repo and its project files.

    $ cd /srv/
    $ sudo git clone https://github.com/HCDigitalScholarship/project_name.git

The URL above comes from the "Clone or download" button on Github. You will then be prompted to enter your username and password.

### Common Git commands
- `$ git status` view the status of the current git repo
- `$ git add <file-path>` add the file or directory to the git working index (e.g., after you've edited some files)
- `$ git commit -m "<commit-message>"` commit the changes in the working index (that you added with the `git add` command). Think of this as a "save progress" command.
- `$ git log` view metadata about previous commits
- `$ git checkout -b <branch>` create a new branch and switch to it
- `$ git checkout <branch>` switch to an already-existing branch
- `$ git checkout <commit-hash>` look at another commit (a previous "save"). The commit hash can be found with the `git log` command.
- `$ git freeze`
- `$ git stash`

## Step X: Install Project Requirements

    $ sudo pip install -r requirements.txt

## Step X: Install Nginx ([source](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-14-04-lts)) 

Begin by typing:

    $ sudo apt-get install nginx

You will probably be prompted for your user's password. Enter it to confirm that you wish to complete the installation. The appropriate software will be downloaded to your server and then automatically installed.

At this point, try visiting the IP or domain of your project. You should see the Nginx welcome screen.  Congratulations! To start nginx, you can also enter `$ sudo /etc/init.d/nginx start`

### Configure nginx
Open up the default virtual host file.

    $ sudo vi /etc/nginx/sites-available/default

Select all and delete (actually move to buffer)

    :%d

Copy this [file](https://github.com/HCDigitalScholarship/Documentation/blob/master/QI). Then press `i` and paste (command-v).  

      server {
    listen 80;
    server_name pennstreaty.haverford.edu;
    root /usr/share/nginx/html;

    location /static/admin {
        alias /usr/local/lib/python-virtualenv/QI/lib/python2.7/site-packages/django/contrib/admin/static/admin;
    }

    location /static/ {
        alias /srv/QI/static_media/;
    }


    location /transcriber {
        try_files $uri /transcriber/index.php?$args;
        }

    location /qi/admin {
        try_files $uri /qi/admin/index.php?$args;
        }

    location /qi {
        try_files $uri /qi/index.php?$args;
        }

    location /themes {
        alias /usr/share/nginx/html/qi/themes;
        }

    location / {
        include /srv/QI/uwsgi_params;
        uwsgi_pass unix:/run/uwsgi/app/QI/QI.socket;
    }

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        include /etc/nginx/fastcgi_params;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_intercept_errors on;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }

This is the configuration file for a project called QI. You'll need to go through the file and change QI to your project name throughout the file. Also make sure that the configuration file lists the correct version of Python for your project.

Now restart Nginx with the new settings:

    $ sudo service nginx restart

Here's something very cool.  From the command line you can run `nginx -c /etc/nginx/nginx.conf -t` to have nginx check your configuration for errors.

Pay attention to the status message on the right [Ok] is great, [fail] means that the settings are yet correct.  You can check the error logs by typing:

    $ tail

## Step 4: Install MySQL ([source](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu))

MySQL is a powerful database management system used for organizing and retrieving data. To install MySQL, open terminal and type in these commands:

    sudo apt-get install mysql-server php5-mysql

During the installation, MySQL will ask you to set a root password. If you miss the chance to set the password while the program is installing, it is very easy to set the password later from within the MySQL shell.

Once you have installed MySQL, we should activate it with this command:

    sudo mysql_install_db

Finish up by running the MySQL set up script:

    sudo /usr/bin/mysql_secure_installation

The prompt will ask you for your current root password.

Type it in.

Enter current password for root (enter for none): 

OK, successfully used password, moving on...
Then the prompt will ask you if you want to change the root password. Go ahead and choose N and move on to the next steps.

It’s easiest just to say Yes to all the options. At the end, MySQL will reload and implement the new changes.

By default, a MySQL installation has an anonymous user, allowing anyone
to log into MySQL without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y                                            
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
... Success!

By default, MySQL comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...
Once you're done with that you can finish up by installing PHP.

## Step 5: Install PHP
PHP is an open source web scripting language that is widely use to build dynamic webpages.  You only need to install PHP if you're using Omeka, Drupal or another PHP-bases content management system. Otherwise, there's no need and you can skip to the next step. 

To install PHP, open terminal and type in this command.

    $ sudo apt-get install php5-fpm

Next, we need to make one small change in the php configuration. Open up php.ini:

    $ sudo nano /etc/php5/fpm/php.ini

Find the line, cgi.fix_pathinfo=1, and change the 1 to 0.  
 
    cgi.fix_pathinfo=0

Now press ctrl-O to save the change and ctrl-X to exit the viewer.

If this number is kept as 1, the PHP interpreter will do its best to process the file that is as near to the requested file as possible. This is a possible security risk. If this number is set to 0, conversely, the interpreter will only process the exact file path—a much safer alternative. 

[This next step may be optional, it was already set when I tried it]
We need to make another small change in the php5-fpm configuration. Open up www.conf:

    $ sudo nano /etc/php5/fpm/pool.d/www.conf

Find the line, listen = 127.0.0.1:9000, and change the 127.0.0.1:9000 to /var/run/php5-fpm.sock.
`listen = /var/run/php5-fpm.sock`
Save and Exit.

Restart php-fpm:

    sudo service php5-fpm restart

## Step X: Set STATIC_ROOT and collectstatic

## Step X: Install uWsgi

    $ pip install uwsgi
