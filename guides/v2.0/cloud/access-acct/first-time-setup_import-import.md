---
layout: default
group: cloud
subgroup: 08_setup
title: Import Magento EE into Magento Enterprise Cloud Edition
menu_title: Import Magento EE into Magento Enterprise Cloud Edition
menu_order: 78
menu_node: 
level3_menu_node: level3child
level3_subgroup: import
version: 2.0
github_link: cloud/access-acct/first-time-setup_import-import.md
---
 

This topic discusses how to import code from your existing Magento EE project to your Magento Enterprise Cloud Edition's Git repository `master` branch.

<div class="bs-callout bs-callout-warning" id="warning" markdown="1">
The procedure discussed in this topic replaces your new Magento Enterprise Cloud Edition project with the contents of your existing Magento installation. Any data, websites, stores, and so on will be lost.

Before you continue, make sure there is nothing in your Magento Enterprise Cloud Edition project you want to keep.
</div>

## Create a remote Git reference {#cloud-import-ref}
This section discusses how to create a remote Git reference from your Cloud Git repository to the repository in which your Magento EE installation is located.

Before you continue, make sure you know the SSH or HTTPS URL for your Magento EE installation Git repository.

To create a remote Git reference:

1.  Log in to your local Magento Enterprise Cloud Edition development machine as, or switch to, the Magento file system owner.
2.  Make a copy of `composer.json` _in a non-tracked directory_ so it doesn't get overwritten.

        cp composer.json ../composer.json.cloud
3.  Rename your Cloud Git remote from `origin` to `cloud-project` to make it clear which repository is which:

        git remote rename origin cloud-project
4.  Add a remote upstream for your existing Magento EE installation:

        git remote add prev-project <git url>
5.  Confirm what you've done so far.

        git remote -v

    Results are displayed as follows.

    <pre class="no-copy">cloud-project   ikyyrqvlgnrai@git.us.magento.cloud:ikyykimjgnrao.git (fetch)
    cloud-project   ikyyrqvlgnrai@git.us.magento.cloud:ikyykimjgnrao.git (push)
    magento ikyyrqvlgnrai@git.us.magento.cloud:ikyykimjgnrao.git (fetch)
    magento ikyyrqvlgnrai@git.us.magento.cloud:ikyykimjgnrao.git (push)
    prev-project    git@github.com:mygitusername/myeereponame.git (fetch)
    prev-project    git@github.com:mygitusername/myeereponame.git (push)</pre>
6.  Make sure you're on the Cloud project `master` branch.

        magento-cloud environment:checkout master
7.  Make sure this `master` branch is set up to import code to the Cloud project.

        git branch -u cloud-project/master

## Import your Magento EE code to your Cloud project {#cloud-import-imp}
Before you continue, make sure you've completed all tasks discussed in the preceding section.

To import your Magento EE code to Cloud:

1.  Fetch the Magento EE branch.

        git fetch prev-project
3.  Reset your Cloud `master` branch to contain the code and the commit history of your Magento EE branch:

        git reset --hard prev-project/<branch name>
4.  Push code from your Magento EE project to your Magento Enterprise Cloud Edition project, overwriting the previous contents and commit history with that of your project:

        git push -f cloud-project master

As the project builds and deploys, many messages are displayed on the screen. A successful deployment is incidated by the following messages:

    Re-deploying environment 43biovskhelhy-master-l5ut8gq.
       Environment configuration:
         mymagento (type: php:7.0, size: S, disk: 2048)
         mysql (type: mysql:10.0, size: S, disk: 2048)
         redis (type: redis:3.0, size: S)
         solr (type: solr:4.10, size: S, disk: 1024)

    Environment routes:
       http://master-o9gv6gq-43biovskhelhy.us.magentosite.cloud/ is served by application `mymagento`
       https://master-o9gv6gq-43biovskhelhy.us.magentosite.cloud/ is served by application `mymagento`

To 43biovskhelhy@git.us.magento.cloud:43bmmwdkhelhy.git
   445b5e8..b597726  master -> master

## Import the Magento database
Before you can use your existing Magento EE clode in Magento Enterprise Cloud Edition, you must import the database.

To import the Magento database in Magento Enterprise Cloud Edition, you must know:

*   The Magento Enterprise Cloud Edition environment's SSH URL
*   The database name, user name, and password of the Cloud database

