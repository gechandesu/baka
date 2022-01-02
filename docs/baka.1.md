## NAME
baka \- do files and databases backup.

## SYNOPSIS
baka \[\-\-version] \[\-\-help | help\] \[\-c | \-\-config=\<file\>\]  
        \<command\> \[\<options\>...\]

## DESCRIPTION
baka allows you to backup MariaDB/MySQL and PostgreSQL directories and databases. baka is designed to perform full backups.

baka is based on handling entries. Processing can be started at once for everyone entries, either only for the selected ones, or for all but the excluded ones.

baka implements only the mechanism for creating archives with files and database dumps and transferring these files to remote storage using third-party tools (such as rsync, s3cmd). To perform scheduled backups, the command baka it is recommended to install it in the operating system task scheduler. For example cron. Examples of usage are in the section EXAMPLES.

From the above points and the possibility of customizing a separate entry, it follows that with using baka, you can configure the backup system quite flexibly.

**ATTENTION**: Despite the laudatory description above, baka should not be regarded as professional backup solutions. Please don't use baka in production-like environments.

### Basic concepts:

**Entry**

Entity describing the elements to be acted upon. There can be any number of these entities. For a more detailed description, see section CONFIGURATION FILES.

## OPTIONS

    -c, --config=<file>     source baka.conf file.
    --version               print version and exit.
    --help, help            print help message and exit.

## COMMANDS

**backup**

Do backup right now. Available options:

    -i, --ignore=<entry>    run backup for all entries, except ignored.
    -e, --entry=<entry>     run backup for selected entry.
    --local                 force local backup.
    --no-verify             don't check archives integrity.
    --dry-run               configuration test.
    --remove                remove old backups (forced).
    --help, help            print help message and exit.

**list**

List backup entries. Available options:

    -v, --verbose   print entries list verbosely (table view).
    --help, help    print help message and exit.

**remove**

Remove old local backups (older than $livetime days). Backup livetime is set in baka.conf.

    -l, --list      print a list of files to be deleted.
    -f, --force     force remove.
    --help, help    show this message and exit.

## CONFIGURATION FILES
Configuration files contain variables that set operating parameters for the entire application or for processing an individual job.

Some of the config variables are arrays. below each variable is labeled with its type. Array parameters allow you to specify multiple instances of data, which will be combined into an array during parsing. For instance:

    copy = /path/to/file
    copy = /path/to/another_file

The end result is an 'copy' array with the values '( /path/to/file /path/to/another_file )'.

Some parameters are required, some are not. Also parameters can only be found in certain contexts.
There are only two contexts: baka.conf and entry.

    baka.conf   Default path: /etc/baka/baka.conf. This contains the values
                for the global baka settings.

    entry       By default entries is stored into /etc/baka/entries/ directory
                in separated files. File name does not matter. Here you can set
                the parameters for backuping the requred data set.

