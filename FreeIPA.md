#### Installer serveur Free IPA

Il faut que la pile IPv6 soit activée dans le kernel même si vous n’utilisez pas IPv6 sur vos interfaces, ne pas utiliser `ipv6.disable=1` dans `sysctl`, préférez `ipv6.disable_ipv6=1`

```bash
ip a
```

Vérifiez que le FQDN de votre machine soit correctement indiqué dans le fichier `/etc/hostname` et qu’il résout correctement avec `dig` `yum install bind-utils`, dans notre exemple il s’agira de freeipa0.idm.demo.local

```bash
dig +short freeipa0.idm.demo.local A
172.16.110.60
```

Si, à ce stade, la résolution inverse ne fonctionne pas, ce n’est pas grave, l’installation de freeipa créera une zone DNS appropriée ; si vous choisissez de ne pas utiliser le DNS de freeipa, il faudra gérer ce point vous même sur votre DNS.

Éditez votre fichier `/etc/hosts` pour qu’il ressemble à ceci

```bash
cat /etc/hosts
172.16.110.60 freeipa0.idm.demo.local freeipa0
```

La première ligne a été ajoutée.
Ouvrez les ports nécessaires du firewall :

```bash
firewall-cmd --permanent --add-port={80/tcp,443/tcp,389/tcp,636/tcp,88/tcp,464/tcp,53/tcp,88/udp,464/udp,53/udp,123/udp}
firewall-cmd --reload
```

Ports TCP :

- 80, 443 : HTTP/HTTPS
- 389, 636 : LDAP/LDAPS
- 88, 464 : kerberos
- 53 : Bind (DNS)

Ports UDP :

- 88, 464 : kerberos
- 53 : Bind (DNS)
- 123 : ntp


Certaines machines virtuelles risquent de manquer rapidement d’entropie pour les opérations de chiffrement de freeipa, il faudra dans ce cas installer et activer rng-tools

```bash
yum install rng-tools
systemctl start rngd
systemctl enable rngd
```

Il n’y as pas de dépôts particuliers à configurer sous CentOS 7 :

```bash
sudo dnf install ipa-server ipa-server-dns
```

