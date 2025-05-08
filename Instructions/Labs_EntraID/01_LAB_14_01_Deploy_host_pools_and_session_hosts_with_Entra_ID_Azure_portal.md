---
lab:
  title: "Labo\_: déployer des pools d’hôtes et des hôtes de session à l’aide du portail Azure (Entra ID)"
  module: 'Module 1.4: Implement host pools and session hosts'
---

# Labo - Déployer des pools d’hôtes et des hôtes de session à l’aide du portail Azure (AD DS)
# Manuel de labo de l’étudiant

## Dépendances du labo

- Un abonnement Azure que vous allez utiliser dans ce labo.
- Un compte d’utilisateur Microsoft Entra avec le rôle Propriétaire dans l’abonnement Azure que vous allez utiliser dans ce labo et avec les autorisations suffisantes pour joindre des appareils au locataire Entra associé à cet abonnement Azure.

## Durée estimée

60 minutes

## Scénario de labo

Vous devez avoir un abonnement à Microsoft Azure. Vous devez déployer l’environnement Azure Virtual Desktop qui utilise les hôtes de session joints à Microsoft Entra.

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

- Déployer les hôtes de session joints à Microsoft Entra dans Azure Virtual Desktop

## Fichiers du labo

- Aucun

## Instructions

### Exercice 1 : implémenter un environnement Azure Virtual Desktop à l’aide des hôtes de session joints à Microsoft Entra
  
Les principales tâches de cet exercice sont les suivantes

1. Préparer l’abonnement Azur pour le déploiement d’un pool d’hôtes Azure Virtual Desktop
1. Déployer un pool d’hôtes Azure Virtual Desktop
1. Ajouter un groupe d’applications à Azure Virtual Desktop
1. Créer un espace de travail Azure Virtual Desktop
1. Permettre d’accéder aux pools d’hôtes Azure Virtual Desktop

#### Tâche 1 : préparer l’abonnement Azure pour le déploiement d’un pool d’hôtes Azure Virtual Desktop

