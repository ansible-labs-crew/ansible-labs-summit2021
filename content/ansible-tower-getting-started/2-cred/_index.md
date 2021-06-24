+++
title = "Inventories, credentials and ad hoc commands"
weight = 2
+++

## Create an Inventory

Let’s get started: The first thing we need is an inventory of your managed hosts. This is the equivalent of an inventory file in Ansible Engine. There is a lot more to it (like dynamic inventories) but let’s start with the basics.

- You should already have the web UI open, if not: Point your browser to the URL you were given, similar to **`https://{{< param "external_automation_controller1" >}}`** (replace "\<N\>" and "\<LABID\>") and log in as `admin` with the password given on the lab landing page.

Create the inventory:

- In the web UI menu on the left side, go to **Resources** → **Inventories**, click the ![Add](../../images/blue_add_dd.png?classes=inline) drop-down and choose **Add Inventory**.

- **Name:** Workshop Inventory

- **Organization:** Default

- Click **Save**

Go back to the **Inventories** list, your new **Workshop Inventory** should show up. Open the **Workshop Inventory** and click the **Hosts** tab, the list will be empty since we have not added any hosts yet.

So let's add some hosts. As mentioned in the intro you have three managed hosts in your lab environment, the names are resolved through the `/etc/hosts` file. The nodes are named `node1`, `node2` and `node3`.

Now add the hosts to the inventory in Automation Controller:

- Click the blue ![Add](../../images/blue_add.png?classes=inline) button.

- **Name:** node1

- Click **Save**

- Click the **Back to Hosts** button and repeat to add **node2** as a second and **node3** as a third node.

You have now created an inventory with three managed hosts.

## Machine Credentials

One of the great features of Automation Controller is to make credentials usable to users without making them visible. To allow Automation Controller to execute jobs on remote hosts, you must configure connection credentials.

{{% notice tip %}}
This is one of the most important features of Automation Controller: **Credential Separation**\! Credentials are defined separately and not with the hosts or inventory settings.
{{% /notice %}}

As this is an important part of your Automation Controller setup, why not make sure that connecting to the managed nodes from Automation Controller is working in the first place?

To test access to the nodes via SSH do the following:

- In your browser bring up the terminal window in code-server (remember this runs on the Automation Controller node)

- From here as user `ec2-user` SSH into `node1` or one of the other nodes and execute `sudo -i`.

{{% notice tip %}}
In your lab environment SSH key authentication has already been configured.
{{% /notice %}}

```bash
[{{< param "internal_control" >}} ~]$ ssh ec2-user@node1
[ec2-user@node1 ~]$
sudo -i
[root@node1 ~]# exit
[ec2-user@node1 ~]$ exit
```

What does this mean?

- Automation Controller user **student{{< param "student" >}}** can connect to the managed hosts with SSH key authentication as user **ec2-user**.

- User **ec2-user** can execute commands on the managed hosts as **root** with `sudo`.

## Configure Machine Credentials

Now we will configure the credentials to access our managed hosts from Automation Controller. In the **Resources** menu choose **Credentials**. Now:

Click the ![Add](../../images/blue_add.png?classes=inline) button to add new credentials

- **Name:** Workshop Credentials

- **Organization:** Click on the magnifying glass, pick **Default** and click **Select**

- **Credential Type:** Open the drop-down menu, and pick the type **Machine**

- **Username:** ec2-user

- **Privilege Escalation Method:** sudo

As we are using SSH key authentication, you have to provide an SSH private key that can be used to access the hosts. You could also configure password authentication here.

Bring up your code-server terminal on Automation Controller, and `cat` the SSH private key:

```bash
[{{< param "internal_control" >}} ~]$ cat .ssh/aws-private.pem
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA2nnL3m5sKvoSy37OZ8DQCTjTIPVmCJt/M02KgDt53+baYAFu1TIkC3Yk+HK1
[...]
-----END RSA PRIVATE KEY-----
```

- Copy the **complete private key** (including "BEGIN" and "END" lines) and paste it into the **SSH Private Key** field in the web UI.

- Click **Save**

Your new credentials have been created, go back to the **Resources -> Credentials -> Workshop Credentials** and note that the SSH key is not visible but shown as "encrypted".

You have now setup credentials for Ansible to access your managed hosts.

## Hold on: Ansible Execution Environments Primer!

Before you run your first Ansible ad hoc commands in Automation Controller it's about time to learn about one of the major new features in Ansible Automation Platform 2: **Execution Environments**!

Before AAP 2 the Automation Platform execution relied on using **bubblewrap** to isolate processes and Python virtual environments (venv) to sandbox dependencies. This lead to a number of issues like maintaining multiple venv, migrating Ansible content between execution nodes and much more. The concept of execution environments (EE) solves this by using Linux containers.

An EE is a container run from an image that contains everything your Ansible Playbook needs to run. It's basically a control node in a box that can be executed everywhere a Linux container can run. There are ready-made image that contain everything you would expect on an Ansible control node, but you can (and probably will) start to build your own, custom image for your very own requirements at some point.

Your Automation Controller has been preconfigured with some standard EE images. So first go through the next section covering ad hoc commands, we'll look into execution environments a bit deeper later.

