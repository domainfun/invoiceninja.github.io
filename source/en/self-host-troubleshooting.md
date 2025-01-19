---
extends: _layouts.self_host
section: content
locale: en
---

# Self Host Troubleshooting

### General Troubleshooting Steps

If you're experiencing issues with your self-hosted Invoice Ninja instance, follow these general troubleshooting steps before diving into the specific sections:

1. Verify that you meet the [minimum system requirements](https://invoiceninja.github.io/en/self-host-installation/#server-requirements).
2. Consult the [Invoice Ninja forum](https://forum.invoiceninja.com/) for community support.
3. Examine the logs for error messages. You can find the logs in the `storage/logs` directory.

## Incomplete / Unsuccessful in app updates

If you application failed to update to the latest installation there are several reasons why this may happen:

1. Timeout: Your system timed out during the upgrade process
2. System requirements may have changed. Prior to upgrading if the version is changing its minor release variable (v5.XXX.1) then you should check the system requirements / release notes to ensure your system is prepared prior to the upgrade. You may need to add a PHP extension or there may be a change to the minimum PHP version required. Patch versions (v5.10.XXX) do not require have any changes to system requirements.

In order to recover from a failed/incomplete update, you may attempt to run the self update again within the application else you will want to follow these steps:

1. Navigate to the releases page on [Github](https://github.com/invoiceninja/invoiceninja/releases)
2. Download the file labelled invoiceninja.tar
3. Unpack this file and overwrite your existing file structure
4. Then either of these two steps:
 a) From the command line at the project root: 

 ```bash
 composer i -o --no-dev
 php artisan migrate
 php artisan optimize:clear
 ```

 b) From a browser window 

 https://yourdomain.com/update?secret=UPDATE_SECRET_VALUE_FROM_ENV_FILE
 
### ERROR: Target class [view] does not exist

This error can usually be resolved by deleting the contents of the bootstrap/cache folder.

## Downgrading to a Specific Version

If you need to downgrade your Invoice Ninja installation to a specific version, follow the steps below. Ensure you have sufficient backups and preparation to avoid data loss or service interruption.

### Step 1: Backup Your Installation

#### 1.1 Backup Your `.env` File
The `.env` file contains critical configuration for your installation. Make a copy of it:

NB! change directory to your Invoice Ninja root folder
```bash
cd /var/www/invoiceninja
```

NB! Make sure to replace {youraccount} with your user folder
```bash
cp .env /home/{youraccount}/invoice-ninja-env-backup-YYYY-MM-DD
```

#### 1.2 Backup Your Database
Use your preferred database management tool to create a backup. Many tutorials are available online based on your specific database setup (e.g., MySQL, PostgreSQL).

#### 1.3 Backup Your Root Folder
For complete recovery, back up your Invoice Ninja root folder:

NB! Make sure to replace {youraccount} with your user folder
```bash
tar -pzcf /home/{youraccount}/invoiceninja-YYYY-MM-DD.tar.gz /var/www/invoiceninja
```

### Step 2: Download the Target Version
Navigate to the [Invoice Ninja Releases Page](https://github.com/invoiceninja/invoiceninja/releases) and identify the version you wish to downgrade to. Download the `invoiceninja.tar` file for that version into the root of your Invoice Ninja folder:

NB! change directory to your Invoice Ninja root folder
```bash
cd /var/www/invoiceninja 
```
```bash
wget https://github.com/invoiceninja/invoiceninja/releases/download/v5.XX.XX/invoiceninja.tar
```

### Step 3: Extract the Files
Extract the downloaded archive, ensuring it overwrites the existing files:

NB! Make sure to replace www-data with your web user
```bash
sudo -u www-data tar -xf invoiceninja.tar 
```

### Step 4: Clear Caches
Run the following commands to clear caches and optimize the application:

NB! Make sure to replace www-data with your web user
```bash
sudo -u www-data php artisan cache:clear
sudo -u www-data php artisan view:clear
sudo -u www-data php artisan config:clear
sudo -u www-data php artisan optimize:clear
sudo -u www-data composer dump-autoload
```

### Step 5: Clean Up
Delete the downloaded `invoiceninja.tar` file to keep your installation tidy:

```bash
cd /var/www/invoiceninja
```
```bash
rm -f invoiceninja.tar
```

### Step 6: Verify the Installation
Access your Invoice Ninja installation through the browser to ensure it is functioning correctly. If errors occur, follow these steps:

1. Delete the Invoice Ninja folder:
    
   ```bash
   rm -rf /var/www/invoiceninja
   ```
   
2. Restore from your backups:
   - Extract your root folder backup.

   ```bash
   tar -zxf /home/{youraccount}/invoiceninja-YYYY-MM-DD.tar.gz -C /
   ```

   - Restore your database backup.

### Notes
- Ensure that the target version’s system requirements match your server’s configuration.
- If you encounter permission issues, verify that the `www-data` user (or equivalent web server user) has the correct access to your installation directory.
- For additional help, consult the community forums.




## SQLSTATE[42S22]: Column not found: 

If you see in your error logs a message such as "Column not found" this indicates that your migrations are not up to date and need to be run there are two ways to force the migrations to run:

1. http://yourdomain.com/update?secret=insert_your_UPDATE_SECRET_variable_here
2. From the project root run the following command:

```bash

php artisan migrate

```

## Cron not running / Queue not running

<x-warning>
It can take up to an hour for the red warning triangle to disappear after correctly configuring your Cron.  

After making any changes to your cron setup you'll want to force a recheck of the cron setting. To do this navigate to http://url/update?secret=
</x-warning>

If you are faced with your recurring invoices not firing, or your reminders not sending, then most likely your cron job isn't working. The first thing is to make sure you have your cron jobs configured correctly by following the guide [here](https://invoiceninja.github.io/en/self-host-installation/#cron-configuration-1) 

If you are using shared hosting, then will need to add an additional parameter to the cron command which looks like this:

```
cd /path/to/root/folder && /usr/bin/php -d register_argc_argv=On artisan schedule:run >> /dev/null 2>&1
```

Please note that on some systems the php location may be different, so confirm with your hosting provider which path to PHP you need to use.

To test your changes, navigate your browser to the update URL which is in the following format:

```
https://yourdomain.com/update?secret=
```

The secret variable is located in your .env file until the key `UPDATE_SECRET` , this will force a recheck and if the cron is working the red error triangle will disappear.

## PDFs don't appear to be updating

If you are using Cloudflare, then most likely Cloudflare could be caching your static data. To force cache busting, edit your nginx.conf file and add in the following snippet

```
location ~* \.pdf$ {
    add_header Cache-Control no-store;
}
```

On Apache based servers, open the [/public/.htaccess](https://github.com/invoiceninja/invoiceninja/blob/master/public/.htaccess#L25) file and update the mod_headers block

```apacheconf
<IfModule mod_headers.c>
    # Blocks Search Engine Indexing
    Header set X-Robots-Tag "noindex, nofollow"

    # Prevents PDF File Caching
    <FilesMatch ".pdf$">
        Header set Cache-Control no-store
    </FilesMatch>
</IfModule>
```

## Email not sending

<x-warning>
If you are using Gmail - Use an [app specific password](https://support.google.com/accounts/answer/185833?hl=en) or ensure you have less secure apps turned on.
</x-warning>

<x-warning>
If you are using Office 365 - You may need to [enable SMTP AUTH](https://docs.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission).
</x-warning>

If you are using gmail smtp relay, then a additional .env variable is required.

For Gmail SMTP Relay also ensure you have configure this service in Google by using the following steps:

```
Go to [Apps > Google Workspace > Gmail > Routing]
Next to SMTP relay service, click Configure.
Set up the SMTP relay service by following the steps in [SMTP relay: Route outgoing non-Gmail messages through Google]
Then, in your env file, use the following:

Gmail SMTP Relay requires a proper EHLO hostname domain to be sent during the SMTP handshake: [127.0.0.1] doesn’t cut it anymore. For that, Laravel has to check for a host domain variable and send it along with the handshake request.

MAIL_EHLO_DOMAIN="server.domain.com"
MAIL_MAILER=smtp
MAIL_HOST=smtp-relay.gmail.com 
MAIL_PORT=587
MAIL_USERNAME=xxxx
MAIL_PASSWORD=xxxx
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=xxxx
MAIL_FROM_NAME=xxxx
```

You will also want to make sure you do not have any firewall rules which may be blocking access to the Google servers, just in case, ensure the following IP addresses are whitelisted:

```
142.251.163.28
172.253.63.28
172.253.118.28
74.125.24.28
142.250.114.28
142.250.4.28
142.251.12.28
172.217.194.28
142.250.31.28
142.251.167.28
172.253.115.28
```

For more detailed information on Gmail Relay, see this post of our forum by community member charles

https://forum.invoiceninja.com/t/emails-not-sending-yet-another-thread-v5-5-55/12401/9

### Mail Mailer configuration

The ```MAIL_MAILER``` field defines which email driver you wish to use, this could be postmark, mailgun, smtp, log - anything that Laravel 9 supports natively is supported in this app.

If the mail config is correct, the next place to check would be to check the error logs for any errors that are being thrown, the error log is found in ```storage/logs/laravel.log```

The final source of information in diagnosing mail troubles is to inspect the ```System Logs``` tab in the dashboard of the application, in here we log any messages from the mail server itself which may be instructive as to the cause of your issues.

If you are using the Queue system ie. QUEUE_CONNECTION=database then you may also want to check the ```jobs``` table in the database, there should be _no_ records in that table... If there are records in the table it means that your queue is not running and therefore no mail jobs are being processed.

It's possible the emails are sent but are blocked for DNS, SPF, DKIM or other reasons. In these cases emailing a test invoice to [mail-tester.com](https://mail-tester.com) can help debug certain problems.

Also, if you see in /storage/logs/invoiceninja.log this line ```error failed with stream_socket_enable_crypto(): SSL operation failed with code 1. OpenSSL Error messages:
error:14090086:SSL routines:ssl3_get_server_certificate:certificate verify failed``` then try running `yum update` on your webserver, it should fix the ca-certificates problem.


## GoDaddy email sending woes

GoDaddy does not allowing sending via third party [SMTP](https://my.godaddy.com/help/send-form-mail-using-an-smtp-relay-server-953) servers. They require sending all email via their own servers. If you need to use GoDaddy, we suggest using a transactional email service such as PostMark to bypass.

## PDF conversion issues.

We strongly recommend using the built in [snappdf](https://github.com/beganovich/snappdf) package which is a highly performant PDF generator based on the headless chrome/chromium binary. This package is perfect for users that have root access to their server and are able to install the required dependencies if needed.

To configure SnapPDF use the following .env vars

```
PDF_GENERATOR=snappdf
```

As of version 5.5.12 we no longer prebundle snappdf in the release files, so if you have a new installation you'll need to manually invoke the download of the chromium binary, from the root of the project run the following:

```
vendor/bin/snappdf download
```

<x-warning>
If PDF generation is broken after an upgrade, you may need to run the above command again, with the force option.

chmod +x vendor/bin/snappdf
sudo -u www-data vendor/bin/snappdf download --force

</x-warning>

Snappdf is also the default PDF engine in our [Docker](https://github.com/invoiceninja/dockerfiles) image, so if you prefer a very simple installation please consider our Docker setup as it is very fast to get going!

You can use this command to test Snappdf:

```
./vendor/bin/snappdf convert --html "<h1>Hello world</h1>" test.pdf
```

A complete list of required dependencies is available [here](https://github.com/beganovich/snappdf#headless-chrome-doesnt-launch-on-unix).


<x-warning>
  In a recent update, chromium/chrome uses the crashpad handler to write to the users home directory .local .config 

  When running snappdf as a user such as www-data this users directory - typically /var/www - may not be writable OR the www-data user may not be able to create the .local / .config directories required to support SnapPDF / Chrome.

  It is advised to chown the home dir of the www user to ensure that crashpad can write to this temp directory and allow PDFs to be generated.

  sudo chown -R www-data:www-data /var/www/.local
  sudo chown -R www-data:www-data /var/www/.config
  
</x-warning>

If you are on shared hosting, snappdf probably will be impossible for you to use as you do not have access to the subsystem to install the required packages. Instead, you will need to use a hosted PDF service, the two that Invoice Ninja v5 supports is [PhantomJS Cloud](https://phantomjscloud.com/) and our own hosted PDF generator which can use for _free_ to generate _unlimited_ PDFs.

### Phantom JS

Phantom JS Cloud is the default PDF engine [PhantomJS Cloud](https://phantomjscloud.com/) to generate your PDFs, the default API key that comes with a clean installation will no reliably generate PDFs, to ensure you can generate PDFs you should register for an API key on the phantomjscloud website and use this key in the .env file.

Phantom JS can be toggled on and off by setting the PHANTOMJS_PDF_GENERATOR to either TRUE or FALSE. The following .env variables are available for configuring PhantomJS.

```
PDF_GENERATOR=phantom
PHANTOMJS_KEY='a-demo-key-with-low-quota-per-ip-address'
PHANTOMJS_SECRET='your-secret-here'
```

The `PHANTOMJS_SECRET` can be any random value, it's used to bypass the client portal password.

If you experience errors with PDF generation, such as `500 Server error` or `Failed to load PDF document` or a continuous loading bar, you must get a PhantomJS key [here](https://dashboard.phantomjscloud.com/dash.html#/signup), Replace it with the prefilled key `a-demo-key-with-low-quota-per-ip-address`. 

<x-warning>
For PhantomJS to work, your Invoice Ninja installation web address must be public; localhost installations or those on private networks won't be able to use PhantomJS Cloud.
</x-warning>

### Hosted Invoice Ninja PDF generation

The default PDF generated as of version 5.5.12 is our hosted platform PDF conversion system. The hosted ninja PDF generator is an offsite PDF generator hosted by Invoice Ninja, which operate similar to PhantomJS. It is important to note that we do not store any information with this service, we simply convert the HTML your system sends into a PDF file which is return on the fly.

```
PDF_GENERATOR=hosted_ninja
```  

## Platform specific issues

### General advice

When facing errors, first set `APP_DEBUG=true` in `.env`

### Process has been signaled with signal "5"

This error message is observed when the queue attempts to perform a action where the queue user does not have the correct permissions. You may see this error if you run command line arguments as a user other than the web user.

This is most commonly see in Invoice Ninja where snappdf has been downloaded from the command line as a regular user, the permissions on the binary may prevent the webuser from executing the chrome binary when generating the PDF.

Always ensure that tasks run on the command line are executed by the web user, on Ubuntu this is typically www-data


<x-warning>
  In a recent update, chromium/chrome uses the crashpad handler to write to the users home directory .local .config 

  When running snappdf as a user such as www-data this users directory - typically /var/www - may not be writable OR the www-data user may not be able to create the .local / .config directories required to support SnapPDF / Chrome.

  It is advised to chown the home dir of the www user to ensure that crashpad can write to this temp directory and allow PDFs to be generated.
</x-warning>

### Erroneous data format for unserializing 'Symfony\Component\Routing\CompiledRoute'

<p>The most common cause of this issue is running multiple version of PHP, if the caches are built with a different version of PHP you may see the above error as differing versions of PHP may not be interoperable on the same installation. Ensure you are running the same CLI and Web PHP version to prevent any errors/</p>

### Unable to connect to database after installation

<p>You may need to restart the queue like this</p>

```
php artisan queue:restart
```

### Nginx: 413 – Request Entity Too Large

This error indicated that the client_max_body_size parameter in NGINX is too small, you will need to edit your nginx config and increase the size

```
client_max_body_size 100M;
```

### Proxy configuration.

For users that rely on configuring a reverse proxy, please consider this post on our forum which details steps which may assist in configuring a reverse proxy.

<a href="https://forum.invoiceninja.com/t/selfhosting-setup-failing/5651/8">Reverse Proxy Invoice Ninja</a>

### Problems with migration

If you are experiencing issues with the migration not running as expected please run through the following checklist:

 * Ensure directories are read/writable by the webuser (ie www-data)
 * Ensure the cron scheduler is running (and working) - You can verify it is working by inspecting the ```jobs``` table in the database, it should be empty
 * Inspect the log file /storage/logs/laravel.log for further information.
 * If you are still experiencing issues, turn on advanced logging by adding the following variable to your .env file. ```EXPANDED_LOGGING=true``` Then attempt the migration again and afterwards inspect the log file in storage/logs/invoiceninja.log

### libatk.so not loading for Google

Pdf generation will not working using the inbuilt PDF engine without some subsystem dependencies, please consult this resource for the list of necessary libraries for each supported platform <a href="https://github.com/beganovich/snappdf#headless-chrome-doesnt-launch-on-unix">Snappdf required libraries</a>

### WebCron configuration

Some systems do not allow cron configurations, one work around is to use a web cron service which can hit a defined endpoint which executes the scheduler via a GET HTTP request. Invoice Ninja has implemented a small service to allow a webcron service to hit the end point:

```
https://domain.com/webcron?secret=
```

To configure the service, you need to add a .env variable

```
WEBCRON_SECRET=password
```

### Installing in a subdirectory.

It is possible to install Invoice Ninja in a subdirectory outside the doc root, to enable this you will need to update the .htaccess file (only if you are using the Apache webserver),

```php
RewriteRule ^(.*)$ public/$1 [L]
```

should be updated to

```php
RewriteRule ^(.*)$ subdirectoryname/public/$1 [L]
```

### Endless setup loop

If you are finding that all your pre setup checks are passing however you keep falling back to the setup screen, this could indicate that you are missing the ```mysql-client``` library which is needed to perform the initial migration. If you are unable to install this for some reason (ie. XAMPP) then you'll need to run the migrations manually by entering the following at the command prompt

```
php artisan migrate:fresh --seed 
```

### flock() expects parameter 1 to be resource, bool given

This error is thrown from deep within PHP and indicates a permissions issue - most likely the public/storage and/or storage/ directory is not writable by the web user, depending on your platform, you'll need to run something like:

```
sudo chown -R www-data:www-data public/storage

sudo find ./ -type d -exec chmod 755 {} \;
```

and/or

```
sudo chown -R www-data:www-data storage/

sudo find ./ -type d -exec chmod 755 {} \;
```

### Unresponsive UI

If for some reason the UI becomes unresponsive, you may need to flush some subsystem configuration and rebuild. It is possible to do this by navigating to the `/update?secret=`  route, ie. https://invoiceninja.test/update?secret= This will perform a number of system clean ups and may resolve issues resulting from an incomplete upgrade. To protect this route, you are advised to add a .env pararameter `UPDATE_SECRET=a_secret_passcode` this will restrict the route to users with the UPDATE_SECRET passcode.

### Logo not appearing in the PDF

It may help to add `LOCAL_DOWNLOAD=true` to the .env file, this will embed the image in the PDF rather than request it over the network.

### Communication link failure: 1153 Got a packet bigger than 'max_allowed_packet'

If you are using the database for your queue's then sometimes you may see an error from MySQL

```
1153 Got a packet bigger than 'max_allowed_packet'
```

This indicates the insertion payload is bigger than MySQL is configured to handle! To work around this, you will need to increate the mysql variable

```
max_allowed_packet
```

To a larger value. Sometimes a value of 1024M is required.

It may also be wise to increase the variable

```
max_connections
```

as similar errors can be reported from the DB.

### 500 error when editing PDF templates

There was a [report](https://forum.invoiceninja.com/t/500-error-when-editing-pdf-invoice-templates-potential-fix/7067) 
from the user who solved 500 error on their server by disabling ModSecurity.

### 500 error when trying to login or edit company details

Try these steps to fix the 500 server error when trying to login or editing company details

1. Download the latest update from the [github releases](https://github.com/invoiceninja/invoiceninja/releases) (not `invoiceninja.tar` but `Source code (zip)`)
2. Upload the zip, extract the files and override them in your /public_html/ (Be careful to not override the .env file or all will be gone)
3. Login to your root and make sure first of all that all files are owned recursively by the user, ex. `sudo chown -R www-data:www-data dir/`
4. Run this command `cd /home/domain.com/public_html/invoiceninja/ && php artisan migrate` or simply `php artisan migrate` whatever works for you, select "YES"
5. If an error occurs like this one

```
PHP Fatal error:  Cannot declare class UpdateDesigns, because the name is already in use in /home/domain.com/public_html/invoiceninja/database/migrations/2021_09_16_115919_update_designs.php on line 0
In 2021_09_16_115919_update_designs.php line n/a: Cannot declare class UpdateDesigns, because the name is already in use
```

Delete that file and retry the command until it works and runs properly.

6. Go to https://domain.com/update?secret=x to be sure the update worked, it should load the login screen and work, you should also be able to edit the company details again.

### Unresolvable dependency resolving [Parameter #0 [ array $options ]] in class App\Utils\CssInlinerPlugin

When changes are made to the container this can causes the cache to become stale in the application preventing it from booting. 

The solution is to clear the contents of the folder ```bootstrap/cache```, by either manually deleting files or by running ```/update?secret=``` which will also delete the contents of this directory. 

### Uncaught ErrorException Collection::offsetExists($key)

This error is observed when the system has Composer v1 installed. Update to Composer 2 using the following command

```
sudo -H composer self-update
```

### file_exists(): open_basedir restriction in effect

If you aren't able to adjust the open_basedir restrictions the following steps may help:

1. Delete bootstrap/cache/config.php
2. Delete all log files in storage/logs

### I've forgotten my password and cannot reset it, HELP!

If you have command line access, you can reset your password manually by following the following steps. From the command line, navigate to the project directory and run

```
php artisan tinker
```

Then find the id of the user for the password reset

```
User::all();
```

Retrieve the user

```
$user = User::find(id_of_user_to_find);
```

Now lets reset the password

```
$user->password = Hash::make('password');
$u->save();
