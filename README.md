# rpi-traefik-hass-pihole-zerotier

The contents of the repository are for those who want to run [Zerotier](https://zerotier.com), [Node-Exporter](https://github.com/prometheus/node_exporter), [Docker](https://docker.io), [Traefik](https://traefik.io), [Pi-hole](https://pi-hole.net/) and [home-assistant](https://www.home-assistant.io/) on one raspberry pi. [Prometheus](https://prometheus.io) metrics are enabled for everything except Zerotier and Pi-hole ([pi-hole exporter](https://github.com/eko/pihole-exporter) is available).

We assume you have/want a Zerotier account, your raspberry pi is NATed in your local home network, you'll want to scrape the metrics endpoints with prometheus from somewhere on the zerotier network.

## Contents

The repository contains an ansible playbook that install the following software on one raspberry pi:

* Zerotier
> Join a given network and accept the host joined through the API. See config options on the [role galaxy page](https://galaxy.ansible.com/m4rcu5nl/zerotier-one).
* Node-exporter
> Download the node-exporter binary and run it only on the zerotier IP. See config options on the [role galaxy page](https://galaxy.ansible.com/undergreen/prometheus-node-exporter).
* Docker
> Install the latest version of docker, enable experimental metrics endpoint on zerotier IP. See more config options on the [role gakaxy page](https://galaxy.ansible.com/geerlingguy/docker).
* Traefik
> Download and run the traefik v2 container, enable prometheus metrics.
* Pi-hole
> Download and run the pi-hole container, set labels for traefik to proxy connections. Configuration and database is persisted.
* Home-assistant
> Download and run the home-assistant container. Configuration and database is persisted.

## Setup

### Python

I'm personally using [pyenv](https://github.com/pyenv/pyenv) and [pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv) to manage the version of python and virtualenvs on my machine, once you have it installed, it will pick up the version of python automatically, I'd suggest you create a virtualenv for ansbile:

```
pyenv install 3.7.3
pyenv virtualenv 3.7.3 ansible
pyenv active ansible
pip install -r requirements.txt
```

### Ansible

To install the roles needed we'll use ansible-galaxy:

```
ansible-galaxy install -r requirements.yml
```

The roles will be installed inside the `role` directory, configuration can be found inside the [asible.cfg](ansible.cfg#L5) file.

I've disabled retry files and host key checking, feel free to change them to your preference, it's all in the [ansible config file](ansible.cfg)

### Secrets

Secrets have been encoded with `ansible-vault`, they can be found in [group_vars/all](group_vars/all). You'll need to update them with your own:

```
ansible-vault encrypt_string '12ac4a1e71e88666' --name 'vault_zerotier_network' --ask-vault-pass
```

Copy from `vault_zerotier_network` and paste it in to the group_vars/all file. Do the same for the other variables. Remove the `ddwrt` ones, if it's not needed, I placed it as an example.

### DNS and localhost

By default the containers are available through traefik on specific hosts, make sure you add entries to your `/etc/hosts` or your networks DNS server (later they can be addded to pi-hole). Example 10.10.10.10 being your raspberry pi IP address:

```
10.10.10.10 traefik
10.10.10.10 pi-hole
10.10.10.10 hass
```

By default http://<raspberry-pi-ip-address> will forward requests to pi-hole. Config done based on [the pi-hole documentation](https://github.com/pi-hole/docker-pi-hole#running-pi-hole-docker).

### Inventory

Make sure you update [the inventory](inventory/servers.ini) with the IP address of your raspberry pi and adjust the zerotier IP address for it.

## Running it

Ansible has a bug in some instances where you need to copy your SSH key to the raspberry-pi before you can run `ansible-playbook`.

```
ssh-copy-id pi@10.10.10.10
```

Ready to run the playbook with the user pi:

```
ansible-playbook -u pi -b -K rpi.yml --ask-vault-pass
```

## Endpoints

The following endpoints should be available, in all cases, `10.10.10.10` is assumed as the local IP of your raspberry pi and `10.100.100.100` as the ZeroTier IP:

http://10.10.10.10 - Pihole interface

http://10.10.10.10:9100/metrics - Node_exporter metrics

http://10.100.100.100:9100/metrics - Node_exporter metrics

http://traefik - Traefik Dashboard

http://pihole - Pihole admin interface

http://hass - Home-assistant lovelace dashboard

http:// - Home-assistant metrics (TO DO)

http://10.10.10.10:9323/metrics - Docker metrics

http://10.100.100.100:9323/metrics - Docker metrics

http://10.100.100.100/metrics - Traefik metrics (TO DO)
