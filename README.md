# Linux Images with Python for use with Ansible Molecule

[![Build](https://github.com/coglinev3/container-python/actions/workflows/build.yml/badge.svg)](https://github.com/coglinev3/container-python/actions/workflows/build.yml) ![GitHub tag (latest by date)](https://img.shields.io/github/v/tag/coglinev3/container-python) [![GitHub license](https://img.shields.io/github/license/coglinev3/container-python)](https://github.com/coglinev3/container-python/blob/master/LICENSE)


In order to test [Ansible](https://docs.ansible.com/ansible/latest/index.html "Ansible Documentation") roles with the help of [Ansible Molecule](https://molecule.readthedocs.io/en/latest/ "Ansible Molecule Documentation"), Docker images with Python 2 (version 2.7) or Python 3 (version 3.5 and higher) are required. To reduce the image size some official Docker images for Linux distributions (like Alpine or Ubuntu) are not shipped with Python 2 or Python 3 preinstalled. For this reason, you cannot use these Docker images with Ansible.
This repository contains images build by Ansible Playbooks and the Ansible role [coglinev3.ansible_python](https://galaxy.ansible.com/coglinev3/ansible_python) (not with Dockerfiles). The Ansible role coglinev3.ansible_python uses the `raw` module (which executes a low-down and dirty SSH command) to install Python on official Linux images if Python is missing.

## Tags

  - `alpine-3.19`, `latest`: Alpine Linux 3.19
  - `alpine-3.18`, Alpine Linux 3.18
  - `alpine-3.17`, Alpine Linux 3.17
  - `alpine-3.16`, Alpine Linux 3.16
  - `alpine-3.15`, Alpine Linux 3.15
  - `alpine-3.14`, Alpine Linux 3.14
  - `alpine-3.13`, Alpine Linux 3.13
  - `alpine-3.12`: Alpine Linux 3.12
  - `almalinux-8`: AlmaLinux 8
  - `almalinux-9`: AlmaLinux 9
  - `amazonlinux-2023`: Amazon Linux 2023
  - `centos-7`: CentOS 7
  - `debian-12`, `debian-bookworm`: Debian 12 (Bookworm)
  - `debian-11`, `debian-bullseye`: Debian 11 (Bullseye)
  - `debian-10`, `debian-buster`: Debian 10 (Buster)
  - `fedora-39`: Fedora 39
  - `fedora-38`: Fedora 38
  - `fedora-37`: Fedora 37
  - `fedora-36`: Fedora 36
  - `fedora-35`: Fedora 35
  - `fedora-34`: Fedora 34
  - `ubuntu-22.04`, `ubuntu-jammy`: Ubuntu 22.04 LTS (Jammy Jellyfish)
  - `ubuntu-20.04`, `ubuntu-focal`: Ubuntu 20.04 LTS (Focal Fossa)
  - `ubuntu-18.04`, `ubuntu-bionic`: Ubuntu 18.04 LTS (Bionic Beaver)

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
- name: Create Linux container with python
  hosts: localhost
  connection: local
  gather_facts: false
  vars:
    container_namespace: "{{ lookup('env','CONTAINER_NAMESPACE') | default('docker.io', true) }}"
    container_distro: "{{ lookup('env','CONTAINER_DISTRO') | default('alpine', true) }}"
    container_distro_version: "{{ lookup('env','CONTAINER_DISTRO_VERSION') | default('3.18', true) }}"
    base_image: "{{ container_namespace }}/{{ container_distro }}:{{ container_distro_version }}"
    container_name: "py_{{ container_distro }}_{{ container_distro_version }}"
    image_namespace: "{{ lookup('env','IMAGE_NAMESPACE') | default('coglinev3', true) }}"
    image_name: "{{ lookup('env','IMAGE_NAME') | default('python', true) }}"
    image_tag: "{{ container_distro }}-{{ container_distro_version }}"
  pre_tasks:
    - name: "Make base image available locally: {{ base_image }}"
      community.docker.docker_image:
        name: '{{ base_image }}'
        source: pull
        force_source: true
    - name: Create the Docker container.
      community.general.docker_container:
        container_default_behavior: no_defaults
        image: '{{ base_image }}'
        name: '{{ container_name }}'
        command: tail -f /dev/null
    - name: Add the newly created container to the inventory.
      ansible.builtin.add_host:
        hostname: '{{ container_name }}'
        ansible_connection: docker
  tasks:
    - name: "Install Python if needed"
      ansible.builtin.import_role:
        name: "coglinev3.ansible_python"
      delegate_to: '{{ container_name }}'
  post_tasks:
    - name: Commit the container.
      ansible.builtin.command: >
        docker commit
        --author "Cogline.v3"
        {{ container_name }} {{ image_namespace }}/{{ image_name }}:{{ image_tag }}
      register: docker_commit
      changed_when: docker_commit.rc == 0
      failed_when: docker_commit.rc != 0
    - name: Remove the container.
      community.general.docker_container:
        container_default_behavior: no_defaults
        name: '{{ container_name }}'
        state: absent
```

## How to Build

You can create an image locally, for example for Ubuntu 22.04, as follows:

```sh
git clone https://github.com/coglinev3/container-python.git
cd container-python
CONTAINER_DISTRO=ubuntu \
CONTAINER_DISTRO_VERSION=22.04 \
IMAGE_ALTERNATE_TAG=ubuntu-jammy \
ansible-playbook container-python.yml
```

## Version

Release: 1.17.1


## License

GNU GPLv3


## Author

Copyright &copy; 2024 Cogline.v3.
