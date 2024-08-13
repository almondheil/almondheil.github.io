---
layout: post
category: til
slug: ansible on localhost
title: Getting started with Ansible on localhost
---

These last few days I've been working hard to use Ansible to manage my localhost configuration, and
since Ansible is really built for big server setups I didn't find very many guides that were helpful.

I want to write this as a help to other people who are in the same situation as me--wanting to use Ansible to manage
their own computer, but not knowing where to start.

## Table of contents

- [Table of Contents](#table-of-contents)
- [First, my scope](#first-my-scope)
- [Files to run Ansible](#files-to-run-ansible)
    - [roles (directory)](#roles-directory)
    - [setup.yml](#setupyml)
    - [ansible.cfg](#ansiblecfg)
    - [inventory.ini](#inventoryini)
    - [host_vars (directory)](#host_vars-directory)
- [Running Ansible](#running-ansible)

## First, my scope

My scope for this is pretty small, because I needed to keep things under control.

- Machine: My laptop
- Linux distro: Arch linux
- Desktop environment: GNOME
- What to manage: Things that aren't already managed by chezmoi (see [this other post](/til/chezmoi-good)), including:
    - packages
    - /etc configurations
    - installing chezmoi

If you want to look at all my files, they're on my github: <https://github.com/almondheil/ansible-arch>{:target="_blank"}

## Files to run Ansible

Once you've installed Ansible, it needs some setup to tell it what machines it is managing and how.

These are the files that I needed to make my setup work. Click one to jump to that header.

- The most important files:

    - [roles](#roles-directory) - a directory of all the different things that Ansible manages.
      This is set up in a very nested way, which I'll get to.

    - [setup.yml](#setupyml) - a "playbook" that tells Ansible all the roles it needs to run.

- The less-but-still-pretty-important files:

    - [ansible.cfg](#ansiblecfg) - configuration for ansible overall

    - [inventory.ini](#inventoryini) - a file that tells Ansible what machines it manages

    - [host_vars](#host_vars-directory) - for each machine Ansible manages, variables specific to that machine

### roles (directory)

At minimum, a role looks like this.

```text
roles
└── role_name
    └── tasks
        └── main.yml
```

There are a LOT of other things that can go under the role_name directory, but the only one I used
while setting up my computer was the files subdirectory. This lets you have files that can get copied
to a location on your computer. which was often helpful with /etc configuration.

main.yml is the most important part -- it gives a list of tasks that need to be run to complete the role.
These include things like running commands, installing packages, and a whole lot more.

Every task is made up of a module (which can be built into Ansible or installed from an external source),
plus some arguments along with it.

Here's an example task that installs a package:

```yaml
- name: Install some package
  become: true
  community.general.pacman:
    name: <package-name>
    state: present
```

The important bits here are:

- The `become: true` line makes this command run as sudo. I am going to run Ansible with my user account, so I need to do this
  in order to get elevated permissions.
- The `community.general.pacman` line tells Ansible to use that as a module.
  See the [module documentation](https://docs.ansible.com/ansible/latest/collections/community/general/pacman_module.html){:target="_blank"} for more if you want.

### setup.yml

The setup.yml file is called a playbook, and it collects together all your roles.

You can also put tasks directly in the playbook, but I feel like this looks sorta bad. With that in mind,
here's a basic way to set up a playbook:

```yaml
---
- hosts: laptop
  roles:
    - { role: role1, tags: ['role1'] }
    - { role: role2, tags: ['role2'] }
    - { role: role3, tags: ['role3'] }
    - { role: role4, tags: ['role4'] }
```

#### Aside: Pre-prompting chezmoi file decryption

I encrypt some of my files with AGE in my chezmoi repo, and I wanted to make
sure these files could be decrypted right away when I started up my computer.
The thing is, the decryption asks for a passphrase. I didn't want to store this
in a file, so my best option was making Ansible prompt me for my passkey.

I did this by using the `vars_prompt` directive in my playbook, where I could
give prompts and variables to store the responses in:

```yaml
- hosts: laptop
  vars_prompt:
    - name: chezmoi_passphrase
      prompt: Chezmoi encrpyion passphrase
  roles:
    - { role: chezmoi, tags: ['chezmoi'] }
    - other roles...
```

Once you do that, you can use `chezmoi_passphrase` as a variable wherever you need to. In my case,
this was part of an `ansible.builtin.expect` module in order to respond with the appropriate passphrase.

### ansible.cfg

This file has very little going on. Basically, I just used it to silence a warning about the python version
on my system. Here are the file contents:

```text
[defaults]
interpreter_python = auto_silent
```

I did this because when Ansible is running locally, it realizes that you could use it to install a new version
of python--which would then change the version of python it is relying on to run. By setting the interpreter to be
silent, I just make it not print this warning. I don't change the python version in my setup, after all.

### inventory.ini

The inventory can also be stored as a directory, but since we're doing only one machine it's
fine to just do a single file.

The file has pretty simple contents:

```ini
[laptop]
127.0.0.1 ansible_connection=local
```

Here, we define a group called "laptop" with a single host (127.0.0.1) inside. 

We also specify ansible_connection=local, which tells Ansible not to try to SSH
into this host since it's running on that host.

### host_vars (directory)

In our case, this directory is set up like so. The only file inside is a YAML
file with the name of our only host, 127.0.0.1.

```text
host_vars
└── 127.0.0.1.yml
```

Inside that file, you can set variables that will be used when Ansible is working
with that host, and you can sub them in with Jinja2 formatting

I just made mine a handful of really simple key:value pairs like my home directory, but
you could do a lot more here if you needed to.

## Running Ansible

To run Ansible, we're going to run the setup.yml playbook that we made. It's necessary to provide
the following info:

- Inventory to use
- Password to use for any sudo operations (or, as Ansible puts it, BECOME)
- Machine(s) to run on
- Playbook to run

These are included in the following command:

```bash
ansible-playbook -i inventory.ini -K -l laptop setup.yml
```

-i specifies the inventory, -K tells Ansible to prompt for the BECOME password,
-l specifies which hosts to run on, and we give the playbook as the last
argument.

We can also optionally specify which tags to run, which narrows down which roles
will happen. This can be with one tag: 

```bash
ansible-playbook -i inventory.ini -K -l laptop setup.yml -t tag
```

Or with multiple:

```bash
ansible-playbook -i inventory.ini -K -l laptop setup.yml -t tag1,tag2
```
