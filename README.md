# baka

Simple and flexible full backup software written in Bash.

More info in manuals:

- [man [En]](https://nixhacks.net/baka/man)
- [man [Ru]](https://nixhacks.net/baka/man-ru)

## Installation


Add repository to /etc/apt/sources.list.d/:

```
sudo echo 'deb [arch=all] http://repos.gch.icu/debian testing main' > /etc/apt/sources.list.d/ge.list
```

Add key:

```
curl -s http://repos.gch.icu/DEB-GPG-KEY | sudo gpg --no-default-keyring --keyring gnupg-ring:/etc/apt/trusted.gpg.d/ge.gpg --import
```

Update package list and install baka:

```
sudo apt update
sudo apt install baka
```

Also you can download DEB package directly: [link](http://repos.gch.icu/debian/packages/testing/).

baka pulls the rsync and s3cmd packages as dependencies.

## Remote storage

To perform a backup to a remote storage, you need to forward SSH keys between servers in the case of rsync, or configure the .s3cmd config, in the case of copying to an S3-compatible storage ([documentation](https://s3tools.org/s3cmd-howto)). Of course, nothing prevents you from doing only local backups and transferring them in any convenient way.
