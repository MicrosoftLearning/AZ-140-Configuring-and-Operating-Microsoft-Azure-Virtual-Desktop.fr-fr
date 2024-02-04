---
lab:
  title: "Labo : Implémenter et gérer le stockage pour AVD (Microsoft Entra\_DS)"
  module: 'Module 2: Implement an AVD Infrastructure'
---

# Labo – Implémenter et gérer le stockage pour AVD (Microsoft Entra DS)
# Manuel de labo de l’étudiant

## Dépendances du labo

- Un abonnement Azure
- Un compte Microsoft ou un compte Microsoft Entra disposant du rôle Administrateur général pour le locataire Microsoft Entra qui est associé à l’abonnement Azure, et disposant du rôle Propriétaire ou Contributeur dans l’abonnement Azure
- Le labo terminé **Préparer le déploiement d’Azure Virtual Desktop (Microsoft Entra DS)**

## Durée estimée

30 minutes

## Scénario du labo

Vous devez implémenter et gérer le stockage pour un déploiement Azure Virtual Desktop dans un environnement Microsoft Entra DS.

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

- Configurer Azure Files en vue du stockage des conteneurs de profils pour Azure Virtual Desktop dans un environnement Microsoft Entra DS

## Fichiers du labo

- Aucun

## Instructions

### Exercice 1 : Configurer Azure Files en vue du stockage des conteneurs de profils pour Azure Virtual Desktop

Les principales tâches de cet exercice sont les suivantes

1. Créer un compte de stockage Azure
1. Créer un partage Azure Files
1. Activer l’authentification Microsoft Entra DS pour le compte Stockage Azure 
1. Configurer les autorisations du partage Azure Files
1. Configurer le niveau des autorisations du répertoire ou du fichier pour Azure Files

#### Tâche 1 : Créer un compte de stockage Azure

