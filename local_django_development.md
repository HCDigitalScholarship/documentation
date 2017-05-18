## Virtualenv and Local Django Development

In this tutorial, we'll install Python virtualenv, clone a Django project to a local repository (on your laptop or desktop machine), and run the Django server.

### Install a package manager for Mac OS
On a Mac? Install homebrew if you haven't already by following the instructions at https://brew.sh/. Homebrew is a package manager for command line tools, similar to the aptitude or yum package managers for Linux. When you install a command line tool via homebrew, it downloads the source code from a trusted repository and unpacks it for you... no other intervention required!

### Install Git
You can install git from the git website, and can use a desktop client if you prefer a graphical interface. Git as a command line tool si very simple to use, and can be installed via homebrew if you don't already have it.
  
  `brew install git`

### Install virtualenv
Next, we'll install virtualenv using pip. Pip is a package manager for Python--similar to package managers for Linux or Mac OS--that downloads source code from trusted repositories. Virtualenv creates a self-contained Python environment that, when activated, allows specific versions of plugins/libraries to be run locally. If, for example, I have NumPy 1.12 installed on my system but I the project I'm developing uses NumPy 1.8, I can install 1.8 within the virtualenv to avoid version conflicts with my sytem installation. When the virtualenv is activated, Python will look for v1.8. When it's deactivated, it will default to the system Python installation and v1.12.
  
  `pip install virtualenv`

You can test to make sure virtualenv was installed by typing
  
  `which virtualenv`

The likely response will be something like `/usr/local/bin/virtualenv`. If you see nothing in response, virtualenv was not installed.

### Create folders for projects and virtualenvs
We also need to create one folder each for projects/Github repositories and for Virtualenvs. We can do this in one step:
  
  `mkdir -p ~/Projects ~/Virtualenvs`
  
Depending on your user permissions, you may need to add `sudo` in front of the `mkdir` command. If the new directories are in your home directory, that will likely not be necessary. Note that the `~` character is a shorthand for your home directory. 

### Create and activate the virtualenv
Let's move to the `~/Virtualenvs` folder and then create the virtualenv for our project, our example here is `pennstreaty`:
  
  `virtualenv name-of-project`
  
This command will create a new folder called `pennstreaty`, complete with folders that should look familiar from your system installation of Python. Within the `bin` folder, there's an executable called `activate`. We `source` this executable to activate the virtualenv:
  
  `source bin/activate`
  
Keep in mind that you can source this file from anywhere on the file system as long as you include the full path to it: 
  
  `source ~/Virtualenvs/name-of-project/bin/activate`

### Clone the repository and install requirements
The next step is to pull the repository to your laptop into the `~/Projects` folder we created. Copy the path to the repo by clicking the green "Clone or Download" button on the Github repository. Then, in the terminal window, type:

  `git clone https://github.com/path-to-repository.git`

Once you have cloned the repo, move into the root directory of the repo. There will be a file in the repo called `requirements.txt` that contains a list of the Python libraries the project needs. We can install all of the libraries with one command:

  `pip install -r requirements.txt`
  
It's possible that some of the Python libraries will have OS-level dependencies. These can be dealt with on a case-by-case basis, but may require you to install some packages through brew or aptitude. 

For projects that have already been deployed and are using a database server, the requirements file will likely include a database connector library--chances are this will be psycopg2 for projects using PostgreSQL and MysqlDB for projects using MYSQL. These require that a database server be set up and configured on your laptop. There's a good chance that this will not be the case, and we don't need to install those dependencies (it will likely throw an error on a database library when you run pip install). You can remove database-related libraries from your local version of the requirements.txt file and try to run `pip install` again.

### Make sure the settings file is in place
There is a good chance that the repo will not include the `settings.py` file required to run the project. Settings files often contain database authentication information, including usernames and passwords for the database server. As such, it is best practice to not track any files that contain authentication credentials. You will need to obtain the settings file from someone. Talk to your supervisor or project lead about gaining access to it. Once you have the file, you'll need to place it in the main app folder alongside `urls.py`, `views.py`, etc. Make sure that the database section of the settings file points to the `db.sqlite3` file for local development.

### Run the Django server
Last but not least, `cd` back into the main folder of the repo where `manage.py` is, and runserver!

`python manage.py runserver`
  
This should start the server on port 8000 by default. You can specify which port the Django server runs on after the runserver command. For example, run `python manage.py runserver 0.0.0.0:8080` to run the server on port 8080 (0.0.0.0 and 127.0.0.1 are IP address stand-ins for 'localhost'). To test the project, open a browser window and navigate to `http://localhost:8000` (or the port number you specified in the runserver command). You should see the home page of the project! Note that as you click around, the Django server logs activity and errors to the terminal window. See the [Django documentation](https://docs.djangoproject.com/en/1.11/ref/django-admin/#runserver) for ways to run the server in the background, and other tricks to the runserver command.
 
Also check the [Django docs](https://docs.djangoproject.com/en/1.11/ref/django-admin) for more things you can do with `manage.py`. You can apply database migrations, add admin user accounts, and more! Remember that you cannot run any Python scripts in the Django project (including the Django server) unless you've activated the virtualenv. 
