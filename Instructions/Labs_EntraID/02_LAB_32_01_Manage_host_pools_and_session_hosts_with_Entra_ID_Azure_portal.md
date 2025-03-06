---
lab:
  title: "Labo\_: gérer des pools d’hôtes et des hôtes de session à l’aide du portail Azure (Entra\_ID)"
  module: 'Module 3.2: Plan and implement user experience and client settings'
---

# Labo - Gérer des pools d’hôtes et des hôtes de session à l’aide du portail Azure (Entra ID)
# Manuel de labo de l’étudiant

## Dépendances du labo

- Un abonnement Azure que vous allez utiliser dans ce labo.
- Un compte d’utilisateur Microsoft Entra avec le rôle Propriétaire ou Collaborateur dans l’abonnement Azure que vous allez utiliser dans ce labo et avec les autorisations suffisantes pour joindre des appareils au locataire Entra associé à cet abonnement Azure.
- Le labo terminé *Déployer des pools d’hôtes et des hôtes de session à l’aide du portail Azure (Entra ID)*

## Durée estimée

30 minutes

## Scénario de labo

Vous disposez d’un environnement Azure Virtual Desktop existant. Vous devez configurer le pool d’hôtes avec les hôtes de session joints à Microsoft Entra pour prendre en charge une gamme de besoins fonctionnels et métier. Ces conditions requises incluent :

- Déployer des hôtes de session supplémentaires pour prendre en charge un nombre accru d’utilisateurs distants
- Réduire le coût de l’environnement Azure Virtual Desktop en optimisant la configuration d’équilibrage de charge du pool d’hôtes et en tirant parti de la fonctionnalité *Démarrer la machine virtuelle à la connexion*
- Optimiser la disponibilité des hôtes de session pendant les heures d’ouverture en implémentant des fenêtres de maintenance
- Activer l’authentification unique auprès des hôtes de session joints à Microsoft Entra
- Optimiser l’utilisation et l’expérience utilisateur (comme la reconnexion automatique des sessions déconnectées)

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

- Configurer les hôtes de session Azure Virtual Desktop joints à Microsoft Entra pour prendre en charge une gamme de besoins fonctionnels et métier

## Fichiers du labo

- Aucun

## Instructions

### Exercice 1 : gérer un environnement Azure Virtual Desktop contenant des hôtes de session joints à Microsoft Entra
  
Les principales tâches de cet exercice sont les suivantes

1. Déployer des hôtes de session de pool d’hôtes Azure Virtual Desktop supplémentaires
1. Examiner et configurer les propriétés du pool d’hôtes
1. Attribuer le rôle RBAC requis à un principal de service Azure Virtual Desktop
1. Configurer les mises à jour planifiées de l’agent
1. Configurer les propriétés RDP du pool d’hôtes

#### Tâche 1 : déployer des hôtes de session de pool d’hôtes Azure Virtual Desktop supplémentaires

1. Si besoin, à partir de votre ordinateur de labo, démarrez un navigateur web, accédez au portail Azure, puis connectez-vous en fournissant des informations d’identification d’un compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utiliserez dans ce labo.

    > **Note** : utilisez les informations d’identification du compte `User1-` répertorié dans l’onglet Ressources sur le côté droit de la fenêtre de session de labo.

