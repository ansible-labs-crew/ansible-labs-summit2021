# Exercise 1.1 - Check the Prerequisites

**Read this in other languages**: ![uk](../images/uk.png) [English](README.md),  ![japan](../images/japan.png)[日本語](README.ja.md).

## Your Lab Environment

In this lab you work in a pre-configured lab environment. You will have access to the following hosts:

| Role                 | Inventory name |
| ---------------------| ---------------|
| Ansible Control Host | ansible        |
| Managed Host 1       | node1          |
| Managed Host 2       | node2          |
| Managed Host 2       | node3          |

## Step 1.1 - Access the Environment

You have been provided access to a virtual terminal/shell via code-server, an open-source Visual Studio code editor:

    	https://student<X>-code.<workshop>.rhdemo.io

Use the above link in your browser by replacing **\<X\>** in student**\<X\>** by the student number and **\<workshop\>** by the workshop name provided to you.

![code-server login](../images/vscode-pwd.png)

Use the provided password to login into the code-server and open a new terminal by heading to the menu item "Terminal" at the top of the page and select "New Terminal". A new section will appear in the lower half of the screen and you will be presented a prompt:

![code-server terminal](../images/vscode-terminal.png)

Now become root:

    [student<X>@ansible ~]$ sudo -i

Most prerequisite tasks have already been done for you:

  - Ansible software is installed

  - `sudo` has been configured on the managed hosts to run commands that require root privileges.

Check Ansible has been installed correctly (your actual Ansible version might differ):

    [root@ansible ~]# ansible --version
    ansible 2.9.5
    [...]

> **Note**
>
> Ansible is keeping configuration management simple. Ansible requires no database or running daemons and can run easily on a laptop. On the managed hosts it needs no running agent.

Log out of the root account again:

    [root@ansible ~]# exit
    logout

> **Note**
>
> In all subsequent exercises you should work as the student\<X\> user on the control node if not explicitly told differently.

## Step 1.2 - Working the Labs

You might have guessed by now this lab is pretty commandline-centric…​ :-)

  - Don’t type everything manually, use copy & paste from the browser when appropriate. But do still take time to think and understand.

  - All labs were prepared using **Vim**, but we understand not everybody loves it. Feel free to use alternative editors. In the lab environment we provide **Midnight Commander** (just run **mc**, function keys can be reached via Esc-\<n\> or simply clicked with the mouse) or **Nano** (run **nano**). Here is a short [editor intro](../0.0-support-docs/editor_intro.md).

> **Tip**
>
> In the lab guide commands you are supposed to run are shown with or without the expected output, whatever makes more sense in the context.

## Step 1.3 - Challenge Labs

You will soon discover that many chapters in this lab guide come with a "Challenge Lab" section. These labs are meant to give you a small task to solve using what you have learned so far. The solution of the task is shown underneath a warning sign.




||[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md#section-1---ansible-engine-exercises) | [Next Excercise ->](../1.2-adhoc)|
|:---|:---:|---:|
| [Next Excercise ->](../1.2-adhoc)|
|---:|
