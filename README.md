# Linux Server Configuration

### 1. Setup development environment

* Create a [new development environment](https://www.udacity.com/account#!/development_environment) provided by Udacity and Amazon.
* Download the private key.
* Move the private key into the folder `~/.ssh/`
`$ mv ~/Downloads/udacity_key.rsa ~/.ssh/`
* Set file permissions - only owner can read and right.
`$ chmod 600 ~/.ssh/udacity_key.rsa`
* SSH into your environment
`$ ssh -i ~/.ssh/udacity_key.rsa root@<PUBLIC-IP-ADDRESS>`

### 2. User management

* Create a new user.
    `$ sudo adduser <new-user>`
* Add user to the 'sudoers' list.
    `$ nano /etc/sudoers.d/<new-user>`
* Give the new user 'sudo' permissions by adding the following line to the above created file
    `student ALL=(ALL) NOPASSWD:ALL`
    
#### To SSH into the environment with the newly created user.

* From the root login - 
`$ nano /etc/ssh/sshd_config`
And set the `PasswordAuthentication` to yes - so that you can ssh into the new user using the set password
* To login in with the new user type in your local terminal(not the virtual machine)
`$ ssh <new-user>@<PUBLIC-IP-ADDRESS>`
    
### 3. Update and upgrade packages [[Reference](http://askubuntu.com/questions/94102/what-is-the-difference-between-apt-get-update-and-upgrade)]

* Update the list of available packages and their versions.
`$ sudo apt-get update`
* Install newer versions of the packages you have.
`$ sudo apt-get upgrade`<sup>[[1](#1-apt-not-working)]</sup>

### 4. Configure local timezone to UTC [[Reference](http://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt)]

* Execute `sudo dpkg-reconfigure tzdata`
* In the interactive window select `None of the above`
* In the menu select `UTC`

### 5. Secure server

* Edit the `sshd_config` file
    * Open the config file - 
    `$ sudo nano /etc/ssh/sshd_config`
    * Change the line `Port 22` -> `Port 2200` and save.

* Restart the SSH service
`/etc/init.d/ssh restart    `

* Create SSH keys. 
    * Generate SSH key on local machine.
    `$ ssh-keygen`
    * The default filename is id_rsa. You get a prompt to change it if you want.
    * Copy the output of the `.pub` file- 
    `$ cat ~/.ssh/<filename>.pub`
    * On your virtual machine, if you don't already have a `~/.ssh` folder create one.
    `$ mkdir ~/.ssh`
    * Create an `authorized_keys` file inside the `.ssh` directory.
    `$ nano ~/.ssh/authorized key`
    * Paste the contents of `.pub` file you had copied in the earlier steps and save
    * Give appropriate permissions to the folder as well as the file - 
        * `sudo chmod 700 ~/.ssh`
        * `sudo chmod 644 ~/.ssh/authorized_keys`
    * Open SSHD config
    `$ sudo nano /etc/ssh/sshd_config`
    * Change `PasswordAuthentication` to `no`.
    * Now login with the new user - 
    `ssh <new-user>@PUBLIC-IP-ADDRESS -p 2200`

    **Note**: Alternatively you can use [these instructions](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server) to do the above steps easily with one command - `ssh-copy-id`

### 6. Configure the uncomplicated firewall(ufw)

* Check the ufw status
`sudo ufw status`
* Deny all incoming and allow all outgoing connections
`sudo ufw default deny incoming`
`sudo ufw default allow outgoing`
* Allow all incoming ssh connections
`sudo ufw allow ssh`
* Allow all incoming http connections
`sudo ufw allow www`
* Allow incoming udp packets on port 123(ntp)
`sudo ufw allow 123/udp`

### 7. Download and install Item Catalog application

#### 7.1 Install and configure git

1. Install git

`sudo apt-get install git`

2. Configure

```
git config --global user.name "Your Name"

git config --global user.email "youremail@domain.com"

git config --list (To view the set name and email in the earlier step)  
```


#### 7.2 Install and configure apache

1. Install apache web server

`sudo apt-get install apache2`

2. Open a browser and enter your public ip address. It should give a "Apache2 Ubuntu Default Page - It works".
 

#### 7.3 Configure and deploy a simple flask application ( [Reference](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) ) 

1. Install and enable mod_wsgi
```
$ sudo apt-get install libapache2-mod-wsgi python-dev
$ sudo a2enmod wsgi
```

2. Create a Flask app
    a. Move to `/var/www` directory.
    
    `cd /var/www`
    b. Create a folder eg. catalog and change to that directory
    
    `sudo mkdir catalog && cd catalog`
    c. Again create a folder catalog which will contain all files of your application
    
    `sudo mkdir catalog && cd catalog`
    d. Make necessary folders and files
    
    `sudo mkdir static templates`
    
Directory structure till now - 
```
|----catalog
|---------catalog
|--------------static
|--------------templates
```

3. create the `__init__.py` file that will contain the flask application logic.

`sudo nano __init__.py`

Add following logic(basic flask app) and then save and close.- 
```
from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hello, I love Digital Ocean!"
if __name__ == "__main__":
    app.run()
```
 



### Problems faced and their solutions

#### 1. `apt` not working

**Problem**: While running `sudo apt-get upgrade`, the terminal had prompted me for configuration of some package which was never completed(because of intermittent internet connection). 
After logging in again I couldn't install using `apt` command.

Error - `Could not get lock /var/lib/dpkg/lock`

**Solution**: http://askubuntu.com/questions/15433/unable-to-lock-the-administration-directory-var-lib-dpkg-is-another-process/315791#315791

#### 2. `sudo: unable to resolve host`

**Problem**: Running a command with `sudo` yields a warning - `sudo: unable to resolve host <host-machine-name>`

**Solution**
* Copy the machine host name.
`cat /etc/hostname` 
* Add the hostname to the hosts file.
`sudo nano /etc/hosts`
* On the first line type in - `127.0.1.1 <machine-host-name>`
* Save and exit.

#### 3. Annoying SSH timeout

**Problem**: SSH session/instance timeout was too less.

#### 4. Unable to open database file

**Problem**: The catalog app couldn't locate the database file.

**Solution**: Worked after moving the database file into `/tmp/`. If you face permission problem later, take a look at the next problem [(Problem 5)](5-attempt-to-write-a-read-only-database).

#### 5. Attempt to write a read-only database

**Problem**: (sqlite3.OperationalError) attempt to write a readonly database.

**Solution**: 
Give write permission to database file.
```
sudo chmod 666 /path/to/database/file.db
```

#### 6. Threading problem 

**Problem**: `SQLite objects created in a thread can only be used in that same thread`

**Solution**: mod_wsgi uses session threading internally. Following lines have to be added to handle threading.
`from flask import scoped_session`
`session = scoped_session(sessionmaker(bind=engine))`
Note: [Reference](http://stackoverflow.com/questions/34009296/using-sqlalchemy-session-from-flask-raises-sqlite-objects-created-in-a-thread-c)

#### 7. Facebook login redirect URL

**Problem**: Facebook login didn't work even after setting auth redirect url to `http://ec2-xx-xx-xxx-xxx.us-west-2.compute.amazonaws.com/`

**Solution**: After a lot of trial and error combinations with that URL, just pondered whether the public IP address - `http://xx.xx.xxx.xxx/` -  would work. And voila!

#### 8. Google and facebook client secrets

**Problem**: The application couldn't locate the JSON file.

**Solution**: Worked after providing the absolute paths of both (Facebook as well as Google) JSON files in the application.


### Extra 

* Glances - System monitoring application (Dependency python-bottle)
