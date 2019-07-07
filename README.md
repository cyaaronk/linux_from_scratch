# linux_from_scratch

This repository includes resources and self-made packaging and configuration scripts for LFS.

References:
1. http://www.linuxfromscratch.org/

# Updates

## SlackBuild

SlackBuild docs in /sbdoc uses /usr/doc and /usr/man to store documentations, but LFS use /usr/share/doc and /usr/share/man.

Solution: move those files into /usr/share,

```bash
cp -ur /usr/doc /usr/share
cp -ur /usr/man /usr/share
rm -r /usr/doc /usr/man
```

and update the changes in package scripts so removepkg knows where to delete your moved files when you remove the packages.

```bash
find /usr/doc -type d -print0 -maxdepth 1 -mindepth 1 | \
  while IFS= read -r -d '' DIR; do \
    PACFILE=`find /var/lib/pkgtools/packages -path "/var/lib/pkgtools/packages/${DIR##*/}*"`
    echo "Update ${PACFILE}"
    find $DIR -print0 -maxdepth 1 -mindepth 1 | \
      while IFS= read -r -d '' FILE; do \
        echo "usr/share/doc${FILE#/usr/doc}" >> $PACFILE
      done
  done
  
```

## Gzip Manpages

Again in SlackBuild docs, the code to gzip manpages are mistyped. Change it as follows:

```bash
gzip -9 -r $PKG/usr/share/man
```

## Tripwire

As tripwire is installed using SlackBuild script, the local and site passwords have to be changed after installing the package.
Edit the config and pol files and sign them with the new keys.
Delete the pol file afterwards.

The steps are as follows:

```bash
DIR=/etc/tripwire
SITE_KEY=$DIR/site.key
LOCAL_KEY=$DIR/`hostname`-local.key

twadmin --generate-keys --site-keyfile $SITE_KEY
twadmin --generate-keys --local-keyfile $LOCAL_KEY
twadmin --create-cfgfile --cfgfile $DIR/tw.cfg \
        --site-keyfile $SITE_KEY $DIR/twcfg.txt
twadmin --create-polfile --cfgfile $DIR/tw.cfg \
        --site-keyfile $SITE_KEY $DIR/twpol.txt
shred   -v -u /etc/tripwire/twpol.txt
```

reference: http://www.linuxfromscratch.org/blfs/downloads/8.2/BLFS-BOOK-8.2-nochunks.html#tripwire

## Sudo

SlackBuild scritps should be run by a lfs user, and the packages should change ownership to root:root using:

```bash
sudo chown -R -h root:root $PKG
```

Then sudo back its ownership to lfs:lfs after makepkg is called.
