<!-- $width: 1300 -->
<!-- $height: 1000 -->
<!-- *page_number: false -->
<!-- $theme: default -->
<!-- footer: Les Jeudis du Libre -->
<!-- prerender: true -->
<!-- editor: Marp 0.0.10 --> 

![](./img/logo.png)


## La gestion de configuration simple et puissante

_Les Jeudis du Libre, Mons_
_20/04/2017_ **TODO**

Sébastien Wains
sebastien.wains@gmail.com

---

![bg 70%](./img/bg.png)

<!-- page_number: true -->

# Qui suis-je ?

#### Depuis 2010 : administrateur systèmes Linux chez ETNIC, pôle de compétence IT de la Fédération Wallonie-Bruxelles

#### Avant : Province de Hainaut, indépendant, Multitel ASBL

#### Blogger presqu'à la retraite ([https://blog.wains.be](https://blog.wains.be))

#### Gradué de l'Institut Supérieur Economique (ISE) de Flénu en section... comptabilité

#### Red Hat Certified Engineer sous RHEL4

#### Contributeur à différents projets Open Source, principalement Salt et Rundeck aujourd'hui

---

![bg 70%](./img/bg.png)

# Les avantages de la gestion de configuration

- efficacité
- stabilité
- suivi
- visibilité
- ...

---

![bg 70%](./img/bg.png)

# Les précurseurs

|Projet|Année|
|:-:|:-:|
|CFEngine|1993|
|Puppet|2005|
|Chef|2009|

---

![bg 70%](./img/bg.png)

# Les petits "nouveaux"

