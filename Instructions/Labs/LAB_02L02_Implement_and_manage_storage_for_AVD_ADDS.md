---
lab:
  title: "Labo : Implémenter et gérer le stockage pour AVD (AD\_DS)"
  module: 'Module 2: Implement an AVD Infrastructure'
---

# Laboratoire : Implémenter et gérer le stockage pour AVD (AD DS)
# Manuel de labo de l’étudiant

## Dépendances du labo

- Un abonnement Azure que vous utiliserez dans ce labo.
- Un compte Microsoft ou un compte Microsoft Entra avec le rôle Propriétaire ou Contributeur dans l’abonnement Azure que vous utiliserez dans ce labo et avec le rôle Administrateur général dans le locataire Microsoft Entra associé à cet abonnement Azure.
- Le labo terminé **Préparer le déploiement d’Azure Virtual Desktop (AD DS)**

## Durée estimée

30 minutes

## Scénario du labo

Vous devez implémenter et gérer le stockage pour un déploiement Azure Virtual Desktop dans un environnement Microsoft Entra DS.

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

- Configurer Azure Files en vue du stockage des conteneurs de profils pour Azure Virtual Desktop

## Fichiers du labo

- Aucun

## Instructions

### Exercice 1 : Configurer Azure Files en vue du stockage des conteneurs de profils pour Azure Virtual Desktop

Les principales tâches de cet exercice sont les suivantes

1. Créer un compte de stockage Azure
1. Créer un partage Azure Files
1. Activer l’authentification AD DS pour le compte de stockage Azure 
1. Configurer les autorisations basées sur le RBAC pour Azure Files
1. Configurer les autorisations du système de fichiers Azure Files

#### Tâche 1 : Créer un compte de stockage Azure