### PARAMETERS SUMMARY
    lo              Required: yes
                    Type: string
                    Context: baka.conf
                    Default: /var/log/baka

                    Path to log file.

    log_format      Required: yes
                    Type: string
                    Context: baka.conf
                    Default: [%d %b %Y %T %z] {log}

                    Log format. It actually is 'date' format string
                    (see date(1)) with special syntax (palceholdes):
                        {log}   will be replaced with actual log string.

    filename_format Required: yes
                    Type: string
                    Context: baka.conf
                    Default: {prefix}{name}_%Y%m%d-%H%M

                    Filename format. Similar to log_format.
                    Special syntax (placeholders):
                        {name}      replaces with file, directory or database
                                    name.
                        {prefix}    replaces with prefix specified in entry
                                    file or by autoprefix (based on entry
                                    file name).

    autoprefix      Required: no
                    Type: boolean
                    Context: baka.conf
                    Default: true

                    Automatic prefix for filenames. Prefix is based on
                    entry file name.

    dir_per_date    Required: yes
                    Type: string
                    Context: baka.conf
                    Default: %Y-%m-%d

                    Directory hierarchy definer. Actually is 'date' format
                    string but interpretated as path. With this variable you
                    can store your backups in hierarchy like this
                    ('%Y/%m/%d' -- year, month, day or any other):
                        2022/
                        |___01/
                        |   |____01/
                        |   |____02/
                        |   |...
                        |___02/
                            |___01/


    livetime        Required: yes
                    Type: integer
                    Context: baka.conf
                    Default: 30

                    The livetime of old backups (number of days).

    allow_commands  Required: no
                    Type: boolean
                    Context: baka.conf
                    Default: false

                    Flag for executing commands in Entries.

    local           Required: must be in at least one context
                    Type: string
                    Context: baka.conf, entry
                    Default: no default

                    Local storage. Contains path to local directory.

    remote          Required: must be in at least one context
                    Type: string
                    Context: baka.conf, entry
                    Default: 'none'

                    Remote storage. Here should be the address of the remote
                    storage. Standard URI is supported:
                        [schema]://[user[:password]]@[host[:port]]/[path]
                    Password can contain URL-encoded characters. See
                    https://en.wikipedia.org/wiki/Percent-encoding
                    Supported schemas:
                        rsync       upload files to remove server via rsync
                                    over SSH.
                        s3          upload files to Amazon S3 or S3 compatible
                                    storage via s3cmd tool.
                    Examples:
                        rsync://backupuser@123.45.67.89:2022/backups
                        s3://mybucketname

                    In order to use rsync, you need a SSH-key. And for s3cmd
                    you need to configure ~/.s3cmd file. See docs:
                    https://s3tools.org/s3cmd-howto
                    Send me feature request if you want to use another
                    schemas. See AUTHOR below.

    prefix          Required: no
                    Type: string
                    Context: entry
                    Default: no default

                    Filename prefix. Overrides 'autoprefix'.

    archive         Required: no
                    Type: array
                    Context: entry
                    Default: no default

                    Path to file or directory to archive (.tar.gz)
                    You can exclude some files with 'exclude' variable.

    exclude         Required: no
                    Type: array
                    Context: entry
                    Default: no default

                    Exclude some files or directories for tar (does not
                    work with 'copy'). You can list files separated by commas
                    or use multiple variable. For example, this is valid:
                        exclude = backups,logs,env,__pycache__
                        exclude = node_modules,.environment

    copy            Required: no
                    Type: array
                    Context: entry
                    Default: no default

                    Copy files or directories without compression using 'cp'.
                    Destination filename will be modified by filename_format.
                    For example 'myentry.conf':
                        local = /backups
                        copy = /srv/http/example.org/data.zip
                    In result:
                        /backups/2022-01-01/myentry.conf_data_20220101-0012.zip

    database        Required: no
                    Type: array
                    Context: entry
                    Default: no default

                    Database URI. Supported databases: MySQL/MariaDB, PostgreSQL.
                    URI format:
                        [schema]://[user[:password]@[host[:port]/[database]
                    Password can contain URL-encoded characters.
                    For example:
                        database = mysql://myuser:myp%40%24%24@localhost:/mydb

    cammand         Required: no
                    Type: array
                    Context: entry
                    Default: no deafult

                    Run Bash commands. Special characters does not matter.
                    You don't need escape anything. Example:
                        command = echo HACK!

## EXAMPLES

An example of using baka. For example, there are two files in /etc/baka/entries/: example.org and example.com. Files content:

example.org:

    local   = /backups
    archive = /srv/http/example.org/public
    exclude = logs,cache
    remote  = s3://bucket/

example.com:

    local   = /backups
    archive = /srv/http/example.com/public
    copy    = /srv/http/example.com/data.zip
    database = mysql://myuser:password@localhost:3306/mydb

In order to start backing up these sites at different times, let's add two tasks to crontab:

    00 00 * * *     /usr/bin/baka backup --entry=example.com
    00 00 */2 * *   /usr/bin/baka backup --entry=example.org

Likewise, you can start backing up more entries by adding exceptions or specifying specific entries. For example like this:

    baka backup --ignore=example.org --ignore=example.com

## BUGS
Report bugs to <gechandev@gmail.com> or <https://gitea.gch.icu/ge/baka>

## AUTHOR
ge (gechandev@gmail.com)
