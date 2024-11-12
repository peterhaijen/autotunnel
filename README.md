# autotunnel

Creates and maintains ssh local and remote SSH tunnels.

# Description

Autotunnel manages SSH tunnels, and more specifically, local and remote tunnels
to and from remote hosts.

It will connect to multiple SSH servers, and can establish multiple SSH
tunnels with each connected SSH client.

In case an SSH client stops running, autotunnel will automatically restart it, thus
ensuring that the SSH tunnels are always available.

# Usage

```
Usage:
    autotunnel [OPTION]...

Options and Arguments:
      -c,--configfile     The main configuration file - defaults to /etc/autotunnel.conf

         --debug          Debug output
         --help           Print this info
         --verbose        Verbose output
         --version        Show version
```

# Installation

Install the following dependencies:

```
sudo apt install libappconfig-std-perl libconfig-inifiles-perl
```

# Configuration

The main configuration file is `/etc/autotunnel.conf`.

## Default settings

Settings provided in the `default` section are used as
defaults in all other sections.


```
# This section provides defaults
[default]

# The default target for the SSH connection
host = user@myserver.example.com

The username is optional. In fact, the host specified
can be configured in the local SSH config file, avoiding the
need to specify the username and the fully qualified host:
# host = server

# Drop root privileges to the specified local account to
# run the SSH client and establish the tunnel:
runas = elvis

# Load additional configuration files. Specify a file,
# or a pattern like this:
load = /etc/autotunnel.conf.d/*.conf

# Specify SSH options here:
option = -NT
option = -o ServerAliveInterval=30
option = -o ServerAliveCountMax=2
```

# Sections

A section is a collection of settings, and should specify 1 or more
local or remote tunnels. Each section will inherit the default settings, and
can override the default settings.

Each section will start at most 1 SSH client. All local and remote tunnels
specified in the same section will be handled by the same SSH connection.

This is an example of a section that does a remote forward (a tunnel from
the remote host to a local host) to allow access to a local HTTP server by
anyone connecting to the remote host. No `host` is specified, so the default
specified in the `default` section will be used.

```
[web]
remote = 8080:localhost:80
```

The following example does the same thing, but in this case, the local HTTP
server is not running on the local machine, but is running on a different host
in the same LAN as the local machine.

```
[web]
remote = 8080:192.168.0.5:80
```

An example section that creates a local forward tunnel to connect to a remote SMTP
server. Connecting to localhost:1025 will establish a connection to the remote
SMTP server's port 26.

```
[smtp]
local = 1025:localhost:26
```

# Starting

Autotunnel should run as a service from systemd. Although not absolutely necessary, it expects
to run as root, thus allowing to drop root privileges as necessary to allow each SSH tunnel
to run in the context of a non-priviledged account.

## Configuring systemd

This is an example systemd service file. Copy it to `/etc/systemd/system/autotunnel.service`.

```
[Unit]
Description=Autotunnel - create and maintain SSH tunnels
After=network.target

[Service]
Restart=on-failure
RestartSec=20
ExecStart=/path/to/autotunnel

[Install]
WantedBy=multi-user.target
```

## Manage the service using systemctl

### Start automatically at system boot

```
systemctl enable autotunnel.service
```

### Status

```
systemctl status autotunnel.service
```

### Start

```
systemctl start autotunnel.service
```

### Stop

```
systemctl stop autotunnel.service
```