---
layout: post
title: "Sample Ansible Playbook"
date: 2025-07-05 12:00:00 +0530
categories: [automation, ansible]
tags: [ansible, devops, playbook]
---

Here is a basic Ansible playbook to install Apache:

```yaml
---
- name: Install Apache Web Server
  hosts: webservers
  become: true
  tasks:
    - name: Install Apache
      apt:
        name: apache2
        state: present
