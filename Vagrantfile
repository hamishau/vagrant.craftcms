Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.provider "virtualbox" do |v|
    v.memory = 8192
    v.cpus = 4
  end
  config.vm.hostname = "yourapp.local"
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network "private_network", ip: "192.168.56.10"
  config.vm.synced_folder ".", "/var/www/html",
    :owner => 'www-data',
    :group => 'www-data',
    :mount_options => ['dmode=755', 'fmode=644']
  config.vm.provision "shell", inline: <<-SHELL
    sudo apt update
    sudo apt upgrade -y
    sudo apt install apache2 -y
    sudo apt install mysql-server -y
    sudo apt install php libapache2-mod-php php-mysql php-curl php-imagick php-mbstring php-dom php-zip php-bcmath php-intl npm php-cli unzip -y
    sudo a2enmod rewrite
    cd ~
    curl -sS https://getcomposer.org/installer -o /tmp/composer-setup.php
    HASH=`curl -sS https://composer.github.io/installer.sig`
    php -r "if (hash_file('SHA384', '/tmp/composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
    sudo php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer
    rm /var/www/html/index.html
  SHELL
end
