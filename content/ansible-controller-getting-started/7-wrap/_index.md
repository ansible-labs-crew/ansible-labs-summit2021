+++
title = "Wrap up"
weight = 7
+++

## Final Challenge or Putting it all Together

This is the final challenge where we try to put most of what you have learned together.

## Let’s set the stage

Your team responsible for web application deployments like what they see in automation controller. To use it in their environment they put together these requirements:

- As the webservers can be used for either development purposes or in production, there has to be a way to flag them accordingly as **stage dev** or **stage prod**.

  - Currently `node1.<GUID>.internal` and `node3.<GUID>.internal` should be used as a development systems and `node2.<GUID>.internal` in production.

- Of course the content of the world famous application "index.html" will be different between dev and prod stages.

  - There should be a title on the page stating the environment.

  - There should be a content field.

- The content writer `wweb` should have access to a survey to change the content for dev and prod servers.

## The Git Repository

All code is already in place - this is a automation controller lab after all and not about configuring Apache. Check out the **Ansible Workshop Examples** git repository again at [https://github.com/ansible-labs-crew/playbooks_summit2021](https://github.com/ansible-labs-crew/playbooks_summit2021). You will find the playbook `webcontent.yml`, which calls the role `role_webcontent`.

Compared to the previous Apache installation role there is a major difference: there are now two versions of an `index.html` template, and a task deploying the template file which has a variable as part of the template file name.

Here are the files for you to review (path is relative to the Github repository):

- rhel/apache/roles/role_webcontent/templates/dev_index.html.j2

```html
<body>
<h1>This is a development webserver, have fun!</h1>
{{ dev_content }}
</body>
```

- rhel/apache/roles/role_webcontent/templates/prod_index.html.j2

```html
<body>
<h1>This is a production webserver, take care!</h1>
{{ prod_content }}
</body>
```

- rhel/apache/roles/role_webcontent/tasks/main.yml

{{% notice tip %}}
Only the part deploying the template is shown
{{% /notice %}}

```
[...]
- name: Deploy index.html from template
  template:
    src: "{{ stage }}_index.html.j2"
    dest: /var/www/html/index.html
  notify: apache-restart
```

## Prepare Inventory

There is of course more then one way to accomplish this, but here is what you should do:

- Make sure all three hosts are in the inventory `Webserver`.

- Define a variable `stage` with the value `dev` for the `Webserver` inventory:

  - Add `stage: dev` to the **Details** of inventory `Webserver` by putting it into the **Variables** field beneath the three start-yaml dashes.

{{% notice warning %}}
Make sure to add the variable to the inventory and **not** to a node!
{{% /notice %}}

- In the same way add a variable `stage: prod` but this time only for `node2.<GUID>.internal` (go to the **Hosts** view of the inventory).

{{% notice tip %}}
This way the host variable overrides the variable set at the Inventory level because it's more specific and takes precedence.
{{% /notice %}}

## Create the Template

- Create a new **Template** named `Create Web Content` that

  - Targets the `Webserver` inventory.

  - Uses the Playbook `rhel/apache/webcontent.yml` from the **Ansible Workshop Examples** Project.

  - Defines two variables: `dev_content: default dev content` and `prod_content: default prod content` in **Variables**.

  - Uses `Workshop Credentials` and runs with privilege escalation.

- Save and run the template.

## Check the results

Execute curl to get the web content from each node in your **VS Code** terminal:

```bash
[{{< param "pre_mng_prompt" >}} ~]$ curl -s http://node1.<GUID>.internal
<body>
<h1>This is a development webserver, have fun!</h1>
default dev content
</body>
[lab-user@bastion 0 /etc/ansible]$ curl -s http://node2.<GUID>.internal
<body>
<h1>This is a production webserver, take care!</h1>
default prod content
</body>
[lab-user@bastion 0 /etc/ansible]$ curl -s http://node3.<GUID>.internal
<body>
<h1>This is a development webserver, have fun!</h1>
default dev content
</body>
```

## Add Survey

- Add a survey to the Template to allow changing the variables `dev_content` and `prod_content`.

- Add permissions to the Team `Web Content` so the Template **Create Web Content** can be executed by `wweb`.

- Run the survey as user `wweb`.

Check the results again from your **VS Code** terminal using `curl` as above.

## Solution

<details><summary><b>Solution NOT below</b></summary>
<hr/>
<p>

You have to figure this one out by yourself! ;-)

</p>
<hr/>
</details>

You have done all the required configuration steps in the lab already. If unsure, just refer back to the respective chapters.

## The End

Congratulations, you finished your labs\! We hope you enjoyed your first encounter with automation controller as much as we enjoyed creating the labs.
