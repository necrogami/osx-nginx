Install PHP + Nginx(SSL+HTTP2) + MySQL on macOS
====================================

WORK IN PROGRESS -- This is not complete
====================================

  
* [Requirements](#requirements)
  
* [Homebrew](#homebrew)

* [PHP-FPM](#php-fpm)
    
* [MySQL](#mysql)
   
* [Nginx](#nginx)
  
* [Go](#go)

* [EasyPKI](#easypki)
  
* [More commands](#more-commands)
  
* [Credits](#credits)

## Requirements

* An Intel CPU
  
* OS X 10.10 or higher

* Command Line Tools (CLT) for Xcode: `xcode-select --install`, [developer.apple.com/downloads][1] or [Xcode][2]

* A Bourne-compatible shell for installation (e.g. bash or zsh)

## Homebrew

**Homebrew** - is a package manager for macOS like `apt` for Debian.

To install Homebrew run this command in the terminal:

    $ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
    
Run `brew doctor` and fix all the warnings (outdated Xcode/CLT and unbrewed dylibs are very likely to cause problems) if they will be.

If you already had Homebrew installed, update the existing Homebrew installation as well as the installed packages:

    $ brew update && brew upgrade

[back to topics][0]

## PHP-FPM

Now you can install php:

    $ brew install php
    
Update the `$PATH` environment variable, if you want to use the PHP CLI:

    $ echo 'export PATH="/usr/local/sbin:/usr/local/bin:$PATH"c
    
Use brew services to start or stop the php service:

    $ brew services start php
    $ brew services stop php
    
You can run the service one time if needed:

    $ brew services run php

[back to topics][0]

## MySQL

To install MySQL run this command:

    $ brew install mysql@5.7
    
Note: At this point in time (php@7.2) there is a problem with connecting to MySQL 8 with PHP7.2

Update the `$PATH` environment variable, if you want to use the MySQL CLI:

    $ echo 'export PATH="/usr/local/opt/mysql@5.7/bin:$PATH"' >> ~/.bash_profile && source ~/.bash_profile
     
To start and stop the mysql service:

    $ brew services start mysql@5.7
    $ brew services stop mysql@5.7

Run the MySQL-server:

    $ brew services run mysql@5.7
    
If that fails to run:

    $ mysql.server start
     
To secure your MySQL server you should execute the provided `secure_mysql_installation` binary to change the root password:

    $ mysql_secure_installation

    > Enter current password for root (enter for none):

Press `Enter` since you don't have one set.

    > Change the root password? [Y/n]
     
It's up to you ;)

    > Remove anonymous users? [Y/n]

They are not necessary, so press `Enter`.

    > Disallow root login remotely? [Y/n]

`Enter` — No need to log in as root from any other IP than 127.0.0.1.

    > Remove test database and access to it? [Y/n]

`Enter` — You don't need the testing tables.

    > Reload privilege tables now? [Y/n]

`Enter` — Reload the privilege tables to ensure all of the changes made so far will take effect immediately.

When you done, test mysql:

    $ mysql -u root -p
    > Enter you password

[back to topics][0]

## Nginx

To install Nginx run this command:

    $ brew install nginx
    
To setup auto start/stop:

    $ sudo brew services start nginx
    $ sudo brew services stop nginx
   
Why `sudo`? Because only the root user is allowed to open ports which are < 1024.

Launch the server and test connection:

    $ sudo brew services run nginx
    
Next you need to create few folders, which are necessary for the nginx configuration:

    $ mkdir -p /usr/local/etc/nginx/logs
    $ mkdir -p /usr/local/etc/nginx/sites-available
    $ mkdir -p /usr/local/etc/nginx/sites-enabled
    $ mkdir -p /usr/local/etc/nginx/conf.d
    $ mkdir -p /usr/local/etc/nginx/ssl

The way i store my projects. Is to store them inside separate directory in `$HOME` directory:

    $ mkdir ~/code
    $ cd ~/code
    
I choose the first way, but it's up to you.

Replace the default `nginx.conf` with my custom config from GitHubGist:

    $ sudo mv /usr/local/etc/nginx/nginx.conf /usr/local/etc/nginx/nginx.conf.old
    $ curl -L https://raw.githubusercontent.com/necrogami/osx-nginx/master/nginx.conf -o /usr/local/etc/nginx/nginx.conf

    
And the last step is to setup virtual host. 
You could do it by your self, or you could download my custom virtual host config and setup it:

    $ curl -L  -o /usr/local/etc/nginx/sites-available/test
    $ ln -sfv /usr/local/etc/nginx/sites-available/test /usr/local/etc/nginx/sites-enabled/
    
But don't forget to change the path to the root directory of your host and also to add `test.php` to the root.

After finishing you should restart nginx:

    $ sudo brew services restart nginx

[back to topics][0]

## Go

Install GoLang from homebrew

    $ brew install go

You will need to setup gopath I use a folder `gocode` inside my code folder for ease of use.

    $ echo 'export GOPATH=$HOME/code/gocode' >> ~/.bash_profile && source ~/.bash_profile
    $ echo 'export PATH=$PATH:$GOPATH/bin' >> ~/.bash_profile && source ~/.bash_profile

[back to topics][0]

## EasyPKI

EasyPKI is a library created by google to allow you to create a set of self signed SSL Certificates. This will allow you to generate a full Certificate Authority and be able to manage website SSL for all your projects.

Get the CLI:

    $ go get github.com/google/easypki/cmd/easypki

Now you will want to setup some exports for using easypki:

    $ export PKI_ROOT=~/code/pki
    $ export PKI_ORGANIZATION="Local Inc."
    $ export PKI_ORGANIZATIONAL_UNIT=IT
    $ export PKI_COUNTRY=US
    $ export PKI_LOCALITY="New York"
    $ export PKI_PROVINCE="New York"

    easypki create --expire 10950 --filename root --ca "Local Inc. Certificate Authority"
    easypki create --expire 5475 --ca-name root --filename intermediate --intermediate "Local Inc. - Internal CA"
    easypki create --expire 1825 --ca-name intermediate --dns "*.local.dev" "*.local.dev"

    $ cat ~/code/pki/intermediate/certs/wildcard.local.dev.crt >> /usr/local/etc/nginx/ssl/wildcard.local.dev.pem
    $ cat ~/code/pki/intermediate/certs/intermediate.crt >> /usr/local/etc/nginx/ssl/wildcard.local.dev.pem
    $ cat ~/code/pki/root/certs/root.crt >> /usr/local/etc/nginx/ssl/wildcard.local.dev.pem
    $ cat ~/code/pki/intermediate/keys/wildcard.local.dev.key >> /usr/local/etc/nginx/ssl/wildcard.local.dev.pem
    $ openssl dhparam -out /usr/local/etc/nginx/ssl/dhparam.pem 4096

Open `~/code/pki/root/certs/root.crt` it will ask you which you want to add it to. Add it to the system keychain.
Provide your password.
On the left click on system under keychains.
You should see your `Local Inc. Certificate Authority` cert with a red x on it.
Double click on that cert. 
Expand Trust
Under `When using this certificate: ` select `Always Trust`
Close the window.
Re-Enter your password.

Now any certificate's you generate off that root certificate will be trusted by your system.


## DNSMasq

    $ brew install dnsmasq

    $ echo "address=/dev/127.0.0.1" >> /usr/local/etc/dnsmasq.conf

    $ sudo tee /etc/resolver/dev >/dev/null <<EOF
      nameserver 127.0.0.1
      EOF

    $ sudo brew services start dnsmasq


## Finish Configuartion




## More commands

* `brew install [package]` - 

* `brew list` - Show the list of all installed packages

* `brew search [search term]` - List the possible packages that you can install

* `brew info [package]` - Display some basic information about the package

* `brew remove [package]` - Remove package

* `brew update` - Self-update

* `brew upgrade [package]` - Update package

* `brew doctor` - Self-diagnose

* `brew tap [repo]` - Add new Brew repository

* `brew help` - List Brew commands

* `sudo brew services start nginx` - Start Nginx service

* `sudo brew services stop nginx` - Stop Nginx service

* `sudo brew services restart nginx` - Restart Nginx service

* `brew services start php` - Start PHP-FPM service

* `brew services stop php` - Stop PHP-FPM service

* `brew services restart php` - Restart PHP-FPM service

* `brew services start mysql@5.7` - Start MySQL server

* `brew services stop mysql@5.7` - Stop MySQL server

* `brew services restart mysql@5.7` - Retstart MySQL server

[back to topics][0]

## Credits

Great thanks to [@frdmn][3] who inspired me to write my own guide.

[back to topics][0]

[0]: #install-php--nginxsslhttp2--mysql-on-macos
[1]: https://developer.apple.com/downloads
[2]: https://itunes.apple.com/us/app/xcode/id497799835
[3]: https://github.com/frdmn
