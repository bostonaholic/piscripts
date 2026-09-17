# piscripts

A collection of scripts for Raspberry Pi.

- [Dependencies](#dependencies)
- [Installation](#installation)
- [Unbound (optional)](#unbound-optional)

## Dependencies

Git

```
sudo apt-get install git --no-install-recommends --verbose-versions --yes
```

## Installation

Clone the repository

```
mkdir $HOME/code && cd $HOME/code
git clone https://github.com/bostonaholic/piscripts.git
```

Run the `./script/setup` script

```
./script/setup
```

## Unbound (optional)

Run Pi-hole against a local recursive resolver instead of a public upstream:

```
setup-unbound
```

Installs `unbound` listening on `127.0.0.1#5335`, verifies DNSSEC validation, and sets Pi-hole's upstream DNS to it. Idempotent. Follows the [Pi-hole unbound guide](https://docs.pi-hole.net/guides/dns/unbound/).
