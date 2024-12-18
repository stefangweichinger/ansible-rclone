# ansible-rclone

![Ansible Galaxy Role Downloads](https://img.shields.io/ansible/role/d/stefangweichinger/ansible_rclone?style=for-the-badge)

![Tag](https://img.shields.io/github/v/tag/stefangweichinger/ansible-rclone?sort=semver&style=for-the-badge)

![GitHub Workflow Status (with event)](https://img.shields.io/github/actions/workflow/status/stefangweichinger/ansible-rclone/molecule.yml)

[![Ansible Molecule](https://github.com/stefangweichinger/ansible-rclone/actions/workflows/molecule.yml/badge.svg)](https://github.com/stefangweichinger/ansible-rclone/actions/workflows/molecule.yml)

[![GitHub Open Issues](https://img.shields.io/github/issues/stefangweichinger/ansible-rclone.svg)](https://github.com/stefangweichinger/ansible-rclone/issues)
[![GitHub Stars](https://img.shields.io/github/stars/stefangweichinger/ansible-rclone.svg)](https://github.com/stefangweichinger/ansible-rclone)
[![GitHub Forks](https://img.shields.io/github/forks/stefangweichinger/ansible-rclone.svg)](https://github.com/stefangweichinger/ansible-rclone)



AKA `ansible_rclone` and formerly `rclone`

## Description

### tl;dr

An [Ansible](https://www.ansible.com/) role to install [rclone](https://github.com/ncw/rclone) directly from upstream.

## Usage

1. Clone this repo into your local roles-directory or install via `ansible-galaxy install stefangweichinger.ansible_rclone`
2. Add role to the hosts you want `rclone` installed to.
4. Run `ansible` ...

Please note that this role installs `rclone` from the upstream repository and not from the OS repositories!

And feel free to rate the role on [Ansible Galaxy](https://galaxy.ansible.com/stefangweichinger/ansible_rclone) to improve its "Community Score
". You're welcome!

## Supported Platforms

The build tests use the galaxy-action [molecule action](https://github.com/marketplace/actions/test-ansible-roles-with-molecule) and run on a selected set of distros:

* Arch Linux
* Debian 11,12
* Fedora 39,40
* Ubuntu 2204,2404
* Ubuntu-derived distros: Linux Mint, Pop!\_OS

Some older and newer releases also work with this role, but I decided to remove some of them from `galaxy_info`.
PRs welcome, but we can't test on every platform.

In early 2022 I removed the tests for CentOS as it is not receiving upstream packages anymore.
The role should still work, though.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

### `install_manpages: "true"`

This can be used to toggle the installation of manpages.

### `rclone_manpages_location: "{{ default_rclone_manpages_location }}"`

The location to install `rclone` manpages. The default is an OS specific location, but you can override it anywhere.

### `rclone_manpages_owner:`

These variables allow for setting the ownership of manpages for `rclone`. They are mostly needed if configuring `rclone` to run as an other user than root (maybe a specific backup user or so).

    rclone_manpages_owner:
       OWNER: rclone
       GROUP: rclone

### `rclone_binary_location: "/usr/local/bin/"`

The location to install the `rclone` binary.

### `rclone_binary_owner:`

These variables allow for setting the ownership of the `rclone` binary. They are mostly needed if configuring `rclone` to install for non-root user.

    rclone_binary_location: "/home/rclone/.local/bin/"
    rclone_binary_owner:
       OWNER: rclone
       GROUP: rclone

### `rclone_fact_path: "/etc/ansible/facts.d/rclone.fact"`

The location to ansible local facts for `rclone`. They are mostly needed if you run this role by non-root user.

    rclone_fact_path: "/home/rclone/.config/ansible/facts.d/rclone.fact"

### `rclone_arch: "amd64"`

This variable chooses the target architecture (for example 'amd64').

### `rclone_version: "1.67.0"`

The version of `rclone` to install. `rclone_version` can be set to a specific version number.
Make sure you add the last number as well: "1.56" won't work, "1.56.0" will do (check upstream repo for available releases).

    rclone_version: "1.56.0"

### `rclone_release: "stable"`

Can be set to "stable", or "beta" to install the latest beta version.

    rclone_release: "beta"

### `rclone_config_location: "/root/.config/rclone/rclone.conf"`

The location to install the `rclone` config file to if you provide `rclone_configs`

### `rclone_config_owner:`

These variables allow for setting the ownership of the directory and config file for `rclone`. They are mostly needed if configuring `rclone` to run as an other user than root (maybe a specific backup user or so).

    rclone_config_owner:
       OWNER: rclone
       GROUP: rclone

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

Note that the space after the single quote `'` for `token` is intentional in order to force this into a string. Otherwise, it will be interpreted as an object and have its double quotes be converted to single quotes within the config file which `rclone` cannot parse correctly.

#### rclone_configs detailed example

The `rclone_configs` variable is used to recreate the `rclone.conf` file. This config file stores the `rclone` remotes that have been defined and are usable. This is an alternative to simply copying a stored `rclone.conf` file directly.

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

To enable the role to recreate that config file, you can provide an `rclone_configs` variable as follows.
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

The task to create/update `rclone.conf` registers a variable named `setup_rclone_config`. The attributes of the variable are that of the `ansible.builtin.template` module (e.g. `setup_rclone_config.dest`, `setup_rclone_config.uid`, etc.) and common return values (e.g. `setup_rclone_config.changed`, `setup_rclone_config.failed`, etc.)

You can reference this variable in later tasks if you wish to perform an action when the config has been updated. For example:

```yml
- name: Install and configure rclone
  ansible.builtin.include_role:
    name: stefangweichinger.ansible_rclone
  vars:
    rclone_configs:
      - name: ExampleGoogleDriveRemote
        properties:
          type: drive
          client_id: 12345
          client_secret: 67890

- name: Restart rclone
  ansible.builtin.systemd:
    name: rclone.service
    state: restarted
  when: setup_rclone_config.changed
```

Note: This example assumes you have created the `rclone.service` systemd unit yourself. That action is not a function of this role.

### `rclone_mounts: ""`

This variable allows for the configuration of rclone mounts within your infrastructure. `rclone_mounts` should be a YAML list of objects, each including keys for `name`, `remote_name`, `remote_path`, `local_path`, `auto_mount`, and `extra_args`. Optionally, you can also pass `local_path_mode` (defaults to `0755`), `local_path_owner`, and `local_path_group` (both default to `root`).

This setup enables precise control over multiple mount points, their remote sources, and whether they should be automatically mounted.

The role will attempt to create all defined `remote_path`'s by default, **if** the `remote_path` is not simply `/`. You can disable this by setting `rclone_mkdir` to `false`.

If you use `rclone_mounts`, you must run this role as root using `become: true`.

#### Detailed example for `rclone_mounts`

To define mounts, you'll specify each mount's details in the playbook, allowing Ansible to handle the mounting process as per your configuration. This method is advantageous for managing mounts across multiple systems or ensuring persistent mounts across reboots when combined with the `auto_mount` option.
The `name` defines the mount's identifier, used partly in naming the service. `remote_name` and `remote_path` determine the remote storage location, while `local_path` indicates where it mounts locally.
`auto_mount` controls whether the mount and corresponding service automatically activate, ensuring availability after reboots or redeployments.
`extra_args` allows for passing in extra arguments to the mount command such as `--allow-other`

When incorporating rclone_mounts into your setup, each mount point you define will correspond to a system service, facilitating management and automation.

Example configuration for mounting a directory from Backblaze B2 as a local backup directory:

```yaml
rclone_mounts:
  - name: my-app
    remote_name: BackblazeLM
    remote_path: "/my-app"
    local_path: "/var/backups/my-app/"
    local_path_mode: 0755
    local_path_owner: root
    local_path_group: root
    auto_mount: true
    extra_args: "--allow-other"
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

Note that this module has undergone a name change. It was previously known as `stefangweichinger.rclone`, however Galaxy changed its naming methods. I am  considering a change that will put move it back to this name for simplicity. Ansible Galaxy substitutes `-` to `_` and used to strip a prefix or `ansible`. For more information, see the [this issue](https://github.com/stefangweichinger/ansible-rclone/issues/48).

## Versioning

Changed in January 2023:

We are using [SemVer](https://semver.org/) and **CHANGELOG generation** powered by [Conventional Commits](https://conventionalcommits.org).
Commit messages are now done using [commitizen](https://github.com/commitizen-tools/commitizen), which is run in a pre-commit command.

This is new and will maybe see more finetuning.

## Molecule Testing

The tests of this ansible role use [molecule](https://molecule.readthedocs.io/en/latest/index.html).
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

Please consider [sponsoring this project](https://github.com/sponsors/stefangweichinger) and/or rating the role on [Ansible Galaxy](https://galaxy.ansible.com/stefangweichinger/ansible_rclone).
