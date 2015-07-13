# Omnibus GitLab

This project creates full-stack platform-specific [downloadable packages for GitLab][downloads].
For other installation options please see the
[GitLab installation page][installation].

## Canonical source

The source of omnibus-gitlab is [hosted on
GitLab.com](https://gitlab.com/gitlab-org/omnibus-gitlab) and there are mirrors
to make contributing as easy as possible.

## Documentation version

Please make sure you are viewing the documentation for the version of
omnibus-gitlab you are using. In most cases this should be the highest numbered
stable branch (example shown below).

![documentation version](doc/images/omnibus-documentation-version.png)

## Omnibus fork

Omnibus GitLab is using a fork of [omnibus project](https://github.com/chef/omnibus). Fork is located at [gitlab.com](https://gitlab.com/gitlab-org/omnibus).

## GitLab CI

To setup GitLab CI please see the [separate GitLab CI
documentation](doc/gitlab-ci/README.md).

## Configuration options

GitLab and GitLab CI are configured by setting their relevant options in
`/etc/gitlab/gitlab.rb`. For a complete list of available options, visit the
[gitlab.rb.template][]. New installations starting from GitLab 7.6, will have
all the options of the template listed in `/etc/gitlab/gitlab.rb` by default.

## Installation

Please follow the steps on the [downloads page][downloads].

### After installation

Run `sudo gitlab-ctl status`; the output should look like this:

```
run: nginx: (pid 972) 7s; run: log: (pid 971) 7s
run: postgresql: (pid 962) 7s; run: log: (pid 959) 7s
run: redis: (pid 964) 7s; run: log: (pid 963) 7s
run: sidekiq: (pid 967) 7s; run: log: (pid 966) 7s
run: unicorn: (pid 961) 7s; run: log: (pid 960) 7s
```

If any of the processes is not behaving like expected, try tailing their logs
to see what is wrong.

```
sudo gitlab-ctl tail postgresql
```

Your GitLab instance should reachable over HTTP at the IP or hostname of your server.
You can login as an admin user with username `root` and password `5iveL!fe`.

### Common installation problems

See [doc/common_installation_problems.md](doc/common_installation_problems.md).

#### Apt error 'The requested URL returned error: 403'

See [doc/common_installation_problems.md](doc/common_installation_problems.md#apt-error-the-requested-url-returned-error-403).

#### GitLab is unreachable in my browser

See [doc/common_installation_problems.md](doc/common_installation_problems.md#gitlab-is-unreachable-in-my-browser).

#### GitLab CI shows GitLab login page

See [doc/common_installation_problems.md](doc/common_installation_problems.md#gitlab-ci-shows-gitlab-login-page).

#### Emails are not being delivered

See [doc/common_installation_problems.md](doc/common_installation_problems.md#emails-are-not-being-delivered).

#### Reconfigure freezes at `ruby_block[supervise_redis_sleep] action run`

See [doc/common_installation_problems.md](doc/common_installation_problems.md#reconfigure-freezes-at-ruby_blocksupervise_redis_sleep-action-run).

#### TCP ports for GitLab services are already taken

See [doc/common_installation_problems.md](doc/common_installation_problems.md#tcp-ports-for-gitlab-services-are-already-taken).

#### Git SSH access stops working on SELinux-enabled systems

See [doc/common_installation_problems.md](doc/common_installation_problems.md#git-ssh-access-stops-working-on-selinux-enabled-systems
).

#### Postgres error 'FATAL:  could not create shared memory segment: Cannot allocate memory'

See [doc/common_installation_problems.md](doc/common_installation_problems.md#postgres-error-fatal-could-not-create-shared-memory-segment-cannot-allocate-memory).

#### Reconfigure complains about the GLIBC version

See [doc/common_installation_problems.md](doc/common_installation_problems.md#reconfigure-complains-about-the-glibc-version).

#### Reconfigure fails to create the git user

See [doc/common_installation_problems.md](doc/common_installation_problems.md#reconfigure-fails-to-create-the-git-user).

#### Failed to modify kernel parameters with sysctl

See [doc/common_installation_problems.md](doc/common_installation_problems.md#failed-to-modify-kernel-parameters-with-sysctl).

#### I am unable to install omnibus-gitlab without root access

See [doc/common_installation_problems.md](doc/common_installation_problems.md#i-am-unable-to-install-omnibus-gitlab-without-root-access).

#### gitlab-rake assets:precompile fails with 'Permission denied'

See [doc/common_installation_problems.md](doc/common_installation_problems.md#gitlab-rake-assetsprecompile-fails-with-permission-denied).

#### 'Short read or OOM loading DB' error

See [doc/common_installation_problems.md](doc/common_installation_problems.mdr#short-read-or-oom-loading-db-error).

### Uninstalling omnibus-gitlab

To uninstall omnibus-gitlab, preserving your data (repositories, database, configuration), run the following commands.

```
# Stop gitlab and remove its supervision process
sudo gitlab-ctl uninstall

# Debian/Ubuntu
sudo dpkg -r gitlab

# Redhat/Centos
sudo rpm -e gitlab
```

To remove all omnibus-gitlab data use `sudo gitlab-ctl cleanse`.

To remove all users and groups created by omnibus-gitlab, before removing the gitlab package (with dpkg or yum) run `sudo gitlab-ctl remove_users`. *Note* All gitlab processes need to be stopped before runnign the command.

## Updating

Instructions for updating your Omnibus installation and upgrading from a manual installation are in the [update doc](doc/update.md).

## Starting and stopping

After omnibus-gitlab is installed and configured, your server will have a Runit
service directory (`runsvdir`) process running that gets started at boot via
`/etc/inittab` or the `/etc/init/gitlab-runsvdir.conf` Upstart resource.  You
should not have to deal with the `runsvdir` process directly; you can use the
`gitlab-ctl` front-end instead.

You can start, stop or restart GitLab and all of its components with the
following commands.

```shell
# Start all GitLab components
sudo gitlab-ctl start

# Stop all GitLab components
sudo gitlab-ctl stop

# Restart all GitLab components
sudo gitlab-ctl restart
```

Note that on a single-core server it may take up to a minute to restart Unicorn
and Sidekiq. Your GitLab instance will give a 502 error until Unicorn is up
again.

It is also possible to start, stop or restart individual components.

```shell
sudo gitlab-ctl restart sidekiq
```

Unicorn supports zero-downtime reloads. These can be triggered as follows:

```shell
sudo gitlab-ctl hup unicorn
```

Note that you cannot use a Unicorn reload to update the Ruby runtime.

## Configuration

### Backup and restore omnibus-gitlab configuration

All configuration for omnibus-gitlab is stored in `/etc/gitlab`. To backup your
configuration, just backup this directory.

```shell
# Example backup command for /etc/gitlab:
# Create a time-stamped .tar file in the current directory.
# The .tar file will be readable only to root.
sudo sh -c 'umask 0077; tar -cf $(date "+etc-gitlab-%s.tar") -C / etc/gitlab'
```

You can extract the .tar file as follows.

```shell
# Rename the existing /etc/gitlab, if any
sudo mv /etc/gitlab /etc/gitlab.$(date +%s)
# Change the example timestamp below for your configuration backup
sudo tar -xf etc-gitlab-1399948539.tar -C /
```

Remember to run `sudo gitlab-ctl reconfigure` after restoring a configuration
backup.

NOTE: Your machines SSH host keys are stored in a separate location at `/etc/ssh/`. Be sure to also [backup and restore those keys](https://superuser.com/questions/532040/copy-ssh-keys-from-one-server-to-another-server/532079#532079) to avoid man-in-the-middle attack warnings if you have to perform a full machine restore.

### Configuring the external URL for GitLab

In order for GitLab to display correct repository clone links to your users
it needs to know the URL under which it is reached by your users, e.g.
`http://gitlab.example.com`. Add or edit the following line in
`/etc/gitlab/gitlab.rb`:

```ruby
external_url "http://gitlab.example.com"
```

Run `sudo gitlab-ctl reconfigure` for the change to take effect.


### Storing Git data in an alternative directory

By default, omnibus-gitlab stores Git repository data under
`/var/opt/gitlab/git-data`: repositories are stored in
`/var/opt/gitlab/git-data/repositories`, and satellites in
`/var/opt/gitlab/git-data/gitlab-satellites`.  You can change the location of
the `git-data` parent directory by adding the following line to
`/etc/gitlab/gitlab.rb`.

```ruby
git_data_dir "/mnt/nas/git-data"
```

Note that the target directory and any of its subpaths must not be a symlink.

Run `sudo gitlab-ctl reconfigure` for the change to take effect.

If you already have existing Git repositories in `/var/opt/gitlab/git-data` you
can move them to the new location as follows:

```shell
# Prevent users from writing to the repositories while you move them.
sudo gitlab-ctl stop

# Only move 'repositories'; 'gitlab-satellites' will be recreated
# automatically. Note there is _no_ slash behind 'repositories', but there _is_ a
# slash behind 'git-data'.
sudo rsync -av /var/opt/gitlab/git-data/repositories /mnt/nas/git-data/

# Fix permissions if necessary
sudo gitlab-ctl reconfigure

# Double-check directory layout in /mnt/nas/git-data. Expected output:
# gitlab-satellites  repositories
sudo ls /mnt/nas/git-data/

# Done! Start GitLab and verify that you can browse through the repositories in
# the web interface.
sudo gitlab-ctl start
```

### Changing the name of the Git user / group

By default, omnibus-gitlab uses the user name `git` for Git gitlab-shell login,
ownership of the Git data itself, and SSH URL generation on the web interface.
Similarly, `git` group is used for group ownership of the Git data.  You can
change the user and group by adding the following lines to
`/etc/gitlab/gitlab.rb`.

```ruby
user['username'] = "gitlab"
user['group'] = "gitlab"
```

Run `sudo gitlab-ctl reconfigure` for the change to take effect.

### Setting up LDAP sign-in

See [doc/settings/ldap.md](doc/settings/ldap.md).

### Enable HTTPS

See [doc/settings/nginx.md](doc/settings/nginx.md#enable-https).

#### Redirect `HTTP` requests to `HTTPS`.

See [doc/settings/nginx.md](doc/settings/nginx.md#redirect-http-requests-to-https).

#### Change the default port and the ssl certificate locations.

See [doc/settings/nginx.md](doc/settings/nginx.md#change-the-default-port-and-the-ssl-certificate-locations).

### Use non-packaged web-server

For using an existing Nginx, Passenger, or Apache webserver see [doc/settings/nginx.md](doc/settings/nginx.md#using-a-non-bundled-web-server).

## Using a non-packaged PostgreSQL database management server

To connect to an external PostgreSQL or MySQL DBMS see [doc/settings/database.md](doc/settings/database.md) (MySQL support in the Omnibus Packages is Enterprise Only).

## Using a non-packaged Redis instance

See [doc/settings/redis.md](doc/settings/redis.md).

### Adding ENV Vars to the Gitlab Runtime Environment

See
[doc/settings/environment-variables.md](doc/settings/environment-variables.md).

### Changing gitlab.yml settings

See [doc/settings/gitlab.yml.md](doc/settings/gitlab.yml.md).

### Specify numeric user and group identifiers

Omnibus-gitlab creates users for GitLab, PostgreSQL, Redis and NGINX. You can
specify the numeric identifiers for these users in `/etc/gitlab/gitlab.rb` as
follows.

```ruby
user['uid'] = 1234
user['gid'] = 1234
postgresql['uid'] = 1235
postgresql['gid'] = 1235
redis['uid'] = 1236
redis['gid'] = 1236
web_server['uid'] = 1237
web_server['gid'] = 1237
```

### Sending application email via SMTP

See [doc/settings/smtp.md](doc/settings/smtp.md).

### Omniauth (Google, Twitter, GitHub login)

Omniauth configuration is documented in
[doc.gitlab.com](http://doc.gitlab.com/ce/integration/omniauth.html).

### Adjusting Unicorn settings

See [doc/settings/unicorn.md](doc/settings/unicorn.md).

### Setting the NGINX listen address or addresses

See [doc/settings/nginx.md](doc/settings/nginx.md).

### Inserting custom NGINX settings into the GitLab server block

See [doc/settings/nginx.md](doc/settings/nginx.md).

### Inserting custom settings into the NGINX config

See [doc/settings/nginx.md](doc/settings/nginx.md).

## Backups

If you are using non-packaged database see [documentation on using non-packaged database](doc/settings/database.md#using-a-non-packaged-postgresql-database-management-server).

### Creating an application backup

To create a backup of your repositories and GitLab metadata, follow the [backup create documentation](http://doc.gitlab.com/ce/raketasks/backup_restore.html#create-a-backup-of-the-gitlab-system).

Backup create will store a tar file in `/var/opt/gitlab/backups`.

Similarly for CI, backup create will store a tar file in `/var/opt/gitlab/ci-backups`.

If you want to store your GitLab backups in a different directory, add the
following setting to `/etc/gitlab/gitlab.rb` and run `sudo gitlab-ctl
reconfigure`:

```ruby
gitlab_rails['backup_path'] = '/mnt/backups'
```

### Restoring an application backup

See [backup restore documentation](http://doc.gitlab.com/ce/raketasks/backup_restore.html#omnibus-installations).

### Upload backups to remote (cloud) storage

For details check [backup restore document of GitLab CE](https://gitlab.com/gitlab-org/gitlab-ce/blob/966f68b33e1f15f08e383ec68346ed1bd690b59b/doc/raketasks/backup_restore.md#upload-backups-to-remote-cloud-storage).

## Invoking Rake tasks

To invoke a GitLab Rake task, use `gitlab-rake` (for GitLab) or
`gitlab-ci-rake` (for GitLab CI). For example:

```shell
sudo gitlab-rake gitlab:check
sudo gitlab-ci-rake -T
```

Leave out 'sudo' if you are the 'git' user or the 'gitlab-ci' user.

Contrary to with a traditional GitLab installation, there is no need to change
the user or the `RAILS_ENV` environment variable; this is taken care of by the
`gitlab-rake` and `gitlab-ci-rake` wrapper scripts.

## Directory structure

Omnibus-gitlab uses four different directories.

- `/opt/gitlab` holds application code for GitLab and its dependencies.
- `/var/opt/gitlab` holds application data and configuration files that
  `gitlab-ctl reconfigure` writes to.
- `/etc/gitlab` holds configuration files for omnibus-gitlab. These are
  the only files that you should ever have to edit manually.
- `/var/log/gitlab` contains all log data generated by components of
  omnibus-gitlab.

## Omnibus-gitlab and SELinux

Although omnibus-gitlab runs on systems that have SELinux enabled, it does not
use SELinux confinement features:
- omnibus-gitlab creates unconfined system users;
- omnibus-gitlab services run in an unconfined context.

The correct operation of Git access via SSH depends on the labeling of
`/var/opt/gitlab/.ssh`. If needed you can restore this labeling by running
`sudo gitlab-ctl reconfigure`.

Depending on your platform, `gitlab-ctl reconfigure` will install SELinux
modules required to make GitLab work. These modules are listed in
[files/gitlab-selinux/README.md](files/gitlab-selinux/README.md).

NSA, if you're reading this, we'd really appreciate it if you could contribute back a SELinux profile for omnibus-gitlab :)
Of course, if anyone else is reading this, you're welcome to contribute the SELinux profile too.

## Logs

### Tail logs in a console on the server

If you want to 'tail', i.e. view live log updates of GitLab logs you can use
`gitlab-ctl tail`.

```shell
# Tail all logs; press Ctrl-C to exit
sudo gitlab-ctl tail

# Drill down to a sub-directory of /var/log/gitlab
sudo gitlab-ctl tail gitlab-rails

# Drill down to an individual file
sudo gitlab-ctl tail nginx/gitlab_error.log
```

### Runit logs

The Runit-managed services in omnibus-gitlab generate log data using
[svlogd][svlogd]. See the [svlogd documentation][svlogd] for more information
about the files it generates.

You can modify svlogd settings via `/etc/gitlab/gitlab.rb` with the following settings:

```ruby
# Below are the default values
logging['svlogd_size'] = 200 * 1024 * 1024 # rotate after 200 MB of log data
logging['svlogd_num'] = 30 # keep 30 rotated log files
logging['svlogd_timeout'] = 24 * 60 * 60 # rotate after 24 hours
logging['svlogd_filter'] = "gzip" # compress logs with gzip
logging['svlogd_udp'] = nil # transmit log messages via UDP
logging['svlogd_prefix'] = nil # custom prefix for log messages

# Optionally, you can override the prefix for e.g. Nginx
nginx['svlogd_prefix'] = "nginx"
```

### Logrotate

Starting with omnibus-gitlab 7.4 there is a built-in logrotate service in
omnibus-gitlab. This service will rotate, compress and eventually delete the
log data that is not captured by Runit, such as `gitlab-rails/production.log`
and `nginx/gitlab_access.log`. You can configure logrotate via
`/etc/gitlab/gitlab.rb`.

```
# Below are some of the default settings
logging['logrotate_frequency'] = "daily" # rotate logs daily
logging['logrotate_size'] = nil # do not rotate by size by default
logging['logrotate_rotate'] = 30 # keep 30 rotated logs
logging['logrotate_compress'] = "compress" # see 'man logrotate'
logging['logrotate_method'] = "copytruncate" # see 'man logrotate'
logging['logrotate_postrotate'] = nil # no postrotate command by default

# You can add overrides per service
nginx['logrotate_frequency'] = nil
nginx['logrotate_size'] = "200M"

# You can also disable the built-in logrotate service if you want
logrotate['enable'] = false
```

### UDP log shipping (GitLab Enterprise Edition only)

You can configure omnibus-gitlab to send syslog-ish log messages via UDP.

```ruby
logging['udp_log_shipping_host'] = '1.2.3.4' # Your syslog server
logging['udp_log_shipping_port'] = 1514 # Optional, defaults to 514 (syslog)
```

Example log messages:

```
<13>Jun 26 06:33:46 ubuntu1204-test production.log: Started GET "/root/my-project/import" for 127.0.0.1 at 2014-06-26 06:33:46 -0700
<13>Jun 26 06:33:46 ubuntu1204-test production.log: Processing by ProjectsController#import as HTML
<13>Jun 26 06:33:46 ubuntu1204-test production.log: Parameters: {"id"=>"root/my-project"}
<13>Jun 26 06:33:46 ubuntu1204-test production.log: Completed 200 OK in 122ms (Views: 71.9ms | ActiveRecord: 12.2ms)
<13>Jun 26 06:33:46 ubuntu1204-test gitlab_access.log: 172.16.228.1 - - [26/Jun/2014:06:33:46 -0700] "GET /root/my-project/import HTTP/1.1" 200 5775 "https://172.16.228.169/root/my-project/import" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
2014-06-26_13:33:46.49866 ubuntu1204-test sidekiq: 2014-06-26T13:33:46Z 18107 TID-7nbj0 Sidekiq::Extensions::DelayedMailer JID-bbfb118dd1db20f6c39f5b50 INFO: start

2014-06-26_13:33:46.52608 ubuntu1204-test sidekiq: 2014-06-26T13:33:46Z 18107 TID-7muoc RepositoryImportWorker JID-57ee926c3655fcfa062338ae INFO: start

```

## Starting a Rails console session

If you need access to a Rails production console for your GitLab installation
you can start one with the command below. Please be warned that it is very easy
to inadvertently modify, corrupt or destroy data from the console.

```shell
# start a Rails console for GitLab
sudo gitlab-rails console

# start a Rails console for GitLab CI
sudo gitlab-ci-rails console
```

This will only work after you have run `gitlab-ctl reconfigure` at least once.

## Using a MySQL database management server (Enterprise Edition only)

See [doc/settings/database.md](doc/settings/database.md).

### Create a user and database for GitLab

See [doc/settings/database.md](doc/settings/database.md).

### Configure omnibus-gitlab to connect to it

See [doc/settings/database.md](doc/settings/database.md).

### Seed the database (fresh installs only)

See [doc/settings/database.md](doc/settings/database.md).

## Only start omnibus-gitlab services after a given filesystem is mounted

If you want to prevent omnibus-gitlab services (nginx, redis, unicorn etc.)
from starting before a given filesystem is mounted, add the following to
`/etc/gitlab/gitlab.rb`:

```ruby
# wait for /var/opt/gitlab to be mounted
high_availability['mountpoint'] = '/var/opt/gitlab'
```

## Building your own package

See [the separate build documentation](doc/build.md).

## Running a custom GitLab version

It is not recommended to make changes to any of the files in `/opt/gitlab`
after installing omnibus-gitlab: they will either conflict with or be
overwritten by future updates. If you want to run a custom version of GitLab
you can [build your own package](doc/build.md) or use [another installation
method][CE README].

## Acknowledgments

This omnibus installer project is based on the awesome work done by Chef in
[omnibus-chef-server][omnibus-chef-server].

[downloads]: https://about.gitlab.com/downloads/
[CE README]: https://gitlab.com/gitlab-org/gitlab-ce/blob/master/README.md
[omnibus-chef-server]: https://github.com/opscode/omnibus-chef-server
[database.yml.mysql]: https://gitlab.com/gitlab-org/gitlab-ce/blob/master/config/database.yml.mysql
[svlogd]: http://smarden.org/runit/svlogd.8.html
[installation]: https://about.gitlab.com/installation/
[gitlab.rb.template]: https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/files/gitlab-config-template/gitlab.rb.template
