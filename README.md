# piscripts

A collection of scripts for Raspberry Pi.

- [Dependencies](#dependencies)
- [Installation](#installation)
- [Unbound (optional)](#unbound-optional)
- [Tailscale (optional)](#tailscale-optional)

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

## Tailscale (optional)

Join the Pi to your tailnet, with `tailscaled` enabled at boot and restarted automatically if it crashes:

```
setup-tailscale
```

Optional environment variables:

```
TS_AUTHKEY=tskey-...          # unattended login; without it a login URL is printed
TS_HOSTNAME=pihole            # Tailscale machine name (default: hostname)
TS_ROUTES=192.168.200.0/24    # comma-separated subnets to advertise
TS_EXIT_NODE=1                # advertise this Pi as an exit node
TS_PIHOLE_DNS=1               # let Pi-hole answer DNS for tailnet clients
```

Runs with `--accept-dns=false` so Pi-hole keeps its own upstream instead of Tailscale's DNS. Idempotent.