1. À partir de votre ordinateur de labo, démarrez un navigateur web, accédez au [Portail Azure](https://portal.azure.com), puis connectez-vous en fournissant des informations d’identification d’un compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utiliserez dans ce labo.
1. À partir de votre ordinateur de labo, dans le Portail Azure, recherchez et sélectionnez **Machines virtuelles**, puis, dans le panneau **Machines virtuelles**, sélectionnez l’entrée **az140-cl-vm11a**. Cette opération ouvre le panneau **az140-cl-vm11a**.
1. Dans le panneau **az140-cl-vm11a**, sélectionnez **Se connecter**, puis dans le menu déroulant, sélectionnez **Bastion**. Sous l’onglet **Bastion** du panneau **az140-cl-vm11a\| Se connecter**, sélectionnez **Utiliser Bastion**.
1. Lorsque vous y êtes invité, fournissez les informations d’identification suivantes et sélectionnez **Connecter** :

   |Paramètre|Valeur|
   |---|---|
   |Nom d’utilisateur|**aadadmin1@adatum.com**|
   |Mot de passe|Mot de passe précédemment défini|

1. Dans la session Bastion vers la machine virtuelle Azure **az140-cl-vm11a**, ouvrez Microsoft Edge, accédez au [Portail Azure](https://portal.azure.com) et connectez-vous en fournissant le nom d’utilisateur principal du compte d’utilisateur **aadadmin1** et le mot de passe défini lors de la création de ce compte.

   >**Remarque** : Vous pouvez identifier l’attribut de nom d’utilisateur principal (UPN) du compte **aadadmin1** en consultant la boîte de dialogue de ses propriétés à partir de la console Utilisateurs et ordinateurs Active Directory ou en revenant à votre ordinateur de labo et en consultant ses propriétés à partir du panneau Locataire Microsoft Entra dans le Portail Azure.

1. Dans la session Bastion vers **az140-cl-vm11a**, dans la fenêtre Microsoft Edge affichant le Portail Azure, recherchez et sélectionnez **Comptes de stockage**. Sélectionnez ensuite **+ Créer** dans le panneau **Comptes de stockage**.
1. Sous l’onglet **Options de base** du volet **Créer un compte de stockage**, spécifiez les paramètres suivants (conservez les valeurs par défaut pour les autres) :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|le nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Resource group|le nom d’un nouveau groupe de ressources **az140-22a-RG**|
   |Nom du compte de stockage|n’importe quel nom global unique d’une longueur comprise entre 3 et 15 caractères, commençant par une lettre et composé de lettres minuscules et de chiffres|
   |Emplacement|le nom d’une région Azure hébergeant l’environnement de labo Azure Virtual Desktop|
   |Performances|**Standard**|
   |Réplication|**Stockage localement redondant (LRS)**|

   >**Remarque** : Vérifiez que la longueur du nom du compte de stockage ne dépasse pas 15 caractères. Le nom va servir à créer un compte d’ordinateur dans le domaine Active Directory Domain Services (AD DS) intégré au locataire Microsoft Entra associé à l’abonnement Azure contenant le compte de stockage. Cela permet une authentification basée sur AD DS lors de l’accès aux partages de fichiers hébergés dans ce compte de stockage.

1. Sous l’onglet **Informations de base** du volet **Créer un compte de stockage**, sélectionnez **Vérifier + créer**, attendez la fin du processus de validation, puis sélectionnez **Créer**.

   >**Remarque** : attendez que le compte de stockage soit créé. Cela doit prendre environ deux minutes.

#### Tâche 2 : Créer un partage Azure Files

1. Dans la session Bastion vers **az140-cl-vm11a**, dans la fenêtre Microsoft Edge affichant le Portail Azure, revenez au panneau **Comptes de stockage** et sélectionnez l’entrée représentant le compte de stockage nouvellement créé.
1. Dans le panneau compte de stockage, dans le menu vertical situé à gauche, dans la section **Stockage de données**, sélectionnez **Partages de fichiers**, puis **+ Partage de fichiers**.
1. Dans le panneau **Nouveau partage de fichiers**, spécifiez les paramètres suivants et sélectionnez **Créer** (laissez d’autres paramètres avec leurs valeurs par défaut) :

   |Paramètre|Valeur|
   |---|---|
   |Nom|**az140-22a-profiles**|

#### Tâche 3 : Activer l’authentification Microsoft Entra DS pour le compte Stockage Azure

1. Dans la session Bastion vers **az140-cl-vm11a**, dans la fenêtre Microsoft Edge affichant le Portail Azure, dans le panneau affichant les propriétés du compte de stockage créé dans la tâche précédente, dans le menu vertical de gauche, dans la section **Stockage de données**, sélectionnez **Partages de fichiers**. 
1. Dans la section **Paramètres du partage de fichiers**, en regard de l’étiquette **Active Directory**, sélectionnez le lien **Non configuré**.
1. Dans la section **Activer une source Active Directory**, dans le rectangle intitulé **Azure Active Directory Domain Services**, sélectionnez **Configurer**.
1.  Dans le panneau **Accès basé sur l’identité**, sélectionnez l’option **Activé**, puis **Enregistrer**.

#### Tâche 4 : Configurer les autorisations basées sur le RBAC pour Azure Files

1. Dans la session Bastion vers **az140-cl-vm11a**, dans la fenêtre Microsoft Edge affichant le Portail Azure, dans le panneau affichant les propriétés du compte de stockage créé précédemment dans cet exercice, dans le menu vertical de gauche, dans la section **Stockage des données**, sélectionnez **Partages de fichiers** et l’entrée **az140-22a-profiles** dans la liste des partages.
1. Dans le panneau **az140-22a-profiles**, dans le menu vertical de gauche, sélectionnez **Access Control (IAM).**
1. Dans le panneau **az140-22a-profiles \| Access Control (IAM)**, sélectionnez **+ Ajouter** et **Ajouter une attribution de rôle** depuis le menu déroulant.
1. Dans le panneau **Ajouter une attribution de rôle**, sélectionnez **Collaborateur de partage SMB des données du fichier de stockage**, puis **Suivant** :
1. Dans le panneau **Membres**, sélectionnez **Attribuer l’accès à**, puis cliquez sur **+ Sélectionner des membres**.
1. Dans le panneau **Sélectionner des membres**, dans la zone de texte **Sélectionner**, tapez **az140-wvd-ausers** et cliquez sur **Sélectionner**.
1. Dans le panneau **Membres**, sélectionnez deux fois **Vérifier + attribuer**.
1. Répétez les étapes 3 à 8 ci-dessus, spécifiez les paramètres suivants :

   |Paramètre|Valeur|
   |---|---|
   |Role|**Contributeur élevé de partage SMB de données de fichier de stockage**|
   |Sélectionnez|**az140-wvd-aadmins**|

   > **Remarque** : Vous allez utiliser le compte d’utilisateur **aadadmin1**, membre du groupe **az140-wvd-aadmins**, pour configurer les autorisations des partages de fichiers. 

#### Tâche 5 : Configurer le niveau des autorisations du répertoire ou du fichier pour Azure Files

1. Dans la session Bastion vers **az140-cl-vm11a**, ouvrez une fenêtre d’**invite de commandes** et, à partir de la fenêtre d’**invite de commandes**, exécutez la commande suivante pour mapper un lecteur au partage cible (remplacez l’espace réservé `<storage-account-name>` par le nom du compte de stockage) :

   ```cmd
   net use Z: \\<storage-account-name>.file.core.windows.net\az140-22a-profiles
   ```

1. Dans la session Bastion vers **az140-cl-vm11a**, ouvrez Explorateur de fichiers, accédez au lecteur:Z nouvellement mappé, affichez sa boîte de dialogue **Propriétés**, sélectionnez l’onglet **Sécurité**, sélectionnez **Modifier**, puis **Ajouter** dans la boîte de dialogue **Sélectionner des utilisateurs, des ordinateurs, des comptes de service et des groupes**. Assurez-vous que la zone de texte **À partir de cet emplacement** contient l’entrée **adatum.com**. Dans la zone de texte **Entrez le nom de l’objet pour sélectionner**, tapez **az140-wvd-ausers**, puis cliquez sur **OK**.
1. De retour sous l’onglet **Sécurité** de la boîte de dialogue affichant les autorisations du lecteur mappé, assurez-vous de la sélection de l’entrée **az140-wvd-aussers**. Sélectionnez la boîte de dialogue **Modifier** dans la colonne **Autoriser** et cliquez sur **OK**. Évaluez le message affiché dans la zone de texte **Sécurité Windows** et cliquez sur **Oui**. 
1. De retour sous l’onglet **Sécurité** de la boîte de dialogue affichant les autorisations du lecteur mappé, sélectionnez **Modifier**, puis **Ajouter** dans la boîte de dialogue **Sélectionner des utilisateurs, ordinateurs, comptes de service et groupes**. Assurez-vous que la zone de texte **À partir de cet emplacement** contient l’entrée **adatum.com**. Dans la zone de texte **Entrer le nom de l’objet pour sélectionner**, tapez **az140-wvd-aadmins** et cliquez sur **OK**.
1. De retour sous l’onglet **Sécurité** de la boîte de dialogue affichant les autorisations du lecteur mappé, assurez-vous de la sélection de l’entrée **az140-wvd-aadmins**. Sélectionnez la boîte de dialogue **Contrôle total** dans la colonne **Autoriser** et cliquez sur **OK**. 
1. Sous l’onglet **Sécurité** de la boîte de dialogue affichant les autorisations du lecteur mappé, sélectionnez **Modifier**. Dans la liste des groupes et des noms d’utilisateurs, sélectionnez l’entrée **Utilisateurs authentifiés**, puis **Supprimer**.
1. Tout en restant sur l’écran Modifier, dans la liste des groupes et noms d’utilisateurs, sélectionnez l’entrée **Utilisateurs**, ensuite **Supprimer**, cliquez sur **OK** et encore deux fois sur **OK** pour terminer le processus. 

   >**Remarque** : Vous pouvez également définir des autorisations à l’aide de l’utilitaire en ligne de commande **icacls**. 
