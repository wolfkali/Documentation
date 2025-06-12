#### Afficher les informations sur les slots OTP

Cette commande affiche les informations sur les slots OTP de la YubiKey.

```bash
ykman otp info
```
#### Supprimer les slots OTP

 Ces commandes suppriment les slots OTP 1 et 2 de la YubiKey.
 
```bash
ykman otp delete 1 ykman otp delete 2
```
#### Réinitialiser la YubiKey

 Ces commandes réinitialisent la YubiKey pour les modes FIDO et PIV.
 
```bash
ykman fido reset ykman piv reset
```
#### Configurer les tentatives de PIN pour le mode PIV

Cette commande configure le nombre de tentatives de PIN pour le mode PIV à 15 tentatives pour le PIN et le PUK.

```bash
ykman piv access set-retries 15 15
```

#### Changer le PUK pour le mode PIV

 Cette commande permet de changer le PUK (Personal Unblocking Key) pour le mode PIV.
 
```bash
ykman piv access change-puk
```

#### Changer le PIN pour le mode PIV

Cette commande permet de changer le PIN pour le mode PIV.

```bash 
ykman piv access change-pin
```

### Exemples d'utilisation détaillée

#### Afficher les informations sur les slots OTP

Cette commande affiche les informations sur les slots OTP de la YubiKey. Par exemple, elle peut indiquer que les slots 1 et 2 sont vides.

```bash
ykman otp info
``` 
#### Supprimer les slots OTP

Ces commandes suppriment les slots OTP 1 et 2 de la YubiKey. Cela peut être utile pour réinitialiser les slots OTP.

```bash 
ykman otp delete 1 ykman otp delete 2
```

#### Réinitialiser la YubiKey

 Ces commandes réinitialisent la YubiKey pour les modes FIDO et PIV. La réinitialisation efface toutes les données et configurations associées à ces modes.
 
```bash
ykman fido reset ykman piv reset
```
#### Configurer les tentatives de PIN pour le mode PIV

Cette commande configure le nombre de tentatives de PIN pour le mode PIV à 15 tentatives pour le PIN et le PUK. Cela permet de définir le nombre de tentatives avant que la YubiKey ne se bloque.

```bash
ykman piv access set-retries 15 15
```
#### Changer le PUK pour le mode PIV

```bash
ykman piv access change-puk
```

#### Changer le PIN pour le mode PIV

Cette commande permet de changer le PIN pour le mode PIV. Le PIN est utilisé pour accéder aux fonctionnalités PIV de la YubiKey.

```bash
ykman piv access change-pin
```


 
