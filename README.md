Ansible for CMS
===============

This repository provides basic ansible playbooks to conveniently
- deliver CMS and isolate configuration files to contest servers,
- start a specific contest on them (this assumes you have configured systemd services for CMS) and
- restart `cms-resource.service`.

Prerequisites
-------------
Ansible allows to run pre-configured instructions on servers, including copying files there, modifying remote files, or running remote commands. The instructions to run are configured in `yaml` configuration files called playbooks. The servers (hosts) are listed in what is called an inventory file.

Ansible is included in just about any Linux distribution and can most likely be installed with your package manager.

Simple host configuration
-------------------------
In a simple one-server configuration like in the German selection for the IOI, the inventory file will just consist of that single server (host). So, if you are one of the German coaches, it suffices to put the following in a file `hosts`:

```bash
root@contest.informatik-olympiade.de ansible_user=root ansible_python_interpreter=auto_silent
```

copy_configs.yaml
-----------------
This playbook will copy a set of configuration files from your machine to the host(s). Note that as-is, it always writes four configuration files at once: `cms.conf`, `cms.ranking.conf`, `isolate` and an nginx site put under `/etc/nginx/sites-available/` with the name of the file you provide. Therefore, you will need to have and provide all of the right configuration files locally. The previous files on the hosts will be copied to backup files.

You can use this playbook as follows, where `[path]` is the local path to the configuration files.
```bash
ansible-playbook copy_configs.yaml \
                 -i hosts \
                 -e "cmsconf=[path]/cms.conf cmsrankingconf=[path]/cms.ranking.conf isolate=[path]/isolate cmsnginxconf=[path]/cms"
```

<!-- Alternatively, you can run `ansible-playbook copy_configs.yaml -i hosts` and will be prompted for the paths to the files. -->

Note that this will not create a link to the nginx site file from `sites-enabled`. You will need to create it manually if you want the nginx site to be available and have not done so already.

> :warning: This playbook does not (re)start any services. You need to do that yourself when appropriate.

start_contest.yaml
------------------
This playbook will change the host's `/home/cms/resource-service.conf` (this path may need to be adapted for your server) file with the `contest_id` you provide. If it has changed, it then restarts `cms-resource.service`. This will usually be the appropriate way to ensure a certain contest (or the all-contests mode) is running.

You can use this playbook as follows, where `[id]` is the ID of the contest you want to start, or `ALL` if you want to start the all-contests mode.
```bash
ansible-playbook start_contests.yaml \
                 -i hosts \
                 -e "contest_id=[id]"
```

Alternatively, you can run `ansible-playbook start_contests.yaml -i hosts` and will be prompted for the contest id.

> :warning: If the `CONTEST_ID` has not changed on the host and `cms-resource.service` was not running before, this command will not start it! You can use `restart_resource.yaml` in that case.

restart_resource.yaml
---------------------
This playbook restarts the `cms-resource.service`.


You can use this playbook as follows.
```bash
ansible-playbook restart_resource.yaml \
                 -i hosts
```

TODO
----
- Provide a convenient way to select the contest to start with `start_contest.yaml`. Currently, you have to guess or already know the ID of the contest, or run and cancel the script beforehand to find out.
- Let `copy_configs.yaml` have an option to change only some configuration files. Perhaps by ignoring them if an empty path was provided.
- Let `copy_configs.yaml` restart services as appropriate, perhaps after a prompt.
- Let `start_contest.yaml` show a prompt if it should (re)start `cms-resource.service` even though `contest_id` has not changed.
