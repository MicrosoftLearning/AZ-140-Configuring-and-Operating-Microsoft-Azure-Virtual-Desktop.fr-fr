---
lab:
  title: 'Labo : Package des applications Azure Virtual Desktop (AD DS)'
  module: 'Module 4: Manage User Environments and Apps'
---

# Labo - Empaqueter des applications Azure Virtual Desktop (AD DS)
# Manuel de labo de l’étudiant

## Dépendances du labo

- Un abonnement Azure
- Un compte Microsoft ou un compte Microsoft Entra disposant du rôle Administrateur général pour le locataire Microsoft Entra qui est associé à l’abonnement Azure, et disposant du rôle Propriétaire ou Contributeur dans l’abonnement Azure
- Le labo terminé **Préparer le déploiement d’Azure Virtual Desktop (AD DS)**
- Le labo terminé **gestion des profils Azure Virtual Desktop (AD DS)**
- Le labo terminé **Configurer des stratégies d’accès conditionnel pour WVD (AD DS)**

## Durée estimée

60 minutes

## Scénario du labo

Vous devez combiner et déployer des applications Azure Virtual Desktop dans un environnement Active Directory Domain Services (AD DS).

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

- Préparer et créer des packages d’applications MSIX
- Implémenter une image d’attachement d’application MSIX pour Azure Virtual Desktop dans un environnement Microsoft Entra DS
- Implémenter l’attachement d’application MSIX sur Azure Virtual Desktop dans l’environnement AD DS

## Fichiers du labo

-  \\\\AZ-140\\AllFiles\\Labos\\04\\az140-42_azuredeploycl42.json
-  \\\\AZ-140\\AllFiles\\Labos\\04\\az140-42_azuredeploycl42.parameters.json

## Instructions

