---
lab:
  title: "Labo : Implémenter et gérer les profils Azure Virtual Desktop (Microsoft Entra\_DS)"
  module: 'Module 4: Manage User Environments and Apps'
---

# Labo : Implémenter et gérer les profils Azure Virtual Desktop (Microsoft Entra DS)
# Manuel de labo de l’étudiant

## Dépendances du labo

- Un abonnement Azure
- Un compte Microsoft ou un compte Microsoft Entra disposant du rôle Administrateur général pour le locataire Microsoft Entra qui est associé à l’abonnement Azure, et disposant du rôle Propriétaire ou Contributeur dans l’abonnement Azure
- Un environnement Azure Virtual Desktop provisionné dans le labo **Introduction à Azure Virtual Desktop (Microsoft Entra DS)**

## Durée estimée

30 minutes

## Scénario du labo

Vous devez implémenter la gestion des profils Azure Virtual Desktop dans un environnement Microsoft Entra DS.

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

- Configurer Azure Files en vue du stockage des conteneurs de profils pour Azure Virtual Desktop dans un environnement Microsoft Entra DS
- Implémenter des profils basés sur FSLogix pour Azure Virtual Desktop dans un environnement Microsoft Entra DS

## Fichiers du labo

- Aucun

## Instructions

### Exercice 1 : Implémenter des profils basés sur FSLogix pour Azure Virtual Desktop

Les principales tâches de cet exercice sont les suivantes

1. Configurer le groupe Administrateurs local sur les machines virtuelles de l’hôte de la session Azure Virtual Desktop
1. Configurer des profils basés sur FSLogix sur des machines virtuelles de l’hôte de la session Azure Virtual Desktop
1. Tester des profils basés sur FSLogix avec Azure Virtual Desktop
1. Supprimer les ressources du labo Azure

#### Tâche 1 : Configurer le groupe Administrateurs local sur les machines virtuelles de l’hôte de la session Azure Virtual Desktop

