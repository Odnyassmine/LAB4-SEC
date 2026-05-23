# LAB4-SEC
# Laboratoire — Analyse Statique d’un APK Android

## 🎯 Objectifs du laboratoire

À la fin de ce laboratoire, vous serez capable de :

- Préparer un environnement d’analyse Android
- Vérifier l’intégrité d’un fichier APK
- Explorer la structure interne d’un APK
- Décompiler une application Android avec JADX
- Identifier les permissions et composants sensibles
- Rechercher des informations sensibles codées en dur
- Convertir des fichiers DEX en JAR avec dex2jar
- Comparer les résultats entre JADX et JD-GUI
- Rédiger un rapport professionnel d’analyse statique
- Organiser et nettoyer un workspace d’analyse

---

# 🧰 Outils nécessaires

| Outil | Rôle |
|---|---|
| ADB | Interaction avec Android |
| JADX GUI | Décompilation APK |
| dex2jar | Conversion DEX → JAR |
| JD-GUI | Analyse de bytecode Java |
| unzip / ZIP | Extraction d’APK |
| Android Studio (optionnel) | Génération d’APK |
| APK Extractor (optionnel) | Extraction d’APK depuis Android |

---

# 📦 Pré-requis

- Java installé
- Android SDK installé (optionnel mais recommandé)
- APK à analyser
- Émulateur Android ou appareil de test
- Outils ajoutés au PATH système

---

# Task 1 — Préparer le workspace et vérifier l’APK

## 🎯 Objectif

Créer un environnement de travail organisé et vérifier l’intégrité de l’APK.

---

## Créer le dossier de travail

### Windows

```powershell
mkdir C:\APK-Analysis
cd C:\APK-Analysis
```

### Linux/macOS

```bash
mkdir ~/APK-Analysis
cd ~/APK-Analysis
```

Copier ensuite l’APK dans ce dossier.

---

## Vérifier que l’APK est une archive ZIP valide

### Windows

```powershell
Get-Content -Path .\app-debug.apk -TotalCount 4 | Format-Hex
```

### Linux/macOS

```bash
hexdump -n 4 app-debug.apk
```

---

### Ce qu’il faut observer

Les premiers octets doivent commencer par :

```text
50 4B
```

ce qui correspond à :

```text
PK
```

signature standard des fichiers ZIP.

---

## Lister le contenu de l’APK

### Windows

```powershell
Add-Type -Assembly System.IO.Compression.FileSystem

$apk = Join-Path (Get-Location).Path "UnCrackable-Level1.apk"

Test-Path $apk

[System.IO.Compression.ZipFile]::OpenRead($apk).Entries |
Select-Object -ExpandProperty FullName -First 20
```

### Linux/macOS

```bash
unzip -l app-debug.apk | head -20
```

---

## Calculer le hash SHA-256

### Windows

```powershell
Get-FileHash -Algorithm SHA256 .\app-debug.apk
```

### Linux/macOS

```bash
sha256sum app-debug.apk
```

---

## Vérifier la signature APK (optionnel)

### Windows

```powershell
& "C:\Program Files (x86)\Android\android-sdk\build-tools\<version>\apksigner.bat" verify --verbose .\app-debug.apk
```

### Linux/macOS

```bash
apksigner verify --verbose app-debug.apk
```

---

## ✅ Checkpoint

- Dossier de travail créé
- APK identifié comme archive valide
- Hash SHA-256 enregistré
- Structure de base de l’APK observée

---

# Task 2 — Extraire ou obtenir l’APK

## 🎯 Objectif

Disposer d’un APK valide pour l’analyse.

---

## Option A — APK fourni

Si l’APK est fourni :

- vérifier sa provenance
- documenter la source dans le rapport

---

## Option B — Générer un APK avec Android Studio

### Étapes

1. Ouvrir le projet Android
2. Aller dans :

```text
Build > Build Bundle(s) / APK(s) > Build APK(s)
```

3. Localiser :

```text
app/build/outputs/apk/debug/app-debug.apk
```

4. Copier l’APK dans le workspace

---

## Option C — Extraire un APK depuis Android

### Étapes

1. Installer APK Extractor
2. Sélectionner l’application
3. Exporter l’APK
4. Transférer le fichier sur le PC

---

## ✅ Checkpoint

- APK disponible dans le workspace
- Provenance documentée
- Taille du fichier notée

---

# Task 3 — Analyse avec JADX GUI

## 🎯 Objectif

Explorer la structure de l’APK et analyser le manifeste.

---

## Lancer JADX GUI

### Windows

```powershell
start "" "C:\Path\to\jadx-gui.exe"
```

### Linux/macOS

```bash
/path/to/jadx-gui
```

---

## Ouvrir l’APK

Dans JADX :

