#Server Setup

This is a reminder for me on how to set up a webserver with Linux NginX Php Ruby mySQL

##Initial Setup

**1. Change Root Password**
~~~ sh
$ passwd root
~~~

**2. Create Admin User**
~~~ sh
$ adduser newUserName
~~~

**3. Add new User to Sudoers**
~~~ sh
$ visudo
~~~
* **Find** ```root   ALL=(ALL:ALL) ALL``` **and add the following underneath**
    
        newUserName   ALL=(ALL:ALL) ALL

**4. Reconfig SSH**
~~~ sh
$ nano /etc/ssh/sshd_config
~~~
* **Change Port**

        Port NewPortNumber 

* **Disable Root Login**

        PermitRootLogin no
        
* **Only Allow specific users**

        UseDNS no
        AllowUsers newUserName

**5. Reset Timezone**
~~~ sh
$ dpkg-reconfigure tzdata
~~~

**6. Update System**
~~~ sh
$ apt-get update
~~~
~~~ sh
$ apt-get upgrade
~~~
~~~ sh
$ apt-get dist-upgrade
~~~

##System Installs
**1. Install and Config Firewall**
~~~ sh
$ apt-get install ufw
~~~
~~~ sh
$ ufw default deny incoming
~~~
~~~ sh
$ ufw default allow outgoing
~~~
~~~ sh
$ ufw allow 80/tcp
~~~
~~~ sh
$ ufw allow 443/tcp
~~~
~~~ sh
$ ufw allow youSSHPortNumber/tcp
~~~
~~~ sh
$ ufw enable
~~~

**2. Install and Config PostFix**
* **Install Postfix**
~~~ sh
$ apt-get installpostfix mailutils
~~~

* **Edit Postfix config file**
~~~ sh
$ nano /etc/postfix/main.cf
~~~

  **Change** `myhostname = localhost` **to**
  ~~~
  myhostname = YourServersHostName ie. mail.exampleserver.com 
  ~~~

* **Restart Postfix**
~~~ sh
$ service postfix restart
~~~

**3. Install and Config Fail2Ban**
* **Install Fail2Ban**
~~~ sh 
$ apt-get install fail2ban
~~~