|Projet|Année|GitHub|
|:-:|:-:|:-|
|Rexify|2010|![](./img/github-rexify.png)`
|Salt|2011|![](./img/github-salt.png)
|Ansible|2012|![](./img/github-ansible.png)

---

![bg 70%](./img/bg.png)

# Un peu d'histoire

#### En 2010, l'ETNIC avait environ 150 serveurs Linux (70% virtuels)

- Des ressources techniques : DNS, relais SMTP, webmail, forward et reverse proxy, serveurs web, database, etc.
- Des projets et applications métiers : ESB, data warehouse, gestion électronique documentaire, SAP, formulaires intelligents, CMS, etc.

Utilisateurs : 

- 160 employés ETNIC
- 5000 utilisateurs au Ministère de la Communauté Française
- 1100 utilisateurs à l'ONE
- 130000 enseignants

---

![bg 70%](./img/bg.png)

# Un peu d'histoire

#### Les problèmes constatés dès les premières semaines

- Authentification `root` par mot de passe...
- ...connu de presque tout le monde (devs, ops, consultants !)
- Des distributions différentes (SuSE Enterprise, OpenSuse, Debian, RedHat, RedHat Enterprise)
- Pas d'utilisation de packages, ou alors trouvés sur rpmfind et autres
- Aucune gestion des mises à jour
- Des services SSH, NTP, SMTP, DNS mal ou pas configurés
- Des inconsistences entre environnements d'un même projet (`service alfresco start` en ACC, `service tomcat start` en PRD) ou des configurations backend d'un même cluster
- Pas d'authentification centralisée
- Services sécurisés par SSL self-signed ou pas du tout
- Installation OS entièrement à la main depuis l'ISO qui traine sur un PC
- Monitoring quasi absent
- Un équivalent temps plein pour remettre de l'ordre dans tout ça...

---

![bg 70%](./img/bg.png)

# Un peu d'histoire

#### Par où commencer ?

- La situation est critique partout
- Il faut définir des standards et nouvelles méthodes de travail
- Conscientiser le management afin de dégager des ressources
- Dire "non tu ne seras plus root en production" à quasi tout le monde
- Il faut améliorer la manière d'installer les nouveaux serveurs et leurs configurations
- Et migrer, migrer, migrer les anciens !
- Mais aussi travailler sur les nouveaux projets 

# :scream:

---

![bg 70%](./img/bg.png)

# Un peu d'histoire

#### Pour faire pour un mieux et dans l'immédiat je pars sur ces standards

- Installation d'un serveur : template VMware RHEL5 installation minimale
- Installation des services de bases et configurations : script bash

---

![bg 70%](./img/bg.png)

# Un peu d'histoire

### Mais après à peine quelques semaines...

```
$ wc -l install_rhel.sh 
664 install_rhel.sh
```

Et je ne gère que le strict minimum !



---

![bg 70%](./img/bg.png)

# Pourquoi pas... Puppet ?

#### Fin 2011 je participe à une formation Puppet


- à l'époque Puppet est toujours en mode "pull" (le client demande au serveur si il y a un changement qui le concerne)
- la recommandation du formateur pour une infrastructure telle que la notre était un pull toutes les 30 minutes et au moins deux masters (charge). Risques de delta jusqu'à 29 minutes entre serveurs d'un même cluster.
- la gestion de configuration, le remote execution et facter sont trois composants séparés installés individuellements (Puppet, MCollective, Facter)
- la syntaxe n'est pas très claire (Ruby DSL)

#### Conclusion : pas vraiment convaincu

---

# Pourquoi Salt ? La genèse...

![bg 70%](./img/bg.png)

#### Premiers tests en mai 2013

- un serveur `salt-master` pour gérer tous les clients `salt-minion`
- gestion de configuration de services de base  pour commencer (SSH, SMTP, NTP)
- remote execution (`yum upgrade x`, `uptime`)
- récupération d'informations sur le parc (CPU, mémoire, version OS)
- code dans SVN
- test du code via `test=True`

---

![bg 70%](./img/bg.png)

# Les avantages (en 2013)

- Orchestration "event-driven" via un `event bus` sur le master
- Ecrit en Python avec des possibilités d'extensions intéressantes
- YAML et Jinja (mais attention à la syntaxe !)
- Modèle client/serveur (`minion`/`master`)
- Mode push ET pull
- Début d'un support de Windows
- Gestion de configuration, remote execution, facts dans un seul package
- Simple ! De l'installation à un premier fichier de configuration géré : 12 minutes
- Communauté enthousiaste et dynamique

---

![bg 70%](./img/bg.png)

# Les inconvénients (en 2013)

- Installation d'un agent (salt-minion)
- Agent et ses dépendances (Python, ZeroMQ, msgpack) éparpillés dans les dépôts (Redhat, EPEL)
- Master et minion doivent être obligatoirement à la même version
- Faille de sécurité importante (PKI)
- Pas de support Python 3
- Installation de Salt-API impossible
- Pull requests acceptés 5 minutes montre en main
- Release cycle "Chuck Norris"
- Régressions régulières
- Quelques gros bugs :
  - `reload: True` qui fait un restart du service --> plus d'internet pour 5000 utilisateurs pendant le restart de 4 nodes Squid
  - ZeroMQ sous RHEL5 qui provoque la perte régulière des minions

---

![bg 70%](./img/bg.png)

# Salt aujourd'hui

- Toujours pas de support Python 3
- SaltStack fourni des dépôts avec toutes les dépendances [0]
- Le support de Windows et MacOS est avancé
- Ils ont engagé une équipe de testeurs : moins de releases, quasi plus de régressions
- Salt SSH pour gérer les "dumb" devices qui embarquent Python 2.6 ou plus
- Salt Proxy pour gérer les "super dumb" devices n'embarquant pas de stack Python
- Salt API fonctionne ! Intégrations possibles avec Jenkins, Rundeck, Satellite, etc.
- Un web GUI dans la version enterprise

[0] [https://repo.saltstack.com](https://repo.saltstack.com)

---

![bg 70%](./img/bg.png)

# Salt à l'ETNIC aujourd'hui

- L'équipe Linux a triplé il y a un an et Salt a été adopté immédiatement par les deux nouveaux membres
- Un consultant de SaltStack est venu auditer notre infrastructure
- Salt Community 2016.11 (dernière release stable)
- 260 serveurs Red Hat gérés (virtualisation 98%) :thumbsup:
- Deux salt-master (un `develop` pour les serveurs DEV/ACC, un `master` pour les serveurs PRD)
- Trois salt-master et trois salt-minion pour le développement des `states`
- Encore quelques serveurs legacy non gérables :bomb:
- Un nouveau serveur virtuel RHEL7 complètement provisionné et intégré en moins de 10 minutes grâce à Salt Cloud et Rundeck [0]
- Tout le code dans un dépôt Gitlab
- Basé sur un vrai workflow de développement [1]

[0] [http://www.rundeck.org](http://www.rundeck.org)
[1] [https://www.atlassian.com/git/tutorials/comparing-workflows/](https://www.atlassian.com/git/tutorials/comparing-workflows/)

---

# Workflow de développement

![](./img/git-workflow.png)

---

![150%](./img/under-control.jpg)

---

![bg 70%](./img/bg.png)

# Fonctionnement de base

#### Glossaire

`master` : serveur de gestion  
`minion` : serveur géré  
`modules` : module d'exécution ayant différentes fonctions (ex : file.managed)
`states` : état de configuration (package installé, fichier configuré, service démarré, etc.)  
`grains` : informations relativement statiques d'un minion  
`pillar` : information dynamique stockée sur le master à disposition des minions 
`beacons` : fonctionnalité permettant de monitorer des processus hors Salt (charge système, RAM, fichier, nombre de sessions HAproxy, etc.) et envoyer des messages sur l'`event bus`  
`reactors` : action déclenchée en réaction à un évenement sur l'`event bus`  
`top.sls` : défini quels `states` et `pillars` sont appliqués sur quels minions  
`init.sls` : manifest d'un `state`, `pillar`  

:heavy_exclamation_mark: Curieusement dans la documentation et la configuration, SaltStack parle de `states` et `grains` (pluriel) mais de `pillar` (singulier)

---

![bg 70%](./img/bg.png)

# Fonctionnement de base

#### Mode client/server

![140%](./img/infra.png)

- Les minions restent connectés constamment au master (message bus ZeroMQ)
- Le master doit accepter la clé d'un nouveau minion (PKI)
- Sur le master, deux ports écoutent :
  - TCP/4505 : le bus de communication avec les minions
  - TCP/4506 : pour les messages de retour des minions
- Le salt-master peut tourner en utilisateur non privilégié
- Le salt-minion doit tourner en root

---

![bg 70%](./img/bg.png)

# Installation

Un master contrôle plusieurs minions.

#### Sur le master :

`yum install salt-master`

#### Sur les minions :

`yum install salt-minion`


---

![bg 70%](./img/bg.png)

# Configuration du master

/etc/salt/master :

```yaml
file_roots:
  base:
    - /srv/salt/states

