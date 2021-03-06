+++
title = "Collections from Playbook"
weight = 2
+++

For the following exercise, we will use a collection written by the Ansible Core Team. The name of the author is therefore "ansible". You can find a list of all modules and collections written by the Ansible Core Team on [Ansible Galaxy](https://galaxy.ansible.com/ansible). Head over there and have a good look around!

As you can see they maintain several collections and roles. One of their collections is called "posix" and we can find the documentation and additional details on the [Ansible Galaxy POSIX Collection](https://galaxy.ansible.com/ansible/posix) page.

One of the modules provided by this collection allows us to manage [SELinux](https://www.redhat.com/en/topics/linux/what-is-selinux) settings. The fully qualified collection name for this module is therefore `ansible.posix.selinux`.

You can find more details about [using collections](https://docs.ansible.com/ansible/latest/user_guide/collections_using.html) in the [Ansible Documentation](https://docs.ansible.com/).

## Install the Ansible Collection

The `ansible.posix.selinux` module which we want to use for this exercise, is part of the `ansible.posix` collection. We have to install this collection first, before we can use its modules. The `ansible-galaxy` command line tool can be used to automate the installation. It is preconfigured to search for roles and collections on [Ansible Galaxy](https://galaxy.ansible.com/) so we can just specify the collection name and it will take care of the rest.

Bring up a browser window with **VS Code** and open a terminal. In the terminal run:

    [{{< param "control_prompt" >}} ~]$ ansible-galaxy collection install ansible.posix

This will install the collection on your system, only if it wasn't installed before. To force the installation, for example to make sure you're on the latest version, you can add the force switch `-f`.

    [{{< param "control_prompt" >}} ~]$ ansible-galaxy collection install -f ansible.posix

This will always download and install the latest version, even if it was already up to date. Ansible Collections can have dependencies for other Ansible Collections as well - if you want to make sure those dependencies are refreshed as well, you can use the `--force-with-deps` switch.

By default the installation is stored in your local `~/.ansible` directory. This can be overwritten by using the `-p /path/to/collection` switch. Keep in mind though that `ansible-playbook` will only use this directory, if you change your `ansible.cfg` accordingly. To check your current configuration, you can dump your configuration and search for `collection`.

```bash
[{{< param "control_prompt" >}} ~]$ ansible-config dump | grep -i collection
COLLECTIONS_PATHS(default) = ['/home/student{{< param "student" >}}/.ansible/collections', '/usr/share/ansible/collections']
```

## Browse the Documentation

The `ansible-doc` command only searches the system directories for documentation. You can still use it though to read up on modules you installed from Ansible Collections by using the fully qualified collection name.

Let's have a look at the module documentation for the `selinux` module of the `ansible.posix` collection, which we are going to use in the next part of the exercise:

    [{{< param "control_prompt" >}} ~]$ ansible-doc ansible.posix.selinux
    > SELINUX    (/home/student{{< param "student" >}}/.ansible/collections/ansible_collections/ansible/posix/plugins/modules/selinux.py)

Note the start of the output, you can see the location of the module. For educational purposes run the `ansible-doc` again, but this time not specifying the collection:

    [{{< param "control_prompt" >}} ~]$ ansible-doc selinux
    > SELINUX    (/usr/lib/python3.6/site-packages/ansible/modules/system/selinux.py)

You can see how the command this time pulled the documentation for the module that was installed with Ansible.

{{% notice note %}}
Depending on your screen resolution you might have to press `q` to leave the documentation viewer.
{{% /notice %}}

## Write an Ansible Playbook

We want to use the SELinux module to make sure it is configured in enforcing mode. SELinux is a kernel feature which brings extra security to our Linux system and it is highly recommended to always keep it enabled and in enforcing mode. If you're new to SELinux, there is a nice article on [What is SELinux](https://www.redhat.com/en/topics/linux/what-is-selinux) to get you started.

Let's write a simple playbook which enables SELinux and sets it to enforcing mode on the local machine. In this lab you can use the visual **VS Code** editor or run an editor of your choice from the terminal. Create the Playbook `enforce-selinux.yml` with the following content:

```
---
- name: set SELinux to enforcing
  hosts: localhost
  become: yes
  tasks:
  - name: set SElinux to enforcing
    ansible.posix.selinux:
      policy: targeted
      state: enforcing
```

Make sure you save the playbook as `enforce-selinux.yml` in your student users home directory.

{{% notice note %}}
Pay special attention to the module name. Typically you would see something like `selinux`, but since we are using a module provided by an Ansible Collection, we have to specify the fully qualified collection name.
{{% /notice %}}

## Test the playbook

Noe let's run the Playbook and see what happens:

    [{{< param "control_prompt" >}} ~]$ ansible-playbook enforce-selinux.yml

You should see output like this:

    [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

    PLAY [set SELinux to enforcing] ***********************************************************************************

    TASK [Gathering Facts] ********************************************************************************************
    ok: [localhost]

    TASK [set SElinux to enforcing] ***********************************************************************************
    ok: [localhost]

    PLAY RECAP ********************************************************************************************************
    localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

If SELinux was not set to enforcing mode before, you might see "changed" instead of "ok". If it did say "changed" and you run it a second time, you should now see "ok" - the magic of [Ansible idempotency](https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html).

## Simplify the namespace

If you use many modules from Ansible Collections in your Playbook, the \<author>.\<collection> prefix can become quite annoying and reading your Playbook can become harder as well.

You can use the `collections` keyword to skip defining the namespace with every task. In your terminal edit the Playbook `enforce-selinux.yml` to look like this, basically adding the `collections:` section and changing the module name from FQCN to the simple module name:

```
---
- name: set SELinux to enforcing
  hosts: localhost
  become: yes
  collections:
  - ansible.posix
  tasks:
  - name: set SElinux to enforcing
    selinux:
      policy: targeted
      state: enforcing
```

{{% notice tip %}}
Although the syntax looks similar to how you specify roles, this works different. They keyword `roles` will execute the `tasks/main.yml` in each role. The `collections` keyword is merely a shortcut so you can skip the author and namespace every time you use a module in a task.
{{% /notice %}}

## Test the change

Now run the Playbook again, you shouldn't see any difference in the output. As explained before, the `collections` keyword only simplifies writing your Playbook!

{{% notice warning %}}
We are explaining the `collections` keyword here for completeness. It is however recommended to always use the fully qualified collection name. The internal lookup can deliver unexpected results if there are many overlapping or overriding module names, which can be avoided by always using the full name. Since most modern code editors provide auto completion, it's not too much of an issue when typing the code either.
{{% /notice %}}
