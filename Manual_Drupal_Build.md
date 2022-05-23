# Installing a clean latest Drupal build on a local environment:

### Before you begin
  
  - usernames and passwords can be chosen by you, this document uses "fossa" as the user
  - sudo commands may require a password, which you would have set up 
  - use good passwords or store good ones in plaintext
  - if you are using vsphere you will need to know those login credentials
  - if you have a cleanly built ubuntu 20.04 to build on, this machine can be anywhere you want, just be safe and smart
  - nano is included in this build
  - you need access to lsulibraries github (https://github.com/lsulibraries/)
  - all exceutable steps intended for command line execution will be visible in this document in:
  - ```"text like this"```

---
 **Make sure the VM is always shut down properly. This means either hitting 'Pause'(suspend) in the VMWare UI or shutting down inside Ubuntu. Failing to do so may risk machine corruption.**

---

### Initial Setup
**Getting VMWare**
 - Download VMWare Workstation Pro 16 from Tigerware “onthehub.com” It is free for Faculty and staff 
 - When prompted by setup, make sure "Add VMWare tools to system path option" is checked 
 - If you're using linux be aware of this https://github.com/mkubecek/vmware-host-modules
 - Save the serial number (write it down). You will need it to verify your version of VMWare when you start it at a later step.
 - Download Ubuntu 20.04 LTS ISO from https://ubuntu.com/download/server 
 - Enter the license when prompted; this was included in the order details you received after downloading VMWare from onthehub.com

 ---
 **Creating a Virtual Machine**
 - Click 'File' &rightarrow; 'New Virtual Machine...'
 - Select 'Typical' configuration &rightarrow; Enter the Ubuntu ISO path
 - 'Full name' and 'user name' are typically 'fossa', password is up to you
 - 'Virtual machine name' is typically dev(yourname) all lowercase
 - Select 20GB, split option

 **Initial settings**
 - Once setup completes you will be prompted with the following:
    - Language &rightarrow;  English &rightarrow; Done
    - Network Connections  &rightarrow; verify you have a connection  &rightarrow; Done
    - Configure proxy  &rightarrow; leave blank  &rightarrow; Done
    - Configure Ubuntu archive mirror  &rightarrow; do nothing  &rightarrow;  Done
    - Guided storage configuration  &rightarrow; use default (use an entire disk, set up as LVM)  &rightarrow; Done
    - File system summary  &rightarrow; Done  &rightarrow; Continue
    - Enter name, server name(machine name), username and password  &rightarrow; Done
    - Hit enter to enable 'Install OpenSSH server'  &rightarrow; Done
    - Featured Server snaps  &rightarrow; dont check anything  &rightarrow; Done
    - Ubuntu will now install, wait for 'Install Complete' banner at the top and then select 'Reboot' option
    - press 'Enter' to bring up login prompt after boot sequence

---
## Configuring Ubuntu

This is when we will input the following commands:
- ```sudo apt update```
- ```sudo apt upgrade -y``` (this may take some time)
- ```sudo add-apt-repository ppa:ondrej/php -y```
- ```sudo apt update```
- ```sudo apt upgrade -y```
- ```sudo apt install -y git php7.4 php7.4-curl php7.4-gd php7.4-xml php7.4-mysql php7.4-mbstring zip unzip php-zip mariadb-server npm```
- ```php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"```
- ```php -r "if (hash_file('sha384', 'composer-setup.php') === '756890a4488ce9024fc62c56153228907f1545c228516cbf63f885e036d37e9a59d27d63f46af1d4d07ee0f76181c7d3') { echo 'Installer verified'; } else { echo  'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"```
- ```php composer-setup.php```
- ```php -r "unlink('composer-setup.php');"```
- ```sudo mv composer.phar /usr/local/bin/composer```
- ```sudo chown fossa:root /var/www/html```
- ```cd /var/www/html/```
- ```composer create-project drupal/recommended-project drupal_site```

---
### Setting up files and settings
- ```cd drupal_site/web/sites/default```
- ```mkdir files```
- ```chmod 777 files```
- ```cp default.settings.php settings.php```
- ```chmod 777 settings.php```

---
## Securing the database
- ```sudo mysql_secure_installation```
- follow the prompts, disallowing and setting passwords yes to all options
  - ```sudo mysql -u root -p``` enter your password
    - ```alter user root@localhost identified by '<enter-a-new-pass>';```
    - ```flush privileges;```
    - ```exit;``` Now sudo is no longer required to login
  - ```mysql -u root -p``` enter your new password
    - ```create database drupal;```
    - ```CREATE USER drupalDBuser@localhost IDENTIFIED BY '<createpassword>';```
    - ```GRANT ALL ON *.* TO drupalDBuser@localhost IDENTIFIED BY '<yourcreatedpassword>';```
    - ```FLUSH PRIVILEGES;```
    - ```exit;```

---
### Configure Apache

- ```sudo nano /etc/apache2/apache2.conf```

- Replace AllowOveride 'None' with 'All'. This change is towards the bottom of the file the chunk will look like this. 
 
  >```
  >  <Directory /var/www/>
  >          Options Indexes FollowSymLinks
  >          AllowOverride All
  >          Require all granted
  >  </Directory>
  >```

you're changing the third AllowOverride from None to All (CTRL+O saves from nano CTRL+X to close )

- ```sudo nano /etc/apache2/sites-available/000-default.conf```

  >```
  >  DocumentRoot /var/www/html/drupal_site/web
  >```
  > you're adding /drupal_site/web to the DocumentRoot line after /html

- ```sudo a2enmod rewrite```
- ```sudo systemctl restart apache2``` 


---
## Finalizing the network for host use
 **Virtual Network Setup**
 - Right click your virtual machine in the left pane of VMWare  &rightarrow; 'Settings...' &rightarrow; 'Network Adapter'  &rightarrow; change from NAT to Bridged (you will now be disconnected from the network)
 - Click top menu 'Edit' -> 'Virtual Network Editor' -> Highlight VMNet0 and click on "Automatic Settings" -> You will see a list of adapters. **De-select all but the physical network card**. 
    - Click 'Ok' -> click 'Ok' 
  
 - Proceed with "LSU Device Registration" steps below

---
 **LSU Device Registration**
  - Right click your machine 'Settings' &rightarrow; Click 'Network Adapter' &rightarrow; 'Advanced...' to see your mac address
  - Send mac address to Carlos for lsu registration

---
## Installing Drupal
- Open a browser and go to the URL provided by Carlos or the machine IP address
- Select 'English' and 'Standard' installation
- Database name: Drupal
- Database username: drupalDBuser
- Database password: <password you set earlier>
- Enter 'LSU Libraries' for website name
- Enter a dummy email for admin email (you can change this later)
- Username: admin
- Password: <new password>
- Default country: United States
- Default timezone: Chicago

---
**Protect configuration files**
- ```cd /var/www/html/drupal_site/web/sites/default```
- ```chmod 0644 settings.php``` 
- ```cd ../```
- ```chmod 0755 default```
- ```sudo nano default/settings.php```
- Enter this on line 703 (or wherever you like):
  > ```
  >$settings['trusted_host_patterns'] = [
  >  '<urlprovided>$',
  >  '<yourvmIPaddress>$',
  >  '^localhost$',
  >];
  ```
---
**Install Drush**
- ```wget https://github.com/drush-ops/drush/releases/download/8.4.8/drush.phar```
- ```chmod +x drush.phar```
- ```sudo mv drush.phar /usr/local/bin/drush```

---
Congratulations, you now have a clean drupal build with zero errors.
