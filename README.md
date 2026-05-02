# LAB 17 : Cracker OWASP Uncrackable Android Level 3

**Prérequis :**  

- Ghidra

<img width="587" height="446" alt="image" src="https://github.com/user-attachments/assets/c134714f-384e-481f-8918-d5057955c4b6" />
<img width="434" height="289" alt="image" src="https://github.com/user-attachments/assets/713da09b-8cb0-4e5e-ba52-94adcba1389b" />

- apk:
<img width="913" height="37" alt="image" src="https://github.com/user-attachments/assets/f7cfcd93-3538-40d5-a16c-2e1d623b8036" />
<img width="852" height="287" alt="image" src="https://github.com/user-attachments/assets/665af25d-8632-4714-8ed1-926feb6382c3" />


<img width="878" height="482" alt="image" src="https://github.com/user-attachments/assets/632bf14e-1a37-4158-94e6-200b5c4f60b2" />


L’APK officiel `UnCrackable-Level3.apk` a été téléchargé depuis le dépôt OWASP MSTG puis installé sur l’émulateur Android à l’aide de la commande `adb install`. Après installation, l’application s’ouvre correctement et affiche l’écran principal contenant le champ “Enter the Secret String”, ce qui confirme que l’APK est prêt pour l’analyse.

**1) Étape 1 : Analyse statique simple avec Jadx-GUI (comprendre le Java)**  
   
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

**2- Étape 2 : Décompiler l’APK avec apktool**  

l’objectif est de transformer l’APK en dossier modifiable.

<img width="862" height="197" alt="image" src="https://github.com/user-attachments/assets/f9a8ec2d-f2b8-40d4-b17f-b275b182c8f9" />
<img width="704" height="312" alt="image" src="https://github.com/user-attachments/assets/5e4ac8d3-165a-41f4-bcec-75a1aca43bb5" />

**Étape 3 : Patch smali – Supprimer le message « tampered » / root**  

1. Ouvrir le bon fichier:

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

9. Signer l’APK (obligatoire)
   
<img width="857" height="368" alt="image" src="https://github.com/user-attachments/assets/94e58a59-9a86-4508-af79-f0b115dab4fe" />

<img width="848" height="169" alt="image" src="https://github.com/user-attachments/assets/c225ff29-22d7-4ebe-ad95-efd7b342d55a" />

11. Installer et tester
    
<img width="853" height="92" alt="image" src="https://github.com/user-attachments/assets/f05009db-94aa-45d3-9ee0-5cb8c59810a4" />

**Étape 4 : Patch de la librairie native avec Ghidra (anti-debug + anti-Frida)**  

La librairie native libfoo.so a été importée dans Ghidra via la création d’un nouveau projet. Cette étape permet d’analyser le code natif compilé (C/C++) de l’application Android.

<img width="586" height="437" alt="image" src="https://github.com/user-attachments/assets/c0779476-09d8-45da-8043-47b837678e3c" />

Une analyse automatique a été lancée afin que Ghidra identifie les fonctions, les variables et les structures internes de la librairie. Cette étape est essentielle pour faciliter la compréhension du code natif.

<img width="640" height="359" alt="image" src="https://github.com/user-attachments/assets/7563aa65-d967-4d6b-bf03-090c5a2f736c" />

Une recherche de chaînes de caractères a été effectuée dans la librairie. La présence de la chaîne "frida" indique que l’application implémente une protection contre l’outil Frida, utilisé pour le reverse engineering dynamique.

<img width="678" height="501" alt="image" src="https://github.com/user-attachments/assets/2af2983e-bf90-4964-af4b-8eaf3be1aff8" /> 

La fonction FUN_00103910 a été identifiée comme responsable de la protection anti-debug. Le code montre l’utilisation de fonctions système telles que fork(), ptrace() et waitpid(), qui permettent de détecter la présence d’un debugger. 
Pour contourner cette protection, la première instruction de la fonction a été remplacée par l’instruction RET. Cette modification force la fonction à retourner immédiatement sans exécuter les mécanismes de détection. Ainsi, les protections anti-debug et anti-Frida sont désactivées. 

<img width="803" height="348" alt="image" src="https://github.com/user-attachments/assets/cad15d66-59b5-470c-a4af-0b78c1b877fe" />

**Étape 5 : Analyser la logique native de vérification dans libuncrackable3.so** 

