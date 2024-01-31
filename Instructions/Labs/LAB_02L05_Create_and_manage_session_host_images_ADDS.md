---
lab:
  title: 'Labo : Créer et gérer des images d’hôte de session (AD DS)'
  module: 'Module 2: Implement an AVD Infrastructure'
---

# Labo - Créer et gérer des images d’hôte de session (AD DS)
# Manuel de labo de l’étudiant

## Éléments nécessaires pour le labo

- Un abonnement Azure que vous utiliserez dans ce labo.
- Un compte Microsoft ou un compte Microsoft Entra avec le rôle Propriétaire ou Contributeur dans l’abonnement Azure que vous utiliserez dans ce labo et avec le rôle Administrateur général dans le locataire Microsoft Entra associé à cet abonnement Azure.
- Le labo terminé **Préparer le déploiement d’Azure Virtual Desktop (AD DS)**

## Durée estimée

60 minutes

## Scénario du labo

Vous devez créer et gérer des images hôtes Azure Virtual Desktop dans un environnement Microsoft Entra DS.

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

- Créer et gérer des images d’hôte de session WVD

## Fichiers du labo

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.parameters.json

## Instructions

### Exercice 1 : Créer et gérer des images d’hôte de session
  
Les principales tâches de cet exercice sont les suivantes

1. Préparer la configuration d’une image d’hôte Azure Virtual Desktop
1. Déployer Azure Bastion
1. Configurer une image d’hôte Azure Virtual Desktop
1. Créer une image d’hôte Azure Virtual Desktop
1. Provisionner un pool d’hôtes Azure Virtual Desktop à l’aide de l’image personnalisée

#### Tâche 1 : Préparer la configuration d’une image d’hôte Azure Virtual Desktop

