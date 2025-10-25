# Examination 2 - Configure Ansible

To make things as unsurprising and easy as possible, we will all set up
the exact same Ansible configuration in a dedicated folder for this purpose.

Do not use the directory with the Vagrantfile (the `lab_environment` directory
if you are using this `git` repository).

Create a separate directory for this and the following examinations, and keep
the configuration there.

## Create ansible.cfg

Create a directory for Ansible where you will do your work. You can call it
anything, as long as you remember what it is. Also, it should be created
somewhere within your home directory.

In our examples below, we will call it `ansible`.

In our dedicated Ansible directory, create a file called `ansible.cfg`
(there is an example `ansible.cfg` in this same directory that you
 may copy).

Put the following content (only three lines) into this file:

    [defaults]
    host_key_checking = False
    inventory = hosts

Each row in this file either contains a section name within brackets.
`[defaults]` contain default values that are used if nothing more specific
is given.

`host_key_checking` is a mechanism by which `ssh(1)` checks that a hostname
has a given set of encryption keys given as an identoty. This is a security
mechanism used to avoid spoofing and man-in-the-middle attacks out in the
real world. Here, we disable it, otherwise we'll go crazy every time we
destroy or create the Vagrant hosts with new keys.

The `inventory` is where Ansible finds its inventory if you don't say
anything else, and `hosts` is its value. In this context, `hosts` is a file
that need to contain something that can be parsed as an Ansible inventory.
The `hosts` file in the same directory should contain:

    [all:vars]
    ansible_user=deploy
    ansible_ssh_private_key_file=<path_to_the_deploy_key_created_previously>

    [db]
    <IP or name from the Vagrant database host>

    [web]
    <IP or name from the Vagrant webserver host>

Anything within square brackets ('[' and ']') are section names. _all_ is a section
for things that apply to all VMs in this inventory. The section name ending in
'_:vars_' contain variable key/values. Almost every name except _all_ is a valid
name for any section name.

When you have created this file (with the proper values within `<` and `>` are
substituted for the actual values, you should be able to do

    $ ansible --list-hosts all
      hosts (2):
        dbserver
        webserver
    $ ansible -m ping all
    webserver | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python3.12"
        },
        "changed": false,
        "ping": "pong"
    }
    dbserver | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python3.12"
        },
        "changed": false,
        "ping": "pong"
    }

The actual values above may be different in your exact setup, but the `SUCCESS`
values should give an indication that Ansible is able to login to each host
via SSH.

### Troubleshooting

If you get a warning from Ansible that looks something like this

    [WARNING]: Platform linux on host dbserver is using the discovered Python interpreter at /usr/bin/python3.12, but
    future installation of another Python interpreter could change the meaning of that path. See
    https://docs.ansible.com/ansible-core/2.18/reference_appendices/interpreter_discovery.html for more information.

To disable the warning, you may follow the suggestion on the web page above, i.e. in the `ansible.cfg`:

    [defaults]
    ...
    interpreter_python = python3
    ...

This will set the interpreter statically to `python3` which should be available on most modern operating systems.

You can also set it to

    interpreter_python = auto_silent

to make ansible figure out which Python interpreter to use, but stop giving warnings about potential future
incompatibilities.

## QUESTION A

What happens if you run `ansible-inventory --list` in the directory you created above?
A:You get the inventory written out in json format with all its variabels and groups.

## QUESTION B

What happens if you run `ansible-inventory --graph` in the directory you created above?
A: It prints out a very minimalist graph/tree of the inventory.

## QUESTION C

In the `hosts` file, add a new host with the same hostname as the machine you're running
ansible on:

    [controller]
    <hostname_of_your_machine> ansible_connection=local

Now run:

    $ ansible -m ping all

Study the output of this command.

What does the `ansible_connection=local` part mean?
A: If i understand corectly we define a group called "Controller" as in a hostgroup since its inside the host file.
Inside of this hostgroup we put our local computer inside it which should mean; Ansible doesnt have to SSH anymore and run the commands directly since its a local computer, it skips the ssh step.


## BONUS QUESTION

The command `ansible-config` can be used for creating, viewing, validating, and listing
Ansible configuration.

Try running

    $ ansible-config --help

Make an initial configuration file with the help of this command, and write it into a file
called `ansible.cfg.init`. HINT: Redirections in the terminal can be done with '>' or 'tee(1)'.

Open this file and look at the various options you can configure in Ansible.

In your Ansible working directory where the `ansible.cfg' is, run

    $ ansible-config dump

You should get a pager displaying all available configuration values. How does it differ
from when you run the same command in your usual home directory?

A: When I run it in the ansible.cfg folder I can clearly see "CONFIG_FILE() = /home/onur/ansible_home/ansible.cfg" line present in the dump which means i fetches
my custom settings. Whilst it doesn't when Im outside that folder which should mean it runs on "default" settings/values then. This probably means you can have custom settings
in each folder, for each project you're working on.
