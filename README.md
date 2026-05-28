# LAB : Bypass de la Détection Root Android avec Objection

## 1. Introduction

Ce laboratoire a pour objectif de réaliser une analyse dynamique d’une application Android qui effectue une détection de root, puis de contourner cette détection à l’aide de Frida et Objection.

L’application utilisée est :

```text
RootBeer Sample
Package : com.scottyab.rootbeer.sample
APK : RootBeer_Sample-0.9.apk
```

RootBeer Sample est une application de test qui exécute plusieurs contrôles afin de vérifier si un appareil Android est rooté.  
Dans ce lab, l’objectif est de lancer l’application dans un environnement instrumenté, d’exécuter la commande Objection adaptée, puis de vérifier que l’application affiche `NOT ROOTED`.

---

## 2. Objectifs du lab

Les objectifs de ce lab sont :

- installer et vérifier Objection côté PC ;
- vérifier la connexion ADB avec l’émulateur Android ;
- lancer `frida-server` sur Android ;
- installer l’APK RootBeer Sample ;
- identifier le package de l’application cible ;
- lancer Objection sur l’application ;
- exécuter la commande :

```bash
android root disable
```

- vérifier que les checks RootBeer sont interceptés ;
- valider que l’application affiche `NOT ROOTED`.

---

## 3. Environnement de travail

| Élément | Valeur |
|---|---|
| Système hôte | Windows |
| Terminal | PowerShell |
| Appareil Android | Émulateur Android |
| Version Android utilisée | Android 11 |
| Outil d’instrumentation | Frida |
| Surcouche Frida | Objection |
| Application cible | RootBeer Sample |
| Package cible | `com.scottyab.rootbeer.sample` |
| APK utilisé | `RootBeer_Sample-0.9.apk` |

---

## 4. Prérequis

Les prérequis nécessaires sont :

- Python 3.8 ou supérieur ;
- pip ;
- ADB / Android Platform Tools ;
- Frida côté PC ;
- `frida-server` côté Android ;
- Objection ;
- un émulateur ou appareil Android avec le débogage USB activé ;
- l’APK RootBeer Sample.

Vérification rapide :

```powershell
python --version
pip --version
adb version
frida --version
objection --version
```

---

## 5. Vérification de l’appareil Android avec ADB

Sous Windows, le chemin ADB utilisé est :

```powershell
$ADB = "$env:LOCALAPPDATA\Android\Sdk\platform-tools\adb.exe"
```

La connexion avec l’émulateur est vérifiée avec :

```powershell
& $ADB devices
```

Résultat attendu :

```text
List of devices attached
emulator-5554    device
```

Cette étape confirme que l’émulateur est bien connecté et prêt à être utilisé.
 
---

## 6. Installation et vérification d’Objection

Objection peut être installé avec pip :

```powershell
pip install --upgrade objection
```

Vérification :

```powershell
objection --version
objection --help
```

Dans ce lab, Objection est utilisé pour injecter automatiquement des hooks Frida dans l’application cible.

<img width="1054" height="228" alt="image" src="https://github.com/user-attachments/assets/ed1411d0-ef46-4bf5-ba27-1bd044528f7b" />



## 7. Vérification de Frida

La version Frida côté PC est vérifiée avec :

```powershell
frida --version
```

La version Python de Frida peut aussi être vérifiée avec :

```powershell
python -c "import frida; print(frida.__version__)"
```

Les versions de Frida côté PC et de `frida-server` côté Android doivent être alignées.

Capture recommandée :



## 8. Préparation de frida-server sur Android

L’architecture de l’émulateur est identifiée avec :

```powershell
& $ADB shell getprop ro.product.cpu.abi
```

Exemple de résultat :

```text
x86_64
```

Le binaire `frida-server` correspondant est poussé sur l’appareil :

```powershell
& $ADB push .\frida-server /data/local/tmp/frida-server
& $ADB shell chmod 755 /data/local/tmp/frida-server
```

Ensuite, `frida-server` est lancé :

```powershell
& $ADB shell "/data/local/tmp/frida-server -l 0.0.0.0:27042"
```

Dans un deuxième terminal PowerShell, la visibilité de l’appareil est vérifiée :

```powershell
frida-ps -Uai
```

<img width="940" height="488" alt="image" src="https://github.com/user-attachments/assets/bb858ba2-9fc5-4e88-9b15-f8148e16fffd" />

## 9. Installation de l’application RootBeer Sample

L’APK utilisé se trouve sur le PC dans le chemin suivant :

