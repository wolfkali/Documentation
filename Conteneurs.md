
## Les Mécanismes Clés de l'Isolation des Conteneurs

Les conteneurs ne sont pas des machines virtuelles. Leur isolation repose sur des fonctionnalités du noyau Linux qui créent des environnements isolés pour les processus, en leur donnant l'impression qu'ils sont seuls sur le système.

### 1. Isolation des Points de Montage (Mount Namespaces)

Ce mécanisme crée un **espace de montage unique pour chaque conteneur**. Cela signifie que les modifications du système de fichiers (montage de volumes, création de répertoires, etc.) effectuées à l'intérieur d'un conteneur ne sont pas visibles et n'affectent pas le système de fichiers de l'hôte ou d'autres conteneurs. Chaque conteneur a sa propre vue du système de fichiers, isolée des autres. C'est ce qui permet à un conteneur d'avoir un répertoire `/app` sans qu'il n'entre en conflit avec un `/app` sur l'hôte ou dans un autre conteneur.

---

### 2. Isolation des Processus (PID Namespaces)

Le **PID Namespace** donne à chaque conteneur sa propre vue des identifiants de processus (PID). Dans un conteneur, un processus peut avoir le PID 1 (qui est normalement `init` ou `systemd` sur un système classique), même si sur l'hôte, ce processus a un PID totalement différent et beaucoup plus grand. Cela signifie qu'un processus dans un conteneur ne peut pas voir ou interagir avec les processus d'un autre conteneur ou de l'hôte, garantissant l'isolation des processus.

---

### 3. Isolation Réseau (Network Namespaces)

Chaque conteneur obtient son **propre stack réseau isolé**, comprenant ses propres interfaces réseau, ses propres tables de routage, ses propres règles IPtables, et sa propre plage de ports. Cela signifie qu'un conteneur peut écouter sur le port 80 sans entrer en conflit avec un autre conteneur écoutant également sur le port 80, ou avec un service de l'hôte utilisant ce même port. Les conteneurs peuvent communiquer entre eux ou avec l'extérieur via des ponts virtuels configurés par le moteur de conteneurisation.

---

### 4. Isolation des Utilisateurs (User Namespaces)

Ce mécanisme permet de mapper les IDs d'utilisateurs et de groupes du conteneur à des IDs d'utilisateurs et de groupes différents sur le système hôte. Par exemple, l'utilisateur `root` (UID 0) à l'intérieur d'un conteneur peut correspondre à un utilisateur non privilégié sur l'hôte. Cela **améliore considérablement la sécurité**, car même si un attaquant parvient à obtenir les privilèges `root` à l'intérieur d'un conteneur, il n'aurait que des privilèges limités sur le système hôte.

---

### 5. Isolation des Noms d'Hôte (UTS Namespaces)

Le **UTS Namespace** (Unix Time-sharing System) isole le nom d'hôte et le nom de domaine NIS d'un conteneur. Chaque conteneur peut avoir son propre nom d'hôte, distinct de celui de l'hôte physique, ce qui est utile pour l'identification et la configuration de services spécifiques à l'intérieur du conteneur.

---

### 6. Isolation du Système Inter-processus (IPC Namespaces)

Les **IPC Namespaces** isolent les ressources IPC (Inter-Process Communication) telles que les sémaphores, les files de messages et la mémoire partagée. Cela empêche les processus d'un conteneur d'interférer avec les communications inter-processus d'un autre conteneur ou de l'hôte.

---

### 7. Gestion et Limitation des Ressources (Control Groups - cgroups)

Alors que les namespaces assurent l'**isolation** des conteneurs, les **cgroups (control groups)** sont responsables de la **gestion et de la limitation des ressources**. Ils permettent de définir des plafonds pour l'utilisation du CPU, de la mémoire, de l'E/S disque et du réseau par chaque conteneur. Par exemple, vous pouvez garantir qu'un conteneur ne consommera jamais plus de 50% du CPU total, ou qu'il ne pourra utiliser que 2 Go de RAM, évitant ainsi qu'un conteneur "malveillant" ou défectueux n'accapare toutes les ressources de l'hôte et ne déstabilise les autres services.

---

## Tableau Comparatif Technique des Runtimes de Conteneurs

