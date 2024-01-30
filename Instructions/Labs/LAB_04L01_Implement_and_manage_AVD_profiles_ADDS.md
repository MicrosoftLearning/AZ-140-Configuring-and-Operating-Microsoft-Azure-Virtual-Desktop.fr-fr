---
lab:
  title: "Labo : Implémenter et gérer les profils Azure Virtual Desktop (AD\_DS)"
  module: 'Module 4: Manage User Environments and Apps'
---

# Labo : implémenter et gérer les profils Azure Virtual Desktop (AD DS)
# Manuel de labo de l’étudiant

## Dépendances du labo

- Un abonnement Azure que vous utiliserez dans ce labo.
- Un compte Microsoft ou Microsoft Entra avec le rôle Propriétaire ou Contributeur dans l’abonnement Azure que vous utiliserez dans ce labo et avec le rôle Administrateur général dans le locataire Microsoft Entra associé à cet abonnement Azure.
- Le labo **Préparer le déploiement d’Azure Virtual Desktop (AD DS)** terminé
- Le labo **Implémenter et gérer le stockage pour WVD (AD DS)** terminé

## Durée estimée

30 minutes

## Scénario du labo

Vous devez implémenter la gestion des profils Azure Virtual Desktop dans un environnement Active Directory Domain Services (AD DS).

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

- Implémenter des profils basés sur FSLogix pour Azure Virtual Desktop

## Fichiers du labo

- Aucun

## Instructions

### Exercice 1 : Implémenter des profils basés sur FSLogix pour Azure Virtual Desktop

Les principales tâches de cet exercice sont les suivantes

1. Configurer des profils basés sur FSLogix sur des machines virtuelles de l’hôte de la session Azure Virtual Desktop
1. Tester des profils basés sur FSLogix avec Azure Virtual Desktop
1. Supprimer les ressources Azure déployées dans le labo

#### Tâche 1 : Configurer des profils basés sur FSLogix sur des machines virtuelles de l’hôte de la session Azure Virtual Desktop

