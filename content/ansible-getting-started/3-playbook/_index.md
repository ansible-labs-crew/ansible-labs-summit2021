+++
title = "Writing Your First Playbook"
weight = 3
+++

While Ansible ad hoc commands are useful for simple operations, they are not suited for complex configuration management or orchestration scenarios. For such use cases **Playbooks** are the way to go.

Playbooks are files which describe the desired configurations or steps to implement on managed hosts. Playbooks can change lengthy, complex administrative tasks into easily repeatable routines with predictable and successful outcomes.

A Playbook is where you can take some of those ad hoc commands you just ran and put them into a repeatable set of **Plays** and **Tasks**.

A Playbook can have multiple Plays and a Play can have one or multiple Tasks. In a Task a **Module** is called, like the Modules in the previous chapter.

- The goal of a Play is to map a group of hosts to a list of Tasks.
- The goal of a Task is to implement Modules against those hosts.

{{% notice tip %}}
Here is a nice analogy: When Ansible Modules are the tools in your workshop, the Inventory is the list of materials and the Playbooks are the instructions.
{{% /notice %}}

## What are Collections and why should I care?

Ansible Collections are a new distribution format for Ansible content that can include playbooks, roles, modules, and plugins. Ansible Collection names are a combination of two components. The first part is the name of the author who wrote and maintains the Ansible Collection. The second part is the name of the Ansible Collection. This allows one author to have multiple Collections. It also allows multiple authors to have Ansible Collections with the same name.

    <author>.<collection>

These are examples for Ansible Collection names:

- ansible.posix

- geerlingguy.k8s

- theforeman.foreman

To identify a specific module in an Ansible Collection, we add the name of it as the third part:

    <author>.<collection>.<module>

Valid examples for a fully qualified Ansible Collection Name (FQCN):

- ansible.posix.selinux

- geerlingguy.k8s.kubernetes

- theforeman.foreman.user

{{% notice info %}}
Many modules and plugins are part of the "ansible.builtin" collection and are shipped with Ansible and installed automatically. Although not mandatory, it is highly recommended to also use the FQCN for builtin modules, to avoid name clashes or unpredictable behavior. This is why all following examples, will use "ansible.builtin" as part of the FQCN.
{{% /notice %}}

## Playbook Basics

Playbooks are text files written in YAML format and therefore need:

- to start with three dashes (`---`)

- proper indentation using spaces and **not** tabs\!

There are some important concepts:

- **hosts**: the managed hosts to perform the tasks on

- **tasks**: the operations to be performed by invoking Ansible modules and passing them the necessary options.

- **become**: privilege escalation in Playbooks, same as using `-b` in the ad hoc command.

{{% notice tip %}}
The ordering of the contents within a Playbook is important, because Ansible executes plays and tasks in the order they are presented.
{{% /notice %}}

A Playbook should be **idempotent**, so if a Playbook is run once to put the hosts in the correct state, it should be safe to run it a second time and it should make no further changes to the hosts.

{{% notice tip %}}
Most Ansible modules are idempotent, so it is relatively easy to ensure this is true.
{{% /notice %}}

## Creating a Directory Structure and File for your Playbook

Enough theory, it’s time to create your first Playbook. In this lab you create a Playbook to set up an Apache webserver in three steps:

- First step: Install httpd package

- Second step: Enable/start httpd service

- Third step: Create an index.html file

This Playbook makes sure the package containing the Apache webserver is installed on `node1`.

