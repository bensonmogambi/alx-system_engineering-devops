# Postmortem

Upon the release of ALX's System Engineering & DevOps project 0x19, at approximately 06:00 East African Time (EAT) here in Nairobi, Kenya, an outage occurred on an isolated Ubuntu 14.04 container running an Apache web server. GET requests on the server led to 500 Internal Server errors when the expected response was an HTML file defining a simple Holberton WordPress site.

## Debugging Process

Bug debugger Maengu encountered the issue at 19:20 PST. He promptly proceeded to solve the problem.

Checked running processes using ps aux at 19:25 PST. Two apache2 processes - root and www-data - were properly running.

Looked in the sites-available folder of the /etc/apache2/ directory at 19:30 PST. Determined that the web server was serving content located in /var/www/html/.

In one terminal, ran strace on the PID of the root Apache process at 19:35 PST. In another, curled the server. strace gave no useful information.

Repeated step 3, except on the PID of the www-data process at 19:40 PST. strace revealed an -1 ENOENT (No such file or directory) error occurring upon an attempt to access the file /var/www/html/wp-includes/class-wp-locale.phpp.

Looked through files in the /var/www/html/ directory one by one, using Vim pattern matching to try and locate the erroneous .phpp file extension at 19:45 PST. Located it in the wp-settings.php file (Line 137, require_once( ABSPATH. WPINC. '/class-wp-locale.php' );).

Removed the trailing p from the line at 19:50 PST.

Tested another curl on the server at 19:55 PST. 200 A-ok!

Wrote a Puppet manifest to automate the fixing of the error at 20:00 PST.

## Impact 

The outage lasted for approximately 6 hours and impacted an unknown number of users.

## Summation

In short, a typo in the file name class-wp-locale.phpp caused the critical error in wp-settings.php in the WordPress app. The correct file name in the wp-content directory of the application folder was class-wp-locale.php.

The patch involved a simple fix on the typo, removing the trailing p.


## Prevention

This outage was caused by an application error, not a web server error. To prevent future outages:

Test the application before deploying.
Enable status monitoring with a service such as UptimeRobot to alert instantly upon an outage of the website.

Note that in response to this error, a Puppet manifest [0-strace_is_your_friend.pp](https://github.com/bensonmogambi/alx-system_engineering-devops/blob/main/0x17-web_stack_debugging_3/0-strace_is_your_friend)
