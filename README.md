# dokku graphite [![Build Status](https://img.shields.io/circleci/project/github/dokku/dokku-graphite.svg?branch=master&style=flat-square "Build Status")](https://circleci.com/gh/dokku/dokku-graphite/tree/master) [![IRC Network](https://img.shields.io/badge/irc-freenode-blue.svg?style=flat-square "IRC Freenode")](https://webchat.freenode.net/?channels=dokku)

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
graphite:app-links <app>                        # list all graphite service links for a given app
graphite:create <service> [--create-flags...]   # create a graphite service
graphite:destroy <service> [-f|--force]         # delete the graphite service/data/container if there are no links left
graphite:enter <service>                        # enter or run a command in a running graphite service container
graphite:exists <service>                       # check if the graphite service exists
graphite:expose <service> <ports...>            # expose a graphite service on custom port if provided (random port otherwise)
graphite:info <service> [--single-info-flag]    # print the service information
graphite:link <service> <app> [--link-flags...] # link the graphite service to the app
graphite:linked <service> <app>                 # check if the graphite service is linked to an app
graphite:links <service>                        # list all apps linked to the graphite service
graphite:list                                   # list all graphite services
graphite:logs <service> [-t|--tail]             # print the most recent log(s) for this service
graphite:nginx-expose <service> <domain>        # expose the graphite service's grafana via an nginx vhost
graphite:nginx-unexpose <service>               # expose the graphite service's grafana via an nginx vhost
graphite:promote <service> <app>                # promote service <service> as STATSD_URL in <app>
graphite:restart <service>                      # graceful shutdown and restart of the graphite service container
graphite:start <service>                        # start a previously stopped graphite service
graphite:stop <service>                         # stop a running graphite service
graphite:unexpose <service>                     # unexpose a previously exposed graphite service
graphite:unlink <service> <app>                 # unlink the graphite service from the app
graphite:upgrade <service> [--upgrade-flags...] # upgrade service <service> to the specified versions
```

## Usage

Help for any commands can be displayed by specifying the command as an argument to graphite:help. Please consult the `graphite:help` command for any undocumented commands.

### Basic Usage

### create a graphite service

```shell
# usage
dokku graphite:create <service> [--create-flags...]
```

flags:

- `-C|--custom-env "USER=alpha;HOST=beta"`: semi-colon delimited environment variables to start the service with
- `-i|--image IMAGE`: the image name to start the service with
- `-I|--image-version IMAGE_VERSION`: the image version to start the service with
- `-p|--password PASSWORD`: override the user-level service password
- `-r|--root-password PASSWORD`: override the root-level service password

Create a graphite service named lolipop:

```shell
dokku graphite:create lolipop
```

You can also specify the image and image version to use for the service. It *must* be compatible with the dokku/docker-grafana-graphite image. 

```shell
export STATSD_IMAGE="dokku/docker-grafana-graphite"
export STATSD_IMAGE_VERSION="${PLUGIN_IMAGE_VERSION}"
dokku graphite:create lolipop
```

You can also specify custom environment variables to start the graphite service in semi-colon separated form. 

```shell
export STATSD_CUSTOM_ENV="USER=alpha;HOST=beta"
dokku graphite:create lolipop
```

### print the service information

```shell
# usage
dokku graphite:info <service> [--single-info-flag]
```

flags:

- `--config-dir`: show the service configuration directory
- `--data-dir`: show the service data directory
- `--dsn`: show the service DSN
- `--exposed-ports`: show service exposed ports
- `--id`: show the service container id
- `--internal-ip`: show the service internal ip
- `--links`: show the service app links
- `--service-root`: show the service root directory
- `--status`: show the service running status
- `--version`: show the service image version

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

### list all graphite services

```shell
# usage
dokku graphite:list 
```

List all services:

```shell
dokku graphite:list
```

### print the most recent log(s) for this service

```shell
# usage
dokku graphite:logs <service> [-t|--tail]
```

flags:

- `-t|--tail`: do not stop when end of the logs are reached and wait for additional output

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

flags:

- `-a|--alias "BLUE_DATABASE"`: an alternative alias to use for linking to an app via environment variable
- `-q|--querystring "pool=5"`: ampersand delimited querystring arguments to append to the service link

A graphite service can be linked to a container. This will use native docker links via the docker-options plugin. Here we link it to our 'playground' app. 

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

It is possible to change the protocol for `STATSD_URL` by setting the environment variable `STATSD_DATABASE_SCHEME` on the app. Doing so will after linking will cause the plugin to think the service is not linked, and we advise you to unlink before proceeding. 

```shell
dokku config:set playground STATSD_DATABASE_SCHEME=statsd2
dokku graphite:link lolipop playground
```

This will cause `STATSD_URL` to be set as:

```
statsd2://lolipop:SOME_PASSWORD@dokku-graphite-lolipop:8125/lolipop
```

### unlink the graphite service from the app

```shell
# usage
dokku graphite:unlink <service> <app>
```

You can unlink a graphite service:

> NOTE: this will restart your app and unset related environment variables

```shell
dokku graphite:unlink lolipop playground
```

### Service Lifecycle

The lifecycle of each service can be managed through the following commands:

### enter or run a command in a running graphite service container

```shell
# usage
dokku graphite:enter <service>
```

A bash prompt can be opened against a running service. Filesystem changes will not be saved to disk. 

```shell
dokku graphite:enter lolipop
```

You may also run a command directly against the service. Filesystem changes will not be saved to disk. 

```shell
dokku graphite:enter lolipop touch /tmp/test
```

### expose a graphite service on custom port if provided (random port otherwise)

```shell
# usage
dokku graphite:expose <service> <ports...>
```

Expose the service on the service's normal ports, allowing access to it from the public interface (`0.0.0.0`):

```shell
dokku graphite:expose lolipop 8125 8126 80 2003
```

### unexpose a previously exposed graphite service

```shell
# usage
dokku graphite:unexpose <service>
```

Unexpose the service, removing access to it from the public interface (`0.0.0.0`):

```shell
dokku graphite:unexpose lolipop
```

### promote service <service> as STATSD_URL in <app>

```shell
# usage
dokku graphite:promote <service> <app>
```

If you have a graphite service linked to an app and try to link another graphite service another link environment variable will be generated automatically:

```
DOKKU_STATSD_BLUE_URL=statsd://other_service:ANOTHER_PASSWORD@dokku-graphite-other-service:8125/other_service
```

You can promote the new service to be the primary one:

> NOTE: this will restart your app

```shell
dokku graphite:promote other_service playground
```

This will replace `STATSD_URL` with the url from other_service and generate another environment variable to hold the previous value if necessary. You could end up with the following for example:

```
STATSD_URL=statsd://other_service:ANOTHER_PASSWORD@dokku-graphite-other-service:8125/other_service
DOKKU_STATSD_BLUE_URL=statsd://other_service:ANOTHER_PASSWORD@dokku-graphite-other-service:8125/other_service
DOKKU_STATSD_SILVER_URL=statsd://lolipop:SOME_PASSWORD@dokku-graphite-lolipop:8125/lolipop
```

### start a previously stopped graphite service

```shell
# usage
dokku graphite:start <service>
```

Start the service:

```shell
dokku graphite:start lolipop
```

### stop a running graphite service

```shell
# usage
dokku graphite:stop <service>
```

Stop the service and the running container:

```shell
dokku graphite:stop lolipop
```

### graceful shutdown and restart of the graphite service container

```shell
# usage
dokku graphite:restart <service>
```

Restart the service:

```shell
dokku graphite:restart lolipop
```

### upgrade service <service> to the specified versions

```shell
# usage
dokku graphite:upgrade <service> [--upgrade-flags...]
```

flags:

- `-C|--custom-env "USER=alpha;HOST=beta"`: semi-colon delimited environment variables to start the service with
- `-i|--image IMAGE`: the image name to start the service with
- `-I|--image-version IMAGE_VERSION`: the image version to start the service with
- `-R|--restart-apps "true"`: whether to force an app restart

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

List all graphite services that are linked to the 'playground' app. 

```shell
dokku graphite:app-links playground
```

### check if the graphite service exists

```shell
# usage
dokku graphite:exists <service>
```

Here we check if the lolipop graphite service exists. 

```shell
dokku graphite:exists lolipop
```

### check if the graphite service is linked to an app

```shell
# usage
dokku graphite:linked <service> <app>
```

Here we check if the lolipop graphite service is linked to the 'playground' app. 

```shell
dokku graphite:linked lolipop playground
```

### list all apps linked to the graphite service

```shell
# usage
dokku graphite:links <service>
```

List all apps linked to the 'lolipop' graphite service. 

```shell
dokku graphite:links lolipop
```

### Disabling `docker pull` calls

If you wish to disable the `docker pull` calls that the plugin triggers, you may set the `GRAPHITE_DISABLE_PULL` environment variable to `true`. Once disabled, you will need to pull the service image you wish to deploy as shown in the `stderr` output.

Please ensure the proper images are in place when `docker pull` is disabled.