1. À partir de votre ordinateur de labo, démarrez un navigateur web, accédez au [Portail Azure](https://portal.azure.com), puis connectez-vous en fournissant des informations d’identification d’un compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utiliserez dans ce labo.
1. Dans le Portail Azure, ouvrez le volet **Cloud Shell** en sélectionnant l’icône de la barre d’outils juste à droite de la zone de texte de recherche.
1. Lorsque vous êtes invité à sélectionner **Bash** ou **PowerShell**, sélectionnez **PowerShell**. 

   >**Remarque** : si c’est la première fois que vous démarrez **Cloud Shell** et que vous voyez le message **Vous n’avez aucun stockage monté**, sélectionnez l’abonnement que vous utilisez dans ce labo, puis sélectionnez **Créer un stockage**. 

1. À partir de la session PowerShell dans le volet **Cloud Shell**, exécutez la commande suivante pour démarrer les machines virtuelles d’hôte de session Azure Virtual Desktop que vous utiliserez dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21a-RG' | Start-AzVM
   ```

   >**Remarque** : Attendez que les machines virtuelles Azure s’exécutent avant de passer à l’étape suivante.
   
      
1. À partir de la session PowerShell dans le volet **Cloud Shell**, exécutez la commande suivante pour activer la communication à distance PowerShell sur les hôtes de session.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21a-RG' | Enable-AzVMPSRemoting
   ```
   
1. Fermer Cloud Shell
1. À partir de votre ordinateur de labo, dans le Portail Azure, recherchez et sélectionnez **Machines virtuelles**, puis, dans le panneau **Machines virtuelles**, sélectionnez l’entrée **az140-cl-vm11a**. Cette opération ouvre le panneau **az140-cl-vm11a**.
1. Dans le panneau **az140-cl-vm11a**, sélectionnez **Se connecter**, puis dans le menu déroulant, sélectionnez **Bastion**. Sous l’onglet **Bastion** du panneau **az140-cl-vm11a\| Se connecter**, sélectionnez **Utiliser Bastion**.
1. Lorsque vous y êtes invité, fournissez les informations d’identification suivantes et sélectionnez **Connecter** :

   |Paramètre|Valeur|
   |---|---|
   |Nom d’utilisateur|**aadadmin1@adatum.com**|
   |Mot de passe|Mot de passe précédemment configuré|

1. Dans le menu Démarrer de la session Bastion vers **az140-cl-vm11a**, accédez au dossier **Outils d’Administration Windows**, développez-le et sélectionnez **Utilisateurs et ordinateurs Active Directory**.
1. Dans la console**Utilisateurs et ordinateurs Active Directory**, cliquez avec le bouton droit sur le nœud du domaine, sélectionnez **Nouveau**, ensuite **Unité d’organisation**. Dans la boîte de dialogue **Nouvel Objet – Unité d’organisation**, dans la zone de texte **Nom**, tapez **Utilisateurs ADDC** et sélectionnez **OK**.
1. Dans la console **Utilisateurs et ordinateurs Active Directory**, cliquez avec le bouton droit sur **Utilisateurs ADDC**, sélectionnez **Nouveau**, ensuite **Groupe**. Dans la boîte de dialogue **Nouvel objet – Groupe**, spécifiez les paramètres suivants et sélectionnez **OK** :

   |Paramètre|Value|
   |---|---|
   |Nom du groupe|**Administrateurs locaux**|
   |Nom du groupe (pré-Windows 2000)|**Administrateurs locaux**|
   |Group scope|**Global**|
   |Type de groupe|**Sécurité**|

1. Dans la console **Utilisateurs et ordinateurs Active Directory**, affichez les propriétés du groupe **Administrateurs locaux**. Basculez vers l’onglet **Membres**, sélectionnez **Ajouter** dans la boîte de dialogue **Sélectionner des utilisateurs, contacts, ordinateurs, comptes de service ou groupes**. Dans la boîte de dialogue **Entrer les noms d’objets à sélectionner**, tapez **aadadmin1;wvdaadmin1** et sélectionnez **OK**.
1. Dans le menu Démarrer de la session Bastion vers **az140-cl-vm11a**, accédez au dossier **Outils d’Administration Windows**, développez-le et sélectionnez **Gestion des stratégies de groupe**.
1. Dans la console **Gestion des stratégies de groupe**, accédez à l’unité d’organisation **Ordinateurs AADDC**, cliquez avec le bouton droit sur l’icône **GPO des ordinateurs AADDC** et sélectionnez **Modifier**.
1. Dans la console **Éditeur de gestion des stratégies de groupe**, développez **Configuration de l’ordinateur**, **Stratégies**, **Paramètres Windows**, **Paramètres de sécurité**, cliquez avec le bouton droit sur **Groupes restreints** et sélectionnez **Ajouter un groupe**.
1. Dans la boîte de dialogue **Ajouter un groupe**, dans la zone de texte **Groupe**, sélectionnez **Parcourir**. Sans la boîte de dialogue **Sélectionner des groupes**, dans **Entrer les noms d’objets à sélectionner**, tapez **Administrateurs locaux** et sélectionnez **OK**.
1. Dans la boîte de dialogue **Ajouter un groupe**, sélectionnez **OK**.
1. Dans la boîte de dialogue **Propriétés d’ADATUM\Administrateurs locaux**, dans la section intitulée **Ce groupe est membre de**, sélectionnez **Ajouter**. Dans la boîte de dialogue **Appartenance au groupe**, tapez **Administrateurs**, sélectionnez **OK** et à nouveau **OK** pour finaliser la modification.

   >**Remarque** : Assurez-vous d’utiliser la section intitulée **Ce groupe est membre de**

1. Dans la session Bastion vers la machine virtuelle Azure az140-cl-vm11a, démarrez PowerShell ISE en tant qu’Administration et exécutez la commande suivante pour redémarrer les deux hôtes Azure Virtual Desktop afin de déclencher le traitement de la stratégie de groupe :

   ```powershell
   $servers = 'az140-21-p1-0','az140-21-p1-1'
   Restart-Computer -ComputerName $servers -Force -Wait
   ```

1. Attendez la fin du script. Ce processus prend environ 3 minutes.

#### Tâche 2 : Configurer des profils basés sur FSLogix sur des machines virtuelles de l’hôte de la session Azure Virtual Desktop

1. Dans la session Bastion vers **az140-cl-vm11a**, démarrez une session Bureau à distance sur **az140-21-p1-0** et, lorsque vous y êtes invité, connectez-vous avec le nom d’utilisateur **ADATUM\wvdaadmin1** et le mot de passe défini lors de la création de ce compte d’utilisateur. 

   > **Remarque** : Si la connexion RDP ne peut pas se connecter, utilisez le Portail Azure pour vous connecter à la machine virtuelle à l’aide de Bastion.

1. Dans la session Bureau à distance vers **az140-21-p1-0**, démarrez Microsoft Edge et accédez à la [page de téléchargement FSLogix](https://aka.ms/fslogix_download). Téléchargez les fichiers binaires d’installation compressés FSLogix, extrayez-les dans le dossier **C:\\Source**. Accédez au sous-dossier **x64\\Version** et utilisez **FSLogixAppsSetup.exe** pour installer Microsoft FSLogix Apps avec les paramètres par défaut.

   > **Remarque** : L’installation de FXLogic peut ne pas être nécessaire, selon que l’image l’inclut déjà. Une installation de FX Logix requiert un redémarrage.

1. Dans la session Bureau à distance vers **az140-21-p1-0**, démarrez **Windows PowerShell ISE** en tant qu’Administrateur et, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour installer la dernière version du module PowerShellGet (sélectionnez **Oui** lorsque vous êtes invité à confirmer) :

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. Depuis la console **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour installer la dernière version du module Az PowerShell (sélectionnez **Oui pour tout** lorsque vous êtes invité à confirmer) :

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

1. Depuis la console **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour modifier la stratégie d’exécution :

   ```powershell
   Set-ExecutionPolicy RemoteSigned -Force
   ```

1. Depuis la console **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour vous connecter à votre abonnement Azure :

   ```powershell
   Connect-AzAccount
   ```

1. Lorsque vous y êtes invité, connectez-vous avec les informations d’identification Microsoft Entra du compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utilisez dans ce labo.
1. Depuis la console **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour récupérer le nom du compte de stockage Azure que vous avez configuré précédemment dans ce labo :

   ```powershell
   $resourceGroupName = 'az140-22a-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName   
   ```

1. Dans la session Bureau à distance pour **az140-21-p1-0**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour configurer les paramètres du registre de profils :

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22a-profiles'
   New-Item -Path $profilesParentKey -Name $profilesChildKey -Force
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$fileShareName"
   ```
   >**Remarque** Si la commande génère une erreur, passez à l’étape suivante.
   
1. Dans la session Bureau à distance pour **az140-21-p1-0**, cliquez avec le bouton droit sur **Démarrer** et sélectionnez **Exécuter** dans le menu contextuel. Dans la zone de texte **Ouvrir** de la boîte de dialogue **Exécuter**, tapez ce qui suit et sélectionnez **OK** pour lancer la fenêtre **Utilisateurs et groupes locaux** :

   ```cmd
   lusrmgr.msc
   ```

1. Dans la console **Utilisateurs et groupes locaux**, notez les quatre groupes dont le nom commence par la chaîne **FSLogix** :

   - Liste d’exclusion ODFC FSLOGIX
   - Liste d’inclusion ODFC FSLogix
   - Liste d’exclusion du profil FSLogix
   - Liste d’inclusion du profil FSLogix

1. Dans la console **Utilisateurs et groupes locaux**, double-cliquez sur l’entité du groupe **Liste d’inclusion du profil FSLogix**, remarquez qu’il contient le groupe **\\Tout le monde**, puis sélectionnez **OK** pour fermer la fenêtre **Propriétés** du groupe. 
1. Dans la console **Utilisateurs et groupes locaux**, double-cliquez sur l’entité du groupe **Liste d’exclusion du profil FSLogix**, remarquez qu’il ne contient aucun membre de groupe par défaut, puis sélectionnez **OK** pour fermer la fenêtre **Propriétés** du groupe. 

   > **Remarque** : Pour offrir une expérience utilisateur cohérente, vous devez installer et configurer des composants FSLogix sur tous les hôtes de session Azure Virtual Desktop. Vous effectuerez cette tâche sans assistance sur l’autre hôte de session dans notre environnement de labo. 

1. Dans la session Bureau à distance pour **az140-21-p1-0**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour installer les composants FSLogix sur l’hôte de la session **az140-21-p1-1** :

   ```powershell
   $server = 'az140-21-p1-1' 
   $localPath = 'C:\Source\x64'
   $remotePath = "\\$server\C$\Source\x64\Release"
   Copy-Item -Path $localPath\Release -Destination $remotePath -Filter '*.exe' -Force -Recurse
   Invoke-Command -ComputerName $server -ScriptBlock {
      Start-Process -FilePath $using:localPath\Release\FSLogixAppsSetup.exe -ArgumentList '/quiet' -Wait
   } 
   ```

1. Dans la session Bureau à distance vers **az140-21-p1-0**, démarrez **Windows PowerShell ISE** en tant qu’Administrateur et, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour configurer les paramètres du Registre de profils sur l’hôte de la session **az140-21-p1-1** :

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22a-profiles'
   Invoke-Command -ComputerName $server -ScriptBlock {
      New-Item -Path $using:profilesParentKey -Name $using:profilesChildKey -Force
      New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
      New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$using:fileShareName"
   }
   ```

   > **Remarque** : Avant de tester la fonctionnalité de profil basé sur FSLogix, vous devez supprimer le profil mis en cache localement du compte ADATUM\wvdaadmin1 qui va servir au test des hôtes de session Azure Virtual Desktop utilisés dans le labo précédent.

1. Basculez vers la session Bastion vers **az140-cl-vm11a**. Dans la session Bastion vers**az140-cl-vm11a **, basculez vers la fenêtre **Administrateur : Windows PowerShell ISE** et, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour supprimer le profil mis en cache localement du compte ADATUM\aaduser1 :

   ```powershell
   $userName = 'aaduser1'
   $servers = 'az140-21-p1-0','az140-21-p1-1'
   Get-CimInstance -ComputerName $servers -Class Win32_UserProfile | Where-Object { $_.LocalPath.split('\')[-1] -eq $userName } | Remove-CimInstance
   ```

#### Tâche 3 : Tester des profils basés sur FSLogix avec Azure Virtual Desktop

1. Dans la session Bastion vers **az140-cl-vm11a**, basculez vers le client Bureau à distance.
1. Dans la session Bastion pour **az140-cl-vm11a**, dans la fenêtre du client **Bureau à distance**, dans la liste des applications, double-cliquez sur **Invite de commandes**, fournissez le mot de passe (lorsque vous y êtes invité) et vérifiez qu’il ouvre une fenêtre d’**invite de commandes**. 

   > **Remarque** : Au départ, le démarrage de l’application peut prendre quelques minutes, mais par la suite, il doit être beaucoup plus rapide.

1. Dans le coin supérieur gauche de la fenêtre **Invite de commandes**, cliquez avec le bouton droit sur l’icône **Invite de commandes** et, dans le menu déroulant, sélectionnez **Propriétés**.
1. Dans la boîte de dialogue **Propriétés de l’invite de commandes**, sélectionnez l’onglet **Police**, modifiez la taille et les paramètres de police, puis sélectionnez **OK**.
1. Dans la fenêtre **Invite de commandes**, tapez **logoff** et appuyez sur la touche **Entrer** pour vous déconnecter de la session Bureau à distance.
1. Dans la session Bastion sur **az140-cl-vm11a**, dans la fenêtre du client **Bureau à distance**, dans la liste des applications, double-cliquez sur **SessionDesktop** et assurez-vous qu’il ouvre une session Bureau à distance. 
1. Dans la session **SessionDesktop**, cliquez avec le bouton droit sur **Démarrer**, dans le menu contextuel, sélectionnez **Exécuter**, dans la boîte de dialogue **Exécuter**, dans la zone de texte **Ouvrir**, tapez **cmd** et sélectionnez **OK** pour lancer une fenêtre **Invite de commandes** :
1. Assurez-vous que les propriétés de la fenêtre **Invite de commandes** correspondent à celles définies précédemment dans cette tâche.
1. Dans la session **SessionDesktop**, réduisez toutes les fenêtres, cliquez avec le bouton droit sur le bureau, dans le menu contextuel, sélectionnez **Nouveau** et, dans le menu en cascade, sélectionnez **Raccourci**. 
1. Sur la page **De quel élément souhaitez-vous créer un raccourci ?** de l’Assistant **Créer un raccourci**, dans la zone de texte **Tapez l’emplacement de la zone de texte de l’élément**, tapez **Bloc-notes** et sélectionnez **Suivant**.
1. Sur la page **Quel nom souhaitez-vous donner au raccourci** de l’assistant **Créer un raccourci**, dans la zone de texte **Tapez un nom pour ce raccourci**, tapez **Bloc-notes** et sélectionnez **Terminer**.
1. Dans la session **SessionDesktop**, cliquez avec le bouton droit sur **Démarrer**, dans le menu contextuel, sélectionnez **Arrêter ou se déconnecter**, puis, dans le menu en cascade, sélectionnez **Se déconnecter**.
1. De retour dans la session Bastion vers **az140-cl-vm11a**, dans la fenêtre du client **Bureau à distance**, dans la liste des applications, puis double-cliquez sur **SessionDesktop** pour démarrer une nouvelle session Bureau à distance. 
1. Dans la session **SessionDesktop**, vérifiez que le raccourci **Bloc-notes** s’affiche sur le bureau.
1. Dans la session **SessionDesktop**, cliquez avec le bouton droit sur **Démarrer**, dans le menu contextuel, sélectionnez **Arrêter ou se déconnecter**, puis, dans le menu en cascade, sélectionnez **Se déconnecter**.
1. Basculez de la session Bastion vers **az140-cl-vm11a**, basculez vers la fenêtre Microsoft Edge affichant la Portail Azure.
1. Dans la fenêtre Microsoft Edge affichant le Portail Azure, revenez au panneau **Comptes de stockage** et sélectionnez l’entrée représentant le compte de stockage créé dans l’exercice précédent.
1. Dans le panneau Compte de stockage, dans la section **Services de fichiers**, sélectionnez **Partages de fichiers**, puis **az140-22a-profiles** dans la liste des partages de fichiers. 
1. Dans le panneau **az140-22a-profiles**, sélectionnez **Parcourir** et assurez-vous qu’il contient un dossier dont le nom est une combinaison de l’identificateur de sécurité (SID) du compte **ADATUM\\aaduser1**, suivi du suffixe **_aaduser1**.
1. Sélectionnez le dossier identifié à l’étape précédente et remarquez qu’il contient un fichier unique nommé **Profile_aaduser1.vhd**.

### Exercice 2 : Supprimer les ressources du labo Azure (facultatif)

1. Supprimez un déploiement Microsoft Entra DS en suivant les instructions décrites dans [Désactiver ou supprimer un domaine managé Azure Active Directory Domain Services à partir du portail Azure]( https://docs.microsoft.com/en-us/azure/active-directory-domain-services/delete-aadds).
1. Supprimez tous les groupes de ressources Azure provisionnés dans les labos Microsoft Entra DS de ce cours en suivant les instructions décrites dans [Suppression d’un groupe de ressources et de ressources Azure Resource Manager](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/delete-resource-group?tabs=azure-portal).
