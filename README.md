# dokku graphite [![Build Status](https://img.shields.io/travis/dokku/dokku-graphite.svg?branch=master "Build Status")](https://travis-ci.org/dokku/dokku-graphite) [![IRC Network](https://img.shields.io/badge/irc-freenode-blue.svg "IRC Freenode")](https://webchat.freenode.net/?channels=dokku)

Official graphite plugin for dokku. Currently defaults to installing [graphite 6.4.4](https://hub.docker.com/_/graphite/).

## Requirements

- dokku 0.12.x+
- docker 1.8.x

## Installation

```shell
# on 0.12.x+
sudo dokku plugin:install https://github.com/dokku/dokku-graphite.git graphite
```

## Commands

```
graphite:app-links <app>                           # list all graphite service links for a given app
graphite:backup <service> <bucket-name> [--use-iam] # creates a backup of the graphite service to an existing s3 bucket
graphite:backup-auth <service> <aws-access-key-id> <aws-secret-access-key> <aws-default-region> <aws-signature-version> <endpoint-url> # sets up authentication for backups on the graphite service
graphite:backup-deauth <service>                   # removes backup authentication for the graphite service
graphite:backup-schedule <service> <schedule> <bucket-name> [--use-iam] # schedules a backup of the graphite service
graphite:backup-schedule-cat <service>             # cat the contents of the configured backup cronfile for the service
graphite:backup-set-encryption <service> <passphrase> # sets encryption for all future backups of graphite service
graphite:backup-unschedule <service>               # unschedules the backup of the graphite service
graphite:backup-unset-encryption <service>         # unsets encryption for future backups of the graphite service
graphite:clone <service> <new-service> [--clone-flags...] # create container <new-name> then copy data from <name> into <new-name>
graphite:connect <service>                         # connect to the service via the graphite connection tool
graphite:create <service> [--create-flags...]      # create a graphite service
graphite:destroy <service> [-f|--force]            # delete the graphite service/data/container if there are no links left
graphite:enter <service>                           # enter or run a command in a running graphite service container
graphite:exists <service>                          # check if the graphite service exists
graphite:export <service>                          # export a dump of the graphite service database
graphite:expose <service> <ports...>               # expose a graphite service on custom port if provided (random port otherwise)
graphite:import <service>                          # import a dump into the graphite service database
graphite:info <service> [--single-info-flag]       # print the connection information
graphite:link <service> <app> [--link-flags...]    # link the graphite service to the app
graphite:linked <service> <app>                    # check if the graphite service is linked to an app
graphite:links <service>                           # list all apps linked to the graphite service
graphite:list                                      # list all graphite services
graphite:logs <service> [-t|--tail]                # print the most recent log(s) for this service
graphite:nginx-expose <service> <domain>           # expose the graphite service's grafana via an nginx vhost
graphite:nginx-unexpose <service>                  # expose the graphite service's grafana via an nginx vhost
graphite:promote <service> <app>                   # promote service <service> as STATSD_URL in <app>
graphite:restart <service>                         # graceful shutdown and restart of the graphite service container
graphite:start <service>                           # start a previously stopped graphite service
graphite:stop <service>                            # stop a running graphite service
graphite:unexpose <service>                        # unexpose a previously exposed graphite service
graphite:unlink <service> <app>                    # unlink the graphite service from the app
graphite:upgrade <service> [--upgrade-flags...]    # upgrade service <service> to the specified versions
```

## Usage

Help for any commands can be displayed by specifying the command as an argument to graphite:help. Please consult the `graphite:help` command for any undocumented commands.

### Basic Usage
### list all graphite services

```shell
# usage
dokku graphite:list 
```

examples:

List all services:

```shell
dokku graphite:list
```
### create a graphite service

```shell
# usage
dokku graphite:create <service> [--create-flags...]
```

examples:

Create a graphite service named lolipop:

```shell
dokku graphite:create lolipop
```

You can also specify the image and image version to use for the service. It *must* be compatible with the ${plugin_image} image. :

```shell
export STATSD_IMAGE="${PLUGIN_IMAGE}"
export STATSD_IMAGE_VERSION="${PLUGIN_IMAGE_VERSION}"
dokku graphite:create lolipop
```

You can also specify custom environment variables to start the graphite service in semi-colon separated form. :

```shell
export STATSD_CUSTOM_ENV="USER=alpha;HOST=beta"
dokku graphite:create lolipop
```
### print the connection information

```shell
# usage
dokku graphite:info <service> [--single-info-flag]
```

examples:

Get connection information as follows:

```shell
dokku graphite:info lolipop
```

You can also retrieve a specific piece of service info via flags:

```shell
dokku graphite:info lolipop --config-dir
dokku graphite:info lolipop --data-dir
dokku graphite:info lolipop --dsn
dokku graphite:info lolipop --exposed-ports
dokku graphite:info lolipop --id
dokku graphite:info lolipop --internal-ip
dokku graphite:info lolipop --links
dokku graphite:info lolipop --service-root
dokku graphite:info lolipop --status
dokku graphite:info lolipop --version
```
### print the most recent log(s) for this service

```shell
# usage
dokku graphite:logs <service> [-t|--tail]
```

examples:

You can tail logs for a particular service:

```shell
dokku graphite:logs lolipop
```

By default, logs will not be tailed, but you can do this with the --tail flag:

```shell
dokku graphite:logs lolipop --tail
```
### link the graphite service to the app

```shell
# usage
dokku graphite:link <service> <app> [--link-flags...]
```

examples:

A graphite service can be linked to a container. This will use native docker links via the docker-options plugin. Here we link it to our 'playground' app. :

> NOTE: this will restart your app

```shell
dokku graphite:link lolipop playground
```

The following environment variables will be set automatically by docker (not on the app itself, so they won’t be listed when calling dokku config):

```
DOKKU_STATSD_LOLIPOP_NAME=/lolipop/DATABASE
DOKKU_STATSD_LOLIPOP_PORT=tcp://172.17.0.1:8125
DOKKU_STATSD_LOLIPOP_PORT_8125_TCP=tcp://172.17.0.1:8125
DOKKU_STATSD_LOLIPOP_PORT_8125_TCP_PROTO=tcp
DOKKU_STATSD_LOLIPOP_PORT_8125_TCP_PORT=8125
DOKKU_STATSD_LOLIPOP_PORT_8125_TCP_ADDR=172.17.0.1
```

The following will be set on the linked application by default:

```
STATSD_URL=statsd://lolipop:SOME_PASSWORD@dokku-graphite-lolipop:8125/lolipop
```

The host exposed here only works internally in docker containers. If you want your container to be reachable from outside, you should use the 'expose' subcommand. Another service can be linked to your app:

```shell
dokku graphite:link other_service playground
```

It is possible to change the protocol for statsd_url by setting the environment variable statsd_database_scheme on the app. Doing so will after linking will cause the plugin to think the service is not linked, and we advise you to unlink before proceeding. :

```shell
dokku config:set playground STATSD_DATABASE_SCHEME=statsd2
dokku graphite:link lolipop playground
```

This will cause statsd_url to be set as:

```
statsd2://lolipop:SOME_PASSWORD@dokku-graphite-lolipop:8125/lolipop
```
### unlink the graphite service from the app

```shell
# usage
dokku graphite:unlink <service> <app>
```

examples:

You can unlink a graphite service:

> NOTE: this will restart your app and unset related environment variables

```shell
dokku graphite:unlink lolipop playground
```
### delete the graphite service/data/container if there are no links left

```shell
# usage
dokku graphite:destroy <service> [-f|--force]
```

examples:

Destroy the service, it's data, and the running container:

```shell
dokku graphite:destroy lolipop
```

### Service Lifecycle

The lifecycle of each service can be managed through the following commands:

### connect to the service via the graphite connection tool

```shell
# usage
dokku graphite:connect <service>
```

examples:

Connect to the service via the graphite connection tool:

```shell
dokku graphite:connect lolipop
```
### enter or run a command in a running graphite service container

```shell
# usage
dokku graphite:enter <service>
```

examples:

A bash prompt can be opened against a running service. Filesystem changes will not be saved to disk. :

```shell
dokku graphite:enter lolipop
```

You may also run a command directly against the service. Filesystem changes will not be saved to disk. :

```shell
dokku graphite:enter lolipop touch /tmp/test
```
### expose a graphite service on custom port if provided (random port otherwise)

```shell
# usage
dokku graphite:expose <service> <ports...>
```

examples:

Expose the service on the service's normal ports, allowing access to it from the public interface (0. 0. 0. 0):

```shell
dokku graphite:expose lolipop ${PLUGIN_DATASTORE_PORTS[@]}
```
### unexpose a previously exposed graphite service

```shell
# usage
dokku graphite:unexpose <service>
```

examples:

Unexpose the service, removing access to it from the public interface (0. 0. 0. 0):

```shell
dokku graphite:unexpose lolipop
```
### promote service <service> as STATSD_URL in <app>

```shell
# usage
dokku graphite:promote <service> <app>
```

examples:

If you have a graphite service linked to an app and try to link another graphite service another link environment variable will be generated automatically:

```
DOKKU_STATSD_BLUE_URL=statsd://other_service:ANOTHER_PASSWORD@dokku-graphite-other-service:8125/other_service
```

You can promote the new service to be the primary one:

> NOTE: this will restart your app

```shell
dokku graphite:promote other_service playground
```

This will replace statsd_url with the url from other_service and generate another environment variable to hold the previous value if necessary. You could end up with the following for example:

```
STATSD_URL=statsd://other_service:ANOTHER_PASSWORD@dokku-graphite-other-service:8125/other_service
DOKKU_STATSD_BLUE_URL=statsd://other_service:ANOTHER_PASSWORD@dokku-graphite-other-service:8125/other_service
DOKKU_STATSD_SILVER_URL=statsd://lolipop:SOME_PASSWORD@dokku-graphite-lolipop:8125/lolipop
```
### graceful shutdown and restart of the graphite service container

```shell
# usage
dokku graphite:restart <service>
```

examples:

Restart the service:

```shell
dokku graphite:restart lolipop
```
### start a previously stopped graphite service

```shell
# usage
dokku graphite:start <service>
```

examples:

Start the service:

```shell
dokku graphite:start lolipop
```
### stop a running graphite service

```shell
# usage
dokku graphite:stop <service>
```

examples:

Stop the service and the running container:

```shell
dokku graphite:stop lolipop
```
### upgrade service <service> to the specified versions

```shell
# usage
dokku graphite:upgrade <service> [--upgrade-flags...]
```

examples:

You can upgrade an existing service to a new image or image-version:

```shell
dokku graphite:upgrade lolipop
```

### Service Automation

Service scripting can be executed using the following commands:

### list all graphite service links for a given app

```shell
# usage
dokku graphite:app-links <app>
```

examples:

List all graphite services that are linked to the 'playground' app. :

```shell
dokku graphite:app-links playground
```
### create container <new-name> then copy data from <name> into <new-name>

```shell
# usage
dokku graphite:clone <service> <new-service> [--clone-flags...]
```

examples:

You can clone an existing service to a new one:

```shell
dokku graphite:clone lolipop lolipop-2
```
### check if the graphite service exists

```shell
# usage
dokku graphite:exists <service>
```

examples:

Here we check if the lolipop graphite service exists. :

```shell
dokku graphite:exists lolipop
```
### check if the graphite service is linked to an app

```shell
# usage
dokku graphite:linked <service> <app>
```

examples:

Here we check if the lolipop graphite service is linked to the 'playground' app. :

```shell
dokku graphite:linked lolipop playground
```
### list all apps linked to the graphite service

```shell
# usage
dokku graphite:links <service>
```

examples:

List all apps linked to the 'lolipop' graphite service. :

```shell
dokku graphite:links lolipop
```

### Data Management

The underlying service data can be imported and exported with the following commands:

### import a dump into the graphite service database

```shell
# usage
dokku graphite:import <service>
```

examples:

Import a datastore dump:

```shell
dokku graphite:import lolipop < database.dump
```
### export a dump of the graphite service database

```shell
# usage
dokku graphite:export <service>
```

examples:

By default, datastore output is exported to stdout:

```shell
dokku graphite:export lolipop
```

You can redirect this output to a file:

```shell
dokku graphite:export lolipop > lolipop.dump
```

### Backups

Datastore backups are supported via AWS S3 and S3 compatible services like [minio](https://github.com/minio/minio).

You may skip the `backup-auth` step if your dokku install is running within EC2 and has access to the bucket via an IAM profile. In that case, use the `--use-iam` option with the `backup` command.

Backups can be performed using the backup commands:

### sets up authentication for backups on the graphite service

```shell
# usage
dokku graphite:backup-auth <service> <aws-access-key-id> <aws-secret-access-key> <aws-default-region> <aws-signature-version> <endpoint-url>
```

examples:

Setup s3 backup authentication:

```shell
dokku graphite:backup-auth lolipop AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY
```

Setup s3 backup authentication with different region:

```shell
dokku graphite:backup-auth lolipop AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_REGION
```

Setup s3 backup authentication with different signature version and endpoint:

```shell
dokku graphite:backup-auth lolipop AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_REGION AWS_SIGNATURE_VERSION ENDPOINT_URL
```

More specific example for minio auth:

```shell
dokku graphite:backup-auth lolipop MINIO_ACCESS_KEY_ID MINIO_SECRET_ACCESS_KEY us-east-1 s3v4 https://YOURMINIOSERVICE
```
### removes backup authentication for the graphite service

```shell
# usage
dokku graphite:backup-deauth <service>
```

examples:

Remove s3 authentication:

```shell
dokku graphite:backup-deauth lolipop
```
### creates a backup of the graphite service to an existing s3 bucket

```shell
# usage
dokku graphite:backup <service> <bucket-name> [--use-iam]
```

examples:

Backup the 'lolipop' service to the 'my-s3-bucket' bucket on aws:

```shell
dokku graphite:backup lolipop my-s3-bucket --use-iam
```
### sets encryption for all future backups of graphite service

```shell
# usage
dokku graphite:backup-set-encryption <service> <passphrase>
```

examples:

Set a gpg passphrase for backups:

```shell
dokku graphite:backup-set-encryption lolipop
```
### unsets encryption for future backups of the graphite service

```shell
# usage
dokku graphite:backup-unset-encryption <service>
```

examples:

Unset a gpg encryption key for backups:

```shell
dokku graphite:backup-unset-encryption lolipop
```
### schedules a backup of the graphite service

```shell
# usage
dokku graphite:backup-schedule <service> <schedule> <bucket-name> [--use-iam]
```

examples:

Schedule a backup:

> 'schedule' is a crontab expression, eg. "0 3 * * *" for each day at 3am

```shell
dokku graphite:backup-schedule lolipop "0 3 * * *" my-s3-bucket
```

Schedule a backup and authenticate via iam:

```shell
dokku graphite:backup-schedule lolipop "0 3 * * *" my-s3-bucket --use-iam
```
### cat the contents of the configured backup cronfile for the service

```shell
# usage
dokku graphite:backup-schedule-cat <service>
```

examples:

Cat the contents of the configured backup cronfile for the service:

```shell
dokku graphite:backup-schedule-cat lolipop
```
### unschedules the backup of the graphite service

```shell
# usage
dokku graphite:backup-unschedule <service>
```

examples:

Remove the scheduled backup from cron:

```shell
dokku graphite:backup-unschedule lolipop
```

### Disabling `docker pull` calls

If you wish to disable the `docker pull` calls that the plugin triggers, you may set the `GRAPHITE_DISABLE_PULL` environment variable to `true`. Once disabled, you will need to pull the service image you wish to deploy as shown in the `stderr` output.

Please ensure the proper images are in place when `docker pull` is disabled.