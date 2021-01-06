PatchManagement
===============

Because a system update is rarely just a `yum update -y`, `patchmanagement` is an Ansible role to easily manage the update process and all things around it for servers under RedHat, CentOS, Debian or Ubuntu OS.

During a patch management, if you want:

* to set the target in a maintenance mode in your supervision
* to send a message at the begining or the end of the process
* react when falling
* etc.

this role will make your life easier.

In all case, remember that you should never connect to a server to perform its upgrade.

Requirements
------------

Nothing special.

Features
--------

* Take some notes about the system - ip address and routes, runing process, mountpoints (for troubleshooting ! Don't forget it !)
* Play custom tasks for all servers (see `pm_before_update_tasks_file` vars)
* Play custom tasks for the target server (see playbook example chapter)

Our main goal :

* Patch the server with apt or yum

And after :
* Set Ansible facts with the current date and if wanted an env var.
* Play custom tasks for the target server (see playbook example chapter)
* Play custom tasks for all servers (see `pm_after_update_tasks_file` vars)

Role Variables
--------------

* `pm_restart_after_update`: default `true`

Will the target restart after the PM. This ensure the last kernel is running and that the server and its services are able to reboot. Consider doing it regularly.

* `pm_logpath`: default `/etc/ansible/facts.d/PM.log`

Where to store the date of PM that are successfull of failed.

* `pm_fact_name`: default `pm`

What is the fact name we want ?

* `pm_set_env_variable`: default `true`
* `pm_env_file_path`: default `/etc/profile.d/last_pm_date.sh

Do we set an env variable with last_pm date and where to store the script ?

* `pm_manage_yum_clean_all`: default `true`

Launch a yum clean all before updating
You should set to false if, for example, you have ever download the RPMs

* `pm_manage_apt_clean`: `true`

Launch a apt clean before updating

* `pm_manage_apt_autoremove`: `true`

Autoremove deb packages

* `pm_apt_verbose_package_list: `false`

Print apt result list

* `pm_before_update_tasks_file`:

For example `custom_tasks/pm_before_update_tasks_file.yml`. You can configure the role to launch this task file before update the server. Every tasks in the targeted file will be played.

Consider placing in some tasks like sending a message, taking a snapshot, setting a maintenance mode, etc.

* `pm_after_update_tasks_file`:

For example `custom_tasks/pm_after_update_tasks_file.yml`. You can configure the role to launch this task file before update the server. Every tasks in the targeted file will be played.

Example Playbook
----------------

Configuring a playbook is quiet easy as you can see:

    - name: Start a Patch Management
      hosts: servers
      vars:
        pm_before_update_tasks_file: custom_tasks/ pm_before_update_tasks_file.yml
        pm_after_update_tasks_file: custom_tasks/pm_after_update_tasks_file.yml
      tasks:
        - name: "Include patchmanagement"
          include_role:
            name: "patchmanagement"

In addition to this playbook, you can create 2 other tasks files by servers that will be before and after upgrading them. It is usefull for example to remove node from a LB, flushing caches, restarting services, etc.  

Unlike variables `pm_before_update_tasks_file` and `pm_after_update_tasks_file` that target tasks files which perform actions for all targets, these files focus on actions specific to a single server.

In the playbook directory, create the directory `custom_tasks` and files named `before_pm_{{ inventory_hostname_short }}_custom_tasks.yml` and `after_pm_{{ inventory_hostname_short }}_custom_tasks.yml`.

      - name: Play a custom tasks for this server define thanks to the after_pm_{{ inventory_hostname_short }}_custom_tasks.yml file.
        debug:
          msg: "This tasks is a custom tasks define in after_pm_{{ inventory_hostname_short }}_custom_tasks.yml file"

Molecule testing
----------------

You can test this role via molecule and docker :

      $ molecule test

See `molecule/default/converge.yml` for a good example on how to use this role.

License
-------

MIT

Author Information
------------------

This role was originally written by Antoine Le Morvan for Vivalto Sante based on the work of Nicolas Martin and the Ansible Team at Claranet France / BU RMP (Ismaël Ouattara and EliE Deloumeau).
