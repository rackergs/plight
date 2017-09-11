﻿# Plight [![Build Status](https://travis-ci.org/rackerlabs/plight.svg?branch=master)](https://travis-ci.org/rackerlabs/plight)
An application agnostic tool to represent node availability.

## What is this nonsense?
In many of our deployment pipelines we had to gather credentials from any number of
systems, such as load balancers and monitoring, to disable nodes.  Each system might
have different requirements for authentication, vary quality of APIs or automation
libraries, etc. With stricter RBAC requirements we sometime ran into a system that
required admin access to do a simple state change.  Aside from the cumbersome
credentials management, access levels can be a problem in more compliance-oriented
environments.

Our original goals:
* Remove a node from being active in a load balancer pool
* Ensure monitoring knows the node is not active
* Determine if the node is done draining connections before continuing maintenance

Most of these external systems did have some means of internally managing the state,
whether via health checks or some other function.  Thus our initial approach was to
work with our developers to enable some kind of health state in their applications
that we could configure our systems to utilize.  However, this path leaves "off the
shelf" software without a path.

Once we tried to cover all of our use cases we decided the best route would be to
separate this functionality into a separate standalone web service.

### But what about that 3rd goal??
Early on we thought we could easily expose active connections on the host through
this service.  But really its a separate function, and only tangentially related.
If this is something you are interested we’d recommend checking out where we did
implement it, the [Ansible](http://www.ansible.com/) [wait_for module](http://docs.ansible.com/wait_for_module.html) as state=drained. A [standalone
implementation](https://github.com/gregswift/wait_for_drain) is available, but not really encouraged as it was more a proof of
concept.

## Why is it called Plight?
    a dangerous, difficult, or otherwise unfortunate situation

Originally it was called 'nodestatus', we took to the thesaurus. Based on where we
were with the original path to solve this problem we landed on plight.

## Installation

### Fedora or EL-based:
We have a [COPR](http://copr-fe.cloud.fedoraproject.org/coprs/xaeth/Plight) that is kept current with releases. After enabling that repository install using:
```yum install plight```

### From source
```make install```

### From puppet
* TODO: push our puppet module to be public and publish to forge

### From ansible
* [The role in Ansible Galaxy](https://galaxy.ansible.com/list#/roles/621)
* [The git repository for the role](https://github.com/gregswift/ansible-plight)

## Configuration

### States
By default Plight comes with 3 explicit states, which as of the 0.1.0 series are configurable in ```plight.conf```.

State   | Status Code | Message
--------|-------------|--------
Enabled | 200         | node is available
Disabled| 404         | node is unavailable
Offline | 503         | node is offline

Long term we are going to change the Status codes to default to 200 for these three
states. We maintained the states from the previous release for compatability
purposes.

### Enable the service
```
chkconfig plightd on
service plightd start
```

  or

```
systemctl enable plightd
systemctl start plightd
```

### Firewall
The default port configured for plight is ```10101```. In our examples directory there is a service entry for firewalld.

## Usage

### Changing states
Using ```plight --help``` from the cli will give you a list of all valid plight
commands, which includes start, stop, and a dynamically generated list based on
configured states.

#### Put a mode into maintenance mode
```plight disable```

#### Put a mode into offline mode
```plight offline```

#### Return a node to active mode
```plight enable```

#### List the configured states
```plight list-states```

#### Checking the current state of a node
```plight status```  
      or  
```curl http://localhost:10101 -D -```

## Licensing
All files contained with this distribution are licenced either under the [Apache License v2.0](http://www.apache.org/licenses/LICENSE-2.0) or the [GNU General Public License v2.0](http://www.gnu.org/licenses/gpl-2.0.html). You must agree to the terms of these licenses and abide by them before viewing, utilizing, modifying, or distributing the source code contained within this distribution.

## Build Notes

### Tests
```make test```

### Manual
* Install via makefile
```sudo make install```

### Fedora/EL
* Generate the RPM
```make``` or ```make rpms```

#### EL5
* Requires buildsys-macros installed

#### COPR (publishing RPMs)
COPR: https://copr.fedoraproject.org/coprs/xaeth/Plight/

* Generate SRPM
```make srpm```
* Publish SRPM to a publicly available HTTP or FTP repo
* Load the build into COPR
```copr-cli build Plight http://example.com/paht/to/plight.src.rpm```

### Debian
* Generate deb package
``` make debs```

#### PPA (publishing DEBs)
* Generate source package bits
```make debsrc```
* Change to ./artifacts/debs/ and generate signed source with changes
```
cd artifacts/debs
debbuild -S -sa
```
* Push to PPA
```dput ppa:gregswift/plight plight_VERSION_source.changes```
