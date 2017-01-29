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

### 5. Configure the uncomplicated firewall(ufw)

* 
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


