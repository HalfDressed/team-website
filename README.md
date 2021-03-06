# Team Website

## Development instructions

1.  Have PHP installed
2.  Have SQLite installed
3.  Ask Michael for the `config_local.yml` to connect your development machine 
    to the database
    1.  After receiving the `config_local.yml` file, copy it into the `/app/config` folder
    2.  Be sure to keep this file secret -- don't check it into version control, etc.
2.  Run `php -S localhost:8000 -t . index.php` from the command line
3.  Visit `localhost:8000` to see the website
4.  Visit `localhost:8000/bolt` to access the admin panel
    -   Username == `devadmin`, password == `password`.
    -   Note that the username and password is only for the dev build. The actual server
        should obviously not use these credentials

Note: in development, the website is currently set up to automatically create and use a 
SQLite database. This may change in the future.

## Deploying to production

1.  Add the production server as a remote to your git repo:

        git remote add prod ssh://root@192.241.205.218/var/repo/website.git

2.  Then, once you've made a change to the code, push to both origin and then to prod:

        git push origin master
        git push prod master

    (Pushing to origin updates the Github repo; pushing to prod automatically updates 
    the website).

    You will need to provide a password when pushing to prod -- it's the same password
    you use when manually remoting into production (see below section).

3.  To update `config_local.yml`, either do it from Bolt's admin panel or ssh into 
    the server and do it yourself.


## Remoting into production

If you want to SSH into production, or transfer a file via SFTP, the host is `uwformula.com`, and the 
username is `root`. The password is either the team password, or an older variation of it. Use the 
default port.

(So, if you wanted to SSH in on linux, you'd run `ssh root@uwformula.com`)

Alternatively, you can directly use the IP address as the host: `192.241.205.218`.

If you're trying to modify something about the website, you should generally prefer using the git-push
method, as detailed above, so that we can keep everything version-controlled. The main exception to
this rule is if you want to add something you explicitly _don't_ want to be version controlled (such
as the `config_local.yml` file), want to work with something we don't track in the repo (database
info, uploaded photographs, etc), or want to do something unrelated to the main website.


## Production instructions

To setup a new server completely from scratch, run the following commands (assumption:
this is for a new DigitalOcean instance).

    # Install all dependencies
    sudo apt-get update
    sudo apt-get install git apache2
    sudo apt-get install php5 php5-cli libapache2-mod-php5
    sudo apt-get install php5-gd php5-curl php5-pgsql
    sudo apt-get install postgresql postgresql-contrib

    # Setup Git push-to-deploy -- follow the instructions here:
    # https://www.digitalocean.com/community/tutorials/how-to-set-up-automatic-deployment-with-git-with-a-vps
    # 
    # Use /var/www/team-website for the server live directory, and 
    # use /var/repo/website.git for the server repository
    #
    # After completing this step, /var/www/team-website should contain all the
    # files for the team website.

    # Setup permissions
    cd /var/www/team-website
    chmod -R 777 files/ app/cache/ appconfig/ theme/

    # Setup apache
    cp /etc/apache2/sites-available/000-default.conf
    /etc/apache2/sites-available/team-website.conf

    # Modify apache config files (will have to do some manual editing

    sudo vim /etc/apache2/sites-available/team-website.conf
    # Find and change the following lines
    # 
    #   ServerAdmin uwfsae@uw.edu
    #   ServerName uwformula.com
    #   ServerAlias www.uwformula.com
    #   DocumentRoot /var/www/team-website

    sudo vim /etc/apache2/sites-available/000-confi
    # Find and change the following lines:
    #
    #   DocumentRoot /var/www/team-website

    sudo vim /etc/apache2/apache2.conf
    # Find and change the following lines:
    #
    #   <Directory /var/www>
    #       Options Indexes FollowSymLinks MultiViews
    #       AllowOverride All
    #       Order allow,deny
    #       allow from all
    #       Require all granted
    #   </Directory>

    # Enable Apache
    a2enmod rewrite
    sudo a2ensite team-website.conf
    service apache2 restart

    # Create a Postgres DB for website
    sudo adduser team-website-db
    sudo i -u postgres

    # Do the following as the 'postgres' user:
 
        # select the name 'team-website-db', select 'no' for all options
        createuser --interactive
        createdb team-website-db
    
        psql
        alter user team-website-db with password '<password here>';
        \q

        exit

    # Now back to root
    vim /var/www/team-website/app/config/config_local.yml
    # Add the following:
    # 
    #   database:
    #       driver: postgres
    #       username: team-website-db
    #       password: <password here>
    #       databasename: team-website-db





## Todo

-   Figure out how to integrate website with other webpages
-   Figure out how to simplify admin panel UI/iron out some of bolt's 
    rough edges/write instructions
-   Verify that website looks ok on older versions of IE/corporate-style
    computers
-   Write instructions for setting up and using Composer
