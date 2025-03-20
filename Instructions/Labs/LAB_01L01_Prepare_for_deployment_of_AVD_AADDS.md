---
lab:
  title: 'Labo : Préparer le déploiement d’Azure Virtual Desktop (Microsoft Entra DS)'
  module: 'Module 1: Plan an AVD Architecture'
---

# Labo : préparer le déploiement d’Azure Virtual Desktop (Microsoft Entra DS)
# Manuel de labo de l’étudiant

## Dépendances du labo

- Un abonnement Azure
- Un compte Microsoft ou un compte Microsoft Entra disposant du rôle Administrateur général pour le locataire Microsoft Entra qui est associé à l’abonnement Azure, et disposant du rôle Propriétaire ou Contributeur dans l’abonnement Azure

## Durée estimée

150 minutes

>**Remarque** : L’approvisionnement d’une instance Microsoft Entra DS comporte environ 90 minutes d’attente.

## Scénario du labo

Vous devez préparer le déploiement d’Azure Virtual Desktop dans un environnement Azure Active Directory Domain Services (Microsoft Entra DS)

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

- Implémenter un domaine Microsoft Entra DS
- Configurer l’environnement de domaine Microsoft Entra DS

## Fichiers du labo

-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json

## Instructions

### Exercice 0 : Augmenter le nombre de quotas des processeurs virtuels

Les principales tâches de cet exercice sont les suivantes

1. Identifier l’utilisation actuelle des processeurs virtuels
1. Demander une augmentation de quota des processeurs virtuels

#### Tâche 1 : Identifier l’utilisation actuelle des processeurs virtuels

