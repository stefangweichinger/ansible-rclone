# ansible-rclone

[![Build Status](https://travis-ci.org/stefangweichinger/ansible-rclone.svg?branch=molecule_tests)](https://travis-ci.org/stefangweichinger/ansible-rclone)

ansible role to install [rclone](https://github.com/ncw/rclone)

Usage:

1. clone this repo into your local roles-directory
2. add role to the hosts you want rclone installed to:
3. The variable `rclone_version` can be defined as a release on github or `beta`. 

``` ---
- hosts: rclone-hosts
  roles:
    - rclone
```
4. The variable `install_manpages` can be used to toggle the installation of manpages.

## new molecule based tests

pls feel free to review my first steps into testing the role with molecule
additional distros might be added/tested soon
