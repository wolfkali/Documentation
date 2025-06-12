Il existe plusieurs méthodes pour effectuer la vérification de domaine avec **Certbot**, et l'une d'elles peut être plus adaptée à votre situation, surtout si vous ne souhaitez pas arrêter les services ou si vous ne pouvez pas utiliser le mode `standalone` avec le port 80.

Voici les principales méthodes de vérification que vous pouvez utiliser avec Certbot :
#### **Méthode HTTP-01 (Challenge HTTP via un serveur web)**

Cela nécessite d'avoir un serveur web déjà en place sur **port 80** ou d'avoir la possibilité de configurer un serveur web pour accepter les requêtes HTTP sur ce port. Si vous utilisez déjà un serveur web comme **Nginx** ou **Apache**, vous pouvez simplement laisser Certbot gérer cette validation automatiquement.

- **Pré-requis** : Le port 80 doit être ouvert et accessible de l'extérieur.
- **Commande** :

```bash
sudo certbot --webroot -w /var/www/html -d domain.net
```

- Le flag `--webroot` indique à Certbot de placer un fichier de validation dans le répertoire webroot (`/var/www/html` dans cet exemple).
    - Certbot va automatiquement créer le fichier de validation à cet emplacement, et Let's Encrypt vérifiera l'existence de ce fichier en accédant à `http://domain.net/.well-known/acme-challenge/<token>`.

#### **Méthode DNS-01 (Challenge DNS)**

Si vous ne pouvez pas utiliser HTTP-01 (par exemple, si le port 80 est déjà utilisé ou si vous ne voulez pas arrêter votre serveur web), vous pouvez utiliser le challenge **DNS-01**, qui consiste à ajouter un enregistrement DNS spécifique pour valider la possession du domaine.

- **Pré-requis** : Vous devez être capable de modifier les enregistrements DNS de votre domaine.
- **Commande** :

```bash
sudo certbot -d domain.net --manual --preferred-challenges dns certonly
```

- Cette commande vous demandera de créer un enregistrement **TXT** dans vos paramètres DNS pour le domaine `domain.net`.
- Après avoir ajouté l'enregistrement TXT (généralement quelque chose comme `"_acme-challenge.domain.net"`), vous devrez attendre que les changements DNS se propagent, puis indiquer à Certbot que vous avez terminé.

Exemple de message que vous recevrez :

```bash
Please create a DNS TXT record at:
_acme-challenge.domain.net

with the following value:
<token_value>
```

Une fois l'enregistrement DNS ajouté, vous devrez valider la demande en appuyant sur **Entrée** dans Certbot.

#### **Méthode TLS-ALPN-01 (Challenge TLS)**

Cette méthode peut être utilisée si vous avez déjà un serveur web qui écoute sur **port 443** et que vous souhaitez que Certbot valide le domaine sans utiliser HTTP ou DNS. Elle est moins courante, mais elle est parfois utilisée dans des configurations spécifiques (comme lorsqu'on utilise un reverse proxy ou un autre mécanisme pour gérer les connexions HTTPS).

- **Pré-requis** : Le port 443 doit être accessible pour les vérifications TLS.
- **Commande** :

```bash
sudo certbot --preferred-challenges tls-alpn-01 -d domain.net
```

#### **Utilisation de Certbot avec un Reverse Proxy**

Si vous utilisez **HAProxy**, **Nginx**, ou un autre reverse proxy et que vous souhaitez obtenir un certificat sans arrêter d'autres services, vous pouvez aussi utiliser Certbot avec la méthode `standalone` en associant le proxy pour rediriger correctement le trafic vers Certbot pendant la validation.

```bash
sudo certbot certonly --standalone -d domain.net --http-01-port 8080
```

Configurez **HAProxy** ou **Nginx** pour rediriger temporairement le trafic HTTP (port 80) vers ce port, pendant que Certbot effectue la validation.
### Exemple pour un reverse proxy :

1. Utilisez `standalone` pour effectuer la validation sur un port différent (par exemple, un port temporaire, comme le port 8080) :

#### Installation de **Let's Encrypt** (Certbot)

Certbot est un outil de Let's Encrypt permettant de générer des certificats SSL gratuits. Installez-le avec la commande suivante :

```bash
sudo dnf install certbot
```

#### Obtenir un certificat SSL

Pour générer un certificat SSL pour votre domaine (par exemple `votredomaine.com`), utilisez la commande

```bash
sudo certbot certonly --standalone -d domain.net --http-01-port 8080
```

Certbot générera les certificats SSL et les stockera dans `/etc/letsencrypt/live/votredomaine.com/`.
#### Automatiser la renouvellement du certificat

Certbot configure automatiquement une tâche cron pour renouveler les certificats. Vous pouvez vérifier la tâche cron en exécutant :

```bash
sudo systemctl list-timers
```

Vous pouvez aussi tester manuellement le renouvellement en exécutant :

```bash
sudo certbot renew --dry-run
```