```text
File > Open file...
```

---

## Explorer la structure

Observer notamment :

```text
resources/AndroidManifest.xml
resources/res/values/strings.xml
com.example.app
```

---

## Analyse du manifeste

Identifier :

- package principal
- versionName
- minSdk
- targetSdk

---

## Vérifier les permissions

Chercher :

```xml
<uses-permission>
```

---

## Identifier les composants exportés

Chercher :

```xml
android:exported="true"
```

ou :

```xml
<intent-filter>
```

---

## Vérifier les configurations sensibles

Rechercher :

```xml
android:usesCleartextTraffic="true"
android:debuggable="true"
```

---

## Explorer les ressources importantes

Analyser :

- `strings.xml`
- `network_security_config.xml`
- autres fichiers XML

---

## ⚠️ Explication sécurité

Un composant :

```xml
android:exported="true"
```

peut être appelé par d’autres applications Android.

Cela augmente la surface d’attaque.

---

## ✅ Checkpoint

- Package principal identifié
- Permissions listées
- Composants exportés identifiés
- Configurations sensibles notées
- Ressources importantes analysées

---

# Task 4 — Recherche de chaînes sensibles

## 🎯 Objectif

Identifier les informations sensibles codées en dur.

---

## Recherche globale dans JADX

Utiliser :

```text
Ctrl + F
```

---

## Rechercher des URLs

```text
http://
https://
api
endpoint
server
```

---

## Rechercher des secrets

```text
token
api_key
apikey
secret
password
jwt
oauth
```

---

## Rechercher des indicateurs de debug

```text
DEBUG
debug
test
staging
firebase
crashlytics
```

---

## Pour chaque découverte

Documenter :

- valeur trouvée
- emplacement
- niveau de risque
- description

---

## Grille de sévérité

| Niveau | Description |
|---|---|
| Faible | Information publique |
| Moyen | Donnée sensible limitée |
| Élevé | Secret critique |

---

## ✅ Checkpoint

- Recherches effectuées
- Minimum 5 observations documentées
- Niveau de risque évalué
- Emplacements précis enregistrés

---

# Task 5 — Convertir DEX vers JAR avec dex2jar

## 🎯 Objectif

Transformer le bytecode Android en JAR.

---

## Extraire les fichiers DEX

### Windows

```powershell
mkdir dex_out

Add-Type -Assembly System.IO.Compression.FileSystem

$zip = [System.IO.Compression.ZipFile]::OpenRead(".\app-debug.apk")

$zip.Entries |
Where-Object { $_.Name -like "classes*.dex" } |
ForEach-Object {
    [System.IO.Compression.ZipFileExtensions]::ExtractToFile(
        $_,
        ".\dex_out\$($_.Name)",
        $true
    )
}

$zip.Dispose()
```

### Linux/macOS

```bash
mkdir -p dex_out

unzip -j app-debug.apk "classes*.dex" -d dex_out
```

---

## Vérifier les fichiers DEX

### Windows

```powershell
dir dex_out
```

### Linux/macOS

```bash
ls -la dex_out
```

---

## Convertir DEX → JAR

### Windows

```powershell
.\d2j-dex2jar.bat "C:\APK-Analysis\dex_out\classes.dex" -o "C:\APK-Analysis\app.jar"
```

### Linux/macOS

```bash
./d2j-dex2jar.sh ~/APK-Analysis/dex_out/classes.dex -o ~/APK-Analysis/app.jar
```

---

## Gestion du multi-dex

### Windows

```powershell
Get-ChildItem -Path .\dex_out -Filter "classes*.dex" |
ForEach-Object {
    .\d2j-dex2jar.bat $_.FullName -o ".\$($_.BaseName).jar"
}
```

### Linux/macOS

```bash
for file in dex_out/classes*.dex; do
    ./d2j-dex2jar.sh "$file" -o "${file%.dex}.jar"
done
```

---

## ✅ Checkpoint

- Fichiers DEX extraits
- Conversion réussie
- JAR générés
- Multi-dex géré si nécessaire

---

# Task 6 — Comparaison JADX vs JD-GUI

## 🎯 Objectif

Comparer plusieurs outils de décompilation.

---

## Lancer JD-GUI

### Windows

```powershell
start "" "C:\Path\to\jd-gui.exe"
```

### Linux/macOS

```bash
/path/to/jd-gui
```

---

## Comparer les deux outils

Analyser :

- navigation
- lisibilité
- support Android
- gestion Kotlin
- obfuscation
- ressources

---

## Exemple de comparaison

| Aspect | JADX GUI | JD-GUI |
|---|---|---|
| Navigation | Structure Android complète | Structure Java uniquement |
| Kotlin | Meilleur support | Support limité |
| Obfuscation | Reconstruction partielle | Noms obfusqués conservés |
| Ressources | XML accessibles | Ressources absentes |
| Annotations | Bon support Android | Support limité |