- Retrouver la fonction native correspondante
Dans Ghidra :
Allez dans Symbol Tree → cherchez Java_sg_vantagepoint_uncrackable3_Check_check_code

<img width="959" height="471" alt="image" src="https://github.com/user-attachments/assets/8d3deafd-4d08-4f71-9131-54da72dd52dc" />

Observer la structure générale du pseudo-code

<img width="959" height="438" alt="image" src="https://github.com/user-attachments/assets/83c8fc4d-dadf-4f06-8ff8-c6011ee4e467" />

Dans la fonction native FUN_001012c0, plusieurs éléments indiquent la présence d’obfuscation. On observe notamment l’utilisation d’un générateur pseudo-aléatoire de type LCG (0x41c64e6d + 0x3039), des appels répétés à malloc(0x10) ainsi que la construction d’une liste chaînée complexe (_1_sub_doit__opaque_list1_1). Ces éléments n’ont aucun impact sur la logique de vérification du mot de passe et servent uniquement à compliquer l’analyse statique. On en déduit que cette partie du code constitue du bruit d’obfuscation destiné à masquer la logique réelle de l’application.

<img width="383" height="215" alt="image" src="https://github.com/user-attachments/assets/51afb49d-3cc3-4bd2-9dbc-e96f8f5beb74" />
<img width="407" height="160" alt="image" src="https://github.com/user-attachments/assets/6a27b8c1-5320-4bf6-ab35-c84e182f7922" />
<img width="464" height="192" alt="image" src="https://github.com/user-attachments/assets/0459c7c7-da73-4419-a9f0-c7518dbaf297" />

5. Se concentrer sur la fin de la fonction
   
<img width="488" height="325" alt="image" src="https://github.com/user-attachments/assets/f28dfd45-d7c1-4755-ac7c-cc2e526dd23e" />

6. Relever les constantes importantes
   
1d 08 11 13 0f 17 49 15  0d 00 03 19 5a 1d 13 15
08 0e 5a 00 17 08 13 14

La fin de la fonction FUN_001012c0 contient les données utiles du challenge. Trois constantes de type qword ont été identifiées, représentant au total 24 octets. Après conversion en format little-endian, ces valeurs correspondent à une clé encodée. Ces données seront utilisées dans une étape ultérieure pour reconstituer la clé secrète via une opération XOR.

### Questions de réflexion

**Pourquoi l’obfuscateur ajoute-t-il autant d’instructions répétitives ?**  
L’obfuscateur ajoute des instructions répétitives afin de compliquer l’analyse statique et ralentir le reverse engineering. Ces blocs inutiles augmentent artificiellement la taille et la complexité du code, rendant difficile l’identification de la logique réelle.

**Pourquoi les écritures finales dans param_1 sont-elles plus importantes que les 90 malloc ?**  
Les écritures finales dans param_1 contiennent les données réellement utilisées pour la vérification du mot de passe, contrairement aux nombreux appels à malloc, qui ne servent qu’à créer du bruit, ces écritures construisent la clé encodée utilisée dans la comparaison.

**Quel avantage de sécurité apporte une vérification native par rapport à une vérification pure Java ?**  
La vérification native est plus difficile à analyser car elle est compilée en code machine et non en bytecode Java facilement décompilable, cela rend le reverse engineering plus complexe et protège mieux la logique sensible.

**Comment un développeur défensif pourrait-il rendre cette clé encore plus difficile à extraire ?**  
Un développeur pourrait utiliser du chiffrement dynamique, générer la clé à l’exécution, combiner plusieurs techniques d’obfuscation, ou encore vérifier l’environnement d’exécution (anti-debug, anti-Frida) pour compliquer davantage l’analyse.

### Validation de l’étape

La méthode check.check_code() est responsable de la validation finale de l’entrée utilisateur, elle délègue cette vérification à une fonction native afin de masquer la logique. La fonction `FUN_001012c0` contient cette logique, mais elle est fortement obfusquée avec du bruit comme des appels répétés à malloc, un générateur pseudo-aléatoire (LCG) et des structures inutiles. Ces éléments compliquent l’analyse mais ne participent pas à la vérification. En réalité, la partie essentielle se situe à la fin de la fonction, où un buffer est rempli avec des constantes encodées. Ce buffer est ensuite utilisé pour comparer l’entrée utilisateur, ce qui en fait l’élément clé du challenge.







