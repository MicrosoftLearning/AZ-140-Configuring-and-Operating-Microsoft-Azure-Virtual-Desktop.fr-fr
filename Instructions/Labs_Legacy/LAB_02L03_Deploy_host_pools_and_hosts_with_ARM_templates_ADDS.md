---
lab:
  title: 'Labo : Déployer des pools d’hôtes et des hôtes en utilisant des modèles Azure Resource Manager (AD DS)'
  module: 'Module 2: Implement an AVD Infrastructure'
---

# Labo : Déployer des pools d’hôtes et des hôtes en utilisant des modèles Azure Resource Manager
# Manuel de labo de l’étudiant

## Dépendances du labo

- Un abonnement Azure que vous allez utiliser dans ce labo.
- Un compte Microsoft ou Microsoft Entra avec le rôle Propriétaire ou Contributeur dans l’abonnement Azure que vous allez utiliser dans ce labo et avec le rôle Administrateur général dans le locataire Microsoft Entra associé à cet abonnement Azure.
- Le labo terminé **Préparer le déploiement d’Azure Virtual Desktop (AD DS)**
- Le labo terminé **Déployer des pools d’hôtes et des hôtes de session à l’aide du Portail Azure (AD DS)**

## Durée estimée

45 minutes

## Scénario du labo

Vous devez automatiser le déploiement de pools d’hôtes et d’hôtes Azure Virtual Desktop au moyen de modèles Azure Resource Manager.

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

- Déployer des pools d’hôtes et des hôtes Azure Virtual Desktop en utilisant des modèles Azure Resource Manager

## Fichiers du labo

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuredeployhp23.parameters.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuremodifyhp23.parameters.json

## Instructions

### Exercice 1 : Déployer des pools d’hôtes et des hôtes Azure Virtual Desktop en utilisant des modèles Azure Resource Manager
  
Les principales tâches de cet exercice sont les suivantes

1. Préparer le déploiement du pool d’hôtes Azure Virtual Desktop en utilisant un modèle Azure Resource Manager
1. Déployer un pool d’hôtes et des hôtes Azure Virtual Desktop en utilisant un modèle Azure Resource Manager
1. Vérifier le déploiement du pool d’hôtes et des hôtes Azure Virtual Desktop
1. Préparer l’ajout des hôtes au pool d’hôtes Azure Virtual Desktop existant en utilisant un modèle Azure Resource Manager
1. Ajouter des hôtes au pool d’hôtes Azure Virtual Desktop existant en utilisant un modèle Azure Resource Manager
1. Vérifier les modifications apportées au pool d’hôtes Azure Virtual Desktop
1. Gérer les affectations de bureau personnel dans le pool d’hôtes Azure Virtual Desktop

#### Tâche 1 : Préparer le déploiement du pool d’hôtes Azure Virtual Desktop en utilisant un modèle Azure Resource Manager

