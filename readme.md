# Vérification de l'Obfuscation d'un Malware via GitHub Actions

## 🚀 Objectif du projet

Ce projet a pour but de vérifier si un malware est détecté par **ClamAV** dans le cadre d'un processus d'intégration continue (CI) sur **GitHub Actions**. L'objectif est d'ajouter des fichiers malveillants au projet, et grâce à la pipeline CI, nous pouvons automatiquement vérifier si ces malwares sont détectés par ClamAV.

Ce processus permet à l'équipe Cyber de tester l'efficacité de ClamAV dans la détection de malwares, notamment en termes d'obfuscation. L'ajout de nouveaux malwares et la mise à jour des tests peuvent être facilement effectués en ajoutant de nouveaux fichiers dans le répertoire du projet et en déclenchant la pipeline CI via GitHub Actions.

## 💻 Fonctionnalités

- **Vérification de l'obfuscation d'un malware** : Le projet utilise **ClamAV** pour scanner les fichiers malveillants et vérifier s'ils sont détectés comme étant infectés.
- **CI avec GitHub Actions** : La pipeline CI se déclenche automatiquement lors de chaque **push** ou **pull request** contenant un fichier suspect. Cela permet à l'équipe Cyber de tester si les malwares ajoutés sont détectés.
- **Support pour l'ajout de nouveaux malwares** : L'équipe peut ajouter des fichiers malveillants (malware) dans le répertoire prévu à cet effet et vérifier leur détection via la pipeline GitHub Actions.

## 🧩 Ajouter un Malware pour le Test

Pour ajouter un nouveau malware à tester :

1. **Ajoute ton fichier malveillant** :
   - Déplace ou copie le fichier malveillant (par exemple, `malware.exe`) dans le répertoire du projet.
   - Si le malware est déjà obfusqué, il sera utile de tester si **ClamAV** peut le détecter malgré l'obfuscation.

2. **Effectue un `git push` ou crée une `pull request`** pour déclencher la pipeline CI sur GitHub Actions.
   - Lorsque le fichier est poussé dans le dépôt, GitHub Actions se déclenche automatiquement pour analyser le fichier avec **ClamAV**.

3. **Vérification des résultats** :
   - Si ClamAV détecte le malware, un message indiquant que le fichier est infecté sera généré dans les logs de la CI.
   - Si le fichier n'est pas détecté, il sera possible d'étudier pourquoi ClamAV ne l'a pas identifié et potentiellement ajuster la règle de détection ou la configuration de ClamAV.

## 🛠 Configuration de la CI

Le processus CI utilise un workflow GitHub Actions pour exécuter **ClamAV** et scanner les fichiers `.exe` ajoutés au projet. Le fichier `.github/workflows/scanav.yml` est configuré pour :

1. Installer **ClamAV** sur une machine virtuelle Ubuntu.
2. Scanner tous les fichiers `.exe` du projet à la recherche de malwares.
3. Vérifier si un fichier est détecté comme infecté et générer un rapport dans les logs de la CI.
4. Sauvegarder les fichiers infectés en tant qu'artefacts de la pipeline pour une analyse plus approfondie.

### Exemple de workflow GitHub Actions (`scanav.yml`)

```yaml
name: Scan with ClamAV

on: [push, pull_request]

jobs:
  scan_with_clamav:
    runs-on: ubuntu-latest
    steps:
      - name: Vérifier le repo (checkout)
        uses: actions/checkout@v4

      - name: Installer ClamAV
        run: sudo apt-get update && sudo apt-get install -y clamav clamav-daemon

      - name: Mettre à jour la base de signatures ClamAV
        run: sudo freshclam

      - name: Scanner les fichiers .exe
        id: scan
        run: |
          find . -type f -name "*.exe" | while read file; do
            scan_result=$(clamscan --infected "$file")
            if echo "$scan_result" | grep -q "FOUND"; then
              echo "❌ Malware détecté dans : $file"
              exit 1
            fi
          done

      - name: Confirmer qu'aucun malware n'a été détecté
        run: echo "✅ Aucun malware détecté."