1. À partir de votre ordinateur labo, démarrez un navigateur web, accédez au [portail Azure](https://portal.azure.com), puis connectez-vous en fournissant des informations d’identification d’un compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utiliserez dans ce labo.
1. Dans le portail Azure, recherchez et sélectionnez **machines virtuelles** et, dans le panneau **Machines virtuelles**, sélectionnez **az140-dc-vm11**.
1. Dans le panneau **az140-dc-vm11**, sélectionnez **Connecter**, dans le menu déroulant, sélectionnez **Bastion**, sous l’onglet **Bastion** du panneau **az140-dc-vm11 \| Connect**, sélectionnez **Utiliser Bastion**.
1. Lorsque vous y êtes invité, fournissez les informations d’identification suivantes et sélectionnez **Se connecter** :

   |Paramètre|Valeur|
   |---|---|
   |Nom d’utilisateur|**Student@adatum.com**|
   |Mot de passe|**Pa55w.rd1234**|

1. Dans la session Bureau à distance pour **az140-dc-vm11**, démarrez Microsoft Edge et accédez au [portail Azure](https://portal.azure.com). Si vous y êtes invité, connectez-vous à l’aide des informations d’identification Microsoft Entra du compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utilisez dans ce labo.
1. Dans la session Bureau à distance pour **az140-dc-vm11**, dans la fenêtre Microsoft Edge affichant le portail Azure, recherchez et sélectionnez **Comptes de stockage**, puis, dans le panneau **Comptes de stockage**, sélectionnez **+ Créer**.
1. Sous l’onglet **Options de base** du volet **Créer un compte de stockage**, spécifiez les paramètres suivants (conservez les valeurs par défaut pour les autres) :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|le nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Resource group|le nom d’un nouveau groupe de ressources **az140-22-RG**|
   |Nom du compte de stockage|n’importe quel nom global unique d’une longueur comprise entre 3 et 15 caractères, composé de lettres minuscules et de chiffres, et commençant par une lettre|
   |Région|le nom d’une région Azure hébergeant l’environnement de laboratoire Azure Virtual Desktop|
   |Performances|**Standard**|
   |Redondance|**Stockage géo-redondant (GRS)**|
   |Permettre l’accès en lecture aux données en cas d’indisponibilité régionale|enabled|

   >**Remarque** : Vérifiez que la longueur du nom du compte de stockage ne dépasse pas 15 caractères. Le nom va servir à créer un compte d’ordinateur dans le domaine Active Directory Domain Services (AD DS) intégré au locataire Microsoft Entra associé à l’abonnement Azure contenant le compte de stockage. Cela permet l’authentification basée sur AD DS lors de l’accès aux partages de fichiers hébergés dans ce compte de stockage.

1. Sous l’onglet **Informations de base** du volet **Créer un compte de stockage**, sélectionnez **Vérifier + créer**, attendez la fin du processus de validation, puis sélectionnez **Créer**.

   >**Remarque** : attendez que le compte de stockage soit créé. Ce processus prend environ 2 minutes.

#### Tâche 2 : Créer un partage Azure Files

1. Dans la session Bureau à distance pour **az140-dc-vm11**, dans la fenêtre Microsoft Edge affichant le Portail Azure, revenez au panneau **Comptes de stockage** et sélectionnez l’entrée représentant le compte de stockage nouvellement créé.
1. Dans le panneau compte de stockage, dans la section **Stockage de données**, sélectionnez **Partages de fichiers**, puis **+ Partage de fichiers**.
1. Dans le panneau **Nouveau partage de fichiers**, spécifiez les paramètres suivants et sélectionnez **Créer** (laissez d’autres paramètres avec leurs valeurs par défaut) :

   |Paramètre|Valeur|
   |---|---|
   |Nom|**az140-22-profiles**|
   |Niveaux|**Transaction optimisée**|

#### Tâche 3 : Activer l’authentification AD DS pour le compte de stockage Azure 

1. Dans la session Bureau à distance pour **az140-dc-vm11**, ouvrez un autre onglet dans la fenêtre Microsoft Edge, accédez au [référentiel GitHub d’exemples Azure Files](https://github.com/Azure-Samples/azure-files-samples/releases), téléchargez [la version la plus récente du module PowerShell compressé **AzFilesHybrid.zip** et extrayez son contenu dans le dossier **C :\\Allfiles\\Labs\\02** (créez le dossier si nécessaire).
1. Dans la session Bureau à distance pour **az140-dc-vm11**, démarrez **Windows PowerShell ISE** en tant qu’Administrateur et, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour supprimer le flux de données alternatif **Zone.Identifier**, qui a pour valeur **3**, indiquant qu’il a été téléchargé à partir d’Internet :

   ```powershell
   Get-ChildItem -Path C:\Allfiles\Labs\02 -File -Recurse | Unblock-File
   ```

1. Depuis la console **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour vous connecter à votre abonnement Azure :

   ```powershell
   Connect-AzAccount
   ```

1. Lorsque vous y êtes invité, connectez-vous avec les informations d’identification Microsoft Entra du compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utilisez dans ce labo.
1. Dans la session Bureau à distance pour **az140-dc-vm11**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour définir les variables nécessaires pour exécuter le script suivant :

   ```powershell
   $subscriptionId = (Get-AzContext).Subscription.Id
   $resourceGroupName = 'az140-22-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName
   ```

1. Dans la session Bureau à distance pour **az140-dc-vm11**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour créer un objet d’ordinateur AD DS qui représente le compte de stockage Azure que vous avez créé précédemment dans cette tâche et utilisé pour implémenter son authentification AD DS :

   >**Remarque** : Si vous recevez une erreur lors de l’exécution de ce bloc de script, vérifiez que vous êtes dans le même répertoire que le script CopyToPSPath.ps1. Selon la façon dont les fichiers ont été extraits précédemment dans ce laboratoire, ils peuvent se trouver dans un sous-dossier nommé AzFilesHybrid. Dans le contexte PowerShell, remplacez les répertoires pour accéder au dossier à l’aide de **cd AzFilesHybrid**.

   ```powershell
   Set-Location -Path 'C:\Allfiles\Labs\02'
   .\CopyToPSPath.ps1 
   Import-Module -Name AzFilesHybrid
   Join-AzStorageAccountForAuth `
      -ResourceGroupName $ResourceGroupName `
      -StorageAccountName $StorageAccountName `
      -DomainAccountType 'ComputerAccount' `
      -OrganizationalUnitDistinguishedName 'OU=WVDInfra,DC=adatum,DC=com'
   ```

1. Dans la session Bureau à distance pour **az140-dc-vm11**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour vérifier que l’authentification AD DS est activée sur le compte de stockage Azure :

   ```powershell
   $storageaccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName
   $storageAccount.AzureFilesIdentityBasedAuth.ActiveDirectoryProperties
   $storageAccount.AzureFilesIdentityBasedAuth.DirectoryServiceOptions
   ```

1. Vérifiez que la sortie de la commande `$storageAccount.AzureFilesIdentityBasedAuth.ActiveDirectoryProperties` retourne `AD`, ce qui représente le service d’annuaire du compte de stockage, et que la sortie de la commande `$storageAccount.AzureFilesIdentityBasedAuth.DirectoryServiceOptions`, qui représente les informations de domaine d’annuaire, ressemble au format suivant (les valeurs de `DomainGuid`, `DomainSid`et `AzureStorageSid` seront différentes) :

   ```
   DomainName        : adatum.com
   NetBiosDomainName : adatum.com
   ForestName        : adatum.com
   DomainGuid        : 47c93969-9b12-4e01-ab81-1508cae3ddc8
   DomainSid         : S-1-5-21-1102940778-2483248400-1820931179
   AzureStorageSid   : S-1-5-21-1102940778-2483248400-1820931179-2109
   ```

1. Dans la session Bureau à distance pour **az140-dc-vm11**, basculez vers la fenêtre Microsoft Edge affichant le portail Azure, dans le panneau affichant le compte de stockage, sélectionnez **Partages de fichiers** et vérifiez que le paramètre **Active Directory** est **Configuré**.

   >**Remarque** : Vous devrez peut-être actualiser la page du navigateur pour que la modification soit reflétée dans le portail Azure.

#### Tâche 4 : Configurer les autorisations basées sur le RBAC pour Azure Files

1. Dans la session Bureau à distance pour **az140-dc-vm11**, dans la fenêtre Microsoft Edge affichant le Portail Azure, dans le panneau affichant les propriétés du compte de stockage créé précédemment dans cet exercice, dans le menu vertical de gauche, dans la section **Stockage des données**, sélectionnez **Partages de fichiers**.
1. Dans le panneau **Partages de fichiers**, dans la liste des partages, sélectionnez l’entrée **az140-22-profiles**.
1. Dans le panneau **az140-22-profiles**, dans le menu vertical de gauche, sélectionnez **Access Control (IAM)**.
1. Dans le panneau **Contrôle d’accès (IAM)** du compte de stockage, sélectionnez **+ Ajouter** et, dans le menu déroulant, sélectionnez **Ajouter une attribution de rôle**, 
1. Dans le panneau **Ajouter une attribution de rôle**, spécifiez les paramètres suivants et sélectionnez **Vérifier + attribuer** :

   |Paramètre|Valeur|
   |---|---|
   |Role|**Contributeur de partage SMB de données de fichier de stockage**|
   |Attribuer l’accès à|**Utilisateur, groupe ou principal de service**|
   |Sélectionnez|**az140-wvd-users**|

1. Dans le panneau **Contrôle d’accès (IAM)** du compte de stockage, sélectionnez **+ Ajouter** et, dans le menu déroulant, sélectionnez **Ajouter une attribution de rôle**, 
1. Dans le panneau **Ajouter une attribution de rôle**, spécifiez les paramètres suivants et sélectionnez **Vérifier + attribuer** :

   |Paramètre|Valeur|
   |---|---|
   |Role|**Contributeur élevé de partage SMB de données de fichier de stockage**|
   |Attribuer l’accès à|**Utilisateur, groupe ou principal de service**|
   |Sélectionnez|**az140-wvd-admins**|

#### Tâche 5 : Configurer les autorisations du système de fichiers Azure Files

1. Dans la session Bureau à distance pour **az140-dc-vm11**, basculez vers la fenêtre **Administrateur : Windows PowerShell ISE** et, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour créer une variable référençant le nom et la clé du compte de stockage que vous avez créé précédemment dans cet exercice :

   ```powershell
   $resourceGroupName = 'az140-22-RG'
   $storageAccount = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0]
   $storageAccountName = $storageAccount.StorageAccountName
   $storageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $storageAccountName).Value[0]
   ```

1. Depuis la console **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour créer un mappage de lecteur vers le partage de fichiers que vous avez créé précédemment dans cet exercice :

   ```powershell
   $fileShareName = 'az140-22-profiles'
   net use Z: "\\$storageAccountName.file.core.windows.net\$fileShareName" /u:AZURE\$storageAccountName $storageAccountKey
   ```

1. Depuis la console **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour afficher les autorisations actuelles du système de fichiers :

   ```powershell
   icacls Z:
   ```

   >**Remarque** : Par défaut, les **Utilisateurs authentifiés\\NT Authority** et les **Utilisateurs\\BUILTIN** disposent d’autorisations permettant aux utilisateurs de lire les conteneurs de profil d’autres utilisateurs. Vous allez les supprimer et ajouter des autorisations minimales requises à la place.

1. Depuis la console **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour ajuster les autorisations du système de fichiers afin de respecter le principe des privilèges minimum :

   ```powershell
   $permissions = 'ADATUM\az140-wvd-admins'+':(F)'
   cmd /c icacls Z: /grant $permissions
   $permissions = 'ADATUM\az140-wvd-users'+':(M)'
   cmd /c icacls Z: /grant $permissions
   $permissions = 'Creator Owner'+':(OI)(CI)(IO)(M)'
   cmd /c icacls Z: /grant $permissions
   icacls Z: /remove 'Authenticated Users'
   icacls Z: /remove 'Builtin\Users'
   ```

   >**Remarque** : Vous pouvez également définir des autorisations à l’aide de l’Explorateur de fichiers.