---

## ✅ Checkpoint

- Même classe analysée
- 3 différences documentées
- Forces/faiblesses identifiées
- Conclusion rédigée

---

# Task 7 — Rédaction du rapport professionnel

## 🎯 Objectif

Formaliser les résultats de l’analyse.

---

# Template de rapport

```markdown
# Rapport d'analyse statique - [Nom de l'application]

## Informations générales

- **Date d'analyse :** [date]
- **Analyste :** [nom]
- **APK analysé :** [nom_fichier.apk]
- **Version :** [version]
- **Provenance :** [source]
- **Outils utilisés :** JADX GUI v[x.x], dex2jar v[x.x], JD-GUI v[x.x]

---

## Résumé exécutif

Cette analyse statique a révélé [nombre] vulnérabilités potentielles.

Le niveau de risque global est évalué comme [faible/moyen/élevé].

Actions prioritaires recommandées :

1. [action 1]
2. [action 2]
3. [action 3]

---

## Constats détaillés

### Constat #1 : [titre]

**Sévérité :** [Faible/Moyenne/Élevée]

**Description :**
[description]

**Localisation :**
[fichier/classe]

**Impact potentiel :**
[impact]

**Remédiation recommandée :**
[solution]

---

### Constat #2 : [titre]

**Sévérité :** [Faible/Moyenne/Élevée]

**Description :**
[description]

**Localisation :**
[fichier/classe]

**Impact potentiel :**
[impact]

**Remédiation recommandée :**
[solution]

---

## Annexes

### Permissions demandées

- permission 1
- permission 2

### Composants exportés

- composant 1
- composant 2
```

---

## ✅ Checkpoint

- Toutes les sections remplies
- Minimum 3 constats documentés
- Remédiations proposées
- Format professionnel

---

# Task 8 — Nettoyage

## 🎯 Objectif

Nettoyer et organiser l’environnement d’analyse.

---

## Vérifier les données sensibles

Supprimer du rapport :

- tokens
- mots de passe
- clés de production
- données personnelles

---

## Organiser les fichiers

### Windows

```powershell
mkdir .\results

move .\app.jar .\results\
move .\rapport.md .\results\
```

### Linux/macOS

```bash
mkdir -p ./results

mv ./app.jar ./results/
mv ./rapport.md ./results/
```

---

## Supprimer les artefacts temporaires

### Windows

```powershell
Remove-Item -Recurse -Force .\dex_out
```

### Linux/macOS

```bash
rm -rf ./dex_out
```

---

## Supprimer l’APK (optionnel)

### Windows

```powershell
Remove-Item .\app-debug.apk
```

### Linux/macOS

```bash
rm ./app-debug.apk
```

---

## ✅ Checkpoint

- Rapport nettoyé
- Fichiers organisés
- Artefacts supprimés
- Conformité respectée

---

# 🧾 Conclusion

Ce laboratoire introduit les principales techniques d’analyse statique Android :

- exploration d’APK
- décompilation Java
- analyse du manifeste Android
- identification de secrets codés en dur
- conversion DEX → JAR
- comparaison d’outils de reverse engineering

Ces compétences sont fondamentales pour :

- l’audit mobile
- le reverse engineering Android
- l’analyse de sécurité applicative
- les challenges OWASP MSTG
# 📸 Captures d’écran du laboratoire

## Capture 1 — Ouverture de l’APK dans JADX

![Capture JADX](./screenshots/capture_1.png)

---

## Capture 2 — Analyse du fichier AndroidManifest.xml

![AndroidManifest](./screenshots/capture_2.png)

---

## Capture 3 — Exploration des classes Java

![Classes Java](./screenshots/capture_3.png)

---

## Capture 4 — Vérification du hash et structure APK

![Hash APK](./screenshots/capture_4.png)

---

## Capture 5 — Recherche de chaînes sensibles

![Recherche chaînes](./screenshots/capture_5.png)

---

## Capture 6 — Extraction des fichiers DEX

![DEX Extraction](./screenshots/capture_6.png)

---

## Capture 7 — Conversion DEX vers JAR

![DEX2JAR](./screenshots/capture_7.png)

---

## Capture 8 — Analyse dans JD-GUI

![JD-GUI](./screenshots/capture_8.png)

---

# 🧾 Fin du laboratoire
## Capture 6 — Extraction des fichiers DEX

![DEX Extraction](screenshots/capture_6.png)

---

## Capture 7 — Conversion DEX vers JAR

![DEX2JAR](screenshots/capture_7.png)

---

## Capture 8 — Analyse dans JD-GUI

![JD-GUI](screenshots/capture_8.png)

---