>**Important** : Microsoft a renommé **Azure Active Directory** (**Azure AD**) **Microsoft Entra ID**. Pour plus d’informations sur cette modification, consultez [Le nouveau nom d’Azure Active Directory](https://learn.microsoft.com/en-us/entra/fundamentals/new-name). Puisque cette modification est encore en cours d’application, vous rencontrerez peut-être encore des cas d’incompatibilité entre l’instruction du labo et les éléments de l’interface à mesure que vous parcourez chaque exercice. Gardez cela à l’esprit (en particulier, dans ce labo, où **Microsoft Entra Connect** désigne le nouveau nom d’**Azure Active Directory Connect**).

### Exercice 1 : Préparer et créer des packages d’applications MSIX

Les principales tâches de cet exercice sont les suivantes

1. Préparer la configuration des hôtes de session Azure Virtual Desktop
1. Déployer une machine virtuelle Azure exécutant Windows 10 à l’aide d’un modèle de démarrage rapide Azure Resource Manager
1. Préparer la machine virtuelle Azure exécutant Windows 10 pour l’empaquetage MSIX
1. Générer un certificat de signature
1. Télécharger le logiciel dans le package
1. Installer l’outil d’empaquetage MSIX
1. Créer un package MSIX

#### Tâche 1 : Préparer la configuration des hôtes de session Azure Virtual Desktop

1. À partir de votre ordinateur labo, démarrez un navigateur web, accédez au [portail Azure](https://portal.azure.com), puis connectez-vous en fournissant des informations d’identification d’un compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utiliserez dans ce labo.
1. Sur l’ordinateur labo et, dans la fenêtre du navigateur web affichant le portail Azure, ouvrez la session shell **PowerShell** dans le volet **Cloud Shell**.
1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez ce qui suit pour démarrer la session Azure Virtual Desktop héberge des machines virtuelles Azure que vous utiliserez dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM -NoWait
   ```

   >**Remarque** : La commande s’exécute de manière asynchrone (telle que déterminée par le paramètre -NoWait), de sorte que vous serez en mesure d’exécuter une autre commande PowerShell immédiatement après la même session PowerShell, il faudra quelques minutes avant que les machines virtuelles Azure ne soient réellement démarrées. 

   >**Remarque** : Si vous avez activé PSRemoting sur les hôtes de session dans le groupe de ressources az140-21-RG dans la première tâche du labo précédent (Implémenter et gérer les profils AVD), vous pouvez passer directement à la tâche suivante sans attendre que les machines virtuelles Azure démarrent. Si vous n’avez pas précédemment activé PSRemoting sur les hôtes de session dans le groupe de ressources az140-21-RG, attendez que les machines virtuelles démarrent, puis exécutez la commande suivante.

1. À partir de la session PowerShell du **Cloud Shell**, exécutez la commande suivante pour activer la communication à distance PowerShell sur les hôtes de session.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Enable-AzVMPSRemoting
   ```
   
#### Tâche 2 : Déployer une machine virtuelle Azure exécutant Windows 10 à l’aide d’un modèle de démarrage rapide Azure Resource Manager

1. À partir de votre ordinateur labo, dans la fenêtre du navigateur web affichant le portail Azure, dans la barre d’outils du volet Cloud Shell, sélectionnez l’icône **Charger/télécharger des fichiers**, dans le menu déroulant, sélectionnez **Charger**, puis chargez les fichiers **\\\\AZ-140\\AllFiles\\Labos\\04\\az140-42_ azuredeploycl42.json** et **\\\\AZ-140\\AllFiles\\Labos\\04\\az140-42_azuredeploycl42.parameters.json** dans le répertoire de base Cloud Shell.
1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez ce qui suit pour déployer une machine virtuelle Azure exécutant Windows 10 que vous allez utiliser pour créer des packages MSIX et les joindre au domaine Microsoft Entra DS :

   ```powershell
   $vNetResourceGroupName = 'az140-11-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $vNetResourceGroupName).Location
   $resourceGroupName = 'az140-42-RG'
   New-AzResourceGroup -ResourceGroupName $resourceGroupName -Location $location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0402vmDeployment `
     -TemplateFile $HOME/az140-42_azuredeploycl42.json `
     -TemplateParameterFile $HOME/az140-42_azuredeploycl42.parameters.json
   ```

   > **Remarque** : Attendez que le déploiement se termine avant de passer à la tâche suivante. Ceci peut prendre environ 10 minutes. 

#### Tâche 3 : Préparer la machine virtuelle Azure exécutant Windows 10 pour l’empaquetage MSIX

1. À partir de votre ordinateur labo, dans le portail Azure, recherchez et sélectionnez **machines virtuelles** et, dans le panneau **machines virtuelles**, dans la liste des machines virtuelles, sélectionnez l’entrée **az140-cl-vm42**. Cela ouvre le panneau **az140-cl-vm42**.
1. Dans le panneau **az140-cl-vm42**, sélectionnez **Connecter**, dans le menu déroulant, sélectionnez **Bastion**, sous l’onglet **Bastion** du panneau **az140-cl-vm42\| Connect** , sélectionnez **Utiliser bastion**.
1. Lorsque vous y êtes invité, connectez-vous avec le nom d’utilisateur **wvdadmin1@adatum.com** et le mot de passe que vous définissez lors de la création de ce compte d’utilisateur. 
1. Dans la session Bastion pour **az140-cl-vm42**, démarrez **Windows PowerShell ISE** en tant qu’administrateur, à partir de l’administrateur **: Windows PowerShell ISE** console, exécutez la commande suivante pour préparer le système d’exploitation pour l’empaquetage MSIX :

   ```powershell
   Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable
   reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
   reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f
   reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f
   reg add HKLM\Software\Microsoft\RDInfraAgent\MSIXAppAttach /v PackageListCheckIntervalMinutes /t REG_DWORD /d 1 /f
   reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
   ```

   > **Remarque** : La dernière de ces modifications de Registre désactive le contrôle d’accès utilisateur. Cela n’est techniquement pas obligatoire, mais simplifie le processus illustré dans ce labo.

#### Tâche 4 : Générer un certificat de signature

> **Remarque** : Dans ce laboratoire, vous allez utiliser un certificat auto-signé. Dans un environnement de production, vous devez utiliser un certificat émis par une autorité de certification publique ou un certificat interne, selon l’utilisation prévue.

1. Dans la session Bastion pour **az140-cl-vm42**, à partir de l’administrateur **: Console Windows PowerShell ISE**, exécutez ce qui suit pour générer un certificat auto-signé avec l’attribut Common Name défini sur **Adatum**, puis stockez le certificat dans le dossier **Personnel** du magasin de certificats **ordinateur local** :

   ```powershell
   New-SelfSignedCertificate -Type Custom -Subject "CN=Adatum" -KeyUsage DigitalSignature -KeyAlgorithm RSA -KeyLength 2048 -CertStoreLocation "cert:\LocalMachine\My"
   ```

1. Depuis la console **Administrateur : La console Windows PowerShell ISE**, exécutez la commande suivante pour démarrer la console **Certificats** ciblant le magasin de certificats de l’ordinateur local :

   ```powershell
   certlm.msc
   ```

1. Dans le volet **Certificats** console, développez le dossier **Personnel**, sélectionnez le sous-dossier **Certificats**, cliquez avec le bouton droit sur le certificat**Adatum**, dans le menu contextuel, sélectionnez **Toutes les tâches** suivi de **Exporter**. **L’assistant Exportation de certificat** est lancé. 
1. Dans la page **Bienvenue dans l’assistant Exportation de certificats** de **l’assistant Exportation de certificat **, sélectionnez **Suivant**.
1. Dans la page **Exporter la clé privée** de **l’assistant Exportation de certificat **, sélectionnez l’option **Oui, exportez l’option clé privée** et sélectionnez **suivant**.
1. Dans la page **Exporter le format de fichier** de **l’assistant Exportation de certificat**, cochez la case **Exporter toutes les propriétés étendues**, décochez la case **Activer la confidentialité du certificat**, puis sélectionnez **Suivant**.
1. Dans la page **Sécurité** de **l’assistant Exportation de certificat**, cochez la case **Mot de passe**, dans les zones de texte ci-dessous, tapez **Pa55w.rd1234**, puis sélectionnez **suivant**.
1. Dans la page **Fichier à exporter** de **l’assistant Exportation de certificat**, dans la zone de texte **nom de fichier**, sélectionnez **Parcourir**, dans la boîte de dialogue **Enregistrer sous**, accédez au **C :\\Tous les fichiers\\Labos\\dossier** 04 (créez d’abord le dossier), dans la zone de texte **nom de fichier**, tapez **adatum.pfx**, puis sélectionnez **Enregistrer**.
1. De retour sur la page **Fichier à exporter** de **l' assistant Exportation de certificat**, vérifiez que la zone de texte contient l’entrée **C :\\Tous les fichiers\\Labos\\04\\adatum.pfx**, puis sélectionnez **Suivant**.
1. Dans la page **Fin de l’assistant Exportation de certificat** de **l’assistant Exportation de certificat **, sélectionnez **Terminer**, puis sélectionnez **OK** pour confirmer l’exportation réussie. 

   > **Remarque** : Étant donné que vous utilisez un certificat auto-signé, vous devez l’installer dans le magasin de certificats **personnes approuvées** sur les hôtes de session cible.

1. Depuis la console **Administrateur : Console Windows PowerShell ISE**, exécutez la commande suivante pour installer le certificat nouvellement généré dans le magasin de certificats **personnes approuvées** sur les hôtes de session cible :

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   $cleartextPassword = 'Pa55w.rd1234'
   $securePassword = ConvertTo-SecureString $cleartextPassword -AsPlainText -Force
   $localPath = 'C:\Allfiles\Labs\04'
   ForEach ($wvdhost in $wvdhosts){
      $remotePath = "\\$wvdhost\C$\Allfiles\Labs\04\"
      New-Item -ItemType Directory -Path $remotePath -Force
      Copy-Item -Path "$localPath\adatum.pfx" -Destination $remotePath -Force
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Import-PFXCertificate -CertStoreLocation Cert:\LocalMachine\TrustedPeople -FilePath 'C:\Allfiles\Labs\04\adatum.pfx' -Password $using:securePassword
      } 
   }
   ```

#### Tâche 5 : Télécharger le logiciel dans le package

1. Dans la session Bastion pour **az140-cl-vm42**, démarrez **Microsoft Edge** et accédez à **https://github.com/microsoft/XmlNotepad**.
1. Dans la page **microsoft/XmlNotepad** **readme.md**, sélectionnez le lien de téléchargement pour le programme d’installation téléchargeable autonome et téléchargez les fichiers d’installation compressés.
1. Dans la session Bastion pour **az140-cl-vm42**, démarrez l’Explorateur de fichiers, accédez au dossier **Téléchargements**, ouvrez le fichier compressé, copiez le contenu du dossier dans le fichier compressé et collez-le dans le répertoire **C :\\AllFiles\\Labos\\04\\** répertoire. 

#### Tâche 6 : Installer l’outil d’empaquetage MSIX

1. Dans la session Bastion pour **az140-cl-vm42**, démarrez l’application **Microsoft Store**.
1. Dans l’application du **Microsoft Store**, recherchez et sélectionnez **MSIX Packaging Tool**, dans la page **l’outil d’empaquetage MSIX**, sélectionnez **Obtenir**.
1. Lorsque vous y êtes invité, ignorez la connexion, attendez que l’installation se termine, sélectionnez **Ouvrir** et, dans la boîte de dialogue **Envoyer des données de diagnostic**, sélectionnez **Refuser**, 

#### Tâche 7 : Créer un package MSIX

1. Dans la session Bastion pour **az140-cl-vm42**, basculez vers l’administrateur **: Fenêtre Windows PowerShell ISE ** et, à partir de ** l’administrateur: Volet de script Windows PowerShell ISE**, exécutez ce qui suit pour désactiver le service Recherche Windows :

   ```powershell
   $serviceName = 'wsearch'
   Set-Service -Name $serviceName -StartupType Disabled
   Stop-Service -Name $serviceName
   ```

1. Dans la session Bastion pour **az140-cl-vm42**, à partir de l’administrateur **: Windows PowerShell ISE** console, exécutez la commande suivante pour créer le dossier qui hébergera le package MSIX :

   ```powershell
   New-Item -ItemType Directory -Path 'C:\AllFiles\Labs\04\XmlNotepad' -Force
   ```

1. Dans la session Bastion pour **az140-cl-vm42**, à partir de l’administrateur **: Windows PowerShell ISE** console, exécutez la commande suivante pour supprimer le flux de données de remplacement Zone.Identifier des fichiers d’installation extraits, qui a la valeur « 3 » pour indiquer qu’ils ont été téléchargés à partir d’Internet :

   ```powershell
   Get-ChildItem -Path 'C:\AllFiles\Labs\04' -Recurse -File | Unblock-File
   ```

1. Dans la session Bastion pour **az140-cl-vm42**, basculez vers l’interface **MSIX Packaging Tool**, dans la page **Sélectionner la tâche**, sélectionnez l’entrée **package d’application - Créer votre package d’application**. Cela démarre l’assistant **Création d’un package**.
1. Dans la page **Sélectionner l’environnement** de l’assistant **Créer un package**, vérifiez que l’option **Créer un package sur cet ordinateur** est sélectionnée, sélectionnez **Suivant**, puis attendez l’installation du pilote **MSIX Packaging Tool Driver**.
1. Dans la page **Préparer l’ordinateur** de l’assistant **Créer un package**, passez en revue les recommandations. S’il existe un redémarrage en attente, redémarrez le système d’exploitation, reconnectez-vous à l’aide du compte **wvdadmin1@adatum.com** et redémarrez **MSIX Packaging Tool** avant de continuer. 

   >**Remarque** : L’outil d’empaquetage MSIX désactive temporairement Windows Update et Windows Search. Dans ce cas, le service Windows Search est déjà désactivé. 

1. Dans la page **Préparer l’ordinateur** de l’assistant **Créer un nouveau package**, cliquez sur **Suivant**.
1. Dans la page **Sélectionner le programme d’installation** de l’assistant **Créer un nouveau package**, à côté de la zone de texte**Choisir le programme d’installation que vous souhaitez combiner**, sélectionnez **Parcourir**, dans la boîte de dialogue **Ouvrir**, accédez au dossier **C :\\AllFiles\\Labos\\04**, sélectionnez **XmlNotepadSetup.msi**, puis cliquez sur **Ouvrir**, 
1. Dans la page **Sélectionner le programme d’installation** de l’assistant **Création d’un nouveau package**, dans la liste déroulante **Préférence de signature**, sélectionnez l’entrée Signer avec un certificat (.pfx), en regard de la zone de texte **Rechercher le certificat**, Sélectionnez **Parcourir**, dans la boîte de dialogue **Ouvrir**, accédez au **C :\\AllFiles\\Labos\\04** dossier, sélectionnez le fichier **adatum.pfx**, cliquez sur **Ouvrir**, dans la zone de texte **mot de passe**, tapez **Pa55w.rd1234**, puis sélectionnez **Suivant**.
1. Dans la page **Informations du package** de l’Assistant **Créer un nouveau package**, passez en revue les informations du package, vérifiez que le nom de l’éditeur est défini sur **CN=Adatum**, puis sélectionnez **Suivant**. Cela déclenche l’installation du logiciel téléchargé.
1. Dans la **Fenêtre d’installation** de XMLNotepad, acceptez les termes du Contrat de licence et sélectionnez **Installer** et, une fois l’installation terminée, cochez la case **Lancer le Bloc-notes XML**, puis sélectionnez **Terminer**.
1. Lorsque vous y êtes invité, dans la fenêtre **Bloc-notes XML**, sélectionnez **Non**, vérifiez que le Bloc-notes XML est en cours d’exécution, fermez-le, revenez à l’assistant **Créer un nouveau package** dans la fenêtre de **l’outil d’empaquetage MSIX**, et sélectionnez **Suivant**.

   > **Remarque** : Dans ce cas, le redémarrage n’est pas nécessaire pour terminer l’installation.

1. Dans la **page Premières tâches** de lancement de l’assistant **Création d’un package** , passez en revue les informations fournies et sélectionnez **Suivant**.
1. Quand vous avez l’invite **Avez-vous terminé ?**, sélectionnez **Oui, continuer**.
1. Dans la page **rapport Services** de l’assistant **Création d’un package**, vérifiez qu’aucun service n’est répertorié et sélectionnez **Suivant**.
1. Dans la page **Créer un package** de l’assistant **Créer un nouveau package**, dans la zone de texte **Enregistrer l’emplacement**, tapez **C :\\Tous les fichiers\\Labos\\04\\XmlNotepad\XmlNotepad.msix**, puis cliquez sur **Créer**.
1. Dans la boîte de dialogue **Package créé avec succès**, notez l’emplacement du package enregistré et sélectionnez **Fermer**.
1. Basculez vers la fenêtre explorateur de fichiers, accédez au dossier **C :\\Allfiles\\Labos\\04\\XmlNotepad** dossier et vérifiez qu’il contient les fichiers *.msix et *.xml.
1. Copiez le fichier **XmlNotepad.msix** dans le dossier **C :\\Allfiles\\Labos\\04**.


### Exercice 2 : Implémenter une image d’attachement d’application MSIX pour Azure Virtual Desktop dans l’environnement Microsoft Entra DS

Les principales tâches de cet exercice sont les suivantes

1. Activer Hyper-V sur les machines virtuelles Azure exécutant Windows 10 Enterprise Edition
1. Créer une image d’attachement d’application MSIX

#### Tâche 1 : Activer Hyper-V sur les machines virtuelles Azure exécutant Windows 10 Enterprise Edition

1. Dans la session Bastion pour **az140-cl-vm42**, à partir de l’administrateur **: Console Windows PowerShell ISE**, exécutez ce qui suit pour préparer les hôtes Azure Virtual Desktop cibles pour l’attachement d’application MSIX : 

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable
         reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
         reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f
         reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f
         reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
         reg add HKLM\Software\Microsoft\RDInfraAgent\MSIXAppAttach /v PackageListCheckIntervalMinutes /t REG_DWORD /d 1 /f
         Set-Service -Name wuauserv -StartupType Disabled
      }
   }
   ```

1. Dans la session Bastion pour **az140-cl-vm42**, à partir de l’administrateur **: Console Windows PowerShell ISE**, exécutez ce qui suit pour installer Hyper-V et ses outils de gestion, y compris le module PowerShell Hyper-V sur les hôtes Azure Virtual Desktop :

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
      }
   }
   ```

1. Lorsque vous êtes invité à redémarrer le système d’exploitation cible, sélectionnez **Oui**.
1. Dans la session Bastion pour **az140-cl-vm42**, à partir de l’administrateur **: Console Windows PowerShell ISE**, exécutez ce qui suit pour installer Hyper-V et ses outils de gestion, y compris le module PowerShell Hyper-V sur l’ordinateur local :

   ```powershell
   Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
   ```

1. Une fois l’installation des composants Hyper-V terminée, sélectionnez **Oui** pour redémarrer le système d’exploitation. Après le redémarrage, connectez-vous avec le nom d’utilisateur **wvdadmin1@adatum.com** et le mot de passe que vous avez défini lors de la création de ce compte d’utilisateur.

#### Tâche 2 : Créer une image d’attachement d’application MSIX

1. Dans la session Bastion pour **az140-cl-vm42**, démarrez **Microsoft Edge**, accédez à **https://aka.ms/msixmgr**. Cela télécharge automatiquement le fichier **msixmgr.zip** (l’archive de l’outil MSIX mgr) dans le dossier **Téléchargements**.
1. Dans l’Explorateur de fichiers, accédez au dossier **Téléchargements**, ouvrez le fichier compressé et copiez le contenu du dossier ** x64** (y compris le dossier) dans le dossier **C :\\AllFiles\\Labos\\04**. 
1. Dans la session Bastion pour **az140-cl-vm42**, démarrez **Windows PowerShell ISE** en tant qu’administrateur et, à partir de l’administrateur **: Volet de script Windows PowerShell ISE**, exécutez ce qui suit pour créer le fichier de disque dur virtuel qui servira d’image d’attachement d’application MSIX :

   ```powershell
   New-Item -ItemType Directory -Path 'C:\Allfiles\Labs\04\MSIXVhds' -Force
   New-VHD -SizeBytes 128MB -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Dynamic -Confirm:$false
   ```

1. Depuis la console **Administrateur : Volet de script Windows PowerShell ISE**, exécutez ce qui suit pour monter le fichier de disque dur virtuel nouvellement créé :

   ```powershell
   $vhdObject = Mount-VHD -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Passthru
   ```

1. Depuis la console **Administrateur : Volet de script Windows PowerShell ISE**, exécutez ce qui suit pour initialiser le disque, créer une partition, la mettre en forme et lui affecter la première lettre de lecteur disponible :

   ```powershell
   $disk = Initialize-Disk -Passthru -Number $vhdObject.Number
   $partition = New-Partition -AssignDriveLetter -UseMaximumSize -DiskNumber $disk.Number
   Format-Volume -FileSystem NTFS -Confirm:$false -DriveLetter $partition.DriveLetter -Force
   ```

   > **Remarque** : Si une fenêtre contextuelle vous invite à mettre en forme le lecteur F : , sélectionnez **Annuler**.

1. Depuis la console **Administrateur : Volet de script Windows PowerShell ISE**, exécutez ce qui suit pour créer une structure de dossiers qui hébergera les fichiers MSIX et décompressez-les dans le package MSIX que vous avez créé dans la tâche précédente :

   ```powershell
   $appName = 'XmlNotepad'
   New-Item -ItemType Directory -Path "$($partition.DriveLetter):\Apps" -Force
   Set-Location -Path 'C:\AllFiles\Labs\04\x64'
   .\msixmgr.exe -Unpack -packagePath ..\$appName.msix -destination "$($partition.DriveLetter):\Apps" -applyacls
   ```

1. Dans la session Bastion pour **az140-cl-vm42**, dans l’Explorateur de fichiers, accédez au dossier **F :\\Apps** et passez en revue son contenu. Si vous êtes invité à accéder au dossier, sélectionnez **Continuer**.
1. Dans la session Bastion pour **az140-cl-vm42**, à partir de l’administrateur **: Console Windows PowerShell ISE**, exécutez ce qui suit pour démonter le fichier de disque dur virtuel qui servira d’image MSIX :

   ```powershell
   Dismount-VHD -Path "C:\Allfiles\Labs\04\MSIXVhds\$appName.vhd" -Confirm:$false
   ```

### Exercice 3 : Implémenter l’attachement d’application MSIX sur des hôtes de session Azure Virtual Desktop

Les principales tâches de cet exercice sont les suivantes

1. Configurer des groupes Active Directory contenant des hôtes Azure Virtual Desktop
1. Configurer le partage Azure Files pour l’attachement d’application MSIX
1. Monter et inscrire l’image d’attachement d’application MSIX sur les hôtes de session Azure Virtual Desktop
1. Publier des applications MSIX dans un groupe d’applications
1. Valider les fonctionnalités de l’attachement d’application MSIX

#### Tâche 1 : Configurer des groupes Active Directory contenant des hôtes Azure Virtual Desktop

1. Basculez vers l’ordinateur labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **machines virtuelles** et, dans le panneau **Machines virtuelles**, sélectionnez **az140-dc-vm11**.
1. Dans le panneau **az140-dc-vm11**, sélectionnez **Connecter**, dans le menu déroulant, sélectionnez **Bastion**, sous l’onglet **Bastion** du panneau **az140-dc-vm11 \| Connect**, sélectionnez **Utiliser Bastion**.
1. Lorsque vous y êtes invité, fournissez les informations d’identification suivantes et sélectionnez **Connecter** :

   |Paramètre|Valeur|
   |---|---|
   |Nom d’utilisateur|**Étudiant**|
   |Mot de passe|**Pa55w.rd1234**|

1. Dans la session Bastion pour **az140-dc-vm11**, démarrez **Windows PowerShell ISE** en tant qu’administrateur.
1. Depuis la console **Administrateur : Volet de script Windows PowerShell ISE**, exécutez ce qui suit pour créer un objet de groupe AD DS qui sera synchronisé avec le locataire Microsoft Entra utilisé dans ce labo :

   ```powershell
   $ouPath = "OU=WVDInfra,DC=adatum,DC=com"
   New-ADGroup -Name 'az140-hosts-42-p1' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   ```

   > **Remarque** : Vous utiliserez ce groupe pour accorder des autorisations d’hôtes Azure Virtual Desktop aux **az140-42-msixvhds** partage de fichiers.

1. Depuis la console **Administrateur : Console Windows PowerShell ISE**, exécutez la commande suivante pour ajouter des membres aux groupes que vous avez créés à l’étape précédente :

   ```powershell
   Get-ADGroup -Identity 'az140-hosts-42-p1' | Add-AdGroupMember -Members 'az140-21-p1-0$','az140-21-p1-1$','az140-21-p1-2$'
   ```

1. Depuis la console **Administrateur : Volet de script Windows PowerShell ISE**, exécutez ce qui suit pour redémarrer les serveurs membres du groupe « az140-hosts-42-p1 » :

   ```powershell
   $hosts = (Get-ADGroup -Identity 'az140-hosts-42-p1' | Get-ADGroupMember | Select-Object Name).Name
   $hosts | ForEach-Object {Restart-Computer -ComputerName $_ -Force}
   ```

   > **Remarque** : Cette étape garantit que le changement d’appartenance au groupe prend effet. 

1. Dans la session Bastion pour **az140-dc-vm11**, dans le menu **Démarrer**, développez le dossier **Microsoft Entra Connect** et sélectionnez **Microsoft Entra Connect**.
1. Dans la page **Bienvenue dans Microsoft Entra Connect** de la fenêtre **Microsoft Entra Connect**, sélectionnez **Configurer**.
1. Dans la page **Tâches supplémentaires** dans la fenêtre **Microsoft Entra Connect**, sélectionnez **Personnaliser les options de synchronisation** et sélectionnez **Suivant**.
1. Dans la page **Se connecter à Microsoft Entra** dans la fenêtre **Microsoft Entra Connect**, authentifiez-vous à l’aide du nom d’utilisateur principal de **compte d’utilisateur** que vous avez identifié précédemment dans cette tâche avec le mot de passe que vous avez défini lors de la création de ce compte d’utilisateur.
1. Dans la page **Connecter vos répertoires** dans la fenêtre **Microsoft Entra Connect**, sélectionnez **Suivant**.
1. Dans la page **Filtrage du domaine** et de l’unité d’organisation dans la fenêtre **Microsoft Entra Connect**, vérifiez que l’option **Synchroniser les domaines sélectionnés et les unités d’organisation** est sélectionnée, développez le nœud **adatum.com**, cochez la case en regard de l’unité d’organisation **WVDInfra** (laissez les autres cases sélectionnées inchangées), et sélectionnez **Suivant**.
1. Dans la page **Fonctionnalités facultatives** dans la fenêtre **Microsoft Entra Connect**, acceptez les paramètres par défaut, puis sélectionnez **Suivant**.
1. Dans la page **Prêt à configurer** de la fenêtre **Microsoft Entra Connect**, vérifiez que la case à cocher **Démarrer le processus de synchronisation lorsque la configuration se termine** est sélectionnée et sélectionnez **Configurer**.
1. Passez en revue les informations de la page **Configuration complète**, puis sélectionnez **Quitter** pour fermer la fenêtre **Microsoft Entra Connect**.
1. Dans la session Bastion pour **az140-dc-vm11**, démarrez Microsoft Edge et accédez au [portail Azure](https://portal.azure.com). Lorsque vous y êtes invité, connectez-vous à l’aide des informations d’identification Microsoft Entra du compte d’utilisateur avec le rôle Administrateur général dans le locataire Microsoft Entra associé à l’abonnement Azure utilisé dans ce labo.
1. Dans la session Bastion pour **az140-dc-vm11**, dans la fenêtre Microsoft Edge affichant le portail Azure, recherchez et sélectionnez **Azure Active Directory** pour accéder au locataire Microsoft Entra associé à l’abonnement Azure que vous utilisez pour ce labo.
1. Dans le panneau Azure Active Directory, dans la barre de menus verticale située à gauche, dans la section **Gérer**, cliquez sur **groupes**. 
1. Sur le panneau **groupes | Tous les groupes**, dans la liste des groupes, sélectionnez l’entrée **az140-hosts-42-p1**.

   > **Remarque** : Vous devrez peut-être actualiser la page pour que le groupe s’affiche.

1. Dans le panneau **az140-hosts-42-p1**, dans la barre de menus verticale située à gauche, dans la section **Gérer**, cliquez sur **Membres**.
1. Sur le panneau**az140-hosts-42-p1 | Membres**, vérifiez que la liste des **membres directs** inclure les trois hôtes du pool Azure Virtual Desktop que vous avez ajouté au groupe précédemment dans cette tâche.

#### Tâche 2 : Configurer le partage Azure Files pour l’attachement d’application MSIX

1. Sur l’ordinateur labo, revenez à la session Bastion pour **az140-cl-vm42**.
1. Dans la session Bastion pour **az140-cl-vm42**, démarrez Microsoft Edge en mode InPrivate, accédez au [portail Azure](https://portal.azure.com)et connectez-vous en fournissant des informations d’identification d’un compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utiliserez dans ce labo.

   > **Remarque** : Veillez à utiliser le mode InPrivate Microsoft Edge.

1. Dans la session Bastion pour **az140-cl-vm42**, dans la fenêtre Microsoft Edge affichant le portail Azure, recherchez et sélectionnez **comptes de stockage** et, dans le panneau **comptes de stockage**, sélectionnez le compte de stockage que vous avez configuré pour héberger des profils utilisateur.

   > **Remarque** : Cette partie du labo est subordonnée à la fin du labo **gestion des profils Azure Virtual Desktop (AD DS)** ou **gestion des profils Azure Virtual Desktop (Microsoft Entra DS)**

   > **Remarque** : Dans les scénarios de production, vous devez envisager d’utiliser un compte de stockage distinct. Cela nécessite la configuration de ce compte de stockage pour l’authentification Microsoft Entra DS, que vous avez déjà implémenté pour les profils utilisateur d’hébergement du compte de stockage. Vous utilisez le même compte de stockage pour réduire les étapes en double dans les laboratoires individuels.

1. Dans le panneau du compte de stockage, dans le menu vertical situé à gauche, sélectionnez **contrôle d’accès (IAM)**.
1. Dans le panneau **Contrôle d’accès (IAM)** du compte de stockage, sélectionnez **+ Ajouter** et, dans le menu déroulant, sélectionnez **Ajouter une attribution de rôle**, 
1. Dans le panneau **Ajouter une attribution de rôle**, spécifiez les paramètres suivants et sélectionnez **Enregistrer**:

   |Paramètre|Valeur|
   |---|---|
   |Role|**Contributeur élevé de partage SMB de données de fichier de stockage**|
   |Attribuer l’accès à|**Utilisateur, groupe ou principal de service**|
   |Sélectionnez|**az140-wvd-admins**|

   > **Remarque** : Le groupe **az140-wvd-admins** contient le compte d’utilisateur **wvdadmin1** que vous utiliserez pour configurer les autorisations de partage. 

1. Répétez les deux étapes précédentes pour configurer les attributions de rôles suivantes :

   |Paramètre|Valeur|
   |---|---|
   |Role|**Contributeur élevé de partage SMB de données de fichier de stockage**|
   |Attribuer l’accès à|**Utilisateur, groupe ou principal de service**|
   |Sélectionnez|**az140-hosts-42-p1**|

   |Paramètre|Valeur|
   |---|---|
   |Role|**Lecteur de partage SMB de données de fichier de stockage**|
   |Attribuer l’accès à|**Utilisateur, groupe ou principal de service**|
   |Sélectionnez|**az140-wvd-users**|

   > **Remarque** : Les utilisateurs et les hôtes Azure Virtual Desktop ont besoin au moins d’un accès en lecture au partage de fichiers.

1. Dans le panneau compte de stockage, dans le menu vertical situé à gauche, dans la section **Stockage de données**, sélectionnez **Partages de fichiers**, puis **+ Partage de fichiers**.
1. Dans le panneau **Nouveau partage de fichiers**, spécifiez les paramètres suivants et sélectionnez **Créer** (laissez d’autres paramètres avec leurs valeurs par défaut) :

   |Paramètre|Valeur|
   |---|---|
   |Nom|**az140-42-msixvhds**|

1. Dans Microsoft Edge affichant le portail Azure, dans la liste des partages de fichiers, sélectionnez le partage de fichiers nouvellement créé. 

1. Dans la session Bastion pour **az140-cl-vm42**, démarrez **invite de commandes** et, à partir de la fenêtre **invite de commandes**, exécutez ce qui suit pour mapper un lecteur au **az140-42-msixvhds** partage (remplacez l’espace réservé `<storage-account-name>` par le nom du compte de stockage) et vérifiez que la commande s’exécute correctement :

   ```cmd
   net use Z: \\<storage-account-name>.file.core.windows.net\az140-42-msixvhds
   ```

1. Dans la session Bastion pour **az140-cl-vm42**, à partir de la fenêtre **l’invite de commandes**, exécutez ce qui suit pour accorder les autorisations NTFS requises aux comptes d’ordinateur des hôtes de session :

   ```cmd
   icacls Z:\ /grant ADATUM\az140-hosts-42-p1:(OI)(CI)(RX) /T
   icacls Z:\ /grant ADATUM\az140-wvd-users:(OI)(CI)(RX) /T
   icacls Z:\ /grant ADATUM\az140-wvd-admins:(OI)(CI)(F) /T
   ```

   >**Remarque** : Vous pouvez également définir ces autorisations à l’aide de l’Explorateur de fichiers lors de la connexion en tant que **wvdadmin1\@adatum.com**. 

   >**Remarque** : Ensuite, vous allez valider les fonctionnalités de l’attachement d’application MSIX

1. Dans la session Bastion pour **az140-cl-vm42**, à partir de l’administrateur **: Fenêtre Windows PowerShell ISE**, exécutez ce qui suit pour copier le fichier de disque dur virtuel que vous avez créé dans l’exercice précédent dans le partage Azure Files que vous avez créé précédemment dans cet exercice :

   ```powershell
   New-Item -ItemType Directory -Path 'Z:\packages' 
   Copy-Item -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Destination 'Z:\packages' -Force
   ```

#### Tâche 3 : Monter et inscrire l’image d’attachement d’application MSIX sur les hôtes de session Azure Virtual Desktop

1. Dans la session Bastion pour **az140-cl-vm42**, dans la fenêtre Microsoft Edge affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop** et, dans le panneau **Azure Virtual Desktop**, dans le menu vertical situé à gauche, dans la section **Gérer**, sélectionnez **pools d’hôtes**.
1. Dans le panneau **Azure Virtual Desktop \| pools d’hôtes**, dans la liste des pools hôtes, sélectionnez l’entrée **az140-21-hp1**.
1. Dans le panneau **Propriétés\| az140-21-hp1**, dans le menu vertical situé à gauche, dans la section **Gérer**, sélectionnez **packages MSIX**.
1. Dans le panneau **az140-21-hp1\| packages MSIX**, cliquez sur **+ Ajouter**.
1. Dans le panneau **Ajouter un package MSIX**, dans la zone de texte **chemin d’accès de l’image MSIX**, entrez le chemin d’accès au fichier **xmlNotepad.vhd** au format `\\<storage-account-name>.file.core.windows.net\az140-42-msixvhds\packages\XmlNotepad.vhd` (remplacez l’espace réservé `<storage-account-name>` par le nom du compte de stockage hébergeant le **az140-42-msixvhds** partage de fichiers) et cliquez sur **Ajouter**.
1. Dans le panneau **Ajouter un package MSIX**, spécifiez les paramètres suivants, puis cliquez sur **Ajouter**:

   |Paramètre|Valeur|
   |---|---|
   |Chemin d’accès de l’image MSIX|**\\\\\<storage-account-name\>.file.core.windows.net\\az140-42-msixvhds\\XmlNotepad.vhd**, où l’espace réservé `<storage-account-name>` désigne le nom du compte de stockage hébergeant le **az140-42-msixvhds** partage de fichiers|
   |Package MSIX|le nom généré lors de la création du package|
   |Nom d’affichage|**bloc-notes XML**|
   |Type d’enregistrement|**À la demande**|
   |State|**Actif**|

#### Tâche 4 : Publier des applications MSIX dans un groupe d’applications

> **Remarque** : Vous allez publier l’application MSIX sur l’application distante et le groupe d’applications de bureau. 

1. Dans la session Bastion pour **az140-cl-vm42**, dans la fenêtre Microsoft Edge affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop** et, dans le panneau **Azure Virtual Desktop**, dans le menu vertical situé à gauche, dans la section **Gérer**, sélectionnez **groupes d’applications**.
1. Dans le **panneau groupes d’applications \|Azure Virtual Desktop**, sélectionnez l’entrée de groupe d’applications **az140-21-hp1-Utilities-RAG**.
1. Dans le panneau **az140-21-hp1-Utilities-RAG**, dans le menu vertical situé à gauche, dans la section **Gérer**, sélectionnez **Applications**. 
1. Dans le panneau **az140-21-hp1-Utilities-RAG \| Applications**, cliquez sur **+ Ajouter**.
1. Dans le panneau **Ajouter une application**, dans les onglets **Informations de base** et **Icône**, spécifiez les paramètres suivants et sélectionnez **Enregistrer** :

   |Paramètre|Valeur|
   |---|---|
   |Source de l’application|**Attachement d’application**|
   |Package|nom représentant le package inclus dans l’image|
   |Application|**XMLNOTEPAD**|
   |Identificateur d’application|**bloc-notes XML**|
   |Nom d’affichage|**bloc-notes XML**|
   |Description|**bloc-notes XML**|
   |Source d’icônes|**Par défaut**|

1. Revenez au panneau **groupes d’applications \| Azure Virtual Desktop**, puis sélectionnez l’entrée de groupe d’applications **az140-21-hp1-DAG**.
1. Dans le panneau **az140-21-hp1-DAG**, dans le menu vertical situé à gauche, dans la section **Gérer**, sélectionnez **Applications**. 
1. Dans le panneau **az140-21-hp1-DAG \| Applications**, cliquez sur **+ Ajouter**.
1. Dans le panneau **Ajouter une application**, spécifiez les paramètres suivants et sélectionnez **Enregistrer**:

   |Paramètre|Valeur|
   |---|---|
   |Source de l’application|**package MSIX**|
   |Package MSIX|nom représentant le package inclus dans l’image|
   |Nom de l’application|**bloc-notes XML**|
   |Nom d’affichage|**bloc-notes XML**|
   |Description|**bloc-notes XML**|

#### Tâche 5 : Valider les fonctionnalités de l’attachement d’application MSIX

1. Dans la session Bastion pour **az140-cl-vm42**, démarrez Microsoft Edge, accédez à [page de téléchargement du client Windows Desktop](https://go.microsoft.com/fwlink/?linkid=2068602) et, une fois le téléchargement terminé, sélectionnez **Ouvrir le fichier** pour démarrer son installation. Dans la page **Étendue d’installation** de l’assistant **d’installation Bureau à distance**, sélectionnez l’option **Installer pour tous les utilisateurs de cet ordinateur**, puis cliquez sur **Installer**. 
1. Une fois l’installation terminée, vérifiez que le **Lancer le Bureau à distance lorsque le programme d’installation** quitte case est cochée, puis cliquez sur **Terminer** pour démarrer le client Bureau à distance.
1. Dans la fenêtre client **Bureau à distance**, sélectionnez **s’abonner** et, lorsque vous y êtes invité, connectez-vous avec le nom d’utilisateur **aduser1** nom d’utilisateur principal et le mot de passe que vous avez défini lors de la création de ce compte d’utilisateur. 
1. Si vous y êtes invité, dans la fenêtre **Rester connecté à toutes vos applications**, décochez la case **Autoriser mon organisation à gérer mon appareil**, puis cliquez sur **Non, connectez-vous à cette application uniquement**.
1. Dans la fenêtre client **Bureau à distance**, dans la section **az140-21-ws1**, double-cliquez sur l’icône **bloc-notes XML**, lorsque vous y êtes invité, fournissez le mot de passe et vérifiez que le Bloc-notes XML démarre correctement.


### Exercice 4 : Arrêter et libérer des machines virtuelles Azure approvisionnées et utilisées dans le labo

Les principales tâches de cet exercice sont les suivantes

1. Arrêter et libérer les machines virtuelles Azure approvisionnées et utilisées dans le labo

>**Remarque** : Dans cet exercice, vous allez libérer les machines virtuelles Azure approvisionnées et utilisées dans ce labo pour réduire les frais de calcul correspondants

#### Tâche 1 : Libérer des machines virtuelles Azure approvisionnées et utilisées dans le labo

1. Basculez vers l’ordinateur labo et, dans la fenêtre du navigateur web affichant le Portail Azure, ouvrez la session de l’interpréteur de commandes **PowerShell** dans le volet **Cloud Shell**.
1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez la commande suivante pour répertorier toutes les machines virtuelles Azure créées et utilisées dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   Get-AzVM -ResourceGroup 'az140-42-RG'
   ```

1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez ce qui suit pour arrêter et libérer toutes les machines virtuelles Azure que vous avez créées et utilisées dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   Get-AzVM -ResourceGroup 'az140-42-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Remarque** : La commande s’exécute de manière asynchrone (telle que déterminée par le paramètre -NoWait). Par conséquent, même si vous pourrez exécuter une autre commande PowerShell immédiatement après la même session PowerShell, il faudra quelques minutes avant que les machines virtuelles Azure ne soient réellement arrêtées et libérées.
