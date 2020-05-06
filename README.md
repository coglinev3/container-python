# Linux Images with Python for use with Ansible Molecule

[![Build Status](https://travis-ci.com/coglinev3/container-python.svg?branch=master)](https://travis-ci.com/coglinev3/container-python) ![GitHub tag (latest by date)](https://img.shields.io/github/v/tag/coglinev3/container-python) [![GitHub license](https://img.shields.io/github/license/coglinev3/container-python)](https://github.com/coglinev3/container-python/blob/master/LICENSE)


In order to test [Ansible](https://docs.ansible.com/ansible/latest/index.html "Ansible Documentation") roles with the help of [Ansible Molecule](https://molecule.readthedocs.io/en/latest/ "Ansible Molecule Documentation"), Docker images with Python 2 (version 2.7) or Python 3 (version 3.5 and higher) are required. To reduce the image size some official Docker images for Linux distributions (like Alpine or Ubuntu) are not shipped with Python 2 or Python 3 preinstalled. For this reason, you cannot use these Docker images with Ansible.
This repository contains images build by Ansible Playbooks and the Ansible role [coglinev3.ansible_python](https://galaxy.ansible.com/coglinev3/ansible_python) (not with Dockerfiles). The Ansible role coglinev3.ansible_python uses the `raw` module (which executes a low-down and dirty SSH command) to install Python on official Linux images if Python is missing.

## Tags

  - `alpine-3.11`, `latest`: Alpine Linux 3.11
  - `alpine-3.10`: Alpine Linux 3.10
  - `alpine-3.9`: Alpine Linux 3.9
  - `centos-8`: CentOS 8
  - `centos-7`: CentOS 7
  - `centos-6`: CentOS 6
  - `debian-10`, `debian-buster`: Debian 10 (Buster)
  - `debian-9`, `debian-stretch`: Debian 9 (Stretch)
  - `debian-8`, `debian-jessie`: Debian 8 (Jessie)
  - `fedora-32`: Fedora 32
  - `fedora-31`: Fedora 31
  - `fedora-30`: Fedora 30
  - `fedora-29`: Fedora 29
  - `ubuntu-20.04`, `ubuntu-focal`: Ubuntu 20.04 LTS (Focal Fossa)
  - `ubuntu-19.10`, `ubuntu-eoan`: Ubuntu 19.10 (Eoan Ermine)
  - `ubuntu-18.04`, `ubuntu-bionic`: Ubuntu 18.04 LTS (Bionic Beaver)
  - `ubuntu-16.04`, `ubuntu-xenial`: Ubuntu 16.04 LTS (Xenial Xerus)

## Requirements

Ansible and Docker must be installed.


## Environment Variables

The following environment variables are used in conjunction with the playbook.

```sh
# Linux Distribution for the base image (default: centos)
CONTAINER_DISTRO=ubuntu

# Version of the Linux Distribution (default: 7)
CONTAINER_DISTRO_VERSION=16.04

# Namespace for the new image
IMAGE_NAMESPACE=coglinev3                                                                                                                                  

# Name for the new image
IMAGE_NAME=python 

# Optional: Alternate tag for the nєw image
# The default tag is a combination from CONTAINER_DISTRO and CONTAINER_DISTRO_VERSION,
# for example: ubuntu-16.04
# Additionally an alternate tag for the new image can be specified.
IMAGE_ALTERNATE_TAG=ubuntu-xenial
```


## Example Playbook

```yml
---
- hosts: localhost
  connection: local
  gather_facts: false
  vars:
    container_distro: "{{ lookup('env','CONTAINER_DISTRO') | default('centos', true) }}"
    container_distro_version: "{{ lookup('env','CONTAINER_DISTRO_VERSION') | default('7', true) }}"
    base_image: "{{ container_distro }}:{{ container_distro_version }}"
    container_name: "py_{{ container_distro }}_{{ container_distro_version }}"
    image_namespace: "{{ lookup('env','IMAGE_NAMESPACE') | default('coglinev3', true) }}"
    image_name: "{{ lookup('env','IMAGE_NAME') | default('python', true) }}"
    image_tag: "{{ container_distro }}-{{ container_distro_version }}"
  pre_tasks:
    - name: Make the latest version of the base image available locally.
      docker_image:
        name: '{{ base_image }}'
        source: pull
        force_source: true
    - name: Create the Docker container.
      docker_container:
        image: '{{ base_image }}'
        name: '{{ container_name }}'
        command: tail -f /dev/null
    - name: Add the newly created container to the inventory.
      add_host:
        hostname: '{{ container_name }}'
        ansible_connection: docker
  tasks:
    - name: "Install Python if needed"
      import_role:
        name: "coglinev3.ansible_python"
      delegate_to: '{{ container_name }}'
  post_tasks:
    - name: Commit the container.
      command: >
        docker commit
        --author "Cogline.v3"
        {{ container_name }} {{ image_namespace }}/{{ image_name }}:{{ image_tag }}
    - name: Remove the container.
      docker_container:
        name: '{{ container_name }}'
        state: absent
```

## How to Build

You can create an image locally, for example for Ubuntu 18.04, as follows:

```sh
git clone https://github.com/coglinev3/container-python.git
cd container-python
CONTAINER_DISTRO=ubuntu \
CONTAINER_DISTRO_VERSION=18.04 \
IMAGE_ALTERNATE_TAG=ubuntu-bionic \
ansible-playbook container-python.yml
```

## Version

Release: 1.5.0


## License

GNU GPLv3


## Author

Created in 2020, by Cogline.v3.