1. À partir de votre ordinateur de labo, démarrez un navigateur web, accédez au [Portail Azure](https://portal.azure.com), puis connectez-vous en fournissant des informations d’identification d’un compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utiliserez dans ce labo.
1. Dans le Portail Azure, ouvrez le volet **Cloud Shell** en sélectionnant l’icône de barre d’outils directement à droite de la zone de texte de recherche.
1. Lorsque vous êtes invité à sélectionner **Bash** ou **PowerShell**, sélectionnez **PowerShell**. 

   >**Remarque** : si c’est la première fois que vous démarrez **Cloud Shell** et que vous voyez le message **Vous n’avez aucun stockage monté**, sélectionnez l’abonnement que vous utilisez dans ce labo, puis sélectionnez **Créer un stockage**. 

1. Dans le Portail Azure, dans la session PowerShell de **Cloud Shell**, exécutez la commande suivante pour inscrire le fournisseur de ressources **Microsoft.Compute** s’il n’est pas inscrit :

   ```powershell
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Compute'
   ```

1. Dans le Portail Azure, dans la session PowerShell de **Cloud Shell**, exécutez la commande suivante pour vérifier l’état d’inscription du fournisseur de ressources **Microsoft.Compute** :

   ```powershell
   Get-AzResourceProvider -ListAvailable | Where-Object {$_.ProviderNamespace -eq 'Microsoft.Compute'}
   ```

   >**Remarque** : Vérifiez que l’état indique **Inscrit**. Si ce n’est pas le cas, patientez quelques minutes, puis répétez cette étape.

1. Dans le Portail Azure, dans la session PowerShell de **Cloud Shell**, exécutez la commande suivante pour créer une variable avec le nom d’une région Azure (remplacez l’espace réservé `<Azure_region>` par le nom de la région Azure que vous envisagez d’utiliser pour ce labo, par exemple `eastus`) :

   ```powershell
   $location = '<Azure_region>'
   ```

   > **Remarque** : Pour identifier les noms des régions Azure, dans **Cloud Shell**, à l’invite PowerShell, exécutez `(Get-AzLocation).Location`.
   
1. Dans le Portail Azure, dans la session PowerShell de **Cloud Shell**, exécutez la commande suivante pour identifier l’utilisation actuelle des processeurs virtuels et les limites correspondantes pour les machines virtuelles Azure **StandardDSv3Family** :

   ```powershell
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}
   ```

1. Passez en revue le résultat de la commande exécutée à l’étape précédente et vérifiez que vous disposez d’au moins **20** processeurs virtuels disponibles dans **Famille DSv3 Standard** des machines virtuelles Azure dans la région Azure cible. Si c’est déjà le cas, passez directement à l’exercice suivant. Sinon, passez à la tâche suivante de cet exercice. 

#### Tâche 2 : Demander une augmentation de quota des processeurs virtuels

1. Dans le Portail Azure, recherchez et sélectionnez **Abonnements**, puis, dans le panneau **Abonnements**, sélectionnez l’entrée représentant l’abonnement Azure que vous envisagez d’utiliser pour ce labo.
1. Dans le Portail Azure, dans le panneau abonnement du menu vertical à gauche, dans la section **Paramètres**, sélectionnez **Utilisation + quotas**. 
1. Sur le panneau  **Utilisation + quotas** , sélectionnez les flèches déroulantes suivantes dans la barre de recherche supérieure :

   |**Paramètre**|**Valeur**|
   |---|---|
   |**action**|**DSv3 Standard**|
   |**Tous les emplacements**|**Effacer tout**, puis vérifiez *votre emplacement*|
   |**Fournisseur de ressources** | **Microsoft.Compute** |
   
1. Dans l’élément **Processeurs virtuels Famille DSv3 Standard** retourné, sélectionnez l’icône de crayon, **Modifier**.
1. Dans le panneau **Détails du quota**, dans la zone de texte de colonne **Nouvelle limite**, tapez **30**, puis sélectionnez **Enregistrer et continuer**.
1. Permettez à la demande de quota de se terminer.  Après quelques instants, le panneau **Détails du quota** spécifiera l’approbation de la demande et l’augmentation du quota. Fermez le panneau **Détails du quota**.

    >**Remarque** : Selon le choix de la région Azure et la demande actuelle, il peut être nécessaire de créer une demande de support. Pour obtenir des instructions sur le processus de création d’une demande de support, reportez-vous à [Créer une demande de support Azure](https://docs.microsoft.com/en-us/azure/azure-portal/supportability/how-to-create-azure-support-request).

### Exercice 1 : Implémenter un domaine Azure Active Directory Domain Services (AD DS)

Les principales tâches de cet exercice sont les suivantes

1. Créer et configurer un compte d’utilisateur Microsoft Entra pour l’administration du domaine Microsoft Entra DS
1. Déployer une instance Microsoft Entra DS en utilisant le Portail Azure
1. Configurer les paramètres de réseau et d’identité du déploiement Microsoft Entra DS

#### Tâche 1 : Créer et configurer un compte d’utilisateur Microsoft Entra pour l’administration d’un domaine Microsoft Entra DS

1. À partir de votre ordinateur de labo, démarrez un navigateur web, accédez au [Portail Azure](https://portal.azure.com), puis connectez-vous en fournissant les informations d’identification d’un compte d’utilisateur disposant du rôle Propriétaire dans l’abonnement que vous utiliserez dans ce labo et du rôle Administrateur général dans le tenant Microsoft Entra associé à l’abonnement Azure.
1. Dans le navigateur web affichant le Portail Azure, accédez au panneau **Vue d’ensemble** du tenant Microsoft Entra et, dans la section **Gérer** du menu vertical à gauche, cliquez sur **Propriétés**.
1. En bas du panneau **Propriétés** de votre locataire Microsoft Entra, sélectionnez le lien **Gérer la sécurité par défaut** .
1. Dans le panneau **Activer la sécurité par défaut**, si nécessaire, sélectionnez **Non**, puis sélectionnez la case **Mon organisation utilise l’accès conditionnel**, puis sélectionnez **Enregistrer**.
1. Dans le Portail Azure, ouvrez le volet **Cloud Shell** en sélectionnant l’icône de barre d’outils directement à droite de la zone de texte de recherche.
1. Lorsque vous êtes invité à sélectionner **Bash** ou **PowerShell**, sélectionnez **PowerShell**. 

   >**Remarque** : si c’est la première fois que vous démarrez **Cloud Shell** et que vous voyez le message **Vous n’avez aucun stockage monté**, sélectionnez l’abonnement que vous utilisez dans ce labo, puis sélectionnez **Créer un stockage**. 

1. Dans le volet Cloud Shell, exécutez la commande suivante pour vous connecter à votre tenant Microsoft Entra :

   ```powershell
   Connect-AzureAD
   ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour récupérer le nom de domaine du DNS principal du tenant Microsoft Entra associé à votre abonnement Azure :

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   $aadDomainName
   ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour créer des utilisateurs Microsoft Entra qui bénéficieront de privilèges élevés (remplacez l’espace réservé `<password>` par un mot de passe aléatoire et complexe) :

   > **Remarque** : Veillez à ne pas oublier le mot de passe utilisé. Vous en aurez besoin plus loin dans ce labo et dans les labos suivants.

   ```powershell
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName 'aadadmin1' -PasswordProfile $passwordProfile -MailNickName 'aadadmin1' -UserPrincipalName "aadadmin1@$aadDomainName"
   New-AzureADUser -AccountEnabled $true -DisplayName 'wvdaadmin1' -PasswordProfile $passwordProfile -MailNickName 'wvdaadmin1' -UserPrincipalName "wvdaadmin1@$aadDomainName"
   ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour attribuer le rôle Administrateur général au premier des utilisateurs Microsoft Entra récemment créés :

   ```powershell
   $aadUser = Get-AzureADUser -ObjectId "aadadmin1@$aadDomainName"
   $aadRole = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Global administrator'}
   Add-AzureADDirectoryRoleMember -ObjectId $aadRole.ObjectId -RefObjectId $aadUser.ObjectId
   ```

   > **Remarque** : Le module Azure AD PowerShell fait référence au rôle Administrateur général en tant qu’Administrateur d’entreprise.

1. Dans le volet Cloud Shell, exécutez la commande suivante pour identifier le nom d’utilisateur principal de l’utilisateur Microsoft Entra nouvellement créé :

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").UserPrincipalName
   ```

   > **Remarque** : Enregistrez le nom d’utilisateur principal. Vous en aurez besoin plus loin dans cet exercice. 

1. Fermez le volet Cloud Shell.
1. Dans le Portail Azure, recherchez et sélectionnez **Abonnements**, puis, dans le panneau **Abonnements**, sélectionnez l’abonnement Azure que vous utilisez pour ce labo. 
1. Dans le panneau affichant les propriétés de votre abonnement Azure, sélectionnez **Contrôle d’accès (IAM), **sélectionnez **Ajouter**, puis **Ajouter une attribution de rôle**. 
1. Dans le panneau **Ajouter une attribution de rôle**, sélectionnez **Propriétaire**, puis cliquez sur **Suivant**
1. Cliquez sur le lien hypertexte **+Sélectionner des membres**.
1. Dans le panneau **Sélectionner des membres**, sélectionnez l’élément **aadadmin1**, puis cliquez sur le bouton **Sélectionner**, enfin sur **Suivant**.
1. Dans le panneau **Vérifier + attribuer**, sélectionnez le bouton **Vérifier + attribuer**.

   > **Remarque** : Vous allez utiliser le compte **aadadmin1** pour gérer votre abonnement Azure et le tenant Microsoft Entra correspondant à partir d’une machine virtuelle Microsoft Entra DS jointe à une machine virtuelle Azure Windows 10 ultérieurement dans le labo. 


#### Tâche 2 : Déployer une instance Microsoft Entra DS en utilisant le Portail Azure

1. À partir de votre ordinateur de labo, dans le Portail Azure, recherchez et sélectionnez **Microsoft Entra Domain Services** et, dans le panneau **Microsoft Entra Domain Services**, sélectionnez **+ Créer**. Cette action ouvre le panneau **Créer Microsoft Entra Domain Services**.
1. Sous l’onglet **Informations de base** du panneau **Créer Microsoft Entra Domain Services**, spécifiez les paramètres suivants, puis sélectionnez **Suivant** (conservez les autres avec leurs valeurs existantes) :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|le nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Resource group|Sélectionnez Créer **az140-11a-RG**|
   |Nom du domaine DNS|**adatum.com**|
   |Région|nom de la région où vous souhaitez héberger votre déploiement AVD|
   |Référence (SKU)|**Standard**|

   > **Remarque** : Bien que cela ne soit techniquement pas obligatoire, vous devriez en général affecter un nom de domaine Microsoft Entra DS différent de l’espace de noms DNS local ou Azure existant.

1. Sous l’onglet **Mise en réseau** du panneau **Créer Microsoft Entra Domain Service**, à côté de la liste déroulante **Réseau virtuel**, sélectionnez **Créer**.
1. Dans le panneau **Créer un réseau virtuel**, attribuez les paramètres suivants, puis sélectionnez **OK** :

   |Paramètre|Valeur|
   |---|---|
   |Nom|**az140-aadds-vnet11a**|
   |Plage d’adresses|**10.10.0.0/16**|
   |Nom du sous-réseau|**aadds-Subnet**|
   |Plage d’adresses|**10.10.0.0/24**|

1. De retour sous l’onglet **Mise en réseau** du panneau **Créer un réseau virtuel**, sélectionnez **Suivant** (conservez les autres avec leurs valeurs existantes).
1. Sous l’onglet **Administration** du panneau **Créer Microsoft Entra Domain Services**, acceptez les paramètres par défaut et sélectionnez **Suivant**.
1. Sous l’onglet **Synchronisation** du panneau **Créer Microsoft Entra Domain Services**, vérifiez que **Tout** est sélectionné, puis sélectionnez **Suivant**.
1. Sous l’onglet **Paramètres de sécurité** du panneau **Créer Microsoft Entra Domain Services**, acceptez les paramètres par défaut et sélectionnez **Suivant**.
1. Sous l’onglet **Balises** du panneau **Créer Microsoft Entra Domain Services**, acceptez les paramètres par défaut et sélectionnez Suivant
2. Sous l’onglet **Vérifier + créer** du panneau **Créer Microsoft Entra Domain Services**, sélectionnez **Créer**. 
3. Passez en revue la notification concernant les paramètres que vous ne pourrez pas modifier après la création du domaine Microsoft Entra DS, puis sélectionnez **OK**.

   >**Remarque** : Les paramètres pour lesquels vous ne pourrez pas modifier l’approvisionnement suivant d’un domaine Microsoft Entra DS incluent son nom DNS, son abonnement Azure, son groupe de ressources, le réseau virtuel et le sous-réseau hébergeant ses contrôleurs de domaine et le type de forêt.

   > **Remarque** : Attendez la fin du déploiement avant de passer à l’exercice suivant. Cette opération peut prendre 90 minutes environ. 

#### Tâche 3 : Configurer les paramètres de réseau et d’identité du déploiement Microsoft Entra DS

1. À partir de votre ordinateur de labo, dans le Portail Azure, recherchez et sélectionnez **Microsoft Entra Domain Services** et, dans le panneau **Microsoft Entra Domain Services**, sélectionnez l’entrée **adatum.com** pour accéder à l’instance Microsoft Entra DS nouvellement approvisionnée. 
1. Dans le panneau **adatum.com** de l’instance Microsoft Entra DS, cliquez sur l’avertissement commençant par **Des problèmes de configuration ont été détectés pour votre domaine managé**. 
1. Dans le panneau **adatum.com | Diagnostics de configuration**, cliquez sur **Exécuter**.
1. Dans la section **Validation**, développez le volet **Enregistrements DNS**, puis cliquez sur **Corriger**.
1. Dans le panneau **Enregistrements DNS**, cliquez à nouveau sur **Corriger**.
1. Revenez au panneau **adatum.com** de l’instance Microsoft Entra DS et, dans la section **Étapes de configuration requises**, passez en revue les informations relatives à la synchronisation de hachage de mot de passe Microsoft Entra DS. 

   > **Remarque** : Tous les utilisateurs cloud uniquement existants qui doivent être en mesure d’accéder aux ordinateurs de domaine Microsoft Entra DS et à leurs ressources doivent modifier leur mot de passe ou les réinitialiser. Cela s’applique au compte **aadadmin1** que précédemment créé dans ce labo.

1. À partir de votre ordinateur de labo, dans le Portail Azure, ouvrez une session **PowerShell** dans le volet **Cloud Shell**.
1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez la commande suivante pour identifier l’attribut objectID du compte d’utilisateur **aadadmin1** Microsoft Entra :

   ```powershell
   Connect-AzureAD
   $objectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   ```

1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez la commande suivante pour réinitialiser le mot de passe du compte d’utilisateur **aadadmin1**, dont vous avez identifié l’objectId à l’étape précédente (remplacez l’espace réservé `<password>` par un mot de passe aléatoire et complexe) :

   > **Remarque** : Veillez à ne pas oublier le mot de passe utilisé. Vous en aurez besoin plus loin dans ce labo et dans les labos suivants.

   ```powershell
   $password = ConvertTo-SecureString '<password>' -AsPlainText -Force
   Set-AzureADUserPassword -ObjectId $objectId -Password $password -ForceChangePasswordNextLogin $false
   ```

   > **Remarque** : Dans les scénarios réels, vous devez généralement définir la valeur de **-ForceChangePasswordNextLogin** sur $true. Nous avons choisi **$false** dans ce cas pour simplifier les étapes du labo. 

1. Répétez les deux étapes précédentes pour réinitialiser le mot de passe du compte d’utilisateur **wvdaadmin1**.


### Exercice 2 : Configurer l’environnement de domaine Microsoft Entra DS
  
Les principales tâches de cet exercice sont les suivantes

1. Déployer une machine virtuelle Azure exécutant Windows 10 en utilisant des modèles de démarrage rapide Azure Resource Manager
1. Déployer Azure Bastion
1. Passer en revue la configuration par défaut du domaine Microsoft Entra DS
1. Créer des utilisateurs et des groupes AD DS qui seront synchronisés avec Microsoft Entra DS

#### Tâche 1 : Déployer une machine virtuelle Azure exécutant Windows 10 en utilisant des modèles de démarrage rapide Azure Resource Manager

1. À partir de votre ordinateur de labo, dans le Portail Azure, à partir de la session PowerShell dans le volet Cloud Shell, exécutez la commande suivante pour ajouter un sous-réseau nommé **cl-Subnet** au réseau virtuel nommé **az140-adds-vnet11a** créé dans la tâche précédente :

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-aadds-vnet11a'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'cl-Subnet' `
     -AddressPrefix 10.10.255.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. À partir de votre ordinateur de labo, recherchez le fichier de paramètres **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json**, puis ouvrez le fichier en utilisant Visual Studio Code.

1.  À la ligne 21, recherchez la valeur du paramètre domainPassword. Mettez à jour le mot de passe existant dans le fichier de paramètres pour utiliser le mot de passe défini plus tôt dans ce labo pour le compte d’utilisateur **aadadmin1**, puis effectuez l’opération pour **Enregistrer** le fichier.

1. Dans le Portail Azure, dans la barre d’outils du volet Cloud Shell, sélectionnez l’icône **Charger/télécharger des fichiers**, puis dans le menu déroulant, sélectionnez **Charger**, puis chargez les fichiers **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json** et **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json** dans le répertoire de base Cloud Shell.
1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez la commande suivante pour déployer une machine virtuelle Azure exécutant Windows 10 qui servira de client Azure Virtual Desktop dans le domaine Microsoft Entra DS :

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0101vmDeployment `
     -TemplateFile $HOME/az140-11_azuredeploycl11a.json `
     -TemplateParameterFile $HOME/az140-11_azuredeploycl11a.parameters.json
   ```

   > **Remarque** : Le déploiement peut prendre environ 10 minutes. Attendez la fin du déploiement avant de passer à la tâche suivante. 


#### Tâche 2 : Déployer Azure Bastion 

> **Remarque** : Azure Bastion autorise la connexion aux machines virtuelles Azure sans les points de terminaison publics que vous avez déployés dans la tâche précédente de cet exercice, tout en fournissant une protection contre les attaques par force brute qui ciblent les informations d’identification au niveau du système d’exploitation.

> **Remarque** : Vérifiez que votre navigateur dispose de la fonctionnalité de fenêtre contextuelle activée.

1. Dans la fenêtre du navigateur affichant le Portail Azure, ouvrez un autre onglet, et, dans l’onglet du navigateur, accédez au [**Portail Azure**](https://portal.azure.com).
1. Dans le Portail Azure, ouvrez le volet **Cloud Shell** en sélectionnant l’icône de barre d’outils directement à droite de la zone de texte de recherche.
1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez la commande suivante pour ajouter un sous-réseau nommé **AzureBastionSubnet** au réseau virtuel nommé **az140-adds-vnet11** créé précédemment dans cet exercice :

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-aadds-vnet11a'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.10.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Fermez le volet Cloud Shell.
1. Dans le portail Azure, recherchez et sélectionnez **Bastions**, puis dans le panneau **Bastions**, sélectionnez **+ Créer**.
1. Sous l’onglet **De base** du panneau **Créer un bastion**, spécifiez les paramètres suivants et sélectionnez **Vérifier + créer** :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|le nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Resource group|**az140-11a-RG**|
   |Nom|**az140-11a-bastion**|
   |Région|la même région Azure que celle dans laquelle vous avez déployé les ressources lors des tâches précédentes de cet exercice|
   |Niveau|**De base**|
   |Réseau virtuel|**az140-aadds-vnet11a**|
   |Sous-réseau|**AzureBastionSubnet (10.10.254.0/24)**|
   |Adresse IP publique|**Création**|
   |Nom IP publique|**az140-aadds-vnet11a-ip**|

1. Sous l’onglet **Vérifier + créer** du panneau **Créer un bastion**, sélectionnez **Créer** :

   > **Remarque** : Attendez la fin du déploiement avant de passer à la tâche suivante de cet exercice. Le déploiement peut prendre environ 5 minutes.


#### Tâche 3 : Passer en revue la configuration par défaut du domaine Microsoft Entra DS

> **Remarque** : Avant de pouvoir vous connecter au nouvel ordinateur joint à Microsoft Entra DS, vous devez ajouter le compte d’utilisateur avec lequel vous souhaitez vous connecter au groupe Microsoft Entra **Administrateurs AAD DC**. Ce groupe Microsoft Entra est créé automatiquement dans le tenant Microsoft Entra associé à l’abonnement Azure où vous avez approvisionné l’instance Microsoft Entra DS.

> **Remarque** : Vous avez la possibilité de remplir ce groupe avec des comptes d’utilisateur Microsoft Entra existants lorsque vous approvisionnez une instance Microsoft Entra DS.

1. À partir de votre ordinateur de labo, dans le Portail Azure, à partir du volet Cloud Shell, exécutez la commande suivante pour ajouter le compte d’utilisateur **aadadmin1** Microsoft Entra au groupe **Administrateurs AAD DC** Microsoft Entra :

   ```powershell
   Connect-AzureAD
   $groupObjectId = (Get-AzureADGroup -Filter "DisplayName eq 'AAD DC Administrators'").ObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $groupObjectId -RefObjectId $userObjectId
   ```

1. Fermez le volet Cloud Shell.
1. À partir de votre ordinateur de labo, dans le Portail Azure, recherchez et sélectionnez **Machines virtuelles**, puis, dans le panneau **Machines virtuelles**, sélectionnez l’entrée **az140-cl-vm11a**. Cette opération ouvre le panneau **az140-cl-vm11a**.
1. Dans le panneau **az140-cl-vm11a**, sélectionnez **Se connecter**, puis dans le menu déroulant, sélectionnez **Bastion**. Sous l’onglet **Bastion** du panneau **az140-cl-vm11a**, fournissez les informations d’identification suivantes, puis sélectionnez **Utiliser Bastion** :
1. Lorsque vous y êtes invité, connectez-vous en tant qu’utilisateur **aadadmin1** en utilisant son nom principal que vous avez identifié précédemment dans ce labo et le mot de passe défini pour ce compte d’utilisateur lors de sa création plus tôt dans le labo.
1. Dans le service Bastion vers la machine virtuelle **az140-cl-vm11a**, démarrez **Windows PowerShell ISE** en tant qu’Administrateur et, à partir du volet de script **Adminstrateur : Windows PowerShell ISE**, exécutez la commande suivante pour installer l’instance Active Directory et les Outils d’administration de serveur distant associés au DNS :

   ```powershell
   Add-WindowsCapability -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.Dns.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.ServerManager.Tools~~~~0.0.1.0 -Online
   ```

   > **Remarque** : Attendez la fin de l’installation avant de passer à l’étape suivante. Cela peut prendre environ 2 minutes.

1. Dans le service Bastion vers la machine virtuelle Azure **az140-cl-vm11a**, dans le menu **Démarrer**, accédez au dossier **Outils d’administration Windows**, développez-le et, dans la liste des outils, démarrez **Utilisateurs et ordinateurs Active Directory**. 
1. Dans la console **Utilisateurs et ordinateurs Active Directory**, passez en revue la hiérarchie par défaut, notamment les unités d’organisation **Ordinateurs AADDC** et **Utilisateurs AADDC**. Notez que la première inclut le compte d’ordinateur **az140-cl-vm11a** et que la dernière inclut les comptes d’utilisateur synchronisés à partir du tenant Microsoft Entra associé à l’abonnement Azure hébergeant le déploiement de l’instance Microsoft Entra DS. L’unité d’organisation **Utilisateurs AADDC** inclut également le groupe **Administrateurs AAD DC** synchronisé à partir du même tenant Microsoft Entra, ainsi que son appartenance au groupe. Cette appartenance ne peut pas être modifiée directement dans le domaine Microsoft Entra DS et vous devez plutôt la gérer au sein du tenant Microsoft Entra DS. Toutes les modifications sont automatiquement synchronisées avec le réplica du groupe hébergé dans le domaine Microsoft Entra DS. 

   **Conseil :** Si les **Utilisateurs et ordinateurs Active Directory** ne répertorie pas de contenu lié au domaine, cliquez avec le bouton droit sur **Utilisateurs et ordinateurs Active Directory**, puis sélectionnez **Modifier le domaine** et choisissez le domaine **Adatum**.

   > **Remarque** : Actuellement, le groupe inclut uniquement le compte d’utilisateur **aadadmin1**.

1. Dans la console **Utilisateurs et ordinateurs Active Directory**, dans l’unité d’organisation **Utilisateurs AADDC**, sélectionnez le compte d’utilisateur **aadadmin1**, affichez sa boîte de dialogue **Propriétés**, basculez vers l’onglet **Comptes**, puis notez que le suffixe de nom d’utilisateur principal correspond au nom de domaine DNS Microsoft Entra principal et n’est pas modifiable. 
1. Dans la console **Utilisateurs et ordinateurs Active Directory**, passez en revue le contenu de l’unité d’organisation **Contrôleurs de domaine** et notez qu’elle inclut des comptes d’ordinateur de deux contrôleurs de domaine avec des noms générés de manière aléatoire. 

#### Tâche 4 : Créer des utilisateurs et des groupes AD DS qui seront synchronisés avec Microsoft Entra DS

1. Dans le service Bastion vers la machine virtuelle Azure **az140-cl-vm11a**, ouvrez Microsoft Edge, accédez au [Portail Azure](https://portal.azure.com) et connectez-vous en fournissant le nom d’utilisateur principal du compte d’utilisateur **aadadmin1** avec le mot de passe défini plus tôt dans ce labo en tant que son mot de passe.
1. Dans le Portail Azure, ouvrez **Cloud Shell**.
1. Lorsque vous êtes invité à sélectionner **Bash** ou **PowerShell**, sélectionnez **PowerShell**. 

   >**Remarque** : Étant donné que c’est la première fois que vous démarrez **Cloud Shell** en utilisant le compte d’utilisateur **aadadmin1**, vous devez configurer son répertoire de base Cloud Shell. Lorsque vous voyez le message **Vous n’avez aucun stockage monté**, sélectionnez l’abonnement que vous utilisez dans ce labo, puis **Créer un stockage**. 

1. À partir de la session PowerShell du volet Cloud Shell, exécutez la commande suivante pour vous connecter afin d’authentifier votre tenant Microsoft Entra :

   ```powershell
   Connect-AzureAD
   ```

1. À partir de la session PowerShell du volet Cloud Shell, exécutez la commande suivante pour récupérer le nom de domaine du DNS principal du tenant Microsoft Entra associé à votre abonnement Azure :

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez la commande suivante pour créer les comptes d’utilisateur Microsoft Entra que vous utiliserez dans les labos à venir (remplacez l’espace réservé `<password>` par un mot de passe aléatoire et complexe) :

   > **Remarque** : Veillez à ne pas oublier le mot de passe utilisé. Vous en aurez besoin plus loin dans ce labo et dans les labos suivants.

   ```powershell
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   $aadUserNamePrefix = 'aaduser'
   $userCount = 1..9
   foreach ($counter in $userCount) {
     New-AzureADUser -AccountEnabled $true -DisplayName "$aadUserNamePrefix$counter" -PasswordProfile $passwordProfile -MailNickName "$aadUserNamePrefix$counter" -UserPrincipalName "$aadUserNamePrefix$counter@$aadDomainName"
   } 
   ```

1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez la commande suivante pour créer un groupe Microsoft Entra nommé **az140-wvd-aadmins** et l’ajouter aux comptes d’utilisateur **aadadmin1** et **wvdaadmin1** :

   ```powershell
   $az140wvdaadmins = New-AzureADGroup -Description 'az140-wvd-aadmins' -DisplayName 'az140-wvd-aadmins' -MailEnabled $false -SecurityEnabled $true -MailNickName 'az140-wvd-aadmins'
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'wvdaadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   ```

1. À partir du volet Cloud Shell, répétez l’étape précédente pour créer des groupes Microsoft Entra pour les utilisateurs que vous utiliserez dans les labos à venir et ajoutez-les aux comptes d’utilisateur Microsoft Entra précédemment créés :

   >**Remarque** : Remarque : En raison de la taille limitée du Presse-papiers sur la machine virtuelle, toutes les applets de commande répertoriées ne sont pas copiées correctement. Ouvrez Bloc-notes sur la machine virtuelle et copiez-y toutes les applets de commande en utilisant la construction de commande Saisir le texte, Saisir le texte du Presse-papiers qui fait partie du contrôle Lightning Bolt. Une fois que vous avez vérifié que toutes les applets de commande se trouvent dans Bloc-notes, coupez et collez-les dans des blocs de Cloud Shell et exécutez-les.

   ```powershell
   $az140wvdausers = New-AzureADGroup -Description 'az140-wvd-ausers' -DisplayName 'az140-wvd-ausers' -MailEnabled $false -SecurityEnabled $true -MailNickName 'az140-wvd-ausers'
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser2'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser3'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser4'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser5'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser6'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser7'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser8'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser9'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId

   $az140wvdaremoteapp = New-AzureADGroup -Description "az140-wvd-aremote-app" -DisplayName "az140-wvd-aremote-app" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-aremote-app"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser5'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser6'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId

   $az140wvdapooled = New-AzureADGroup -Description "az140-wvd-apooled" -DisplayName "az140-wvd-apooled" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-apooled"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser2'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser3'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser4'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId

   $az140wvdapersonal = New-AzureADGroup -Description "az140-wvd-apersonal" -DisplayName "az140-wvd-apersonal" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-apersonal"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser7'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser8'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser9'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   ```

1. Fermez le volet Cloud Shell.
1. Dans le service Bastion vers la machine virtuelle Azure **az140-cl-vm11a**, dans la fenêtre Microsoft Edge affichant le Portail Azure, recherchez et sélectionnez le panneau **Azure Active Directory**, dans le panneau de votre tenant Microsoft Entra, dans la barre de menus verticale située à gauche dans la section **Gérer**, sélectionnez **Utilisateurs** et, dans le panneau **Utilisateurs \| Tous les utilisateurs**, vérifiez que de nouveaux comptes d’utilisateur ont été créés.
1. Revenez au panneau du tenant Microsoft Entra, dans la barre de menus verticale située à gauche, dans la section **Gérer**, sélectionnez **Groupes** et, dans le panneau **Groupes \| Tous les groupes**, vérifiez que de nouveaux comptes de groupe ont été créés.
1. Dans le service Bastion vers la machine virtuelle Azure **az140-cl-vm11a**, basculez vers la console **Utilisateurs et ordinateurs Active Directory**, dans la console **Utilisateurs et ordinateurs Active Directory**, accédez à l’unité d’organisation **Utilisateurs AADDC** et vérifiez qu’elle contient les mêmes comptes d’utilisateur et de groupe.

   >**Remarque** : Il est possible que vous deviez actualiser la vue de la console.