1. À partir de votre ordinateur labo, démarrez un navigateur web, accédez au [portail Azure](https://portal.azure.com), puis connectez-vous en fournissant des informations d’identification d’un compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utiliserez dans ce labo.
1. Dans le portail Azure, recherchez et sélectionnez **Machines virtuelles** et, dans le panneau **Machines virtuelles**, sélectionnez **az140-21-p1-0**.
1. Dans le panneau **az140-21-p1-0**, sélectionnez **Démarrer** et attendez que l’état de la machine virtuelle devienne **Exécution**.
1. Dans le panneau **az140-21-p1-0**, sélectionnez **Connecter**, dans le menu déroulant, sélectionnez **Bastion**, sous l’onglet **Bastion** du panneau **az140-21-p1-0 \| Connecter**, sélectionnez **Utiliser Bastion**.
1. À l’invite, connectez-vous avec les informations d’identification suivantes :

   |Paramètre|Valeur|
   |---|---|
   |Nom d’utilisateur|**student@adatum.com**|
   |Mot de passe|**Pa55w.rd1234**|

1. Dans la session Bastion vers **az140-21-p1-0**, démarrez Microsoft Edge et accédez au [portail Azure](https://portal.azure.com). Si vous y êtes invité, connectez-vous à l’aide des informations d’identification Microsoft Entra du compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utilisez dans ce labo.
1. Dans la session Bastion vers **az140-21-p1-0**, dans la fenêtre Microsoft Edge affichant le portail Azure, ouvrez une session PowerShell dans le volet Cloud Shell. 
1. À partir de la session PowerShell dans le volet **Cloud Shell**, exécutez la commande suivante pour démarrer les machines virtuelles d’hôte de session Azure Virtual Desktop que vous utiliserez dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**Remarque** : Attendez que les machines virtuelles Azure s’exécutent avant de passer à l’étape suivante.

1. À partir de la session PowerShell dans le volet **Cloud Shell**, exécutez la commande suivante pour activer la communication à distance PowerShell sur les hôtes de session.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Enable-AzVMPSRemoting
   ```
   
1. Fermer Cloud Shell
1. Dans la session Bastion vers **az140-21-p1-0**, démarrez Microsoft Edge, accédez à la [page de téléchargement de FSLogix](https://aka.ms/fslogix_download), téléchargez les fichiers binaires d’installation compressés de FSLogix, extrayez-les dans le dossier **C:\\Allfiles\\Labs\\04** (créez-le si nécessaire), accédez au sous-dossier **x64\\Release**, double-cliquez sur le fichier **FSLogixAppsSetup.exe** pour lancer l’Assistant **Installation des applications Microsoft FSLogix** et suivez les étapes d’installation avec les paramètres par défaut.

   > **Remarque** : L’installation de FXLogic n’est pas nécessaire si l’image l’inclut déjà.

1. Dans la session Bastion vers **az140-21-p1-0**, démarrez **Windows PowerShell ISE** en tant qu’administrateur et, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour installer la dernière version du module PowerShellGet (sélectionnez **Oui** lorsque vous êtes invité à confirmer) :

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
1. Dans la session Bastion vers **az140-21-p1-0**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour récupérer le nom du compte stockage Azure que vous avez configuré précédemment dans ce labo :

   ```powershell
   $resourceGroupName = 'az140-22-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName   
   ```

1. Dans la session Bastion vers **az140-21-p1-0**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour configurer les paramètres du registre de profils :

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22-profiles'
   New-Item -Path $profilesParentKey -Name $profilesChildKey –Force
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$fileShareName"
   ```

1. Dans la session Bastion vers **az140-21-p1-0**, cliquez avec le bouton droit sur **Démarrer**, dans le menu contextuel, sélectionnez **Exécuter**, dans la boîte de dialogue **Exécuter**, dans la zone de texte **Ouvrir**, tapez ce qui suit et sélectionnez **OK** pour lancer la console **Utilisateurs et groupes locaux** :

   ```cmd
   lusrmgr.msc
   ```

1. Dans la console **Utilisateurs et groupes locaux**, notez les quatre groupes dont le nom commence par la chaîne **FSLogix** :

   - Liste d’exclusion ODFC FSLOGIX
   - Liste d’inclusion ODFC FSLogix
   - Liste d’exclusion du profil FSLogix
   - Liste d’inclusion du profil FSLogix

1. Dans la console **Utilisateurs et groupes locaux**, dans la liste des groupes, double-cliquez sur le groupe **Liste d’inclusion de profil FSLogix**, notez qu’il inclut le groupe **\\Tout le monde**, puis sélectionnez **OK** pour fermer la fenêtre **Propriétés** du groupe. 
1. Dans la console **Utilisateurs et groupes locaux**, dans la liste des groupes, double-cliquez sur le groupe **Liste d’exclusion de profil FSLogix**, notez qu’il n’inclut aucun membre de groupe par défaut, puis sélectionnez **OK** pour fermer la fenêtre **Propriétés** du groupe. 

   > **Remarque** : Pour offrir une expérience utilisateur cohérente, vous devez installer et configurer des composants FSLogix sur tous les hôtes de session Azure Virtual Desktop. Vous effectuerez cette tâche sans assistance sur les autres hôtes de session dans notre environnement lab. 

1. Dans la session Bastion vers **az140-21-p1-0**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour installer les composants FSLogix sur les hôtes de session **az140-21-p1-1** et **az140-21-p1-2** :

   ```powershell
   $servers = 'az140-21-p1-1', 'az140-21-p1-2'
   foreach ($server in $servers) {
      $localPath = 'C:\Allfiles\Labs\04\x64'
      $remotePath = "\\$server\C$\Allfiles\Labs\04\x64\Release"
      Copy-Item -Path $localPath\Release -Destination $remotePath -Filter '*.exe' -Force -Recurse
      Invoke-Command -ComputerName $server -ScriptBlock {
         Start-Process -FilePath $using:localPath\Release\FSLogixAppsSetup.exe -ArgumentList '/quiet' -Wait
      } 
   }
   ```

   > **Remarque** : Attendez que l’exécution du script se termine. Cela peut prendre environ 2 minutes.

1. Dans la session Bastion vers **az140-21-p1-0**, à partir du volet de script **Administrateur : Volet de script Windows PowerShell ISE**, exécutez la commande suivante pour configurer les paramètres de registre de profils sur les hôtes de session **az140-21-p1-1** et **az140-21-p1-1** :

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22-profiles'
   foreach ($server in $servers) {
      Invoke-Command -ComputerName $server -ScriptBlock {
         New-Item -Path $using:profilesParentKey -Name $using:profilesChildKey –Force
         New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
         New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$using:storageAccountName.file.core.windows.net\$using:fileShareName"
      }
   }
   ```

   > **Remarque** : Avant de tester la fonctionnalité de profil basé sur FSLogix, vous devez supprimer le profil mis en cache localement du compte **ADATUM\\aduser1** que vous utiliserez pour tester les hôtes de session Azure Virtual Desktop que vous avez utilisés dans le labo précédent.

1. Dans la session Bastion vers **az140-21-p1-0**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour supprimer le profil mis en cache localement du compte **ADATUM\\aduser1** sur toutes les machines virtuelles Azure servant d’hôtes de session :

   ```powershell
   $userName = 'aduser1'
   $servers = 'az140-21-p1-0','az140-21-p1-1', 'az140-21-p1-2'
   Get-CimInstance -ComputerName $servers -Class Win32_UserProfile | Where-Object { $_.LocalPath.split('\')[-1] -eq $userName } | Remove-CimInstance
   ```

#### Tâche 2 : Tester des profils basés sur FSLogix avec Azure Virtual Desktop

1. Basculez vers votre ordinateur labo, à partir de l’ordinateur labo, dans la fenêtre du navigateur affichant le portail Azure, recherchez et sélectionnez **Machines virtuelles** et, dans le panneau **Machines virtuelles**, sélectionnez l’entrée **az140-cl-vm11**.
1. Dans le panneau **az140-cl-vm11**, sélectionnez **Connecter**, dans le menu déroulant, sélectionnez **Bastion**, sous l’onglet **Bastion** du panneau **az140-cl-vm11\| Connecter**, sélectionnez **Utiliser bastion**.
1. Lorsque vous y êtes invité, fournissez les informations d’identification suivantes et sélectionnez **Connecter** :

   |Paramètre|Valeur|
   |---|---|
   |Nom d’utilisateur|**Student@adatum.com**|
   |Mot de passe|**Pa55w.rd1234**|

1. Dans la session Bastion vers **az140-cl-vm11**, cliquez sur **Démarrer** et, dans le menu **Démarrer**, cliquez sur **Bureau à distance** pour démarrer le client Bureau à distance.
1. Dans la session Bastion vers **az140-cl-vm11**, dans la fenêtre du client **Bureau à distance**, sélectionnez **S’abonner** et, quand vous y êtes invité, connectez-vous avec les informations d’identification **aduser1**.

   >**Remarque** : si vous n’êtes pas invité à vous abonner, vous devrez peut-être vous résilier un abonnement précédent.
3. dans la liste des applications, double-cliquez sur **Invite de commandes**, lorsque vous y êtes invité, indiquez le mot de passe du compte **aduser1** et vérifiez qu’une fenêtre **Invite de commandes** s’ouvre correctement.
4. Dans le coin supérieur gauche de la fenêtre **Invite de commandes**, cliquez avec le bouton droit sur l’icône **Invite de commandes** et, dans le menu déroulant, sélectionnez **Propriétés**.
5. Dans la boîte de dialogue **Propriétés de l’invite de commandes**, sélectionnez l’onglet **Police**, modifiez la taille et les paramètres de police, puis sélectionnez **OK**.
6. Dans la fenêtre **Invite de commandes**, tapez **logoff** et appuyez sur la touche **Entrée** pour vous déconnecter de la session Bureau à distance.
7. Dans la session Bastion vers **az140-cl-vm11**, dans la fenêtre cliente **Bureau à distance**, dans la liste des applications, double-cliquez sur **SessionDesktop** sous az140-21-ws1 et vérifiez qu’il lance une session Bureau à distance. 
8. Dans la session **SessionDesktop**, cliquez avec le bouton droit sur **Démarrer**, dans le menu contextuel, sélectionnez **Exécuter**, dans la boîte de dialogue **Exécuter**, dans la zone de texte **Ouvrir**, tapez **cmd** et sélectionnez **OK** pour lancer une fenêtre **Invite de commandes** :
9. Vérifiez que les paramètres de la fenêtre **Invite de commandes** correspondent à ceux que vous avez configurés précédemment dans cette tâche.
10. Dans la session **SessionDesktop**, réduisez toutes les fenêtres, cliquez avec le bouton droit sur le bureau, dans le menu contextuel, sélectionnez **Nouveau** et, dans le menu en cascade, sélectionnez **Raccourci**. 
11. Sur la page **De quel élément souhaitez-vous créer un raccourci ?** de l’Assistant **Créer un raccourci**, dans la zone de texte **Tapez l’emplacement de la zone de texte de l’élément**, tapez **Bloc-notes** et sélectionnez **Suivant**.
12. Sur la page **Quel nom souhaitez-vous donner au raccourci** de l’assistant **Créer un raccourci**, dans la zone de texte **Tapez un nom pour ce raccourci**, tapez **Bloc-notes** et sélectionnez **Terminer**.
13. Dans la session **SessionDesktop**, cliquez avec le bouton droit sur **Démarrer**, dans le menu contextuel, sélectionnez **Arrêter ou se déconnecter**, puis, dans le menu en cascade, sélectionnez **Se déconnecter**.
14. Retournez à la session Bastion vers **az140-cl-vm11**, dans la fenêtre de client **Bureau à distance**, dans la liste des applications, puis double-cliquez sur **SessionDesktop** pour démarrer une nouvelle session Bureau à distance. 
15. Dans la session **SessionDesktop**, vérifiez que le raccourci **Bloc-notes** s’affiche sur le bureau.
16. Dans la session **SessionDesktop**, cliquez avec le bouton droit sur **Démarrer**, dans le menu contextuel, sélectionnez **Arrêter ou se déconnecter**, puis, dans le menu en cascade, sélectionnez **Se déconnecter**.
17. Basculez vers votre ordinateur labo et, dans la fenêtre Microsoft Edge affichant le portail Azure, accédez au panneau **Comptes de stockage** et sélectionnez l’entrée représentant le compte de stockage que vous avez créé dans l’exercice précédent.
18. Dans le panneau Compte de stockage, dans la section **Services de fichiers**, sélectionnez **Partages de fichiers**, puis, dans la liste des partages de fichiers, sélectionnez **az140-22-profiles**. 
19. Dans le panneau **az140-22-profiles**, sélectionnez **Parcourir** et vérifiez que son contenu inclut un dossier qui se compose d’une combinaison de l’identificateur de sécurité (SID) du compte **ADATUM\\aduser1** suivi du suffixe **_aduser1**.
20. Sélectionnez le dossier que vous avez identifié à l’étape précédente et notez qu’il contient un fichier unique nommé **Profile_aduser1.vhd**.

### Exercice 2 : Arrêter et libérer les machines virtuelles Azure approvisionnées et utilisées dans le labo

Les principales tâches de cet exercice sont les suivantes

1. Arrêter et libérer les machines virtuelles Azure approvisionnées et utilisées dans le labo

>**Remarque** : Dans cet exercice, vous allez libérer les machines virtuelles Azure approvisionnées et utilisées dans ce labo pour réduire les frais de calcul correspondants

#### Tâche 1 : Libérer les machines virtuelles Azure approvisionnées et utilisées dans le labo

1. Basculez vers l’ordinateur labo et, dans la fenêtre du navigateur web affichant le portail Azure, ouvrez la session shell **PowerShell** dans le volet **Cloud Shell**.
1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez la commande suivante pour répertorier toutes les machines virtuelles Azure créées et utilisées dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez ce qui suit pour arrêter et libérer toutes les machines virtuelles Azure que vous avez créées et utilisées dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Remarque** : La commande s’exécute de manière asynchrone (telle que déterminée par le paramètre -NoWait). Par conséquent, même si vous pourrez exécuter une autre commande PowerShell immédiatement après la même session PowerShell, il faudra quelques minutes avant que les machines virtuelles Azure ne soient réellement arrêtées et libérées.