Vérifiez les options disponibles avec ipa-server-install –help ; dans notre exemple nous allons faire une installation complète sur idm.demo.local. Il est systématiquement conseillé de déclarer FreeIPA sur un nouveau sous-domaine pour éviter tout conflit potentiel avec un autre serveur kerberos présent (AD…) (voir [https://www.freeipa.org/page/DNS#Caveats](https://www.freeipa.org/page/DNS#Caveats)). Si vous n’en avez pas, il est tout a fait possible d’installer idm directement sur votre domaine, voici la commande que nous allons utiliser

```bash
ipa-server-install -U -p MotDePasse1 -a MotDePasse2 --ip-address=172.16.110.60 -n idm.demo.local -r IDM.DEMO.LOCAL --hostname=freeipa0.idm.demo.local --setup-dns --auto-forwarders --auto-reverse
```

xplications des options :

- -U : mode unattended, sans intervention de l’utilisateur (à désactiver en cas de soucis)
- -p MotDePasse1 : Mot de passe du directory manager, admin ultime côté LDAP
- -a MotDePasse2 : Mot de passe de l’administrateur freeipa/kerberos
- –ip-address= : l’ip de votre serveur
- -n : le domaine géré par freeipa
- -r : le realm kerberos, par convention le nom du domaine en majuscule
- –hostname= : fqdn du serveur
- –setup-dns : freeipa intègre la gestion de son propre dns si vous n’en possédez pas. Il est possible ne pas utiliser l’option pour gérer votre propre DNS
- –auto-forwarders : création automatique des forwarders en se basant sur le fichier /etc/resolv.conf du serveur, pour la résolution des noms internet
- –auto-reverse : création automatique de la zone reverse DNS

Une fois les enregistrements ajoutés, vérifiez qu’ils sont bien pris en compte :

```bash
dig +short srv _kerberos-master._tcp.idm.demo.local.
0 100 88 freeipa0.idm.demo.local.
```

Vérifiez le bon fonctionnement du serveur avec la commande **kinit** et **ipa user-find**

```bash
ipa user-find admin

--------------
1 user matched
--------------
User login: admin
Last name: Administrator
Home directory: /home/admin
Login shell: /bin/bash
Principal alias: admin@IDM.DEMO.LOCAL
UID: 407200000
GID: 407200000
Account disabled: False
----------------------------
Number of entries returned 1
----------------------------
```

#### Redirection DNS

Ouvrez le fichier de configuration de BIND. Il pourrait se trouver dans le répertoire `/etc/named.conf`

```bash
sudo nano /etc/named.conf
```

Dans ce fichier, vous pouvez ajouter ou modifier la section des `forwarders` en ajoutant les serveurs DNS externes vers lesquels les requêtes seront envoyées.

Voici un exemple de configuration pour le DNS forwarding :

```bash
options {
    directory "/var/named";  // Répertoire des fichiers de zone DNS

    // Serveurs DNS de forwarding
    forwarders {
        1.1.1.1;  // DNS Cloudflare
        1.0.0.1;  // DNS Cloudflare
    };

    // Autres options de configuration
    allow-query { any; };
    recursion yes;
};
```

Une fois que vous avez ajouté les serveurs de forwarding dans le fichier de configuration BIND, il est nécessaire de redémarrer le service BIND pour que les modifications prennent effet.

```bash
sudo systemctl restart named
```

Cela va redémarrer le serveur DNS BIND et appliquer les paramètres de forwarding que vous avez définis.

#### Ajouter un serveur au Domaine FreeIPA

**Vérifier la connectivité réseau** : Assurez-vous que le PC que vous essayez d'ajouter au domaine peut accéder au serveur **FreeIPA**. Cela inclut la résolution DNS correcte du serveur FreeIPA et la connectivité réseau (pas de firewall ou de restrictions).

- Testez la résolution DNS du serveur FreeIPA avec `dig` ou `nslookup` :

```bash
nslookup lx-ipa-01.sousou.loc
```

- Cette commande doit renvoyer l'adresse IP correcte du serveur FreeIPA.
- Vérifiez également la connectivité en pingant l'adresse IP du serveur FreeIPA :

```bash
ping lx-ipa-01.sousou.loc
```

**Vérifier que `realmd` et les paquets nécessaires sont installés** : Si vous n'avez pas installé les paquets nécessaires, cela pourrait également entraîner l'échec de la commande. Assurez-vous que `realmd`, `sssd`, et `krb5-workstation` sont installés sur votre machine.

```bash
sudo dnf install freeipa-client
```

**Vérifier la disponibilité du service FreeIPA** : Assurez-vous que le serveur FreeIPA est en ligne et répond aux requêtes. Sur le serveur FreeIPA, vous pouvez vérifier les services nécessaires (comme le service **IPA** et **Kerberos**). Vérifiez également que le serveur est accessible via **LDAP** et **Kerberos**.

Vous pouvez vérifier le statut des services sur le serveur FreeIPA avec :

```bash
sudo ipa service-status
```

Vous pouvez renommer la machine 

```bash 
sudo hostnamectl set-hostname nouveau-nom
```

Assurez-vous que le fichier `/etc/hosts` contient une entrée pour votre nom d'hôte et l'adresse IP associée. Ajoutez (ou modifiez) la ligne suivante dans le fichier `/etc/hosts`

```bash
127.0.1.1   hostname.sousou.loc hostname
```

Si les tests DNS et réseau sont corrects, réessayez la commande `realm discover`

```bash
sudo realm discover lx-ipa-01.sousou.loc
```

Si la découverte du domaine réussit, vous pouvez alors joindre le domaine FreeIPA avec la commande :

```bash
sudo realm join --user=admin lx-ipa-01.sousou.loc
```

Si malgré tout cela vous rencontrez toujours des difficultés, il pourrait être utile de vérifier les journaux du serveur FreeIPA pour voir s'il y a des problèmes de configuration ou d'accès. Vous pouvez consulter les journaux avec :

```bash
sudo journalctl -u ipa
```

#### **Dissocier une machine du domaine FreeIPA**

Si vous avez une machine sous Linux ou macOS qui est jointe à un domaine FreeIPA, vous pouvez la retirer du domaine en utilisant `realm` ou les commandes appropriées en fonction de la distribution.

###### Utiliser la commande `realm leave`

La commande `realm leave` permet de retirer la machine du domaine.

1. **Vérifier l'état actuel de la machine** (facultatif) :

```bash
realm list
```

Cette commande vous montre à quel domaine la machine est actuellement jointe.

**Détacher la machine du domaine** : Pour quitter le domaine, utilisez la commande suivante (remplacez `lx-ipa-01.sousou.loc` par le nom de votre domaine) :

```bash
sudo realm leave lx-ipa-01.sousou.loc
```

- Vous serez invité à fournir des informations d'authentification (nom d'utilisateur et mot de passe d'un administrateur de domaine).
    
- **Vérification** : Après avoir exécuté cette commande, vous pouvez vérifier que la machine a bien quitté le domaine en utilisant de nouveau `realm list`, qui devrait indiquer que la machine n'est plus dans le domaine.

```
ipa-client-install -U --domain=sousou.loc --realm=SOUSOU.LOC --server=lx-ipa-02.sousou.loc --mkhomedir -p admin -w 123456789
```