There is a [best practice](http://docs.ansible.com/ansible/playbooks_best_practices.html) on the preferred directory structures for playbooks.  We strongly encourage you to read and understand these practices as you develop your Ansible ninja skills.  That said, our playbook today is very basic and creating a complex structure will just confuse things.

Instead, we are going to create a very simple directory structure for our playbook, and add just a couple of files to it.

In your browser, bring up your **code-server** terminal (you opened one in the first section) and create a directory called `ansible-files` in your home directory and change into it:

```bash
[{{< param "control_prompt" >}} ~]$ mkdir ansible-files
[{{< param "control_prompt" >}} ~]$ cd ansible-files/
```

Now use **code-server** to add a file called `apache.yml` with the following content.

{{% notice tip %}}
If you are unsure how to use **code-server** (basically like VSCode), have a quick look at the [Visual Studio Code Server introduction](../../vscode-intro/)
{{% /notice %}}

```
---
- name: Apache server installed
  hosts: node1
  become: yes
```

This shows one of Ansible’s strengths: The Playbook syntax is easy to read and understand. In this Playbook:

- A name is given for the play via `name:`.

- The host to run the playbook against is defined via `hosts:`.

- We enable user privilege escalation with `become:`.

{{% notice tip %}}
You obviously need to use privilege escalation to install a package or run any other task that requires root permissions. This is done in the Playbook by `become: yes`.
{{% /notice %}}

Now that we've defined the play, let's add a task to get something done. We will add a task in which yum will ensure that the Apache package is installed in the latest version. Modify the file so that it looks like the following listing using the **code-server** editor:

```
---
- name: Apache server installed
  hosts: node1
  become: yes
  tasks:
  - name: latest Apache version installed
    ansible.builtin.yum:
      name: httpd
      state: latest
```

{{% notice tip %}}
Since playbooks are written in YAML, alignment of the lines and keywords is crucial. Make sure to vertically align the *t* in `task` with the *b* in `become`. Once you are more familiar with Ansible, make sure to take some time and study a bit the [YAML Syntax](http://docs.ansible.com/ansible/YAMLSyntax.html).
{{% /notice %}}

In the added lines:

- We started the tasks part with the keyword `tasks:`.

- A task is named and the module for the task is referenced. Here it uses the `yum` module.

- Parameters for the module are added:

  - `name:` to identify the package name

  - `state:` to define the wanted state of the package

{{% notice tip %}}
The module parameters are individual to each module. If in doubt, look them up again with `ansible-doc`.
{{% /notice %}}

Save your playbook.

## Running the Playbook

Playbooks are executed using the `ansible-playbook` command on the control node. Before you run a new Playbook it’s a good idea to check for syntax errors. Head over to the **code-server** terminal and run:

```bash
[{{< param "control_prompt" >}} ansible-files]$ ansible-playbook --syntax-check apache.yml
```

Now you should be ready to run your Playbook:

```bash
[{{< param "control_prompt" >}} ansible-files]$ ansible-playbook apache.yml
```

The output should not report any errors but provide an overview of the tasks executed and a **Play Recap** summarizing what has been done. There is also a task called **Gathering Facts** listed: this is a built-in task that runs automatically at the beginning of each **Play**. It collects information about the managed nodes. Exercises later on will cover this in more detail.

Use SSH to make sure Apache has been installed on `node1`. The necessary IP address is provided in the inventory. Grep for the IP address there and use it to SSH to the node.

```bash
[{{< param "control_prompt" >}} ansible-files]$ grep node1 ~/lab_inventory/hosts
node1 ansible_host=11.22.33.44
[{{< param "control_prompt" >}} ansible-files]$ ssh 11.22.33.44
Last login: Wed May 15 14:03:45 2019 from 44.55.66.77
Managed by Ansible
[...]
[ec2-user@node1 ~]$ rpm -qi httpd
Name        : httpd
Version     : 2.4.6
[...]
```

Log out of `node1` with the command `exit` so that you are back on the control host, and verify the installed package with an Ansible ad hoc command\!

```bash
[{{< param "control_prompt" >}} ansible-files]$ ansible node1 -m command -a 'rpm -qi httpd'
```

Run the Playbook a second time, and compare the output: The output changed from **changed** to **ok**, and the color changed from yellow to green. Also the **PLAY RECAP** output is different now. This make it easy to spot what Ansible actually did.

## Extend your Playbook: Start & Enable Apache

The next part of the Playbook makes sure the Apache webserver is enabled and started on `node1`.

On the control host, as your student user, edit the file `~/ansible-files/apache.yml` again to add a second task using the `service` module. The Playbook should now look like this:

```
---
- name: Apache server installed
  hosts: node1
  become: yes
  tasks:
  - name: latest Apache version installed
    ansible.builtin.yum:
      name: httpd
      state: latest
  - name: Apache enabled and running
    ansible.builtin.service:
      name: httpd
      enabled: true
      state: started
```

Again: what these lines do is easy to understand:

- a second task is created and named

- a module is specified (`service`)

- parameters for the module are supplied

With the second task we make sure the Apache server is indeed running on the target machine. Run your extended Playbook:

```bash
[{{< param "control_prompt" >}} ansible-files]$ ansible-playbook apache.yml
```

Note the output now: Some tasks are shown as **ok** in green and one is shown as **changed** in yellow.

- Use an Ansible ad hoc command again to make sure Apache has been enabled and started, e.g. with: `systemctl status httpd`.

- Run the Playbook a second time to get used to the change in the output.

## Extend your Playbook: Create an index.html

Check that the tasks were executed correctly and Apache is accepting connections: Make an HTTP request using Ansible’s `uri` module in an ad hoc command from the control node. Make sure to replace the **IP** with the IP for the node from the inventory.

{{% notice warning %}}
Expect a lot of red lines and a 403 status
{{% /notice %}}

```bash
[{{< param "control_prompt" >}} ansible-files]$ ansible localhost -m uri -a "url=http://<IP>"
```

There are a lot of red lines and an error: As long as there is not at least an `index.html` file to be served by Apache, it will throw an ugly **HTTP Error 403: Forbidden** status and Ansible will report an error.

So why not use Ansible to deploy a simple `index.html` file? Create the file `~/ansible-files/index.html` on the control node:

```html
<body>
<h1>Apache is running fine</h1>
</body>
```

You already used Ansible’s `copy` module to write text supplied on the command line into a file. Now you’ll use the module in your Playbook to actually copy a file:

On the control node as your student user edit the file `~/ansible-files/apache.yml` and add a new task utilizing the `copy` module. It should now look like this:

```
---
- name: Apache server installed
  hosts: node1
  become: yes
  tasks:
  - name: latest Apache version installed
    ansible.builtin.yum:
      name: httpd
      state: latest
  - name: Apache enabled and running
    ansible.builtin.service:
      name: httpd
      enabled: true
      state: started
  - name: copy index.html
    ansible.builtin.copy:
      src: ~/ansible-files/index.html
      dest: /var/www/html/
```

You are getting used to the Playbook syntax, so what happens? The new task uses the `copy` module and defines the source and destination options for the copy operation as parameters.

Run your extended Playbook:

```bash
[{{< param "control_prompt" >}} ansible-files]$ ansible-playbook apache.yml
```

- Have a good look at the output

- Run the ad hoc command  using the "uri" module from further above again to test Apache: The command should now return a friendly green "status: 200" line, amongst other information.

## Practice: Apply to Multiple Host

This was nice but the real power of Ansible is to apply the same set of tasks reliably to many hosts.

- So what about changing the apache.yml Playbook to run on `node1` **and** `node2` **and** `node3`?

As you might remember, the inventory lists all nodes as members of the group `web`:

```ini
[web]
node1 ansible_host=11.22.33.44
node2 ansible_host=22.33.44.55
node3 ansible_host=33.44.55.66
```

{{% notice tip %}}
The IP addresses shown here are just examples, your nodes will have different IP addresses.
{{% /notice %}}

Change the Playbook to point to the group `web`:

```
---
- name: Apache server installed
  hosts: web
  become: yes
  tasks:
  - name: latest Apache version installed
    ansible.builtin.yum:
      name: httpd
      state: latest
  - name: Apache enabled and running
    ansible.builtin.service:
      name: httpd
      enabled: true
      state: started
  - name: copy index.html
    ansible.builtin.copy:
      src: ~/ansible-files/index.html
      dest: /var/www/html/
```

Now run the Playbook:

```bash
[{{< param "control_prompt" >}} ansible-files]$ ansible-playbook apache.yml
```

Finally check if Apache is now running on all servers. Identify the IP addresses of the nodes in your inventory first, and afterwards use them each in the ad hoc command with the uri module as we already did with the `node1` above. All output should be green.
