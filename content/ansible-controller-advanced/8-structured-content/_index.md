+++
title = "Well Structured Content Repositories"
weight = 8
+++

## OPTIONAL EXERCISE

It’s a common part of the learning curve for Ansible and automation controller: At some point you will have written so many playbooks that a need for structure comes up. Where to put the Playbooks, what about the Templates, Files and so on.

The main recommendations are:

- Put your content in a version control system like Git or SVN. This comes naturally since Ansible code is in text form anyway, and thus can be managed easily.

- Group your code by logical units, called [roles](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html) in Ansible.

  - Example: have all code, config templates and files for the apache web server in one role, and all code, configuration and sql statements for the database in another role. That way the code becomes much better to read and handle, and roles can be made re-usable and shared between projects, teams or with the global community.

Of course, which structure works best in the end depends on the individual requirements, but we will highlight some common ground rules which apply to almost all use cases.

The first recommendation is to separate *specific code* from *reusable/generic code* from *data*:

- specific code: Playbooks and their direct dependencies which are not shared outside the realm of the project or team.

- generic code: All content that will be used across multiple projects.

- data: This is mostly the inventory or the inventory scripts and the corresponding variables for hosts and groups. In many use cases it is advisable to have a dedicated inventory for each life-cycle environment.

{{% notice tip %}}
Data content files can be in the same Git repository, each in its own directory (e.g. dev, test, qa, prod). Alternatively, for example in larger environments or with dedicated teams per environment there can be one Git repository for each environment. We recommend to put special focus on [splitting out host and group data](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#splitting-out-host-and-group-specific-data).
{{% /notice %}}

{{% notice warning %}}
Be careful to *not* have separate code repositories for each environment. It would go against the purpose of testing the *same* code as you push it through your life-cycle, only varying the data / inventory. If you have difficulties to keep the same code throughout all your environments we recommend to re-think the structure of our code and what you put into your inventory.
{{% /notice %}}

## Example repository

So, let’s get started with an example. The content and repo-structure in this lab is mostly aligned to the [Ansible best practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#content-organization) and is explained in more detail there (we've had to simplify a bit for the lab).

Since we want to store all content in a repository, we have to create a simplistic Git server on our control host. In a more typical environment, you would work with GitLab, Gitea, or any other commercial Git server.

```
[{{< param "pre_mng_prompt" >}} ~]$ wget https://raw.githubusercontent.com/ansible-labs-crew/playbooks_adv_summit2021/master/simple_git.yml
[{{< param "pre_mng_prompt" >}} ~]$ ansible-playbook simple_git.yml
```

Next we will clone the repository on the control host. To enable you to work with git on the command line the SSH key for user *{{< param "user_name" >}}* was already added to the Git user *git*. Next, clone the repository on the control machine:

    [{{< param "pre_mng_prompt" >}} ~]$ git clone {{< param "git_user" >}}@{{< param "internal_control" >}}:{{< param "content_git_uri" >}}
    # Message "warning: You appear to have cloned an empty repository." is OK and can be ignored
    [{{< param "pre_mng_prompt" >}} ~]$ git config --global push.default simple
    [{{< param "pre_mng_prompt" >}} ~]$ git config --global user.name "Your Name"
    [{{< param "pre_mng_prompt" >}} ~]$ git config --global user.email you@example.com
    [{{< param "pre_mng_prompt" >}} ~]$ cd structured-content/

{{% notice tip %}}
The repository is currently empty. The three config commands are just there to avoid warnings from Git.
{{% /notice %}}

You are now going to add some default directories and files:

    [{{< param "pre_mng_prompt" >}} structured-content]$ touch {staging,production}

This command creates two inventory files: in this case we have different stages with different hosts which we keep in separate inventory files. Note that those files are right now still empty and need to be filled with content to work properly.

In the current setup we have two instances. Let’s assume that `{{< param "internal_host1" >}}` is part of the staging environment, and `{{< param "internal_host2" >}}` is part of the production environment. To reflect that in the inventory files, edit the two empty inventory files to look like this:

    [{{< param "pre_mng_prompt" >}} structured-content]$ cat staging
    [staging]
    {{< param "internal_host1" >}}

    [{{< param "pre_mng_prompt" >}} structured-content]$ cat production
    [production]
    {{< param "internal_host2" >}}

Next we add some directories:

- directories for host and group variables

- A **roles** directory where the main part of our automation logic
  will be in.

- For demonstration purpose we also will add a **library** directory: it can contain Ansible code related to a project like custom modules, plugins, etc.

```bash
[{{< param "pre_mng_prompt" >}} structured-content]$ mkdir -p {group_vars,host_vars,library,roles}
```

Now let's add two roles we’ll later use in this example. First we’ll create a structure where we’ll add content later. This can easily be achieved with the command `ansible-galaxy`: it creates **role skeletons** with all appropriate files and directories already in place.

```bash
[{{< param "pre_mng_prompt" >}} structured-content]$ ansible-galaxy init --offline --init-path=roles security
[{{< param "pre_mng_prompt" >}} structured-content]$ ansible-galaxy init --offline --init-path=roles apache
```

{{% notice tip %}}
Even if a good role is generally self-explanatory, it still makes sense to have proper documentation. The right location to document roles are the files **meta/main.yml** and **README.md**.
{{% /notice %}}

The roles are empty, so we need to add a few tasks to each. In the last chapters we set up an Apache webserver and used some security tasks. Let’s add that code to our roles by editing the two task files:

    [{{< param "pre_mng_prompt" >}} structured-content]$ cat roles/apache/tasks/main.yml
    ---
    # tasks file for apache
    - name: latest Apache version installed
      yum:
        name: httpd
        state: latest
    - name: latest firewalld version installed
      yum:
        name: firewalld
        state: latest
    - name: firewalld enabled and running
      service:
        name: firewalld
        enabled: true
        state: started
    - name: firewalld permits http service
      firewalld:
        service: http
        permanent: true
        state: enabled
        immediate: yes
    - name: Apache enabled and running
      service:
        name: httpd
        enabled: true
        state: started

    [{{< param "pre_mng_prompt" >}} structured-content]$ cat roles/security/tasks/main.yml
    ---
    # tasks file for security
    - name: "HIGH | RHEL-07-010290 | PATCH | The Red Hat Enterprise Linux operating system must not have accounts configured with blank or null passwords."
      replace:
        dest: "{{ item }}"
        follow: true
        regexp: 'nullok ?'
      with_items:
        - /etc/pam.d/system-auth
        - /etc/pam.d/password-auth

    - name: "MEDIUM | RHEL-07-010210 | PATCH | The Red Hat Enterprise Linux operating system must be configured to use the shadow file to store only encrypted representations of passwords."
      lineinfile:
        dest: /etc/login.defs
        regexp: ^#?ENCRYPT_METHOD
        line: "ENCRYPT_METHOD SHA512"

    - name: "SCORED | 1.1.1.2 | PATCH | Remove freevxfs module"
      modprobe:
        name: freevxfs
        state: absent

We also need to create a playbook to call the roles from. This is often call `site.yml`, since it keeps the main code for the setup of our environment. Create the file:

    [{{< param "pre_mng_prompt" >}} structured-content]$ cat site.yml
    ---
    - name: Execute apache and security roles
      hosts: all

      roles:
        - { role: apache }
        - { role: security }

So we have prepared a basic structure for quite some content - call `tree` to look at it.

<details><summary><b>Click here for Solution</b></summary>
<hr/>
<p>

``` bash
[{{< param "control_prompt" >}} structured-content]$ tree
.
├── group_vars
├── host_vars
├── library
├── production
├── roles
│   ├── apache
│   │   ├── defaults
│   │   │   └── main.yml
│   │   ├── files
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── meta
│   │   │   └── main.yml
│   │   ├── README.md
│   │   ├── tasks
│   │   │   └── main.yml
│   │   ├── templates
│   │   ├── tests
│   │   │   ├── inventory
│   │   │   └── test.yml
│   │   └── vars
│   │       └── main.yml
│   └── security
│       ├── defaults
│       │   └── main.yml
│       ├── files
│       ├── handlers
│       │   └── main.yml
│       ├── meta
│       │   └── main.yml
│       ├── README.md
│       ├── tasks
│       │   └── main.yml
│       ├── templates
│       ├── tests
│       │   ├── inventory
│       │   └── test.yml
│       └── vars
│           └── main.yml
├── site.yml
└── staging
```

{{% notice tip %}}
In real life, you should remove the unnecessary roles sub-directories to keep the structure easier to understand and maintain.
{{% /notice %}}
</p>
<hr/>
</details>

Since we so far created the code only locally on the control host, we need to add it to the repository and push it:

```bash
[{{< param "pre_mng_prompt" >}} structured-content]$ git add production roles site.yml staging
[{{< param "pre_mng_prompt" >}} structured-content]$ git commit -m "Adding inventories and apache security roles"
[{{< param "pre_mng_prompt" >}} structured-content]$ git push
```

## Launch it\!

### From the Command Line

The code can now be launched. We start at the command line. Call the playbook `site.yml` with the appropriate inventory and privilege escalation:

    [{{< param "pre_mng_prompt" >}} structured-content]$ ansible-playbook -i staging site.yml -b

Watch how the changes are done to the target machines. Afterwards, we could similarly execute the playbook against the production stage, but we want to keep something for controller to do, so we just check it:

    [{{< param "pre_mng_prompt" >}} structured-content]$ ansible-playbook -i production site.yml -b --list-hosts --list-tasks

Call e.g. `curl {{< param "internal_host1" >}}` to get the default page.

### From controller

To configure and use this repository as a **Source Control Management (SCM)** system in controller you have to create credentials again, this time to access the Git repository over SSH. This credential is user/key based, and we need the following playbook (assuming the `CONTROLLER_` environment variables are still defined):

```yaml
---
- name: Create git SCM
  hosts: localhost
  tasks:
    - name: create GitLab credentials
      awx.awx.credential:
        name: git
        credential_type: Source Control
        organization: Default
        state: present
        inputs:
          ssh_key_data: '{{ lookup("file", "~/.ssh/<GUID>key.pem") }}'
          username: git
    - name: create git project
      awx.awx.project:
        name: Structured Content Repository
        organization: Default
        scm_credential: git
        scm_type: git
        scm_update_on_launch: true
        scm_delete_on_update: true
        scm_url: "{{< param "git_user" >}}@{{< param "internal_control" >}}:{{< param "content_git_uri" >}}"
        state: present
```

{{% notice warning %}}
Make sure to replace the GUID in the code before starting it!
{{% /notice %}}

Save the playbook as `scm.yml` and run it with `ansible-playbook`.

Now you’ve created the Project in controller. Earlier on the command line you’ve setup a staged environment by creating and using two different inventory files. But how can we get the same setup in controller? We use another way to define Inventories\! It is possible to use inventory files provided in a SCM repository as an inventory source. This way we can use the inventory files we keep in Git.

In your controller web UI, open the **Resources ⇒ Inventories** view. Then click the ![add](../../images/blue_add_dd.png?classes=inline) button and choose to create a new **Inventory**. In the next view:

- **Name:** Structured Content Inventory

- Click **Save**

- Click the button **Sources** which is now active at the top

- Click the ![add](../../images/blue_add.png?classes=inline) button (the top right one)

- **Name:** Production

- **Source:** Pick **Sourced from a Project**

- **Project:** Structured Content Repository

- In the **Inventory file** drop down menu, pick **production**

- Click the green **Save** button and click on **Back to Sources**.

And now for the staging inventory:

- Click the ![add](../../images/blue_add.png?classes=inline) button again

- In the next view, add as **Name:** Staging

- **Source:** Pick **Sourced from a Project**

- **Project:** Structured Content Repository

- In the **Inventory file** drop down menu, pick **staging**

- Click the green **Save** button

- In the screen below, click on **Back to Sources** and then **Sync all**.

To make sure that the project based inventory worked, click on the **Resources ⇒ Hosts** button and make sure the two hosts are listed and tagged with the respective stages. To verify this, click on each host and check the **Groups** tab.

Now create a template to execute the `site.yml` against both stages at the same time and associate the credentials.

{{% notice tip %}}
Please note that in a real world use case you might want to have different templates to address the different stages separately.
{{% /notice %}}

Expand your `scm.yml` playbook with the necessary tasks.

```yaml
    - name: create site playbook
      awx.awx.job_template:
        name: Structured Content Execution
        inventory: Structured Content Inventory
        project: Structured Content Repository
        playbook: site.yml
        become_enabled: true
        credential: AWX Credentials
        execution_environment: Ansible Engine 2.9 execution environment
```

<details><summary><b>Click here for Solution (entire Playbook)</b></summary>
<hr/>
<p>

```yaml
- name: Create git SCM
  hosts: localhost
  tasks:
    - name: create GitLab credentials
      awx.awx.credential:
        name: git
        credential_type: Source Control
        organization: Default
        state: present
        inputs:
          ssh_key_data: '{{ lookup("file", "~/.ssh/<GUID>key.pem") }}'
          username: git
    - name: create git project
      awx.awx.project:
        name: Structured Content Repository
        organization: Default
        scm_credential: git
        scm_type: git
        scm_update_on_launch: true
        scm_delete_on_update: true
        scm_url: "{{< param "git_user" >}}@{{< param "internal_control" >}}:{{< param "content_git_uri" >}}"
        state: present
    - name: create site playbook
      awx.awx.job_template:
        name: Structured Content Execution
        inventory: Structured Content Inventory
        project: Structured Content Repository
        playbook: site.yml
        become_enabled: true
        credential: AWX Credentials
        execution_environment: Ansible Engine 2.9 execution environment
```

</p>
<hr/>
</details>

Now in the controller web UI go to **Resources ⇒ Templates**, launch the job template **Structured Content Execution** and watch the results.

## Adding External Roles

So far we have only worked with content inside a single repository. While this drastically reduces complexity already, the largest benefit is in sharing roles among multiple teams or departments and keeping them in a central place. In this section we will show how to reference shared roles in your code and execute them together on your behalf.

In enterprise environments it is common to share roles via internal git repositories, often one git repository per role. If a role might be interesting and re-used by the world wide Ansible community, they can be shared on our central platform [Ansible Galaxy](https://galaxy.ansible.com/). The advantage of Ansible Galaxy is that it features basic automatic testing and community ratings to give the interested users an idea of the quality and reusability of a role.

To use external roles in a project, they need to be referenced in a file called [`roles/requirements.yml`](https://docs.ansible.com/ansible/latest/reference_appendices/galaxy.html#installing-multiple-roles-from-a-file), for example like this:

```
# Import directly from Galaxy
- src: geerlingguy.nginx
# Import from a local Git repository
- src: http://control.example.com/gitea/git/external-role.git
  version: master
  name: external-role_locally
```

The `requirements.yml` needs to be read - either on the command line by invoking `ansible-galaxy`, or automatically by automation controller during project check outs. In both cases the file is read, and the roles are checked out and stored locally, and the roles can be called in playbooks. The advantage of controller here is that it takes care of all that - including authorization to the Git repo, finding a proper place to store the role, updating it when needed and so on.

In this example, we will include a role which ships a simple `index.html` file as template and reloads the apache web server. The role is already shared in GitHub at **https://github.com/ansible-labs-summit-crew/shared-apache-role**.

To include it with the existing structured content, first we have to create a file called `roles/requirements.yml` and reference the role there:

Let's create a `roles/requirements.yml` file:

```bash
[{{< param "pre_mng_prompt" >}} structured-content]$ cat roles/requirements.yml
---
- src: https://github.com/ansible-labs-summit-crew/shared-apache-role.git
  scm: git
  version: master
```

{{% notice tip %}}
In a production environment you may want to change the `version` to a fixed version or tag, to make sure that only tested and verified code is checked out and used. But this strongly depends on how you develop your code and which branching model you use.
{{% /notice %}}

Next, we reference the role itself in our playbook. Change the **site.yml** Playbook to look like this:

```bash
[{{< param "pre_mng_prompt" >}} structured-content]$ cat site.yml
---
- name: Execute apache and security roles
  hosts: all

  roles:
    - { role: apache}
    - { role: security }
    - { role: shared-apache-role }
```

Because controller uses your Git repo, you’ve to add, commit and push the
changes:

```bash
[{{< param "pre_mng_prompt" >}} structured-content]$ git add site.yml roles/
[{{< param "pre_mng_prompt" >}} structured-content]$ git commit -m "Add roles/requirements.yml referencing shared role"
[{{< param "pre_mng_prompt" >}} structured-content]$ git push
```

## Launch in controller

Just in case, make sure to update the Project in controller: in the menu at **Resources**, click **Projects**, and click on the sync button next to **Structured Content Repository**.

Afterwards, go to **Resources ⇒ Templates** and launch the **Structured Content Execution** job template. As you will see in the job output, the external role is called just the way the other roles are called:

    TASK [shared-apache-role : deploy content] *************************************
    changed: [{{< param "internal_host2" >}}]
    changed: [{{< param "internal_host1" >}}]

Validate again with `curl` the result and you are done\!

This was quite something to follow through, so let’s review:

- You successfully integrated a shared role provided from a central source into your automation code.

- This way, you can limit your automation code to things really relevant and individual to the task and your environment, while everything generic is consumed from a shared resource.
