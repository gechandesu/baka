## NAME
baka \- do files and databases backup.

## SYNOPSIS
baka \[\-\-version | -V\] \[\-\-help | help\] \<command\> \[\<options\>...\]

## DESCRIPTION
baka allows you to backup MariaDB/MySQL and PostgreSQL directories and databases.

baka is based on handling entries. Processing can be started at once for everyone entries, either only for the selected ones, or for all but the excluded ones.

baka implements only the mechanism for creating archives with files and database dumps and transferring these files to remote storage using third-party tools (such as rsync, s3cmd). To perform scheduled backups, the command baka it is recommended to install it in the operating system task scheduler. For example cron. Examples of usage are in the section EXAMPLES.

From the above points and the possibility of customizing a separate entry, it follows that with using baka, you can configure the backup system quite flexibly.

**ATTENTION**: Despite the laudatory description above, baka should not be regarded as professional backup solutions. Please don't use baka in production-like environments.

### Basic concepts:

**Entry**

Entity describing the elements to be acted upon. There can be any number of these entities. For a more detailed description, see section CONFIGURATION FILES.

**Remote transport**

A method for delivering files to a remote storage. In version 0.1.0 it could be sync using rsync, upload files to S3 compatible storage with using the s3cmd utility or 'none', which means refusing to upload files to remote storage.

## COMMANDS

**backup**

Do backup right now. Available options:

    -i, --ignore=<entry>    run backup for all entries, except ignored.
    -e, --entry=<entry>     run backup for selected entry.
    --local                 force local backup.
    --no-verify             don't check archives integrity.
    --remove                remove old backups (forced).
    --help, help            print help message and exit.

**list**

List backup entries. Available options:

    -v, --verbose   print entries list verbosely (default).
    -s, --short     short format (names).
    -S              short format (pathes).
    --help, help    print help message and exit.

**test**

Test configuration incude all entries. Available options:

    -v, --verbose   print output verbosely.
    --help, help    print help message and exit.

**show**

Print main configuration.

**edit**

Edit main configuration in default text editor.

**remove**

Remove old local backups (older than $livetime days). Backup livetime is set in main.conf.

    -f, --force     force remove.
    --help, help    show this message and exit.

## CONFIGURATION FILES
### MAIN.CONF

Default path: /etc/baka/main.conf. This contains the values for the global baka settings.

Required parameters:

     log            log file (/var/log/baka).
     log_df         time format in the log. Example: %d %b %Y %T %z
                    More details in 'man date'.
     log_format     format of the log line. Has special syntax.
                    Example:
                        [%time] %log
                    Here:
                        %time   time formatted by 'log_df'.
                        %log    data string
     df             time format in file names. See 'man date'
     nf             file name format. Has special syntax.
                    Example:
                        %type_%name_%time
                    Here:
                        %type   content type. The archive name will be
                                added word 'dump' for database
                                or 'backup' for files.
                        %name   is the name of a directory or database.
                        %time   time formatted with 'df'.
     remote         remote transport.
     livetime       the lifetime of old backups (number of days).

Optional parameters:

    allow_commands  flag for executing commands from entries.
                    Default: Disabled
    ssh_uri         URI for SSH connection for rsync.
    ssh_port        specify port if different from standard 22.
    s3_uri          URI to connect to the S3 repository.
    local           local folder to save all backups. Everything
                    'dest' in entries will be ignored.

### ENTRIES

By default entries is stored into /etc/baka/entries/ directory in separated files. File name does not matter.

Here you can set the parameters for backuping the requred data set. See examples below.

**DESTINATION DIR**

This is this required parameter. It will be used if the `local` parameter is not specified in main.conf. You can specify it only once in an entry.

This is the only required parameter. And the only one that can be specified only once. In addition to the destination folder, at least one variable with data must be specified.

Example:

    dest = /home/user/backups

**FILES**

Files or directories to backup. Syntax:

    files = /path

Also you can do this:

    files = /path/1 path/2

You can add the `files` variable as many times as you like.

**EXCLUSIONS**

This parameter is in addition to the `files` parameter. With its help you can specify which files or folders should be excluded from the backup.

List files and folders separated by commas without spaces on one line:

    exclude = env,logs,__pycache__

Or split them into multiple lines:

    exclude = env
    exclude = logs
    exclude = __pycache__

**DATABASES**

baka can dump MariaDB/MySQL (shortcut 'mysql') and PostgreSQL (shortcut 'postgres') databases.

Syntax:

    [dmbs] = host:port:database:user:password

For example:

    postgres = localhost:5432:mydb:user:password

You can use shortened syntax too (use standart host and port):

    [dmbs] = host:database:user:password
    [dmbs] = database:user:password

**NOTE:** That due to the nature of the parser, colons and hash characters cannot be used in the database password.

**COMMANDS**

In addition to files and databases, baka can execute an arbitrary Bash command for you. For example:

    command = echo "Hello, World!"

This feature is disabled by default and must be added in the main config. DON'T USE UNTRUSTED COMMANDS IN ENTRY! This feature is pottentially dangerous. Make sure that access to the entries is restricted.

## EXAMPLES

An example of using baka. For example, there are two files in /etc/baka/entries/: example.org and example.com. Files content:

example.org:

    dest = /home/user/backups/example.org
    files = /srv/example.org/public
    exclude = logs, cache

example.com:

    dest = /home/user/backups/example.com
    files = /srv/example.org/public
    files = /srv/example.org/storage
    mysql = mydb: dbuser: password

In order to start backing up these sites at different times, let's add two tasks to crontab:

    00 00 * * *     /usr/bin/baka backup --entry=example.com
    00 00 */2 * *   /usr/bin/baka backup --entry=example.org

Likewise, you can start backing up more entries by adding exceptions or specifying specific entries. For example like this:

    baka backup --ignore=example.org --ignore=example.com

## BUGS
Report bugs to <gechandev@gmail.com> or <https://gitea.gch.icu/gd/baka>

## AUTHOR
gd (gechandev@gmail.com)
