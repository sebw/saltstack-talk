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

#### Contributeur à différents projets Open Source, principalement Salt ces jours-ci

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

- CFEngine (1993)
- Puppet (2005)
- Chef (2009)
- ...

---

![bg 70%](./img/bg.png)

# Les petits "nouveaux"

- Salt (2011)
- Ansible (2012)
- ...

---

![bg 70%](./img/bg.png)

# Pourquoi Salt ?

#### En 2010, l'ETNIC avait environ 150 serveurs Linux

- DNS, relais SMTP, forward et reverse proxy, etc.
- Des applications métiers : GED, SAP, développements internes, etc.

---

![bg 70%](./img/bg.png)

# Pourquoi Salt ?

#### Les problèmes constatés dès les premiers jours

- Authentification `root` par mot de passe...
- ...connu de presque tout le monde (devs/ops)
- Des distributions différentes (SuSE Enterprise, OpenSuse, Debian, RedHat, RedHat Enterprise)
- Des applications installées à la main (compilées) et jamais mises à jour
- Des NTP, SMTP, DNS mal ou pas configurés
- Des inconsistences entre environnements d'un même projet (`service alfresco start` en ACC, `service tomcat start` en PRD)
- Installation OS entièrement à la main depuis l'ISO qui traine sur un PC
- Un équivalent temps plein pour remettre de l'ordre dans tout ça...

---

![bg 70%](./img/bg.png)

# Pourquoi pas... Puppet ?

#### Fin 2011 je participe à une formation Puppet

- à l'époque Puppet est toujours en mode "pull"
- la recommandation du formateur pour une infrastructure telle que la notre est un pull toutes les 30 minutes et au moins deux masters (charge).
- la gestion de configuration, le remote execution et facter sont des composants séparés
- la syntaxe n'est pas très claire (Ruby DSL)

---

# Pourquoi Salt ?

![bg 70%](./img/bg.png)

#### Premiers tests en 2013 avec une dizaine de serveurs

- gestion de configuration simple pour commencer (SMTP, NTP)
- remote execution (yum upgrade)
- récupération d'informations sur le parc (facts)

---

![bg 70%](./img/bg.png)

# Pourquoi Salt ?

#### 260 serveurs gérés en décembre 2016 :thumbsup:
#### Un nouveau serveur complètement provisionné en moins de 10 minutes
#### Tout le code dans un dépôt Gitlab

---

![bg 70%](./img/bg.png)

# Les avantages ?

#### Event-driven
#### Ecrit en Python (aucune connaissance requise, mais c'est un plus)
#### YAML et Jinja
#### Modèle client/serveur (`minion`/`master`)
#### Mode push et pull
#### Gestion de configuration, remote execution, facts dans un seul package
#### De l'installation à un premier fichier de configuration géré ? 
## 10 minutes

---

![bg 70%](./img/bg.png)

# Les désavantages

#### Installation d'un agent (salt-minion)
#### Et des dépendances (Python, ZeroMQ, msgpack)

---

![bg 70%](./img/bg.png)

# Fonctionnement de base

#### Mode client/server

**TODO** --> image

- Les minions restent connectés constamment au master (message bus ZeroMQ)
- Le master doit accepter la clé d'un nouveau minion (PKI)

---

![bg 70%](./img/bg.png)

# Fonctionnement de base

#### Terminologie

`master` : serveur de gestion
`minion` : serveur géré
`state` : état de configuration (package installé, fichier configuré, etc.)
`grain` : information sur un minion
`pillar` : information stockée sur le master à disposition des minions
`top.sls` : fichier de définition des `states` ou `pillars` sur les minions
`init.sls` : fichier de départ de tout `states` ou `pillars`
`reactors` : action en réaction à un évenement sur le bus

---

![bg 70%](./img/bg.png)

# Installation

Un master contrôle plusieurs minions.

#### Sur le master :

`yum install salt-master`

Configuration sous `/etc/salt/master` et `/etc/salt/master.d/`

#### Sur les minions :

`yum install salt-minion`

Configuration sous `/etc/salt/minion` et `/etc/salt/minion.d/`

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

On vérifie si il répond bien :

```yaml
[root@salt-master ~]# salt 'salt-minion' test.ping
salt-minion:
    True
```

Ceci n'est pas un ping ICMP ! C'est juste une fonction Python exécutée sur le minion qui fait `return True`.

---

![bg 70%](./img/bg.png)

# Avant de commencer

Le code est au service de l'infrastructure

Les bonnes pratiques de développement s'appliquent

Système de contrôle de version (SVN, Git, etc.)

## KISS : keep it simple, stupid

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
/srv/salt/states/motd/motd
/srv/salt/states/top.sls
```

---

![bg 70%](./img/bg.png)

# top.sls

```yaml
[root@salt-master /]# cat /srv/salt/states/top.sls
base:
  '*':
    - motd
```

Dans l'environnement `base`, sur tous les `minions`, appliquer le `state` motd.

---

![bg 70%](./img/bg.png)

# Notre premier state : motd

MOTD : "message of the day", message affiché à la connexion au serveur

Notre manifest d'installation :

```yaml
[root@salt-master ~]# cat /srv/salt/states/motd/init.sls
motd:                            <-- ID unique
  file.managed                   <-- module.fonction
    - name: /etc/motd            <-- le fichier géré
    - source: salt://motd/motd   <-- template à utiliser
    - template: jinja            <-- type de template
```

`salt://` est un genre de moteur HTTP embarqué dans Salt, on peut spécifier d'autres types de sources : `http://`, `https://`, `ftp://`, etc.

Il existe plusieurs types de template, on peut utiliser du Python pur pour des choses plus avancées.

---

![bg 70%](./img/bg.png)

# Notre premier state : motd

Le template de configuration :

```jinja
[root@salt-master ~]# cat /srv/salt/states/motd/motd
Bonjour et bienvenue sur {{ grains['fqdn'] }}
Mon master est {{ grains['master'] }}
```

---

![bg 70%](./img/bg.png)

# Notre premier state : motd

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
Liste non exhaustive.

---

![bg 70%](./img/bg.png)

# Notre premier state : motd

Appliquons la configuration :

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

# Notre premier state : motd

Vérifions sur notre minion :

```bash
[root@salt-minion ~]# cat /etc/motd
Bonjour et bienvenue sur salt-minion
Mon master est 10.211.55.26
```
