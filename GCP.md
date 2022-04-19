# APSIM Web Infrastructure

Most (all?) of the apsim web infrastructure is hosted on GCP VMs. We have a windows VM (GCP name is apsim2 - more colloquially known as apsimdev) which runs several sites behind IIS. We also have a newer Linux VM (apsim-web) which runs several websites in docker behind apache.

## apsim2 (apsimdev)

This is the old windows server which historically hosted everything. It may be accessed by remote desktop at apsimdev.apsim.info. There are quite a lot of websites still running here, but the only ones which are actually used are:

- [POStats](POStats.md)
- APSoil
- The old APSIM.Builds site which is still accessed by old versions of apsim and parts of the old apsim CI process

All of the websites and their data are stored on D:\ drive which is a GCP volume.

## apsim-web (dev.apsim.info)

apsim-web is a Linux VM. We are in the process of migrating most of our web infrastructure here. All of the existing infrastructure running here is running inside docker, with the configuration files all stored in the [apsim-web](https://github.com/hol430/apsim-web) repository.

SSH access to the server is publickey-only. Password authentication is disabled, so new users must have their public key added by an existing user. The user account used to manage the web services is webmaster.

The websites' persistent data is all stored in a GCP volume mounted at /data.

## DNS

The apsim.info domain name is managed manually through [GoDaddy](https://www.godaddy.com/en-au).
