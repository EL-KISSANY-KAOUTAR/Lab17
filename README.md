# LAB 17 : Cracker OWASP Uncrackable Android Level 3

Ghidra
<img width="587" height="446" alt="image" src="https://github.com/user-attachments/assets/c134714f-384e-481f-8918-d5057955c4b6" />
<img width="434" height="289" alt="image" src="https://github.com/user-attachments/assets/713da09b-8cb0-4e5e-ba52-94adcba1389b" />

apk
<img width="913" height="37" alt="image" src="https://github.com/user-attachments/assets/f7cfcd93-3538-40d5-a16c-2e1d623b8036" />
<img width="852" height="287" alt="image" src="https://github.com/user-attachments/assets/665af25d-8632-4714-8ed1-926feb6382c3" />

<img width="878" height="482" alt="image" src="https://github.com/user-attachments/assets/632bf14e-1a37-4158-94e6-200b5c4f60b2" />
L’APK officiel `UnCrackable-Level3.apk` a été téléchargé depuis le dépôt OWASP MSTG puis installé sur l’émulateur Android à l’aide de la commande `adb install`. Après installation, l’application s’ouvre correctement et affiche l’écran principal contenant le champ “Enter the Secret String”, ce qui confirme que l’APK est prêt pour l’analyse.

Étape 1 : Analyse statique simple avec Jadx-GUI (comprendre le Java)
<img width="959" height="484" alt="image" src="https://github.com/user-attachments/assets/a1538d3a-d227-442b-849a-ef32d30932fd" />

<img width="959" height="491" alt="image" src="https://github.com/user-attachments/assets/53eef9a3-2e99-4bf8-97b3-9e6ec4469eb1" />
La méthode verifyLibs() effectue des vérifications d’intégrité sur les fichiers natifs (libfoo.so) ainsi que sur le fichier classes.dex. Ces vérifications reposent sur des valeurs CRC comparées aux valeurs attendues. En cas de modification détectée, la variable tampered est définie à 31337.

Points importants observe dans le MainActivity :

System.loadLibrary("foo") charge la librairie native libfoo.so.
verifyLibs() vérifie l’intégrité des fichiers .so et classes.dex.
tampered = 31337 indique que l’application détecte une modification.
Debug.isDebuggerConnected() sert à détecter un débogueur.
RootDetection.checkRoot1/2/3() vérifie si l’appareil est rooté.
IntegrityCheck.isDebuggable() vérifie si l’application est en mode debug.
verify() appelle check_code() pour valider le secret saisi.

Étape 2 : Décompiler l’APK avec apktool 

l’objectif est de transformer l’APK en dossier modifiable.

<img width="862" height="197" alt="image" src="https://github.com/user-attachments/assets/f9a8ec2d-f2b8-40d4-b17f-b275b182c8f9" />
<img width="704" height="312" alt="image" src="https://github.com/user-attachments/assets/5e4ac8d3-165a-41f4-bcec-75a1aca43bb5" />




