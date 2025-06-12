#### Vérifier le disque

Avant de formater, il est important de vérifier que le disque **/dev/sdb** est bien celui que vous voulez formater. Utilisez la commande suivante pour lister les disques et leurs partitions :

```bash
sudo lsblk
```

Cette commande vous permet de vérifier le disque **/dev/sdb** et s'il est bien connecté.

#### Démonter le disque

Vous devrez d'abord le démonter avant de pouvoir le formater. Utilisez la commande suivante pour démonter le disque :

```bash
sudo umount /dev/sdb*
```

Cela va démonter toutes les partitions du disque. Si vous avez plusieurs partitions, cette commande les démontera toutes.

#### Créer une nouvelle table de partition**

Si vous voulez repartir de zéro et créer une nouvelle table de partition (ce qui effacera toutes les partitions existantes), utilisez `parted` ou `gdisk` (si vous travaillez avec des partitions GPT).

Pour créer une nouvelle table de partitions de type **GPT** :

```bash
sudo parted /dev/sdb mklabel gpt
```

Cela effacera toutes les partitions existantes sur le disque **/dev/sdb**.

#### Créer une nouvelle partition

Si vous n'avez pas encore de partition sur **/dev/sdb**, vous pouvez en créer une avec `parted` ou `fdisk`.

Pour créer une partition entière de type primaire sur le disque **/dev/sdb** avec `parted` :

```bash
sudo parted /dev/sdb mkpart primary xfs 0% 100%
```

Cela crée une partition unique utilisant tout l'espace disponible.
#### Formater la partition en XFS

Une fois la partition créée (par exemple **/dev/sdb1**), vous pouvez la formater avec le système de fichiers XFS en utilisant la commande `mkfs.xfs`.

Formatez la partition avec :

```bash
sudo mkfs.xfs /dev/sdb1
```

Cela formate la partition **/dev/sdb1** en **XFS**.
#### Monter la partition

Après avoir formaté le disque, vous pouvez le monter pour y accéder. Pour cela, vous pouvez créer un point de montage (par exemple `/mnt/disque`), puis monter la partition dessus :

1. Créez un répertoire de montage :

```bash
sudo mkdir /mnt/disque
```

Montez la partition **/dev/sdb1** sur ce répertoire :

```bash
sudo mount /dev/sdb1 /mnt/disque
```

#### Ajouter un montage automatique

Si vous souhaitez que la partition se monte automatiquement au démarrage, vous pouvez l'ajouter au fichier `/etc/fstab`.

Ouvrez le fichier `/etc/fstab` avec un éditeur de texte :

```bash
sudo vim /etc/fstab
```

Ajoutez la ligne suivante à la fin du fichier (en supposant que vous avez monté la partition sur `/mnt/disque`) :

```bash
/dev/sdb1    /mnt/disque    xfs    defaults    0    0
```

Enregistrez et quittez l'éditeur.

## Comparatif Approfondi des Systèmes de Fichiers