<div class="bs-callout bs-callout-info" id="info" markdown="1">
This topic discusses how to import the [integration system]({{ page.baseurl }}cloud/discover-arch.html#cloud-arch-int) database. The database connection information is different for [staging]({{ page.baseurl }}cloud/discover-arch.html#cloud-arch-stage) and [production]({{ page.baseurl }}cloud/discover-arch.html#cloud-arch-prod) systems. You'll need the assistance of Magento Support before you can migrate your integration system database to staging or production.
</div>

### Find the SSH URL
You can find the environment's SSH URL in any of the following ways:

*   From the [project Web Interface]({{ page.baseurl }}cloud/project/project-webint-basic.html#project-access)
*   Using the following command:

        magento-cloud environment:ssh --pipe

An SSH URL is similar to the following:

    43bkopvkhelhy-master-l8uv4kp@ssh.us.magentosite.cloud

### Database access
The name of the database can be found in the `$MAGENTO_CLOUD_RELATIONSHIPS` environment variable. Display the variable with the following command. The database name is stored under `databases->path`. The password is found under `databases->password`.

To find database access information:

1.  If you haven't already done so, log in to your local system as the Magento file system owner.
2.  Enter the following command:

        magento-cloud environment:ssh
3.  At the command prompt, enter the following command:

        echo $MAGENTO_CLOUD_RELATIONSHIPS | base64 -d | json_pp

The database connection information is displayed:

{% highlight yaml %}

"database" : [
      {
         "username" : "user",
         "query" : {
            "is_master" : true
         },
         "path" : "main",
         "port" : 3306,
         "host" : "database.internal",
         "password" : "",
         "scheme" : "mysql",
         "ip" : "192.0.2.150"
      }
   ]

{% endhighlight %}

In the preceding example, the database name is `main`, its listen port is `3306`, its host name is `database.internal`, its root user name is `user` and the user has no password.

### Transfer the database dump from Magento EE to Cloud
Use the `rsync` command as follows to transfer the database dump from your Magento EE system to the Magento Enterprise Cloud Edition environment.

Now that you have created the dump, move it to the var directory of the application you are importing into:

    rsync <db dump file name> <cloud ssh url>:var/db.sql.gz

### Drop and re-create the Cloud database
SSH into the cloud environment and empty the existing database, if it is populated. If you have done any work you would like to refer to later that's been done in the Cloud environment, then make a backup of that first. 

To drop and re-create the Cloud database:

1.  SSH to the Cloud environnment.

        magento-cloud environment:ssh
2.  Connect to the database.

        mysql -h <db-host> -P <db-port> -p -u <db-user> <db-name>

    For example, if your Cloud databases uses default values, enter:

        mysql -h database.internal -u user main
3.  Drop the database. At the `MariaDB [main]>` prompt, enter:

        drop database main;
4.  Re-create the database:

        create database <database name>;
5.  Exit the `MariaDB [main]>` prompt.

        exit
6.  At the shell command prompt, enter the followign command to re-create the database.

        zcat var/db.sql.gz | sed -e 's/DEFINER[ ]*=[ ]*[^*]*\*/\*/' | mysql -h <db-host> -P <db-port> -p -u <db-user> <db-name> 

    For example,

         zcat var/db.sql.gz | sed -e 's/DEFINER[ ]*=[ ]*[^*]*\*/\*/' | mysql -h database.internal -p -u user main

### Update base URLs
Before you can access Magento from your local Cloud development system, you must change the Base URLs in the Magento database. Base URLs are stored in the `core_config_data` table.

The following example shows how to change _only_ the insecure URL but you can use the same procedure to change secure URLs as well.

To update the unsecure base URL:

1.  Find the value of the integration system URL:

        magento-cloud url
2.  SSH to the Cloud integration server:

        magento-cloud ssh
3.  Connect to the database.

        mysql -h <db-host> -P <db-port> -p -u <db-user> <db-name>

    For example, if your Cloud databases uses default values, enter:

        mysql -h database.internal -u user main
4.  Show the contents of the `core_config_data` table.

        SHOW * from core_config_data;

    Note the `path` of `web/unsecure/base_url`; this is the value you'll change.
5.  Enter the following command to change the value of `path` to your integration server's unsecure base URL:

        UPDATE core_config_data SET value=<Cloud unsecure base URL> WHERE path=web/unsecure/base_url;
6.  Confirm the change by entering the following command:

        SHOW * from core_config_data;
7.  If the change was successful, enter `exit` to exit the `[Maria DB]` prompt.
8.  Continue with the next section.

<div class="bs-callout bs-callout-info" id="info" markdown="1">
For your system to be fully functional, you must also set unsecure and secure URLs for the default scope as well as for all websites, stores, and store views.
</div>

## Copy the encryption key

1.  SSH to the Cloud environnment.

        magento-cloud environment:ssh
2.  Open `app/etc/env.php` in a text editor.
3.  Replace the existing value of `key` with your Magento EE key.
4.  Save your changes to `env.php` and exit the text editor.

If `env.php` does not exist, create it with the following contents:

{% highlight php startinline=true %}
return array (
  'crypt' =>
  array (
    'key' => '<your encryption key>',
  ),
);
{% endhighlight %}

## Import media
To import media files into your Cloud environment:

1.  SSH to the Cloud environnment.

        magento-cloud environment:ssh
2.  Enter the following command to clear existing media files:

        rm -rf pub/media/*
4.  Enter the following command to extract the media files to the `pub/media` directory:

        tar -xzf var/media.tgz pub/media

## Clear the cache
On the Cloud environment, flush the cache:

    bin/magento cache:flush

After the cache flushes, enter `exit` to close the SSH tunnel.

## Push the changes
TBD

## Verify the import
To verify everything imported properly, perform the following tasks in your local Cloud development environment:

1.  On your Cloud environment, enter the following commands to find the information to log in to the Magento Admin and to view the storefront:

        magento-cloud environment:url
        magento-cloud variable:list