pillar_roots:
  base:
    - /srv/salt/pillars
```

Démarrage du service : `service salt-master start`

---

![bg 70%](./img/bg.png)

# Configuration des minions

/etc/salt/minion :

```yaml
master: salt-master
```

Démarrage du service : `service salt-minion start`

---

![bg 70%](./img/bg.png)

# Accepter un minion

Depuis le master : `salt-key -y -a salt-minion` 

# Vérifier le statut de nos minions

```bash
[root@salt-master ~]# salt-key
Accepted Keys:
salt-minion <=================== notre minion est accepté :-)
Denied Keys:
Unaccepted Keys:
Rejected Keys:
```

---

![bg 70%](./img/bg.png)

# On vérifie s'il répond bien

```bash
[root@salt-master ~]# salt 'salt-minion' test.ping
salt-minion:
    True
```

Ceci n'est pas un ping ICMP !

Voici ce qui est en réalité exécuté sur le minion (code simplifié de /usr/lib/python2.7/site-packages/salt/modules/test.py):

```python
def ping():
    return True
```


---

![bg 70%](./img/bg.png)

# Rappel avant d'écrire nos premières lignes de code

Le code qu'on va développer est au service de l'infrastructure (un bug dans le code = un downtime possible)

Les bonnes pratiques de développement s'appliquent ! Définir des guidelines de développement (syntaxe, structure des fichiers, etc.)

Système de contrôle de versions (SVN, Git, Mercurial) avec un workflow de développement

Ne rien pousser en production qui n'a pas été testé et validé

Et surtout :

## KISS : Keep It Simple, Stupid

> La perfection est atteinte, non pas lorsqu'il n'y a plus rien à  ajouter, mais lorsqu'il n'y a plus rien à  retirer. ~ Antoine de Saint-Exupéry

---

![bg 70%](./img/bg.png)

# Arborescence sur le master

```bash
[root@salt-master /]# find /srv
/srv
/srv/salt
/srv/salt/pillars
/srv/salt/pillars/top.sls
/srv/salt/pillars/passwords
/srv/salt/pillars/passwords/init.sls
/srv/salt/states
/srv/salt/states/motd
/srv/salt/states/motd/init.sls
/srv/salt/states/motd/motd.jinja
/srv/salt/states/selinux
/srv/salt/states/selinux/init.sls
/srv/salt/states/top.sls
```

---

![bg 70%](./img/bg.png)

# top.sls pour les states

```yaml
[root@salt-master /]# cat /srv/salt/states/top.sls
base:
  '*':
    - motd
    
  'G@os:RedHat':
    - selinux

  'G@ROLE:solr':
    - solr