| Système de Fichier | Journalisation                                                     | Limite Taille Fichier            | Limite Taille Volume       | Fonctionnalités Clés Détaillées                                                                                                                                                                                                                                                    | Points Forts Techniques                                                                                                                                                                                                                                    | Points Faibles Techniques                                                                                                                                                                                                                                                                                                                                                                 | Plateformes Principales                                                                                   |
| ------------------ | ------------------------------------------------------------------ | -------------------------------- | -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| **NTFS**           | Oui (Métadonnées et données)                                       | 16 EiB                           | 256 TiB (Windows 10/11)    | **Journalisation transactionnelle**, Listes de contrôle d'accès (ACL) pour sécurité granulaire, Compression/Chiffrement (EFS), Support des liens physiques/symboliques, Quotas de disques, Volume Shadow Copy Service (VSS).                                                       | Très robuste pour la récupération post-crash, excellente sécurité au niveau du système de fichiers, bon pour les serveurs Windows et les environnements d'entreprise. Performances élevées pour les gros fichiers.                                         | Fragmentation importante sur des écritures/suppressions fréquentes de petits fichiers. Overhead de journalisation peut affecter légèrement les performances sur certains workloads. Compatibilité native limitée hors Windows.                                                                                                                                                            | Windows (principal), serveurs Windows.                                                                    |
| **FAT32**          | Non                                                                | 4 GiB                            | 2 TiB                      | Structure simple de table d'allocation de fichiers. Pas de fonctionnalités de sécurité, de journalisation ou d'intégrité de données avancées. Pas de support de permissions.                                                                                                       | Compatibilité maximale et universelle avec presque tous les OS et appareils. Léger et simple à implémenter.                                                                                                                                                | Vulnérable à la corruption de données en cas de coupure de courant. Performances limitées. Ne supporte pas les grands fichiers ni les grandes partitions, ce qui le rend obsolète pour les disques durs modernes.                                                                                                                                                                         | Clés USB, cartes mémoire, appareils photo, consoles de jeux, compatibilité inter-OS.                      |
| **exFAT**          | Non                                                                | 16 EiB                           | 128 PiB                    | Version étendue de FAT32 avec support des grandes tailles de fichiers/volumes. Pas de journalisation, pas de sécurité avancée. Optimisé pour la mémoire flash.                                                                                                                     | Compromis idéal entre compatibilité universelle et support des grands fichiers/volumes pour les médias amovibles. Faible surcharge, bon pour les cartes SD/clés USB.                                                                                       | Manque de robustesse (pas de journalisation) comparé à NTFS/ext4. Pas de gestion des permissions ou d'intégrité des données.                                                                                                                                                                                                                                                              | Clés USB/cartes SD de grande capacité, échange de fichiers entre Windows/macOS/Linux.                     |
| **APFS**           | Oui (Métadonnées)                                                  | 8 EiB                            | 8 EiB (par conteneur APFS) | **Copy-on-Write (CoW)** pour l'efficacité des snapshots et des clones. Chiffrement complet du disque natif. Partage d'espace de stockage entre les volumes. Checksums pour l'intégrité des métadonnées. Optimisé pour les SSD et la mémoire flash.                                 | Performances exceptionnelles sur SSD, snapshots rapides et efficaces, robustesse accrue grâce à CoW et checksums. Très adapté aux architectures Apple modernes.                                                                                            | Exclusif à l'écosystème Apple. Pas de journalisation des données par défaut (seulement métadonnées), bien que la nature CoW minimise les risques. Complexité pour les utilisateurs non habitués.                                                                                                                                                                                          | macOS, iOS, tvOS, watchOS.                                                                                |
| **ext4**           | Oui (Journalisation complète : métadonnées et données optionnelle) | 16 TiB (fichier), 1 EiB (volume) | 1 EiB                      | **Journalisation robuste**, Allocation étendue (extents) pour des performances améliorées sur les grands fichiers, Pre-allocation, Delayed allocation, Support des millions d'inodes (nombre de fichiers), Groupes de blocs.                                                       | Très stable, mature et performant sur Linux. Robuste contre la corruption. Bonne gestion des petits et grands fichiers. Système de fichiers par défaut et bien éprouvé sur Linux.                                                                          | Manque de fonctionnalités avancées de CoW, snapshots natifs (bien que LVM puisse le faire), ou checksums intégrés au niveau du FS comparé à ZFS/Btrfs. Fragmentation possible.                                                                                                                                                                                                            | Linux (principal), serveurs Linux.                                                                        |
| **XFS**            | Oui (Métadonnées)                                                  | 8 EiB (fichier), 8 EiB (volume)  | 8 EiB                      | Optimisé pour la **scalabilité et les performances sur de très grands systèmes de fichiers** (millions de téraoctets). Allocation d'espace basée sur des extentions, journaux par allocation. Support des attributs étendus. Grande efficacité d'écriture sur les grands fichiers. | **Excellent pour les workloads d'écriture intensive** et les très grands fichiers (bases de données, médias). Très faible fragmentation sur les grands volumes. Bonne résilience grâce à la journalisation des métadonnées. Très bonne performance en E/S. | Moins efficace pour les systèmes avec un très grand nombre de petits fichiers (par rapport à ext4). La récupération d'un système de fichiers XFS corrompu peut être plus complexe que pour ext4. La journalisation ne couvre pas les données par défaut, ce qui peut entraîner une perte de données récentes non synchronisées après un crash si l'application n'a pas fait de `fsync()`. | Linux (serveurs de fichiers, bases de données, HPC, stockage de médias), utilisé sur IRIX historiquement. |
| **ZFS**            | Oui (Transactionnelle CoW)                                         | 16 EiB (fichier), 256 ZiB (pool) | 256 ZiB                    | **Pools de stockage transactionnels**, Copy-on-Write (CoW), **Checksums de bout en bout** (intégrité des données), Snapshots et clones illimités, Compression et déduplication des données natives, Auto-réparation (scrubbing), RAID logiciel intégré (RAID-Z).                   | **Intégrité des données inégalée**, flexibilité du stockage, robustesse extrême, fonctionnalités avancées pour la gestion des données. Idéal pour les systèmes où la perte de données est inacceptable.                                                    | Consommation de RAM élevée (surtout pour la déduplication), complexité de configuration et de gestion pour les non-experts. Moins de support natif sur les OS grand public (souvent via FUSE ou sur des OS spécifiques).                                                                                                                                                                  | Solaris, FreeBSD, Illumos, OpenZFS sur Linux (via module noyau).                                          |
| **Btrfs**          | Oui (Transactionnelle CoW)                                         | 16 EiB                           | 16 EiB                     | **Copy-on-Write (CoW)**, Snapshots et sous-volumes, Checksums des données et métadonnées, RAID logiciel intégré, Compression transparente, Déduplication (expérimental), Balanceur de données.                                                                                     | Offre beaucoup de fonctionnalités avancées de ZFS sur Linux. Très flexible pour la gestion de l'espace disque. Idéal pour les installations Linux avec des besoins de snapshots et de résilience.                                                          | Encore considéré comme moins mature que ZFS pour les environnements de production les plus critiques par certains, bien que sa stabilité s'améliore constamment. La documentation peut être moins complète que celle des systèmes plus anciens.                                                                                                                                           | Linux (système de fichiers racine, NAS, serveurs de développement).                                       |


