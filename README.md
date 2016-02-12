# restore-percona-backup

Using Percona's xtrabackup, download and prepare a full or incremental
backup, then launch a MySQL instance based on the data from the backup.

We generate our backups by piping `xtrabackup --stream=xbstream | gzip`,
so restores are done using `gzip -dc | xbstream -x`.

## Using the Image

1) Get a restore token from Deploybot
2) Launch the docker image with the deploy token: `docker run -p 3306 "scpr:restore-percona-backup (token)"`
3) Once MySQL is launched, connect with: `mysql -h $(docker-machine ip default) -u root`

## Restore Steps

Given a JSON file containing an array of backup URLs, where array[0] is
base and array[1..-1] are incrementals in ascending order...

* Download base and incrementals
* Uncompress base
* Prepare base using `xtrabackup --prepare --apply-log-only --target-dir=/base/`

For each incremental:

* Uncompress incremental
* Apply incremental to base using `xtrabackup --prepare --apply-log-only --target-dir=/base/ --incremental-dir=/inc-x/`

Finally:

* Run `xtrabackup --prepare --target-dir=/base/`
* Copy backup files into /var/lib/mysql
* Set up access permissions