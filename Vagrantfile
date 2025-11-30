Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-24.04"

  config.vm.provider "virtualbox" do |v|
    v.memory = 8192
    v.cpus = 4
  end

  config.vm.hostname = "yourapp.local"
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network "private_network", ip: "192.168.56.10"

  config.vm.synced_folder ".", "/var/www/html",
    owner: "www-data",
    group: "www-data",
    mount_options: ["dmode=755", "fmode=644"]

  config.vm.provision "shell", privileged: true, inline: <<-'SHELL'
    set -euo pipefail
    set -x
    export DEBIAN_FRONTEND=noninteractive
    CRAFT_DIR="/var/www/html/craft"
    ADMIN_EMAIL="${ADMIN_EMAIL:-admin@yourapp.local}"
    ADMIN_USER="${ADMIN_USER:-admin}"
    ADMIN_PASS="${ADMIN_PASS:-admin123}"
    SITE_NAME="${SITE_NAME:-Your App}"
    SITE_URL="${SITE_URL:-http://yourapp.local:8080}"

    apt-get update
    apt-get dist-upgrade -y
    apt-get install -y --no-install-recommends \
      apache2 \
      postgresql postgresql-contrib \
      php libapache2-mod-php php-pgsql php-curl php-imagick php-mbstring php-xml php-zip php-bcmath php-intl php-cli php-apcu \
      npm unzip curl git ca-certificates openssl

    systemctl enable apache2
    a2enmod rewrite headers expires

    sudo -u postgres psql -tAc "SELECT 1 FROM pg_database WHERE datname='yourapp'" | grep -q 1 || sudo -u postgres createdb yourapp
    sudo -u postgres psql -c "ALTER USER postgres WITH PASSWORD 'admin';"

    if ! command -v composer >/dev/null 2>&1; then
      curl -sS https://getcomposer.org/installer -o /tmp/composer-setup.php
      HASH=$(curl -sS https://composer.github.io/installer.sig)
      php -r "if (hash_file('SHA384', '/tmp/composer-setup.php') !== '$HASH') exit(1);"
      php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer
      rm -f /tmp/composer-setup.php
    fi

    if [ ! -d "$CRAFT_DIR" ]; then
      COMPOSER_ALLOW_SUPERUSER=1 composer create-project craftcms/craft "$CRAFT_DIR" --no-interaction --no-ansi
      chown -R www-data:www-data "$CRAFT_DIR"
    fi

    if [ -d "$CRAFT_DIR" ]; then
      # Preserve existing app/security keys if present, otherwise generate.
      if [ -f "$CRAFT_DIR/.env" ]; then
        # shellcheck disable=SC1090
        . "$CRAFT_DIR/.env" || true
      fi
      CRAFT_APP_ID=${CRAFT_APP_ID:-"CraftCMS-$(cat /proc/sys/kernel/random/uuid 2>/dev/null || openssl rand -hex 12)"}
      CRAFT_SECURITY_KEY=${CRAFT_SECURITY_KEY:-$(openssl rand -hex 32)}

      cat > "$CRAFT_DIR/.env" <<EOF
DB_DRIVER=pgsql
DB_SERVER=127.0.0.1
DB_PORT=5432
DB_DATABASE=yourapp
DB_USER=postgres
DB_PASSWORD=admin
CRAFT_APP_ID=${CRAFT_APP_ID}
CRAFT_ENVIRONMENT=dev
CRAFT_SECURITY_KEY=${CRAFT_SECURITY_KEY}
CRAFT_DEV_MODE=true
CRAFT_ALLOW_ADMIN_CHANGES=true
CRAFT_DISALLOW_ROBOTS=true
EOF
      chown www-data:www-data "$CRAFT_DIR/.env"

      if [ ! -f "$CRAFT_DIR/config/db.php" ]; then
        cat > "$CRAFT_DIR/config/db.php" <<'EOF'
<?php
use craft\helpers\App;

return [
    'driver' => App::env('DB_DRIVER'),
    'server' => App::env('DB_SERVER'),
    'port' => App::env('DB_PORT'),
    'database' => App::env('DB_DATABASE'),
    'user' => App::env('DB_USER'),
    'password' => App::env('DB_PASSWORD'),
    'schema' => App::env('DB_SCHEMA') ?: 'public',
    'tablePrefix' => App::env('DB_TABLE_PREFIX') ?: '',
];
EOF
        chown www-data:www-data "$CRAFT_DIR/config/db.php"
      fi
    fi

    if [ -d "$CRAFT_DIR" ]; then
      cd "$CRAFT_DIR"
      rm -f "$CRAFT_DIR/.provisioned-install"
      if ! sudo -u www-data php craft install/check --interactive=0 >/dev/null 2>&1; then
        sudo -u www-data php craft install --interactive=0 \
          --email="$ADMIN_EMAIL" \
          --username="$ADMIN_USER" \
          --password="$ADMIN_PASS" \
          --site-name="$SITE_NAME" \
          --site-url="$SITE_URL" \
          --language="en-US"
      fi
      if sudo -u www-data php craft install/check --interactive=0 >/dev/null 2>&1; then
        touch "$CRAFT_DIR/.provisioned-install"
        chown www-data:www-data "$CRAFT_DIR/.provisioned-install"
      else
        echo "Craft install did not complete; check database credentials and logs." >&2
        exit 1
      fi
    fi

    cp /var/www/html/000-default.conf /etc/apache2/sites-available/000-default.conf
    rm -f /var/www/html/index.html
    systemctl reload apache2
  SHELL
end
