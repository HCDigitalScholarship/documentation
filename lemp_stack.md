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
- `$ git stash`Temporarily stores all modified tracked files

## Step X: Install Project Requirements

    $ sudo pip install -r requirements.txt

Test the app to make sure all requirements are installed by running 

    $ python manage.py runserver

If you don't get any errors, then the app is properly configured and you can move on to the next step!

## Step X: Install uWsgi
What is an application server and why are we using it? 

Sockets: uWSGI can create multiple types of sockets, and we will use two of them here--HTTP and Unix. We'll start with an HTTP socket so that we can reqeust HTML from the app without having to also configure a web server. Once we are sure that uWSGI is properly configured, we will switch to a Unix file socket for performance and security reasons.

    $ pip install uwsgi

Create a new file /etc/uwsgi/apps-available/project_name.ini and paste in the following contents:

    [uwsgi]

    #Environmental settings: 
    uid = www-data
    gid = www-data
    #socket = /run/uwsgi/app/duchemin/duchemin.socket
    
First we are setting the user and group that can access the file socket, and specifying the location of the file socket. We specify the /run/uwsgi/app folder because we know the uWSGI service has permission to access it. The file socket location line should be commented out initially for reasons that will be explained in a moment.

    # For testing: curl -L http://localhost:5000
    socket = localhost:5000
    protocol = http

These lines are only there for testing purposes, and they create the HTTP socket. They will eventually be commented out. We can test the socket by requesting HTML using the "curl" command from the command line. 

    # Application settings
    plugin = python
    chdir = /srv/DuChemin
    #virtualenv = /usr/local/lib/python-virtualenv/ticha-django
    module=duchemin.wsgi:application

These lines contain specifics for the application server hooking into the app.
`plugin` specifies the type of application. In most cases, e.g. a Django app, this will be `python`, but it could also be ruby, python3, mongoDB, PHP, and so on. See the uWSGI docs for all of the available plugins.

`chdir` specifies the project root directory. For a Flask or Django project, this is the folder that contains `manage.py`

`virtualenv` specifies the location of the virtual environment.

`module` defines the application for uWSGI. The name of the application should be identical to to the name specified in the `WSGI_APPLICATION` line of `settings.py`

    # Performance tuning
    processes = 4
    threads = 2

The python plugin by default does not have threads enabled, so we enable threading support here. We can adjust these lines based on performance once the project is deployed.

Save and quit.

Create a symbolic link in `/apps-enabled` to the `.ini` file in `/apps-available` to activate the configuration:

    $ sudo ln -s /etc/uwsgi/apps-enabled/config.ini /etc/uwsgi/apps-available/config.ini

Or switch to root (sudo su -) to execute this command.

Restart the service
    
    $ sudo service uwsgi restart
    
Now let's test the socket to see if we can get an HTML response:
    
    $ curl -L http://localhost:5000

The `-L` option follows redirects, which is necessary if your project redirects from the index page.

If you get an HTML response, you've successfully configured uWSGI! If you get an error or no response, check the logs at `/var/log/uwsgi/app/nameofapp.log`

Once you've successfully configured uWSGI, comment out the two lines related to the HTTP socket and uncomment the line for the Unix file socket. Save, quit, and restart uWSGI.

## Step X: Install Nginx ([source](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-14-04-lts)) 

Begin by typing:

    $ sudo apt-get install nginx

You will probably be prompted for your user's password. Enter it to confirm that you wish to complete the installation. The appropriate software will be downloaded to your server and then automatically installed.

At this point, try visiting the IP or domain of your project. You should see the Nginx welcome screen.  Congratulations! To start nginx, you can also enter `$ sudo /etc/init.d/nginx start`

### Configure nginx
Create and open a new config file (Nginx config files do not have extensions)

    $ sudo vi /etc/nginx/sites-available/projectname 

We'll start by creating a server block enclosed in curly braces:

    server {
        listen 80;
        server_name projectname.haverford.edu
        root /srv/projectRoot
        
     # ...location directives go here...
     
     }
     
