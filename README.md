# ansible-rclone

[![Build Status](https://travis-ci.org/stefangweichinger/ansible-rclone.svg?branch=master)](https://travis-ci.org/stefangweichinger/ansible-rclone)

[![GitHub Open Issues](https://img.shields.io/github/issues/stefangweichinger/ansible-rclone.svg)](https://github.com/stefangweichinger/ansible-rclone/issues)
[![GitHub Stars](https://img.shields.io/github/stars/stefangweichinger/ansible-rclone.svg)](https://github.com/stefangweichinger/ansible-rclone)
[![GitHub Forks](https://img.shields.io/github/forks/stefangweichinger/ansible-rclone.svg)](https://github.com/stefangweichinger/ansible-rclone)

[![Ansible Role](https://img.shields.io/ansible/role/15522.svg)](https://galaxy.ansible.com/stefangweichinger/rclone)
[![Ansible Role Downloads](https://img.shields.io/ansible/role/d/15522.svg)](https://galaxy.ansible.com/stefangweichinger/rclone)


ansible role to install [rclone](https://github.com/ncw/rclone)

Please note that this role installs rclone from the upstream repository and not from the OS repositories!

## Usage

1. Clone this repo into your local roles-directory or install via `ansible-galaxy install stefangweichinger.rclone`.
2. Add role to the hosts you want rclone installed to.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    install_manpages: "true"

This can be used to toggle the installation of manpages.

    rclone_arch: "amd64"

This variable chooses the target architecture (for example 'amd64').

    rclone_version: ""

The version of rclone to install. `rclone_version` is no longer set as a default, but if provided as a variable, can be set to a specific version number or "beta" to install the latest beta version.

    rclone_config_location: "/root/.config/rclone/rclone.conf"

The location to install the rclone config file if you provide `rclone_configs`

    rclone_configs: ""

This variable allows for predefined remote configs to be loaded. `rclone_configs` must be a YAML list with a name variable and a properties variable which is another list of arbitrary key value pairs. See the example below which would configure a Google Drive remote:

```
rclone_configs:
  - name: ExampleGoogleDriveRemote
    properties:
      type: drive
      client_id: 12345
      client_secret: 67890
      token: {"access_token":"","token_type":"","refresh_token":"","expiry":""}
```

### rclone_configs detailed example

The `rclone_configs` variable is used to recreate the `rclone.conf` file. This config file stores the rclone remotes that have been defined and are usable. This is an alternative to simply copying a stored `rclone.conf` file directly.

The `rclone_configs` simply takes a list of YAML objects which must have a `name` which will map to the name of the remote, and a `properties` which can be any key, value pairs which will map to the variables stored under that remote. These should be the ones expected by the `rclone.conf` file for the remote type you're configuring.

For example, to recreate a standard encrypted Google Drive mount setup, the `rclone.conf` will look similar to below, assuming you have your encypted files stored in the "media" folder on Google Drive:

```
[google-drive]
type = drive
client_id = <CLIENT_ID>
client_secret = <CLIENT_SECRET>
token = {"access_token":"<ACCESS_TOKEN>","token_type":"Bearer","refresh_token":"<REFRESH_TOKEN>","expiry":"<DATETIME>"}
root_folder_id = <ROOT_FOLDER_ID>

[encrypted-media]
type = crypt
remote = google-drive:media
filename_encryption = standard
password = <PASSWORD>
password2 = <PASSWORD2>
```

To enable ansible-rclone to recreate that config file, you can provide an `rclone_configs` variable as follows.
Note that this should always be encrypted if stored publicly as it gives access to your remotes:

```
rclone_configs:
  - name: google-drive
    properties:
      type: drive
      client_id: <CLIENT_ID>
      client_secret: <CLIENT_SECRET>
      token: {"access_token":"<ACCESS_TOKEN>","token_type":"Bearer","refresh_token":"<REFRESH_TOKEN>","expiry":"<DATETIME>"}
      root_folder_id = <ROOT_FOLDER_ID>
  - name: encrypted-media
    properties:
      type: crypt
      remote: google-drive:media
      filename_encryption: standard
      password: <PASSWORD>
      password2: <PASSWORD2>

```

## Dependencies

None.

## Example Playbook

    - hosts: rclone-hosts
      roles:
        - rclone

## Tests

New tests using [molecule](https://molecule.readthedocs.io/en/latest/index.html)

pls feel free to review my first steps into testing the role with molecule
additional distros might be added/tested soon

Currently there are two scenarios defined for running [molecule](https://molecule.readthedocs.io/en/latest/index.html):

* default:    runs tests over all defined distros at once, I use this for a local and overall test
* travis:     this one runs one travis-test-job per distro and is called via .travis.yml
    
