---
lab:
  title: "Labo : Préparer le déploiement d’Azure Virtual Desktop (AD\_DS)"
  module: 'Module 1: Plan an AVD Architecture'
---

# Labo : Préparer le déploiement d’Azure Virtual Desktop (AD DS)
# Manuel de labo de l’étudiant

## Dépendances du labo

- Un abonnement Azure que vous utiliserez dans ce labo.
- Un compte Microsoft ou un compte Microsoft Entra avec le rôle Propriétaire ou Contributeur dans l’abonnement Azure que vous utiliserez dans ce labo et avec le rôle Administrateur général dans le tenant (locataire) Microsoft Entra associé à cet abonnement Azure.

## Durée estimée

60 minutes

## Scénario du labo

Vous devez préparer le déploiement d’un environnement Active Directory Domain Services (AD DS)

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

- Déployer une forêt à domaine unique Active Directory Domain Services (AD DS) à l’aide de machines virtuelles Azure
- Intégrer une forêt AD DS avec un locataire Microsoft Entra

## Fichiers du labo

-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploydc11.parameters.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.parameters.json

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

1. Dans le Portail Azure, dans la session PowerShell de **Cloud Shell**, exécutez la commande suivante pour inscrire les fournisseurs de ressources **Microsoft.Compute** et **Microsoft.Network**, au cas où ils ne sont pas inscrits :

   ```powershell
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Compute'
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Network'
   ```

1. Dans le Portail Azure, dans la session PowerShell de **Cloud Shell**, exécutez la commande suivante pour vérifier l’état d’inscription du fournisseur de ressources **Microsoft.Compute** :

   ```powershell
   Get-AzResourceProvider -ListAvailable | Where-Object {$_.ProviderNamespace -eq 'Microsoft.Compute'}
   ```

   >**Remarque** : Vérifiez que l’état indique **Inscrit**. Si ce n’est pas le cas, patientez quelques minutes, puis répétez cette étape.

1. Dans le Portail Azure, dans la session PowerShell de **Cloud Shell**, exécutez la commande suivante pour définir l’emplacement des commandes suivantes (remplacez l’espace `<Azure_region>` réservé par le nom de la région Azure que vous envisagez d’utiliser pour ce labo, par exemple `eastus`) :

   ```powershell
   $location = '<Azure_region>'
   ```

1. Dans le Portail Azure, dans la session PowerShell de **Cloud Shell**, exécutez la commande suivante pour identifier l’utilisation actuelle des processeurs virtuels et les limites correspondantes pour les machines virtuelles Azure **StandardDSv3Family** et **StandardBSFamily** : 

   ```powershell
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}
   ```

   > **Remarque** : Pour identifier les noms des régions Azure, dans **Cloud Shell**, à l’invite PowerShell, exécutez `(Get-AzLocation).Location`.
   
1. Passez en revue le résultat de la commande exécutée à l’étape précédente et vérifiez que vous disposez d’au moins **30** processeurs virtuels disponibles dans la liste de **Processeurs virtuels de famille DSv3 standard** des machines virtuelles Azure dans la région Azure cible. Si c’est déjà le cas, passez directement à l’exercice suivant. Sinon, passez à la tâche suivante de cet exercice. 

#### Tâche 2 : Demander une augmentation de quota des processeurs virtuels