* **Edit Config File**
  First copy jail.conf to jail.local
  ~~~ sh 
  $ cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
  ~~~

  Edit jail.local
  ~~~ sh 
  $ nano /etc/fail2ban/jail.local
  ~~~
          
  Change Destemail
  ~~~
  destemail = your@email.com
  ~~~
  
  Change Action change `action = %(action_)s to 
  ~~~
  action = %(action_mwl)s
  ~~~
* **Restart Service**
  ~~~ sh      
  $ service fail2ban restart
  ~~~

##Install Application
* **1. Install curl, git, and python-software-properties**
  ~~~ sh
  $ apt-get install curl git-core python-software-properties
  ~~~
* **2. Install NginX**
  To get monst current version add the NginX repo
  ~~~ sh
  $ add-apt-repository ppa:nginx/stable
  ~~~

  Update apt
  ~~~ sh
  $ apt-get update
  ~~~
  
  Install NginX
  ~~~ sh
  $ apt-get install nginx
  ~~~

* **3. Install Php5-fpm**
  ~~~ sh
  $ apt-get install php5-fpm
  ~~~    

* **4. Install MySQL**
  ~~~ sh
  $ apt-get install mysql-server
  ~~~
  
  Run MySQL Secure Installation
  ~~~ sh
  $ mysql_secure_installation
  ~~~

##Configure Php5-fpm  
* **Configure php.ini**
    **cgi.fix_pathinfo**
    
    Leaving at 1 is a big security flaw for nginx and php. Read about it under "Passing Uncontrolled Requests to Php" in http://wiki.nginx.org/Pitfalls
  **Change** `;cgi.fix_pathinfo=1` **to**
  ~~~
  cgi.fix_pathinfo=0
  ~~~
      
* **Change php listen to unix socket**
  **Open `www.conf`**
    ~~~ sh 
    $ nano /etc/php5/fpm/pool.d/www.conf
    ~~~
  
  **Change** `listen = 127.0.0.1:9000` **to**
    ~~~
    listen = /var/run/php5-fpm.sock
    ~~~
* **Restart Php5-fpm**
  ~~~ sh
  $ service php5-fpm restart
  ~~~

##Configure NginX
**For a php site add the following server block to the sites-available folder** 

https://www.digitalocean.com/community/articles/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-12-04
~~~
server {
  listen   80;     
  root /usr/share/nginx/www;
  index index.php index.html index.htm;

  server_name example.com;

  location / {
    try_files $uri $uri/ /index.html;
  }

  error_page 404 /404.html;

  error_page 500 502 503 504 /50x.html;
  location = /50x.html {
    root /usr/share/nginx/www;
  }

  # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
  location ~ \.php$ {
    #fastcgi_pass 127.0.0.1:9000; #Not used b/c we are using Unix Socket
    # With php5-fpm:
    fastcgi_pass unix:/var/run/php5-fpm.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;                
  }
}
~~~

**For a Ruby Site add the following server block** 

http://codetunes.com/2012/nginxunicorn-configuration-for-multi-app-servers/
~~~
upstream unicorn-appname { 
  server unix:/tmp/unicorn.appname.sock; 
}
            
server {
  listen 80;
  server_name appname.com;
  root /var/www/appname/current/public;
            
  location / {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Real-IP  $remote_addr;
    proxy_set_header Host $http_host;
    proxy_redirect off;
            
    if (!-f $request_filename) {
      proxy_pass http://unicorn-appname;
      break;
    }
  }
}
~~~
##Install Rbenv and Ruby
Rbenv and gems install to a users home directory. So Create the user you want to use Ruby with and switch to that user...
* **1. Install Ruby Dependancies**
  ~~~ sh
  $ sudo apt-get install build-essential openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison nodejs subversion
  ~~~

* **2. Install Rbenv** Below is Installation as of 7/10/13 Install As per https://github.com/sstephenson/rbenv#installation
  ~~~ sh
  $ git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
  ~~~

* **3. Add ~/.rbenv/bin to your $PATH for access to the rbenv command-line utility.**
  ~~~ sh
  $ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.profile
  ~~~

* **4. Add rbenv init to your shell to enable shims and autocompletion.***
  ~~~ sh
  $ echo 'eval "$(rbenv init -)"' >> ~/.profile
  ~~~

* **5. Restart your shell as a login shell so the path changes take effect. You can now begin using rbenv.**
  ~~~ sh
  $ exec $SHELL -l
  ~~~

* **6. Install Ruby-Build** Below is installation as of 7/10/13 Install as per https://github.com/sstephenson/ruby-build
  ~~~ sh
  $ git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
  ~~~

* **7. Install a version of Ruby**
  ~~~ sh
  $ rbenv install 2.0.0-p247
  ~~~

* **8. Rehash**
  ~~~ sh
  $ rbenv rehash
  ~~~

* **9.Set Default Ruby Version**
  ~~~ sh
  $ rbenv global 2.0.0-p247
  ~~~

* **10. Update Ruby Gems**
  ~~~ sh
  $ gem update --system
  ~~~

* **11. Install Bundler**
  ~~~ sh
  $ gem install bundler
  ~~~

* **12. Rehash**
  ~~~ sh
  $ rbenv rehash
  ~~~

* **13. Install Rails**
  ~~~ sh
  $ gem install rails
  ~~~
  
  I have been having to do this b/c docs won't install properly...
  ~~~ sh
  $ gem install rails --no-ri --no-rdoc
  ~~~

##Unicorn
* **1. Add Unicorn to Gemfile**
  ~~~
  gem 'unicorn'
  ~~~

* **2. Create unicorn config file in** `/config/unicorn.rb`
  ~~~
  # The rule of thumb is to use 1 worker per processor core available,
  # however since we'll be hosting many apps on this server,
  # we need to take a less aggressive approach
  worker_processes 1
            
  working_directory "/var/www/appname/current"
      
  # Listen on unix socket
  listen "/tmp/unicorn.appname.sock", :backlog => 64
            
  pid "/var/www/appname/current/tmp/pids/unicorn.pid"
            
  stderr_path "/var/www/appname/current/log/unicorn.log"
  stdout_path "/var/www/appname/current/log/unicorn.log"
  ~~~

* **3. Add our pids folder**
  ~~~ sh
  $ mkdir tmp/pids
  ~~~

* **4. We can start the Unicorn Server with**
  ~~~ sh
  $ bundle exec unicorn -D -c /var/www/appname/current/config/unicorn.rb
  ~~~

* **5. Unicorn init.d Script
  ~~~ 
#!/bin/sh
  set -e
            
  # Feel free to change any of the following variables for your app:
  TIMEOUT=${TIMEOUT-60}
  APP_ROOT=/home/ptadmin/ruby.purdytest.com
  APP_USER=ptadmin
  PID=$APP_ROOT/tmp/pids/unicorn.pid
  ENV=development
  CMD="cd $APP_ROOT && bundle exec unicorn_rails -E $ENV -D -c $APP_ROOT/config/unicorn.rb"
  action="$1"
  set -u
            
  old_pid="$PID.oldbin"
            
  cd $APP_ROOT || exit 1
            
  sig () {
    test -s "$PID" && kill -$1 `cat $PID`
  }
            
  oldsig () {
    test -s $old_pid && kill -$1 `cat $old_pid`
  }
            
  case $action in
  start)
      sig 0 && echo >&2 "Already running" && exit 0
      su --login $APP_USER -c "$CMD"
      ;;
  stop)
      sig QUIT && exit 0
      echo >&2 "Not running"
      ;;
  force-stop)
      sig TERM && exit 0          
      echo >&2 "Not running"
      ;;
  restart|reload)
      sig HUP && echo reloaded OK && exit 0
      echo >&2 "Couldn't reload, starting '$CMD' instead"
      su --login $APP_USER -c "$CMD"
      ;;
  upgrade)
      if sig USR2 && sleep 2 && sig 0 && oldsig QUIT
      then
        n=$TIMEOUT
        while test -s $old_pid && test $n -ge 0
        do
          printf '.' && sleep 1 && n=$(( $n - 1 ))
        done
        echo
          
        if test $n -lt 0 && test -s $old_pid
        then
          echo >&2 "$old_pid still exists after $TIMEOUT seconds"
          exit 1
        fi
        exit 0
      fi
      echo >&2 "Couldn't upgrade, starting '$CMD' instead"
      su --login $APP_USER -c "$CMD"
      ;;
  reopen-logs)
    sig USR1
    ;;
  *)
    echo >&2 "Usage: $0 "
    exit 1
    ;;
  esac
  ~~~

* **Make File Executable**
  ~~~ sh
  $ chmod +x /etc/init.d/unicorn.appname
  ~~~

* **Now we can use the following Commands**
  ~~~ sh
  $ service unicorn.appname start
  $ service unicorn.appname stop
  $ service unicorn.appname restart
  ~~~

* **Make run on Startup**
  ~~~ sh
  $ update-rc.d blah defaults
  ~~~

##Done!
**Following links are helpful**
  * LEMP and Nginx Php config: https://www.digitalocean.com/community/articles/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-12-04
  * Used the Ruby dependancies: https://www.digitalocean.com/community/articles/how-to-install-ruby-on-rails-on-ubuntu-12-04-from-source
  * Rbenv: https://www.digitalocean.com/community/articles/how-to-install-ruby-on-rails-on-ubuntu-12-04-lts-with-rbenv--2
  * Rbenv Install: https://github.com/sstephenson/rbenv
  * Ruby-Build Install: https://github.com/sstephenson/ruby-build
  * MultiServer Unicorn & NginX config: http://codetunes.com/2012/nginxunicorn-configuration-for-multi-app-servers/
  * Run Script on Startup: http://www.debian-administration.org/articles/28
  
