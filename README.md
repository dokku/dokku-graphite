# dokku graphite [![Build Status](https://img.shields.io/travis/dokku/dokku-graphite-grafana.svg?branch=master "Build Status")](https://travis-ci.org/dokku/dokku-graphite-grafana) [![IRC Network](https://img.shields.io/badge/irc-freenode-blue.svg "IRC Freenode")](https://webchat.freenode.net/?channels=dokku)

Official grafana/graphite/statsd plugin for dokku. Currently defaults to installing [docker-grafana-graphite 3.0.1](https://hub.docker.com/r/dokkupaas/docker-grafana-graphite) ([source](https://github.com/dokku/docker-grafana-graphite)).

## requirements

- dokku 0.12.x+
- docker 1.8.x

## installation

```shell
# on 0.4.x+
sudo dokku plugin:install https://github.com/dokku/dokku-graphite-grafana.git graphite
```

## commands

```
graphite:app-links <app>          List all graphite service links for a given app
graphite:backup <name> <bucket> (--use-iam) NOT IMPLEMENTED
graphite:backup-auth <name> <aws_access_key_id> <aws_secret_access_key> (<aws_default_region>) (<aws_signature_version>) (<endpoint_url>) NOT IMPLEMENTED
graphite:backup-deauth <name>     NOT IMPLEMENTED
graphite:backup-schedule <name> <schedule> <bucket> NOT IMPLEMENTED
graphite:backup-schedule-cat <name> NOT IMPLEMENTED
graphite:backup-set-encryption <name> <passphrase> NOT IMPLEMENTED
graphite:backup-unschedule <name> NOT IMPLEMENTED
graphite:backup-unset-encryption <name> NOT IMPLEMENTED
graphite:clone <name> <new-name>  NOT IMPLEMENTED
graphite:connect <name>           NOT IMPLEMENTED
graphite:create <name>            Create a graphite service with environment variables
graphite:destroy <name>           Delete the service, delete the data and stop its container if there are no links left
graphite:enter <name> [command]   Enter or run a command in a running graphite service container
graphite:exists <service>         Check if the graphite service exists
graphite:export <name> > <file>   NOT IMPLEMENTED
graphite:expose <name> [port]     Expose a graphite service on custom port if provided (random port otherwise)
graphite:import <name> <file>     NOT IMPLEMENTED
graphite:info <name>              Print the connection information
graphite:link <name> <app>        Link the graphite service to the app
graphite:linked <name> <app>      Check if the graphite service is linked to an app
graphite:list                     List all graphite services
graphite:logs <name> [-t]         Print the most recent log(s) for this service
graphite:nginx-expose <name> <domain> Expose the graphite service's grafana via an nginx vhost
graphite:nginx-unexpose <name>    Unexpose the graphite service's grafana
graphite:promote <name> <app>     Promote service <name> as STATSD_URL in <app>
graphite:restart <name>           Graceful shutdown and restart of the graphite service container
graphite:start <name>             Start a previously stopped graphite service
graphite:stop <name>              Stop a running graphite service
graphite:unexpose <name>          Unexpose a previously exposed graphite service
graphite:unlink <name> <app>      Unlink the graphite service from the app
graphite:upgrade <name>           Upgrade service <service> to the specified version
```

## usage

```shell
# create a graphite service named lolipop
dokku graphite:create lolipop

# you can also specify the image and image
# version to use for the service
# it *must* be compatible with the
# dokkupaas/docker-grafana-graphite image
export GRAPHITE_IMAGE="dokkupaas/docker-grafana-graphite"
export GRAPHITE_IMAGE_VERSION="3.0.1"
dokku graphite:create lolipop

# you can also specify custom environment
# variables to start the graphite service
# in semi-colon separated form
export GRAPHITE_CUSTOM_ENV="USER=alpha;HOST=beta"
dokku graphite:create lolipop

# get connection information as follows
dokku graphite:info lolipop

# you can also retrieve a specific piece of service info via flags
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

# a bash prompt can be opened against a running service
# filesystem changes will not be saved to disk
dokku graphite:enter lolipop

# you may also run a command directly against the service
# filesystem changes will not be saved to disk
dokku graphite:enter lolipop ls -lah /

# a graphite service can be linked to a
# container this will use native docker
# links via the docker-options plugin
# here we link it to our 'playground' app
# NOTE: this will restart your app
dokku graphite:link lolipop playground

# the following environment variables will be set automatically by docker (not
# on the app itself, so they won’t be listed when calling dokku config)
#
#   DOKKU_GRAPHITE_LOLIPOP_NAME=/random_name/GRAPHITE
#   DOKKU_GRAPHITE_LOLIPOP_PORT=tcp://172.17.0.1:8125
#   DOKKU_GRAPHITE_LOLIPOP_PORT_8125_UDP=udp://172.17.0.1:8125
#   DOKKU_GRAPHITE_LOLIPOP_PORT_8125_UDP_PROTO=tcp
#   DOKKU_GRAPHITE_LOLIPOP_PORT_8125_UDP_PORT=8125
#   DOKKU_GRAPHITE_LOLIPOP_PORT_8125_UDP_ADDR=172.17.0.1
#
# and the following will be set on the linked application by default
#
#   STATSD_URL=statsd://dokku-graphite-lolipop:8125
#
# NOTE: the host exposed here only works internally in docker containers. If
# you want your container to be reachable from outside, you should use `expose`.

# another service can be linked to your app
dokku graphite:link other_service playground

# since STATSD_URL is already in use, another environment variable will be
# generated automatically
#
#   DOKKU_STATSD_BLUE_URL=statsd://dokku-graphite-other_service:8125

# you can then promote the new service to be the primary one
# NOTE: this will restart your app
dokku graphite:promote other_service playground

# this will replace STATSD_URL with the url from other_service and generate
# another environment variable to hold the previous value if necessary.
# you could end up with the following for example:
#
#   STATSD_URL=statsd://dokku-graphite-other_service:8125
#   DOKKU_STATSD_BLUE_URL=statsd://dokku-graphite-other_service:8125
#   DOKKU_STATSD_SILVER_URL=statsd://dokku-graphite-lolipop:8125

# you can also unlink a graphite service
# NOTE: this will restart your app and unset related environment variables
dokku graphite:unlink lolipop playground

# you can tail logs for a particular service
dokku graphite:logs lolipop
dokku graphite:logs lolipop -t # to tail

# finally, you can destroy the container
dokku graphite:destroy lolipop
```

## Changing database adapter

It's possible to change the protocol for STATSD_URL by setting
the environment variable STATSD_DATABASE_SCHEME on the app:

```
dokku config:set playground STATSD_DATABASE_SCHEME=statsd2
dokku graphite:link lolipop playground
```

Will cause STATSD_URL to be set as
statsd2://dokku-graphite-lolipop:8125

CAUTION: Changing STATSD_DATABASE_SCHEME after linking will cause dokku to
believe the graphite is not linked when attempting to use `dokku graphite:unlink`
or `dokku graphite:promote`.
You should be able to fix this by

- Changing STATSD_URL manually to the new value.

OR

- Set STATSD_DATABASE_SCHEME back to its original setting
- Unlink the service
- Change STATSD_DATABASE_SCHEME to the desired setting
- Relink the service

## Disabling `docker pull` calls

If you wish to disable the `docker pull` calls that the plugin triggers, you may set the `GRAPHITE_DISABLE_PULL` environment variable to `true`. Once disabled, you will need to pull the service image you wish to deploy as shown in the `stderr` output.

Please ensure the proper images are in place when `docker pull` is disabled.