{{% notice tip %}}
Linux containers are technologies that allow you to package and isolate applications with their entire runtime environment. This makes it easy to move the contained application between environments and nodes while retaining full functionality. 
In this lab you'll use the command `podman` later on. Podman is a daemonless container engine for developing, managing, and running Open Container Initiative (OCI) containers and container images on your Linux System. 
{{% /notice %}}
## Run Ad Hoc Commands

As you’ve probably done with Ansible before you can run ad hoc commands from Automation Controller as well.

- In the web UI go to **Resources → Inventories → Workshop Inventory**

- Click the **Hosts** button to change into the hosts view and select the three hosts by checking the boxes to the left of the host entries.

- Click **Run Command**. In the next screen you have to specify the ad hoc command:

  - As **Module** choose **ping**
  - Click **Next**
  - As **Execution Environment** choose **Controller Default EE**
  - Click **Next**
  - For **Machine Credential** choose **Workshop Credentials**.

  - Click **Launch**, and watch the output. It should report **SUCCESS** for all nodes, of course.

The simple **ping** module doesn’t need options. For other modules you need to supply the command to run as an argument. Run another ad hoc command, this time try the **command** module to find the user ID of the executing user using an ad hoc command.

- **Module:** command

- **Arguments:** id

How about trying to get some more private information from the system? Try to print out **/etc/shadow**.

- **Module:** command

- **Arguments:** cat /etc/shadow

{{% notice warning %}}
Expect an error!
{{% /notice %}}

Oops, the last one didn’t went well, all red.

Re-run the last ad hoc command but this time tick the **Enable Privilege Escalation** box.

As you see, this time it worked. For tasks that have to run as root you need to escalate the privileges. This is the same as the **become: yes** you’ve probably used often in your Ansible Playbooks.

## Challenge Lab: Ad Hoc Commands

Okay, a small challenge: Run an ad hoc to make sure the package "tmux" is installed on all hosts. If unsure, consult the Ansible documentation either via the web or by running `ansible-doc yum` in the VS Code terminal on your Automation Controller control host.

<details><summary><b>Click here for Solution</b></summary>
<hr/>
<p>

- **Module:** yum
- **Arguments:** name=tmux
- Tick **Enable Privilege Escalation**

</p>
<hr/>
</details>

{{% notice tip %}}
The yellow output of the command indicates Ansible has actually done something (here it needed to install the package). If you run the ad hoc command a second time, the output will be green and inform you that the package was already installed. So yellow in Ansible doesn’t mean "be careful"…​ ;-).
{{% /notice %}}

{{% notice tip %}}
Try to click one of the output lines in the window showing the job output. A small window with a lot of information about the job like module args will open. Have a look and close it again.
{{% /notice %}}

## Execution Environments: A deeper look

As promised let's look a bit deeper into execution environments. During the section covering ad hoc commands you have already seen you have to choose an execution environment, the same will hold true for running Playbooks later on. In your Automation Controller web UI, go to **Administration → Execution Environments**. You'll see a list of the configured execution environments and original location of the image, in our case the images are provided in the **quay.io** container registry. Here you could add your own registry with custom EE images, too.

So what happens, when Automation Controller runs an ad hoc command or Playbook? Let's see... 

You should already have your **VS Code** terminal open in another browser tab, if not open https://{{< param "external_code" >}} and do **Terminal -> New Terminal**. In this terminal:

- You should be your **student\<N>** user, become the **awx** user by running `sudo -u awx -i`
- First let's look for the image:
```
[awx@ansible-1 ~]$ podman images
REPOSITORY              TAG     IMAGE ID      CREATED       SIZE
quay.io/ansible/awx-ee  0.2.0   68b8d8c4702d  2 months ago  1.25 GB
```
- Yup, it's there as it was pulled from the registry when you first run an ad hoc command.
- Now we want to observe how an EE is run as a container when you execute your automation. `podman ps -w 2` will list running containers, the `-w` option updates the output regularly. Run the command, it will show you something like this:
```
CONTAINER ID  IMAGE   COMMAND  CREATED  STATUS  PORTS   NAMES
```
- Keep podman running, now it's time to execute some automation. 
- In the web UI run an ad hoc command again. Go to **Resources → Inventories → Workshop Inventory**
- In the **Hosts** view select the three hosts.
- Click **Run Command**. Specify the ad hoc command as you did before:
  - **Module**: command
  - **Arguments**: sleep 60
  - **Next**
  - **Execution Environment**: Controller Default EE
  - **Next**
  - **Machine Credential**: Workshop Credentials
  - Click **Launch**

Now go back to the **VS Code** terminal. You'll see a container is launched to run the ad hoc command and then removed again:

```
CONTAINER ID  IMAGE                         COMMAND               CREATED       STATUS            PORTS   NAMES
c8e9a13ab475  quay.io/ansible/awx-ee:0.2.0  ssh-agent sh -c t...  1 second ago  Up 2 seconds ago          ansible_runner_18
```
{{% notice tip %}}
The `sleep 60` command was only used to keep the container running for some time. Your output will differ slightly.
{{% /notice %}}

- Feel free to relaunch the job again!
- Stop `podman` with `CTRL-C`.

This is how Automation Controller uses Linux containers to run automation in it's own environment. You should see how this is beneficial:
- you don't have to worry about making sure dependencies are synced to other Automation controller nodes
- you could of course use EEs to run Ansible automation on the commandline outside of Automation Controller e.g. for development purposes!

 

