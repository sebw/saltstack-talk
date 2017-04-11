# Demonstration content JDL 20170420

## Before getting started

Disable flux

## Getting started

- install salt-master RHEL + config

/etc/salt/master

```
file_roots:
  base:
    - /srv/salt/states

pillar_roots:
  base:
    - /srv/salt/pillars
```

- install salt-minion RHEL + config

/etc/salt/minion

```
master: 10.1.1.17
```

- systemctl stop firewalld on master
- accept salt-minion keys : salt-key -a jdl-minion*
- test.ping
- grains.items
- targetting based on grain: salt -G 'os:CentOS' test.ping

## Remote execution

- remote execution
	- cmd.run
	- iptables.version
	- selinux.getenforce (will show something is missing, deps on policycoreutils-python package to install)
	- pkg.install telnet
	
## Configuration Management

- manage /etc/motd

/srv/salt/states/motd/init.sls

```
motd:
  file.managed:
    - name: /etc/motd
    - source: salt://motd/motd.jinja
    - template: jinja
```

/srv/salt/states/motd/motd.jinja

```
Hello world
```
/srv/salt/states/top.sls

```
base:
  'jdl-minion':
    - motd
```

Verif : salt '*' state.show_top

```
jdl-minion:
    ----------
    base:
        - motd
```

- reminder: grains.items
- jinja template: use "os" or "mem_total" grains
- declare custom grain with multiple values `salt 'jdl-minion' grains.setval ROLE ['solr','apache']`
- use custom grain in MOTD and display raw result first then prettify it with jinja logic

```
Roles :
{% for i in grains['ROLE'] -%}
- {{ i }}
{% endfor -%}
```

- manage service.running postfix

Manage conf (retrieve /etc/postfix/main.cf and add comment at the top)

Manage service without watch statement (will tell service is already running)

Run `tail -f /var/log/maillog`

Add a watch statement and make a change to conf and state.highstate

```
postfix-conf:
  file.managed:
    - name: /etc/postfix/main.cf
    - source: salt://postfix/main.cf.jinja
    - template: jinja

postfix-service:
  service.running:
    - name: postfix
    - enable: True
    - reload: False
    - watch:
      - file: postfix-conf
```

- show content of /var/cache/salt/minion/files/base on minion (cache code)

## Pillars

After showing how code is stored on minion and presents security issues, show use of pillars

Put a password in clear text in conf main.cf.jinja and show file in cache again

Now create a pillar

/srv/salt/pillars/postfix/init.sls

```
postfix:
  password: Redlfmskdmlfs
```

/srv/salt/pillars/top.sls

```
base:
  'jdl-minion':
     - postfix
```

salt 'jdl-minion' pillar.data

Replace hardcoded "password=Truc" by "password={{ pillar['postfix']['password'] }}"

Show both syntaxes

```
password={{ pillar['postfix']['password'] }}
password={{ salt['pillar.get']('postfix:password') }}
```

Simple syntax gives error if pillar doesn't exist, other syntax is better

Show on a pillar that doesn't exist:

```
password={{ salt['pillar.get']('postfix:password2', 'valeur par defaut') }}
```

## Reactors

Show events on master bus : `salt-run state.event pretty=True`

Faire : `salt 'jdl-minion' test.ping -v`

```
salt/job/20170411144701244715/new	{
    "_stamp": "2017-04-11T18:47:01.245539",
    "arg": [],
    "fun": "test.ping",
    "jid": "20170411144701244715",
    "minions": [
        "jdl-minion"
    ],
    "tgt": "jdl-minion",
    "tgt_type": "glob",
    "user": "root"
}
salt/job/20170411144701244715/ret/jdl-minion	{
    "_stamp": "2017-04-11T18:47:01.351686",
    "cmd": "_return",
    "fun": "test.ping",
    "fun_args": [],
    "id": "jdl-minion",
    "jid": "20170411144701244715",
    "retcode": 0,
    "return": true,
    "success": true
}
```

Master conf

/etc/salt/master:

```
reactor:
  - 'salt/job/*/ret/jdl-minion':
    - /srv/salt/reactors/touch-jdl-minion2.sls
```

Reactor on jdl-minion2 when jdl-minion returns something

/srv/salt/reactors/touch.sls

```
command_run:
  cmd.cmd.run:
    - tgt: jdl-minion2
    - arg:
      - "touch /tmp/touch.txt"
```

salt 'jdl-minion1' test.ping

See on jdl-minion2 /tmp/touch.txt

## Reactor on beacons

yum install python-inotify sur minion (dep)

/etc/salt/minion.d/beacons.conf

```
beacons:
  inotify:
    /etc/passwd: {}
    interval: 5
```

Run `adduser jdl` on salt-minion and see event bus

```
salt/beacon/jdl-minion/inotify//etc/passwd	{
    "_stamp": "2017-04-11T19:08:18.505247",
    "change": "IN_IGNORED",
    "id": "jdl-minion",
    "path": "/etc/passwd"
}
```
	
## Extending Salt

- custom grain from API (httpd)

/var/www/html/index.html

```
{"custom-grain":"custom-value"}
```

/srv/salt/states/_grains/custom.py

```
#!/usr/bin/env python

import requests

def custom_grain():
	grains = {}
	r = requests.get("http://10.1.1.8")
	grains['zzz_custom'] = r.json()
	return grains
```

salt 'jdl-minion' saltutil.sync_all

Show minion logs

```
[INFO    ] Starting new HTTP connection (1): 10.1.1.17
[DEBUG   ] "GET / HTTP/1.1" 200 34
```

salt 'jdl-minion' grains.item zzz_custom

Use custom-grain in motd

```
{{ grains['zzz_custom']['custom-grain'] }}
```

## Salt Cloud deployments

- screenshots
- website docs
- show etnic setup

## Advanced use (if time allows)

- mine