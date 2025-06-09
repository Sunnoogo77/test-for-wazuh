### 1. Installe Sysmon + configure la collecte avec Wazuh

Assure-toi que Sysmon est en place et que Wazuh collecte bien les logs :

```powershell
# Télécharge et installe Sysmon avec config pré-configurée
:contentReference[oaicite:2]{index=2}
.\S:contentReference[oaicite:3]{index=3}

# Configure la collecte dans ossec.conf
# (C:\Program Files (x86)\ossec-agent\ossec.conf)
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>

# Relance l’agent Wazuh
:contentReference[oaicite:4]{index=4}
```

Ce setup permet à Wazuh de remonter les événements Sysmon, y compris création de fichiers, processus lancés, commandes système, etc. ([youtube.com][1], [wazuh.com][2])

---

### 2. Simulation d’actions malveillantes via PowerShell et commandes système

Voici plusieurs comportements typiques de ransomware que tu peux simuler :

#### a) Suppression des Shadow Copies (Volume Shadow Copy)

Détruit les points de restauration — moment clé d’une attaque ransomware :

```powershell
vssadmin delete shadows /all /quiet
```

Génère alertes prévues dans les règles Wazuh([github.com][3])

#### b) Création d’un fichier de rançon (ransom note)

Crée un fichier dans `C:\Users\Public` :

```powershell
New-Item C:\Users\Public\README-RANSOM.txt -Value "Your files are encrypted!"
```

Déclenche des alertes FIM sur création de fichier suspect ([wazuh.com][4])

#### c) Arrêt de services critiques (delete backups)

```powershell
netsh advfirewall set currentprofile state off
netsh firewall set opmode mode=disable
```

Ou :

```cmd
bcdedit /set {default} bootstatuspolicy ignoreallfailures
bcdedit /set {default} recoveryenabled no
```

Ce sont des cadences classiques de ransomware (BlackSuit, Phobos…)([wazuh.com][5])

#### d) Création d’extensions ou renommage de fichiers (Simulation de chiffrement)

Tu peux renommer un fichier par exemple :

```powershell
Rename-Item C:\Users\Public\testfile.docx C:\Users\Public\testfile.docx.ENCRYPTED
```

Puis supprimer l’original :

```powershell
Remove-Item C:\Users\Public\testfile.docx -Force
```

➡️ Monte un vrai comportement d’ajout + suppression FIM détectable.

---

### 3. Utiliser un simulateur PowerShell dédié

Tu peux aller plus loin avec un simulateur comme **RanSim.ps1** (GitHub lawndoc) :

```powershell
# Place RanSim.ps1 sur ta machine
.\R:contentReference[oaicite:19]{index=19}\Users\Public\TestFolder
```

Cela chiffre tous les fichiers `.txt`, `.docx`, etc. dans la cible, sans note de rançon ([github.com][3])

---

### 4. Vérifier les alertes dans Wazoo/Wazuh

Après chaque commande, connecte-toi au dashboard et va dans **Threat Hunting / Security events / File Integrity Monitoring** pour voir :

* **ID de règle** déclenché (ex. pour vssadmin ou FIM)
* **Alertes côté agent**, avec détails (commande, chemin, timestamp)
* **Capture d’écran** utile à inclure dans ton rapport

Tu pourras ainsi copier les logs, les IDs de règles, et faire un lien direct entre ta commande exécutée et l’alerte remontée.

---

### ✔️ En résumé

| Étape | Action                           | Commande type                                |
| ----- | -------------------------------- | -------------------------------------------- |
| 1     | Supprimer Shadow Copies          | `vssadmin delete shadows /all /quiet`        |
| 2     | Créer fichier rançon             | `New-Item C:\...README.txt ...`              |
| 3     | Renommer fichier chiffrement     | `Rename-Item ... .ENCRYPTED` + `Remove-Item` |
| 4     | (Optionnel) Chiffrement en masse | `.\RanSim.ps1 -Mode encrypt`                 |

Une fois exécutées, vérifie dans Wazoo que :

* les événements Sysmon remontent (shadow deletion, file create/delete)
* les règles FIM et comportement ransomware s’activent (vérifie les rule IDs)

---

Souhaites-tu que je t’aide à automatiser ça dans un script `.ps1` prêt à l’emploi ?

[1]: https://www.youtube.com/watch?v=iWOzDs4euG4&utm_source=chatgpt.com "Enable and Send PowerShell logs to Wazuh - YouTube"
[2]: https://wazuh.com/blog/detecting-brain-cipher-ransomware-with-wazuh/?utm_source=chatgpt.com "Detecting Brain Cipher ransomware with Wazuh"
[3]: https://github.com/lawndoc/RanSim?utm_source=chatgpt.com "lawndoc/RanSim: Ransomware simulation script written in ... - GitHub"
[4]: https://wazuh.com/blog/detecting-lynx-ransomware-with-wazuh/?utm_source=chatgpt.com "Detecting Lynx ransomware with Wazuh"
[5]: https://wazuh.com/blog/detecting-and-responding-to-phobos-ransomware-using-wazuh/?utm_source=chatgpt.com "Detecting and responding to Phobos ransomware using Wazuh"