```text
C:\Users\halim\Downloads\RootBeer_Sample-0.9.apk
```

Installation de l’APK :

```powershell
$APK = "C:\Users\halim\Downloads\RootBeer_Sample-0.9.apk"
& $ADB install -r $APK
```

Résultat attendu :

```text
Success
```

<img width="977" height="121" alt="image" src="https://github.com/user-attachments/assets/820717fb-3d1c-42ac-8e7d-c9aeed1925f4" />

## 10. Identification du package de l’application

Après installation, le package est recherché avec :

```powershell
& $ADB shell pm list packages | Select-String -Pattern "root"
```

Résultat attendu :

```text
package:com.scottyab.rootbeer.sample
```

Le package utilisé pour Objection est donc :

```text
com.scottyab.rootbeer.sample
```

Vérification avec Frida :

```powershell
frida-ps -Uai | Select-String -Pattern "root"
```


## 11. Lancement de l’application avant bypass

L’application est lancée normalement avec ADB :

```powershell
& $ADB shell monkey -p com.scottyab.rootbeer.sample -c android.intent.category.LAUNCHER 1
```

Avant le bypass, une application de root detection peut afficher que l’appareil est rooté ou présenter plusieurs checks détectés.

Exemples de contrôles réalisés par RootBeer :

```text
Root Management Apps
Potentially Dangerous Apps
Root Cloaking Apps
TestKeys
BusyBoxBinary
SU Binary
For RW Paths
Dangerous Props
Root via native check
SELinux Flag Is Enabled
Magisk specific checks
```


## 12. Lancement d’Objection sur l’application cible

L’application est d’abord arrêtée :

```powershell
& $ADB shell am force-stop com.scottyab.rootbeer.sample
```

Objection est ensuite lancé sur le package cible :

```powershell
objection -g com.scottyab.rootbeer.sample explore
```

La console Objection s’ouvre sur l’application cible :

```text
com.scottyab.rootbeer.sample (run) on (Android: 11) [usb] #
```

<img width="1048" height="558" alt="image" src="https://github.com/user-attachments/assets/62e70409-4a36-43fb-a257-b9fd02851a75" />


## 13. Désactivation de la détection root

Dans la console Objection, la commande suivante est exécutée :

```bash
android root disable
```

Cette commande installe des hooks Frida destinés à masquer les indicateurs de root.

Résultat observé dans la console :

```text
(agent) Registering job ... Name: root-detection-disable
(agent) RootBeer->detectRootCloakingApps() check detected, marking as false.
(agent) RootBeer->detectTestKeys() check detected, marking as false.
(agent) RootBeer->checkForBinary() check detected, marking as false.
(agent) RootBeer->checkSuExists() check detected, marking as false.
(agent) RootBeer->checkForDangerousProps() check detected, marking as false.
(agent) RootBeerNative->checkForRoot() check detected, marking as 0.
```

<img width="975" height="312" alt="image" src="https://github.com/user-attachments/assets/4d610e77-4e48-4209-b2c3-717062873fb7" />

Interprétation :

```text
Objection a intercepté plusieurs méthodes de RootBeer.
Les méthodes de détection sont forcées à retourner false ou 0.
Cela masque les indicateurs de root détectés par l’application.
```

---

## 14. Compréhension de la commande `android root disable`

La commande :

```bash
android root disable
```

installe des hooks Java via Frida pour neutraliser les contrôles root courants.

Elle peut agir sur plusieurs types de vérifications :

### 14.1 Propriétés système Android

Certaines applications vérifient la valeur :

```text
android.os.Build.TAGS
```

Si cette valeur contient :

```text
test-keys
```

l’appareil peut être considéré comme rooté.

Objection peut modifier le comportement attendu afin de faire croire que l’appareil utilise des valeurs inoffensives, comme :

```text
release-keys
```

### 14.2 Présence de fichiers sensibles

Les applications de root detection cherchent souvent des fichiers comme :

```text
/system/bin/su
/system/xbin/su
/sbin/su
/system/app/Superuser.apk
/system/bin/busybox
/system/xbin/busybox
```

Objection peut intercepter les appels à `java.io.File.exists()` afin de faire croire que ces fichiers n’existent pas.

### 14.3 Exécution de commandes système

Les applications peuvent exécuter des commandes comme :

```text
su
which su
busybox
mount
getprop
```

Objection peut intercepter les appels à :

```text
Runtime.getRuntime().exec()
```

afin d’empêcher ces commandes de révéler la présence du root.