## Tableau Comparatif Détaillé des Tables de Partitionnement : MBR vs GPT

| Caractéristique                            | Master Boot Record (MBR)                                                                                                                        | GUID Partition Table (GPT)                                                                                                                                                                          |
| :----------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Localisation de la table**               | Début du disque (premier secteur)                                                                                                               | Au début et à la fin du disque (pour redondance)                                                                                                                                                    |
| **Nombre de Partitions**                   | **4 partitions primaires maximum** ou 3 primaires + 1 étendue (contenant des partitions logiques illimitées)                                    | **128 partitions primaires par défaut** (configurable à plus)                                                                                                                                       |
| **Taille Maximale du Disque Supportée**    | **2 Tio (Téraoctets)**                                                                                                                          | **9.4 Zio (Zettaoctets)** (théoriquement bien plus, dépendant de l'OS)                                                                                                                              |
| **Taille Maximale de Partition Supportée** | **2 Tio**                                                                                                                                       | **9.4 Zio** (théoriquement bien plus)                                                                                                                                                               |
| **Identifiant de Partition**               | Code Hexadécimal de 1 octet (ex: `0x07` pour NTFS)                                                                                              | **GUID (Globally Unique Identifier)** de 16 octets (UUID)                                                                                                                                           |
| **Bootloader**                             | **Premier stage du bootloader** dans le MBR (code de démarrage, table de partitions).                                                           | **Système UEFI** requis (souvent via un petit "EFI System Partition" - ESP) qui gère le boot.                                                                                                       |
| **Robustesse / Redondance**                | **Point de défaillance unique** (le MBR est stocké à un seul endroit). Corruption du MBR = perte de l'accès aux partitions.                     | **Redondance multiple** : Table de partitions primaire et secondaire (backup) à l'autre extrémité du disque. CRC32 (Cyclic Redundancy Check) pour vérifier l'intégrité des en-têtes et de la table. |
| **Vérification d'Intégrité**               | Aucune (pas de checksums)                                                                                                                       | **CRC32 checksums** pour l'en-tête de la GPT et pour la table de partitions elle-même.                                                                                                              |
| **Secteurs de Démarrage Protégés**         | Aucun mécanisme de protection.                                                                                                                  | **"Protective MBR"** : Un faux MBR est créé au début du disque GPT pour empêcher les outils MBR de reconnaître le disque comme vide et d'écraser la table GPT.                                      |
| **Utilisation Historique**                 | **PC IBM et leurs compatibles** depuis les années 1980. Système de partitionnement traditionnel.                                                | Introduit avec le **standard UEFI** (Unified Extensible Firmware Interface) au début des années 2000.                                                                                               |
| **Compatibilité OS**                       | **Universel** (Windows, Linux, macOS, anciens systèmes).                                                                                        | **Windows (Vista 64-bit et plus récent), macOS (Intel et Apple Silicon), Linux (noyau 2.6.25 et plus récent)**. Nécessite généralement un firmware UEFI pour le démarrage.                          |
| **Architecture de Démarrage**              | **BIOS hérité (Legacy BIOS)**                                                                                                                   | **UEFI (Unified Extensible Firmware Interface)**                                                                                                                                                    |
| **Outils de Gestion**                      | `fdisk`, `cfdisk`, `parted` (Linux), Gestion des disques (Windows)                                                                              | `gdisk`, `parted` (Linux), Gestion des disques (Windows), `diskutil` (macOS)                                                                                                                        |
| **Sécurité du Démarrage**                  | Aucune fonctionnalité de sécurité de démarrage intégrée.                                                                                        | Supporte le **Secure Boot** (démarrage sécurisé) via UEFI, empêchant le chargement de bootloaders non signés.                                                                                       |
| **Utilisation Typique**                    | Anciens ordinateurs, disques durs de petite capacité (moins de 2 To), systèmes nécessitant une compatibilité maximale avec des OS très anciens. | Disques durs modernes (plus de 2 To), SSD, systèmes d'exploitation récents (Windows 64-bit, macOS, Linux modernes), serveurs, stations de travail.                                                  |

