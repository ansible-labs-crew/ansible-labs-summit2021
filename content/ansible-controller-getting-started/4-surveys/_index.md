+++
title = "Surveys"
weight = 4
+++

You might have noticed the **Survey** tab when looking at a **Template**. A survey is a way to create a simple form to ask for parameters that get used as variables when a **Template** is launched as a **Job**.

You have installed Apache on all hosts in the job you just run. Now we’re going to extend on this:

- Use a proper role which has a Jinja2 template to deploy an `index.html` file.

- Create a job **Template** with a survey to collect the values for the `index.html` template.

- Launch the job **Template**.

Additionally, the role will also make sure that the Apache configuration is properly set up - in case it got mixed up during the other exercises.

{{% notice tip %}}
The survey feature only provides a simple query for data - it does not support four-eye principles, queries based on dynamic data or nested menus.
{{% /notice %}}

## The Apache-configuration Role

The Playbook and the role with the Jinja template already exist in the directory `rhel/apache` in the Github repository [https://github.com/ansible-labs-crew/playbooks_summit2021](https://github.com/ansible-labs-crew/playbooks_summit2021) you already configured as a Project.

 Head over to the Github UI and have a look at the content: the playbook `apache_role_install.yml` merely references the role. The role can be found in the `roles/role_apache` subdirectory.

- Inside the role, note the two variables in the `templates/index.html.j2` template file marked by `{{…​}}`\.

- Also, check out the tasks in `tasks/main.yml` that deploy the file from the template.

What is this Playbook doing? It creates a file (**dest**) on the managed hosts from the template (**src**).

The role also deploys a static configuration for Apache. This is to make sure that all changes done in the previous chapters are overwritten and your examples work properly.

Because the Playbook and role is located in the same Github repo as the `apache_install.yml` Playbook you don't have to configure a new project for this exercise.

## Create a Template with a Survey

Now you create a new Template that includes a survey.

### Create Template

- Go to **Templates**, click the ![add](../../images/blue_add.png?classes=inline) button and choose **Add job template**

- **Name:** Create index.html

- Configure the template to use:

  - `Webserver` as Inventory.

  - `Ansible Workshop Examples` as the **Project**.

  - `Default execution environment` for the **Execution Environment**.

  - `apache_role_install.yml` as the **Playbook** to execute.

  - `Workshop Credentials` as credentials.

  - privilege escalation.

Try for yourself, the solution is below.

<details><summary><b>Click here for Solution</b></summary>
<hr/>
<p>

- **Name:** Create index.html

- **Job Type:** Run

- **Inventory:** Webserver

- **Project:** Ansible Workshop Examples

- **Execution Environment:** Default execution environment

- **Playbook:** `rhel/apache/apache_role_install.yml`

- **Credentials:** Workshop Credentials

- **Options:** Privilege Escalation

- Click **Save**

</p>
<hr/>
</details>

{{% notice warning %}}
Do not run the template yet!
{{% /notice %}}

### Add the Survey

- In the Template, go to the **Survey** tab

- Click **Add** and fill in:

  - **Question:** First Line

  - **Answer variable name:** first_line

  - **Answer Type:** Text

- Click **Save**

- In the same way add a second survey question

  - **Question:** Second Line

  - **Answer variable name:** second_line

  - **Answer type:** Text

- Click **Save**

Now click the blue **Preview** button to see how the survey is going to look. Close the preview again.

To enable the survey, while still on the Survey tab switch the slider button to **On**
## Launch the Template

Now launch the **Create index.html** job template.

Before the actual launch the survey will ask for **First Line** and **Second Line**. Fill in some text and click **Next**. The next window shows the launch details and at the bottom the values the survey has prompted for. If all is good run the Job by clicking **Launch**.

{{% notice tip %}}
Note how the two survey lines are shown on the **Details** tab of the Job view as **Variables**.
{{% /notice %}}

After the job has completed, check the Apache homepage. In your **VS Code** terminal, execute `curl` against `http://node1.<GUID>.internal`:

```bash
[{{< param "pre_mng_prompt" >}} ~]$ curl http://node1.<GUID>.internal
<body>
<h1>Apache is running fine</h1>
<h1>This is survey field "First Line": line one</h1>
<h1>This is survey field "Second Line": line two</h1>
</body>
```

Note how the two variables where used by the playbook to create the content of the `index.html` file.

## What About Some Practice?

Here is a list of tasks:

{{% notice warning %}}
**Please make sure to finish these steps as the next chapter depends on it!**
{{% /notice %}}

- Take the inventory `Webserver` and add node `node2.<GUID>.internal` to it.

- Run the **Create index.html** Template again.

- Verify the results on `http://node2.<GUID>.internal` by using `curl`.
