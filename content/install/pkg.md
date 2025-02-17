---
title: Installing packages
parent: Installation
nav_order: 20
---

# Installing packages

## SCION Services
Prebuilt SCION packages are available for Ubuntu (or other Debian based systems) for x86, x86-64, arm32 and arm64 platforms.

You can install SCION from our `.deb`-packages by running:

```shell
sudo apt-get install apt-transport-https
echo "deb [trusted=yes] https://packages.netsec.inf.ethz.ch/debian all main" | sudo tee /etc/apt/sources.list.d/scionlab.list
sudo apt-get update
sudo apt-get install scionlab
```

This will install all the SCION services (which come as individual packages),
`systemd` unit files to run the services as well as a helper script to
install the SCION configuration generated by the SCIONLab coordination website.


## Configuration

After [creating or modifying your AS](/content/config/create_as/) in the SCIONLab coordination website, you can deploy the generated configuration to your machine.
For this, the recommended approach is to simply run the `scionlab-config` script that is included in the `scionlab` package.

### `scionlab-config`
This script allows to conveniently fetch and install the configuration for your AS from the SCIONLab website.

```shell
sudo scionlab-config --host-id=<...> --host-secret=<...>
```
The required `host-id` and `host-secret` will be displayed on the SCIONLab website.
The script will (re-)start all the configured services (and OpenVPN client, if configured).
The `host-id` and `host-secret` information will be stored in
`/etc/scion/gen/scionlab-config.json` and will not have to be entered
again.  To update the configuration after modifying your AS, simply run
```shell
sudo scionlab-config
```

Run `scionlab-config --force` to get the configuration even if the SCIONLab coordination
website thinks you're running the latest version, e.g. in case you've locally
modified your configuration and want it to be reset.


{% include alert type="note" content="
The SCION services will be run as user `scion`.
The configuration files for the SCION services are kept in `/etc/scion/`,
log files will be placed in `/var/log/scion/`,
and state (sqlite db-files) will be placed in `/var/run/scion/`.
" %}


### Unpack configuration manually
As an alternative to running `scionlab-config`, you can manually download the
configuration tarfile from the SCIONLab website and unpack it.

1. Download the configuration tarfile from the SCIONLab coordination website.

2. If using VPN, extract the `client.conf` to `/etc/openvpn/` and (re-)start OpenVPN

        #!shell
        sudo systemctl restart openvpn@client

3. Extract the `gen/` subdirectory to `/etc/scion/`
4. Enable the systemd units listed in `scionlab-services.txt` and restart SCION:

        #!shell
        # Stop SCION and disable services referring to old configuration
        sudo systemctl stop scionlab.target
        sudo rm /etc/systemd/system/scionlab.target.wants/*
        sudo systemctl daemon-reload
        # Enable each unit listed in the scionlab-services.txt file
        while read line; do
          sudo systemctl enable $line
        done < scionlab-services.txt
        # Start SCION
        sudo systemctl start scionlab.target

[//]: # (TODO mkdocs is broken here, the f*in < renderes as &lt; ffs)


## Running SCION

If using VPN, ensure that the OpenVPN-client is up **before** starting the SCION services.
```shell
sudo systemctl start openvpn@client
```
Check that the expected `tun0` tunnel-interface is created before continuing. Please refer to corresponding [troubleshooting](/content/faq/troubleshooting/) page.

[//]: # (TODO This may become obsolete if openvpn@client is included as a dependency for the BRs.)


The SCION services are configured as `systemd`-units that are controlled by the target `scionlab.target`.
To start all the configured SCION services, simply run

```shell
sudo systemctl start scionlab.target
```


Check the status of the SCION services using
```shell
sudo systemctl list-dependencies scionlab.target
```

To stop the services, run
```shell
sudo systemctl stop scionlab.target
```

Find the log files of the SCION services in `/var/log/scion/`.


## Applications

Most of our applications in [`scion-apps`](https://github.com/netsec-ethz/scion-apps/) are available as packages too.
Assuming you've added the SCIONLab packages list above, to install all applications in one go simply run

```shell
sudo apt-get install scion-apps-*
```

or install them individually, e.g. for [bwtester](/content/apps/bwtester/)
```shell
sudo apt-get install scion-apps-bwtester
```
