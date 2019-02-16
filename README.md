# ansible-rclone

[![Build Status](https://travis-ci.org/stefangweichinger/ansible-rclone.svg?branch=master)](https://travis-ci.org/stefangweichinger/ansible-rclone)

ansible role to install [rclone](https://github.com/ncw/rclone)

## Usage

1. Clone this repo into your local roles-directory
2. Add role to the hosts you want rclone installed to:

``` ---
- hosts: rclone-hosts
  roles:
    - rclone

## Variables

```
1. The variable `install_manpages` can be used to toggle the installation of manpages.
2. The variable `rclone_arch` chooses the target architecture (for example 'amd64').
2. The variable `rclone_version` chooses the version of rclone ('1.64').

## new molecule based tests

New tests using [molecule](https://molecule.readthedocs.io/en/latest/index.html)

pls feel free to review my first steps into testing the role with molecule
additional distros might be added/tested soon
