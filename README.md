# dokku graphite [![Build Status](https://img.shields.io/github/actions/workflow/status/dokku/dokku-graphite/ci.yml?branch=master&style=flat-square "Build Status")](https://github.com/dokku/dokku-graphite/actions/workflows/ci.yml?query=branch%3Amaster) [![IRC Network](https://img.shields.io/badge/irc-libera-blue.svg?style=flat-square "IRC Libera")](https://webchat.libera.chat/?channels=dokku)

Official graphite plugin for dokku. Currently defaults to installing [dokku/docker-grafana-graphite 6.4.4](https://hub.docker.com/r/dokku/docker-grafana-graphite/).

## Requirements

- dokku 0.35.x+
- docker 1.8.x

## Installation

```shell
# on 0.35.x+
sudo dokku plugin:install https://github.com/dokku/dokku-graphite.git --name graphite
```

## Commands

```
graphite:app-links [<app>]                         # list all Graphite service links for a given app
graphite:create <service> [--create-flags...]      # create a Graphite service
graphite:destroy <service> [-f|--force]            # delete the Graphite service/data/container if there are no links left
graphite:enter <service>                           # enter or run a command in a running Graphite service container
graphite:exists <service>                          # check if the Graphite service exists
graphite:expose <service> <ports...>               # expose a Graphite service on custom host:port if provided (random port on the 0.0.0.0 interface if otherwise unspecified)
graphite:info [<service>] [--info-flags...]        # print the service information
graphite:link <service> [<app>] [--link-flags...]  # link the Graphite service to the app
graphite:linked <service> [<app>]                  # check if the Graphite service is linked to an app
graphite:links <service>                           # list all apps linked to the Graphite service
graphite:list                                      # list all Graphite services
graphite:logs <service> [-t|--tail [<tail-num>]]   # print the most recent log(s) for this service
graphite:mount [--replace] <service> <source:container-dir[:options]>... # mount a host path or docker volume into the service container
graphite:nginx-expose <service> [domain]           # expose the Graphite service's grafana via an nginx vhost
graphite:nginx-unexpose <service>                  # unexpose the Graphite service's grafana
graphite:pause <service>                           # pause a running Graphite service
graphite:promote <service> [<app>]                 # promote service <service> as STATSD_URL in <app>
graphite:restart <service>                         # graceful shutdown and restart of the Graphite service container
graphite:set <service> <key> <value>               # set or clear a property for a service
graphite:start <service>                           # start a previously stopped Graphite service
graphite:stop <service>                            # stop a running Graphite service
graphite:unexpose <service>                        # unexpose a previously exposed Graphite service
graphite:unlink <service> [<app>] [-n|--no-restart] # unlink the Graphite service from the app
graphite:unmount [--all] <service> [<source:container-dir>...] # remove one or all mounts from the service container
graphite:upgrade <service> [--upgrade-flags...]    # upgrade service <service> to the specified versions
```

## Usage

Help for any commands can be displayed by specifying the command as an argument to graphite:help. Plugin help output in conjunction with any files in the `docs/` folder is used to generate the plugin documentation. Please consult the `graphite:help` command for any undocumented commands.

### Basic Usage

### create a Graphite service

```shell
# usage
dokku graphite:create <service> [--create-flags...]
```

flags:

- `-c|--config-options <string>`: extra arguments for the process the service container runs, not docker flags; use mount for mounts
- `-C|--custom-env <string>`: semi-colon delimited environment variables to start the service with
- `-i|--image <string>`: the image name to start the service with
- `-I|--image-version <string>`: the image version to start the service with
- `-N|--initial-network <string>`: the initial network to attach the service to
- `--log-driver <string>`: the docker logging driver to run the service container with (default: the daemon's own)
- `--log-opt <strings>`: a comma-separated list of key=value docker log options for the service container
- `-m|--memory <int>`: container memory limit in megabytes (default: unlimited)
- `-p|--password <string>`: override the user-level service password
- `-P|--post-create-network <strings>`: a comma-separated list of networks to attach the service container to after service creation
- `-S|--post-start-network <strings>`: a comma-separated list of networks to attach the service container to after service start
- `--restart <string>`: the docker restart policy to run the service container with (default: always)
- `-r|--root-password <string>`: override the root-level service password
- `-s|--shm-size <string>`: override shared memory size for the service docker container
- `--volume <stringArray>`: a host path or docker volume to mount into the service container, as <source>:<container-dir>[:<options>], repeatable

Create a graphite service named lollipop:

```shell
dokku graphite:create lollipop
```

You can also specify the image and image version to use for the service. It *must* be compatible with the dokku/docker-grafana-graphite image.

```shell
export STATSD_IMAGE="dokku/docker-grafana-graphite"
export STATSD_IMAGE_VERSION="6.4.4"
dokku graphite:create lollipop
```

An image other than dokku/docker-grafana-graphite has no version to fall back on, because the version this plugin pins belongs to dokku/docker-grafana-graphite, so name one alongside it.

```shell
dokku graphite:create lollipop --image <image> --image-version <version>
```

You can also specify custom environment variables to start the graphite service in semicolon-separated form.

```shell
export STATSD_CUSTOM_ENV="USER=alpha;HOST=beta"
dokku graphite:create lollipop
```

The container log is bounded by whatever `dokku logs:set --global max-size` says, and by dokku's own default where it says nothing, which a service may override for itself.

```shell
dokku graphite:create lollipop --log-opt max-size=20m,max-file=3
```

The container is restarted by docker whenever it stops, which a service may change for itself.

```shell
dokku graphite:create lollipop --restart unless-stopped
```

The config options are handed to the process the container runs, not to docker, so a host path or docker volume is mounted with --volume, which may be repeated.

```shell
dokku graphite:create lollipop --volume /var/lib/dokku/data/storage/lollipop:/opt/extra:ro
```

### delete the Graphite service/data/container if there are no links left

```shell
# usage
dokku graphite:destroy <service> [-f|--force]
```

flags:

- `-f|--force`: force the destruction of the service

Destroy the service, it's data, and the running container:

```shell
dokku graphite:destroy lollipop
```

A service that is still linked to an app is not destroyed, and the apps it is linked to are named. Unlink them first.

### print the service information

```shell
# usage
dokku graphite:info [<service>] [--info-flags...]
```

flags:

- `--backend`: show the execution backend the service was created with
- `--backup-authenticated`: show whether backup credentials are stored for the service
- `--backup-bucket`: show the bucket scheduled backups are shipped to
- `--backup-encrypted`: show whether scheduled backups are encrypted with a passphrase
- `--backup-keyserver`: show the keyserver backup public keys are fetched from
- `--backup-public-key-id`: show the gpg public key id backups are encrypted with
- `--backup-schedule`: show the cron schedule backups run on
- `--backup-use-iam`: show whether scheduled backups authenticate with an instance role
- `--config-dir`: show the service configuration directory
- `--config-options`: show the config options the service container is run with
- `--custom-env`: show the custom environment the service container is run with
- `--data-dir`: show the service data directory
- `--database-name`: show the name of the database inside the service
- `--definition`: show the definition the service was created with
- `--dsn`: show the service DSN
- `--exposed-ports`: show service exposed ports
- `--id`: show the service container id
- `--image`: show the image the service runs
- `--image-version`: show the image version the service was created with
- `--initial-network`: show the initial network being connected to
- `--internal-ip`: show the service internal ip
- `--links`: show the service app links
- `--log-driver`: show the docker logging driver the service container is run with
- `--log-opt`: show the docker log options the service container is run with
- `--memory`: show the memory limit the service container is run with
- `--mounts`: show the host paths and docker volumes mounted into the service container
- `--post-create-network`: show the networks to attach to after service container creation
- `--post-start-network`: show the networks to attach to after service container start
- `--restart-policy`: show the restart policy the service container is run with
- `--service`: show the name of the service
- `--service-root`: show the service root directory
- `--shm-size`: show the shared memory size the service container is run with
- `--status`: show the service running status
- `--version`: show the service image version

Get connection information as follows:

```shell
dokku graphite:info lollipop
```

Alongside the connection information this reports the properties set on the service, the state it was created with, and its backup settings. A property that was never set, or that was unset, reports as empty. Omit the service to report on every graphite service:

```shell
dokku graphite:info
```

The information can be read by machine, one json object per service:

```shell
dokku graphite:info lollipop --format json
```

You can also retrieve a specific piece of service info via a flag, which prints it on its own:

```shell
dokku graphite:info lollipop --dsn
dokku graphite:info lollipop --status
dokku graphite:info lollipop --initial-network
```

> NOTE: a flag cannot be combined with --format, and only one may be given

The properties graphite:set writes are reported under the names it takes, so a value read here can be written back:

```shell
dokku graphite:set lollipop initial-network my-network
```

### list all Graphite services

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
dokku graphite:logs <service> [-t|--tail [<tail-num>]]
```

flags:

- `-t|--tail <int>`: tail the logs, optionally showing this many lines

You can tail logs for a particular service:

```shell
dokku graphite:logs lollipop
```

By default, logs will not be tailed, but you can do this with the --tail flag:

```shell
dokku graphite:logs lollipop --tail
```

By default the last 100 lines are shown, but a different count can be specified:

```shell
dokku graphite:logs lollipop --tail=5
```

### link the Graphite service to the app

```shell
# usage
dokku graphite:link <service> [<app>] [--link-flags...]
```

flags:

- `-a|--alias <string>`: an alternative alias to use for the config url exported to the app
- `-n|--no-restart`: whether to skip restarting the app
- `-q|--querystring <string>`: ampersand delimited querystring arguments to append to the service url

A graphite service can be linked to a container. This will use native docker links via the docker-options plugin. Here we link it to our `playground` app.

> NOTE: this will restart your app

```shell
dokku graphite:link lollipop playground
```

The following environment variables will be set automatically by docker (not on the app itself, so they won’t be listed when calling dokku config):

```
DOKKU_STATSD_LOLLIPOP_NAME=/lollipop/DATABASE
DOKKU_STATSD_LOLLIPOP_PORT=tcp://172.17.0.1:8125
DOKKU_STATSD_LOLLIPOP_PORT_8125_TCP=tcp://172.17.0.1:8125
DOKKU_STATSD_LOLLIPOP_PORT_8125_TCP_PROTO=tcp
DOKKU_STATSD_LOLLIPOP_PORT_8125_TCP_PORT=8125
DOKKU_STATSD_LOLLIPOP_PORT_8125_TCP_ADDR=172.17.0.1
```

The following will be set on the linked application by default:

```
STATSD_URL=statsd://:SOME_PASSWORD@dokku-graphite-lollipop:8125
```

The host exposed here only works internally in docker containers. If you want your container to be reachable from outside, you should use the `expose` subcommand. Another service can be linked to your app:

```shell
dokku graphite:link other_service playground
```

It is possible to change the protocol for `STATSD_URL` by setting the environment variable `STATSD_DATABASE_SCHEME` on the app. Doing so after linking means unlink no longer finds the variable it set, and leaves it in place, so we advise you to unlink before proceeding.

```shell
dokku config:set playground STATSD_DATABASE_SCHEME=statsd2
dokku graphite:link lollipop playground
```

This will cause `STATSD_URL` to be set as:

```
statsd2://:SOME_PASSWORD@dokku-graphite-lollipop:8125
```

### unlink the Graphite service from the app

```shell
# usage
dokku graphite:unlink <service> [<app>] [-n|--no-restart]
```

flags:

- `-n|--no-restart`: whether to skip restarting the app

You can unlink a graphite service:

> NOTE: this will restart your app and unset related environment variables

```shell
dokku graphite:unlink lollipop playground
```

An app is still linked after its `STATSD_URL` is changed to point elsewhere, and is unlinked the same way. The variable it now holds is not the service's, so it is left alone, nothing is unset, the app is not restarted, and a warning says so.

### set or clear a property for a service

```shell
# usage
dokku graphite:set <service> <key> <value>
```

Set the network to attach after the service container is started:

```shell
dokku graphite:set lollipop post-create-network custom-network
```

Set multiple networks:

```shell
dokku graphite:set lollipop post-create-network custom-network,other-network
```

Unset the post-create-network value:

```shell
dokku graphite:set lollipop post-create-network
```

Set the keyserver a public key for backup encryption is fetched from:

```shell
dokku graphite:set lollipop backup-keyserver hkp://keys.example.com
```

Cap the container log at a size of your own rather than the one it inherits:

```shell
dokku graphite:set lollipop log-opt max-size=20m,max-file=3
```

Keep the log unbounded, which is what a service had before there was anything to say here:

```shell
dokku graphite:set lollipop log-opt max-size=unlimited
```

Send the container log somewhere other than the daemon's own driver:

```shell
dokku graphite:set lollipop log-driver journald
```

Restart the container unless it was stopped on purpose, including across a docker restart:

```shell
dokku graphite:set lollipop restart-policy unless-stopped
```

Go back to always restarting the container:

```shell
dokku graphite:set lollipop restart-policy
```

> NOTE: a log setting or a restart policy reaches the container the next time one is built. graphite:restart keeps the container it has, so use graphite:stop and then graphite:start on a service that is already running.

### mount a host path or docker volume into the service container

```shell
# usage
dokku graphite:mount [--replace] <service> <source:container-dir[:options]>...
```

flags:

- `--replace`: replace the service's entire set of mounts with the ones given
- `--volume-chown <string>`: a chown option, recorded but not applied; not valid with --replace
- `--volume-options <string>`: comma-separated docker mount options, such as z or nocopy; not valid with --replace
- `--volume-readonly`: mount the volume read only; not valid with --replace
- `--volume-subpath <string>`: a subpath within the source, recorded but not applied; not valid with --replace

Mount a host directory into the service container:

```shell
dokku graphite:mount lollipop /var/lib/dokku/data/storage/lollipop:/opt/extra
```

The source is an absolute host path, which must already exist, or the name of a docker volume. Options follow a second colon: ro or rw, docker's own mount options, and volume-subpath=<path> and volume-chown=<option>, which are recorded but not applied:

```shell
dokku graphite:mount lollipop /var/lib/dokku/data/storage/lollipop:/opt/extra:ro,z
```

The same can be said with flags instead:

```shell
dokku graphite:mount lollipop /var/lib/dokku/data/storage/lollipop:/opt/extra --volume-readonly --volume-options z
```

Mounting the same source at the same directory again rewrites its options:

```shell
dokku graphite:mount lollipop /var/lib/dokku/data/storage/lollipop:/opt/extra
```

Replace every mount the service has with the ones given:

```shell
dokku graphite:mount --replace lollipop /srv/a:/opt/a:ro /srv/b:/opt/b
```

> NOTE: a mount reaches the container the next time one is built. graphite:restart keeps the container it has, so use graphite:stop and then graphite:start on a service that is already running.

### remove one or all mounts from the service container

```shell
# usage
dokku graphite:unmount [--all] <service> [<source:container-dir>...]
```

flags:

- `--all`: remove every mount the service has

Remove a mount, naming it the way it was mounted:

```shell
dokku graphite:unmount lollipop /var/lib/dokku/data/storage/lollipop:/opt/extra
```

Remove every mount the service has:

```shell
dokku graphite:unmount --all lollipop
```

> NOTE: the mount is removed from the container the next time one is built. graphite:restart keeps the container it has, so use graphite:stop and then graphite:start on a service that is already running.

### Service Lifecycle

The lifecycle of each service can be managed through the following commands:

### enter or run a command in a running Graphite service container

```shell
# usage
dokku graphite:enter <service>
```

A bash prompt can be opened against a running service. Filesystem changes will not be saved to disk.

> NOTE: disconnecting from ssh while running this command may leave zombie processes due to moby/moby#9098

```shell
dokku graphite:enter lollipop
```

You may also run a command directly against the service. Filesystem changes will not be saved to disk.

```shell
dokku graphite:enter lollipop touch /tmp/test
```

### expose a Graphite service on custom host:port if provided (random port on the 0.0.0.0 interface if otherwise unspecified)

```shell
# usage
dokku graphite:expose <service> <ports...>
```

Expose the service on the service's normal ports, allowing access to it from the public interface (`0.0.0.0`):

```shell
dokku graphite:expose lollipop 8125 8126 80 81 2003
```

Expose the service on the service's normal ports, with the first on a specified ip address (127.0.0.1):

```shell
dokku graphite:expose lollipop 127.0.0.1:8125 8126 80 81 2003
```

### unexpose a previously exposed Graphite service

```shell
# usage
dokku graphite:unexpose <service>
```

Unexpose the service, removing access to it from the public interface (`0.0.0.0`):

```shell
dokku graphite:unexpose lollipop
```

### promote service <service> as STATSD_URL in <app>

```shell
# usage
dokku graphite:promote <service> [<app>]
```

If you have a graphite service linked to an app and try to link another graphite service another link environment variable will be generated automatically:

```
DOKKU_STATSD_BLUE_URL=statsd://:ANOTHER_PASSWORD@dokku-graphite-other-service:8125/other_service
```

You can promote the new service to be the primary one:

> NOTE: this will restart your app

```shell
dokku graphite:promote other_service playground
```

This will replace `STATSD_URL` with the url from other_service and generate another environment variable to hold the previous value if necessary. You could end up with the following for example:

```
STATSD_URL=statsd://:ANOTHER_PASSWORD@dokku-graphite-other-service:8125/other_service
DOKKU_STATSD_BLUE_URL=statsd://:ANOTHER_PASSWORD@dokku-graphite-other-service:8125/other_service
DOKKU_STATSD_SILVER_URL=statsd://:SOME_PASSWORD@dokku-graphite-lollipop:8125/lollipop
```

### start a previously stopped Graphite service

```shell
# usage
dokku graphite:start <service>
```

Start the service:

```shell
dokku graphite:start lollipop
```

A service comes back on the version it was created with, or was last upgraded to, whatever version the plugin ships now. The image is fetched if the host no longer has it. A service that has never recorded a version and has no container left to read one from cannot be placed, and is reported rather than started on a guess. Use graphite:upgrade to say which version it should run.

### stop a running Graphite service

```shell
# usage
dokku graphite:stop <service>
```

Stop the service and removes the running container:

```shell
dokku graphite:stop lollipop
```

### pause a running Graphite service

```shell
# usage
dokku graphite:pause <service>
```

Pause the running container for the service:

```shell
dokku graphite:pause lollipop
```

### graceful shutdown and restart of the Graphite service container

```shell
# usage
dokku graphite:restart <service>
```

Restart the service:

```shell
dokku graphite:restart lollipop
```

### upgrade service <service> to the specified versions

```shell
# usage
dokku graphite:upgrade <service> [--upgrade-flags...]
```

flags:

- `-c|--config-options <string>`: extra arguments for the process the service container runs, not docker flags; use mount for mounts
- `-C|--custom-env <string>`: semi-colon delimited environment variables to start the service with
- `-i|--image <string>`: the image to upgrade the service to
- `-I|--image-version <string>`: the image version to upgrade the service to
- `-N|--initial-network <string>`: the initial network to attach the service to
- `--log-driver <string>`: the docker logging driver to run the service container with (default: the daemon's own)
- `--log-opt <strings>`: a comma-separated list of key=value docker log options for the service container
- `-P|--post-create-network <strings>`: a comma-separated list of networks to attach the service container to after service creation
- `-S|--post-start-network <strings>`: a comma-separated list of networks to attach the service container to after service start
- `--restart <string>`: the docker restart policy to run the service container with (default: always)
- `-R|--restart-apps`: whether to stop and start the linked apps around the upgrade
- `-s|--shm-size <string>`: override shared memory size for the service docker container
- `--volume <stringArray>`: a host path or docker volume to mount into the service container, as <source>:<container-dir>[:<options>], repeatable

You can upgrade an existing service to a new image or image-version:

```shell
dokku graphite:upgrade lollipop
```

This is the only command that changes the version a service runs. With no version named it moves to the newest the service's own major version ships, which leaves the data where it is.

```shell
dokku graphite:upgrade lollipop --image-version 1.2.3
```

Moving across a major version has to be asked for by name, because it is not a tag change: the data is mounted somewhere different under the new one, and pointing the version back does not undo it. A service keeps the mounts it has unless --volume is passed, which replaces them, and each one is checked against the new container before the old one is taken away.

```shell
dokku graphite:upgrade lollipop --volume /var/lib/dokku/data/storage/lollipop:/opt/extra:ro
```

### Service Automation

Service scripting can be executed using the following commands:

### list all Graphite service links for a given app

```shell
# usage
dokku graphite:app-links [<app>]
```

List all graphite services that are linked to the `playground` app.

```shell
dokku graphite:app-links playground
```

### check if the Graphite service exists

```shell
# usage
dokku graphite:exists <service>
```

Here we check if the lollipop graphite service exists.

```shell
dokku graphite:exists lollipop
```

### check if the Graphite service is linked to an app

```shell
# usage
dokku graphite:linked <service> [<app>]
```

Here we check if the lollipop graphite service is linked to the `playground` app.

```shell
dokku graphite:linked lollipop playground
```

### list all apps linked to the Graphite service

```shell
# usage
dokku graphite:links <service>
```

List all apps linked to the `lollipop` graphite service.

```shell
dokku graphite:links lollipop
```

Renaming an app moves its link onto the new name, and cloning an app links the clone as well as the original.

### Custom Commands

This datastore adds the following commands of its own:

### expose the Graphite service's grafana via an nginx vhost

```shell
# usage
dokku graphite:nginx-expose <service> [domain]
```

Expose the Graphite service's grafana via an nginx vhost:

> NOTE: with no domain, grafana answers to grafana-<service>.<vhost> for every global vhost

```shell
dokku graphite:nginx-expose lollipop
dokku graphite:nginx-expose lollipop example.com
```

### unexpose the Graphite service's grafana

```shell
# usage
dokku graphite:nginx-unexpose <service>
```

Unexpose the Graphite service's grafana:

```shell
dokku graphite:nginx-unexpose lollipop
```

### Disabling `docker image pull` calls

If you wish to disable the `docker image pull` calls that the plugin triggers, you may set the `GRAPHITE_DISABLE_PULL` environment variable to `true`. Once disabled, you will need to pull the service image you wish to deploy as shown in the `stderr` output.

Please ensure the proper images are in place when `docker image pull` is disabled.
