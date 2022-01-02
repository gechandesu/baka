# baka 0.2.0

WARNING! All changes in this version breaks backward capatibility!

# CLI

New options:

- `--config`. Specify your baka.conf file.
- `backup --dry-run`. Test configuration without backup.
- `remove --list`. Print files to delete.

Changed:

- `list` command now has two options `--verbose` and `--help`. Verbose output show Entry name, path and local and remote storages in table view.

Removed:

- `show` command. View configuration file directly instead.
- `edit` command. Edit configuration file directly instead.
- `test` command. Use `backup --dry-run` instead.

# baka.conf (former name: main.conf)

Added:

- Default parameter values now is built in baka. Override it in baka.conf
- `entries` variable. You can set entries directory in baka.conf
- `filename_format`. Special formatting for filenames.
- `autoprefix`. Add automatic prefix to filenames (based on entry file name).
- `dir_per_date`. Backups path format. See baka(1).

Changed:

- `dest` now is `local` and can be used in baka.conf and entries.
- `remote` now is URI (like `[scheme]://[user[:password]]@[host[:port]][/path]`) and can be used in baka.conf and entries.

Removed:

- `nf`, use `filename_format` instead
- `df`, use `filename_format`
- `log_df`, use `log_format` istead

# Entries

Added:

- `local`. This is renamed `dest`.
- `remote`. Now you can specify remote storage per entry.
- `prefix`. Override filename prefix.
- `archive`. Renamed `files`. Archive files into **.tar.gz**.
- `copy`. Simple file copy  without compression.
- `database`. Replacement for `mysql` and `postrges` variables. Provide DB URI.

Removed:

- `dest`. Use `local` instead.
- `mysql` and `postgres` variables. Use `database` instead.
- `files` variable. Use `archve` instead.

# baka 0.1.2

Initial release.
