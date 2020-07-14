## Docker

This directory contains Docker-based flow to run Teleport Plugins locally, and
is indended for manual QA with the [Test Plan](../testplan.md) / testing
purposuses.

### Setup

This flow builds on top of
[Teleport's own Docker flow](https://github.com/gravitational/teleport/tree/master/docker).
Teleport's own Docker image and services are managed with that flow.

#### Overview

The setup process builds up several Docker images, and then helps you configure
the Teleport cluster on those images and some plugins to run together.

Docker images are responsible for the software (i.e. have the correct version of
Teleport Enterprise to run the plugins), but the further confuguration, like
what specific certificates and addresses to use for different services is
performed _after Dockerfiles are built_:

- `Dockerfile` is generally responsible for the software running in the
  container.
- `docker-compose.yml` is responsible for baseline configuration for the cluster
  to work together.
- `make config-*` subcommands will help you setup specific configs for specific
  plugins, and they should work even if the config format changes in the fugure.

#### Getting started with Teleport's Docker flow

First prepare your teleport directory to work with the Docker flow. The flow
assumes that you have `teleport` alongside `teleport-plugins`, and they have the
same parent directory.

#### Building `teleport:latest`

You'll need Teleport's `teleport:latest` Docker image to run plugins. This
command will build it:

```bash
# In the parent directory of teleport-plugins
git clone git@gitnub.com:gravitational/teleport.git
cd teleport
make docker
```

If the command above fails, try building the buildbox fist:

```bash
cd ../build.assts
make buildbox
```

This will build the buildbox container for you, and tag it for quay. If you
don't have access to that, you can run the command that make runs yourself, but
with `-t teleport-builbox:go-{GOVERSION}`, like that:

```bash
# In the teleport/build.assets dir:
docker build \
	--build-arg UID=(id -u) \
	--build-arg GID=(id -g) \
	--build-arg RUNTIME=go1.13.2 \
	--cache-from quay.io/gravitational/teleport-buildbox:go1.13.2 \
	-t teleport-buildbox:go1.13.2
	.
```

#### Building `teleport-ent:latest`

Teleport Plugins require Enterprise version of Teleport to run correctly, and
`teleport:latest` won't have it by default, so you'll need to build a special
Docker image running the enterprise version. This flow calls this immage
`teleport-ent:latest`, and it's build like this:

```shell
# In your main teleport-plugins/docker directory
make build-teleport-image
```

_*Note*: you can pass a `-e RELEASE=binary-teleport-ent-name-to-download` to
docker build command if you want to — that would install a specified Teleport
Enterprise version to the container. All the build does, actually, is it takes
the OSS built `teleport:latest`, downloads the Teleport Enterprise edition, and
installs it._

_*Note*: this setup requires you to bring your own Teleport Enterprise License
and put it to `data/var/lib/teleport/license.pam`. Enterprise features, and
hence the whole flow, might not work otherwise._

After that, we'll build the teleport's image from teleport-plugin's directory.
We'll still use teleport's own Dockerfile, but the services in
`docker-compose.yml` are different, and teleport-plugins test flow uses it's own
`data` directory, just so if you've been testing both teleport and
teleport-plugins, they shouldn't interfere with each other.

Please refer to Teleport's Docker documentation for more details about it's
configuration.

#### Plugin(s) Docker images

Each plugin will build it's own Docker image internally, but those will be built
automatically with `docker-compose` via a makefile command, described below.

#### Configuration

For the plugins to work, each plugin needs:

- A valid configuration file.
- A user, role, and a valid set of ceriticates to authenticate itself with
  Teleport server.
- A set of TLS certificates if the plugin has any web hook servers in it.

### Starting

```bash
make up # Starts a single node Teleport cluster and a Teleport Slack plugin alongside it.
```

### Stopping

```bash
make down
```

### TODO

Todo items for this document:

1. [x] Read the test plan thorougly
2. [x] Describe how to launch test scenarios (which teleport instances and
       plugin instances will we need?)
3. [x] Start implementing the instances in `docker-compose` and `Makefile`.
4. [ ] Implement Slack Plugin Dockerfile + service, write about how to set up
       it's config
5. [ ] Ben reviews if the Slack part is valid
