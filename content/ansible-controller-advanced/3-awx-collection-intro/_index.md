+++
title = "There is more to automation controller than the Web UI"
weight = 3
+++

This is an advanced automation controller lab so we don’t really want you to use the web UI for everything. To fully embrace automation and adopt the [infrastructure as code](https://en.wikipedia.org/wiki/Infrastructure_as_code) methodology, we want to use Ansible to configure our automation controller cluster.

Since automation controller is exposing all of its functionality via REST API, we can automate everything. Instead of using the API directly, it is highly recommended to use the [AWX](https://github.com/ansible/awx/tree/devel/awx_collection) or [automation controller](https://cloud.redhat.com/ansible/automation-hub/repo/published/ansible/controller) Ansible Collection (the second link will only work for you, if you have an active Red Hat Ansible Automation Platform Subscription) to setup, configure and maintain your Red Hat Ansible Automation Platform Cluster.

{{% notice info %}}
For the purpose of this lab, we will use the community AWX collection. Red Hat Customers will prefer the supported Ansible automation controller collection. Since this requires an active subscription and we want to make the lab usable for everyone, we will stick with the AWX collection for the purpose of the lab.
{{% /notice %}}

First, we need to install the AWX Collection. Installing Ansible Collections is super easy:

```bash
[{{< param "control_prompt" >}} ~]$ ansible-galaxy collection install awx.awx:19.1.0
```

{{% notice note %}}
The AWX collection is updated very often. To make sure the following lab instructions will work for you, we install specifically version 19.1.0. We regularly update these instruction to keep up to date and don't fall behind too much. However, to make sure the lab works for you, we suggest you to use the specified version.
{{% /notice %}}

## Authentication

Before we can do any changes on our automation controller, we have to authenticate our user. There are several methods available to provide authentication details to the modules. In this lab, we want to use environment variables.

```bash
# the Base URI of our automation controller cluster node
[{{< param "control_prompt" >}} ~]$ export CONTROLLER_HOST=https://{{< param "external_controller1" >}}
# the user name
[{{< param "control_prompt" >}} ~]$ export CONTROLLER_USERNAME=admin
# and the password
[{{< param "control_prompt" >}} ~]$ export CONTROLLER_PASSWORD='{{< param "secret_password" >}}'
# do not verify the SSL certificate, in production, you will use proper SSL certificates and not need this option or set it to True
[{{< param "control_prompt" >}} ~]$ export CONTROLLER_VERIFY_SSL=false
```

{{% notice warning %}}
Make sure you define the environment variables in the same shell you want to later run your Ansible Playbook from, otherwise the Playbook will fail due to authentication errors.
{{% /notice %}}

## Create an inventory

Let's start with a very simple example to see how things work.

```yaml
---
- name: Configure automation controller
  hosts: localhost
  become: false
  gather_facts: false
  tasks:
  - name: Create an inventory
    awx.awx.inventory:
      name: AWX inventory
      organization: Default
```

Since we are calling the REST API of automation controller, the Ansible Playbook is running on **localhost**, but the module will connect to the URL provided by the **TOWER_HOST** environment variable.

{{% notice info %}}
You might see a warning message "You are using the awx version of this collection but connecting to Red Hat Ansible Automation Platform". This can be ignored. As said above, we intentionally use the AWX Community Collection for the purpose of the lab. As a Red Hat Customer you would probably prefer the supported Ansible Collection instead.
{{% /notice %}}

Run and test your playbook and verify everything works as expected, by logging ino the automation controller Web UI.

## Add hosts to inventory

Now that we have the empty inventory created, add your two managed hosts using their internal hostnames **`{{< param "internal_host1" >}}`** and **`{{< param "internal_host2" >}}`**, again using the AWX Ansible Collection. The module to add hosts to an inventory is called **awx.awx.host**. Read the documentation and try to figure out how to add the necessary tasks to the Ansible Playbook.

{{% notice info %}}
Use **ansible-doc awx.awx.host** to open the documentation for the specified module.
{{% /notice %}}

<details><summary><b>Click here for Solution</b></summary>
<hr/>
<p>

```yaml
---
- name: Configure automation controller
  hosts: localhost
  become: false
  gather_facts: false
  tasks:
  - name: Create an inventory
    awx.awx.inventory:
      name: AWX inventory
      organization: Default
  - name: Add hosts to inventory
    awx.awx.host:
      name: "{{  item }}"
      inventory: AWX inventory
      state: present
    loop:
      - {{< param "internal_host1" >}}
      - {{< param "internal_host2" >}}
```

</p>
<hr/>
</details>

Run and test your playbook and verify everything works as expected, by logging ino the automation controller Web UI.

## Create Machine Credentials

{{% notice tip %}}
SSH keys have already been created and distributed in your lab environment and `sudo` has been setup on the managed hosts to allow password-less login. When you SSH into a host as user **student{{< param "student" >}}** from **{{< param "internal_control" >}}** you will become user **ec2-user** on the host you logged in.
{{% /notice %}}

Now we want to configure these credentials to access our managed hosts from automation controller. Your private key is stored in `~/.ssh/aws-private.pem` and already configured on the remote machine. Try to find the necessary attributes in the **awx.awx.credential** module documentation.

<details><summary><b>Click here for Solution</b></summary>
<hr/>
<p>

```yaml
---
- name: Configure automation controller
  hosts: localhost
  become: false
  gather_facts: false
  tasks:
  - name: Create an inventory
    awx.awx.inventory:
      name: AWX inventory
      organization: Default
  - name: Add hosts to inventory
    awx.awx.host:
      name: "{{  item }}"
      inventory: AWX inventory
      state: present
    loop:
      - {{< param "internal_host1" >}}
      - {{< param "internal_host2" >}}
  - name: Machine Credentials
    awx.awx.credential:
      name: AWX Credentials
      kind: ssh
      organization: Default
      inputs:
        username: ec2-user
        ssh_key_data: "{{ lookup('file', '~/.ssh/aws-private.pem' ) }}"
```

</p>
<hr/>
</details>

{{% notice info %}}
If you run this Ansible Playbook multiple times, you will notice the **awx.awx.credential** module is not idempotent! Since we store the SSH key encrypted, the Ansible Module is unable to verify it has already been set and didn't change. This is what we want and expect from a secure system, but it also means Ansible has no means to verify it and hence overrides the SSH key or password every time the Ansible Playbook is executed.
{{% /notice %}}

Run and test your playbook and verify everything works as expected, by logging ino the automation controller Web UI.

## Create Projects

The Ansible content used in this lab is hosted on Github in the project [https://github.com/goetzrieger/ansible-labs-playbooks.git](https://github.com/goetzrieger/ansible-labs-playbooks.git). The next step is to add a project to import the Ansible Playbooks. As before, try to figure out the necessary parameters by reading the documentation of the **awx.awx.project** module documentation.

<details><summary><b>Click here for Solution</b></summary>
<hr/>
<p>

```yaml
---
- name: Configure automation controller
  hosts: localhost
  become: false
  gather_facts: false
  tasks:
  - name: Create an inventory
    awx.awx.inventory:
      name: AWX inventory
      organization: Default
  - name: Add hosts to inventory
    awx.awx.host:
      name: "{{  item }}"
      inventory: AWX inventory
      state: present
    loop:
      - {{< param "internal_host1" >}}
      - {{< param "internal_host2" >}}
  - name: Machine Credentials
    awx.awx.credential:
      name: AWX Credentials
      kind: ssh
      organization: Default
      inputs:
        username: ec2-user
        ssh_key_data: "{{ lookup('file', '~/.ssh/aws-private.pem' ) }}"
  - name: AWX Project
    awx.awx.project:
      name: AWX Project
      organization: Default
      state: present
      scm_update_on_launch: True
      scm_delete_on_update: True
      scm_type: git
      scm_url: https://github.com/goetzrieger/ansible-labs-playbooks.git
```

</p>
<hr/>
</details>

Run and test your playbook and verify everything works as expected, by logging ino the automation controller Web UI.

If you wonder why **awx.awx.credential** is not idempotent, read the info box in the [previous chapter](#create-machine-credentials).

## Create Job Templates

Before running an Ansible **Job** from your automation controller cluster you must create a **Job Template**, again business as usual for automation controller users. For this part of the Ansible Playbook, we will use the **awx.awx.job_template** module. The name of the Ansible Playbook you want run is `apache_install.yml`.

<details><summary><b>Click here for Solution</b></summary>
<hr/>
<p>

```yaml
---
- name: Configure automation controller
  hosts: localhost
  become: false
  gather_facts: false
  tasks:
  - name: Create an inventory
    awx.awx.inventory:
      name: AWX inventory
      organization: Default
  - name: Add hosts to inventory
    awx.awx.host:
      name: "{{  item }}"
      inventory: AWX inventory
      state: present
    loop:
      - {{< param "internal_host1" >}}
      - {{< param "internal_host2" >}}
  - name: Machine Credentials
    awx.awx.credential:
      name: AWX Credentials
      kind: ssh
      organization: Default
      inputs:
        username: ec2-user
        ssh_key_data: "{{ lookup('file', '~/.ssh/aws-private.pem' ) }}"
  - name: AWX Project
    awx.awx.project:
      name: AWX Project
      organization: Default
      state: present
      scm_update_on_launch: True
      scm_delete_on_update: True
      scm_type: git
      scm_url: https://github.com/goetzrieger/ansible-labs-playbooks.git
  - name: AWX Job Template
    awx.awx.job_template:
      name: Install Apache
      organization: Default
      state: present
      inventory: AWX inventory
      become_enabled: True
      playbook: apache_install.yml
      project: AWX Project
      credential: AWX Credentials

```

</p>
<hr/>
</details>

Run and test your playbook and verify everything works as expected, by logging ino the automation controller Web UI.

## Verify the cluster

We are working in a clustered environment. To verify that the resources were created on all instances properly, login to the other controller nodes web UI's.

Have a look around, everything we automatically configured on one controller instance with our Ansible Playbook was **synchronized automatically** to the other nodes. Inventory, credentials, projects, templates, all there.

## Challenge Labs

Try to add the necessary tasks to your Ansible Playbook to run the **Job Template** you created.

<details><summary><b>Click here for Solution</b></summary>
<hr/>
<p>

This is a Challenge Lab! No solution here. If you don't know where to look, check out the documentation of the **awx.awx.job_template** module.

</p>
<hr/>
</details>