1. Dans la fenêtre du navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop** et, dans la page **Azure Virtual Desktop**, dans la barre de menus verticale, dans la section **Gére**, sélectionnez **Pools d’hôtes**.
1. Dans la page **Pool d’hôtes Azure Virtual Desktop \|**, dans la liste des pools d’hôtes, sélectionnez **az140-21-hp1**.
1. Dans la page **az140-21-hp1**, dans la barre de menus verticale, dans la section **Gérer**, sélectionnez **Hôtes de session** et vérifiez que le pool se compose de deux hôtes. 
1. Dans la page **Hôtes de session az140-21-hp1\|**, sélectionnez **+ Ajouter**.
1. Dans l’onglet **Informations de base** de la page **Ajouter des machines virtuelles à un pool d’hôtes**, passez en revue les paramètres préconfigurés et sélectionnez **Suivant : Machines virtuelles**.
1. Dans l’onglet **Machines virtuelles** de la page **Ajouter des machines virtuelles à un pool d’hôtes**, spécifiez les paramètres suivants et sélectionnez **Examiner et créer** (laissez les autres avec leurs paramètres par défaut) :

    > **Note** : lorsque vous définissez la valeur du **préfixe de nom**, basculez vers l’onglet Ressources situé à droite de la fenêtre de session de labo et identifiez la chaîne de caractères entre *User1-* et le caractère *@*. Utilisez cette chaîne pour remplacer l’espace réservé *aléatoire*.

    |Paramètre|Valeur|
    |---|---|
    |Resource group|**az140-21e-RG**|
    |Préfixe de nom|**sh**-*random*|
    |Emplacement des machines virtuelles|Nom de la région Azure dans laquelle vous avez déployé les deux premières machines virtuelles hôtes de session|
    |Options de disponibilité|**Aucune redondance de l’infrastructure requise**|
    |Type de sécurité|**Lancement fiable des machines virtuelles**|
    |Image|**Windows 11 Entreprise multisession, version 23H2 + Microsoft 365 Apps**|
    |Taille de la machine virtuelle|**Standard D2s v3**|
    |Nombre d'ordinateurs virtuels|**1**|
    |Type de disque du système d’exploitation|**SSD Standard**|
    |Taille du disque de système d’exploitation|**Taille par défaut (128 Go)**|
    |Diagnostics de démarrage|**Activer avec le compte de stockage managé (recommandé)**|
    |Réseau virtuel|**az140-vnet11e**|
    |Sous-réseau|**hp1-Subnet**|
    |Groupe de sécurité réseau|**De base**|
    |Ports d’entrée publics|**Aucun**|
    |Sélectionnez le répertoire que vous souhaitez rejoindre|**Microsoft Entra ID**|
    |Inscrire une machine virtuelle avec Intune|**Non**|
    |Nom d'utilisateur|**Étudiant**|
    |Mot de passe|Le même mot de passe que vous avez utilisé lors du déploiement des hôtes de session dans le labo *Déployer des pools d’hôtes et des hôtes de session à l’aide du portail Azure (Entra ID)* 
    |Confirmer le mot de passe|Le même mot de passe que vous avez spécifié précédemment|

    > **Note** : le mot de passe doit comporter au moins 12 caractères et se composer d’une combinaison de caractères minuscules, de caractères majuscules, de chiffres et de caractères spéciaux. Pour plus d’informations, reportez-vous aux informations relatives aux [exigences de mot de passe lors de la création d’une machine virtuelle Azure](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm-).

    > **Remarque** : Comme vous l’avez probablement remarqué, il est possible de modifier l’image et le préfixe des machines virtuelles au fur et à mesure que vous ajoutez des hôtes de session au pool existant. En général, cela n’est pas recommandé, sauf si vous envisagez de remplacer toutes les machines virtuelles du pool. 

1. Dans l’onglet **Examiner et créer** de la page **Ajouter des machines virtuelles à un pool d’hôtes**, sélectionnez **Créer**.

    > **Remarque** : N’attendez pas que le processus d’approvisionnement se termine, mais passez à la tâche suivante. Le processus d’approvisionnement peut prendre environ 20 minutes. 

#### Tâche 2 : vérifier et configurer les propriétés du pool d’hôtes

1. À partir de l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop**, dans la page **Azure Virtual Desktop**, dans la section **Gérer** du menu de navigation vertical, sélectionnez **Pools d’hôtes** et, dans la page **Pools d’hôtes Azure Virtual Desktop \|**, sélectionnez **az140-21-hp1**. 
1. Dans la page **az140-21-hp1**, dans la section **Paramètres**, sélectionnez **Propriétés**.
1. Dans la page **Propriétés az140-21-hp1 \|**, consultez les options de configuration disponibles, notamment :

    - **Type de groupe d’applications préféré** : cette option définit le type de groupe d’applications préféré pour le pool d’hôtes **Desktop** ou **RemoteApp**. Si les utilisateurs finaux ont des applications RemoteApp et Desktop publiées dans leur direction dans le pool d’hôtes, ils voient uniquement le type d’application sélectionné dans leur flux.
    - **Démarrer la machine virtuelle à la connexion** : l’activation de cette option permet aux utilisateurs de démarrer des machines virtuelles individuelles dans le pool d’hôtes à partir de l’état Libéré.
    - **Environnement de validation** : le pool d’hôtes de validation est destiné à tester les modifications du service avant qu’elles ne soient déployées en production.
    - **Algorithme d’équilibrage de charge** : cette option offre le choix entre l’équilibrage de charge en largeur d’abord et en profondeur d’abord. L’équilibrage de charge en largeur d’abord répartit les nouvelles sessions utilisateur entre tous les hôtes de session du pool d’hôtes. L’équilibrage de charge en profondeur d’abord répartit les nouvelles sessions utilisateur vers l’hôte de session disponible doté du plus grand nombre de connexions, sans avoir atteint sa limite maximale de sessions.

