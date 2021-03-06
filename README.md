Taskcluster Backup
==================
[![Build Status](https://travis-ci.org/taskcluster/taskcluster-backup.svg?branch=master)](https://travis-ci.org/taskcluster/taskcluster-backup)

A simple backup for taskcluster tables to protect from situations where a bug or a typo accidentally deletes a table. The backup strategy is quite
straightforward in an attempt to avoid bugs.

0. A list of all accounts and tables managed by taskcluster-auth is fetched from that service and any tables we wish to ignore is filtered out.
0. For each table in parallel up to a configurable concurrency, we fetch rows using fast-azure-storage. Each row is `JSON.stringified` and written to a stream.
0. The stream is passed into zstd for compressions and then on to `s3.upload`.
0. When everything is uploaded, we're done.

For instructions on using this within Mozilla, you can view the "Restore Azure Table from Backup" How-To article in the Taskluster space of [Mana](https://mana.mozilla.org).

Backup
------
This is run automatically from [a hook](https://tools.taskcluster.net/hooks/#taskcluster/azure-backup). From there you can configure which
accounts/tables to ignore or include with environment variables. The bucket for the backups can be found in the console [here](https://console.aws.amazon.com/s3/buckets/taskcluster-backups). If you want, you can run the backup manually as well. You'll need to copy `user-config-example.yml` to `user-config.yml` and then fill out all of the appropriate fields.

```
./taskcluster-backup backup
```

Restore
-------

This is a somewhat simpler operation than backup because it takes AWS and Azure creds directly instead of going through taskcluster-auth. This must be this way because in a situation where we're restoring from backup, it is likely that taskcluster-auth is inoperable. Again copy over `user-config-example.yml` to `user-config.yml` and fill out all of the appropriate fields.

You can get AWS creds from the AWS console and get Azure creds by generating SAS creds for the tables from in the Azure console.

```
./taskcluster-backup restore
```

Heroku Settings
---------------

We do not automatically backup the Heroku settings (application config variables).
This information must be stored separately from backups, as it contains (among other things) encryption keys without which the backed-up data is useless.

Heroku backups are done by hand:

```shell
# Install the Heroku CLI (by any means; this is just one) and login
npm install -g heroku-cli
heroku login

# dump the config variables; the list of apps is in the script
# ..and insert into the password store
./dump heroku-config.sh | pass insert -f -m heroku-config
```
