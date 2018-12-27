# Druw infrastructure

This repository sets a up a centos 7 based infrastructure for a research data repository based on samvera/hyrax (a Ruby/Rails application). It requires separately cloning the [druw repository](https://github.com/UW-Libraries/druw). It will build either a development or a fullstack (development) environment.

---

## Development and fullstack (dev) box prerequisites:
 - Vagrant
 - Virtualbox

### Install centos 7 virtualbox image
    vagrant box add centos/7 https://app.vagrantup.com/centos/boxes/7

On Windows, you might be given a choice between libvirt or virtualbox. Choose *virtualbox*.

### Check that it has installed
    vagrant box list

and you should see 'centos/7' listed

---

## For development environment
### Clone this repo
    git clone git@github.com:UW-Libraries/druw-infrastructure.git
    cd druw-infrastructure

### Copy *.yml.template to *.yml.
    cp vars.yml.template vars.yml
    cp private.yml.template private.yml

Edit application_home if you want it to install in someplace other than /home/vagrant/druw   

### Start your vagrant box
    vagrant up --provider virtualbox

### ssh into vagrant box
    vagrant ssh

### scp your github private key into guest OS .ssh dir (command will vary)
    scp [yourgithubprivatekey] ~/.ssh

If the git clone below doesn't work, you might need to do either of the following:

1. Rename your private key to `id_rsa`

2. Change its permissions: `chmod 600 id_rsa`

### Clone the [druw repo](https://github.com/UW-Libraries/druw) into wherever you specified application_home to be in vars.yml 
    cd ~   
    git clone git@github.com:UW-Libraries/druw.git

### cd to ~/druw/config, then copy all *.yml.template files to *.yml.
    cd ~/druw/config   
    for f in `ls *.yml.template |rev | cut -d '.' --complement -f 1 |rev`; do cp $f{.template,}; done

### Copy config/initializers/devise.rb.template to config/initializers/devise.rb
    cp ~/druw/config/initializers/devise.rb.template ~/druw/config/initializers/devise.rb

---

### Change to vagrant sync dir and run ansible playbook for development.yml
    cd /vagrant   
    ansible-playbook -i inventory development.yml

### Start Up DRUW for the First Time

 You will have to start the following commands manually. You will probably also have to hit enter to return your prompt after each service starts up.   

* `cd /home/vagrant/druw`

* Start development solr:  
    `bundle exec solr_wrapper -d solr/config/ --collection_name hydra-development &`

  * If solr 7 gives you problems, run solr 6 instead:  
    `bundle exec solr_wrapper -d solr/config/ --collection_name hydra-development --version 6.6.1 &`

* Start FCRepo - your fedora project instance:  
    `bundle exec fcrepo_wrapper -p 8984 &`

* Create a default admin set. You only need to do this step ONCE when you first create your new VM:  
    `rails hyrax:default_admin_set:create`

* Generate a work type. You only need to do this step ONCE when you first create your new VM:  
    `rails generate hyrax:work GenericWork`

* Start development rails server  
    `rails server -p 3000 -b 0.0.0.0`

When you start up DRUW in the future, you will only need to start up Solr, FCrepo, and Rails. You do NOT need to recreate the default admin set.

### Check DRUW (Hyrax) is Running
Open a browser and go to http://localhost:3000. The initial load will take a bit (you'll see activity in SSH window as the rails server processes the request).

---

## For fullstack environment

This will create an instance of the druw application that does not allow commits back to the repo.

### cd into /vagrant, copy/edit private.yml.template, copy/edit vars-full.yml.template, and run ansible playbook for fullstack.yml
    cd /vagrant   
    cp private.yml.template private.yml   
    cp vars-full.template vars-full.yml   
    ansible-playbook -i inventory fullstack.yml   

### cd to /var/druw/config, then copy all *.yml.template files to *.yml.
    cd /var/druw/config   
    for f in `ls *.yml.template |rev | cut -d '.' --complement -f 1 |rev`; do cp $f{.template,}; done

### Copy/edit /var/druw/config/initializers/devise.rb.template
    cp /var/druw/config/initializers/devise.rb.template /var/druw/config/initializers/devise.rb


### Start Up DRUW for the First Time

 Run the following commands.   

* `cd /var/druw`

* Create a default admin set. You only need to do this step ONCE when you first create your new VM:  
    `sudo rails hyrax:default_admin_set:create RAILS_ENV=production`

* Generate a work type. You only need to do this step ONCE when you first create your new VM:  
    `sudo rails generate hyrax:work Work RAILS_ENV=production`

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
    git clone git@github.com:UW-Libraries/druw-infrastructure.git

### cd into druw-infrastructure, copy/edit private.yml.template, copy/edit vars-full.yml.template (change ansible_target to demoserver), and run fullstack.yml playbook
    cd druw-infrastructure   
    cp private.yml.template private.yml   
    cp vars-full.template vars-full.yml
    ansible-playbook -i inventory --ask-become-pass fullstack.yml

---

On the demo server

### cd to /var/druw/config, then copy all *.yml.template files to *.yml.
    cd /var/druw/config   
    for f in `ls *.yml.template |rev | cut -d '.' --complement -f 1 |rev`; do cp $f{.template,}; done

### Copy/edit /var/druw/config/initializers/devise.rb.template
    cp /var/druw/config/initializers/devise.rb.template /var/druw/config/initializers/devise.rb

### restart httpd, troubleshoot, maybe re-run ansible playbook.
