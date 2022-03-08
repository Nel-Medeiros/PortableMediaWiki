# Portable MediaWiki

We will be installing MediaWiki on a flash drive and enabling infoboxes.

### Tutorials referenced
[MediaWiki install](https://www.mediawiki.org/wiki/Manual:Installing_MediaWiki_on_XAMPP)
[Infobox install](https://www.mediawiki.org/wiki/Manual:Importing_Wikipedia_infoboxes_tutorial)

### Overall Steps

 1. Install XAMPP
 2. Create a MySQL Database

## Installing XAMPP

 - The first step to install MediaWiki is to download and install XAMPP, which is an Apache distribution containing MariaDB, PHP, and Perl. [Click here](https://www.apachefriends.org/xampp-files/8.1.2/xampp-windows-x64-8.1.2-0-VS16-installer.exe) to download version 8.1.2.
 - Run the executable and install it in a folder called "xampp" in the root of the flash drive.
 - At the end of the installation, open XAMPP's control panel. Start both Apache and MySQL.

## Creating a database
-   Direct your browser to  [http://localhost/dashboard/](http://localhost/dashboard/)
-   Go to  **phpMyAdmin**  at the top of the page
-   Click  **Databases**  on the top bar.
-   Choose a name e.g. wikidb, select a  [collation](https://dev.mysql.com/doc/refman/5.7/en/charset-general.html)  e.g. if your wiki is using utf8 for its default charset then choose utf8_unicode_ci, and click  **Create**. (PT = utf8_general_ci)
-   Go into the new database and click on  **Privileges**  on the top bar.
-   Click  **Add user account**
-   Enter a name for the user (e.g., wikiuser), a password and for the hostname select  **Local/localhost**. You can leave all the rest blank. You may want to write down your database name, user name, and password, because you'll need those to install MediaWiki.
-   Make sure the option  **"Grant all privileges on database "xxx"** is selected
-   For  **Global privileges**  select check all
-   Click  **Go**.
## Editing PHP.ini

To make sure that the Apache HTTP server doesn't time out during the creation of the databases, modify the php.ini config file:

1.  Open the php.ini (can be found in the folder  _xampp/php_) file using your favorite text editor and find the following section and change to  **max_execution_time = 600**.
2. Look for Dynamic Extensions and make sure "extension=intl" is uncommented.
3.  Save the file and restart Apache
## Setting up MediaWiki

-   [Download MediaWiki](https://releases.wikimedia.org/mediawiki/1.37/mediawiki-1.37.1.tar.gz "download:mediawiki/1.37/mediawiki-1.37.1.tar.gz") version 1.37.1.
-   Extract all your MediaWiki files and folders into a subfolder e.g. mywiki\ of the  **htdocs**  folder, wherever that is e.g. e:\xampp\htdocs\.  
    MediaWiki is downloaded as  **.tar.gz**  file. To unpack such archive the following command can be used in Windows cmd or PowerShell:  `tar xzvf mediawiki-1.37.1.tar.gz`.
-   Direct your browser to the appropriate page, e.g. http://localhost/mywiki
-   Follow the MediaWiki installer's instructions, entering the database name, user name, and password you wrote down during the "creating your database" step above.
- Parser hooks selected:
  - InputBox
  - ParserFunctions
  - Scribunto
  - TemplateData
-   If you enabled the  ["security" option](http://www.apachefriends.org/en/xampp-windows.html#1221), then you need to  **Use superuser account**. This means the MySQL root account and its password.
-   After MediaWiki tells you that everything went smoothly, save your  [LocalSettings.php](https://www.mediawiki.org/wiki/LocalSettings.php "LocalSettings.php")  file to your wiki's root folder, e.g. e:\xampp\htdocs\mywiki.
-   Direct your browser once again to the appropriate page, e.g. http://localhost/mywiki. It should take you to the Main Page of your new wiki. Congratulations! You're done.
-   Add any extra  [extensions](https://www.mediawiki.org/wiki/Manual:Extensions "Manual:Extensions")  your wiki is going to require.
**Wiki already works**
## Installing TemplateStyles Extension
- [Download](https://www.mediawiki.org/wiki/Special:ExtensionDistributor/TemplateStyles) the file
- Put it in the */extensions* folder and unpack it using `tar xzvf TemplateStyles-REL1_37-b7e6c34.tar.gz`
- Add the following code at the bottom of your  LocalSettings.php: `wfLoadExtension( 'TemplateStyles' );`
## Installing WikiBase Extension
- Download [Composer v2.1.14](https://getcomposer.org/download/2.1.14/composer.phar) and put it in the *\php* folder (Note: to use Composer and PHP without changing PATH variables, you will need to write the complete path to each of them, eg.: `E:\xampp\php\php.exe E:\xampp\php\composer.phar`
- Download WikiBase from GIT, running these commands in the *\extensions* folder:

      git clone -b REL1_37 https://github.com/wikimedia/mediawiki-extensions-Wikibase.git Wikibase
      cd Wikibase
      git submodule update --init --recursive # get the dependencies using submodules

- Create and add the following code into `composer.local.json` at the root of your MediaWiki installation:

      {
        "extra": {
          "merge-plugin": {
            "include": [
              "extensions/Wikibase/composer.json"
            ]
          }
        }
      }
- Then run `composer install --no-dev` in the root of MediaWiki's installation.
- Add the following to `LocalSettings.php`:

      wfLoadExtension( 'WikibaseRepository', "$IP/extensions/Wikibase/extension-repo.json" );
      require_once "$IP/extensions/Wikibase/repo/ExampleSettings.php";
      wfLoadExtension( 'WikibaseClient', "$IP/extensions/Wikibase/extension-client.json" );
      require_once "$IP/extensions/Wikibase/client/ExampleSettings.php";
- Run some maintenance scripts:

      php maintenance/update.php
      php extensions/Wikibase/lib/maintenance/populateSitesTable.php

- And then a Wikibase script:

      php extensions/Wikibase/repo/maintenance/rebuildItemsPerSite.php
      php extensions/Wikibase/client/maintenance/populateInterwiki.php
