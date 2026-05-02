# LAB 17 : Cracker OWASP Uncrackable Android Level 3

Prérequis: 
- Ghidra

<img width="587" height="446" alt="image" src="https://github.com/user-attachments/assets/c134714f-384e-481f-8918-d5057955c4b6" />
<img width="434" height="289" alt="image" src="https://github.com/user-attachments/assets/713da09b-8cb0-4e5e-ba52-94adcba1389b" />

- apk:
<img width="913" height="37" alt="image" src="https://github.com/user-attachments/assets/f7cfcd93-3538-40d5-a16c-2e1d623b8036" />
<img width="852" height="287" alt="image" src="https://github.com/user-attachments/assets/665af25d-8632-4714-8ed1-926feb6382c3" />


<img width="878" height="482" alt="image" src="https://github.com/user-attachments/assets/632bf14e-1a37-4158-94e6-200b5c4f60b2" />

L’APK officiel `UnCrackable-Level3.apk` a été téléchargé depuis le dépôt OWASP MSTG puis installé sur l’émulateur Android à l’aide de la commande `adb install`. Après installation, l’application s’ouvre correctement et affiche l’écran principal contenant le champ “Enter the Secret String”, ce qui confirme que l’APK est prêt pour l’analyse.

1) Étape 1 : Analyse statique simple avec Jadx-GUI (comprendre le Java)
   
- Ouvre l’APK dans Jadx-GUI (double-clic sur le fichier):

<img width="959" height="484" alt="image" src="https://github.com/user-attachments/assets/a1538d3a-d227-442b-849a-ef32d30932fd" />

on va dans sg.vantagepoint.uncrackable3 → MainActivity.
Et on Regarde :

<img width="959" height="491" alt="image" src="https://github.com/user-attachments/assets/53eef9a3-2e99-4bf8-97b3-9e6ec4469eb1" />

La méthode verifyLibs() effectue des vérifications d’intégrité sur les fichiers natifs (libfoo.so) ainsi que sur le fichier classes.dex. Ces vérifications reposent sur des valeurs CRC comparées aux valeurs attendues. En cas de modification détectée, la variable tampered est définie à 31337.

Points importants observe dans le MainActivity :

- System.loadLibrary("foo") charge la librairie native libfoo.so.
- verifyLibs() vérifie l’intégrité des fichiers .so et classes.dex.
- tampered = 31337 indique que l’application détecte une modification.
- Debug.isDebuggerConnected() sert à détecter un débogueur.
- RootDetection.checkRoot1/2/3() vérifie si l’appareil est rooté.
- IntegrityCheck.isDebuggable() vérifie si l’application est en mode debug.
- verify() appelle check_code() pour valider le secret saisi.

2- Étape 2 : Décompiler l’APK avec apktool 

l’objectif est de transformer l’APK en dossier modifiable.

<img width="862" height="197" alt="image" src="https://github.com/user-attachments/assets/f9a8ec2d-f2b8-40d4-b17f-b275b182c8f9" />
<img width="704" height="312" alt="image" src="https://github.com/user-attachments/assets/5e4ac8d3-165a-41f4-bcec-75a1aca43bb5" />

Étape 3 : Patch smali – Supprimer le message « tampered » / root

1. Ouvrir le bon fichier (attention : le chemin est important !)

vscode
<img width="959" height="473" alt="image" src="https://github.com/user-attachments/assets/39cc4335-5e09-4cce-a524-5c9ec19e8f90" />

MainActivity.smali
<img width="959" height="497" alt="image" src="https://github.com/user-attachments/assets/0aa65285-5985-4ca8-9afe-2716940643da" />

2. Chercher le bloc d’erreur (Ctrl + F)

<img width="959" height="491" alt="image" src="https://github.com/user-attachments/assets/69177c41-b13b-482c-b5ca-5e4f5c4207b2" />
Résultat 1 → la méthode complète showDialog (qui crée le popup « This is unacceptable... »)

<img width="721" height="440" alt="image" src="https://github.com/user-attachments/assets/d6d10977-756b-4701-a233-95e8b729755e" />
Résultat 2 → L’appel dans onCreate ← C’EST CELUI QUE NOUS ALLONS PATCHER

<img width="733" height="352" alt="image" src="https://github.com/user-attachments/assets/2932e580-c9af-480e-a254-b5e493f268e5" />
Résultat 3 → la méthode synthétique access$000

3. Le bloc exact à modifier (copie-colle)

Remplace tout le bloc d’erreur par :
<img width="839" height="361" alt="image" src="https://github.com/user-attachments/assets/6492efa0-9e20-456a-8954-6760d34491ed" />

5. Bonus : Neutraliser complètement la méthode showDialog (optionnel mais très propre)

<img width="835" height="216" alt="image" src="https://github.com/user-attachments/assets/bb038a6f-8e1c-4e67-b98f-8bd6600c5750" />

7. Recompiler l’APK
<img width="804" height="147" alt="image" src="https://github.com/user-attachments/assets/16f58d79-061a-4e7c-bfcd-6e4ed215862d" />

8. Signer l’APK (obligatoire)
<img width="857" height="368" alt="image" src="https://github.com/user-attachments/assets/94e58a59-9a86-4508-af79-f0b115dab4fe" />
<img width="848" height="169" alt="image" src="https://github.com/user-attachments/assets/c225ff29-22d7-4ebe-ad95-efd7b342d55a" />

9. Installer et tester
<img width="853" height="92" alt="image" src="https://github.com/user-attachments/assets/f05009db-94aa-45d3-9ee0-5cb8c59810a4" />

Étape 4 : Patch de la librairie native avec Ghidra (anti-debug + anti-Frida)

<img width="586" height="437" alt="image" src="https://github.com/user-attachments/assets/c0779476-09d8-45da-8043-47b837678e3c" />
<img width="640" height="359" alt="image" src="https://github.com/user-attachments/assets/7563aa65-d967-4d6b-bf03-090c5a2f736c" />

<img width="678" height="501" alt="image" src="https://github.com/user-attachments/assets/2af2983e-bf90-4964-af4b-8eaf3be1aff8" /> 

<img width="803" height="348" alt="image" src="https://github.com/user-attachments/assets/cad15d66-59b5-470c-a4af-0b78c1b877fe" />









