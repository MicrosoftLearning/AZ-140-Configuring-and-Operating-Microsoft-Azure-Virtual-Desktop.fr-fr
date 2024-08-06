---
lab:
  title: "Labo : Créer et configurer des pools d’hôtes et des hôtes de session (Microsoft Entra\_DS)"
  module: 'Module 2: Implement an AVD Infrastructure'
---

# Labo - Créer et configurer des pools d’hôtes et des hôtes de session (Microsoft Entra DS)
# Manuel de labo de l’étudiant

## Dépendances du labo

- Un abonnement Azure
- Un compte Microsoft ou un compte Microsoft Entra disposant du rôle Administrateur général pour le locataire Microsoft Entra qui est associé à l’abonnement Azure, et disposant du rôle Propriétaire ou Contributeur dans l’abonnement Azure
- Le labo terminé **Préparer le déploiement d’Azure Virtual Desktop (Microsoft Entra DS)**

## Durée estimée

60 minutes

## Scénario du labo

Vous devez créer et configurer des pools d’hôtes et des hôtes de session dans un environnement Azure Active Directory Domain Services (Microsoft Entra DS).

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

- Configurer un environnement Azure Virtual Desktop dans un domaine Microsoft Entra DS. 
- Valider l’environnement Azure Virtual Desktop dans un domaine Microsoft Entra DS. 

## Fichiers du labo

- Aucun 

## Instructions

### Exercice 1 : Configurer un environnement Azure Virtual Desktop
  
Les principales tâches de cet exercice sont les suivantes

1. Préparer le domaine AD DS et l’abonnement Azure pour le déploiement d’un pool d’hôtes Azure Virtual Desktop
1. Déployer un pool d’hôtes Azure Virtual Desktop
1. Configurer des groupes d’applications Azure Virtual Desktop
1. Configurer des espaces de travail Azure Virtual Desktop

#### Tâche 1 : Préparer le domaine AD DS et l’abonnement Azure pour le déploiement d’un pool d’hôtes Azure Virtual Desktop

