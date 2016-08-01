# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

$provision = <<SCRIPT
MYSQL_DB=vagrant
MYSQL_USER=vagrant
MYSQL_PASS=vagrant

apt-get update

debconf-set-selections <<< 'mysql-server-5.5 mysql-server/root_password password root'
debconf-set-selections <<< 'mysql-server-5.5 mysql-server/root_password_again password root'
apt-get -y install mysql-server-5.5 php5-mysql apache2 php5

# Configure php
sed -i 's/upload_max_filesize = .*/upload_max_filesize = 10M/' /etc/php5/apache2/php.ini

# If phpmyadmin does not exist, install it
if [ ! -f /etc/phpmyadmin/config.inc.php ];
then

    # Used debconf-get-selections to find out what questions will be asked
    # This command needs debconf-utils

    # Handy for debugging. clear answers phpmyadmin: echo PURGE | debconf-communicate phpmyadmin

    echo 'phpmyadmin phpmyadmin/dbconfig-install boolean false' | debconf-set-selections
    echo 'phpmyadmin phpmyadmin/reconfigure-webserver multiselect apache2' | debconf-set-selections

    echo 'phpmyadmin phpmyadmin/app-password-confirm password root' | debconf-set-selections
    echo 'phpmyadmin phpmyadmin/mysql/admin-pass password root' | debconf-set-selections
    echo 'phpmyadmin phpmyadmin/password-confirm password root' | debconf-set-selections
    echo 'phpmyadmin phpmyadmin/setup-password password root' | debconf-set-selections
    echo 'phpmyadmin phpmyadmin/database-type select mysql' | debconf-set-selections
    echo 'phpmyadmin phpmyadmin/mysql/app-pass password root' | debconf-set-selections

    echo 'dbconfig-common dbconfig-common/mysql/app-pass password root' | debconf-set-selections
    echo 'dbconfig-common dbconfig-common/mysql/app-pass password' | debconf-set-selections
    echo 'dbconfig-common dbconfig-common/password-confirm password root' | debconf-set-selections
    echo 'dbconfig-common dbconfig-common/app-password-confirm password root' | debconf-set-selections
    echo 'dbconfig-common dbconfig-common/app-password-confirm password root' | debconf-set-selections
    echo 'dbconfig-common dbconfig-common/password-confirm password root' | debconf-set-selections

    apt-get -y install phpmyadmin
fi

if [ ! -f /var/log/databasesetup ];
then
    echo "CREATE USER '$MYSQL_USER'@'localhost' IDENTIFIED BY '$MYSQL_PASS'" | mysql -uroot -proot
    echo "CREATE DATABASE $MYSQL_DB" | mysql -uroot -proot
    echo "GRANT ALL ON ${MYSQL_DB}.* TO '$MYSQL_USER'@'localhost'" | mysql -uroot -proot
    echo "flush privileges" | mysql -uroot -proot

    touch /var/log/databasesetup
fi

if [ ! -h /var/www ];
then
    rm -rf /var/www
    ln -fs /vagrant /var/www

    cat >/etc/apache2/sites-enabled/*default* <<EOL
    <VirtualHost *:80>
      DocumentRoot /var/www/web
      <Directory /var/www/web>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        SetEnv APPLICATION_ENV development
      </Directory>
    </VirtualHost>
EOL

    #Replace envvars with vagrant credentials
    rm -rf /etc/apache2/envvars

    cat >/etc/apache2/envvars <<EOL
    # envvars - default environment variables for apache2ctl

    # this won't be correct after changing uid
    unset HOME

    # for supporting multiple apache2 instances
    if [ "${APACHE_CONFDIR##/etc/apache2-}" != "${APACHE_CONFDIR}" ] ; then
            SUFFIX="-${APACHE_CONFDIR##/etc/apache2-}"
    else
            SUFFIX=
    fi

    # Since there is no sane way to get the parsed apache2 config in scripts, some
    # settings are defined via environment variables and then used in apache2ctl,
    # /etc/init.d/apache2, /etc/logrotate.d/apache2, etc.
    export APACHE_RUN_USER=vagrant
    export APACHE_RUN_GROUP=vagrant
    # temporary state file location. This might be changed to /run in Wheezy+1
    export APACHE_PID_FILE=/var/run/apache2/apache2$SUFFIX.pid
    export APACHE_RUN_DIR=/var/run/apache2$SUFFIX
    export APACHE_LOCK_DIR=/var/lock/apache2$SUFFIX
    # Only /var/log/apache2 is handled by /etc/logrotate.d/apache2.
    export APACHE_LOG_DIR=/var/log/apache2$SUFFIX

    ## The locale used by some modules like mod_dav
    export LANG=C
    ## Uncomment the following line to use the system default locale instead:
    #. /etc/default/locale

    export LANG

    ## The command to get the status for 'apache2ctl status'.
    ## Some packages providing 'www-browser' need '--dump' instead of '-dump'.
    #export APACHE_LYNX='www-browser -dump'

    ## If you need a higher file descriptor limit, uncomment and adjust the
    ## following line (default is 8192):
    #APACHE_ULIMIT_MAX_FILES='ulimit -n 65536'

    ## If you would like to pass arguments to the web server, add them below
    ## to the APACHE_ARGUMENTS environment.
    #export APACHE_ARGUMENTS=''

    ## Enable the debug mode for maintainer scripts.
    ## This will produce a verbose output on package installations of web server modules and web application
    ## installations which interact with Apache
    #export APACHE2_MAINTSCRIPT_DEBUG=1
EOL

    a2enmod rewrite
    service apache2 restart
fi

#install composer
if [ ! -f /usr/local/bin/composer ];
then
   php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
   php -r "if (hash_file('SHA384', 'composer-setup.php') === 'e115a8dc7871f15d853148a7fbac7da27d6c0030b848d9b3dc09e2a0388afed865e6a3d6b3c0fad45c48e2b5fc1196ae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
   php composer-setup.php
   php -r "unlink('composer-setup.php');"
   sudo mv composer.phar /usr/local/bin/composer
fi

#install git
if [ ! -f /usr/bin/git ];
then
   sudo apt-get install -y git
fi

#clean the mess
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get autoremove -y

SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "trusty64"
  config.vm.box_url = "http://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box"

  config.vm.provider "virtualbox" do |v|
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
  end

  config.vm.network :forwarded_port, guest: 80, host: 8080
  config.vm.network :private_network, ip:"192.168.70.8"

  #Change if the OS support NFS
  #config.vm.synced_folder(".", "/vagrant", :nfs => true)
  config.vm.synced_folder(".", "/vagrant")

  config.vm.provision :shell, :inline => $provision

  #Plugin cachier
  if Vagrant.has_plugin?("vagrant-cachier")
     config.cache.scope = :machine

     config.cache.synced_folder_opts = {
        type: :nfs,
        mount_options: ['rw', 'vers=3', 'tcp', 'nolock']
     }

     config.cache.enable :generic, {
        "cache"  => { cache_dir: "/vagrant/app/cache" },
        "logs"   => { cache_dir: "/vagrant/app/logs" },
        "vendor" => { cache_dir: "/vagrant/vendor" },
     }
  end


end
