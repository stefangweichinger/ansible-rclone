# ansible-rclone

[![Build Status](https://travis-ci.org/stefangweichinger/ansible-rclone.svg?branch=master)](https://travis-ci.org/stefangweichinger/ansible-rclone)

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
