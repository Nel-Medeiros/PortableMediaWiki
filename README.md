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

1. Open the php.ini (can be found in the folder  _xampp/php_) file using your favorite text editor and find the following section and change to  **max_execution_time = 600**.
2. Look for Dynamic Extensions and make sure "extension=intl" is uncommented.
3. Change upload_max_filesize = 150M post_max_size = 150M
4. Save the file and restart Apache
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
- In the file located in */extensions/Scribunto/includes/engines/LuaStandalone/LuaStandaloneInterpreter.php* comment out line code that looks like `$cmd = '"' . $cmd . '"';`

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
      
- Copy the content of `MediaWiki:Common.css` and `MediaWiki:Common.js` into your wiki (create the page).
      
## Bonus 1: Importing Templates from Wikipedia
### Step 1: Determine info boxes needed
-   Find all infoboxes used on English Wikipedia at  [w:Category:Infobox templates](https://en.wikipedia.org/wiki/Category:Infobox_templates "w:Category:Infobox templates")
-   If there's an info box being used in an article you'd like to import:
    -   Visit the page's edit screen
    -   Scroll to the bottom and find the appropriate page name for the infobox template you'd like to import

### Step 2: Export Wikipedia templates

-   Go to English Wikipedia's  [Special:Export](https://en.wikipedia.org/wiki/Special:Export "w:Special:Export")  page and enter the full page names of the infobox templates from step 1.
-   For your first infobox to work, you'll also need to include the master infobox template,  `Template:Infobox`
-   Be sure to check the  **Include templates**  checkbox to include other necessary templates
-   Note that required templates may also include Scribunto modules in the  _module_  namespace
-   Most templates put their documentation in a subpage, so you may want to include  `/doc`  subpage for each template you're importing
    -   _Example:_  for  `Template:Infobox`  you should also download  `Template:Infobox/doc`

### Step 3: Export Wikipedia Modules

Many of templates used in Wikipedia, especially the collapsible navigation templates, utilize Scribunto. Without these modules installed, the templates will produce lua error messages. Please expand the list below of modules required for inboxes that are needed.

The following is a list of at least some modules that you should export from English Wikipedia to an XML file. Just cut and paste this list into the list of files to be exported (you might want to try doing one by one, in case it times out doing a batch, or use `php importDump.php --conf ../LocalSettings.php /path_to/dumpfile.xml.gz --username-prefix=""` in the maintenance folder):

-   [w:Module:Citation](https://en.wikipedia.org/wiki/Module:Citation "w:Module:Citation")
-   [w:Module:Hatnote](https://en.wikipedia.org/wiki/Module:Hatnote "w:Module:Hatnote")
-   [w:Module:Unsubst](https://en.wikipedia.org/wiki/Module:Unsubst "w:Module:Unsubst")
-   [w:Module:Sidebar](https://en.wikipedia.org/wiki/Module:Sidebar "w:Module:Sidebar")
-   [w:Module:Message_box](https://en.wikipedia.org/wiki/Module:Message_box "w:Module:Message box")
-   [w:Module:Navbox](https://en.wikipedia.org/wiki/Module:Navbox "w:Module:Navbox")
-   [w:Module:Arguments](https://en.wikipedia.org/wiki/Module:Arguments "w:Module:Arguments")
-   [w:Module:Yesno](https://en.wikipedia.org/wiki/Module:Yesno "w:Module:Yesno")
-   [w:Module:Authority_control](https://en.wikipedia.org/wiki/Module:Authority_control "w:Module:Authority control")
-   [w:Module:Infobox](https://en.wikipedia.org/wiki/Module:Infobox "w:Module:Infobox")
-   [w:Module:Navbar](https://en.wikipedia.org/wiki/Module:Navbar "w:Module:Navbar")

-   [w:Module:InfoboxImage](https://en.wikipedia.org/wiki/Module:InfoboxImage "w:Module:InfoboxImage")

-   [w:Module:Check_for_unknown_parameters](https://en.wikipedia.org/wiki/Module:Check_for_unknown_parameters "en:Module:Check for unknown parameters")

-   [w:Module:If_empty](https://en.wikipedia.org/wiki/Module:If_empty "en:Module:If empty")

### Step 4: Import infobox templates and Modules

-   Go to your wiki's  [Special:Import](https://www.mediawiki.org/wiki/Special:Import "Special:Import")  page and select the file you created from the export in step 2.

-   The  _modules_  you exported in step 3 will NOT import correctly unless you first have  **successfully installed Scribunto**, which will create the namespace for "modules." After Scribunto is installed, the modules should import properly. Afterward, if you see templates that give a lua error message, try to identify the missing module that must be imported.
-   Since many templates incorporate images, it is helpful to modify LocalSettings.php to allow instant commons import of images: `$wgUseInstantCommons = true;`
- 
### Step 5: De-Wikipediaify

-   You may want to review all of the files that you just imported for references to Wikipedia pages.
-   You can certainly leave them in place - and are encouraged to update them as links to the appropriate Wikipedia article (possibly using  [interwiki links](https://www.mediawiki.org/wiki/Manual:Interwiki_table "Manual:Interwiki table"))
-   Pay special attention to categories, which may be buried in the template's code, and may include reference to Wikipedia
    -   Again, you can leave them, but you may wish to update them to your wiki's name or categorization policy

### Step 6: Additional templates needed

-   Once your import is complete, you should check the template on your wiki to see if it functions correctly
-   A helpful step is to once again refer to the edit page's templates list
    -   If there are  [redlinks](https://www.mediawiki.org/wiki/Manual:Glossary#Redlink "Manual:Glossary"), go back to  [step 2](https://www.mediawiki.org/wiki/Manual:Importing_Wikipedia_infoboxes_tutorial#Step_2:_Export_Wikipedia_templates)  and repeat this process to import the missing templates

## Bonus 2 Backups
### Backing-up

_Main article:  [Manual:Backing up a wiki](https://www.mediawiki.org/wiki/Special:MyLanguage/Manual:Backing_up_a_wiki "Special:MyLanguage/Manual:Backing up a wiki")_

-   In SQL admin, go to the wiki database (typically wikidb), and click Export. Check the first box under "structure" (DROP TABLES), and check the "save as file" checkbox near the bottom. Click Go and save the file to the backup location.
-   Save a copy of the wiki folder, e.g. mywiki from c:\xampp\htdocs\mywiki to the backup location.

### Restoring

_Main article:  [Manual:Restoring a wiki from backup](https://www.mediawiki.org/wiki/Special:MyLanguage/Manual:Restoring_a_wiki_from_backup "Special:MyLanguage/Manual:Restoring a wiki from backup")_

-   Install XAMPP on the new server.
-   In SQL admin:
    -   Create a new blank database with the default options and a name of your choice.
    -   Import the database file you backed-up.
    -   Change the SQL password of the root for that db (in privileges tab)
-   Copy the wiki folder from back-up into the new htdocs folder.
-   Change LocalSettings.php to reflect the new db username and password.

#### Maximum execution time exceeded

If you get  _Fatal error: Maximum execution time of xx seconds exceeded ..._  edit the file config.inc.php in folder phpMyAdmin and set  `$cfg['ExecTimeLimit'] = 0;`