The `listen` line opens port 80 for HTTP requests. In some cases, we'll also listen on port 443 for HTTPS requests.

`server_name` accepts requests with this value as the root of the URL. This could also be the IP address for a Droplet. In any case, it does NOT need to have a DNS record yet.

`root` refers to the root folder of the project. For a Django project, this is the folder containing `manage.py`.

Now we'll add location directives as needed. Location directives interpret HTTP requests for URLs and routes those requests to the appropriate folders on the file system of the web server. 

Location directives are marked by `location /path { }` in which a path in the request URL is either routed to a directory on the server, or a request for a specified filetype is sent to the application that can interpret it. For example, to serve PHP, we'll use the following location rule (this also requires that the `php5-fpm` is installed, which you can do by running `sudo apt-get install php5-fpm`). Just copy and paste this block into your config file:

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        include /etc/nginx/fastcgi_params;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_intercept_errors on;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }
    }

`fastcgi_params` defines parameters that FastCGI will pass to the PHP interpreter from the client request.  

When `fastcgi_intercept_errors` is set to `on`, it passes Nginx errors on to the application so that a specific error page can be displayed. For example, Omeka sites already have 404 and 500 error pages, so those pages will be displayed instead of Nginx's default error pages.

`fastcgi_pass` is specifying the location of the socket to the PHP interpreter. If you've installed `php5-fpm`, it will create that socket in `/var/run/`. 

To serve the index page of a Django project, use this block:

    location / {
                include /etc/nginx/uwsgi_params;
                uwsgi_pass unix:/path/to/uwsgi/socket.socket;
        }

The `uwsgi_params` defines parameters that uWSGI will use to complete a request handled by Nginx. It makes information from the client request available to the application server such as the protocol of a request (e.g. HTTP) or the method of the request (e.g. GET, POST, etc.). This file is included out of the box when you install Nginx, so unless it's a very unique case we will almost always point to this default version.

`uwsgi_pass` must match the path to the Unix file socket specified in the uWSGI configuration file for the project. `/path/to/uwsgi/socket.socket` will usually look something like `/run/uwsgi/app/ticha-django-site/ticha-django-site.socket`

If the images for our project are stored in another folder outside the project, we can use something like:

    location /images {
        alias /var/www/html/images;
        }

`root` will append the path to the specified request, whereas `alias` will replace the request with the specified path. For example, the above directive will route a request for `project.haverford.edu/images` to `/var/www/html/images` directory on the server. If we wrote the same directive with `root` instead, like this:

    location /images {
        root /var/www/html/images;
        }

Then the path would be `/var/www/html/images/images`.

If you are using `alias`, remember to stay consistent in the usage of slashes ("`/`"). If the request ends with a slash (e.g. "`/images/`") then the path must also end with a slash (e.g. "`alias /var/www/html/images/`"). Otherwise, a request may produce a path that is missing a slash! A good way to test your location directives is through the "Inspect" feature of your browser along with the Nginx logs (`/var/log/nginx/error.log` and `/var/log/nginx/access.log`) to see if the request matches the path you want.

Another useful directive is `try_files`, which enables Nginx to look for files of the same type in a specified folder:

    location ~ \.(mp3|mp4) {
       root /www/media;
    }

Generally speaking, `location` directives like this give us lots of control over the structure of our URLs.

Once you've written all of your location directives, save and quit. 

Here's something very cool.  From the command line you can run `nginx -c /etc/nginx/sites-available/project.conf -t` to have nginx check your configuration for errors.

To enable this configuration, you'll need to create a symbolic link in the `apps-enabled` folder by typing:

    $ ln -s /etc/nginx/sites-enabled/projectname /etc/nginx/sites-available/projectname
    
Finally, restart the Nginx service to activate your project configuration:

    $ sudo service nginx restart

Pay attention to the status message on the right [Ok] is great, [fail] means that the settings are not yet correct.  Remember you can check the error logs by typing:

    $ tail /var/log/nginx/error.log
    
Testing Nginx config by configuring local /etc/hosts file

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

