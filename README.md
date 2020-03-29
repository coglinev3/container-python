# Linux Images with Python for use with Ansible Molecule

[![Build Status](https://travis-ci.com/coglinev3/container-python.svg?branch=master)](https://travis-ci.com/coglinev3/container-python) ![GitHub tag (latest by date)](https://img.shields.io/github/v/tag/coglinev3/container-python) [![GitHub license](https://img.shields.io/github/license/coglinev3/container-python)](https://github.com/coglinev3/container-python/blob/master/LICENSE)


In order to test [Ansible](https://docs.ansible.com/ansible/latest/index.html "Ansible Documentation") roles with the help of [Ansible Molecule](https://molecule.readthedocs.io/en/latest/ "Ansible Molecule Documentation"), Docker images with Python 2 (version 2.7) or Python 3 (version 3.5 and higher) are required. To reduce the image size some official Docker images for Linux distributions (like Alpine or Ubuntu) are not shipped with Python 2 or Python 3 preinstalled. For this reason, you cannot use these Docker images with Ansible.
This repository contains images build by Ansible Playbooks and the Ansible role [coglinev3.ansible_python](https://galaxy.ansible.com/coglinev3/ansible_python) (not with Dockerfiles). The Ansible role coglinev3.ansible_python uses the `raw` module (which executes a low-down and dirty SSH command) to install Python on official Linux images if Python is missing.

## Tags

  - `Alpine 3.9`: Alpine Linux 3.9
  - `Alpine 3.10`: Alpine Linux 3.10
  - `Alpine 3.11`: Alpine Linux 3.11
  - `ubuntu-16.04`, `ubuntu-xenial`: Ubuntu 16.04 LTS (Xenial Xerus)
  - `ubuntu-18.04`, `ubuntu-bionic`: Ubuntu 18.04 LTS (Bionic Beaver)

## Requirements

Ansible and Docker must be installed.

## Example Playbook

```yml
---
- hosts: localhost
  connection: local
  gather_facts: no
  vars:
    base_image: ubuntu
    base_image_tag: 16.04
    container_name: py_ubuntu_1604
    image_namespace: coglinev3
    image_name: python
    image_tag: ubuntu-16.04
  pre_tasks:
    - name: Make the latest version of the base image available locally.
      docker_image:
        name: '{{ base_image }}:{{ base_image_tag }}'
        source: pull
        force_source: true
    - name: Create the Docker container.
      docker_container:
        image: '{{ base_image }}:{{ base_image_tag }}'
        name: '{{ container_name }}'
        command: sleep infinity
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

You can build the image with

```sh
git clone https://github.com/coglinev3/container-python.git
cd container-python
ansible-playbook python-ubuntu-16.04.yml
```

## Version

Release: 1.1.1

## License

GNU GPLv3

## Author

Created in 2020, by Cogline.v3.