1. À partir de votre ordinateur de labo, démarrez un navigateur web, accédez au [portail Azure](https://portal.azure.com), puis connectez-vous en utilisant les informations d’identification d’un compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utiliserez dans ce labo.
1. À partir de votre ordinateur de labo, dans le Portail Azure, recherchez et sélectionnez **Machines virtuelles**, puis, dans le panneau **Machines virtuelles**, sélectionnez l’entrée **az140-cl-vm11a**. Cette opération ouvre le panneau **az140-cl-vm11a**.
1. Dans le panneau **az140-cl-vm11a**, sélectionnez **Se connecter**, puis dans le menu déroulant, sélectionnez **Bastion**. Sous l’onglet **Bastion** du panneau **az140-cl-vm11a\| Se connecter**, sélectionnez **Utiliser Bastion**.
1. Lorsque vous y êtes invité, fournissez les informations d’identification suivantes et sélectionnez **Connecter** :

   |Paramètre|Valeur|
   |---|---|
   |Nom d’utilisateur|**aadadmin1@adatum.com**|
   |Mot de passe|Mot de passe défini dans le labo précédent|

1. Dans Bastion sur la machine virtuelle Azure **az140-cl-vm11a**, ouvrez Microsoft Edge, accédez au [portail Azure](https://portal.azure.com) et connectez-vous en fournissant le nom d’utilisateur principal du compte d’utilisateur **aadadmin1** et le mot de passe que vous avez défini lors de la création de ce compte.

   >**Remarque** : Vous pouvez identifier l’attribut de nom d’utilisateur principal (UPN) du compte **aadadmin1** en consultant sa boîte de dialogue de propriétés à partir de la console Utilisateurs et ordinateurs Active Directory ou en revenant à votre ordinateur de labo et en consultant ses propriétés à partir du panneau Locataire Microsoft Entra dans le portail Azure.

1. Dans la session Bastion sur **az140-cl-vm11a**, dans la fenêtre Microsoft Edge affichant le portail Azure, ouvrez une session PowerShell dans **Cloud Shell** et exécutez le fournisseur de ressources **Microsoft.DesktopVirtualization** suivant :

   ```powershell
   Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
   ```

1. Dans la session Bastion sur **az140-cl-vm11a**, dans la fenêtre Microsoft Edge affichant le portail Azure, recherchez et sélectionnez **Réseaux virtuels**, puis dans le panneau **Réseaux virtuels**, sélectionnez l’entrée **az140-aadds-vnet11a**. 
1. Dans le panneau **az140-aadds-vnet11a**, sélectionnez **Sous-réseaux**. Dans le panneau **Sous-réseaux**, sélectionnez **+ Sous-réseau**. Dans le panneau **Ajouter un sous-réseau**, dans la zone de texte **Nom**, tapez **hp1-Subnet**, laissez tous les autres paramètres avec leurs valeurs par défaut, puis sélectionnez **Enregistrer**. 

#### Tâche 2 : Déployer un pool d’hôtes Azure Virtual Desktop

1. Dans la session Bastion sur **az140-cl-vm11a**, dans la fenêtre Microsoft Edge affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop**. Dans le panneau **Azure Virtual Desktop**, dans le menu vertical situé à gauche, dans la section **Gérer**, sélectionnez **Pools d’hôtes**, puis dans le panneau **Azure Virtual Desktop \| Pool d’hôtes**, sélectionnez **+ Créer**. 
1. Sous l’onglet **Informations de base** du panneau **Créer un pool d’hôtes**, spécifiez les paramètres suivants et sélectionnez **Suivant : Machines virtuelles >** :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|le nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Resource group|nom d’un nouveau groupe de ressources **az140-21a-RG**|
   |Nom du pool d’hôtes|**az140-21a-hp1**|
   |Emplacement|nom de la région Azure dans laquelle vous avez déployé l’instance Microsoft Entra DS précédemment dans ce labo|
   |Environnement de validation|**Aucun**|
   |Type de groupe d’applications préféré|**Bureau**|
   |Type de pool d’hôtes|**Groupé**|
   |Limite de session maximale|**12**|
   |Algorithme d’équilibrage de charge|**À largeur prioritaire**|

   > **Remarque** : si un utilisateur a à la fois des applications RemoteApp et de bureau publiées, le type de groupe d’applications préféré détermine laquelle des deux apparaîtra dans son flux.

1. Sous l’onglet **Machines virtuelles** du panneau **Créer un pool d’hôtes**, spécifiez les paramètres suivants (laissez les autres avec leurs valeurs par défaut) et sélectionnez **Suivant : Espace de travail >** (remplacez l’espace réservé *<Azure_AD_domain_name>* par le nom du locataire Microsoft Entra associé à l’abonnement dans lequel vous avez déployé l’instance Microsoft Entra DS et remplacez l’espace réservé `<password>` par le mot de passe que vous avez défini lors de la création du compte aadadmin1) :

   > **Remarque** : Veillez à ne pas oublier le mot de passe que vous avez utilisé. Vous en aurez besoin plus loin dans ce labo et dans les suivants.

   |Paramètre|Valeur|
   |---|---|
   |Ajouter des machines virtuelles|**Oui**|
   |Resource group|**Par défaut, identique à celui du pool d’hôtes**|
   |Préfixe de nom|**az140-21-p1**|
   |Emplacement des machines virtuelles|nom de la région Azure dans laquelle vous avez déployé des ressources dans le premier exercice de ce labo|
   |Options de disponibilité|**Aucune redondance de l’infrastructure requise**|
   |Type de sécurité|**Lancement fiable des machines virtuelles**|
   |Image|**Windows 11 Entreprise multisession + Microsoft 365 Apps, version 22H2**|
   |Taille de la machine virtuelle|**Standard D2s v3**|
   |Nombre d'ordinateurs virtuels|**2**|
   |Type de disque du système d’exploitation|**SSD Standard**|
   |Réseau virtuel|**az140-aadds-vnet11a**|
   |Sous-réseau|**hp1-Subnet (10.10.1.0/24)**|
   |Groupe de sécurité réseau|**De base**|
   |Ports d’entrée publics|**Aucun**|
   |Sélectionnez le répertoire que vous souhaitez rejoindre|**Active Directory**|
   |UPN de jonction de domaine AD|**aadadmin1@adatum.com**|
   |Mot de passe|Utiliser le mot de passe pour aadadmin1|
   |Spécifier un domaine ou une unité|**Oui**|
   | Domaine à rejoindre|**adatum.com**|
   |Chemin de l’unité d’organisation|**OU=AADDC Computers,DC=adatum,DC=com**|
   |Nom d’utilisateur du compte Administrateur de machine virtuelle|**student**|
   |Mot de passe du compte Administrateur de machine virtuelle|**Pa55w.rd1234**|

1. Sous l’onglet **Espace de travail** du panneau **Créer un pool d’hôtes**, spécifiez les paramètres suivants et sélectionnez **Vérifier + créer** :

   |Paramètre|Valeur|
   |---|---|
   |Inscrire un groupe d'applications de bureau|**Aucun**|

1. Sous l’onglet **Vérifier + créer** du panneau **Créer un pool d’hôtes**, sélectionnez **Créer**.

   > **Remarque** : Attendez la fin du déploiement. Cette opération prend environ 15 minutes.

#### Tâche 3 : Configurer des groupes d’applications Azure Virtual Desktop

1. Dans la session Bastion sur **az140-cl-vm11a** dans le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop**, puis dans le panneau **Azure Virtual Desktop**, sélectionnez **Groupes d’applications**.
1. Dans le panneau **Azure Virtual Desktop \| Groupes d’application**, sélectionnez le groupe d’applications de bureau **az140-21a-hp1-DAG** généré automatiquement.
1. Dans le panneau **az140-21a-hp1-DAG**, dans le menu vertical situé à gauche, dans la section **Gérer**, sélectionnez **Affectations**.
1. Dans le panneau **az140-21a-hp1-DAG \| Affectations**, sélectionnez **+ Ajouter**.
1. Dans le panneau **Sélectionner des utilisateurs ou des groupes d’utilisateurs Microsoft Entra**, sélectionnez **az140-wvd-apooled**, puis cliquez sur **Sélectionner**.
1. Revenez au panneau **Azure Virtual Desktop \| Groupes d’applications** et sélectionnez **+ Créer**.
1. Sous l’onglet **Informations de base** du panneau **Créer un groupe d’applications**, spécifiez les paramètres suivants et sélectionnez **Suivant : Applications >** :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|le nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Resource group|**az140-21a-RG**|
   |Pool d’hôtes|**az140-21a-hp1**|
   |Type de groupe d’applications|**RemoteApp**|
   |Nom du groupe d'applications|**az140-21a-hp1-Office365-RAG**|

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
1. Dans le panneau **Sélectionner des utilisateurs ou des groupes d’utilisateurs Microsoft Entra**, sélectionnez **az140-wvd-aremote-app**, puis cliquez sur **Sélectionner**.
1. De retour sous l’onglet **Affectations** du panneau **Créer un groupe d’applications**, sélectionnez **Suivant : Espace de travail >**.
1. Sous l’onglet **Espace de travail** du panneau **Créer un espace de travail**, spécifiez le paramètre suivant et sélectionnez **Vérifier + créer** :

   |Paramètre|Valeur|
   |---|---|
   |Inscrire le groupe d'applications|**Aucun**|

1. Sous l’onglet **Vérifier + créer** du panneau **Créer un groupe d’applications**, sélectionnez **Créer**.

   > **Remarque** : Maintenant, vous allez créer un groupe d’applications en fonction du chemin du fichier comme source d’application.

1. Dans la session Bastion sur **az140-cl-vm11a**, dans la fenêtre du navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop**, puis dans le panneau ** Azure Virtual Desktop**, sélectionnez **Groupes d’applications**.
1. Dans le panneau **Azure Virtual Desktop \| Groupes d’applications**, sélectionnez **+ Créer**. 
1. Sous l’onglet **Informations de base** du panneau **Créer un groupe d’applications**, spécifiez les paramètres suivants et sélectionnez **Suivant : Applications >** :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|le nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Resource group|**az140-21a-RG**|
   |Pool d’hôtes|**az140-21a-hp1**|
   |Type de groupe d’applications|**RemoteApp**|
   |Nom du groupe d'applications|**az140-21a-hp1-Utilities-RAG**|

1. Sous l’onglet **Applications** du panneau **Créer un groupe d’applications**, sélectionnez **+ Ajouter des applications**.
1. Dans le panneau **Ajouter une application**, spécifiez les paramètres suivants et sélectionnez **Enregistrer** :

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
1. Dans le panneau **Sélectionner des utilisateurs ou des groupes d’utilisateurs Microsoft Entra**, sélectionnez **az140-wvd-aremote-app** et **az140-wvd-aadmins**, puis cliquez sur **Sélectionner**.
1. De retour sous l’onglet **Affectations** du panneau **Créer un groupe d’applications**, sélectionnez **Suivant : Espace de travail >**.
1. Sous l’onglet **Espace de travail** du panneau **Créer un espace de travail**, spécifiez le paramètre suivant et sélectionnez **Vérifier + créer** :

   |Paramètre|Valeur|
   |---|---|
   |Inscrire le groupe d'applications|**Aucun**|

1. Sous l’onglet **Vérifier + créer** du panneau **Créer un groupe d’applications**, sélectionnez **Créer**.

#### Tâche 4 : Configurer des espaces de travail Azure Virtual Desktop

1. Dans la session Bastion sur **az140-cl-vm11a**, dans la fenêtre Microsoft Edge affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop**, puis dans le panneau **Azure Virtual Desktop**, sélectionnez **Espaces de travail**.
1. Dans le panneau **Azure Virtual Desktop \| Espaces de travail**, sélectionnez **+ Créer**. 
1. Sous l’onglet **Informations de base** du panneau **Créer un espace de travail**, spécifiez les paramètres suivants et sélectionnez **Suivant : Groupes d’applications >** :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|le nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Resource group|**az140-21a-RG**|
   |Nom de l’espace de travail|**az140-21a-ws1**|
   |Nom convivial|**az140-21a-ws1**|
   |Emplacement|nom de la région Azure dans laquelle vous avez déployé les ressources au cours de ce labo||

1. Sous l’onglet **Groupes d’applications** du panneau **Créer un espace de travail**, spécifiez les paramètres suivants :

   |Paramètre|Valeur|
   |---|---|
   |Inscrire des groupes d’applications|**Oui**|

1. Sous l’onglet **Espace de travail** du panneau **Créer un espace de travail**, sélectionnez **+ Inscrire des groupes d’applications**.
1. Dans le panneau **Ajouter des groupes d’applications**, sélectionnez le signe plus à côté des entrées **az140-21a-hp1-DAG**, **az140-21a-hp1-Office365-RAG** et **az140-21a-hp1-Utilities-RAG**, puis cliquez sur **Sélectionner**. 
1. De retour sous l’onglet **Groupes d’applications** du panneau **Créer un espace de travail**, sélectionnez **Vérifier + créer**.
1. Sous l’onglet **Vérifier + créer** du panneau **Créer un espace de travail**, sélectionnez **Créer**.

### Exercice 2 : Valider l’environnement Azure Virtual Desktop
  
Les principales tâches de cet exercice sont les suivantes

1. Installer le client Bureau à distance Microsoft sur un ordinateur Windows 10
1. S’abonner à un espace de travail Azure Virtual Desktop
1. Tester les applications Azure Virtual Desktop

#### Tâche 1 : Installer le client Bureau à distance Microsoft sur un ordinateur Windows 10

1. Dans la session Bastion sur **az140-cl-vm11a**, démarrez Microsoft Edge et accédez à la [page de téléchargement du client Windows Desktop](https://go.microsoft.com/fwlink/?linkid=2068602), puis lorsque vous y êtes invité, effectuez son installation en suivant les invites. Sélectionnez l’option **Installer pour tous les utilisateurs sur cet ordinateur**. 
1. Une fois l’installation terminée, démarrez le client Bureau à distance.

#### Tâche 2 : S’abonner à un espace de travail Azure Virtual Desktop

1. Dans la fenêtre du client **Bureau à distance**, sélectionnez **S’abonner**, puis lorsque vous y êtes invité, connectez-vous avec les informations d’identification **aaduser1** (avec son attribut userPrincipalName comme nom d’utilisateur et mot de passe que vous avez définis lors de la création de ce compte). 

   > **Remarque** : Sinon, dans la fenêtre du client **Bureau à distance**, sélectionnez **S’abonner avec l’URL**. Dans le volet **S’abonner à un espace de travail**, dans **E-mail ou URL d’espace de travail**, tapez **https://rdweb.wvd.microsoft.com/api/arm/feeddiscovery**, sélectionnez **Suivant**, puis une fois que vous y êtes invité, connectez-vous avec les informations d’identification **aaduser1** (avec son attribut userPrincipalName comme nom d’utilisateur et **Pa55w.rd1234** comme mot de passe). 

   > **Remarque** : Le nom d’utilisateur principal **aaduser1** doit être au format **aaduser1@***<Azure_AD_domain_name>*, où l’espace réservé *<Azure_AD_domain_name>* correspond au nom du locataire Microsoft Entra associé à l’abonnement dans lequel vous avez déployé l’instance Microsoft Entra DS.

1. Dans la fenêtre **Rester connecté à toutes vos applications**, décochez la case **Autoriser mon organisation à gérer mon appareil** et sélectionnez **Non, se connecter à cette application uniquement**. 
1. Vérifiez que la page **Bureau à distance** affiche la session SessionDesktop incluse dans le groupe d’applications az140-21-hp1-DAG généré automatiquement et publié dans l’espace de travail et associé au compte d’utilisateur **aduser1** via son appartenance au groupe. 

   > **Remarque** : ce comportement est normal, car le **type de groupe d’applications préféré** du pool d’hôtes est actuellement défini sur **Bureau**.

#### Tâche 3 : Tester des applications Azure Virtual Desktop

1. Dans la session Bastion sur **az140-cl-vm11a**, dans la fenêtre du client **Bureau à distance**, dans la liste des applications, double-cliquez sur **SessionDesktop** et assurez-vous qu’il ouvre une session Bureau à distance. 

   > **Remarque** : Au départ, le démarrage de l’application peut prendre quelques minutes, mais par la suite, le démarrage de l’application doit être beaucoup plus rapide.

   > **Remarque** : Si vous recevez l’invite de connexion **Bienvenue dans Microsoft Teams**, refermez-la. 

1. Dans la session **Session de Bureau**, cliquez avec le bouton droit sur **Démarrer** et sélectionnez **Exécuter**. Dans la zone de texte **Ouvrir** de la boîte de dialogue **Exécuter**, tapez **cmd** et sélectionnez **OK**. 
1. Dans la session **Bureau par défaut**, à l’invite de commandes, tapez **hostname** et appuyez sur la touche **Entrée** pour afficher le nom de l’ordinateur sur lequel la session Bureau à distance s’exécute.
1. Vérifiez que le nom affiché est **az140-21-p1-0**, **az140-21-p1-1** ou **az140-21-p1-2**.
1. À l’invite de commandes, tapez **logoff**, puis appuyez sur la touche **Entrée** pour vous déconnecter de la session de Bureau.

   > **Remarque** : ensuite, vous allez modifier le **type de groupe d’applications préféré** en le définissant sur **RemoteApp**.

1. Dans la session Bastion sur **az140-dc-vm11**, dans la fenêtre du navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop** et, dans le panneau **Azure Virtual Desktop**, dans la barre de menus verticale, dans la **section Gérer**, sélectionnez **Pools d’hôtes**.
1. Dans le panneau **Azure Virtual Desktop \| Pools d’hôtes**, dans la liste des pools d’hôtes, sélectionnez **az140-21-hp1**.
1. Dans le panneau **az140-21-hp1**, dans la barre de menus verticale, dans la **section Paramètres**, sélectionnez **Propriétés**. Dans le **type de groupe d’applications préféré**, sélectionnez **Application distante**, puis **Enregistrer**. 
1. Dans la session Bastion sur **az140-cl-vm11**, dans la fenêtre du client **Bureau à distance**, sélectionnez le symbole de points de suspension dans le coin supérieur droit et, dans le menu déroulant, sélectionnez **Actualiser**.
1. Vérifiez que la page **Bureau à distance** affiche les applications individuelles incluses dans les deux groupes d’applications que vous avez créés et publiés dans l’espace de travail, qui sont également associes au compte d’utilisateur **aduser1** via son appartenance au groupe. 

   > **Remarque** : ce comportement est normal, car le **type de groupe d’applications préféré** du pool d’hôtes est désormais défini sur **RemoteApp**.

1. Dans la session Bastion sur **az140-cl-vm11a**, dans la fenêtre du client **Bureau à distance**, dans la liste des applications, double-cliquez sur **Invite de commandes** et vérifiez qu’elle lance une fenêtre d’**invite de commandes**. Lorsque vous êtes invité à s’authentifier, tapez le mot de passe que vous définissez lors de la création du compte d’utilisateur **aduser1**, cochez la case **Me rappeler**, puis sélectionnez **OK**.
1. À l’invite de commandes, tapez **logoff**, puis appuyez sur l'**Entrée** touche pour vous déconnecter de la session d’application distante actuelle.

### Exercice 3 : Arrêter et libérer les machines virtuelles Azure approvisionnées et utilisées dans le labo

Les principales tâches de cet exercice sont les suivantes

1. Arrêter et libérer les machines virtuelles Azure approvisionnées et utilisées dans le labo

>**Remarque** : Dans cet exercice, vous allez libérer les machines virtuelles Azure approvisionnées et utilisées dans ce labo pour réduire les frais de calcul correspondants

#### Tâche 1 : Libérer des machines virtuelles Azure approvisionnées et utilisées dans le labo

1. Basculez vers l’ordinateur labo et, dans la fenêtre du navigateur web affichant le Portail Azure, ouvrez la session de l’interpréteur de commandes **PowerShell** dans le volet **Cloud Shell**.
1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez la commande suivante pour répertorier toutes les machines virtuelles Azure créées et utilisées dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21a-RG'
   ```

1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez ce qui suit pour arrêter et libérer toutes les machines virtuelles Azure que vous avez créées et utilisées dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21a-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Remarque** : La commande s’exécute de manière asynchrone (telle que déterminée par le paramètre -NoWait). Par conséquent, même si vous pourrez exécuter une autre commande PowerShell immédiatement après la même session PowerShell, il faudra quelques minutes avant que les machines virtuelles Azure ne soient réellement arrêtées et libérées.

