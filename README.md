# Ansible-Hackathon

The quick intro to Ansible ... and grasp the POWER!

## Welcome to the hackathon

In this 3 hour workshop/hackathon you will be hacking with Ansible to get your hands dirty the way TeamKaizen did. This is basically how we started.

### Resources

To hack along with the cool kids you need machines! We have created 2 machines for you. And like all the cool kids... we did in the Azure Cloud via ... of course Ansible!

Your setup is like this

```ascii
  +--------------+               +------------------+
  |              |     winrm     |                  |
  | host (linux) +---------------> target (windows) |
  |              |               |                  |
  +--------------+               +------------------+
```

A control machine that is Linux based, and the target machine, a Windows Server.

From the Linux machine you will be executing commands that controls the Windows machines. We hope that you like terminals. What about nano? Hmmm. Let's see. We use the Git-Bash as a client. This is to get from your machine to the cloud machine. You can remote into the Windows machines via Remote Desktop. IP will be given. This is NOT a production like environment. This is just for fun.

There are a few tasks that you must perform as a part of this hackathon.

## Tasks

This will be your assignments that you can execute in order to get some hands-on with Ansible.

### Task 1

Get access to the Linux machine from your local machine. Let's open a **git-bash** like when you checkout code. In the terminal you write:

```bash
ssh [USER]@[ANSIBLE_HOST_PUBLIC_IP]
```

This will prompt you for a password. Enter it. We will be writing the password on the board. This should give you access to the Ansible control machine.

### Task 2

Clone THIS repo into the Linux host machine. Just:

```bash
git clone [THE_REPO_THAT_WE_WILL_BE_USING]
```

and cd into this repo. The repo holds all the initial Ansible relevant files for this hackathon.

### Task 3

Getting our hands dirty with Ansible! First up: Inventory.

You have 2 machines at your disposal, this is Ansible *inventory*. The control machine that you currently have SSH into and the Windows Server. In this hackathon the inventory can be found in: `./inventories/sandbox/hosts`.

Enter your Windows Server Private IP in the file.

```bash
nano ./inventories/sandbox/hosts
```

> Nano Tip: Ctrl+x to exit in terminal.

Congrats! Now you have inventory for Ansible.

### Task 4

Windows and access. Where are my credentials for my Windows machines? Yes. You are right. You are missing that. But this is easy to do.
The group variables file `./inventories/sandbox/group_vars/win` holds variables of inventory under the `[win]` group.

> We don't need a variable file for the linux group in this hackathon.

Now edit the `win` file with:

```bash
nano ./inventories/sandbox/group_vars/win
```

Change the `[USER]`and `[PASSWORD]` with the real credentials.

```yml
---
# file: group_vars/win
ansible_user: [USER]
ansible_password: [PASSWORD]
ansible_port: 5986
ansible_connection: winrm
# The following is necessary for Python 2.7.9+ (or any older Python that has backported SSLContext, eg, python 2.7.5 on RHEL7) when using default Win$
ansible_winrm_server_cert_validation: ignore
```

Congrats! Next. Let's see if the servers are reachable.

### Task 5

Are your inventory alive? Let's ping them - using ansible ad hoc commands.

> Command: `ansible [RESOURCE] -i [PATH_TO_YOUR_INVENTORY_FILE] -m [ANSIBLE_MODULE] -a[ARGUMENTS_TO_MODULE]`

We are using two different modules for this:

* [ping](https://docs.ansible.com/ansible/latest/modules/ping_module.html) for linux targets
* [win_ping](https://docs.ansible.com/ansible/latest/modules/win_ping_module.html#win-ping-module) for Windows targets

For Linux, run:

```bash
ansible linux -i ./inventories/sandbox/hosts -m ping
```

And for the Windows resources, run:

```bash
ansible win -i ./inventories/sandbox/hosts -m win_ping
```

Output will look like this:

```bash
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

and

```bash
srv1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

This should be green. If not... you are in trouble :)

### Task 6

Assuming that Task 5 was a success, let's head over to playbooks.

We start with the basics. To call a ansible playbook the syntax is fairly simple. You write

```bash
ansible-playbook -i [PATH_TO_YOUR_INVENTORY_FILE] [PLAYBOOK_TO_EXECUTE]
```

You have no playbooks yet. Let's create one.

The task here is to create a file on the linux host machine using the [copy](https://docs.ansible.com/ansible/latest/modules/copy_module.html?highlight=copy) module.

This will put a "hello world" text into a file and place it under `/tmp/testfile.txt`

Create a file in the root of your repo. Call it `opgave1-create-localfile.yml`. Run this command:

```bash
nano ./opgave1-create-localfile.yml
```

Put in the code

```yml
---
- name: This is a hello-world example
  hosts: linux
  tasks:
  - name: Create a file called '/tmp/testfile.txt' with the content 'hello world'.
    copy:
      content: "hello world\n"
      dest: /tmp/testfile.txt
```

Note the `hosts: linux`. This means that this playbook is running tasks on inventory under the linux group in the `hosts` files.

Now execute the playbook with

```bash
ansible-playbook -i ./inventories/stageing/hosts opgave1-create-localfile.yml
```

This will gather some facts of the target machine(s) so Ansible has some context of en environment that it is running in.
and run the task: "Create a file called '/tmp/testfile.txt' with the content 'hello world'"

So:

- green: ok
- yellow: The task changed a state on the inventory
- red: Error

Now try running the task again. :)

Validate that the playbook, by:

```bash
cat /tmp/testfile.txt
```

### Task 7

Ok. So now you have created a localfile. Great. What if you would want a file on all your servers? And only the Windows machines. Easy.

Let's create another playbook with the title `opgave2-copy-file-to-remote-servers.yml` with

```bash
nano ./opgave2-copy-file-to-remote-servers.yml
```

In that file put

```yml
---
- name: Create Directory On Remote Servers
  hosts: win
  tasks:
    - name: Create Directory
      win_file:
        path: C:\Temp\
        state: directory

- name: Copy File To Remote Servers
  hosts: win
  tasks:
    - name: Copy file
      win_copy:
        src: ./hejsa.txt
        dest: C:\Temp\hejsa.txt
```

> Copying files to Windows targets uses [win_copy](https://docs.ansible.com/ansible/latest/modules/win_copy_module.html#win-copy-module)

**Notice** the `hosts: win` element. Meaning that we are targeting the group "win".
The playbook above is using the "hejsa.txt" file came with the checkout.

Execute the playbook with

```bash
ansible-playbook -i ./inventories/sandbox/hosts opgave2-copy-file-to-remote-servers.yml
```

After the execution you should have a file under the Windows Server "C:\\Temp" folder. Use Remote Desktop to verify it.

### Task 8

We can do a lot with Ansible. One of the features it has is transforming files. It's Jinja2 transformation - just like our cookiecutter that we use to create pipelines. No copy of existing stuff and rename the internals of files. The chance of missing a change or leaving a default value is happening from time to time. So the Jinja2 can be used to transform a template file into the desired result rather than copy a file from existing project and tweak it. Here we can have input and change the desired data and it's very readable.

Let's try it!

Create a new file called `hejsa.txt.j2`. That is the transformation file. In it you should paste the following

```bash
Hejsa, {{ yourname }}

joke:

hvad var det første Michael Jackson sagde da han gik ind i en tøjbutik?
 - har i billy jeans?
```

I guess it's obivious that the {{ yourname }} is a variable that is used in the transformation. But how is it getting here? Well the playbook we are about to make says is all. It will have a user prompt input in the beginning and use that input in the transformation. The variable can de a default variable or be become present in the transformation context in a lots of different ways. Let's use the easiest way for now. A user input.

Create a new playbook with the name "opgave3-jinja2-transformation-and-copy.yml". By the way... this playbook also copies the new file that we are creating to the Windows servers.

In the playbook put

```yml
---
- name: Get input from the user
  hosts: linux
  gather_facts: False
  vars_prompt:
    - name: "yourname"
      prompt: "what is your name?"
      default: "Nobody"
      private: no

  tasks:
   - name: Transform
     win_template:
        src: hejsa.txt.j2
        dest: /tmp/hejsa-new.txt

- name: Create Directory Test
  hosts: win
  tasks:
    - name: Create Directory
      win_file:
        path: C:\Temp\
        state: directory

- name: Copy To Servers
  hosts: win
  tasks:
    - name: Copy file
      win_copy:
        src: /tmp/hejsa-new.txt
        dest: C:\Temp\hejsa-new.txt
```

> [win_template](https://docs.ansible.com/ansible/latest/modules/win_template_module.html?highlight=win_template): Templating files for Windows targets

The first task gets input from the user and creates a file on the local control machine. The second task ensures that there is a folder on the target machines that are called "Temp" in the root of C. The third task performs the copy of the file to the group "win"

Execute your new playbook with

```bash
ansible-playbook -i ./inventories/sandbox/hosts opgave3-jinja2-transformation-and-copy.yml
```

See the result with Remote Desktop

### Task 9

Hey... wait... is this all? NO. Again we can do so much with Ansible in virtually any context. It's just a ssh or a powershell connection needed to make it work. And chances that somebody else solved your problem is very high. Say you need to install a feature on a Windows server. Well... easy... there is a module called [win_feature](https://docs.ansible.com/ansible/latest/modules/win_feature_module.html?highlight=win_feature) you can use the your playbook.

Enough talk... more action!

Let's install some common tools on all our servers. That is the Windows servers. Let's use "chocolatey" for it. Hmmm. Hold your horses!!! There is a ready-to-use module for this called [win_chocolatey](https://docs.ansible.com/ansible/latest/modules/win_chocolatey_module.html?highlight=win_chocolatey). Wow!

Create a new playbook called `opgave4-install-commontools-via-chocolatey.yml`. 

In this file put

```yml
---
- name: Installing Via Chocolatey
  hosts: win

  tasks:
  - name: Install common software
    win_chocolatey:
      name: '{{ item }}'
      state: present
    loop:
      - notepadplusplus
      - 7zip
```

> This playbook uses [loops](https://docs.ansible.com/ansible/2.7/user_guide/playbooks_loops.html)

Put in some of your favorite tools from Chocolatey. Goto [https://chocolatey.org/](https://chocolatey.org/) to find more.

Execute your new playbook with (you might have guessed it... hopefully)

```bash
ansible-playbook -i ./inventories/sandbox/hosts opgave4-install-commontools-via-chocolatey.yml
```

Use Remote Desktop to see that your tools have been installed.

### Task 10

!!! BONUS !!!

We have just installed the tools. What would we have to do to uninstall a tool again?

> **Hint** what does "state: present" mean?

### The End

This concludes the Ansible-Hackathon for now. Comments suggestions and other related we can talk about afterwards. Come talk. Email. Slack. Morse-Code us!

- Dan dgg@kamstrup.com
- Bo boh@kamstrup.com