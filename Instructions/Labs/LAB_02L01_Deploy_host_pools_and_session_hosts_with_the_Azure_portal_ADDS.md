---
lab:
  title: 'Labo : Déployer des pools d’hôtes et des hôtes de session à l’aide du portail Azure (AD DS)'
  module: 'Module 2: Implement an AVD Infrastructure'
---

# Labo - Déployer des pools d’hôtes et des hôtes de session à l’aide du portail Azure (AD DS)
# Manuel de labo de l’étudiant

## Dépendances du labo

- Un abonnement Azure que vous allez utiliser dans ce labo.
- Un compte Microsoft ou Microsoft Entra avec le rôle Propriétaire ou Contributeur dans l’abonnement Azure que vous allez utiliser dans ce labo et avec le rôle Administrateur général dans le locataire Microsoft Entra associé à cet abonnement Azure.
- Le labo terminé **Préparer le déploiement d’Azure Virtual Desktop (AD DS)**

## Durée estimée

60 minutes

## Scénario du labo

Vous devez créer et configurer des pools d’hôtes et des hôtes de session dans un environnement Active Directory Domain Services (AD DS).

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

- Implémenter un environnement Azure Virtual Desktop dans un domaine AD DS
- Valider un environnement Azure Virtual Desktop dans un domaine AD DS

## Fichiers du labo

- Aucun

## Instructions

### Exercice 1 : Implémenter un environnement Azure Virtual Desktop dans un domaine AD DS
  
Les principales tâches de cet exercice sont les suivantes

1. Préparer le domaine AD DS et l’abonnement Azure pour le déploiement d’un pool d’hôtes Azure Virtual Desktop
1. Déployer un pool d’hôtes Azure Virtual Desktop
1. Gérer les hôtes de session du pool d’hôtes Azure Virtual Desktop
1. Configurer des groupes d’applications Azure Virtual Desktop
1. Configurer des espaces de travail Azure Virtual Desktop

#### Tâche 1 : Préparer le domaine AD DS et l’abonnement Azure pour le déploiement d’un pool d’hôtes Azure Virtual Desktop

1. À partir de votre ordinateur labo, démarrez un navigateur web, accédez au [portail Azure]( ), puis connectez-vous en fournissant des informations d’identification d’un compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utiliserez dans ce labo.
1. Dans le Portail Azure, recherchez et sélectionnez **machines virtuelles** et, dans le panneau **Machines virtuelles**, sélectionnez **az140-dc-vm11**.
1. Dans le panneau **az140-dc-vm11**, sélectionnez **Connecter**, dans le menu déroulant, sélectionnez **Bastion**, sous l’onglet **Bastion** du panneau **az140-dc-vm11 \| Connecter**, sélectionnez **Utiliser Bastion**.
1. Lorsque vous y êtes invité, fournissez les informations d’identification suivantes et sélectionnez **Connecter** :

   |Paramètre|Valeur|
   |---|---|
   |Nom d’utilisateur|**Étudiant**|
   |Mot de passe|**Pa55w.rd1234**|

1. Dans la session Bureau à distance pour **az140-dc-vm11**, démarrez **Windows PowerShell ISE** en tant qu’administrateur.
1. Dans la session Bureau à distance pour **az140-dc-vm11**, à partir de l’administrateur **: Windows PowerShell ISE** console, exécutez la commande suivante pour créer une unité d’organisation qui hébergera les objets ordinateur des hôtes Azure Virtual Desktop :

   ```powershell
   New-ADOrganizationalUnit 'WVDInfra' –path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. Depuis la console **Administrateur : Console Windows PowerShell ISE** , exécutez la commande suivante pour vous connecter à votre abonnement Azure :

   ```powershell
   Connect-AzAccount
   ```

1. Lorsque vous y êtes invité, fournissez les informations d’identification du compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utilisez dans ce labo.
1. Depuis la console **Administrateur : Windows PowerShell ISE** console, exécutez la commande suivante pour identifier le nom d’utilisateur principal du compte **aduser1** :

   ```powershell
   (Get-AzADUser -DisplayName 'aduser1').UserPrincipalName
   ```

   > **Remarque** : Enregistrez le nom d’utilisateur principal que vous avez identifié à cette étape. Vous en aurez besoin plus tard dans ce labo.

1. Depuis la console **Administrateur : Console Windows PowerShell ISE**, exécutez ce qui suit pour inscrire le fournisseur de ressources **Microsoft.DesktopVirtualization** :

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
   ```

