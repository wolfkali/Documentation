### Installation de **HAProxy**

Commencez par installer HAProxy en utilisant `dnf` :

```bash
sudo dnf install haproxy
```

#### Démarrer et activer HAProxy

Ensuite, démarrez et activez HAProxy pour qu'il se lance automatiquement au démarrage du système

```bash
sudo systemctl start haproxy
sudo systemctl enable haproxy
```

#### Vérification de l'installation

Vérifiez que HAProxy fonctionne correctement :

```bash
sudo systemctl status haproxy
```

#### Modifier la configuration de HAProxy

Ouvrez le fichier de configuration de HAProxy (`/etc/haproxy/haproxy.cfg`) :

```bash
sudo vim /etc/haproxy/haproxy.cfg
```

Ajoutez la configuration SSL dans la section globale et frontale. Par exemple :

```bash
global
    log /dev/log    local0
    maxconn 200

defaults
    log     global
    option  httplog
    timeout connect 5000ms
    timeout client  50000ms
    timeout server  50000ms

frontend https_front
	    bind *:443 ssl crt /etc/letsencrypt/live/votredomaine.com/fullchain.pem key /etc/letsencrypt/live/votredomaine.com/privkey.pem

	# ACL pour reconnaitre le nom de domaine 
	acl is_votredomaine_com hdr(host) -i votredomaine.com 
	acl is_otherdomain_com hdr(host) -i otherdomain.com

	# Route selon l'ACL 
	use_backend backend_votredomaine if is_votredomaine_com 
	use_backend backend_otherdomain if is_otherdomain_com

    default_backend http_back

backend http_back
    server web1 127.0.0.1:80 check

```

Ici, HAProxy écoute sur le port 443 (HTTPS) et utilise les certificats générés par Let's Encrypt.
#### Redémarrer HAProxy

Après avoir enregistré les modifications, redémarrez HAProxy :

```bash
sudo systemctl restart haproxy
```

#### Vérification

Assurez-vous que le serveur HTTPS fonctionne correctement en accédant à votre domaine via un navigateur.
#### Installer `haproxy` avec le module `http-request set-header`

Dans HAProxy, vous pouvez ajouter des en-têtes HTTP sécurisés pour améliorer la sécurité de votre serveur. Ajoutez les lignes suivantes à la configuration de votre frontend HTTPS.

Ouvrez à nouveau le fichier de configuration :

```bash
sudo vim /etc/haproxy/haproxy.cfg
```

Ajoutez les règles suivantes dans la section `frontend` :

```bash
frontend https_front
    bind *:443 ssl crt /etc/letsencrypt/live/votredomaine.com/fullchain.pem key /etc/letsencrypt/live/votredomaine.com/privkey.pem
    http-response set-header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
    http-response set-header X-Frame-Options "DENY"
    http-response set-header X-XSS-Protection "1; mode=block"
    http-response set-header X-Content-Type-Options "nosniff"
    http-response set-header Referrer-Policy "no-referrer"
    default_backend http_back
```

Ces en-têtes incluent :

- **Strict-Transport-Security** (HSTS) : Force l'utilisation de HTTPS.
- **X-Frame-Options** : Empêche les attaques de type clickjacking.
- **X-XSS-Protection** : Protection contre les attaques XSS.
- **X-Content-Type-Options** : Empêche le navigateur de détecter de manière incorrecte le type de contenu.
- **Referrer-Policy** : Limite l'envoi de l'URL de référence.

Redémarrez HAProxy après modification :

```bash
sudo systemctl restart haproxy
```




