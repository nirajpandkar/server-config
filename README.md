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
    
### 3. Update and upgrade packages

* Update the list of available packages and their versions.
`$ sudo apt-get update`

* Install newer versions of the packages you have.
`$ sudo apt-get upgrade`<sup>[1](#1-.-apt-not-working)</sup>


### Problems faced and their solutions

#### 1. `apt` not working

**Preface**: While running `sudo apt-get upgrade`, the terminal had prompted me for configuration of some package which was never completed(because of intermittent internet connection). 
After logging in again I couldn't install using `apt` command.
Error - `Could not get lock /var/lib/dpkg/lock`
**Solution**: http://askubuntu.com/questions/15433/unable-to-lock-the-administration-directory-var-lib-dpkg-is-another-process/315791#315791