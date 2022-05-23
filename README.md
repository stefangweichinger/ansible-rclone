# ansible-rclone
AKA `ansible_rclone` and formerly `rclone`

[![Ansible Molecule](https://github.com/stefangweichinger/ansible-rclone/actions/workflows/molecule.yml/badge.svg)](https://github.com/stefangweichinger/ansible-rclone/actions/workflows/molecule.yml)

[![GitHub Open Issues](https://img.shields.io/github/issues/stefangweichinger/ansible-rclone.svg)](https://github.com/stefangweichinger/ansible-rclone/issues)
[![GitHub Stars](https://img.shields.io/github/stars/stefangweichinger/ansible-rclone.svg)](https://github.com/stefangweichinger/ansible-rclone)
[![GitHub Forks](https://img.shields.io/github/forks/stefangweichinger/ansible-rclone.svg)](https://github.com/stefangweichinger/ansible-rclone)

[![Ansible Role](https://img.shields.io/ansible/role/46861.svg)](https://galaxy.ansible.com/stefangweichinger/ansible_rclone)
[![Ansible Role Downloads](https://img.shields.io/ansible/role/d/46861.svg)](https://galaxy.ansible.com/stefangweichinger/ansible_rclone)

## Description

An Ansible role to install [rclone](https://github.com/ncw/rclone)

Please note that this role installs rclone from the upstream repository and not from the OS repositories!

## Usage

1. Clone this repo into your local roles-directory or install via `ansible-galaxy install stefangweichinger.ansible_rclone`
2. Add role to the hosts you want rclone installed to.

## Supported Platforms

The build tests run on a selected set of distros:

* Debian 9,10,11
* Fedora 34,35,36
* Ubuntu 1804,2004
* Ubuntu derivated distros: Linux Mint, Pop!\_OS

Some older releases also work with this role, but I decided to remove some of them from `galaxy_info`.
PRs welcome, but we can't test on every platform.

In early 2022 I removed the tests for CentOS as it is not receiving upstream packages anymore.
The role should still work, though.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

### `install_manpages: "true"`

This can be used to toggle the installation of manpages.

### `rclone_arch: "amd64"`

This variable chooses the target architecture (for example 'amd64').

### `rclone_version: "1.57.0"`

The version of rclone to install. `rclone_version` can be set to a specific version number.
Make sure you add the last number as well: "1.56" won't work, "1.56.0" will do (check upstream repo for available releases).

    rclone_version: "1.56.0"

### `rclone_release: "stable"`

Can be set to "stable", or "beta" to install the latest beta version.

    rclone_release: "beta"

### `rclone_config_location: "/root/.config/rclone/rclone.conf"`

The location to install the rclone config file if you provide `rclone_configs`

### `rclone_configs: ""`

This variable allows for predefined remote configs to be loaded. `rclone_configs` must be a YAML list with a name variable and a properties variable which is another list of arbitrary key value pairs. See the example below which would configure a Google Drive remote:

```
rclone_configs:
  - name: ExampleGoogleDriveRemote
    properties:
      type: drive
      client_id: 12345
      client_secret: 67890
      token: ' {"access_token":"","token_type":"","refresh_token":"","expiry":""}'
```

Note that the space after the single quote `'` for `token` is intentional in order to force this into a string. Otherwise, it will be interpreted as an object and have its double quotes be converted to single quotes within the config file which rclone cannot parse correctly.

#### rclone_configs detailed example

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
      token: ' {"access_token":"<ACCESS_TOKEN>","token_type":"Bearer","refresh_token":"<REFRESH_TOKEN>","expiry":"<DATETIME>"}'
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

## Troubleshooting

**Module Name Issue**:
```
[WARNING]: - stefangweichinger.rclone was NOT installed successfully: - sorry, stefangweichinger.rclone was not found on https://galaxy.ansible.com/api/
```

Note that this module has undergone a name change. It was previously known as `stefangweichinger.rclone`, however Galaxy changed it's naming methods. We are working on a change that will put move it back to this name for simplicity. Ansible Galaxy substitutes `-` to `_` and used to strip a prefix or `ansible`. For more information, see the [this issue](https://github.com/stefangweichinger/ansible-rclone/issues/48).

## Molecule Testing

The tests of this ansible role use [molecule](https://molecule.readthedocs.io/en/latest/index.html)
I use the GitHub Action [Test Ansible roles with Molecule](https://github.com/marketplace/actions/test-ansible-roles-with-molecule) for these tests.

### (current) CI with GitHub Actions

In November 2020 I switched over to running the CI pipeline within Github Actions:

https://github.com/stefangweichinger/ansible-rclone/actions

[![Ansible Molecule](https://github.com/stefangweichinger/ansible-rclone/actions/workflows/molecule.yml/badge.svg)](https://github.com/stefangweichinger/ansible-rclone/actions/workflows/molecule.yml)

This, like this whole repo, is a work in progress.

### tox

In January 2022 I learned about [tox](https://tox.wiki/en/latest/)
`tox` can be used for testing against various different Python versions, Ansible versions, etc.
It is run within the molecule tests now.

### (disabled) Travis CI

At first I used Travis CI, see the logs at https://travis-ci.org/github/stefangweichinger/ansible-rclone

## [Author Information](#author-information)

[Stefan G. Weichinger](https://www.oops.co.at/)

Please consider [sponsoring me](https://github.com/sponsors/stefangweichinger) (upcoming).

