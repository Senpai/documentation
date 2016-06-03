---
title: Migrate Sites to Pantheon
description: General instructions for preparing and migrating remotely-hosted Drupal or WordPress sites to Pantheon.
categories: [developing]
tags: [migrate, getting-started]
keywords: migrate, migrating site, migrate from remote host, migrate existing site, migrate from other host, migrate from another host, how to migrate an existing site, alternate host, another host, migration, migrations, migrates, move site to pantheon, move from remote host, move from current host, move hosts, changing hosting providers, how to move hosting to pantheon
---

Your site migration has four phases. You’ll package your site, import it, test it out, and then change DNS and go live. With a good plan and understanding of the platform, the process will run smoothly.

We recommend migrating WordPress sites from another host using the [Pantheon Migration](https://wordpress.org/plugins/bv-pantheon-migration/) plugin, developed by [BlogVault](https://blogvault.net/). For more details, see [Migrate to Pantheon: WordPress](/docs/migrate-wordpress).

## Create Archives

In this phase, you will create an archive of your site. Archives can be stored in a single file or as three separate files.

Are you running another application in addition to WordPress or Drupal? You need to remove it. See [Platform Considerations](/docs/platform-considerations/#one-application-per-site).

You’ll need to package up your:

- **Codebase** - All executable code, including core, custom and contrib modules or plugins, themes, and libraries. For the suggested directory listing of your site’s codebase, see our [Drupal](/docs/drupal-export#manually-create-archive) or [WordPress](/docs/wordpress-export#manually-create-separate-site-archives) export documentation.

- **Database** - A single .sql dump, contains the content and active state of the site's configurations.

- **Files** - Anything in `sites/default/files` for Drupal or `wp-content/uploads` for WordPress. This houses a combination of uploaded content from site users, along with generated stylesheets, aggregated scripts, image styles, etc.


### Prepare Your Site For Export

Follow these best practices before exporting your site:

* **Put the source site into maintenance mode** by going to Configuration > Development > Maintenance in Drupal, or with extra code (we recommend the [WP Maintenance Mode](https://wordpress.org/plugins/wp-maintenance-mode/)) plugin in WordPress.  This will prevent the contents of your database from getting out of sync while you’re exporting.
* **Upgrade to the latest version of Drupal or WordPress core**. If your site runs an old version of core, our import process forces you to upgrade. Doing it before importing can avoid problems stemming from an out-of-sync database and codebase, and can expose incompatibilities that should be fixed.

<div class="alert alert-info" role="alert">
<h4>Note</h4>
Due to <a href="https://codex.wordpress.org/Upgrading_WordPress_-_Extended_Instructions#Upgrading_Across_Multiple_Versions">WordPress's incremental upgrade practice</a>, we highly recommend upgrading WordPress to the latest version in place or in a local environment before attempting to import the site to Pantheon. Importing an old site will upgrade WordPress code directly to the latest version.</div>
* **Clear all caches**. This removes unnecessary and out-of-date files from both the database and your filesystem, which will save time and valuable space.
* Take a look at your codebase and **remove any non-core code from your site** that you aren’t planning on running on Pantheon.
* If you’ve been using the database for things other than Drupal or WordPress, you should **drop or skip any unnecessary or unrelated database tables** that your site doesn’t need.

### Plan the Import
You can import during the site creation process using the importer tool or manually after the site has been created.

**The Importer Tool**

Using our importer during the site creation process has the following effects on the codebase:

 - New Git history
 - Replacement and upgrade to the latest core version from our [Drops-8](https://github.com/pantheon-systems/drops-8), [Drops-7](https://github.com/pantheon-systems/drops-7), [Drops-6](https://github.com/pantheon-systems/drops-6), or [wordpress](https://github.com/pantheon-systems/wordpress) repository
 - Assignment of the appropriate site framework (listed above) as the code upstream, used for core updates

<div class="alert alert-info" role="alert">
<h4>Reminder</h4>Importing automatically upgrades to the latest version of core. It's a best practice to keep core up-to-date to benefit from security and bug fixes, but if you use a site or distribution that relies on an outdated version of core, you may experience incompatibilities. If you experience issues, see the troubleshooting documentation for your <a href="https://codex.wordpress.org/Updating_WordPress#Troubleshooting">WordPress</a> or <a href="https://www.drupal.org/troubleshooting"> Drupal</a> upstream.</div>

The importer accepts either single file site archives or separate archives of the code, database, and files. It accepts file uploads up to 100MB, and can download publicly-accessible archives up to 500MB. Acceptable file types include `.tar`, `.zip`, `.gzip`, and `.sql`.

File size limits are per archive. Providing three files instead of one effectively increases the entire site import size limit to 1.5GB (500MB code, 500MB database, 500MB files).

**Manual Site Import**

Manually import the site outside of our importer tool if any of the following apply:

- Your site exceeds file size limit for uploads.
- Your site requires an upstream to an organizational or public distribution.
- You would like to preserve the site's existing Git history.
- Your site is running Drupal 8.

Import code, database, and files after creating the site using a combination of command line tools (Git, mysql-cli, and rsync) or with Git and the Site Dashboard's Workflow tool. See [Migrate to Pantheon: Manual Site Import](/docs/manual-import) for detailed instructions.

### Create Single File Archives
 Sites that can be packaged with a total archived size less than 500MB are able to use single file archives during the import process. You can create these archives with [Drush](/docs/drupal-export#create-archive-using-drush) or [Backup and Migrate](/docs/drupal-export#create-archive-using-backup-and-migrate) for Drupal sites, and [Plugins](/docs/wordpress-export#export-wordpress-via-plugins) for WordPress.

### Create Separate Archives of Code, Database, and Files

If your site cannot be packaged as a single archive less than 500MB, you'll need to create separate archives of each part of your site. For step-by-step instructions, see [Exporting an Existing WordPress Site](/docs/wordpress-export#manually-create-separate-site-archives) or [Exporting an Existing Drupal Site](/docs/drupal-export#manually-create-archive).

## Import Your Site

### Import the Site Archive from the Command Line
Single file site archives less than 500MB, downloadable from a publicly accessible URL, can import from the command line with [Terminus](/docs/terminus/), the Pantheon command-line interface.

```
terminus sites import [--site=<name>] [--label=<label>] [--org=<org>] [--url=<url>]
```

### Create and Name the Site

Click the [**Create New Site**](https://dashboard.pantheon.io/sites/create) button at either the user or Organization Dashboard. Enter a name and click **Create Site**.

### Choose a Start State
To use our site importer, select **Archive Importer**.
To import after creation, select the intended upstream and install it, and skip ahead to [import after creation](#import-after-creation).

### Import Archives

If your site is in a single file archive, upload the file or provide the publicly-accessible URL for the importer to download, and click **Import Site**. <div class="alert alert-info" role="alert">

<h4>Note</h4>
Modify Dropbox URLs so they end in <code>dl=1</code> instead of the default <code>dl=0</code>. This forces a download of your archive and avoids the Dropbox landing page.  </div>
 ![Single Archive Import](/source/docs/assets/images/single-archive-import.png)

If you prepared separate code, database, and files archives:

 1. Click **Provide separate code, database, and files archives**.
 2. Import each archive via the corresponding file upload or URL field.
 3. Click **Import Site** and wait for your site to complete.

When it completes, proceed to [test your site](#test-your-site).

### Import After Creation

At the new site's Dashboard, clone the code repository with Git. Once cloned, synchronize the code locally and merge in favor of the Pantheon master branch for any conflicts. Then, push the code back up to your Pantheon site repository. For instructions on how to clone using Git, see [Starting with Git](/docs/git/).

If the database and/or files meet the file size limits described above, you can import them into the Dev environment using the Workflow > Import tool.
 ![Import tool for database and files](/source/docs/assets/images/import-tool-db-and-files.png)

If the database is larger than 500MB, use the **Connection Info** panel to connect and import via a MySQL client or using the MySQL command line.

If the files archive is larger than 500MB, use an SFTP client or rsync to upload the uncompressed files.

See [Migrate to Pantheon: Manual Site Import](/docs/manual-import) for further details.

## Test Your Site
When the site's code, database, and files are all in place, verify everything is working as expected. At the Site Dashboard, click **Visit Development Site** for initial verification.

We recommend:

 - Using the Status tool in the Site Dashboard
 - Enabling [New Relic Pro](/docs/new-relic)
 - [Automated user acceptance testing](/docs/guides/wordpress-automated-testing) with Behat, Selenium, or Casper.js
 - Load testing using tools like [Blazemeter](/docs/guides/load-testing-with-blazemeter/)
 - Manual [user acceptance testing](https://en.wikipedia.org/wiki/Acceptance_testing#User_acceptance_testing)

### WordPress Troubleshooting
#### Sessions Error
```
Warning: session_start(): user session functions not defined
```  
This error means you have some code (plugin or theme) that's using PHP Sessions, which require a plugin to work on Pantheon. Read more on [WordPress and PHP Sessions](/docs/wordpress-sessions).

## Go Live
Follow the [Going Live](/docs/going-live) checklist for a successful launch.