1. Dans la page **Propriétés az140-21-hp1 \|**, dans la liste déroulante **Algorithme d’équilibrage de charge**, sélectionnez **Profondeur d’abord**.
1. Dans la zone de texte **Limite de session maximale**, entrez **8**.
1. Dans la page **Propriétés az140-21-hp1 \|**, définissez **Démarrer la machine virtuelle à la connexion **sur **Oui**.

    > **Note** : l’option *Démarrer la machine virtuelle à la connexion* vous permet de réduire les coûts en permettant aux utilisateurs finaux de mettre sous tension les machines virtuelles utilisées en tant qu’hôtes de session uniquement quand ils sont nécessaires. Pour les pools d’hôtes personnels, l’option *Démarrer la machine virtuelle à la connexion* met uniquement sous tension une machine virtuelle hôte de session existante qui est déjà affectée ou peut être affectée à un utilisateur. Pour les pools d’hôtes groupés, l’option *Démarrer la machine virtuelle à la connexion* met uniquement sous tension une machine virtuelle hôte de session lorsqu’aucune machine virtuelle n’est activée et que d’autres machines virtuelles ne sont activées que lorsque la première machine virtuelle atteint la limite de session.

1. Dans la page **Propriétés az140-21-hp1 \|**, sélectionnez **Enregistrer**.

    > **Note** : l’utilisation de l’option *Démarrer la machine virtuelle à la connexion* nécessite d’attribuer le rôle de contrôle d’accès en fonction du rôle (RBAC) *Contributeur de mise sous tension de la visualisation du Bureau* au principal du service *Azure Virtual Desktop* dans l’étendue de l’abonnement Azure. 

#### Tâche 3 : attribuer le rôle RBAC requis à un principal de service Azure Virtual Desktop

1. Sur l’ordinateur de labo, dans la fenêtre de navigateur web affichant le portail Azure, ouvrez une session PowerShell dans le volet Azure Cloud Shell.

    > **Note** : si vous y êtes invité, dans le volet **Mise en route**, dans la liste déroulante **Abonnement**, sélectionnez le nom de l’abonnement Azure que vous utilisez dans ce labo, puis sélectionnez **Appliquer**.

1. Dans la session PowerShell dans le volet Azure Cloud Shell, exécutez la commande suivante pour récupérer la valeur de la propriété ID de l’abonnement Azure que vous utilisez dans ce labo et stockez-la dans une variable `$subId` :

    ```powershell
    $subId = (Get-AzSubscription).Id
    ```

1. Exécutez la commande suivante pour créer une variable $parameters, qui stocke une table de hachage qui contient les valeurs du nom de définition de rôle RBAC, l’application Microsoft Entra représentant le principal de service **Azure Virtual Desktop** et l’étendue de l’abonnement :

    ```powershell
    $parameters = @{
        RoleDefinitionName = "Desktop Virtualization Power On Contributor"
        ApplicationId = "9cdead84-a844-4324-93f2-b2e6bb768d07"
        Scope = "/subscriptions/$subId"
    }
    ```

1. Exécutez la commande suivante pour créer une attribution de rôle RBAC :

    ```powershell
    New-AzRoleAssignment @parameters
    ```

1. Fermez le volet Cloud Shell.

#### Tâchev4 : configurer les mises à jour planifiées de l’agent

> **Note** : la fonctionnalité Mises à jour planifiées de l’agent vous permet de créer jusqu’à deux fenêtres de maintenance pour l’agent Azure Virtual Desktop, la pile côte à côte et l’agent de supervision Geneva afin que les mises à jour ne se produisent pas pendant les heures d’ouverture. 

1. Dans le navigateur affichant le portail Azure, retournez à la page du pool d’hôtes **az140-21-hp1**.
1. Dans la page **az140-21-hp1**, dans la barre de menus verticale, dans la section **Paramètres**, sélectionnez l’entrée **Mises à jour planifiées de l’agent** et, dans la page **Mises à jour planifiées de l’agent az140-21-hp1 \|**, cochez la case **Mises à jour planifiées de l’agent**.
1. Dans la section **Planification**, cochez la case **Utiliser le fuseau horaire local de l’hôte de session**.
1. Dans la section **Fenêtre de maintenance**, dans la liste déroulante **Jour**, sélectionnez **Samedi** et, dans la liste déroulante **Heure**, sélectionnez **23:00**.
1. Sélectionnez **Appliquer**.

#### Tâche 5 : Configurer les propriétés RDP du pool d’hôtes

