+++
title = "Using Variables"
weight = 4
+++

Previous exercises showed you the basics of Ansible Engine.  In the next few exercises, we are going to teach some more advanced Ansible skills that will add flexibility and power to your playbooks.

Ansible exists to make tasks simple and repeatable.  We also know that not all systems are exactly alike and often require some slight change to the way an Ansible playbook is run. Enter variables.

Ansible supports variables to store values that can be used in Playbooks. Variables can be defined in a variety of places and have a clear precedence. Ansible substitutes the variable with its value when a task is executed.

Variables are referenced in Playbooks by placing the variable name in double curly braces:

```
Here comes a variable {{ variable1 }}
```

Variables and their values can be defined in various places: the inventory, additional files, on the command line, etc.

The recommended practice to provide variables in the inventory is to define them in files located in two directories named `host_vars` and `group_vars`:

- To define variables for a group `servers`, a YAML file named `group_vars/servers` with the variable definitions is created.

- To define variables specifically for a host `node1`, the file `host_vars/node1` with the variable definitions is created.

{{% notice tip %}}
Host variables take precedence over group variables (more about precedence can be found in the [docs](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable)).
{{% /notice %}}

## Create Variable Files

For understanding and practice let’s do a lab. Following up on the theme "Let’s build a webserver. Or two. Or even more…​", you will change the `index.html` to show the development environment (dev/prod) a server is deployed in.

On the ansible control host, as the `student` user, create the directories to hold the variable definitions in `~/ansible-files/`:

```bash
[{{< param "control_prompt" >}} ansible-files]$ mkdir host_vars group_vars
```

Now create two files containing variable definitions. We’ll define a variable named `stage` which will point to different environments, `dev` or `prod`:

- Create the file `~/ansible-files/group_vars/web` with this content:

```
---
stage: dev
```

- Create the file `~/ansible-files/host_vars/node2` with this content:

```
---
stage: prod
```

What is this about?

- For all servers in the `web` group the variable `stage` with value `dev` is defined. So as default we flag them as members of the dev environment.

- For server `node2` this is overridden and the host is flagged as a production server.

## Create index.html Files

Now create two files in `~/ansible-files/`:

One called `prod_index.html` with the following content:

```html
<body>
<h1>This is a production webserver, take care!</h1>
</body>
```

And the other called `dev_index.html` with the following content:

```html
<body>
<h1>This is a development webserver, have fun!</h1>
</body>
```

## Create the Playbook

Now you need a Playbook that copies the prod or dev `index.html` file - according to the "stage" variable.

Create a new Playbook called `deploy_index_html.yml` in the `~/ansible-files/` directory.

{{% notice tip %}}
Note how the variable "stage" is used in the name of the file to copy.
{{% /notice %}}

```
---
- name: Copy index.html
  hosts: web
  become: yes
  tasks:
  - name: copy index.html
    ansible.builtin.copy:
      src: ~/ansible-files/{{ stage }}_index.html
      dest: /var/www/html/index.html
```

- Run the Playbook:

```bash
[{{< param "control_prompt" >}} ansible-files]$ ansible-playbook deploy_index_html.yml
```

## Test the Result

The Playbook should copy different files as index.html to the hosts, use `curl` to test it. Check the inventory again if you forgot the IP addresses of your nodes.

```bash
[{{< param "control_prompt" >}} ansible-files]$ grep node ~/lab_inventory/hosts
node1 ansible_host=11.22.33.44
node2 ansible_host=22.33.44.55
node3 ansible_host=33.44.55.66
[{{< param "control_prompt" >}} ansible-files]$ curl http://11.22.33.44
<body>
<h1>This is a development webserver, have fun!</h1>
</body>
[{{< param "control_prompt" >}} ansible-files]$ curl http://22.33.44.55
<body>
<h1>This is a production webserver, take care!</h1>
</body>
[{{< param "control_prompt" >}} ansible-files]$ curl http://33.44.55.66
<body>
<h1>This is a development webserver, have fun!</h1>
</body>
```

{{% notice tip %}}
If by now you think: There has to be a smarter way to change content in files…​ you are absolutely right. This lab was done to introduce variables, you are about to learn about templates in one of the next chapters.
{{% /notice %}}

## Ansible Facts

Ansible facts are variables that are automatically discovered by Ansible from a managed host. Remember the "Gathering Facts" task listed in the output of each `ansible-playbook` execution? At that moment the facts are gathered for each managed node. Facts can also be pulled by the `setup` module. They contain useful information stored into variables that administrators can reuse.

To get an idea what facts Ansible collects by default, on your control node as your student user run:

```bash
[{{< param "control_prompt" >}} ansible-files]$ ansible node1 -m setup
```

This might be a bit too much, you can use filters to limit the output to certain facts, the expression is shell-style wildcard:

```bash
[{{< param "control_prompt" >}} ansible-files]$ ansible node1 -m setup -a 'filter=ansible_eth0'
```

Or what about only looking for memory related facts:

```bash
[{{< param "control_prompt" >}} ansible-files]$ ansible node1 -m setup -a 'filter=ansible_*_mb'
```

## Challenge Lab: Facts

- Try to find and print the distribution (Red Hat) of your managed hosts. On one line, please.

{{% notice tip %}}
Use grep to find the fact, then apply a filter to only print this fact.
{{% /notice %}}

<details><summary><b>Click here for Solution</b></summary>
<hr/>
<p>

```bash
[{{< param "control_prompt" >}} ansible-files]$ ansible node1 -m setup|grep distribution
[{{< param "control_prompt" >}} ansible-files]$ ansible node1 -m setup -a 'filter=ansible_distribution' -o
```

</p>
<hr/>
</details>

## Using Facts in Playbooks

Facts can be used in a Playbook like variables, using the proper naming, of course. Create this Playbook as `facts.yml` in the `~/ansible-files/` directory:

```
---
- name: Output facts within a playbook
  hosts: all
  tasks:
  - name: Prints Ansible facts
    ansible.builtin.debug:
      msg: The default IPv4 address of {{ ansible_fqdn }} is {{ ansible_default_ipv4.address }}
```

{{% notice tip %}}
The "debug" module is handy for e.g. debugging variables or expressions.
{{% /notice %}}

Execute it to see how the facts are printed:

```bash
[{{< param "control_prompt" >}} ansible-files]$ ansible-playbook facts.yml

PLAY [Output facts within a playbook] ******************************************

TASK [Gathering Facts] *********************************************************
ok: [node3]
ok: [node2]
ok: [node1]
ok: [ansible]

TASK [Prints Ansible facts] ****************************************************
ok: [node1] =>
  msg: The default IPv4 address of node1 is 172.16.190.143
ok: [node2] =>
  msg: The default IPv4 address of node2 is 172.16.30.170
ok: [node3] =>
  msg: The default IPv4 address of node3 is 172.16.140.196
ok: [ansible] =>
  msg: The default IPv4 address of ansible is 172.16.2.10

PLAY RECAP *********************************************************************
ansible                    : ok=2    changed=0    unreachable=0    failed=0
node1                      : ok=2    changed=0    unreachable=0    failed=0
node2                      : ok=2    changed=0    unreachable=0    failed=0
node3                      : ok=2    changed=0    unreachable=0    failed=0
```