### 14.4 Méthodes RootBeer

Dans ce lab, Objection a intercepté directement plusieurs méthodes RootBeer, notamment :

```text
detectRootCloakingApps()
detectTestKeys()
checkForBinary()
checkSuExists()
checkForDangerousProps()
RootBeerNative.checkForRoot()
```

Ces méthodes ont été forcées à retourner un résultat négatif.

---

## 15. Validation après bypass

Après l’exécution de `android root disable`, l’application RootBeer Sample affiche :

```text
NOT ROOTED
```

Tous les checks visibles sont affichés en vert.

<img width="399" height="763" alt="image" src="https://github.com/user-attachments/assets/fd49f3a2-b415-4879-9941-f03ce462d656" />


Interprétation :

```text
L’application ne détecte plus le root.
Les checks RootBeer ont été neutralisés par les hooks Objection.
Le bypass de la root detection est validé.
```

---

## 16. Commandes utiles dans Objection

Pendant l’analyse, plusieurs commandes Objection peuvent être utilisées pour explorer l’application :

### Aide sur les commandes root

```bash
help android root
```

### Recherche de classes liées au root

```bash
android hooking search classes root
```

### Recherche de méthodes liées au root

```bash
android hooking search methods root
```

### Recherche de méthodes `isRoot`

```bash
android hooking search methods isRoot
```

### Recherche de classes RootBeer

```bash
android hooking search classes RootBeer
```

---

## 17. Résumé des résultats

| Test | Résultat |
|---|---|
| ADB détecte l’émulateur | Réussi |
| Frida fonctionne | Réussi |
| Objection fonctionne | Réussi |
| RootBeer Sample installé | Réussi |
| Package identifié | `com.scottyab.rootbeer.sample` |
| Application lancée | Réussi |
| Objection attaché à l’application | Réussi |
| Commande `android root disable` exécutée | Réussi |
| Méthodes RootBeer interceptées | Réussi |
| Résultat final dans l’application | `NOT ROOTED` |

---

## 18. Problème observé pendant le lab

Pendant la manipulation, la commande suivante a été exécutée par erreur :

```bash
android sslpinning disable
```

Cette commande concerne le contournement du SSL Pinning et non la détection root.

Elle n’est pas utilisée pour valider ce lab.

La commande correcte pour ce lab est :

```bash
android root disable
```

Après exécution de la commande correcte, les hooks RootBeer ont été installés et le résultat final a été validé.

---



## 19. Commandes principales utilisées

### Déclarer ADB

```powershell
$ADB = "$env:LOCALAPPDATA\Android\Sdk\platform-tools\adb.exe"
```

### Vérifier l’émulateur

```powershell
& $ADB devices
```

### Installer RootBeer Sample

```powershell
$APK = "C:\Users\halim\Downloads\RootBeer_Sample-0.9.apk"
& $ADB install -r $APK
```

### Identifier le package

```powershell
& $ADB shell pm list packages | Select-String -Pattern "root"
```

### Lancer l’application

```powershell
& $ADB shell monkey -p com.scottyab.rootbeer.sample -c android.intent.category.LAUNCHER 1
```

### Lancer Objection

```powershell
objection -g com.scottyab.rootbeer.sample explore
```

### Désactiver la root detection

```bash
android root disable
```

### Arrêter l’application

```powershell
& $ADB shell am force-stop com.scottyab.rootbeer.sample
```

---

## 20. Conclusion

Ce laboratoire a permis de réaliser un bypass de la détection root d’une application Android à l’aide d’Objection.

L’application RootBeer Sample a été installée et exécutée sur un émulateur Android. Après attachement avec Objection, la commande `android root disable` a été exécutée. Les logs Objection montrent que plusieurs méthodes RootBeer ont été détectées puis forcées à retourner des valeurs négatives.

Les méthodes interceptées incluent notamment `detectTestKeys`, `checkForBinary`, `checkSuExists`, `detectRootCloakingApps`, `checkForDangerousProps` et `RootBeerNative.checkForRoot`.

Après l’injection des hooks, l’application affiche `NOT ROOTED`, ce qui confirme que le bypass de la détection root a fonctionné correctement.

Le lab est donc validé.

---

## 21. Remarque éthique

Les techniques présentées dans ce lab doivent être utilisées uniquement dans un cadre autorisé :

- application de test ;
- environnement pédagogique ;
- audit de sécurité encadré ;
- appareil ou application appartenant à l’auditeur.

Elles ne doivent pas être utilisées sur des applications tierces sans autorisation explicite.
