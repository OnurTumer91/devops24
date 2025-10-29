# Examination 17 - sudo rules

In real life, passwordless sudo rules is a security concern. Most of the time, we want
to protect the switching of user identity through a password.

# QUESTION A

Create an Ansible role or playbook to remove the passwordless `sudo` rule for the `deploy`
user on your machines, but create a `sudo` rule to still be able to use `sudo` for everything,
but be required to enter a password.

On each virtual machine, the `deploy` user got its passwordless `sudo` rule from the Vagrant
bootstrap script, which placed it in `/etc/sudoers.d/deploy`.

Your solution should be able to have `deploy` connect to the host, make the change, and afterwards
be able to `sudo`, only this time with a password.

To be clear; we want to make sure that at no point should the `deploy` user be completely without
the ability to use `sudo`, since then we're pretty much locked out of our machines (save for using
Vagrant to connect to he machine and fix the problem).

*Tip*: Check out _validate_ in [ansible.builtin.lineinfile](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/lineinfile_module.html) to ensure a file can be parsed correctly (such as running `visudo --check`)
before being changed.

No password is set for the `deploy` user, so begin by setting the password to `hunter12`.

HINT: To construct a password hash (such as one for use in [ansible.builtin.user](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html), you can use the following command:

    $ openssl passwd -6 hunter12

This will give you a SHA512 password hash that you can use in the `password:` field.

You can verify this by trying to login to any of the nodes without the SSH key, but using the password
provided instead.

To be able to use the password when running the playbooks later, you must use the `--ask-become-pass`
option to `ansible` and `ansible-playbook` to provide the password. You can also place the password
in a file, like with `ansible-vault`, or have it encrypted via `ansible-vault`.


A: Created my hash. Used the password hash in vars as variable, i had to change the " to ' which I think caused an error on my first run. "{{deploy_password_hash}}" into '{{deploy_password_hash}}' and then we simply enforce that sudo has to enter pass.

This didnt work of course since I had some slacking files in etc/sudoers.d/deploy in which I have stated passwordless use. To clean those up I did a playbook
```yaml
---
- name: Clean up old passwordless sudo rules
  hosts: all
  become: true
  gather_facts: false

  tasks:
    - name: Disable old cloud-init sudo rule
      ansible.builtin.file:
        path: /etc/sudoers.d/90-cloud-init-users
        state: absent

    - name: Disable old deploy sudo rule
      ansible.builtin.file:
        path: /etc/sudoers.d/deploy
        state: absent

    - name: Ensure secure sudo rule is in place
      ansible.builtin.copy:
        dest: /etc/sudoers.d/99-deploy
        owner: root
        group: root
        mode: '0440'
        content: |
          deploy ALL=(ALL) ALL
        validate: 'visudo -cf %s'
```
This safely removes the two files I had which was causing some conflicts. I also did a Vagrant snapshot before just in case.

I also did a playbook for fun that would revert the deploy, just so I dont have to play around with vagrant snapshots.
```yaml
---
- name: Revert to passwordless sudo for deploy
  hosts: all
  become: true
  gather_facts: false

  tasks:
    - name: Restore passwordless sudo rule for deploy
      ansible.builtin.copy:
        dest: /etc/sudoers.d/deploy
        owner: root
        group: root
        mode: '0440'
        content: |
          %deploy ALL=(ALL) NOPASSWD: ALL
        validate: 'visudo -cf %s'
```

So now I can clean any existing deploy password rules, execute my ensure sudo playbook and if i want revert back to passwordless with revert_nopassword playbook.


