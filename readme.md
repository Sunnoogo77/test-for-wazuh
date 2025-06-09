## 1. 🔧 Installer Sysmon et configurer Wazuh

```powershell
# 1. Créer dossier
New-Item -ItemType Directory -Path C:\Sysmon -Force

# 2. Télécharger la config
Invoke-WebRequest `
  -Uri "https://wazuh.com/resources/blog/emulation-of-attack-techniques-and-detection-with-wazuh/sysmonconfig.xml" `
  -OutFile "C:\Sysmon\sysmonconfig.xml"

# 3. Extraire Sysmon (chemin à ajuster)
Expand-Archive -Path "<PATH_VERS_SYSNOM_ZIP>\Sysmon.zip" -DestinationPath "C:\Sysmon"

# 4. Installer Sysmon
cd C:\Sysmon
.\Sysmon64.exe -accepteula -i C:\Sysmon\sysmonconfig.xml

# 5. Configurer Wazuh agent
Add-Content `
  -Path "C:\Program Files (x86)\ossec-agent\ossec.conf" `
  -Value @"
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
"@

# 6. Redémarrer Wazuh
Restart-Service -Name wazuh

```


Ces étapes garantissent que Wazuh reçoit bien les logs de Sysmon ([blog.tecnetone.com][1], [wazuh.com][2]).

---

## 2. 🛡️ Simuler des comportements typiques d’un ransomware

Chaque commande suivante déclenchera une détection dans Wazuh/Wazoo :

### a) Supprimer les Shadow Copies

```powershell
vssadmin delete shadows /all /quiet
```

(Déclenche alertes Shadow Copy deletion – rule id 100631…) ([wazuh.com][2], [wazuh.com][3])

---

### b) Désactiver le pare-feu Windows

```powershell
netsh advfirewall set currentprofile state off
netsh firewall set opmode mode=disable
```

(Comportement observé dans Blackbit/Phobos…) ([wazuh.com][4])

---

### c) Modifier le Bootloader (BCDEdit)

```cmd
bcdedit /set {default} recoveryenabled no
bcdedit /set {default} bootstatuspolicy ignoreallfailures
```

(Techniques pour empêcher la récupération – rule ids 100621, 100622…) ([wazuh.com][3])

---

### d) Créer un fichier de rançon

```powershell
New-Item C:\Users\Public\!!!READ_ME_MEDUSA!!!.txt -Value "Your files have been encrypted. Contact attacker@domain"
```

(Interacte avec FIM pour créations multiples de note) ([wazuh.com][5])

---

### e) Renommer / supprimer un fichier (simulate encryption)

```powershell
Rename-Item C:\Users\Public\test.docx test.docx.ENCRYPTED
Remove-Item C:\Users\Public\test.docx -Force
```

(Expose FIM crée/supprime fichiers en masse)&#x20;

---

### f) Terminer des services/processus système

```powershell
net stop "Spooler" /y
taskkill /F /IM notepad.exe /T
```

(Cadre d’actions observées dans Medusa – rule id 100012+) ([wazuh.com][5])

---

## 3. ⚡ Simulation avancée : script RanSim

```powershell
# Télécharger et placer RanSim.ps1
.\RanSim.ps1 -Mode encrypt -TargetPath "C:\Users\Public\TestFolder"
```

(Il chiffre plusieurs fichiers .txt/.docx, déclenchant FIM) ([github.com][6])

---

## 4. 📊 Vérifier les résultats dans Wazoo

Après chaque commande :

1. Accède à **Threat Hunting** ou **Security Events**.
2. Filtre par `rule.id` utile :

   * Shadow Copy : 100631
   * BCDEdit : 100621–622
   * FIM créé/supprimé
   * Taskkill/net stop : 100012 (Medusa) ([wazuh.com][3], [wazuh.com][5])

Tu pourras prendre des captures d’écran pour ton rapport.

---