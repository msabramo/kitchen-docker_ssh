# Kitchen::Docker 

[![CircleCI Build Status](https://circleci.com/gh/peterabbott/kitchen-docker_ssh.png?style=shield&circle-token=bad3411099b80b42c28842fb59a74ee546eee2b8)](https://circleci.com/gh/peterabbott/kitchen-docker_ssh)


[![Travis Build Status](https://travis-ci.org/peterabbott/kitchen-docker_ssh.svg)](https://travis-ci.org/peterabbott/kitchen-docker_ssh)


A Test Kitchen Driver for Docker. This is a fork of http://github.com/portertech/kitchen-docker because the original driver is only sparodically updated.



## Requirements

* [Docker][docker_getting_started]



## Installation and Setup

### Gemfile

Add an entry to your Gemfile. 

```
gem 'kitchen-docker_ssh'
```


To point to the latest development version, you can also point to the git repo
```
gem 'kitchen-docker_ssh', :git => 'https://github.com/peterabbott/kitchen-docker_ssh.git'
```


### Test Kitchen


Please read the Test Kitchen [docs][test_kitchen_docs] for more details.

Example `.kitchen.local.yml`:

```
---
driver:
  name: docker

platforms:
- name: ubuntu
  run_list:
  - recipe[apt]
- name: centos
  driver_config:
    image: centos
    platform: rhel
  run_list:
  - recipe[yum]
```

## Default Configuration

This driver can determine an image and platform type for a select number of
platforms.

Examples:

```
---
platforms:
- name: ubuntu-12.04
- name: centos-6.4
```

This will effectively generate a configuration similar to:

```
---
platforms:
- name: ubuntu-12.04
  driver_config:
    image: ubuntu:12.04
    platform: ubuntu
- name: centos-6.4
  driver_config:
    image: centos:6.4
    platform: centos
```

## Configuration

### binary

The Docker binary to use.

The default value is `docker`.

Examples:

```
  binary: docker.io
```

```
  binary: /opt/docker
```

Windows, requires docker cli to be on the path, otherwise use absolute path: 

```
   binary: docker.exe 
```


### socket

The Docker daemon socket to use. By default, Docker will listen on
`unix:///var/run/docker.sock`, and no configuration here is required. If
Docker is binding to another host/port or Unix socket, you will need to set
this option. If a TCP socket is set, its host will be used for SSH access
to suite containers.

Examples:

```
  socket: unix:///tmp/docker.sock
```

```
  socket: tcp://docker.example.com:4242
```

If you use [Boot2Docker](https://github.com/boot2docker/boot2docker), set your `DOCKER_HOST` environment variable properly (e.g. `export DOCKER_HOST=tcp://192.168.59.103:2375`) or you have to use the following:

```
socket: tcp://192.168.59.103:2375
```

### use_sudo

Flags whether commands should be run as sudo, default true.

```
  use_sudo: false
```

When a remote socket used then use_sudo is set to false.

### image

The Docker image to use as the base for the suite containers. You can find
images using the [Docker Index][docker_index].

The default will be computed, using the platform name (see the Default
Configuration section for more details).

### platform

The platform of the chosen image. This is used to properly bootstrap the
suite container for Test Kitchen. Kitchen Docker currently supports:

* `debian` or `ubuntu`
* `rhel` or `centos`
* `gentoo` or `gentoo-paludis`

The default will be computed, using the platform name (see the Default
Configuration section for more details).

### require\_chef\_omnibus

Determines whether or not a Chef [Omnibus package][chef_omnibus_dl] will be
installed. There are several different behaviors available:

* `true` - the latest release will be installed. Subsequent converges
  will skip re-installing if chef is present.
* `latest` - the latest release will be installed. Subsequent converges
  will always re-install even if chef is present.
* `<VERSION_STRING>` (ex: `10.24.0`) - the desired version string will
  be passed the the install.sh script. Subsequent converges will skip if
  the installed version and the desired version match.
* `false` or `nil` - no chef is installed.

The default value is `true`.

### disable\_upstart

Disables upstart on Debian/Ubuntu containers, as many images do not support a
working upstart.

The default value is `true`.

### provision\_command

Custom command(s) to be run when provisioning the base for the suite containers.

Examples:

```
  provision_command: curl -L https://www.opscode.com/chef/install.sh | bash
```

```
  provision_command:
    - apt-get install dnsutils
    - apt-get install telnet
```

```
driver_config:
  provision_command: curl -L https://www.opscode.com/chef/install.sh | bash
  require_chef_omnibus: false
```

### use\_cache

This determines if the Docker cache is used when provisioning the base for suite
containers.

The default value is `true`.

### use\_sudo

This determines if Docker commands are run with `sudo`.

The default value depends on the type of socket being used. For local sockets, the default value is `true`. For remote sockets, the default value is `false`.

This should be set to `false` if you're using boot2docker, as every command passed into the VM runs as root by default.

### remove\_images

This determines if images are automatically removed when the suite container is
destroyed.

The default value is `false`.

### run_command

Sets the command used to run the suite container.

The default value is `/usr/sbin/sshd -D -o UseDNS=no -o UsePAM=no -o PasswordAuthentication=yes`.

Examples:

```
  run_command: /sbin/init
```

### memory

Sets the memory limit for the suite container in bytes. Otherwise use Dockers
default. You can read more about `memory.limit_in_bytes` [here][memory_limit].

### cpu

Sets the CPU shares (relative weight) for the suite container. Otherwise use
Dockers defaults. You can read more about cpu.shares [here][cpu_shares].

### cpuset

Sets the CPU affinities (i.e. the CPUs on which to allow execution) for the
suite container. Otherwise use Dockers defaults. You can read more in the
[docker-run man page][docker_man].

Examples:

```
  cpuset: 0-3
```

```
  cpuset: '0,1'
```
Notice that when using commas in the `cpuset` value you **must** quote them as
a string.

**Note:** This feature is only available in docker versions >= 1.1.0.

### volume

Adds a data volume(s) to the suite container.

Examples:

```
  volume: /ftp
```

```
  volume:
  - /ftp
  - /srv
```

### volumes_from

Mount volumes managed by other containers.

Examples:

```
  volumes_from: repos
```

```
  volumes_from:
  - repos
  - logging
  - rvm
```

### dns

Adjusts `resolv.conf` to use the dns servers specified. Otherwise use
Dockers defaults.

Examples:

```
  dns: 8.8.8.8
```

```
  dns:
  - 8.8.8.8
  - 8.8.4.4
```
### http\_proxy

Sets an http proxy for the suite container using the `http_proxy` environment variable.

Examples:

```
  http_proxy: http://proxy.host.com:8080
```
### https\_proxy

Sets an https proxy for the suite container using the `https_proxy` environment variable.

Examples:

```
  https_proxy: http://proxy.host.com:8080
```
### forward

Set suite container port(s) to forward to the host machine. You may specify
the host (public) port in the mappings, if not, Docker chooses for you.

Examples:

```
  forward: 80
```

```
  forward:
  - 22:2222
  - 80:8080
```

### hostname

Set the suite container hostname. Otherwise use Dockers default.

Examples:

```
  hostname: foobar.local
```

### privileged

Run the suite container in privileged mode. This allows certain functionality
inside the Docker container which is not otherwise permitted.

The default value is `false`.

Examples:

```
  privileged: true
```

## dockerfile

Use a custom Dockerfile, instead of having Kitchen-Docker build one for you.

Examples:

```
  dockerfile: test/Dockerfile
```

### instance_name

Set the name of container to link to other container(s).

Examples:

```
  instance_name: web
```

### links

Set ```instance_name```(and alias) of other container(s) that connect from the suite container.

Examples:

```
 links: db:db
```

Examples:

```
  links:
  - db:db
  - kvs:kvs
```

### publish_all

Publish all exposed ports to the host interfaces.
This option used to communicate between some containers.

The default value is `false`.

Examples:

```
  publish_all: true
```

### cap_add
Adds a capability to the running container, for example SYS_PTRACE

````
cap_add: 
- SYS_PTRACE

````

### cap_drop
Drops a capability to the running container, for example CHOWN

````
cap_drop: 
- CHOWN

````


## Development

* Source hosted at [GitHub][repo]
* Report issues/questions/feature requests on [GitHub Issues][issues]

Pull requests are very welcome! Make sure your patches are well tested.
Ideally create a topic branch for every separate change you make. For
example:

1. Fork the repo
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Added some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

## Authors

Created and maintained by 

 - [Peter Abbott](peter@piemanpete.com)
 
Based on original work from:
 - [Sean Porter][author] (<portertech@gmail.com>) http://github.com/portertech/kitchen-docker/


## License

Apache 2.0 (see [LICENSE][license])



[author]:           https://github.com/peterabbott
[issues]:           https://github.com/peterabbott/kitchen-docker_ssh/issues
[license]:          https://github.com/peterabbott/kitchen-docker_ssh/blob/master/LICENSE
[repo]:             https://github.com/peterabbott/kitchen-docker_ssh
[driver_usage]:     http://docs.kitchen-ci.org/drivers/usage
[chef_omnibus_dl]:  http://www.getchef.com/chef/install/