1. Dans le Portail Azure, recherchez et sélectionnez **Abonnements**, puis, dans le panneau **Abonnements**, sélectionnez l’entrée représentant l’abonnement Azure que vous envisagez d’utiliser pour ce labo.
1. Dans le portail Azure, ouvrez le panneau abonnement, puis le menu vertical à gauche, et dans la section **Paramètres**, sélectionnez **Utilisation + quotas**. 

   **Remarque :** Vous n’aurez peut-être pas besoin de créer un ticket de support pour augmenter les quotas.

   **Remarque :** La demande d’augmentation du quota nécessite la connexion par authentification multifacteur (MFA). Si vous devez configurer la MFA sur votre compte, reportez-vous à [Planifier un déploiement Azure Active Directory avec authentification multifacteur](https://learn.microsoft.com/en-us/azure/active-directory/authentication/howto-mfa-getstarted). 
   
1. Sur le panneau **Pass Azure – Parrainage | Utilisation + quotas**, sélectionnez **Région**, dans la liste déroulante, sélectionnez la case à côté du nom de la région Azure que vous envisagez d’utiliser pour ce labo, sélectionnez **Appliquer**, vérifiez que l’entrée **Calcul** apparaît dans la liste déroulante à gauche de l’entrée **Région** et, dans la zone de recherche, tapez **Standard DSv3**. 
1. Dans la liste des résultats, sélectionnez la case à côté de l’élément **Processeurs virtuels de famille DSv3 Standard**, sélectionnez l’entrée **Demander une augmentation de quota** dans la barre d’outils puis, dans la liste déroulante, sélectionnez **Entrer une nouvelle limite**.
1. Une fois dans le volet **Demander une augmentation de quota**, dans la zone de texte **Nouvelle limite** , tapez **30**, puis sélectionnez **Envoyer**.
1. Si vous y êtes invité, dans le volet **Demander une augmentation de quota**, sélectionnez **S’authentifier avec l’authentification multifacteur** et suivez les invites pour vous authentifier.
1. Permettez à la demande de quota de se compléter.  Après quelques instants, le panneau **Détails du quota** spécifiera l’approbation de la demande et l’augmentation du quota. Fermez le panneau **Détails du quota**.

   >**Remarque** : Selon le choix de la région Azure et la demande actuelle, il peut être nécessaire de créer une demande de support. Pour obtenir des instructions sur le processus de création d’une demande de support, reportez-vous à [Créer une demande de support Azure](https://docs.microsoft.com/en-us/azure/azure-portal/supportability/how-to-create-azure-support-request).

### Exercice 1 : Membre d’un domaine AD DS

Les principales tâches de cet exercice sont les suivantes

1. Préparer un déploiement de machine virtuelle Azure
1. Déployer une machine virtuelle Azure exécutant un contrôleur de domaine AD DS à l’aide d’un modèle de démarrage rapide Azure Resource Manager
1. Déployer une machine virtuelle Azure exécutant Windows 10 à l’aide d’un modèle de démarrage rapide Azure Resource Manager
1. Déployer Azure Bastion

#### Tâche 1 : Préparer un déploiement de machine virtuelle Azure

1. À partir de votre ordinateur labo, démarrez un navigateur web, accédez au [portail Azure](https://portal.azure.com), puis connectez-vous en fournissant des informations d’identification d’un compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utiliserez dans ce labo.
1. Dans le portail Azure, utilisez la zone de texte **Rechercher dans les ressources, services et documents** en haut de la page pour rechercher et accéder au panneau **Microsoft Entra ID**.
1. Dans le panneau **Vue d’ensemble** du locataire Microsoft Entra, dans le menu vertical à gauche, dans la section **Gérer**, cliquez sur **Propriétés**.
1. En bas du panneau **Propriétés** de votre locataire Microsoft Entra, sélectionnez le lien **Gérer la sécurité par défaut** .
1. Dans le panneau **Activer les paramètres de sécurité par défaut**, si nécessaire, sélectionnez **Désactivé (non recommandé)**, sélectionnez la case d’option **Mon organisation envisage d’utiliser l’accès conditionnel**, sélectionnez **Enregistrer**, puis **Désactiver**.
1. Dans le portail Azure, ouvrez le volet **Cloud Shell** en sélectionnant l’icône de barre d’outils juste à droite de la zone de texte de recherche.
1. Lorsque vous êtes invité à sélectionner **Bash** ou **PowerShell**, sélectionnez **PowerShell**. 

   >**Remarque** : si c’est la première fois que vous démarrez **Cloud Shell** et que vous voyez le message **Vous n’avez aucun stockage monté**, sélectionnez l’abonnement que vous utilisez dans ce labo, puis sélectionnez **Créer un stockage**. 


#### Tâche 2 : Déployer une machine virtuelle Azure exécutant un contrôleur de domaine AD DS à l’aide d’un modèle de démarrage rapide Azure Resource Manager

1. Sur l’ordinateur labo, dans le navigateur web affichant le Portail Azure, à partir de la session PowerShell dans le volet Cloud Shell, exécutez la commande suivante pour créer un groupe de ressources (remplacez l’espace réservé `<Azure_region>` par le nom de la région Azure que vous envisagez d’utiliser pour ce labo, par exemple `eastus`) :

   ```powershell
   $location = '<Azure_region>'
   $resourceGroupName = 'az140-11-RG'
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   ```

1. Dans le portail Azure, fermez le panneau **Cloud Shell**.
1. À partir de votre ordinateur labo, dans la même fenêtre de navigateur web, ouvrez un autre onglet de navigateur web et accédez à une version personnalisée du modèle de démarrage rapide nommé [Créer une machine virtuelle Windows et créer une forêt AD, un domaine et un contrôleur de domaine](https://github.com/az140mp/azure-quickstart-templates/tree/master/application-workloads/active-directory/active-directory-new-domain). 
1. Dans la page **Créer une machine virtuelle Windows et créer une nouvelle forêt AD, un domaine et un contrôleur de domaine**, faites défiler la page et sélectionnez **Déployer sur Azure**. Cela va rediriger automatiquement le navigateur vers le volet **Créer une machine virtuelle Azure avec une nouvelle forêt AD** dans le portail Azure.
1. Dans le volet **Créer une machine virtuelle Azure avec une nouvelle forêt Active Directory**, sélectionnez **Modifier les paramètres**.
1. Dans le panneau **Modifier les paramètres**, sélectionnez **Charger le fichier**, dans la boîte de dialogue **Ouvrir**, sélectionnez**\\\\ AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploydc11.parameters.json**, sélectionnez **Ouvrir**, puis **Enregistrer.** 
1. Dans le volet **Créer une machine virtuelle Azure avec une nouvelle forêt AD**, spécifiez les paramètres suivants (laissez les autres avec leurs valeurs existantes) :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|le nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Resource group|**az140-11-RG**|
   |Nom de domaine|**adatum.com**|

1. Dans le volet **Créer une machine virtuelle Azure avec une nouvelle forêt AD**, cliquez sur **Vérifier + créer**, puis sur **Créer**.

   > **Remarque** : Attendez que le déploiement se termine avant de passer à l’exercice suivant. Le déploiement peut prendre environ 20 à 25 minutes. 

#### Tâche 3 : Déployer une machine virtuelle Azure exécutant Windows 10 à l’aide d’un modèle de démarrage rapide Azure Resource Manager

1. Sur l’ordinateur labo, dans le navigateur web affichant le Portail Azure, ouvrez une session PowerShell dans le volet Cloud Shell et la commande suivante pour ajouter un sous-réseau nommé **cl-Subnet** au réseau virtuel nommé **az140-adds-vnet11** que vous avez créé dans la tâche précédente :

   ```powershell
   $resourceGroupName = 'az140-11-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-adds-vnet11'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'cl-Subnet' `
     -AddressPrefix 10.0.255.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Dans le portail Azure, dans la barre d’outils du volet Cloud Shell, sélectionnez l’icône **Charger/télécharger des fichiers**, puis dans le menu déroulant, sélectionnez **Charger**, puis chargez les fichiers **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_ azuredeploycl11.json** et **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.parameters.json** dans le répertoire de base Cloud Shell.
1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez la commande suivante pour déployer une machine virtuelle Azure exécutant Windows 10 qui servira de client dans le sous-réseau nouvellement créé :

   ```powershell
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0101vmDeployment `
     -TemplateFile $HOME/az140-11_azuredeploycl11.json `
     -TemplateParameterFile $HOME/az140-11_azuredeploycl11.parameters.json
   ```

   > **Remarque** : N’attendez pas que le déploiement se termine, mais passez à la tâche suivante. Ce déploiement peut prendre environ 10 minutes.

#### Tâche 4 : Déployer Azure Bastion 

> **Remarque** : Azure Bastion autorise la connexion aux machines virtuelles Azure sans les points de terminaison publics que vous avez déployés dans la tâche précédente de cet exercice, tout en fournissant une protection contre les attaques par force brute qui ciblent les informations d’identification au niveau du système d’exploitation.

> **Remarque** : Vérifiez que votre navigateur dispose de la fonctionnalité de fenêtre contextuelle activée.

1. Dans la fenêtre du navigateur affichant le Portail Azure, ouvrez un autre onglet, et, dans l’onglet du navigateur, accédez au [Portail Azure](https://portal.azure.com).
1. Dans le Portail Azure, ouvrez le volet **Cloud Shell** en sélectionnant l’icône de barre d’outils directement à droite de la zone de texte de recherche.
1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez la commande suivante pour ajouter un sous-réseau nommé **AzureBastionSubnet** au réseau virtuel nommé **az140-adds-vnet11** créé précédemment dans cet exercice :

   ```powershell
   $resourceGroupName = 'az140-11-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-adds-vnet11'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.0.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Fermez le volet Cloud Shell.
1. Dans le portail Azure, recherchez et sélectionnez **Bastions**, puis dans le panneau **Bastions**, sélectionnez **+ Créer**.
1. Sous l’onglet **De base** du panneau **Créer un bastion**, spécifiez les paramètres suivants et sélectionnez **Vérifier + créer** :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|le nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Resource group|**az140-11-RG**|
   |Nom|**az140-11-bastion**|
   |Région|la même région Azure dans laquelle vous avez déployé les ressources dans les tâches précédentes de cet exercice|
   |Niveau|**De base**|
   |Réseau virtuel|**az140-adds-vnet11**|
   |Sous-réseau|**AzureBastionSubnet (10.0.254.0/24)**|
   |Adresse IP publique|**Création**|
   |Nom IP publique|**az140-adds-vnet11-ip**|

1. Sous l’onglet **Vérifier + créer** du panneau **Créer un bastion**, sélectionnez **Créer** :

   > **Remarque** : Attendez que le déploiement se termine avant de passer à l’exercice suivant. Le déploiement peut prendre environ 10 minutes.

### Exercice 2 : Intégrer une forêt AD DS avec un locataire Microsoft Entra
  
Les principales tâches de cet exercice sont les suivantes

1. Créer des utilisateurs et des groupes AD DS qui seront synchronisés avec Microsoft Entra
1. Configurer le suffixe UPN AD DS
1. Créer un utilisateur Microsoft Entra qui sera utilisé pour configurer la synchronisation avec Microsoft Entra
1. Installer Microsoft Entra Connect

#### Tâche 1 : Créer des utilisateurs et des groupes AD DS qui seront synchronisés avec Microsoft Entra

1. Sur l’ordinateur labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **machines virtuelles** et, dans le panneau **Machines virtuelles**, sélectionnez **az140-dc-vm11**.
1. Dans le panneau **az140-dc-vm11**, sélectionnez **Se connecter**, et dans le menu déroulant, sélectionnez**Se connecter via Bastion**.
1. Lorsque vous y êtes invité, fournissez les informations d’identification suivantes et sélectionnez **Se connecter** :

   |Paramètre|Valeur|
   |---|---|
   |Nom d’utilisateur|**Étudiant**|
   |Mot de passe|**Pa55w.rd1234**|

1. Dans la session Bastion pour **az140-dc-vm11**, démarrez **Windows PowerShell ISE** en tant qu’administrateur.
1. Depuis la console **Administrateur : Volet de script Windows PowerShell ISE **, exécutez la commande suivante pour désactiver la sécurité renforcée d’Internet Explorer pour les administrateurs :

   ```powershell
   $adminRegEntry = 'HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A7-37EF-4b3f-8CFC-4F3A74704073}'
   Set-ItemProperty -Path $AdminRegEntry -Name 'IsInstalled' -Value 0
   Stop-Process -Name Explorer
   ```

1. Depuis la console **Administrateur : Console Windows PowerShell ISE**, exécutez la commande suivante pour créer une unité d’organisation AD DS qui contiendra des objets inclus dans l’étendue de la synchronisation avec le locataire Microsoft Entra utilisé dans ce labo :

   ```powershell
   New-ADOrganizationalUnit 'ToSync' -path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. Depuis la console **Administrateur : Console Windows PowerShell ISE**, exécutez la commande suivante pour créer une unité d’organisation AD DS qui hébergera les objets ordinateur de ordinateurs clients joints à un domaine Windows 10 :

   ```powershell
   New-ADOrganizationalUnit 'WVDClients' -path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. Depuis la console **Administrateur : Volet de script Windows PowerShell ISE**, exécutez la commande suivante pour créer des comptes d’utilisateurs AD DS qui seront synchronisés avec le locataire Microsoft Entra utilisé dans ce labo (remplacez les deux espaces réservés `<password>` avec des mots de passe aléatoires et complexes) :

   > **Remarque** : Assurez-vous de noter les mots de passe utilisés. Vous en aurez besoin plus loin dans ce labo et dans les labos suivants.

   ```powershell
   $ouName = 'ToSync'
   $ouPath = "OU=$ouName,DC=adatum,DC=com"
   $adUserNamePrefix = 'aduser'
   $adUPNSuffix = 'adatum.com'
   $userCount = 1..9
   foreach ($counter in $userCount) {
     New-AdUser -Name $adUserNamePrefix$counter -Path $ouPath -Enabled $True `
       -ChangePasswordAtLogon $false -userPrincipalName $adUserNamePrefix$counter@$adUPNSuffix `
       -AccountPassword (ConvertTo-SecureString '<password>' -AsPlainText -Force) -passThru
   } 

   $adUserNamePrefix = 'wvdadmin1'
   $adUPNSuffix = 'adatum.com'
   New-AdUser -Name $adUserNamePrefix -Path $ouPath -Enabled $True `
       -ChangePasswordAtLogon $false -userPrincipalName $adUserNamePrefix@$adUPNSuffix `
       -AccountPassword (ConvertTo-SecureString '<password>' -AsPlainText -Force) -passThru

   Get-ADGroup -Identity 'Domain Admins' | Add-AdGroupMember -Members 'wvdadmin1'
   ```

   > **Remarque** : Le script crée neuf comptes d’utilisateurs non privilégiés nommés **aduser1** - **aduser9** et un compte privilégié membre du **groupe d’Administration de\\ domaine ADATUM**nommé **wvdadmin1.**

1. Depuis la console **Administrateur : Volet de script Windows PowerShell ISE**, exécutez la commande suivante pour créer des objets de groupe AD DS qui seront synchronisés avec le locataire Microsoft Entra utilisé dans ce labo :

   ```powershell
   New-ADGroup -Name 'az140-wvd-pooled' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-remote-app' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-personal' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-users' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-admins' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   ```

1. Depuis la console **Administrateur : Console Windows PowerShell ISE**, exécutez la commande suivante pour ajouter des membres aux groupes que vous avez créés à l’étape précédente :

   ```powershell
   Get-ADGroup -Identity 'az140-wvd-pooled' | Add-AdGroupMember -Members 'aduser1','aduser2','aduser3','aduser4'
   Get-ADGroup -Identity 'az140-wvd-remote-app' | Add-AdGroupMember -Members 'aduser1','aduser5','aduser6'
   Get-ADGroup -Identity 'az140-wvd-personal' | Add-AdGroupMember -Members 'aduser7','aduser8','aduser9'
   Get-ADGroup -Identity 'az140-wvd-users' | Add-AdGroupMember -Members 'aduser1','aduser2','aduser3','aduser4','aduser5','aduser6','aduser7','aduser8','aduser9'
   Get-ADGroup -Identity 'az140-wvd-admins' | Add-AdGroupMember -Members 'wvdadmin1'
   ```

#### Tâche 2 : Configurer le suffixe UPN AD DS

1. Dans la session Bastion vers **az140-dc-vm11**, à partir de la console **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour installer la dernière version du module PowerShellGet (sélectionnez **Oui** lorsque vous êtes invité à confirmer) :

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. Depuis la console **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour installer la dernière version du module Az PowerShell (sélectionnez **Oui pour tout** lorsque vous êtes invité à confirmer) :

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

   > **Remarque** : Vous devrez peut-être attendre trois à cinq minutes avant qu’une sortie de l’installation du module Az s’affiche. Vous devrez peut-être également attendre cinq minutes **après** l’arrêt de la sortie. Ce comportement est normal.

1. Depuis la console **Administrateur : Console Windows PowerShell ISE** , exécutez la commande suivante pour vous connecter à votre abonnement Azure :

   ```powershell
   Connect-AzAccount
   ```

1. Lorsque vous y êtes invité, fournissez les informations d’identification du compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utilisez dans ce labo.
1. Depuis la console **Administrateur : Console Windows PowerShell ISE**, exécutez la commande suivante pour récupérer la propriété ID du locataire Microsoft Entra associé à votre abonnement Azure :

   ```powershell
   $tenantId = (Get-AzContext).Tenant.Id
   ```

1. Depuis la console **Administrateur : Console Windows PowerShell ISE**, exécutez la commande suivante pour installer et importer la dernière version du module PowerShell Azure AD :

   ```powershell
   Install-Module -Name AzureAD -Force
   Import-Module -Name AzureAD
   ```

1. Depuis la console **Administrateur : Console Windows PowerShell ISE**, exécutez la commande suivante pour vous authentifier auprès de votre locataire Microsoft Entra :

   ```powershell
   Connect-AzureAD -TenantId $tenantId
   ```

1. Lorsque vous y êtes invité, connectez-vous avec les mêmes informations d’identification que celles que vous avez utilisées précédemment dans cette tâche (le compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utilisez dans ce labo). 
1. Depuis la console **Administrateur : Console Windows PowerShell ISE**, exécutez la commande suivante pour récupérer le nom de domaine du DNS principal du locataire Microsoft Entra associé à votre abonnement Azure :

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. Depuis la console **Administrateur : Console Windows PowerShell ISE**, exécutez la commande suivante pour ajouter le nom de domaine du DNS principal du locataire Microsoft Entra associé à votre abonnement Azure à la liste de suffixes UPN de votre forêt AD DS :

   ```powershell
   Get-ADForest|Set-ADForest -UPNSuffixes @{add="$aadDomainName"}
   ```

1. Depuis la console **Administrateur : Volet de script Windows PowerShell ISE**, exécutez la commande suivante pour assigner le nom de domaine du DNS principal du locataire Microsoft Entra associé à votre abonnement Azure comme suffixe UPN pour tous les utilisateurs du domaine AD DS :

   ```powershell
   $domainUsers = Get-ADUser -Filter {UserPrincipalName -like '*adatum.com'} -Properties userPrincipalName -ResultSetSize $null
   $domainUsers | foreach {$newUpn = $_.UserPrincipalName.Replace('adatum.com',$aadDomainName); $_ | Set-ADUser -UserPrincipalName $newUpn}
   ```

1. Depuis la console **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour réaffecter le suffixe UPN **adatum.com** à l’utilisateur du domaine **Student** :

   ```powershell
   $domainAdminUser = Get-ADUser -Filter {sAMAccountName -eq 'Student'} -Properties userPrincipalName
   $domainAdminUser | Set-ADUser -UserPrincipalName 'student@adatum.com'
   ```

#### Tâche 3 : Créer un utilisateur Microsoft Entra qui sera utilisé pour configurer la synchronisation de répertoires

1. Dans la session Bastion vers **az140-dc-vm11**, à partir de la console **Administrateur : Volet de script Windows PowerShell ISE**, exécutez la commande suivante pour créer un nouvel utilisateur Microsoft Entra (remplacez l’espace réservé `<password>` par un mot de passe aléatoire et complexe) :

   > **Remarque** : Assurez-vous de noter le mot de passe utilisé. **Vous en aurez besoin plus loin dans ce labo et dans les labos suivants**.

   ```powershell
   $userName = 'aadsyncuser'
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName $userName -PasswordProfile $passwordProfile -MailNickName $userName -UserPrincipalName "$userName@$aadDomainName"
   ```

1. Depuis la console **Administrateur : Volet de script Windows PowerShell ISE**, exécutez la commande suivante pour attribuer le rôle Administrateur général à l’utilisateur Microsoft Entra nouvellement créé : 

   ```powershell
   $aadUser = Get-AzureADUser -ObjectId "$userName@$aadDomainName"
   $aadRole = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Global administrator'} 
   Add-AzureADDirectoryRoleMember -ObjectId $aadRole.ObjectId -RefObjectId $aadUser.ObjectId
   ```

1. Depuis la console **Administrateur : Volet de script Windows PowerShell ISE**, exécutez la commande suivante pour identifier le nom d’utilisateur principal de l’utilisateur Microsoft Entra nouvellement créé :

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq '$userName'").UserPrincipalName
   ```

   > **Remarque** : Prenez note du nom d’utilisateur principal **et** du mot de passe. Vous en aurez besoin plus loin dans cet exercice. 


#### Tâche 4 : Installer Microsoft Entra Connect

1. Dans la session Bastion vers **az140-dc-vm11**, à partir de la console **Administrateur : Volet de script Windows PowerShell ISE**, exécutez la commande suivante pour activer le TLS 1.2 :

   ```powershell
   New-Item 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -name 'SystemDefaultTlsVersions' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -name 'SchUseStrongCrypto' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -name 'SystemDefaultTlsVersions' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -name 'SchUseStrongCrypto' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -name 'Enabled' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -name 'DisabledByDefault' -value 0 -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -name 'Enabled' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -name 'DisabledByDefault' -value 0 -PropertyType 'DWord' -Force | Out-Null
   Write-Host 'TLS 1.2 has been enabled.'
   ```
   
1. Dans la session Bastion à **az140-dc-vm11**, lancez Internet Explorer et accédez à la [page de téléchargement de Microsoft Edge for Business](https://www.microsoft.com/en-us/edge/business/download).
1. À partir de la [page de téléchargement de Microsoft Edge for Business](https://www.microsoft.com/en-us/edge/business/download), téléchargez la dernière version stable de Microsoft Edge, installez-la, lancez-la et configurez-la avec les paramètres par défaut.
1. Dans la session Bureau à distance de **az140-dc-vm11**, utilisez Microsoft Edge pour accéder au [portail Azure](https://portal.azure.com). Si vous y êtes invité, connectez-vous à l’aide des informations d’identification Microsoft Entra du compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utilisez dans ce labo.
1. Dans le portail Azure, utilisez la zone de texte **Rechercher des ressources, des services et des documents** en haut de la page pour rechercher et accéder au panneau **Microsoft Entra ID** et, dans le panneau de votre locataire Microsoft Entra, dans la section **Gérer** du menu hub, sélectionnez **Microsoft Entra Connect**.
1. Dans le panneau **Microsoft Entra Connect**, sélectionnez le lien **Connect Sync** dans le menu de service, puis sélectionnez le lien **Télécharger Microsoft Entra Connect**. Cela ouvre automatiquement un nouvel onglet de navigateur affichant la page de téléchargement de **Microsoft Entra Connect**.
1. Dans la page de téléchargement de **Microsoft Entra Connect**, sélectionnez **Télécharger**.
1. Si vous êtes invité à exécuter ou enregistrer le programme d’installation de **AzureADConnect.msi** , sélectionnez **Exécuter**. Sinon, ouvrez le fichier une fois téléchargé pour démarrer l’assistant **Microsoft Azure Active Directory Connect.**
1. Dans la page **Bienvenue dans Azure AD Connect** de l’assistant **Microsoft Azure Active Directory Connect**, cochez la case **J’accepte les termes du contrat de licence et la notification de confidentialité**, puis sélectionnez **Continuer**.
1. Dans la page **Configuration rapide** de l’assistant **Microsoft Azure Active Directory Connect**, cliquez sur l’option **Personnaliser**.
1. Sur la page **Installer les composants requis**, laissez toutes les options de configuration facultatives décochées, puis sélectionnez **Installer**.
1. Dans la page **de connexion utilisateur**, vérifiez que seule la **synchronisation de hachage du mot de passe** est activée, puis cliquez sur **Suivant**.
1. Dans la page **Connexion à Azure AD**, authentifiez-vous à l’aide des informations d’identification du compte d’utilisateur **aadsyncuser** que vous avez créé dans l’exercice précédent, puis sélectionnez **Suivant**. 

   > **Remarque** : Indiquez l’attribut userPrincipalName du compte **aadsyncuser** que vous avez enregistré précédemment dans cet exercice et spécifiez le mot de passe que vous avez défini précédemment dans ce labo.

1. Dans la page **Connexion de vos répertoires**, cliquez sur le bouton **Ajouter un répertoire** à droite de l’entrée de forêt **adatum.com**.
1. Dans la fenêtre du **compte de forêt AD**, vérifiez que l’option **Créer un compte AD** est sélectionnée, spécifiez les informations d’identification suivantes, puis cliquez sur **OK** :

   |Paramètre|Valeur|
   |---|---|
   |Nom d’utilisateur|**ADATUM\Student**|
   |Mot de passe|**Pa55w.rd1234**|

1. De retour sur la page **Connexion de vos répertoires**, vérifiez que l’entrée **adatum.com** s’affiche sous la forme d’un répertoire configuré, puis cliquez sur **Suivant**
1. Dans la page **Configuration de la connexion à Azure AD**, notez l’avertissement indiquant que **Les utilisateurs ne pourront pas se connecter à Azure AD à l’aide d’informations d’identification locales si le suffixe UPN ne correspond pas à un domaine vérifié.**, cochez la case **Continuer sans faire correspondre tous les suffixes UPN à des domaines vérifiés**, puis sélectionnez **Suivant**.

   > **Remarque** : Ceci est attendu, étant donné que le locataire Microsoft Entra n’a pas de domaine DNS personnalisé vérifié correspondant à l’un des suffixes UPN du **adatum.com** AD DS.

1. Dans la page **Filtrage du domaine et de l’unité** d’organisation, sélectionnez l’option **Synchroniser les domaines sélectionnés et les unités d’organisation**, développez le nœud adatum.com, décochez toutes les cases, sélectionnez uniquement la case à côté de l’unité d’organisation **ToSync**, puis sélectionnez **Suivant**.
1. Sur la page **Identification de manière unique de vos utilisateurs**, acceptez les paramètres par défaut, puis cliquez sur **Suivant**.
1. Sur la page **Filtrer les utilisateurs et appareils**, acceptez les paramètres par défaut, puis cliquez sur **Suivant**.
1. Sur la page **Fonctionnalités facultatives**, acceptez les paramètres par défaut, puis cliquez sur **Suivant**.
1. Sur la page **Prêt à configurer**, vérifiez que la case **Démarrer le processus de synchronisation une fois la configuration terminée** est cochée, puis cliquez sur **Installer**.

   > **Remarque** : L’installation doit prendre environ cinq minutes.

1. Vérifiez les informations de la page **Configuration terminée**, puis cliquez sur **Quitter** pour fermer la fenêtre **Microsoft Azure Active Directory Connect**.
1. Dans la session Bureau à distance vers **az140-dc-vm11**, dans la fenêtre Microsoft Edge affichant le portail Azure, accédez au volet **Utilisateurs - Tous les utilisateurs** du locataire Microsoft Entra Adatum Lab.
1. Dans le panneau **Utilisateurs \| Tous les utilisateurs**, notez que la liste des objets utilisateur inclut la liste des comptes d’utilisateur AD DS que vous avez créés précédemment dans ce labo, avec l’entrée **Oui** qui apparaît dans la colonne de **synchronisation locale activée** .

   > **Remarque** : Vous devrez peut-être attendre quelques minutes et actualiser la page du navigateur pour que les comptes d’utilisateur AD DS apparaissent.
