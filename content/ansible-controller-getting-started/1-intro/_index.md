+++
title = "Introduction to automation controller"
weight = 1
+++

## Automation controller? Where is Ansible Tower?

During the planning of **Red Hat Ansible Automation Platform 2** the decision was made to rename a number of components. The main reason behind this is to make it clear that e.g. Ansible Engine and Ansible Tower are parts of an comprehensive automation platform.

So the artist formerly known as Ansible Tower is now called "automation controller" (_without_ capitals!).

Automation controller basically is an API for Ansible Automation, most users will get in touch with it through the web-based UI which uses the API underneath. It provides the following features:

- A user-friendly dashboard

- Role based access control

- One-click automation templates

- Management of dynamic inventory sources

- Automation workflows with approval

- A solid audit track ("who did what when")

And much more... as you'll learn in this lab!

## Your automation controller Lab Environment

In this lab you work in a pre-configured lab environment. You will have access to the following hosts:

| Role                   | URL for External Access (if applicable) | Hostname Internal                    |
| ---------------------- | --------------------------------------- | ------------------------------------ |
| Automation controller  | {{< param "external_controller1" >}}    | {{< param "internal_controller1" >}} |
| Visual Code Web UI     | {{< param "external_code" >}}           |                                      |
| Managed RHEL8 Host 1   |                                         | {{< param "internal_host1" >}}       |
| Managed RHEL8 Host 2   |                                         | {{< param "internal_host2" >}}       |
| Managed RHEL8 Host 3   |                                         | {{< param "internal_host3" >}}       |

{{% notice tip %}}
The lab environments in this session have a **{{< param "labid" >}}** and are separated by numbered **{{< param "student" >}}** accounts. You will be able to access the hosts using the external hostnames. Internally the hosts have different names as shown above. Follow the instructions given by the lab facilitators to receive the values for **{{< param "student" >}}** and **{{< param "labid" >}}**!
{{% /notice %}}

{{% notice tip %}}
Automation controller has already been installed and licensed for you, the web UI will be reachable over HTTP/HTTPS.
{{% /notice %}}

{{% notice info %}}
In general, whenever you need a password, even without the placeholder explicitly written, it's the same one.
{{% /notice %}}

## Working the Lab

Some hints to get you started:

- Don’t type everything manually, use copy & paste from the browser when appropriate. But don’t stop to think and understand… ;-)

- To **edit files** or **open a terminal window**, we provide **VS Code** delivered by **VS Code** server, basically the great Visual Studio Code Editor running in your browser. It's running on the bastion node and can be accessed through the URL `https://{{< param "external_code" >}}`

{{% notice tip %}}
Commands you are supposed to run are shown with or without the expected output, whatever makes more sense in the context.
{{% /notice %}}

{{% notice tip %}}
The command line can wrap on the HTML page from time to time. Therefore the output is often separated from the command line for better readability by an empty line. **Anyway, the line you should actually run should be recognizable by the prompt.** :-)
{{% /notice %}}

## Accessing your Lab Environment

You'll get the access information for your lab (URL's, password) from your lab facilitator. Your main points of contact with the lab are the automation controller's web UI and **VS Code** in your browser. You'll use **VS Code** to:

- Open virtual terminals

- Edit files

Now open **VS Code** in your browser using the link provided or use this link by replacing **{{< param "student" >}}** by your {{< param "student_label" >}} and the **{{< param "labid" >}}**:

`https://{{< param "external_code" >}}`

![VS Code login](../../images/vscode-pwd.png)

Use the password provided to login into the **VS Code** server web UI, you can close the **Welcome** tab. Now open a new terminal by heading to the menu item **Terminal** at the top of the page and select **New Terminal**. A new section will appear in the lower half of the screen and you will be greeted with a prompt:

![VS Code terminal](../../images/vscode-terminal.png)

If unsure about the usage, read the [Visual Studio Code Server introduction](../../vscode-intro/), to learn more about how to create and edit files, and to work with the Terminal.

{{% notice warning %}}
There is a **known bug when using VSCode in the Chrome browser**: Under some circumstances/locale settings the keyboard layout in the terminal window (not the visual editor) is mixed up. It works fine in Firefox, though.
{{% /notice %}}

### Direct Access using SSH

Last but not least you can of course use SSH directly to access the bastion node when you have an SSH client ready to go and know your way around:

`ssh lab-user@bastion.{{< param "student" >}}.{{< param "labid" >}}.opentlc.com`

The password is still the same.

Congrats, you now have a shell terminal on your bastion node. From here you run commands or access the other hosts in your lab environment if a lab task requires it.

{{% notice tip %}}
The user you are accessing the terminal as is `lab-user`, but your bastion node is setup to let you become `root` using _sudo_ without a password.
{{% /notice %}}

### Managed Nodes hostnames

As mentioned you can construct your internal hostnames with your **\<GUID>**. But there is an easier way: On your bastion host you can find an Ansible inventory file for your environment. Just look at it in your VSCode terminal and you'll get the internal hostnames:

```bash
[{{< param "pre_mng_prompt" >}} ~]$ cat /etc/ansible/hosts
```

## Dashboard

Let's have a first look at the automation controller: Point your browser to the URL you were given, similar to `https://{{< param "external_controller1" >}}` (replace `{{< param "student" >}}` with your {{< param "student_label" >}} and `{{< param "labid" >}}` with the {{< param "labid_label" >}}) and log in as `admin` with the passwrod provided.

The web UI of the automation controller greets you with a dashboard giving an overview of your automation including:

- Recent job activity

- The number of managed hosts

- Quick pointers to lists of hosts with problems.

The dashboard also displays real time data about the execution of tasks completed in playbooks.

![Automation controller Dashboard](../../images/dashboard.png)

## Concepts

Before we dive further into using automation controller for your automation, you should get familiar with some concepts and naming conventions.

### Projects

Projects are logical collections of Ansible playbooks in automation controller. These playbooks usually reside in a source code version control system supported by automation controller.

### Inventories

An Inventory is a collection of hosts against which jobs may be launched, the same as an Ansible inventory file you might know from working with Ansible on the command line. Inventories are divided into groups and these groups contain the actual hosts. Groups may be populated manually, by entering host names into automation controller, from one of automation controller’s supported cloud providers or through dynamic inventory scripts.

### Credentials

Credentials are utilized by automation controller for authentication when launching Jobs against machines, synchronizing with inventory sources, and importing project content from a version control system. Automation controller credentials are imported and stored encrypted in automation controller, and are not retrievable in plain text on the command line by any user. You can grant users and teams the ability to use these credentials, without actually exposing the credential to the user.

### Templates

A job template is a definition and set of parameters for running an Ansible job. Job templates are useful to execute the same job many times. Job templates also encourage the reuse of Ansible playbook content and collaboration between teams. To execute a job, automation controller requires that you first create a job template.

### Jobs

A job is basically an instance of automation controller launching an Ansible playbook against an inventory of hosts.