1. À partir de votre ordinateur de labo, démarrez un navigateur web, accédez au [Portail Azure](https://portal.azure.com), puis connectez-vous en fournissant des informations d’identification d’un compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utiliserez dans ce labo.
1. Dans le Portail Azure, recherchez et sélectionnez **machines virtuelles** et, dans le panneau **Machines virtuelles**, sélectionnez **az140-dc-vm11**.
1. Dans le panneau **az140-dc-vm11**, sélectionnez **Se connecter**, et dans le menu déroulant, sélectionnez **Se connecter via Bastion**.
1. Lorsque vous y êtes invité, fournissez les informations d’identification suivantes et sélectionnez **Connecter** :

   |Paramètre|Valeur|
   |---|---|
   |Nom d’utilisateur|**Étudiant**|
   |Mot de passe|**Pa55w.rd1234**|

1. Dans la session Bastion pour **az140-dc-vm11**, démarrez **Windows PowerShell ISE** en tant qu’administrateur.
1. Dans la session Bastion vers **az140-dc-vm11**, à partir de la console **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour identifier le nom unique de l’unité organisationnelle nommée **WVDInfra** qui va héberger les objets ordinateurs des hôtes du pool Azure Virtual Desktop :

   ```powershell
   (Get-ADOrganizationalUnit -Filter "Name -eq 'WVDInfra'").distinguishedName
   ```

1. Dans la session Bastion vers **az140-dc-vm11**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour identifier l’attribut de nom d’utilisateur principal du compte **ADATUM\\Étudiant** que vous allez utiliser pour joindre les hôtes Azure Virtual Desktop au domaine AD DS (**student@adatum.com**) :

   ```powershell
   (Get-ADUser -Filter "sAMAccountName -eq 'student'").userPrincipalName
   ```

1. Dans la session Bastion vers **az140-dc-vm11**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour identifier le nom d’utilisateur principal des comptes **ADATUM\\aduser7** et **ADATUM\\aduser8** que vous allez utiliser pour tester les affectations de bureau personnelles plus loin dans ce labo :

   ```powershell
   (Get-ADUser -Filter "sAMAccountName -eq 'aduser7'").userPrincipalName
   (Get-ADUser -Filter "sAMAccountName -eq 'aduser8'").userPrincipalName
   ```

   > **Remarque** : Enregistrez toutes les valeurs de nom d’utilisateur principal que vous avez identifiées **et** le nom unique de l’unité d’organisation WVDInfra. Vous en aurez besoin plus tard dans ce labo.

1. Dans la session Bastion vers **az140-dc-vm11**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour calculer le délai d’expiration du jeton nécessaire pour effectuer un déploiement basé sur un modèle :

   ```powershell
   $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```

   > **Remarque** : La valeur doit ressembler au format `2022-03-27T00:51:28.3008055Z`. Enregistrez-le, car vous allez en avoir besoin dans la prochaine tâche.

   > **Remarque** : Un jeton d’inscription est nécessaire pour autoriser un hôte à rejoindre le pool. La valeur de la date d’expiration du jeton doit être comprise entre une heure et un mois, à partir de la date et de l’heure actuelles.

1. Dans la session Bastion pour **az140-dc-vm11**, démarrez Microsoft Edge et accédez au [portail Azure](https://portal.azure.com). Si vous y êtes invité, connectez-vous à l’aide des informations d’identification Microsoft Entra du compte d’utilisateur avec le rôle Propriétaire dans l’abonnement utilisé dans ce labo.
1. Dans la session Bastion vers **az140-dc-vm11**, dans le Portail Azure, utilisez la zone de texte **Rechercher dans les ressources, services et documents** en haut de la page pour rechercher et accéder aux **Réseaux virtuels**. Dans le panneau **Réseaux virtuels**, sélectionnez ensuite **az140-adds-vnet11**. 
1. Dans le panneau **az140-add-vnet11**, sélectionnez **Sous-réseaux**, dans le panneau **Sous-réseaux**, sélectionnez **+ Sous-réseau**, dans le panneau **Ajouter un sous-réseau**, spécifiez les paramètres suivants (laissez tous les autres paramètres avec leurs valeurs par défaut) et cliquez sur **Enregistrer**:

   |Paramètre|Valeur|
   |---|---|
   |Nom|**hp2-Subnet**|
   |Plage d’adresses de sous-réseau|**10.0.2.0/24**|

1. Dans la session Bastion sur **az140-dc-vm11**, dans le portail Azure, utilisez la zone de texte **Rechercher dans les ressources, services et documents** en haut de la page pour rechercher et accéder à **Groupes de sécurité réseau** puis, dans le panneau **Groupes de sécurité réseau**, sélectionnez l’unique groupe de sécurité réseau.
1. Dans le panneau du groupe de sécurité réseau, dans le menu vertical de gauche, dans la section **Paramètres**, cliquez sur **Propriétés**.
1. Dans le panneau **Propriétés**, cliquez sur l’icône **Copier dans le Presse-papiers** à droite de la zone de texte **ID de la ressource**. 

   > **Remarque** : La valeur devrait ressembler au format `/subscriptions/de8279a3-0675-40e6-91e2-5c3728792cb5/resourceGroups/az140-11-RG/providers/Microsoft.Network/networkSecurityGroups/az140-cl-vm11-nsg`, même si l’Identifiant de l’abonnement diffère. Enregistrez-le, car vous allez en avoir besoin dans la prochaine tâche.
1. Vous devez maintenant avoir **six** valeurs enregistrées. Un nom unique, trois noms d’utilisateurs principaux, une valeur DateTime et l’ID de ressource. Si vous n’avez pas six valeurs enregistrées, relisez cette tâche **avant** de continuer. 

#### Tâche 2 : Déployer un pool d’hôtes et des hôtes Azure Virtual Desktop en utilisant un modèle Azure Resource Manager

1. À partir de votre ordinateur de labo, démarrez un navigateur web, accédez au [Portail Azure](https://portal.azure.com), puis connectez-vous en fournissant des informations d’identification d’un compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utiliserez dans ce labo.
1. À partir de votre ordinateur de labo, dans la même fenêtre de navigateur web, ouvrez un autre onglet et accédez à la page de référentiel du modèles Azure RDS GitHub [Modèle ARM pour créer et approvisionner un nouveau pool d’hôtes Azure Virtual Desktop](https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates/CreateAndProvisionHostPool). 
1. Dans le page **Modèle ARM pour créer et approvisionner un pool d’hôtes Azure Virtual Desktop**, sélectionnez **Déployer sur Azure**. Cela va rediriger automatiquement le navigateur vers le panneau **Déploiement personnalisé** dans le Portail Azure.
1. Dans le panneau **Déploiement personnalisé**, sélectionnez **Modifier les paramètres**.
1. Dans le panneau **Modifier les paramètres**, sélectionnez **Charger le fichier**. Dans la boîte de dialogue **Ouvrir**, sélectionnez **\\\\AZ-140\\AllFiles\\Labos\\02\\az140-23_azuredeployhp23.parameters.json**, puis **Ouvrir** et **Enregistrer**. 
1. De retour dans le panneau **Déploiement personnalisé**, spécifiez les paramètres suivants (en laissant les autres avec leurs valeurs existantes) :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|le nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Groupe de ressources|Créez un **nouveau** groupe de ressources nommé **az140-23-RG**.|
   |Région|le nom de la région Azure dans laquelle vous avez déployé des machines virtuelles Azure hébergeant des contrôleurs de domaine AD DS dans le labo **Préparer le déploiement d’Azure Virtual Desktop (AD DS)**|
   |Emplacement|le nom de la même région Azure que celle définie comme valeur pour les paramètres **Région**|
   |Emplacement de l’espace de travail|le nom de la même région Azure que celle définie comme valeur pour les paramètres **Région**|
   |Groupe de ressources de l’espace de travail|aucun, car s’il est défini comme nul, sa valeur est automatiquement définie pour correspondre au groupe de ressources cible du déploiement|
   |Référence sur tous les groupes d’applications|aucun, car il n’existe aucun groupe d’applications existant dans l’espace de travail cible (il n’existe pas d’espace de travail)|
   |Emplacement de la machine virtuelle|le nom de la même région Azure que celle définie comme valeur pour les paramètres **Emplacement**|
   |Créer un groupe de sécurité réseau|**false**|
   |ID du groupe de sécurité de réseau|la valeur du paramètre resourceID du groupe de sécurité réseau existant que vous avez identifié dans la tâche précédente|
   |Jeton de délai d’expiration| la valeur de l’heure d’expiration du jeton calculée dans la tâche précédente|

   > **Remarque** : Le déploiement provisionne un pool avec le type d’affectation de bureau personnel.

1. Dans le panneau **Déploiement personnalisé**, sélectionnez **Vérifier + créer**, puis **Créer**.

   > **Remarque** : Attendez la fin du déploiement avant de passer à la tâche suivante. Ceci peut prendre environ 15 minutes. 

#### Tâche 3 : Vérifier le déploiement du pool d’hôtes et des hôtes Azure Virtual Desktop

1. À partir de votre ordinateur de labo, dans le navigateur web affichant le Portail Azure, recherchez et sélectionnez **Azure Virtual Desktop**. Dans le panneau **Azure Virtual Desktop**, sélectionnez **Pools d’hôtes** et, dans le panneau **Azure Virtual Desktop \| Pools d’hôtes**, sélectionnez l’entrée **az140-23-hp2** qui représente le pool nouvellement déployé.
1. Dans le panneau **az140-23-hp2**, dans la section **Gérer** du menu vertical de gauche, cliquez sur **Hôtes de session**. 
1. Dans le panneau **az140-23-hp2 \| Hôtes de session**, assurez-vous que le déploiement comporte deux hôtes.
1. Dans le panneau **az140-23-hp2 \| Hôtes de session**, dans la section **Gérer** du menu vertical de gauche, cliquez sur **Groupes d’applications**.
1. Dans le panneau **az140-23-hp2 \| Groupes d'applications**, assurez-vous que le déploiement comprend le groupe d’applications du **Bureau par défaut** nommé **az140-23-hp2-DAG**.

#### Tâche 4 : Préparer l’ajout des hôtes au pool d’hôtes Azure Virtual Desktop existant en utilisant un modèle Azure Resource Manager

1. À partir de votre ordinateur de labo, basculez à la session Bastion vers **az140-dc-vm11**. 
1. Dans la session Bastion vers **az140-dc-vm11**, à partir de la console **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour générer le jeton nécessaire pour joindre de nouveaux hôtes au pool provisionné précédemment dans cet exercice :

   ```powershell
   $registrationInfo = New-AzWvdRegistrationInfo -ResourceGroupName 'az140-23-RG' -HostPoolName 'az140-23-hp2' -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```

1. Dans la session Bastion vers **az140-dc-vm11**, à partir de la console **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour récupérer la valeur du jeton et la coller dans le Presse-papiers :

   ```powershell
   $registrationInfo.Token | clip
   ```

   > **Remarque** : Enregistrez la valeur copiée dans le Presse-papiers (par exemple, en ouvrant le Bloc-notes Windows et en appuyant sur la combinaison de touches Ctrl+V pour coller le contenu du Presse-papiers dans Bloc-notes Windows) du contenu, car vous allez en avoir besoin de la prochaine tâche. Assurez-vous que la valeur utilisée inclut une seule ligne de texte, sans saut de ligne. 

   > **Remarque** : Un jeton d’inscription est nécessaire pour autoriser un hôte à rejoindre le pool. La valeur de la date d’expiration du jeton doit être comprise entre une heure et un mois, à partir de la date et de l’heure actuelles.

#### Tâche 5 : Ajouter des hôtes au pool d’hôtes Azure Virtual Desktop existant en utilisant un modèle Azure Resource Manager

1. À partir de votre ordinateur de labo, dans la même fenêtre de navigateur web, ouvrez un autre onglet et accédez à la page de référentiel du modèles Azure RDS GitHub [Modèle ARM pour ajouter des sessionhosts à un pool d’hôtes Azure Virtual Desktop existant](https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates/AddVirtualMachinesToHostPool). 
1. Dans le page **Modèle ARM pour ajouter des hôtes de session à un pool d’hôtes Azure Virtual Desktop existant**, sélectionnez **Déployer sur Azure**. Cela va rediriger automatiquement le navigateur vers le panneau **Déploiement personnalisé** dans le Portail Azure.
1. Dans le panneau **Déploiement personnalisé**, sélectionnez **Modifier les paramètres**.
1. Dans le panneau **Modifier les paramètres**, sélectionnez **Charger le fichier**. Dans la boîte de dialogue **Ouvrir**, sélectionnez **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-23_azuremodifyhp23.parameters.json**, puis **Ouvrir** et **Enregistrer**. 
1. De retour dans le panneau **Déploiement personnalisé**, spécifiez les paramètres suivants (en laissant les autres avec leurs valeurs existantes) :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|le nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Groupe de ressources|**az140-23-RG**|
   |Jeton du pool d’hôtes|la valeur du jeton généré dans la tâche précédente|
   |Emplacement du pool d’hôtes|le nom de la région Azure dans laquelle vous avez déployé le pool d’hôtes précédemment dans ce labo|
   |Emplacement de la machine virtuelle|le nom de la même région Azure que celle définie comme valeur pour les paramètres **Emplacement du pool d’hôte**|
   |Créer un groupe de sécurité réseau|**false**|
   |ID du groupe de sécurité de réseau|la valeur du paramètre resourceID du groupe de sécurité réseau existant que vous avez identifié dans la tâche précédente|

1. Dans le panneau **Déploiement personnalisé**, sélectionnez **Vérifier + créer**, puis **Créer**.

   > **Remarque** : Attendez la fin du déploiement avant de passer à la tâche suivante. Ceci peut prendre environ 10 minutes.

#### Tâche 6 : Vérifier les modifications apportées au pool d’hôtes Azure Virtual Desktop

1. À partir de votre ordinateur de labo, dans le navigateur web affichant le Portail Azure, recherchez et sélectionnez **Machines virtuelles** et, dans le panneau **Machines virtuelles**, remarquez dans la liste la présence d’une machine virtuelle supplémentaire nommée **az140-23-p2-2**.
1. À partir de votre ordinateur de labo, basculez à la session Bastion vers **az140-dc-vm11**. 
1. Dans la session Bastion vers **az140-dc-vm11**, à partir de la console **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour vous assurer que le troisième hôte a été correctement joint au domaine AD DS **adatum.com** :

   ```powershell
   Get-ADComputer -Filter "sAMAccountName -eq 'az140-23-p2-2$'"
   ```
1. Revenez à votre ordinateur de labo, puis dans le navigateur web affichant le Portail Azure, recherchez et sélectionnez **Azure Virtual Desktop**. Dans le panneau **Azure Virtual Desktop**, sélectionnez **Pools d’hôtes**. Dans le panneau **Azure Virtual Desktop \| Pools d’hôtes**, sélectionnez l’entrée **az140-23-hp2** qui représente le pool nouvellement modifié.
1. Dans le panneau **az140-23-hp2**, évaluez la section **Essentials** et assurez-vous que le **type de pool d’hôtes** est défini sur **Personnel** avec le type d’**Affectation** défini sur **Automatique**.
1. Dans le panneau **az140-23-hp2**, dans la section **Gérer** du menu vertical de gauche, cliquez sur **Hôtes de session**. 
1. Dans le panneau **az140-23-hp2 \| Hôtes de session**, assurez-vous que le déploiement comporte trois hôtes. 

#### Tâche 7 : Gérer les affectations de bureau personnel dans le pool d’hôtes Azure Virtual Desktop

1. Sur votre ordinateur de labo et dans le navigateur web affichant le Portail Azure, dans le panneau **az140-23-hp2 \| Hôtes de session**, dans la section **Gérer** du menu vertical de gauche, sélectionnez **Groupes d’applications**. 
1. Dans le panneau **az140-23-hp2 \| Groupes d’applications**, dans la liste des groupes d’applications, sélectionnez **az140-23-hp2-DAG**.
1. Dans le panneau **az140-23-hp2-DAG**, dans le menu vertical de gauche, sélectionnez **Affectations**. 
1. Dans le panneau **az140-23-hp2-DAG \| Affectations**, sélectionnez **+ Ajouter**.
1. Dans le panneau **Sélectionner des utilisateurs ou des groupes d’utilisateurs Microsoft Entra**, sélectionnez **Groupes**, puis **az140-wvd-personal**, puis cliquez sur **Sélectionner**.

   > **Remarque** : Examinons maintenant l’expérience d’un utilisateur se connectant au pool d’hôtes Azure Virtual Desktop.

1. Depuis votre ordinateur de labo, dans la fenêtre du navigateur affichant le Portail Azure, recherchez et sélectionnez **machines virtuelles** et, dans le panneau **Machines virtuelles**, sélectionnez l’entrée **az140-cl-vm11**.
1. Dans le panneau **az140-cl-vm11**, sélectionnez **Se connecter**, et dans le menu déroulant, sélectionnez**Se connecter via Bastion**.
1. Lorsque vous y êtes invité, fournissez les informations d’identification suivantes et sélectionnez **Connecter** :

   |Paramètre|Valeur|
   |---|---|
   |Nom d’utilisateur|**Student@adatum.com**|
   |Mot de passe|**Pa55w.rd1234**|

1. Dans la session Bastion vers **az140-cl-vm11**, cliquez sur **Démarrer** et, dans le menu **Démarrer**, sélectionnez l’application cliente **Bureau à distance**.
2. Dans la fenêtre Bureau à distance, cliquez sur l’icône de sélection dans le coin supérieur droit, cliquez sur **Se désabonner** dans le menu déroulant et, lorsque vous êtes invité à confirmer, cliquez sur **Continuer**.
3. Dans la session Bastion vers **az140-cl-vm11**, dans la fenêtre Bureau à distance, à la **page Prise en main**, cliquez sur **S’abonner**.
4. Dans la fenêtre du client **Bureau à distance**, sélectionnez **S’abonner** et, lorsque vous y êtes invité, connectez-vous avec les informations d’identification **aduser7**, en fournissant son userPrincipalName et le mot de passe défini lors de la création de ce compte d’utilisateur.

   > **Remarque** : Sinon, dans la fenêtre du client **Bureau à distance**, sélectionnez **S’abonner avec l’URL**. Dans le volet **S’abonner à un espace de travail**, dans l’**E-mail ou l’URL de l’espace de travail**, tapez **https://client.wvd.microsoft.com/api/arm/feeddiscovery**, sélectionnez **Suivant** et, lorsque vous y êtes invité, connectez-vous avec les informations d’identification **aduser7** (à l’aide de son attribut userPrincipalName comme le nom d’utilisateur et le mot de passe définis lors de la création de ce compte). 

1. Dans la page **Bureau à distance**, double-cliquez sur l’icône **SessionDesktop**. Lorsque vous êtes invité à entrer les informations d’identification, saisissez à nouveau le même mot de passe, sélectionnez la boîte de dialogue **Me rappeler**, puis cliquez sur **OK**.
1. Assurez-vous que **aduser7** s’est correctement connecté au moyen du Bureau à distance à un hôte.
1. Dans la session Bureau à distance vers l’un des hôtes en tant qu’**aduser7**, cliquez avec le bouton droit sur **Démarrer**. Dans le menu contextuel, sélectionnez **Arrêter ou déconnecter**, puis cliquez sur **Se déconnecter** dans le menu en cascade.

   > **Remarque** : Nous allons maintenant faire passer l’attribution de bureau personnel du mode direct au mode automatique. 

1. Basculez vers votre ordinateur de labo, vers le navigateur web affichant le Portail Azure, dans le panneau **az140-23-hp2-DAG \| Affectations**, dans la barre d’informations située directement au-dessus de la liste des affectations, cliquez sur le lien **Attribuer une machine virtuelle**. Vous êtes alors redirigé vers le panneau **az140-23-hp2 \| Hôtes de session**. 
1. Dans le panneau **az140-23-hp2 \| Hôtes de session**, vérifiez que l’un des hôtes a répertorié **aduser7** dans la colonne **Utilisateur attribué**.

   > **Remarque** : Cela est attendu, car le pool d’hôtes est configuré pour l’affectation automatique.

1. Sur votre ordinateur de labo, dans la fenêtre du navigateur web affichant le Portail Azure, ouvrez la session de l’interpréteur de commandes **PowerShell** dans le volet **Cloud Shell**.
1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez la commande suivante pour basculer vers le mode d’affectation directe :

    ```powershell
    Update-AzWvdHostPool -ResourceGroupName 'az140-23-RG' -Name 'az140-23-hp2' -PersonalDesktopAssignmentType Direct
    ```

1. Sur votre ordinateur de labo, dans la fenêtre du navigateur web affichant le Portail Azure, accédez au panneau du pool d’hôtes **az140-23-hp2**, évaluez la section **Essentials** et assurez-vous que le **type de pool d’hôtes** est défini sur **Personnel**, avec le **Type d’affectation** défini sur **Direct**.
1. Revenez à la session Bastion vers **az140-cl-vm11**, dans la fenêtre **Bureau à distance**, cliquez sur l’icône de sélection dans le coin supérieur droit, cliquez sur **Se désabonner** dans le menu déroulant et, lorsque vous êtes invité à confirmer, cliquez sur **Continuer**.
1. Dans la session Bastion vers **az140-cl-vm11**, dans la fenêtre **Bureau à distance**, à la page **Prise en main**, cliquez sur **S’abonner**.
1. Lorsque vous êtes invité à vous connecter, dans le volet **Choisir un compte**, cliquez sur **Utiliser un autre compte** et connectez-vous à l’aide du nom d’utilisateur principal du compte d’utilisateur **aduser8** (avec le mot de passe défini lors de la création de ce compte).
1. Dans la page **Bureau à distance**, double-cliquez sur l’icône **SessionDesktop**, assurez-vous de recevoir un message d’erreur indiquant **Nous n’avons pas pu nous connecter, car il n’existe actuellement aucune ressource disponible. Réessayez ultérieurement ou contactez le support technique pour obtenir de l’aide persiste**, puis cliquez sur **OK**.

   > **Remarque** : Cela est attendu, car le pool d’hôtes est configuré pour une attribution directe et **aduser8** n’a pas été affecté à un hôte.

1. Basculez vers votre ordinateur de labo, vers le navigateur web affichant le Portail Azure et, dans le panneau **az140-23-hp2 \| Hôtes de session**, sélectionnez le lien **(Attribuer)** dans la colonne **Utilisateur attribué** en regard de l’un des deux hôtes non attribués restants.
1. Sur l’option **Attribuer un utilisateur**, sélectionnez **aduser8**, cliquez sur **Sélectionner** et, lorsque vous êtes invité à confirmer, cliquez sur **OK**.
1. Revenez à la session Bastion vers **az140-cl-vm11**, dans la fenêtre du **Bureau à distance**, double-cliquez sur l’icône **SessionDesktop**. Lorsque vous êtes invité à entrer le mot de passe, tapez le mot de passe défini lors de la création de ce compte d’utilisateur, cliquez sur **OK** et assurez-vous de pouvoir vous connecter à l’hôte attribué.
1. Dans le Bureau de session vers l’hôte affecté pour **aduser8**, cliquez avec le bouton droit sur **Démarrer**. Dans le menu contextuel, sélectionnez **Arrêter ou déconnecter** puis, dans le menu en cascade, cliquez sur **Se déconnecter**.
1. Dans la session Bastion sur **az140-cl-vm11**, cliquez avec le bouton droit sur **Démarrer**, dans le menu contextuel, sélectionnez **Arrêter ou se déconnecter**, dans le menu en cascade, cliquez sur **Se déconnecter**, puis cliquez sur **Fermer**.

### Exercice 2 : Arrêter et libérer des machines virtuelles Azure approvisionnées dans le labo

Les principales tâches de cet exercice sont les suivantes

1. Arrêter et libérer des machines virtuelles Azure approvisionnées dans le labo

>**Remarque** : Dans cet exercice, vous allez libérer les machines virtuelles Azure approvisionnées dans ce labo pour réduire les frais de calcul correspondants

#### Tâche 1 : Libérer des machines virtuelles Azure approvisionnées dans le labo

1. Basculez vers l’ordinateur labo et, dans la fenêtre du navigateur web affichant le Portail Azure, ouvrez la session de l’interpréteur de commandes **PowerShell** dans le volet **Cloud Shell**.
1. Dans la session PowerShell dans le volet Cloud Shell, exécutez la commande suivante pour répertorier toutes les machines virtuelles Azure créées dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-23-RG'
   ```

1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez ce qui suit pour arrêter et libérer toutes les machines virtuelles Azure que vous avez créées dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-23-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Remarque** : La commande s’exécute de manière asynchrone (telle que déterminée par le paramètre -NoWait). Par conséquent, même si vous pourrez exécuter une autre commande PowerShell immédiatement après la même session PowerShell, il faudra quelques minutes avant que les machines virtuelles Azure ne soient réellement arrêtées et libérées.
