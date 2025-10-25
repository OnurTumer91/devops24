# Examination 1 - Understanding SSH and public key authentication

Connect to one of the virtual lab machines through SSH, i.e.

    $ ssh -i deploy_key -l deploy webserver

Study the `.ssh` folder in the home directory of the `deploy` user:

    $ ls -ld ~/.ssh

Look at the contents of the `~/.ssh` directory:

    $ ls -la ~/.ssh/

## QUESTION A

What are the permissions of the `~/.ssh` directory?
A: The owner has drwx(700)on folder and (600) on private files, which means it can read and write but no one else can.

Why are the permissions set in such a way?
A: If someone else had W they would be able to add their key, if they had R they would be able to see the users sensetive key.
And thus be able to connect or alter who can access.

## QUESTION B

What does the file `~/.ssh/authorized_keys` contain?
A:It contains a list of pub ssh keys, which means its the keys of who can tether themselves without needing to input any password.
F.eg Onur would need to add their key to "authorized_keys" in order to login without password into f.eg "onur@skoldator"

## QUESTION C

When logged into one of the VMs, how can you connect to the
other VM without a password?
A: The most straightforward method is to add the webservers ssh key into dbserver via -> 
ssh-keygen -t ed25519 -N '' -f ~./ssh/id_ed25519 -C 'deploy@webserver'

This creates a ssh key, then we copy it over by:
ssh-copy-id -i ~/.ssh/id_ed25519.pub deploy@dbserver

Then we can safely ssh from webserver to dbserver via:
ssh deploy@dbserver


### Hints:

* man ssh-keygen(1)
* ssh-copy-id(1) or use a text editor

## BONUS QUESTION

Can you run a command on a remote host via SSH? How?
A: by simply inputting it after the ssh command with quatation marks, f.eg:
ssh onur@skoldator "uptime"

which outputs the uptime of that particular machine, its like a blackhole.