1. Dans la session Bureau à distance pour **az140-dc-vm11**, démarrez Microsoft Edge et accédez au [portail Azure](https://portal.azure.com). Si vous y êtes invité, connectez-vous à l’aide des informations d’identification Microsoft Entra du compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utilisez dans ce labo.
1. Dans la session Bureau à distance pour **az140-dc-vm11**, dans le portail Azure, utilisez la zone de texte **Rechercher des ressources, des services et des documents** en haut de la page du portail Azure pour rechercher et accéder aux **réseaux virtuels** et, dans le panneau **réseaux virtuels**, sélectionnez **az140-adds-vnet11**. 
1. Dans le panneau **az140-add-vnet11**, sélectionnez **Sous-réseaux**, dans le panneau **Sous-réseaux** , sélectionnez **+ Sous-réseau**, dans le panneau **Ajouter un sous-réseau**, spécifiez les paramètres suivants (laissez tous les autres paramètres avec leurs valeurs par défaut) et cliquez sur **Enregistrer**:

   |Paramètre|Valeur|
   |---|---|
   |Nom|**hp1-Subnet**|
   |Plage d’adresses de sous-réseau|**10.0.1.0/24**|

#### Tâche 2 : Déployer un pool d’hôtes Azure Virtual Desktop

1. Dans la session Bureau à distance pour **az140-dc-vm11**, dans la fenêtre Microsoft Edge affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop**, dans le panneau **Azure Virtual Desktop**, sélectionnez **pools d’hôtes** et, dans le panneau ** Azure Virtual Desktop \|pools d’hôtes**, sélectionnez **+ Créer**. 
1. Sous l’onglet **Informations de base** du panneau **Créer un pool d’hôtes** , spécifiez les paramètres suivants et sélectionnez **Suivant : Machines virtuelles >** (laissez d’autres paramètres avec leurs valeurs par défaut) :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|le nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Resource group|nom d’un nouveau groupe de ressources **az140-21-RG**|
   |Nom du pool d’hôtes|**az140-21-hp1**|
   |Emplacement|nom de la région Azure dans laquelle vous avez déployé des ressources dans le premier exercice de ce labo ou d’une région proche de celle-ci |
   |Environnement de validation|**Aucun**|
   |Type de groupe d’applications préféré|**Bureau**|
   |Type de pool d’hôtes|**Groupé**|
   |Algorithme d’équilibrage de charge|**À largeur prioritaire**|
   |Limite de session maximale|**12**|

1. Sous l’onglet **Machines virtuelles** du panneau **Créer un pool d’hôtes**, spécifiez les paramètres suivants et sélectionnez **Suivant : Espace de travail >** (laissez d’autres paramètres avec leurs valeurs par défaut) :

   |Paramètre|Valeur|
   |---|---|
   |Ajouter des machines virtuelles Azure|**Oui**|
   |Resource group|**Par défaut, identique à celui du pool d’hôtes**|
   |Préfixe de nom|**az140-21-p1**|
   |Emplacement des machines virtuelles|nom de la région Azure dans laquelle vous avez déployé des ressources dans le premier exercice de ce labo|
   |Options de disponibilité|**Aucune redondance de l’infrastructure requise**|
   |Type de sécurité|**Standard**|
   |Image|**Windows 11 Entreprise multisession + Microsoft 365 Apps, version 22H2**|
   |Taille de la machine virtuelle|**Standard D2s v3**|
   |Nombre d'ordinateurs virtuels|**2**|
   |Type de disque du système d’exploitation|**SSD Standard**|
   |Diagnostics de démarrage|**Activer avec le compte de stockage managé (recommandé)**|
   |Réseau virtuel|**az140-adds-vnet11**|
   |Sous-réseau|**hp1-Subnet (10.0.1.0/24)**|
   |Groupe de sécurité réseau|**De base**|
   |Ports d’entrée publics|**Aucun**|
   |Sélectionnez le répertoire que vous souhaitez rejoindre|**Active Directory**|
   |UPN de jonction de domaine AD|**student@adatum.com**|
   |Mot de passe|**Pa55w.rd1234**|
   |Spécifier un domaine ou une unité|**Oui**|
   | Domaine à rejoindre|**adatum.com**|
   |Chemin de l’unité d’organisation|**OU=WVDInfra,DC=adatum,DC=com**|
   |Nom d'utilisateur|**Étudiant**|
   |Mot de passe|**Pa55w.rd1234**|
   |Confirmer le mot de passe|**Pa55w.rd1234**|

1. Sous l’onglet **Espace de travail** du panneau **Créer un pool d’hôtes**, spécifiez les paramètres suivants et sélectionnez **Vérifier + créer** :

   |Paramètre|Valeur|
   |---|---|
   |Inscrire un groupe d'applications de bureau|**Aucun**|

1. Sous l’onglet **Vérifier + créer** du panneau **Créer un pool d’hôtes**, sélectionnez **Créer**.

   > **Remarque** : Attendez la fin du déploiement. Ceci peut prendre environ 10 minutes.

#### Tâche 3 : Gérer les hôtes de session du pool d’hôtes Azure Virtual Desktop

1. Dans la session Bureau à distance pour **az140-dc-vm11**, dans la fenêtre du navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop** et, dans le panneau **Azure Virtual Desktop**, dans la barre de menus verticale, dans la **section Gérer**, sélectionnez **pools d’hôtes**.
1. Dans le panneau **Azure Virtual Desktop \| Pools d’hôtes**, dans la liste des pools d’hôtes, sélectionnez **az140-21-hp1**.
1. Dans le **panneau az140-21-hp1**, dans la barre de menus verticale, dans la **section Gérer**, sélectionnez **Hôtes de session** et vérifiez que le pool se compose de deux hôtes. 
1. Dans le panneau **az140-21-hp1 \| Session hôtes**, sélectionnez **+ Ajouter**.
1. Sous l’onglet **Informations de base** du panneau **Ajouter des machines virtuelles à un pool d’hôtes**, passez en revue les paramètres préconfigurés et sélectionnez **Suivant : Machines virtuelles**.
1. Sous l’onglet **Machines virtuelles** du panneau **Ajouter des machines virtuelles à un pool d’hôtes**, spécifiez les paramètres suivants et sélectionnez **Vérifier + créer** (laissez d’autres utilisateurs avec leurs paramètres par défaut) :

   |Paramètre|Valeur|
   |---|---|
   |Groupe de ressources|**az140-21-RG**|
   |Préfixe de nom|**az140-21-p1**|
   |Emplacement des machines virtuelles|nom de la région Azure dans laquelle vous avez déployé les deux premières machines virtuelles hôtes de session|
   |Options de disponibilité|**Aucune redondance de l’infrastructure requise**|
   |Type de sécurité|**Standard**|
   |Image|**Windows 11 Entreprise multisession + Microsoft 365 Apps, version 22H2**|
   |Nombre d'ordinateurs virtuels|**1**|
   |Type de disque du système d’exploitation|**SSD Standard**|
   |Diagnostics de démarrage|**Activer avec le compte de stockage managé (recommandé)**|
   |Réseau virtuel|**az140-adds-vnet11**|
   |Sous-réseau|**hp1-Subnet (10.0.1.0/24)**|
   |Groupe de sécurité réseau|**De base**|
   |Ports d’entrée publics|**Aucun**|
   |UPN de jonction de domaine AD|**student@adatum.com**|
   |Mot de passe|**Pa55w.rd1234**|
   |Spécifier un domaine ou une unité|**Oui**|
   | Domaine à rejoindre|**adatum.com**|
   |Chemin de l’unité d’organisation|**OU=WVDInfra,DC=adatum,DC=com**|   
   |Nom d’utilisateur du compte Administrateur de machine virtuelle|**Étudiant**|
   |Mot clé du compte Administrateur de machine virtuelle|**Pa55w.rd1234**|

   > **Remarque** : Comme vous l’avez probablement remarqué, il est possible de modifier l’image et le préfixe des machines virtuelles au fur et à mesure que vous ajoutez des hôtes de session au pool existant. En général, cela n’est pas recommandé, sauf si vous envisagez de remplacer toutes les machines virtuelles du pool. 

1. Dans l’onglet **Vérifier + créer** du panneau **Ajouter des machines virtuelles à un pool d’hôtes** panneau, sélectionnez **Créer**

   > **Remarque** : Attendez que le déploiement se termine avant de passer à la tâche suivante. Le déploiement peut prendre environ 5 minutes. 

#### Tâche 4 : Configurer des groupes d’applications Azure Virtual Desktop

1. Dans la session Bureau à distance pour **az140-dc-vm11**, dans la fenêtre du navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop** et, dans le panneau ** Azure Virtual Desktop**, sélectionnez **groupes d’applications**.
1. Dans le panneau **groupes d’applications\| Azure Virtual Desktop**, notez le groupe d’applications de bureau généré automatiquement **az140-21-hp1-DAG** groupe d’applications de bureau, puis sélectionnez-le. 
1. Dans le panneau **az140-21-hp1-DAG**, sélectionnez **Affectations**.
1. Dans le panneau **az140-21-hp1-DAG \| Affectations**, sélectionnez **+ Ajouter**.
1. Dans le panneau **Sélectionner des utilisateurs ou des groupes d’utilisateurs Microsoft Entra**, sélectionnez **az140-wvd-pooled**, puis cliquez sur **Sélectionner**.
1. Revenez au panneau **Azure Virtual Desktop \| Groupes d’applications**, sélectionnez **+ Créer**. 
1. Sous l’onglet **Informations de base** du panneau **Créer un groupe d’applications**, spécifiez les paramètres suivants et sélectionnez **Suivant : Applications >**:

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|le nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Resource group|**az140-21-RG**|
   |Pool d’hôtes|**az140-21-hp1**|
   |Type de groupe d’applications|**Remote App (RAIL)**|
   |Nom du groupe d'applications|**az140-21-hp1-Office365-RAG**|

1. Sous l’onglet **Applications** du panneau **Créer un groupe d’applications**, sélectionnez **+ Ajouter des applications**.
1. Dans le panneau **Ajouter une application**, spécifiez les paramètres suivants et sélectionnez **Enregistrer** :

   |Paramètre|Valeur|
   |---|---|
   |Source de l’application|**Menu Démarrer**|
   |Application|**Word**|
   |Description|**Microsoft Word**|
   |Demander la ligne de commande|**Aucun**|

1. De retour sous l’onglet **Applications** du panneau **Créer un groupe d’applications**, sélectionnez **+ Ajouter des applications**.
1. Dans le panneau **Ajouter une application**, spécifiez les paramètres suivants et sélectionnez **Enregistrer** :

   |Paramètre|Valeur|
   |---|---|
   |Source de l’application|**Menu Démarrer**|
   |Application|**Excel**|
   |Description|**Microsoft Excel**|
   |Demander la ligne de commande|**Aucun**|

1. De retour sous l’onglet **Applications** du panneau **Créer un groupe d’applications**, sélectionnez **+ Ajouter des applications**.
1. Dans le panneau **Ajouter une application**, spécifiez les paramètres suivants et sélectionnez **Enregistrer** :

   |Paramètre|Valeur|
   |---|---|
   |Source de l’application|**Menu Démarrer**|
   |Application|**PowerPoint**|
   |Description|**Microsoft PowerPoint**|
   |Demander la ligne de commande|**Aucun**|

1. De retour sous l’onglet **Applications** du panneau **Créer un groupe d’applications**, sélectionnez **Suivant : Affectations > **.
1. Sous l’onglet **Affectations** du panneau **Créer un groupe d’applications**, sélectionnez **+ Ajouter des utilisateurs ou des groupes d’utilisateurs Microsoft Entra**.
1. Dans le panneau **Sélectionner des utilisateurs ou des groupes d’utilisateurs Microsoft Entra**, sélectionnez **az140-wvd-remote-app**, puis cliquez sur **Sélectionner**.
1. De retour sous l’onglet **Affectations** du panneau **Créer un groupe d’applications**, sélectionnez **Suivant : Espace de travail >**.
1. Sous l’onglet **Espace de travail** du panneau **Créer un espace de travail**, spécifiez le paramètre suivant et sélectionnez **Vérifier + créer** :

   |Paramètre|Valeur|
   |---|---|
   |Inscrire le groupe d'applications|**Aucun**|

1. Sous l’onglet **Vérifier + créer** du panneau **Créer un groupe d’applications**, sélectionnez **Créer**.

   > **Remarque** : Attendez que le groupe d’applications soit créé. Cela devrait prendre moins d’une minute. 

   > **Remarque** : Ensuite, vous allez créer un groupe d’applications en fonction du chemin d’accès au fichier en tant que source de l’application.

1. Dans la session Bureau à distance pour **az140-dc-vm11**, recherchez et sélectionnez **Azure Virtual Desktop** et, dans le panneau **Azure Virtual Desktop**, sélectionnez **groupes d’applications**.
1. Dans le panneau **groupes d’applications \| Azure Virtual Desktop**, sélectionnez **+ Créer**. 
1. Sous l’onglet **Informations de base** du panneau **Créer un groupe d’applications**, spécifiez les paramètres suivants et sélectionnez **Suivant : Applications >** :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|le nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Resource group|**az140-21-RG**|
   |Pool d’hôtes|**az140-21-hp1**|
   |Type de groupe d’applications|**RemoteApp (RAIL)**|
   |Nom du groupe d'applications|**az140-21-hp1-Utilities-RAG**|

1. Sous l’onglet **Applications** du panneau **Créer un groupe d’applications**, sélectionnez **+ Ajouter des applications**.
1. Dans le panneau **Ajouter une application**, utilisez les onglets **Informations de base** et **Index** pour spécifier les paramètres suivants, puis sélectionnez **Enregistrer** :

   |Paramètre|Valeur|
   |---|---|
   |Source de l’application|**Chemin de fichier**|
   |Chemin d’application|**C:\Windows\system32\cmd.exe**|
   |Nom de l’application|**Invite de commandes**|
   |Nom d’affichage|**Invite de commandes**|
   |Chemin d’accès à l’icône|**C:\Windows\system32\cmd.exe**|
   |Index d’icône|**0**|
   |Description|**Invite de commandes Windows**|
   |Demander la ligne de commande|**Aucun**|

1. De retour sous l’onglet **Applications** du panneau **Créer un groupe d’applications**, sélectionnez **Suivant : Affectations > **.
1. Sous l’onglet **Affectations** du panneau **Créer un groupe d’applications**, sélectionnez **+ Ajouter des utilisateurs ou des groupes d’utilisateurs Microsoft Entra**.
1. Dans le panneau **Sélectionner des utilisateurs ou des groupes d’utilisateurs Microsoft Entra**, sélectionnez **az140-wvd-remote-app** et **az140-wvd-admins**, puis cliquez sur **Sélectionner**.
1. De retour sous l’onglet **Affectations** du panneau **Créer un groupe d’applications**, sélectionnez **Suivant : Espace de travail >**.
1. Sous l’onglet **Espace de travail** du panneau **Créer un espace de travail**, spécifiez le paramètre suivant et sélectionnez **Vérifier + créer** :

   |Paramètre|Valeur|
   |---|---|
   |Inscrire le groupe d'applications|**Aucun**|

1. Sous l’onglet **Vérifier + créer** du panneau **Créer un groupe d’applications**, sélectionnez **Créer**.

#### Tâche 5 : Configurer des espaces de travail Azure Virtual Desktop

1. Dans la session Bureau à distance pour **az140-dc-vm11**, dans la fenêtre Microsoft Edge affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop** et, dans le panneau **Azure Virtual Desktop**, sélectionnez **Espaces de travail**.
1. Dans le panneau **Espaces de travail \| Azure Virtual Desktop**, sélectionnez **+ Créer**. 
1. Sous l’onglet **Informations de base** du panneau **Créer un espace de travail**, spécifiez les paramètres suivants et sélectionnez **Suivant : Groupes d’applications >** :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|le nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Resource group|**az140-21-RG**|
   |Nom de l’espace de travail|**az140-21-ws1**|
   |Nom convivial|**az140-21-ws1**|
   |Emplacement|nom de la région Azure dans laquelle vous avez déployé des ressources dans le premier exercice de ce labo ou d’une région proche de celle-ci|

1. Sous l’onglet **Groupes d’applications** du panneau **Créer un espace de travail**, spécifiez les paramètres suivants :

   |Paramètre|Valeur|
   |---|---|
   |Inscrire des groupes d’applications|**Oui**|

1. Sous l’onglet **Espace de travail** du panneau **Créer un espace de travail**, sélectionnez **+ Inscrire des groupes d’applications**.
1. Dans le panneau **Ajouter des groupes d’applications**, Sélectionnez le signe plus en regard du **az140-21-hp1-DAG**, **az140-21-hp1-Office365-RAG**, et des entrées **az140-21-hp1-Utilities-RAG**, puis cliquez sur **Sélectionner**. 
1. De retour dans l’onglet **Groupes d’applications** du panneau **Créer un espace de travail**, sélectionnez **Vérifier + créer**.
1. Sous l’onglet **Vérifier + créer** du panneau **Créer un espace de travail**, sélectionnez **Créer**.

### Exercice 2 : Valider l’environnement Azure Virtual Desktop
  
Les principales tâches de cet exercice sont les suivantes

1. Installer le client Bureau à distance Microsoft sur un ordinateur Windows 10
1. S’abonner à un espace de travail Azure Virtual Desktop
1. Tester les applications Azure Virtual Desktop

#### Tâche 1 : Installer le client Microsoft Remote Desktop (MSRDC) sur un ordinateur Windows 10

1. Dans la session Bureau à distance pour **az140-dc-vm11**, dans la fenêtre du navigateur affichant le portail Azure, recherchez et sélectionnez **machines virtuelles** et, dans le panneau **machines virtuelles**, sélectionnez l’entrée **az140-cl-vm11**.
1. Dans le panneau **az140-cl-vm11**, faites défiler jusqu’à la section **Opérations** et sélectionnez **Exécuter la commande**. 
1. Dans le panneau **az140-cl-vm11 \| Exécuter**, sélectionnez ** EnableRemotePS**, puis **Exécuter**. 

   > **Remarque** : Attendez que la commande se termine avant de passer à l’étape suivante. Cela peut prendre environ 1 minute. Vous pouvez obtenir des erreurs de texte rouge ciblant le profil public utilisé et non le profil de domaine, si c’est le cas, vous pouvez ignorer et passer à l’étape suivante.

1. Dans la session Bureau à distance pour **az140-dc-vm11**, à partir de l’administrateur **: Volet de script Windows PowerShell ISE**, exécutez la commande suivante pour ajouter tous les membres du groupe **ADATUM\\az140-wvd-users** au groupe **Utilisateurs bureau à distance locaux** sur la machine virtuelle Azure **az140-cl-vm11** exécutant Windows 10 que vous avez déployé dans le labo **Préparer le déploiement d’Azure Virtual Desktop (AD DS)**.

   ```powershell
   $computerName = 'az140-cl-vm11'
   Invoke-Command -ComputerName $computerName -ScriptBlock {Add-LocalGroupMember -Group 'Remote Desktop Users' -Member 'ADATUM\az140-wvd-users'}
   ```

1. Basculez vers votre ordinateur labo, à partir de l’ordinateur labo, dans la fenêtre du navigateur affichant le portail Azure, recherchez et sélectionnez **machines virtuelles** et, dans le panneau **Machines virtuelles**, sélectionnez l’entrée **az140-cl-vm11**.
1. Dans le panneau **az140-cl-vm11**, sélectionnez **Connecter**, dans le menu déroulant, sélectionnez **Bastion**, sous l’onglet **Bastion** du panneau **az140-cl-vm11\| Connecter**, sélectionnez **Utiliser bastion**.
1. Lorsque vous y êtes invité, fournissez les informations d’identification suivantes et sélectionnez **Connecter** :

   |Paramètre|Valeur|
   |---|---|
   |Nom d’utilisateur|**Student@adatum.com**|
   |Mot de passe|**Pa55w.rd1234**|

1. Dans la session Bureau à distance pour **az140-cl-vm11**, démarrez Microsoft Edge et accédez à [page de téléchargement du client Windows Desktop](https://go.microsoft.com/fwlink/?linkid=2068602) et, lorsque vous y êtes invité, sélectionnez **Exécuter** pour démarrer son installation. Dans la page **Étendue d’installation** de l’assistant **d’installation Bureau à distance**, sélectionnez l’option **Installer pour tous les utilisateurs de cet ordinateur**, puis cliquez sur **Installer**. Si vous y êtes invité par le contrôle de compte d’utilisateur pour les informations d’identification d’administration, authentifiez-vous à l’aide du nom d’utilisateur **ADATUM\\ Étudiant** avec **Pa55w.rd1234** comme mot de passe.
1. Une fois l’installation terminée, vérifiez que le **Lancer le Bureau à distance lorsque le programme d’installation** quitte case est cochée, puis cliquez sur **Terminer** pour démarrer le client Bureau à distance.

#### Tâche 2 : S’abonner à un espace de travail Azure Virtual Desktop

1. Dans la fenêtre client **Bureau à distance**, sélectionnez **s’abonner** et, lorsque vous y êtes invité, connectez-vous avec les informations d’identification **aduser1**, en fournissant son nom userPrincipalName que vous avez identifié précédemment dans ce labo et le mot de passe que vous avez défini lors de la création de ce compte.

   > **Remarque** : Sinon, dans la fenêtre client **Bureau à distance**, sélectionnez **s’abonner avec l’URL**, dans le volet **s’abonner à un espace de travail**, dans l’URL **e-mail ou espace de travail**, tapez **https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery**, sélectionnez **Suivant** et, une fois que vous y êtes invité, connectez-vous avec les informations d’identification **aduser1** (à l’aide de son attribut userPrincipalName comme nom d’utilisateur et mot de passe que vous avez définis lors de la création de ce compte). 

1. Si vous y êtes invité, dans la fenêtre **Rester connecté à toutes vos applications**, décochez la case **Autoriser mon organisation à gérer mon appareil** et sélectionnez **Non, connectez-vous à cette application uniquement**. 
1. Vérifiez que la page **Bureau à distance** affiche la liste des applications incluses dans les groupes d’applications publiés dans l’espace de travail et associées au compte d’utilisateur **aduser1** via son appartenance au groupe. 

#### Tâche 3 : Tester des applications Azure Virtual Desktop

1. Dans la session Bureau à distance pour **az140-cl-vm11**, dans la fenêtre cliente **Bureau à distance**, dans la liste des applications, double-cliquez sur **invite de commandes** et vérifiez qu’elle lance une **fenêtre d’invite de commandes**. Lorsque vous êtes invité à s’authentifier, tapez le mot de passe que vous définissez lors de la création du compte d’utilisateur **aduser1**, cochez la case **Me rappeler**, puis sélectionnez **OK**.

   > **Remarque** : Au départ, le démarrage de l’application peut prendre quelques minutes, mais par la suite, le démarrage de l’application doit être beaucoup plus rapide.

   > **Remarque** : Si vous recevez l’invite de connexion **Bienvenue dans Microsoft Teams**, refermez-la.

1. À l’invite de commandes, tapez **nom d’hôte**, puis appuyez sur la touche **Entrée** pour afficher le nom de l’ordinateur sur lequel l’invite de commandes est en cours d’exécution.

   > **Remarque** : Vérifiez que le nom affiché est **az140-21-p1-0**, **az140-21-p1-1** ou **az140-21-p1-2**, au lieu de **az140-cl-vm11**.

1. À l’invite de commandes, tapez **logoff**, puis appuyez sur l'**Entrée** touche pour vous déconnecter de la session d’application distante actuelle.
1. Dans la session Bureau à distance pour **az140-cl-vm11**, dans la fenêtre client **Bureau à distance**, dans la liste des applications, double-cliquez sur **SessionDesktop** et vérifiez qu’elle lance une session Bureau à distance. 
1. Dans la session **Bureau par défaut**, cliquez avec le bouton droit sur **Démarrer**, sélectionnez **Exécuter**, dans la zone de texte **Ouvrir** de la boîte de dialogue **Exécuter**, tapez **cmd** et sélectionnez **OK**. 
1. Dans la session **Bureau par défaut**, à l’invite de commandes, tapez **nom d’hôte**, puis appuyez sur la touche **Entrée** pour afficher le nom de l’ordinateur sur lequel la session Bureau à distance est en cours d’exécution.
1. Vérifiez que le nom affiché est **az140-21-p1-0**, **az140-21-p1-1** ou **az140-21-p1-2**.

### Exercice 3 : Arrêter et libérer des machines virtuelles Azure approvisionnées dans le labo

Les principales tâches de cet exercice sont les suivantes

1. Arrêter et libérer des machines virtuelles Azure approvisionnées dans le labo

>**Remarque** : Dans cet exercice, vous allez libérer les machines virtuelles Azure approvisionnées dans ce labo pour réduire les frais de calcul correspondants

#### Tâche 1 : Libérer des machines virtuelles Azure approvisionnées dans le labo

1. Basculez vers l’ordinateur labo et, dans la fenêtre du navigateur web affichant le Portail Azure, ouvrez la session de l’interpréteur de commandes **PowerShell** dans le volet **Cloud Shell**.
1. Dans la session PowerShell dans le volet Cloud Shell, exécutez la commande suivante pour répertorier toutes les machines virtuelles Azure créées dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez ce qui suit pour arrêter et libérer toutes les machines virtuelles Azure que vous avez créées dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Remarque** : La commande s’exécute de manière asynchrone (telle que déterminée par le paramètre -NoWait). Par conséquent, même si vous pourrez exécuter une autre commande PowerShell immédiatement après la même session PowerShell, il faudra quelques minutes avant que les machines virtuelles Azure ne soient réellement arrêtées et libérées.