1. À partir de votre ordinateur de labo, démarrez un navigateur web, accédez au [portail Azure](https://portal.azure.com), puis connectez-vous en utilisant les informations d’identification d’un compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utiliserez dans ce labo.
1. Dans le portail Azure, ouvrez le volet **Cloud Shell** en sélectionnant l’icône de barre d’outils juste à droite de la zone de texte de recherche.
1. Lorsque vous êtes invité à sélectionner **Bash** ou **PowerShell**, sélectionnez **PowerShell**. 
1. Sur l’ordinateur de labo, dans le navigateur web affichant le portail Azure, à partir de la session PowerShell dans le volet Cloud Shell, exécutez ce qui suit pour créer un groupe de ressources qui contiendra l’image d’hôte Azure Virtual Desktop :

   ```powershell
   $vnetResourceGroupName = 'az140-11-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $vnetResourceGroupName).Location
   $imageResourceGroupName = 'az140-25-RG'
   New-AzResourceGroup -Location $location -Name $imageResourceGroupName
   ```

1. Dans le portail Azure, dans la barre d’outils du volet Cloud Shell, sélectionnez l’icône **Charger/télécharger des fichiers**, dans le menu déroulant, sélectionnez **Charger**, puis chargez les fichiers **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_ azuredeployvm25.json** et **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-25_azuredeployvm25.parameters.json** dans le répertoire de base Cloud Shell.
1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez ce qui suit pour déployer une machine virtuelle Azure exécutant Windows 10 qui servira de client Azure Virtual Desktop dans le sous-réseau nouvellement créé :

   ```powershell
   New-AzResourceGroupDeployment `
     -ResourceGroupName $imageResourceGroupName `
     -Name az140lab0205vmDeployment `
     -TemplateFile $HOME/az140-25_azuredeployvm25.json `
     -TemplateParameterFile $HOME/az140-25_azuredeployvm25.parameters.json
   ```

   > **Remarque** : Attendez que le déploiement se termine avant de passer à l’exercice suivant. Le déploiement peut prendre environ 10 minutes.

#### Tâche 2 : Déployer Azure Bastion 

> **Remarque** : Azure Bastion autorise la connexion aux machines virtuelles Azure sans les points de terminaison publics que vous avez déployés dans la tâche précédente de cet exercice, tout en fournissant une protection contre les attaques par force brute qui ciblent les informations d’identification au niveau du système d’exploitation.

> **Remarque** : Vérifiez que votre navigateur dispose de la fonctionnalité de fenêtre contextuelle activée.

1. Dans la fenêtre du navigateur affichant le portail Azure, ouvrez un autre onglet, puis accédez au portail Azure à partir de cet onglet.
1. Dans le portail Azure, ouvrez le volet **Cloud Shell** en sélectionnant l’icône de barre d’outils juste à droite de la zone de texte de recherche.
1. Dans la session PowerShell du volet Cloud Shell, exécutez la commande suivante pour ajouter un sous-réseau nommé **AzureBastionSubnet** au réseau virtuel nommé **az140-25-vnet** que vous avez créé précédemment dans cet exercice :

   ```powershell
   $resourceGroupName = 'az140-25-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-25-vnet'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.25.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Fermez le volet Cloud Shell.
1. Dans le portail Azure, recherchez et sélectionnez **Bastions**, puis dans le panneau **Bastions**, sélectionnez **+ Créer**.
1. Sous l’onglet **De base** du panneau **Créer un bastion**, spécifiez les paramètres suivants et sélectionnez **Vérifier + créer** :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|le nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Resource group|**az140-25-RG**|
   |Nom|**az140-25-bastion**|
   |Région|la même région Azure dans laquelle vous avez déployé les ressources dans les tâches précédentes de cet exercice|
   |Niveau|**De base**|
   |Réseau virtuel|**az140-25-vnet**|
   |Sous-réseau|**AzureBastionSubnet (10.25.254.0/24)**|
   |Adresse IP publique|**Création**|
   |Nom de l’IP publique|**az140-25-vnet-ip**|

1. Sous l’onglet **Vérifier + créer** du panneau **Créer un bastion**, sélectionnez **Créer** :

   > **Remarque** : Attendez que le déploiement se termine avant de passer à l’exercice suivant. Le déploiement peut prendre environ 5 minutes.

#### Tâche 3 : Configurer une image d’hôte Azure Virtual Desktop

1. Dans le portail Azure, recherchez et sélectionnez **Machines virtuelles**, puis dans le panneau **Machines virtuelles**, sélectionnez **az140-25-vm0**.
1. Dans le panneau **az140-25-vm0**, sélectionnez **Se connecter**, puis dans le menu déroulant, sélectionnez **Bastion**. Sous l’onglet **Bastion** du panneau **az140-25-vm0\| Se connecter**, sélectionnez **Utiliser Bastion**.
1. Lorsque vous y êtes invité, fournissez les informations d’identification suivantes et sélectionnez **Se connecter** :

   |Paramètre|Valeur|
   |---|---|
   |Nom d’utilisateur|**Étudiant**|
   |Mot de passe|**Pa55w.rd1234**|

   > **Remarque** : Vous allez commencer par installer les fichiers binaires FSLogix.

1. Dans la session Bastion sur **az140-25-vm0**, démarrez **Windows PowerShell ISE** en tant qu’administrateur.
1. Dans la session Bastion sur **az140-25-vm0**, à partir de la console **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour créer un dossier que vous utiliserez comme emplacement temporaire pour la configuration de l’image :

   ```powershell
   New-Item -Type Directory -Path 'C:\Allfiles\Labs\02' -Force
   ```

1. Dans la session Bastion sur **az140-25-vm0**, démarrez Microsoft Edge, accédez à la [page de téléchargement de FSLogix](https://aka.ms/fslogix_download), téléchargez les fichiers binaires d’installation compressés FSLogix dans le dossier **C:\\Allfiles\\Labs\\02**, puis à partir de l’Explorateur de fichiers, extrayez le sous-dossier **x64** dans le même dossier.
1. Dans la session Bastion sur **az140-25-vm0**, basculez vers la fenêtre **Administrateur : Windows PowerShell ISE**, puis à partir de la console **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour effectuer l’installation de OneDrive par ordinateur :

   ```powershell
   Start-Process -FilePath 'C:\Allfiles\Labs\02\x64\Release\FSLogixAppsSetup.exe' -ArgumentList '/quiet' -Wait
   ```

   > **Remarque** : Attendez que l’installation se termine. Cela peut prendre environ 1 minute. Si l’installation déclenche un redémarrage, reconnectez-vous à **az140-25-vm0**.

   > **Remarque** : Ensuite, vous allez parcourir l’installation et la configuration de Microsoft Teams (pour votre apprentissage personnel, car Teams est déjà présent sur l’image utilisée pour ce labo).

1. Dans la session Bastion sur **az140-25-vm0**, cliquez avec le bouton droit sur **Démarrer**et sélectionnez **Exécuter** dans le menu contextuel. Dans la boîte de dialogue **Exécuter**, dans la zone de texte **Ouvrir**, tapez **cmd** et appuyez sur la touche **Entrée** pour démarrer l’**Invite de commandes**.
1. Dans la fenêtre **Administrateur : C:\windows\system32\cmd.exe**, à partir de l’invite de commandes, exécutez ce qui suit pour préparer l’installation par ordinateur de Microsoft Teams :

   ```cmd
   reg add "HKLM\Software\Microsoft\Teams" /v IsWVDEnvironment /t REG_DWORD /d 1 /f
   ```

1. Dans la session Bastion sur **az140-25-vm0**, dans Microsoft Edge, accédez à la [page de téléchargement de Microsoft Visual C++ Redistributable](https://aka.ms/vs/16/release/vc_redist.x64.exe) et enregistrez **VC_redist.x64** dans le dossier **C:\\Allfiles\\Labs\\02**.
1. Dans la session Bastion sur **az140-25-vm0**, basculez vers la fenêtre **Administrateur : C:\windows\system32\cmd.exe**, puis à partir de l’invite de commandes, exécutez ce qui suit pour effectuer l’installation de Microsoft Visual C++ Redistributable :

   ```cmd
   C:\Allfiles\Labs\02\vc_redist.x64.exe /install /passive /norestart /log C:\Allfiles\Labs\02\vc_redist.log
   ```

1. Dans la session Bastion sur **az140-25-vm0**, dans Microsoft Edge, accédez à la page de documentation intitulée [Déployer l’application de bureau Teams sur la machine virtuelle](https://docs.microsoft.com/en-us/microsoftteams/teams-for-vdi#deploy-the-teams-desktop-app-to-the-vm), cliquez sur le lien **Version 64 bits**, puis lorsque vous y êtes invité, enregistrez le fichier **Teams_windows_x64.msi** dans le dossier **C:\\Allfiles\\Labs\\02**.
1. Dans la session Bastion sur **az140-25-vm0**, basculez vers la fenêtre **Administrateur : C:\windows\system32\cmd.exe**, puis à partir de l’invite de commandes, exécutez ce qui suit pour effectuer l’installation par ordinateur de Microsoft Teams :

   ```cmd
   msiexec /i C:\Allfiles\Labs\02\Teams_windows_x64.msi /l*v C:\Allfiles\Labs\02\Teams.log ALLUSER=1
   ```

   > **Remarque** : Le programme d’installation prend en charge les paramètres ALLUSER=1 et ALLUSERS=1. Le paramètre ALLUSER=1 est réservé aux installations par ordinateur dans les environnements VDI. Le paramètre ALLUSERS=1 peut être utilisé dans des environnements VDI et non-VDI. 
   > **Remarque** Si vous rencontrez une erreur indiquant **Une autre version du produit est déjà installée**, effectuez les étapes suivantes : Accédez à **Panneau de configuration > Programmes > Programmes et fonctionnalités**, cliquez avec le bouton droit sur le **programme d’installation Teams à l’échelle de l’ordinateur**, puis sélectionnez **Désinstaller**. Passez à la suppression du programme et réexécutez l’étape 13 ci-dessus. 

1. Dans la session Bastion sur **az140-25-vm0**, démarrez **Windows PowerShell ISE** en tant qu’administrateur, puis à partir de la console **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour installer Microsoft Edge Chromium (pour votre apprentissage personnel, car Edge est déjà présent sur l’image utilisée pour ce labo).

   ```powershell
   Start-BitsTransfer -Source "https://aka.ms/edge-msi" -Destination 'C:\Allfiles\Labs\02\MicrosoftEdgeEnterpriseX64.msi'
   Start-Process -Wait -Filepath msiexec.exe -Argumentlist "/i C:\Allfiles\Labs\02\MicrosoftEdgeEnterpriseX64.msi /q"
   ```

   > **Remarque** : Attendez que l’installation se termine. Cela peut prendre environ 2 minutes.

   > **Remarque** : Si vous êtes dans un environnement multilingue, vous aurez probablement besoin d’installer des modules linguistiques. Pour obtenir les détails de cette procédure, reportez-vous à l’article Microsoft Docs [Ajouter des modules linguistiques à une image Windows 10 multisession](https://docs.microsoft.com/en-us/azure/virtual-desktop/language-packs).

   > **Remarque** : Ensuite, vous allez désactiver les Mises à jour automatiques Windows, désactiver l’Assistant Stockage, configurer la redirection du fuseau horaire et configurer la collecte des données de télémétrie. Normalement, vous devez d’abord appliquer toutes les mises à jour actuelles en premier. Pour réduire la durée du labo, ignorez cette étape.

1. Dans la session Bastion sur **az140-25-vm0**, basculez vers la fenêtre **Administrateur : C:\windows\system32\cmd.exe**, puis à partir de l’invite de commandes, exécutez ce qui suit pour désactiver les Mises à jour automatiques :

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v NoAutoUpdate /t REG_DWORD /d 1 /f
   ```

1. Dans la fenêtre **Administrateur : C:\windows\system32\cmd.exe**, à partir de l’invite de commandes, exécutez ce qui suit pour désactiver l’Assistant Stockage :

   ```cmd
   reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\StorageSense\Parameters\StoragePolicy" /v 01 /t REG_DWORD /d 0 /f
   ```

1. Dans la fenêtre **Administrateur : C:\windows\system32\cmd.exe**, à partir de l’invite de commandes, exécutez ce qui suit pour configurer la redirection du fuseau horaire :

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fEnableTimeZoneRedirection /t REG_DWORD /d 1 /f
   ```

1. Dans la fenêtre **Administrateur : C:\windows\system32\cmd.exe**, à partir de l’invite de commandes, exécutez ce qui suit pour désactiver la collecte des données de télémétrie du Hub de commentaires :

   ```cmd
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection" /v AllowTelemetry /t REG_DWORD /d 0 /f
   ```

1. Dans la fenêtre **Administrateur : C:\windows\system32\cmd.exe**, à partir de l’invite de commandes, exécutez ce qui suit pour supprimer le dossier temporaire que vous avez créé précédemment dans cette tâche :

   ```cmd
   rmdir C:\Allfiles /s /q
   ```

1. Dans la fenêtre **Administrateur : C :\windows\system32\cmd.exe**, à partir de l’invite de commandes, exécutez l’utilitaire Nettoyage de disque, puis cliquez sur **OK** une fois terminé :

   ```cmd
   cleanmgr /d C: /verylowdisk
   ```

#### Tâche 4 : Créer une image d’hôte Azure Virtual Desktop

1. Dans la session Bastion sur **az140-25-vm0**, dans la fenêtre **Administrateur : C:\windows\system32\cmd.exe**, à partir de l’invite de commandes, exécutez l’utilitaire sysprep pour préparer le système d’exploitation afin de générer une image et l’arrêter automatiquement :

   ```cmd
   C:\Windows\System32\Sysprep\sysprep.exe /oobe /generalize /shutdown /mode:vm
   ```

   > **Remarque** : Attendez la fin du processus de sysprep. Cela peut prendre environ 2 minutes. Cela arrête automatiquement le système d’exploitation. 

1. Sur l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Machines virtuelles**, puis dans le panneau **Machines virtuelles**, sélectionnez **az140-25-vm0**.
1. Dans le panneau **az140-25-vm0**, dans la barre d’outils située au-dessus de la section **Fonctionnalités essentielles**, cliquez sur **Actualiser**. Vérifiez que l’**État**de la machine virtuelle Azure est passé sur **Arrêtée**. Cliquez sur **Arrêter**, puis lorsque vous êtes invité à confirmer, cliquez sur **OK** pour basculer la machine virtuelle Azure vers l’état **Arrêtée (désallouée)**.
1. Dans le panneau **az140-25-vm0**, vérifiez que l’**État**de la machine virtuelle Azure est passé sur **Arrêtée (désallouée)**, puis dans la barre d’outils, cliquez sur **Capturer**. Cela affiche automatiquement le panneau **Créer une image** .
1. Sous l’onglet **Informations de base** du panneau **Créer une image**, spécifiez les paramètres suivants :

   |Paramètre|Valeur|
   |---|---|
   |Partager l’image dans la galerie de calcul Azure|**Oui, la partager dans une galerie sous forme de version d’image**|
   |Supprimer automatiquement cette machine virtuelle après avoir créé l’image|case décochée|
   |Galerie de calcul Azure cible|nom d’une nouvelle galerie **az14025imagegallery**|
   |État du système d'exploitation|**Généralisé**|

1. Sous l’onglet **Informations de base** du panneau **Créer une image**, sous la zone de texte **Définition d’image de machine virtuelle cible**, cliquez sur **Créer nouveau**.
1. Dans **Créer une définition d’image de machine virtuelle**, spécifiez les paramètres suivants et cliquez sur **OK** :

   |Paramètre|Valeur|
   |---|---|
   |Nom de la définition d’image de machine virtuelle|**az140-25-host-image**|
   |Serveur de publication|**MicrosoftWindowsDesktop**|
   |Offre|**office-365**|
   |SKU|**win11-22h2-avd-m365**|

1. De retour sous l’onglet **Informations de base** du panneau **Créer une image**, spécifiez les paramètres suivants et cliquez sur **Vérifier + créer** :

   |Paramètre|Valeur|
   |---|---|
   |Numéro de version|**1.0.0**|
   |Exclure de la plus récente|case décochée|
   |Date de fin de vie|Un an après la date actuelle|
   |Nombre de réplicas par défaut|**1**|
   |Nombre de réplicas de la région cible|**1**|
   |Type de compte de stockage|**SSD Premium LRS**|

1. Sous l’onglet **Vérifier + créer** du panneau **Créer une image**, cliquez sur **Créer**.

   > **Remarque** : Attendez la fin du déploiement. Ceci peut prendre environ 20 minutes.

1. Sur votre ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Galeries de calcul Azure**. Dans le panneau **Galeries de calcul Azure**, sélectionnez l’entrée **az14025imagegallery**, puis dans le panneau ****az14025imagegallery****, vérifiez la présence de l’entrée **az140-25-host-image** représentant l’image qui vient d’être créée.

#### Tâche 5 : Provisionner un pool d’hôtes Azure Virtual Desktop à l’aide d’une image personnalisée

1. Sur l’ordinateur de labo, dans le portail Azure, utilisez la zone de texte **Rechercher dans les ressources, services et documents** en haut de la page pour rechercher et accéder à **Réseaux virtuels**, puis dans le panneau **Réseaux virtuels**, sélectionnez **az140-adds-vnet11**. 
1. Dans le panneau **az140-adds-vnet11**, sélectionnez **Sous-réseaux**. Dans le panneau **Sous-réseaux**, sélectionnez **+ Sous-réseau**, puis dans le panneau **Ajouter un sous-réseau**, spécifiez les paramètres suivants (laissez tous les autres paramètres avec leurs valeurs par défaut) et cliquez sur **Enregistrer** :

   |Paramètre|Valeur|
   |---|---|
   |Nom|**hp4-Subnet**|
   |Plage d’adresses de sous-réseau|**10.0.4.0/24**|

1. Sur l’ordinateur de labo, dans le portail Azure, dans la fenêtre de navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop**. Dans le panneau **Azure Virtual Desktop**, sélectionnez **Pools d’hôtes**, puis dans le panneau **Azure Virtual Desktop \| Pools d’hôtes**, sélectionnez **+ Créer**. 
1. Sous l’onglet **Informations de base** du panneau **Créer un pool d’hôtes**, spécifiez les paramètres suivants et sélectionnez **Suivant : Machines virtuelles >** :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|le nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Resource group|**az140-25-RG**|
   |Nom du pool d’hôtes|**az140-25-hp4**|
   |Emplacement|nom de la région Azure dans laquelle vous avez déployé des ressources dans le premier exercice de ce labo|
   |Environnement de validation|**Aucun**|
   |Type de pool d’hôtes|**Groupé**|
   |Limite de session maximale|**50**|
   |Algorithme d’équilibrage de charge|**À largeur prioritaire**|

1. Sous l’onglet **Machines virtuelles** du panneau **Créer un pool d’hôtes**, spécifiez les paramètres suivants :

   |Paramètre|Valeur|
   |---|---|
   |Ajouter des machines virtuelles Azure|**Oui**|
   |Resource group|**Par défaut, identique à celui du pool d’hôtes**|
   |Préfixe de nom|**az140-25-p4**|
   |Emplacement des machines virtuelles|nom de la région Azure dans laquelle vous avez déployé des ressources dans le premier exercice de ce labo|
   |Options de disponibilité|**Aucune redondance de l’infrastructure requise**|
   
1. Sous l’onglet **Machines virtuelles** du panneau **Créer un pool d’hôtes**, directement sous la liste déroulante **Image**, cliquez sur le lien **Voir toutes les images**.
1. Dans le panneau **Sélectionner une image**, sous **Autres éléments**, cliquez sur **Images partagées**, puis dans la liste des images partagées, sélectionnez **az140-25-host-image**. 
1. De retour sous l’onglet **Machines virtuelles** du panneau **Créer un pool d’hôtes**, spécifiez les paramètres suivants et sélectionnez **Suivant : Espace de travail >**

   |Paramètre|Valeur|
   |---|---|
   |Taille de la machine virtuelle|**Standard D2s v3**|
   |Nombre d'ordinateurs virtuels|**1**|
   |Type de disque du système d’exploitation|**SSD Standard**|
   |Réseau virtuel|**az140-adds-vnet11**|
   |Sous-réseau|**hp4-Subnet (10.0.4.0/24)**|
   |Groupe de sécurité réseau|**De base**|
   |Ports d’entrée publics|**Oui**|
   |Ports d’entrée à autoriser|**RDP**|
   |UPN de jonction de domaine AD|**student@adatum.com**|
   |Mot de passe|**Pa55w.rd1234**|
   |Spécifier un domaine ou une unité|**Oui**|
   | Domaine à rejoindre|**adatum.com**|
   |Chemin de l’unité d’organisation|**OU=WVDInfra,DC=adatum,DC=com**|
   |Nom d'utilisateur|Étudiant|
   |Mot de passe|Pa55w.rd1234|
   |Confirmer le mot de passe|Pa55w.rd1234|

1. Sous l’onglet **Espace de travail** du panneau **Créer un pool d’hôtes**, spécifiez les paramètres suivants et sélectionnez **Vérifier + créer** :

   |Paramètre|Valeur|
   |---|---|
   |Inscrire un groupe d'applications de bureau|**Aucun**|

1. Sous l’onglet **Vérifier + créer** du panneau **Créer un pool d’hôtes**, sélectionnez **Créer**.

   > **Remarque** : Attendez la fin du déploiement. Ceci peut prendre environ 10 minutes.
   > 
   > **Remarque** Si le déploiement échoue parce que la limite de quota a été atteinte, effectuez les étapes décrites dans le premier labo pour demander automatiquement une augmentation de quota de la limite Standard D2sv3 à 30.

   > **Remarque** : Après le déploiement d’hôtes basés sur des images personnalisées, pensez à exécuter l’outil Virtual Desktop Optimization Tool, disponible dans [son dépôt GitHub](https://github.com/The-Virtual-Desktop-Team/).


### Exercice 2 : Arrêter et libérer les machines virtuelles Azure provisionnées dans le labo

Les principales tâches de cet exercice sont les suivantes

1. Arrêter et libérer les machines virtuelles Azure provisionnées dans le labo

>**Remarque** : Dans cet exercice, vous allez libérer les machines virtuelles Azure provisionnées dans ce labo pour réduire les frais de calcul correspondants

#### Tâche 1 : Libérer les machines virtuelles Azure provisionnées dans le labo

1. Basculez sur l’ordinateur de labo, puis dans la fenêtre de navigateur web affichant le portail Azure, ouvrez la session shell **PowerShell** dans le volet **Cloud Shell**.
1. Dans la session PowerShell du volet Cloud Shell, exécutez la commande suivante pour lister toutes les machines virtuelles Azure créées dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-25-RG'
   ```

1. Dans la session PowerShell du volet Cloud Shell, exécutez la commande suivante pour arrêter et libérer toutes les machines virtuelles Azure que vous avez créées dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-25-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Remarque** : La commande s’exécute de manière asynchrone (comme l’indique le paramètre -NoWait), donc même si vous pouvez exécuter une autre commande PowerShell dans la même session PowerShell immédiatement après, il faut quelques minutes avant que les machines virtuelles Azure ne soient réellement arrêtées et libérées.