| Caractéristique           | runc                                                                | crun                                                                          | youki                                                                | gVisor (runsc)                                                                                                      | Kata Containers                                                                                  | Firecracker                                                                                                                                                    | containerd                                                                                | CRI-O                                                                                                        |
| ------------------------- | ------------------------------------------------------------------- | ----------------------------------------------------------------------------- | -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| **Catégorie**             | OCI Runtime (Bas Niveau)                                            | OCI Runtime (Bas Niveau)                                                      | OCI Runtime (Bas Niveau)                                             | Sandboxed Runtime (Bas Niveau)                                                                                      | Sandboxed Runtime (Bas Niveau)                                                                   | Micro-VM Monitor (Hyperviseur)                                                                                                                                 | OCI Runtime (Haut Niveau) / CRI                                                           | OCI Runtime (Haut Niveau) / CRI (pour Kubernetes)                                                            |
| **Développeur Principal** | Docker (maintenant OCI)                                             | Red Hat                                                                       | Suse/Community                                                       | Google                                                                                                              | Open Source (via CNCF)                                                                           | Amazon (AWS)                                                                                                                                                   | CNCF (Cloud Native Computing Foundation)                                                  | Red Hat (via CNCF)                                                                                           |
| **Langage de Prod.**      | Go                                                                  | C                                                                             | Rust                                                                 | Go                                                                                                                  | Go, Rust (agent)                                                                                 | Rust                                                                                                                                                           | Go                                                                                        | Go                                                                                                           |
| **Niveau d'Isolation**    | **Namespaces/Cgroups** (partage noyau hôte)                         | **Namespaces/Cgroups** (partage noyau hôte)                                   | **Namespaces/Cgroups** (partage noyau hôte)                          | **Application Kernel** (interception syscalls en userspace)                                                         | **Micro-VM** (noyau invité séparé via KVM)                                                       | **Micro-VM** (hyperviseur KVM minimaliste)                                                                                                                     | Gère l'exécution via un runtime OCI (ex: `runc`, `crun`).                                 | Gère l'exécution via un runtime OCI (ex: `runc`, `crun`, Kata).                                              |
| **Surcharge (Overhead)**  | Très faible                                                         | Très faible (plus faible que `runc`)                                          | Très faible (comparable à `crun`)                                    | Moyenne (coût des interceptions de syscalls)                                                                        | Moyenne (légère VM overhead)                                                                     | Très faible (optimisé pour rapidité/légèreté)                                                                                                                  | Modérée (gestion de l'état, API)                                                          | Modérée (spécifique K8s, minimaliste)                                                                        |
| **Surface d'Attaque**     | Noyau Linux de l'hôte                                               | Noyau Linux de l'hôte                                                         | Noyau Linux de l'hôte                                                | Réduite (noyau applicatif en userspace)                                                                             | Faible (noyau invité + hyperviseur minimal)                                                      | Très faible (hyperviseur minimal, pas de périphérique traditionnel)                                                                                            | Plus grande que les runtimes bas niveau purs (gestion images, réseau, etc.)               | Plus grande que les runtimes bas niveau purs (gestion images, réseau, etc.), mais optimisée pour K8s.        |
| **Support Rootless**      | Oui (via User Namespaces)                                           | **Excellent** (conçu avec à l'esprit)                                         | Bon                                                                  | Oui (via User Namespaces)                                                                                           | Oui (via User Namespaces)                                                                        | Non directement (implique une VM)                                                                                                                              | Oui (si le runtime bas niveau le supporte et bien configuré)                              | Oui (si le runtime bas niveau le supporte et bien configuré)                                                 |
| **Rapidité de Démarrage** | Très rapide                                                         | Très rapide (souvent plus rapide que `runc`)                                  | Très rapide                                                          | Rapide (mais léger overhead de démarrage de `runsc`)                                                                | Relativement rapide (VM légère)                                                                  | Extrêmement rapide (quelques ms pour la VM)                                                                                                                    | Rapide (appelle un runtime bas niveau)                                                    | Très rapide (optimisé pour les pods Kubernetes)                                                              |
| **Gestion des Images**    | Non (prend un OCI bundle)                                           | Non (prend un OCI bundle)                                                     | Non (prend un OCI bundle)                                            | Non (prend un OCI bundle)                                                                                           | Non (prend un OCI bundle)                                                                        | Non (prend un noyau et un rootfs)                                                                                                                              | **Oui** (pull/push, gestion du stockage des images)                                       | **Oui** (pull d'images OCI, stockage via `containers/storage`)                                               |
| **Gestion du Réseau**     | Configure les namespaces réseau                                     | Configure les namespaces réseau                                               | Configure les namespaces réseau                                      | Gère le réseau au niveau de l'application kernel                                                                    | Utilise un stack réseau invité léger                                                             | Gère le réseau au niveau de l'hyperviseur pour la micro-VM                                                                                                     | Gère les interfaces réseau pour les conteneurs (via CNI)                                  | Gère le réseau pour les pods via CNI                                                                         |
| **Intégration OCI**       | **Implémente OCI Runtime Spec**                                     | **Implémente OCI Runtime Spec**                                               | **Implémente OCI Runtime Spec**                                      | Implémente OCI Runtime Spec (via `runsc`)                                                                           | Implémente OCI Runtime Spec (par son `kata-runtime` qui appelle l'hyperviseur)                   | N'est pas un runtime OCI à lui seul, mais une base pour d'autres runtimes OCI (ex: `containerd` peut lancer Firecracker via `containerd-shim-v2-firecracker`). | **Implémente OCI Image Spec et Distribution Spec**. Utilise un OCI Runtime Spec.          | **Implémente OCI Image Spec et OCI Runtime Spec**. Conforme CRI.                                             |
| **Cas d'Usage Typique**   | Exécution de conteneurs Linux standard.                             | Exécution de conteneurs Linux, privilégié pour la performance et le rootless. | Exécution de conteneurs Linux, alternative sécurisée et performante. | Exécution de charges de travail non fiables (sandbox).                                                              | Exécution de charges de travail sensibles nécessitant une isolation forte (Multi-tenant, CI/CD). | Fonctions serverless (AWS Lambda), isolation pour MicroVMs.                                                                                                    | Runtime par défaut pour Docker, Kubernetes (via CRI), gestion du cycle de vie complet.    | Runtime privilégié pour Kubernetes, axé sur la conformité CRI et la légèreté.                                |
| **Avantages Clés**        | Standard de facto, très stable, largement testé.                    | Rapide, léger, excellent support rootless.                                    | Moderne, écrit en Rust (sécurité/performance), bonne alternative.    | Isolation par userspace, réduit surface d'attaque, compatible K8s/Docker.                                           | Isolation forte au niveau VM, sécurité accrue, compatibilité K8s.                                | Extrêmement rapide/léger, haute densité, idéal serverless, forte isolation.                                                                                    | Solution complète, mature, gère images, supporte plusieurs runtimes bas niveau.           | Léger, spécialement optimisé pour Kubernetes, sans démon lourd.                                              |
| **Inconvénients Clés**    | Moins performant/léger que `crun`/`youki`. Moins de focus rootless. | Moins connu ou adopté que `runc` historiquement.                              | Moins mature que `runc`/`crun`.                                      | Coût en performance pour les syscalls intensives. Compatibilité parfois limitée avec toutes les applications Linux. | Surcharge VM (même légère). Exige KVM. Plus complexe à déployer que `runc`.                      | N'est pas un runtime OCI complet. Nécessite un "shim" (ex: `containerd-shim-v2-firecracker`). Pas d'OS complet dans la VM.                                     | Peut être perçu comme "lourd" comparé à CRI-O pour K8s si seul le runtime est nécessaire. | Exclusif à Kubernetes, ne fournit pas de CLI pour la gestion directe de conteneurs (comme Docker ou Podman). |


## Comparatif Détaillé des Moteurs et Outils de Conteneurs

|Caractéristique|Docker Engine|Podman|containerd|Buildah|Skopeo|LXC/LXD|Rancher Desktop|OrbStack|
|---|---|---|---|---|---|---|---|---|
|**Catégorie**|Moteur de Conteneur (Complet)|Moteur de Conteneur (Complet)|Runtime de Conteneur (Haut Niveau) / CRI|Outil de Construction d'Images|Outil de Gestion d'Images|Technologie de Conteneurisation OS-Level|Solution Desktop (Dev)|Solution Desktop (Dev, macOS)|
|**Architecture**|**Client-Serveur** (démon `dockerd`)|**Daemonless** (pas de démon central)|Daemon (client `ctr` ou via CRI)|Daemonless|Daemonless|Client-Serveur (démon `lxd`)|Client-Serveur (via VM sous Windows/macOS)|Client-Serveur (via VM optimisée)|
|**Développeur Principal**|Docker, Inc.|Red Hat|CNCF (Cloud Native Computing Foundation)|Red Hat|Red Hat|Canonical|SUSE Rancher|OrbStack Team|
|**Langage de Prod.**|Go|Go|Go|Go|Go|Go, C (noyau LXC)|Electron, Go, Rust|Swift, Go, Rust|
|**Niveau d'Abstraction**|Haut niveau (utilisateur final)|Haut niveau (utilisateur final)|Intermédiaire (utilisé par K8s, Docker)|Spécialisé (construction)|Spécialisé (gestion images)|Moyen (conteneurs système)|Haut niveau (interface graphique)|Haut niveau (interface graphique)|
|**Principales Fonctionnalités**|Build, Run, Push/Pull, Swarm, Docker Compose, Volumes, Réseaux, CLI riche.|Build, Run, Push/Pull, Pods (K8s concept), Rootless, Systemd integration, CLI compatible Docker.|Gestion du cycle de vie des conteneurs, gestion des images, stockage, réseau CNI. Utilisé par Kubernetes.|Construction d'images OCI à partir de Dockerfiles ou "scratch".|Transfert, copie, inspection et suppression d'images entre registres/stockages.|Conteneurs système légers (similaires à des VM), gestion de snapshots, migration à chaud.|Fournit Docker Engine/containerd/Kubernetes localement. Interface graphique.|Fast VM, Kubernetes, Docker Engine / containerd pour macOS.|
|**Rootless Containers**|Support via `rootlesskit` (optionnel)|**Support Natif et Excellent** (par défaut)|Oui (si le runtime bas niveau le supporte)|Oui|Oui|Oui (via User Namespaces)|Oui (si le moteur sous-jacent le supporte)|Oui (si le moteur sous-jacent le supporte)|
|**Intégration Kubernetes**|Docker Desktop intègre Kubernetes. Docker Compose peut générer des manifests K8s.|**Intégration forte avec les concepts Kubernetes (Pods)**. Peut générer des manifests YAML K8s.|C'est le runtime par défaut de Kubernetes (implémente CRI).|Non directement (outil de build).|Non directement (outil de gestion images).|Peut être utilisé comme runtime pour K8s avec containerd/CRI-O.|Intègre des versions légères de Kubernetes.|Intègre Kubernetes.|
|**Gestion des Images**|Intégrée (Docker Hub)|Intégrée (tous registres OCI)|Intégrée (tous registres OCI)|**Spécialisé dans la création d'images**|**Spécialisé dans la manipulation d'images**|Non (gère des "images" de conteneurs système, pas OCI)|Intégrée (via le moteur choisi)|Intégrée (via le moteur choisi)|
|**Compatibilité CLI**|Standard `docker` CLI|`podman` CLI (très compatible avec `docker` CLI via alias)|`ctr` CLI (plus bas niveau, orienté runtime) / CRI|`buildah` CLI (spécifique au build)|`skopeo` CLI (spécifique aux images)|`lxc` CLI|Interface graphique et CLI des moteurs sous-jacents|Interface graphique et CLI des moteurs sous-jacents|
|**Sécurité**|Démon root peut être une surface d'attaque.|**Daemonless, Rootless par défaut** : Sécurité intrinsèquement améliorée.|Robuste, mais nécessite des runtimes bas niveau sécurisés.|Conçu pour la sécurité du build.|Axé sur la sécurité des opérations sur les images.|Bonne isolation, mais plus proche de l'OS hôte que les conteneurs Docker/Podman.|Bonnes pratiques d'isolation via VM.|Optimisé sécurité macOS.|
|**Dépendances**|Démon `dockerd`, `containerd`, `runc`|`containers/storage`, `containers/image`, `runc`/`crun` (optionnel)|`runc`/`crun`, CNI plugins|`containers/storage`, `containers/image`|`containers/image`|Noyau Linux, `lxcfs`|`containerd`/Docker Engine, Kubernetes (k3s), VM|VM (Hypervisor.framework)|
|**OS Supportés**|Linux, macOS, Windows|Linux, macOS, Windows|Linux, Windows|Linux, macOS, Windows|Linux, macOS, Windows|Linux (natif)|Linux, macOS, Windows|macOS seulement|
|**Cas d'Usage Typique**|Développement, CI/CD, déploiement d'applications, orchestration simple (Swarm).|Développement sécurisé, déploiements sans démon, environnements RHEL, intégration Systemd, Pods K8s locaux.|Runtime pour Kubernetes, base pour les solutions de conteneurisation.|Création d'images sans Dockerfile, contrôle fin du processus de build.|Gestion centralisée des images, migration entre registres, audit d'images.|Virtualisation légère de systèmes d'exploitation, environnements de test isolés.|Développement local avec Kubernetes ou Docker/containerd.|Développement local rapide et léger sur macOS.|

## Installer Kubernetes
#### Mis à jour de Debian

Cette commande met à jour les paquets Debian et installe les mises à jour disponibles.
 
```bash 
sudo apt update && sudo apt upgrade -y
```
#### Installer Docker

Cette commande télécharge et installe Docker, puis ajoute l'utilisateur actuel au groupe Docker.

```bash 
curl -fsSL https://get.docker.com | sh sudo usermod -aG docker $USER
```
#### Sécuriser Docker

 Ces commandes créent des groupes et des utilisateurs pour sécuriser Docker et configurent le remapping des utilisateurs.
 
```bash
sudo groupadd -g 500000 dockergroup &&
sudo groupadd -g 501000 dockergroup-user &&
sudo useradd -u 500000 -g dockergroup -s /bin/false dockergroup &&
sudo useradd -u 501000 -g dockergroup-user -s /bin/false dockergroup-user
echo "dockergroup:500000:65536" >> /etc/subuid &&
echo "dockergroup:500000:65536" >> /etc/subgid

echo "
{
  \"users-remap\": \"default\"
}
" > /etc/docker/daemon.json

```

#### Désactiver le swap sur le moment

Cette commande désactive le swap temporairement.
 
```bash
sudo swapoff -a
```
#### Désactiver le swap au démarrage

 Cette commande ouvre le fichier ```/etc/fstab``` pour édition. Vous devez commenter ou supprimer les lignes de swap pour désactiver le swap au démarrage.
 
```bash 
sudo vim /etc/fstab
```
#### Mise en place du dépôt APT

 Ces commandes mettent à jour les paquets APT, installent les dépendances nécessaires, ajoutent la clé GPG du dépôt Kubernetes, et ajoutent le dépôt Kubernetes à la liste des sources APT.

```bash
sudo apt update && sudo apt install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo add-apt-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

#### Installation des binaires Kubernetes

 Ces commandes installent les binaires Kubernetes (kubelet, kubeadm, kubectl, kubernetes-cni) et activent le service kubelet.

```bash 
sudo apt install -y kubelet kubeadm kubectl kubernetes-cni sudo systemctl enable kubelet
```

#### Initialisation sur la master

 Cette commande initialise le nœud master Kubernetes avec l'adresse IP spécifiée, le nom du nœud, et le CIDR du réseau pod.
 
```bash
sudo kubeadm init --apiserver-advertise-address="My IP" --node-name $HOSTNAME --pod-network-cidr=10.244.0.0/16
```

#### Supprimer containerd

 Ces commandes suppriment le fichier de configuration containerd et redémarrent le service containerd.
 
```bash
sudo rm /etc/containerd/config.toml sudo systemctl restart containerd
```

#### Création du fichier de configuration

Ces commandes créent le répertoire de configuration Kubernetes, copient le fichier de configuration admin.conf dans le répertoire de l'utilisateur, et ajustent les permissions.

```bash
mkdir -p $HOME/.kube sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
#### Ajouter pod pour la gestion du réseau internet

Cette commande active la gestion des règles iptables pour les ponts réseau.

```bash 
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

Pour plus de détails, vous pouvez consulter la documentation officielle de Kubernetes :

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