```

Appliquer le state `motd` sur tous les `minions`, 

Pour les OS de type `RedHat`, appliquer le state `selinux`.

Ne jamais cibler un serveur sur base de son nom ! Le top.sls doit être le plus **générique** possible.

---

![bg 70%](./img/bg.png)

# Notre premier state : `motd`

MOTD : "message of the day", message affiché à la connexion au serveur

Notre manifest d'installation :

```yaml
[root@salt-master ~]# cat /srv/salt/states/motd/init.sls
motd:                            <-- ID unique
  file.managed                   <-- module.fonction (comme dans Python)
    - name: /etc/motd            <-- le fichier géré
    - source: salt://motd/motd.jinja   <-- template à utiliser
    - template: jinja            <-- type de template
```

`salt://` est un serveur HTTP embarqué dans Salt, on peut spécifier d'autres types de sources : `http://`, `https://`, `ftp://`, `file:///`, etc.

Par défaut jinja, plusieurs moteurs de template supportés. KISS !

---

![bg 70%](./img/bg.png)

# Notre premier state : `motd`

Le template jinja :

```jinja
[root@salt-master ~]# cat /srv/salt/states/motd/motd.jinja
Bonjour et bienvenue sur {{ grains['fqdn'] }}
Mon master est {{ grains['master'] }}
```


---

![bg 70%](./img/bg.png)

# Notre premier state : `motd`

Pour lister les grains disponibles sur un minion :

```yaml
[root@salt-master ~]# salt 'salt-minion' grains.items
salt-minion:
    ----------
[...]
    fqdn:
        salt-minion
    master:
        10.211.55.26
    mem_total:
        988
    osfullname:
        Red Hat Enterprise Linux Server
    osmajorrelease:
        7
[...]
```

---

![bg 70%](./img/bg.png)

# Notre premier state : `motd`

Appliquons la configuration avec `state.highstate` :

```bash
[root@salt-master ~]# salt 'salt-minion' state.highstate
salt-minion:
----------
          ID: motd
    Function: file.managed
        Name: /etc/motd
      Result: True
     Comment: File /etc/motd updated
     Started: 09:53:06.610581
    Duration: 78.358 ms
     Changes:
              ----------
              diff:
                  New file
              mode:
                  0644

Summary for salt-minion
------------
Succeeded: 2 (changed=1)
Failed:    0
------------
Total states run:     2
Total run time:  79.083 ms                                         79 ms !
```

---

![bg 70%](./img/bg.png)

# Notre premier state : `motd`

Vérifions sur notre minion :

```bash
[root@salt-minion ~]# cat /etc/motd
Bonjour et bienvenue sur salt-minion
Mon master est 10.211.55.26
```

---

![bg 70%](./img/bg.png)

# Allons plus loin...

Gestion d'un service et de sa configuration !

```yaml
postfix-pkg:
  pkg.installed:
    - name: postfix

postfix-service:
  service.running:
    - enable: True
    - reload: False
    - require:
      - pkg: postfix-pkg
    - watch:
      - file: postfix-conf

postfix-conf:
  file.managed:
    - name: /etc/postfix/master
    - source: salt://postfix/master
    - template: jinja
    - require:
      - pkg: postfix-pkg
```

