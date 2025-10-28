# Examination 9 - Use Ansible Vault for sensitive information

In the previous examination we set a password for the `webappuser`. To keep this password
in plain text in a playbook, or otherwise, is a huge security hole, especially
if we publish it to a public place like GitHub.

There is a way to keep sensitive information encrypted and unlocked at runtime with the
`ansible-vault` tool that comes with Ansible.

https://docs.ansible.com/ansible/latest/vault_guide/index.html

*IMPORTANT*: Keep a copy of the password for _unlocking_ the vault in plain text, so that
I can run the playbook without having to ask you for the password.

# QUESTION A

Make a copy of the playbook from the previous examination, call it `09-mariadb-password.yml`
and modify it so that the task that sets the password is injected via an Ansible variable,
instead of as a plain text string in the playbook.

# QUESTION B

When the [QUESTION A](#question-a) is solved, use `ansible-vault` to store the password in encrypted
form, and make it possible to run the playbook as before, but with the password as an
Ansible Vault secret instead.

A: Password is stored in devops24/ansible_home/vars/vault.txt
