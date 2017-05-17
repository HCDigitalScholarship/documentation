##

On a Mac? Install homebrew if you haven't already

  https://brew.sh/

Install git via homebrew if not already installed
  
  `brew install git`

Next, we'll install virtualenv. Virtualenv creates a self-contained Python environment that, when activated, allows specific versions of plugins/libraries to be run locally. If, for example, I have NumPy 1.12 installed on my system but I the project I'm developing uses NumPy 1.8, I can install 1.8 within the virtualenv to avoid version conflicts with my sytem installation of Python. When the virtualenv is activated, python will look for v1.8. When it's deactivated, it will default to the system Python installation and v1.12.
  
  `pip install virtualenv`

You can test to make sure virtualenv was installed by typing
  
  `which virtualenv`

The likely response will be something like `/usr/local/bin/virtualenv`. If you see nothing in response, virtualenv was not installed.
  
We also need to create one folder each for projects/Github repositories and for Virtualenvs. We can do this in one step:
  
  `sudo mkdir -p ~/Projects ~/Virtualenvs`
  
Note that the `~` character is a shorthand for your home directory. 

Let's move to the `~/Virtualenvs` folder and then create the virtualenv for our project, our example here is `pennstreaty`:
  
  `virtualenv name-of-project`
  
This command will create a new folder called `pennstreaty`, complete with folders that should look familiar from your system installation of Python. Within the `bin` folder, there's an executable called `activate`. We `source` this executable to activate the virtualenv:
  
  `source bin/activate`
  
Keep in mind that you can source this file from anywhere on the file system as long as you include the full path to it: 
  
  `source ~/Virtualenvs/name-of-project/bin/activate`

The next step is to pull the repository to your laptop into the `~/Projects` folder we created. Copy the path to the repo by clicking the green "Clone or Download" button on the Github repository. Then, in the terminal window, type:

  `git clone https://github.com/path-to-repository.git`

Once you have cloned the repo, move into the root directory of the repo. There will be a file in the repo called `requirements.txt` that contains a list of the Python libraries the project needs. We can install all of the libraries with one command:

  `pip install -r requirements.txt`

There is a good chance that the repo will not include the `settings.py` file required to run the project. Settings files often contain database authentication information, including usernames and passwords for the database server. As such, it is best practice to not track any files that contain authentication credentials. You will need to obtain the settings file from someone. Talk to your supervisor or project lead about gaining access to it. Once you have the file, you'll need to place it in the main app folder alongside `urls.py`, `views.py`, etc. Make sure that the database section of the settings file points to the `db.sqlite3` file for local development.

Last but not least, `cd` back into the main folder of the repo where `manage.py` is, and runserver!

`python manage.py runserver`
  
 This should start the server on port 8000 by default. You can specify which port the Django server runs on after the runserver command. For example, add `0.0.0.0:8080` to run the server on port 8080. To test the project, open a browser window and navigate to `http://localhost:8000`. You should see the home page!  