1. À partir de votre ordinateur de labo, démarrez un navigateur web, accédez au portail Azure à l’adresse [https://portal.azure.com](https://portal.azure.com), puis connectez-vous en fournissant des informations d’identification d’un compte d’utilisateur avec le rôle de propriétaire dans l’abonnement que vous utiliserez dans ce labo.

    > **Note** : utilisez les informations d’identification du compte `User1-` répertorié dans l’onglet Ressources sur le côté droit de la fenêtre de session de labo.

1. Dans le portail Azure, démarrez une session PowerShell dans Azure Cloud Shell.

    > **Note** : si vous y êtes invité, dans le volet **Mise en route**, dans la liste déroulante **Abonnement**, sélectionnez le nom de l’abonnement Azure que vous utilisez dans ce labo, puis sélectionnez **Appliquer**.

1. Dans la session PowerShell du volet Azure Cloud Shell, exécutez la commande suivante pour inscrire le fournisseur de ressources **Microsoft.DesktopVirtualization** :

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
    ```

    > **Note** : n’attendez pas la fin de l’inscription. Cela peut prendre quelques minutes.

1. Fermez le volet Cloud Shell.
1. Dans le portail Azure, recherchez et sélectionnez **Réseaux virtuels**, puis, dans la page **Réseaux virtuels**, sélectionnez **+ Créer**.
1. Sous l’onglet **Informations de base** du panneau **Créer un réseau virtuel**, spécifier les paramètres suivants et sélectionner **Suivant** :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|Nom d’un nouveau groupe de ressources **az140-11e-RG**|
    |Nom du réseau virtuel|**az140-vnet11e**|
    |Région|Nom de la région Azure dans laquelle vous souhaitez déployer l’environnement Azure Virtual Desktop|

1. Sous l’onglet **Sécurité**, acceptez les paramètres par défaut et sélectionnez **Suivant**.
1. Dans l’onglet **Adresses IP**, appliquez les paramètres suivants (modifiez la valeur par défaut si nécessaire) :

    |Paramètre|Valeur|
    |---|---|
    |Espace d’adressage IP|**10.20.0.0/16**|

1. Sélectionnez l’icône Modifier (crayon) en regard de l’entrée de sous-réseau **par défaut**, dans le volet **Modifier**, spécifiez les paramètres suivants (laissez les autres avec leurs valeurs existantes) et sélectionnez **Enregistrer** :

    |Paramètre|Valeur|
    |---|---|
    |Nom|**hp1-Subnet**|
    |Adresse de début|**10.20.1.0**|
    |Activer le sous-réseau privé (aucun accès sortant par défaut)|Désactivé|

1. De retour dans l’onglet **Adresses IP**, sélectionnez **Examiner et créer**, puis, dans l’onglet **Examiner et créer**, sélectionnez **Créer**.

    > **Note** : n’attendez pas la fin du processus d’approvisionnement. Cette étape prend généralement moins d’une minute.

1. Dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Microsoft Entra ID**.
1. Dans la page **Présentation** du locataire Microsoft Entra associé à votre abonnement, dans la section **Gérer** du menu de navigation vertical, sélectionnez **Utilisateurs**.
1. Dans la page **Utilisateurs**, dans la zone de texte **Rechercher**, entrez le nom du compte `User1-` répertorié dans l’onglet Ressources à droite de la fenêtre de session de labo.
1. Dans la liste des résultats de la recherche, sélectionnez l’entrée de compte d’utilisateur avec le nom correspondant.
1. Sur la page affichant les propriétés du compte d’utilisateur, dans la section **Gérer** du menu de navigation vertical, sélectionnez **Groupes**.
1. Dans la page **Groupes**, enregistrez le nom du groupe commençant par le préfixe **AVD-DAG** (vous en aurez besoin plus loin dans ce labo).
1. Revenez à la page **Utilisateurs**, dans la zone de texte **Rechercher**, entrez le nom du compte `User2-` répertorié dans l’onglet Ressources à droite de la fenêtre de session de labo.
1. Dans la liste des résultats de la recherche, sélectionnez l’entrée de compte d’utilisateur avec le nom correspondant.
1. Sur la page affichant les propriétés du compte d’utilisateur, dans la section **Gérer** du menu de navigation vertical, sélectionnez **Groupes**.
1. Dans la page **Groupes**, enregistrez le nom du groupe commençant par le préfixe **AVD-RemoteApp** (vous en aurez besoin plus loin dans ce labo).

#### Tâche 2 : Déployer un pool d’hôtes Azure Virtual Desktop

1. Sur l’ordinateur de labo, dans la fenêtre de navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop**, dans la page **Azure Virtual Desktop**, dans la section **Gérer**, sélectionnez **Pools d’hôtes**, puis, dans la page **Pools d’hôtes Azure Virtual Desktop \|**, sélectionnez **+ Créer**. 
1. Dans l’onglet **Informations de base** de la page **Créer un pool d’hôtes**, spécifiez les paramètres suivants et sélectionnez **Suivant : Hôtes de session >** (laissez les autres paramètres avec leurs valeurs par défaut) :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|Nom d’un nouveau groupe de ressources **az140-21e-RG**|
    |Nom du pool d’hôtes|**az140-21-hp1**|
    |Emplacement|Nom de la région Azure dans laquelle déployer votre environnement Azure Virtual Desktop|
    |Environnement de validation|**Aucun**|
    |Type de groupe d’applications préféré|**Bureau**|
    |Type de pool d’hôtes|**Groupé**|
    |Créer une configuration d’hôte de session|**Non**|
    |Algorithme d’équilibrage de charge|**À largeur prioritaire**|

    > **Note** : lors de l’utilisation de l’algorithme d’équilibrage de charge en largeur d’abord, le paramètre de limite de session maximale est facultatif.

1. Dans l’onglet **Hôtes de session** de la page **Créer un pool d’hôtes**, spécifiez les paramètres suivants et sélectionnez **Suivant : Espace de travail >** (laissez les autres paramètres avec leurs valeurs par défaut) :

    > **Note** : lorsque vous définissez la valeur du **préfixe de nom**, basculez vers l’onglet Ressources situé à droite de la fenêtre de session de labo et identifiez la chaîne de caractères entre *User1-* et le caractère *@*. Utilisez cette chaîne pour remplacer l’espace réservé *aléatoire*.

    |Paramètre|Valeur|
    |---|---|
    |Ajouter des machines virtuelles|**Oui**|
    |Resource group|**Par défaut, identique à celui du pool d’hôtes**|
    |Préfixe de nom|**sh**-*random*|
    |Type de machine virtuelle|**Machine virtuelle Azure**|
    |Emplacement des machines virtuelles|Nom de la région Azure dans laquelle déployer votre environnement Azure Virtual Desktop|
    |Options de disponibilité|**Aucune redondance de l’infrastructure requise**|
    |Type de sécurité|**Lancement fiable des machines virtuelles**|
    |Image|**Windows 11 Entreprise multisession, version 23H2 + Microsoft 365 Apps**|
    |Taille de la machine virtuelle|**Standard D2s v3**|
    |Nombre d'ordinateurs virtuels|**2**|
    |Type de disque du système d’exploitation|**SSD Standard**|
    |Taille du disque de système d’exploitation|**Taille par défaut (128 Gio)**|
    |Diagnostics de démarrage|**Activer avec le compte de stockage managé (recommandé)**|
    |Réseau virtuel|**az140-vnet11e**|
    |Sous-réseau|**hp1-Subnet**|
    |Groupe de sécurité réseau|**De base**|
    |Ports d’entrée publics|**Aucun**|
    |Sélectionnez le répertoire que vous souhaitez rejoindre|**Microsoft Entra ID**|
    |Inscrire une machine virtuelle avec Intune|**Non**|
    |Nom d'utilisateur|**Étudiant**|
    |Mot de passe|Toute chaîne suffisamment complexe de caractères qui sera utilisée comme mot de passe pour le compte Administrateur intégré|
    |Confirmer le mot de passe|La même chaîne de caractères que vous avez spécifiée précédemment|

    > **Note** : le mot de passe doit comporter au moins 12 caractères et se composer d’une combinaison de caractères minuscules, de caractères majuscules, de chiffres et de caractères spéciaux. Pour plus d’informations, reportez-vous aux informations relatives aux [exigences de mot de passe lors de la création d’une machine virtuelle Azure](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm-).

1. Dans l’onglet **Espace de travail** de la page **Créer un pool d’hôtes**, confirmez le paramètre suivant et sélectionnez **Examiner et créer** :

    |Paramètre|Valeur|
    |---|---|
    |Inscrire un groupe d'applications de bureau|**Non**|

1. Dans l’onglet **Examiner et créer** de la page **Créer un pool d’hôtes**, sélectionnez **Créer**.

    > **Remarque** : Attendez la fin du déploiement. Ceci peut prendre environ 20 minutes.

#### Tâche 3 : créer un groupe d’applications Azure Virtual Desktop

1. À partir de l’ordinateur de labo, dans la fenêtre du navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop**, puis dans la page **Azure Virtual Desktop**, sélectionnez **Groupes d’applications**.
1. Dans la page **Groupes d’applications Azure Virtual Desktop \|**, notez le groupe d’applications de bureau généré automatiquement **az140-21-hp1-DAG**, puis sélectionnez-le. 
1. Dans la page **az140-21a-hp1-DAG**, dans le menu vertical situé à gauche, dans la section **Gérer**, sélectionnez **Affectations**.
1. Dans la page **Affectations az140-21-hp1-DAG \|**, sélectionnez **+ Ajouter**.
1. Dans la page **Sélectionner des utilisateurs ou des groupes d’utilisateurs Microsoft Entra**, sélectionnez **Groupes**, dans la zone de recherche, tapez le nom complet du groupe **AVD-DAG** que vous avez identifié dans la première tâche de cet exercice, cochez la case en regard du nom du groupe, puis cliquez sur **Sélectionner**.
1. Revenez à la page **Groupes d’applications Azure Virtual Desktop \|**, sélectionnez **+ Créer**. 
1. Dans l’onglet **Informations de base** de la page **Créer un groupe d’applications**, spécifiez les paramètres suivants et sélectionnez **Suivant : Applications >**  :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|**az140-21e-RG**|
    |Pool d’hôtes|**az140-21-hp1**|
    |Type de groupe d’applications|**Application distante**|
    |Nom du groupe d'applications|**az140-21-hp1-Office365-RAG**|

1. Dans l’onglet **Applications** de la page **Créer un groupe d’applications**, sélectionnez **+ Ajouter des applications**.
1. Dans la page **Ajouter une application**, spécifiez les paramètres suivants et sélectionnez **Vérifier + ajouter**, puis cliquez sur **Ajouter** :

    |Paramètre|Valeur|
    |---|---|
    |Source de l’application|**Menu Démarrer**|
    |Application|**Word**|
    |Nom complet|**Microsoft Word**|
    |Description|**Microsoft Word**|
    |Demander la ligne de commande|**Non**|

1. De retour dans l’onglet **Applications** de la page **Créer un groupe d’applications**, sélectionnez **+ Ajouter des applications**.
1. Dans la page **Ajouter une application**, spécifiez les paramètres suivants et sélectionnez **Vérifier + ajouter**, puis cliquez sur **Ajouter** :

    |Paramètre|Valeur|
    |---|---|
    |Source de l’application|**Menu Démarrer**|
    |Application|**Excel**|
    |Nom complet|**Microsoft Excel**|
    |Description|**Microsoft Excel**|
    |Demander la ligne de commande|**Non**|

1. De retour dans l’onglet **Applications** de la page **Créer un groupe d’applications**, sélectionnez **+ Ajouter des applications**.
1. Dans la page **Ajouter une application**, spécifiez les paramètres suivants et sélectionnez **Vérifier + ajouter**, puis cliquez sur **Ajouter** :

    |Paramètre|Valeur|
    |---|---|
    |Source de l’application|**Menu Démarrer**|
    |Application|**PowerPoint**|
    |Nom complet|**Microsoft PowerPoint**|
    |Description|**Microsoft PowerPoint**|
    |Demander la ligne de commande|**Non**|

1. De retour dans l’onglet **Applications** de la page **Créer un groupe d’applications**, sélectionnez **Suivant : Affectations >**.
1. Dans l’onglet **Affectations** de la page **Créer un groupe d’applications**, sélectionnez **+ Ajouter des utilisateurs ou des groupes d’utilisateurs Microsoft Entra**.
1. Dans la page **Sélectionner des utilisateurs ou des groupes d’utilisateurs Microsoft Entra**, sélectionnez **Groupes**, tapez le nom complet du groupe **AVD-RemoteApp** que vous avez identifié dans la première tâche de cet exercice, cochez la case en regard du nom du groupe, puis cliquez sur **Sélectionner**.
1. De retour dans l’onglet **Affectations** de la page **Créer un groupe d’applications**, sélectionnez **Suivant : Espace de travail >**.
1. Dans l’onglet **Espace de travail** de la page **Créer un espace de travail**, spécifiez le paramètre suivant et sélectionnez **Examiner et créer** :

    |Paramètre|Valeur|
    |---|---|
    |Inscrire le groupe d'applications|**Non**|

1. Dans l’onglet **Examiner et créer** de la page **Créer un groupe d’applications**, sélectionnez **Créer**.

    > **Remarque** : Attendez que le groupe d’applications soit créé. Cela devrait prendre moins d’une minute. 

    > **Remarque** : Ensuite, vous allez créer un groupe d’applications en fonction du chemin d’accès au fichier en tant que source de l’application.

1. Dans la fenêtre du navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop**, puis dans la page **Azure Virtual Desktop**, sélectionnez **Groupes d’applications**.
1. Dans la page **Groupes d’applications Azure Virtual Desktop \|**, sélectionnez **+ Créer**. 
1. Dans l’onglet **Informations de base** de la page **Créer un groupe d’applications**, spécifiez les paramètres suivants et sélectionnez **Suivant : Applications >**  :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|**az140-21e-RG**|
    |Pool d’hôtes|**az140-21-hp1**|
    |Type de groupe d’applications|**RemoteApp**|
    |Nom du groupe d'applications|**az140-21-hp1-Utilities-RAG**|

1. Dans l’onglet **Applications** de la page **Créer un groupe d’applications**, sélectionnez **+ Ajouter des applications**.
1. Dans la page **Ajouter une application**, dans l’onglet **Informations de base**, spécifiez les paramètres suivants et sélectionnez **Suivant** :

    |Paramètre|Valeur|
    |---|---|
    |Source de l’application|**Chemin de fichier**|
    |Chemin d’application|**C:\Windows\system32\cmd.exe**|
    |Identificateur d’application|**Invite de commandes**|
    |Nom d’affichage|**Invite de commandes**|
    |Description|**Invite de commandes Windows**|
    |Demander la ligne de commande|**Aucun**|

1. Sous l’onglet **Icône**, spécifiez les paramètres suivants, puis sélectionnez **Vérifier + ajouter**, puis sélectionnez **Ajouter** :

    |Paramètre|Valeur|
    |---|---|
    |Chemin d’accès à l’icône|**C:\Windows\system32\cmd.exe**|
    |Index d’icône|0|

1. De retour dans l’onglet **Applications** de la page **Créer un groupe d’applications**, sélectionnez **Suivant : Affectations >**.
1. Dans l’onglet **Affectations** de la page **Créer un groupe d’applications**, sélectionnez **+ Ajouter des utilisateurs ou des groupes d’utilisateurs Microsoft Entra**.
1. Dans la page **Sélectionner des utilisateurs ou des groupes d’utilisateurs Microsoft Entra**, sélectionnez **Groupes**, tapez le nom complet du groupe **AVD-RemoteApp** que vous avez identifié dans la première tâche de cet exercice, cochez la case en regard du nom du groupe, puis cliquez sur **Sélectionner**.
1. De retour dans l’onglet **Affectations** de la page **Créer un groupe d’applications**, sélectionnez **Suivant : Espace de travail >**.
1. Dans l’onglet **Espace de travail** de la page **Créer un espace de travail**, spécifiez le paramètre suivant et sélectionnez **Examiner et créer** :

    |Paramètre|Valeur|
    |---|---|
    |Inscrire le groupe d'applications|**Non**|

1. Dans l’onglet **Examiner et créer** de la page **Créer un groupe d’applications**, sélectionnez **Créer**.

    > **Remarque** : Attendez que le groupe d’applications soit créé. Cela devrait prendre moins d’une minute. 

#### Tâche 4 : créer un espace de travail Azure Virtual Desktop

1. Sur l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop** et, dans la page **Azure Virtual Desktop**, sélectionnez **Espaces de travail**.
1. Dans la page **Espaces de travail Azure Virtual Desktop \|**, sélectionnez **+ Créer**. 
1. Dans l’onglet **Informations de base** de la page **Créer un espace de travail**, spécifiez les paramètres suivants et sélectionnez **Suivant : Groupes d’applications >**  :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|**az140-21e-RG**|
    |Nom de l’espace de travail|**az140-21-ws1**|
    |Nom convivial|**az140-21-ws1**|
    |Emplacement|Nom de la région Azure dans laquelle vous avez déployé des ressources dans le premier exercice de ce labo ou d’une région proche de celle-ci|

1. Dans l’onglet **Groupes d’applications** de la page **Créer un espace de travail**, spécifiez les paramètres suivants :

    |Paramètre|Valeur|
    |---|---|
    |Inscrire des groupes d’applications|**Oui**|

1. Dans l’onglet **Espace de travail** de la page **Créer un espace de travail**, sélectionnez **+ Inscrire des groupes d’applications**.
1. Dans la page **Ajouter des groupes d’applications**, sélectionnez le signe plus en regard des entrées **az140-21-hp1-DAG**, **az140-21-hp1-Office365-RAG** et **az140-21-hp1-Utilities-RAG**, puis cliquez sur **Sélectionner**. 
1. De retour dans l’onglet **Groupes d’applications** de la page **Créer un espace de travail**, sélectionnez **Examiner et créer**.
1. Dans l’onglet **Examiner et créer** de la page **Créer un espace de travail**, sélectionnez **Créer**.

#### Tâche 5 : permettre d’accéder aux pools d’hôtes Azure Virtual Desktop

> **Note** : lorsque Microsoft Entra a joint des hôtes de session, vous devez affecter aux utilisateurs et administrateurs Azure Virtual Desktop des rôles Azure appropriés de contrôle d’accès en fonction du rôle (RBAC). En particulier, le rôle *Connexion de l’utilisateur aux machines virtuelles* est requis pour se connecter aux hôtes de session et le rôle *Connexion de l’administrateur aux machines virtuelles* est requis pour les privilèges d’administration locaux. 

1. Sur l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Groupes de ressources**, puis, dans la page **Groupes de ressources**, sélectionnez **az140-21e-RG**.
1. Dans la page **az140-21e-RG**, dans le menu de navigation vertical, sélectionnez **Contrôle d’accès (IAM)**.
1. Dans la page **Contrôle d’accès (IAM) az140-21e-RG \|**, sélectionnez **+ Ajouter**, puis dans le menu déroulant, sélectionnez **Ajouter une attribution de rôle**.
1. Dans l’onglet **Rôle** de la page **Ajouter une attribution de rôle**, vérifiez que l’onglet **Rôles de fonction de tâche** est sélectionné, dans la zone de texte de recherche, entrez **Connexion de l’utilisateur aux machines virtuelles**, dans la liste des résultats, sélectionnez **Connexion de l’utilisateur aux machines virtuelles**, puis sélectionnez **Suivant**.
1. Dans l’onglet **Membres** de la page **Ajouter une attribution de rôle**, vérifiez que l’option **Utilisateur, groupe ou principal de service** est sélectionnée, cliquez sur **+ Sélectionner des membres**, dans le volet **Sélectionner des membres**, recherchez le groupe **AVD-RemoteApp** que vous avez identifié dans la première tâche de cet exercice, puis cliquez sur **Sélectionner**.
1. Dans l’onglet **Membres** de la page **Ajouter une attribution de rôle**, cliquez sur **Suivant**.
1. Dans l’onglet **Type d’attribution** de la page **Ajouter une attribution de rôle**, définissez le **Type d’attribution** sur **Actif**, puis sélectionnez **Vérifier + attribuer**.
1. Dans l’onglet **Vérifier + attribuer** de la page **Ajouter une attribution de rôle**, sélectionnez **Vérifier + attribuer**. 
1. De retour sur la page **Contrôle d’accès (IAM) az140-21e-RG \|**, sélectionnez **+ Ajouter** puis, dans le menu déroulant, sélectionnez **Ajouter une attribution de rôle**.
1. Dans l’onglet **Rôle** de la page **Ajouter une attribution de rôle**, vérifiez que l’onglet **Rôles de fonction de tâche** est sélectionné, dans la zone de texte de recherche, entrez **Connexion de l’administrateur aux machines virtuelles**, dans la liste des résultats, sélectionnez **Connexion de l’administrateur aux machines virtuelles**, puis sélectionnez **Suivant**.
1. Dans l’onglet **Membres** de la page **Ajouter une attribution de rôle**, vérifiez que l’option **Utilisateur, groupe ou principal de service** est sélectionnée, cliquez sur **+ Sélectionner des membres**, dans le volet **Sélectionner des membres**, recherchez le groupe **AVD-DAG** que vous avez identifié dans la première tâche de cet exercice, puis cliquez sur **Sélectionner**.
1. De retour dans l’onglet **Membres** de la page **Ajouter une attribution de rôle**, définissez le **Type d’attribution** sur **Actif**, puis sélectionnez **Vérifier + attribuer**, et à nouveau **Vérifier + attribuer**. 
