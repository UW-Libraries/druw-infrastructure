# Druw infrastructure

This repository sets a up a centos 7 based infrastructure for a research data repository based on samvera/hyrax (a Ruby/Rails application). It requires separately cloning the [druw repository](https://bitbucket.org/uwlib/druw). It will build either a development or a fullstack environment.

---

## Development and fullstack (dev) box prerequisites:
 - Vagrant
 - Virtualbox

### Install centos 7 virtualbox image
    vagrant box add centos/7 https://atlas.hashicorp.com/centos/boxes/7

On Windows, you might be given a choice between libvirt or virtualbox. Choose *virtualbox*.

### Check that it has installed
    vagrant box list

and you should see 'centos/7' listed

### Clone this repo
    git clone git@bitbucket.org:uwlib/druw-infrastructure.git
    cd druw-infrastructure

### Copy vars.yml.template to vars.yml.
    cp vars.yml.template vars.yml

Edit application_home if you want it to install in someplace other than /home/vagrant/sufia   

### Start your vagrant box
    vagrant up --provider virtualbox

### ssh into vagrant box
    vagrant ssh

### scp your bitbucket private key into .ssh dir
    scp [yourbitbucketprivatekey] ~/.ssh

If the git clone below doesn't work, you might need to do either of the following:

1. Rename your private key to `id_rsa`

2. Change its permissions: `chmod 600 [yourprivatekey]`

### Clone the [druw repo](https://bitbucket.org/uwlib/druw) into wherever you specified application_home to be in vars.yml (Eg. if building a fullstack environment, change to "/var/druw")  
    cd ~   
    git clone git@bitbucket.org:uwlib/druw.git

### cd to ~/druw/config, then copy all *.yml.template files to *.yml.
    cd ~/druw/config   
    for f in `ls *.yml.template |rev | cut -d '.' --complement -f 1 |rev`; do cp $f{.template,}; done

### Copy config/initializers/devise.rb.template to config/initializers/devise.rb
    cp ~/druw/config/initializers/devise.rb.template ~/druw/config/initializers/devise.rb

---

## For development environment

### Change to vagrant sync dir and run ansible playbook for development.yml
    cd /vagrant   
    ansible-playbook -i inventory development.yml

### Start Up DRUW for the First Time

 You will have to start the following commands manually. You will probably also have to hit enter to return your prompt after each service starts up.   

* `cd /home/vagrant/druw`

* Start development solr   
    `bundle exec solr_wrapper -d solr/config/ --collection_name hydra-development &`

* Start FCRepo - your fedora project instance   
    `bundle exec fcrepo_wrapper -p 8984 &`

* Create a default admin set. You only need to do this step ONCE when you first create your new VM:
    `rails hyrax:default_admin_set:create`

* Generate a work type. You only need to do this step ONCE when you first create your new VM:
    `rails generate hyrax:work Generic_Work`

* Start development rails server   
    `rails server -p 3000 -b 0.0.0.0`

When you start up DRUW in the future, you will only need to start up Solr, FCrepo, and Rails. You do NOT need to recreate the default admin set.

### Check DRUW (Hyrax) is Running
Open a browser and go to http://localhost:3000. The initial load will take a bit (you'll see activity in SSH window as the rails server processes the request).

---

## For fullstack environment

### Move druw repo to /var
    sudo mv druw /var

### Change to vagrant sync dir, copy/edit private.yml.template, and run ansible playbook for fullstack.yml
    cd /vagrant
    cp private.yml.template private.yml   
    ansible-playbook -i inventory fullstack.yml

Change the values in private.yml

### Start Up DRUW for the First Time

 You will have to start the following commands manually. You will probably also have to hit enter to return your prompt after each service starts up.   

* `cd /var/druw`

* Create a default admin set. You only need to do this step ONCE when you first create your new VM:
    `sudo rails hyrax:default_admin_set:create RAILS_ENV=production`

* Generate a work type. You only need to do this step ONCE when you first create your new VM:
    `sudo rails generate hyrax:work Generic_Work RAILS_ENV=production`

* Restart apache   
    `sudo systemctl restart httpd`

### Check DRUW (Hyrax) is Running
Open a browser and go to http://localhost:1080 (or whereever set your port 80). The initial load will take a bit.

---

## Create a User
Go to http://localhost:3000/users/sign_up and create a new user (you will be making this user an admin in the next step).

## Create admin user
Follow the instructions on the [Hyrax Management Guide](https://github.com/samvera/hyrax/wiki/Hyrax-Management-Guide) under 'Admin users'. https://github.com/samvera/hyrax/wiki/Making-Admin-Users-in-Hyrax

 - First create a user from your browser at localhost:3000
 - Open another terminal and vagrant ssh in: `vagrant ssh `
 - Go to your application_home: `cd [application_home]`
 - Note: the hydra-role-management gem has already been installed and the roles generated in the druw repository.
 - Start a rails console: `RAILS_ENV=development bundle exec rails c`
 - Search or scroll down to "Add an initial admin user via command-line" in that github page mentioned above.
 - Follow those directions.

---

## Build demo site

On a computer with ansible installed

### Clone druw-infrastructure onto a computer that has ansible installed on it.
    git clone git@bitbucket.org:uwlib/druw-infrastructure.git

### cd into checked out repo and copy/edit vars.yml.template and private.yml.template

 - ```cd druw-infrastructure```
 - ansible_target - change to demoserver
 - application_home - change it to /var/druw

On the demo server

### ssh onto demo server (ssh -A or copy bitbucket key over.)

### Clone the druw repo and move it /var/druw
    cd ~   
    git clone git@bitbucket.org:uwlib/druw.git
    sudo mv druw /var

### cd to /var/druw/config, then copy all *.yml.template files to *.yml.
    cd /var/druw/config   
    for f in `ls *.yml.template |rev | cut -d '.' --complement -f 1 |rev`; do cp $f{.template,}; done

Edit /var/druw/config/secrets.yml

 - In production stanza, add the line ```secret_key_base = 'your key just copied from the private.yml.template"```
 - After this entire thing is built you could generate a secret key if you wanted to replace this ```bundle exec rails secret```

### Copy /var/druw/config/initializers/devise.rb.template to /var/druw/config/initializers/devise.rb
    cp /var/druw/config/initializers/devise.rb.template /var/druw/config/initializers/devise.rb

Edit /var/druw/config/initializers/devise.rb

 - add the line ```config.secret_key = 'your key that you just copied from private.yml.template"```
 - After this entire thing is built you could generate a secret key if you wanted to replace this ```bundle exec rails secret```

On the computer with ansible installed

### Go to druw-infrastructure repo

Run ```ansible-playbook -i inventory --ask-become-pass fullstack.yml```