1. Dans le navigateur web affichant le portail Azure, dans la page **az140-21-hp1**, dans la barre de menus verticale, dans la section **Paramètres**, sélectionnez l’entrée **Propriétés RDP**.
1. Dans l’onglet **Informations de connexion** de la page **Propriétés RDP az140-21-hp1 \|**, consultez les options de configuration disponibles, notamment :

    - **Authentification unique Microsoft Entra** : cette option détermine si les connexions tenteront de tirer parti de l’authentification Microsoft Entra pour se connecter aux hôtes de session joints à Microsoft Entra et fournissent ainsi une expérience d’authentification unique effective. Notez qu’il n’est pas nécessaire que l’ordinateur client soit joint à Microsoft Entra. 
    - **Protocole CredSSP (Credential Security Support Provider** : cette option contrôle l’utilisation du protocole CredSSP pour l’authentification. Le protocole CredSSP permet de transférer en toute sécurité les informations d’identification de l’utilisateur à partir de l’appareil client vers l’hôte de session Bureau à distance. Toutefois, ses fonctionnalités n’incluent pas la prise en charge de l’authentification Entra ID.
    - **Autre interpréteur de commandes** : cette option vous permet de spécifier un exécutable à démarrer chaque fois qu’une nouvelle connexion à un hôte de session est établie. Ce paramètre s’applique uniquement aux hôtes de session exécutant Windows Server.
    - **Nom du proxy KDC** : cette option permet de rediriger via proxy le trafic d’authentification Kerberos vers les contrôleurs de domaine Active Directory.

    > **Note** : étant donné que trois de ces options ne sont pas applicables dans notre scénario (qui implique les hôtes de session joints à Microsft Entra sans aucune présence de services de domaine Active Directory), vous ne configurerez que la première. Cette option correspond à la propriété RDP `enablerdsaadauth:i:value`.

1. Dans la liste déroulante de l’**Authentification unique Microsoft Entra**, sélectionnez l’option **Les onnexions utiliseront l’authentification Microsoft Entra pour fournir l’authentification unique**, puis sélectionnez **Enregistrer**.

    > **Important** : il est essentiel de garder à l’esprit que l’activation de cette propriété RDP spécifique n’est qu’une des étapes nécessaires pour implémenter la fonctionnalité d’authentification unique. D’autres actions applicables à ce scénario incluent l’activation de l’authentification Microsoft Entra pour RDP dans le locataire Entra et la configuration des groupes d’appareils, qui ne sont pas pris en charge dans la version actuelle de l’environnement de labo, par conséquent, ne sont pas incluses dans les instructions. Pour obtenir la liste complète des actions nécessaires pour implémenter l’authentification unique pour Microsoft Entra ID, reportez-vous à [Configurer l’authentification unique pour Azure Virtual Desktop à l’aide de l’authentification Microsoft Entra ID](https://learn.microsoft.com/en-us/azure/virtual-desktop/configure-single-sign-on).

1. Dans la page **Propriétés RDP az140-21-hp1 \|**, sélectionnez l’onglet **Comportement de session** et passez en revue les options de configuration disponibles, notamment :

    - **Reconnexion** : cette option détermine si l’ordinateur client tente automatiquement de se reconnecter à l’ordinateur distant en cas d’arrêt de la connexion.
    - **Détection automatique de bande passante** : cette option détermine s’il faut utiliser la détection automatique de bande passante réseau ou non.
    - **Détection automatique de réseau** : cette option vous permet d’activer la détection automatique du type de réseau. Elle est utilisée conjointement avec **Détection automatique de bande passante**. 
    - **Compression** : cette option détermine si la connexion doit utiliser la compression en bloc.
    - **Lecture vidéo** : cette option permet d’utiliser la diffusion multimédia en continu adaptée au protocole RDP pour la lecture vidéo.

1. Dans l’onglet **Comportement de session**, dans la liste déroulante **Reconnexion**, sélectionnez **Le client tente automatiquement de se reconnecter**, puis sélectionnez **Enregistrer**.
1. Dans la page **Propriétés RDP az140-21-hp1 \|**, sélectionnez l’onglet **Redirection d’appareils** et passez en revue les options de configuration disponibles, y compris deux catégories principales :

    - **Audio et vidéo**
    - **Appareils et ressources locaux**

    > **Note** : par défaut, la redirection s’applique à tous les lecteurs de disque, y compris ceux montés après l’établissement de la connexion initiale.

1. Dans la page **Propriétés RDP az140-21-hp1 \|**, sélectionnez l’onglet **Paramètres d’affichage** et passez en revue les options de configuration disponibles, notamment la prise en charge de plusieurs affichages, du dimensionnement intelligent et des tailles de bureau spécifiques (en pixels). 
1. Dans la page **Propriétés RDP az140-21-hp1 \|**, sélectionnez l’onglet **Avancé** et passez en revue les paramètres de configuration existants. Notez que ces paramètres reflètent les modifications que vous avez apportées précédemment dans cette tâche.