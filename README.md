# Custom Vagrantfile to use with Symfony2

This Vagrantfile assures you that your Symfony2 application will run smoothly and not with a lot of delay where used in normal situations.

For this to work you need to install the vagrant-cachier plugin

```vagrant plugin install vagrant-cachier```

After it just enter the directory where the project and the Vagrantfile are and run ```vagrant up```

Wait a little while the setup installs all of the dependencies.

Access the ip you've defined on the Vagrantfile. If you didn't define one, the default is [192.168.70.8](http://192.168.70.8)

What's inside?
--------------

 [1]Ubuntu 14.04.4 LTS (Trusty 64)
 [2]PHP 5.5.9
 [3]MySQL 14.14
 [4]Custom DB, USER and PASS and auto run migration
 [5]PHPMyAdmin (access: ip/phpmyadmin -u root -p root)
 [6]Vagrant Cachier to speed up your symfony application
 [7]Optional IP Address
 [8]Document root
 [9]Composer
[10]Git

##Enjoy!!