---

![bg 70%](./img/bg.png)

# Portabilité du code entre OS

`pkg.installed` utilisera le gestionnaire de package du système : `yum`, `apt`, `zypper`, etc.
`service.running` démarra le service via ce qu'il trouve parmi `sysVinit`, `systemd`, `upstart`.

# Oui mais...

Noms de packages différents entre distributions (`apache2` vs `httpd`) ?

Map files YAML !



---

![bg 70%](./img/bg.png)

# Un template plus avancé

```jinja
{% if grains['os'] == 'RedHat' and grains['osmajorelease'] == '5' %}
...
{% elif grains['os'] == 'RedHat' and grains['osmajorelease'] >= '6' %}
...
{% elif grains['os'] == 'Debian' %}
...
{% else %}
...
{% endif %}
```

Les grains à notre disposition sont nombreux, mais est-ce suffisant ?

---

![bg 70%](./img/bg.png)

# Définir un nouveau grain manuellement

```bash
master# salt 'salt-minion' grains.setval ROLE ['solr','elastic']
salt-minion:
    ----------
    ROLE:
        - solr
        - elastic
```

On peut alors avoir un template tel que :

```jinja
{% if grains['ROLE'] is defined %}
{% if 'solr' in grains['ROLE'] %}
...
{% endif %}
{% endif %}
```
Ou :
```jinja
{% for i in grains['ROLE'] %}
role-{{ i }}-conf:
  file.managed:
  ...
{% endfor %}
```
---

# Définir un grain automatiquement

Il est possible de récupérer des informations provenant de différentes sources (CMDB, LDAP, DB, API, tc.) et de les stocker dans un grain du minion.

Cette info sera rappatriée au démarrage du minion ou alors via la commande `saltutil.sync_grains`

Ce script Python sera placé sous `/srv/salt/states/_grains/satellite.py`

```python
#!/usr/bin/env python

import requests

def satellite_retrieve_info():
        node_id = __opts__['id']

        satellite_pillar = __pillar__.get('satellite', None)
        username = satellite_pillar['api_username']
        password = satellite_pillar['api_password']
        url = satellite_pillar['api_url'] 

        r = requests.get(url + '/hosts/' + node_id + '/',
        auth=(username, password))

        grains = {}
        grains['INFO_SATELLITE'] = r.json()
	
        return grains
```

---

![bg 70%](./img/bg.png)

# Pillars : stockage de données sensibles !

Pour des raisons de performances, chaque `state` est mis en cache sur le minion à l'exécution de la commande `state.highstate`

Imaginons un state `mysql-users` :

```
frank:
  mysql_user.present:
    - host: localhost
    - password: bobcat
```
Le mot de passe est en clair et va donc se retrouver en cache sur le minion sous `/var/cache/salt/minions/files/base/mysql-users/init.sls`

Cependant depuis n'importe quel autre minion non concerné par ce state, on peut exécuter `salt-call state.sls mysql-users` (pull). Le state sera rappatrié, exécuté, et stocké en cache.

Les pillars ne sont jamais conservés en cache !

---

![bg 70%](./img/bg.png)

# Pillars : stockage de données sensibles !

Alternative avec utilisation d'un pillar :

```
frank:
  mysql_user.present:
    - host: localhost
    - password: {{ salt['pillar.get']('mysq:user:frank') }}
```

Rappel :

**Les pillars sont là pour stocker des informations sensibles et ne sont jamais stockés sur les minions.**

**Chaque pillar présenté à un minion provoque l'ouverture d'un canal dédié et sécurisé entre le master et le minion, que le pillar soit appelé ou non dans un template. Il ne faut donc pas abuser des pillars sous peine de soucis de performances.**


---

![bg 70%](./img/bg.png)

# Démonstration !

#### La production des Jeudis du Libre tient à rassurer le spectateur qu'aucun minion ne sera blessé durant cette présentation.

---

![bg 70%](./img/bg.png)

# Questions ?

---

![bg 70%](./img/bg.png)

# Merci et à tout de suite ! 
# :beer::beer::beer: