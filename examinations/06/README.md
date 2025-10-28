# Examination 6 - Handling Web Server Content

The default web pages for any web server are not very interesting to look at.

If we open a web browser and point it towards the address of our web server,
you will likely get the default content of an unconfigured and unmanaged
web server:

Open Firefox or Chrome and enter the IP address of the web server, either
with http:// or http://

You should se a super snazzy web page from Alma Linux telling the visitor
that the administrator of this server needs to get their act together.

Also note that if you use https:// for secure HTTP, you will get a warning
telling you that you should be very careful accepting non-validating
certificates (such as the one we created earlier). This is normal, and
since we were the ones creating the certificate, we can just add an
exception for accepted certs, or simply use the http:// URL.

We will create a _virtual host_ on our web server, that serves different
content depending on which address it is called by via web browsers.

## Configure the nginx virtual host

The virtual host we will create will be called "example.internal", so that when we
go to http://example.ínternal or https://example.internal, our own web page
will be displayed instead. Obviously, this is a fake address, so we need
to do some black magic on our own machines first.

We will edit the file `/etc/hosts` on our host machine (i.e. the computer
you are working on).

Add the following line to this file, WITHOUT removing any of the other stuff
in this file:

    192.168.121.10  example.internal

Note that you need to be `root` to be able to edit this file, and that the address
given above is just an example. The actual IP of your `webserver` machine is
what we are interested in.

See if you now can resolve the name `example.internal`:

    $ ping -c 1 example.internal
    PING example.internal (192.168.121.10) 56(84) bytes of data.
    64 bytes from example.internal (192.168.121.10): icmp_seq=1 ttl=64 time=0.446 ms
    
    --- example.internal ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.446/0.446/0.446/0.000 ms

Again, the actual IP address it resolves to may be different on your machine.

If you have come this far, we can now move on to the next step.

## Upload our web page to the virtual host directory

Let's make a web page and upload to the web server so we can display our
own content instead.

Make a web page that looks something like this:

```xml
<?xml version="1.0"?>
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Hello Nackademin!</title>
  </head>
  <body>
    <h1>Hello Nackademin!</h1>
    <p>This is a totally awesome web page</p>
    <p>This page has been uploaded with <a href="https://www.ansible.com">Ansible</a>!</p>
  </body>
</html>
```
Note that this web page follows the HTML standards from W3C, in case you are
interested in why it looks the ways it does: https://html.spec.whatwg.org/multipage/

There is a copy of this file in the [files/index.html](files/index.html) directory adjacent to where
you are reading this file. Make sure this file exists in the directory `files/` in your
Ansible working directory too by copying it.

We will create the directory from where the web server will serve the pages under `example.internal`
in `/var/www/example.internal/html`.

Before we do that, we need to configure `nginx` to find the web pages in the new directory.

In the [files/](files/) directory, there is an `nginx` configuration file for `example.internal` called
[files/example.internal.conf](files/example.internal.conf). Copy this file into `files/` in your Ansible
working directory.

Before we do anything else, we will use Ansible to copy this file to `/etc/nginx/conf.d/example.internal.conf`
and then restart the web server.

Begin by copying the `05-web.yml` playbook to `06-web.yml`.

Add a task to the `06-web.yml` playbook BEFORE the web server is restarted that looks like this:

    - name: Ensure the nginx configuration is updated for example.internal
      ansible.builtin.copy:
        src: files/example.internal.conf
        dest: /etc/nginx/conf.d/example.internal.conf

You may now rerun the example playbook and see what happens.

# QUESTION A

In the `06-web.yml` playbook, add a couple of tasks:

* One task to create the directory structure under `/var/www/example.internal/html/`.
* One task to upload our `files/index.html` file to `/var/www/example.internal/html/index.html`.

HINTS:
* The module for creating a directory is, somewhat counterintuitively, called
[ansible.builtin.file](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html)
* If you want to serve files under a non-standard directory (such as the one we create above), we must
  also set the correct SELinux security context type on the directory and files. The context in question
  in this case should be `httpd_sys_content_t` for the `/var/www/example.internal/html/` directory.

A: Done!

# QUESTION B

To each of the tasks that change configuration files in the webserver, add a `register: [variable_name]`.

As an example:

    - name: Set up configuration for HTTPS
      ansible.builtin.copy:
        src: files/https.conf
        dest: /etc/nginx/conf.d/https.conf
      register: result

When the task is run, the result of the task is saved into the variable `result`, which is a dictionary.
This result can be compared in a conditional with the keyword `changed`, such as

    when: result is changed

or

    when: result.changed is true

or

    when: result["changed"] is true

or, more succinctly:

    when: result.changed

or even

    when: result["changed"]

You can check what the variable contains after each task with adding

    - name: Print the value of result
      ansible.builtin.debug:
        var: result

With the use of the `when:` keyword, make a conditional that only restarts the web server if either of
the tasks has had any change.

See https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_conditionals.html#basic-conditionals-with-when

There are several ways to accomplish this, and there is no _best_ way to do this with what we've done so far.

Is this a good way to handle these types of conditionals? What do you think?

A: Yeah it is pretty straght forward and m,akes sure unneccessary restarts are avoided. This uses the ansible principle of "idempotent" as in we make it more efficient and only restart tasks that must restart. For the scale of this project it was pretty easy to use but I dont know what the industry standard is for such procedures.

# BONUS QUESTION

Imagine you had a playbook with hundreds of tasks to be done on several hosts, and each one of these tasks
might require a restart or reload of a service.

Let's say the goal is to avoid restarts as much as possible to minimize downtime and interruptions; how
would you like the flow to work?

Describe in simple terms what your preferred task flow would look like, not necessarily implemented in
Ansible, but in general terms.

A: This ties together with my previous answer regarding what the industry stanard is for handeling conditions and restarts. If we have  a trigger for restart on every sub-task it would take some time but if we instead wait for all the configs and instead have a boolean like value as T/F for a reboot at the end we would only have to reboot once. Much like how windows update works, it downloads every sub-ptach/update, installs them and only then prompts the user for reboot. Not on every small driver/patch/update.
