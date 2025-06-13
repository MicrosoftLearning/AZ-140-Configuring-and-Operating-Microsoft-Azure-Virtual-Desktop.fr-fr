---
lab:
  title: "Labo\_: implémenter Azure\_Private\_Link pour Azure\_Virtual\_Desktop"
  module: 'Module 1.1: Plan, implement, and manage networking for Azure Virtual Desktop'
---

# Labo - Implémenter Azure Private Link pour Azure Virtual Desktop
# Manuel de labo de l’étudiant

## Dépendances du labo

- Un abonnement Azure que vous allez utiliser dans ce labo.
- Un compte d’utilisateur Microsoft Entra avec le rôle Propriétaire dans l’abonnement Azure que vous allez utiliser dans ce labo et avec les autorisations suffisantes pour joindre des appareils au locataire Entra associé à cet abonnement Azure.
- Le labo terminé *Déployer des pools d’hôtes et des hôtes de session à l’aide du portail Azure (Entra ID)*

## Durée estimée

60 minutes

## Scénario de labo

Vous disposez d’un environnement Azure Virtual Desktop existant. Vous devez implémenter la connexion à l’environnement à l’aide d’Azure Private Link. 

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

- Implémenter Azure Private Link pour Azure Virtual Desktop

## Fichiers du labo

- Aucun

## Instructions

### Exercice 1 : implémenter Azure PrivateLink pour Azure Virtual Desktop
  
Les principales tâches de cet exercice sont les suivantes

1. Réinscrire le fournisseur de ressources Azure Virtual Desktop
1. Créer un sous-réseau dans un réseau virtuel Azure.
1. Implémenter un point de terminaison privé pour les connexions à un pool d’hôtes
1. Implémenter un point de terminaison privé pour le téléchargement de flux
1. Implémenter un point de terminaison privé pour la détection de flux initiale
1. Valider la fonctionnalité de point de terminaison privé
1. Autoriser l’accès au réseau public à un pool d’hôtes et à un espace de travail

> **Note** : Azure Virtual Desktop dispose de trois flux de travail auxquels correspondent trois types de ressource à utiliser avec des points de terminaison privés. Ces workflows sont les suivants :

- **Détection de flux initiale** : permet aux clients RDP de découvrir tous les espaces de travail attribués à un utilisateur. Pour implémenter ce flux de travail via Private Link, vous devez créer un point de terminaison privé unique à la sous-ressource globale dans n’importe quel espace de travail qui fait partie de votre déploiement Azure Virtual Desktop. Toutefois, quel que soit l’espace de travail que vous choisissez, il ne peut y avoir qu’un seul point de terminaison privé qui fournit cette fonctionnalité par déploiement
- **Téléchargement de flux** : permet aux clients RDP de télécharger tous les détails de connexion pour tous les espaces de travail qui hébergent les groupes d’applications de l’utilisateur actuel. Pour implémenter ce flux de travail via Private Link, vous devez créer un point de terminaison privé pour la sous-ressource de flux pour chaque espace de travail que vous envisagez de rendre disponible via le point de terminaison privé.
- **Connexions aux pools d’hôtes** : permet aux clients RDP et aux hôtes de session de se connecter à un pool d’hôtes. Pour implémenter ce flux de travail via Private Link, vous devez créer un point de terminaison privé pour la sous-ressource de connexion pour chaque pool d’hôtes que vous envisagez de rendre disponible via le point de terminaison privé.

> **Note** : vous pouvez implémenter ces flux de travail dans les dispositions suivantes :

- Toutes les parties de la connexion (découverte initiale du flux, téléchargement du flux et connexions de session à distance pour les clients et les hôtes de session) utilisent des routes privées.
- Le téléchargement du flux et les connexions de session à distance pour les clients et les hôtes de session utilisent des routes privées, mais la découverte initiale du flux utilise des routes publiques. 
- Seules les connexions de session à distance pour les clients et les hôtes de session utilisent des routes privées ; la découverte initiale du flux et le téléchargement du flux utilisent des routes publiques.
- Les clients et les machines virtuelles hôtes de session utilisent des itinéraires privés. Private Link n’est pas utilisé dans ce scénario.

> **Note** : dans ce labo, vous allez implémenter la première disposition.

#### Tâche 1 : réinscrire le fournisseur de ressources Azure Virtual Desktop

> **Note** : avant de pouvoir utiliser Private Link avec Azure Virtual Desktop, vous devez réinscrire le fournisseur de ressources **Microsoft.DesktopVirtualization**. 

1. Si besoin, à partir de votre ordinateur de labo, démarrez un navigateur web, accédez au portail Azure, puis connectez-vous en fournissant des informations d’identification d’un compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utiliserez dans ce labo.

    > **Note** : utilisez les informations d’identification du compte `User1-` répertorié dans l’onglet Ressources sur le côté droit de la fenêtre de session de labo.

1. À partir de l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Abonnements**, dans la page **Abonnements**, sélectionnez l’abonnement Azure que vous utilisez dans ce labo, puis, dans le menu de navigation vertical, dans la section **Paramètres**, sélectionnez **Fournisseurs de ressources**.
1. Dans l’onglet **Fournisseurs de ressources**, dans la zone de texte de recherche, entrez **Microsoft.DesktopVirtualization**, dans la liste des résultats, sélectionnez le petit cercle à gauche de l’entrée **Microsoft.DesktopVirtualization**, puis sélectionnez **Réinscrire**.

    > **Note** : attendez la fin du processus de réinscription. Cette étape prend généralement moins d’une minute.

#### Tâche 2 : créer un sous-réseau de réseau virtuel Azure

> **Note** : vous pouvez utiliser un sous-réseau existant d’un réseau virtuel Azure pour implémenter des points de terminaison privés dans le scénario de labo, mais il est courant d’utiliser un sous-réseau dédié à cet effet.

1. Sur l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Réseaux virtuels** et, dans la page **Réseaux virtuels**, sélectionnez **az140-vnet11e**.
1. Dans la page **az140-vnet11e**, dans la section **Paramètres** du menu de navigation vertical, sélectionnez **Sous-réseaux**.
1. Dans la page **Sous-réseaux az140-vnet11e \|**, sélectionnez **+ Sous-réseau**.
1. Dans le volet **Ajouter un sous-réseau**, spécifiez les paramètres suivants puis sélectionnez **Ajouter** (conservez les valeurs par défaut des autres paramètres) :

    |Paramètre|Valeur|
    |---|---|
    |Nom|**pe-Subnet**|
    |Adresse de début|**10.20.255.0**|
    |Activer le sous-réseau privé (aucun accès sortant par défaut)|Désactivé|

#### Tâche 3 : implémenter un point de terminaison privé pour les connexions à un pool d’hôtes

1. À partir de l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop**, dans la page **Azure Virtual Desktop**, dans la section **Gérer** du menu de navigation vertical, sélectionnez **Pools d’hôtes** et, dans la page **Pools d’hôtes Azure Virtual Desktop \|**, sélectionnez **az140-21-hp1**. 
1. Dans la page **az140-21-hp1**, dans le menu de navigation vertical, dans la section **Paramètres**, sélectionnez **Mise en réseau**.
1. Dans la page **Mise en réseau az140-21-hp1 \|**, sélectionnez l’onglet **Connexions de point de terminaison privé**, puis **+ Nouveau point de terminaison privé**.
1. Dans l’onglet **Informations de base** de la page **Créer un point de terminaison privé**, spécifiez les paramètres suivants et sélectionnez **Suivant : Ressource >**  :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|**az140-11e-RG**|
    |Nom|**az140-11-pehp1**|
    |Nom de l’interface réseau|**az140-11-pehp1-nic**|
    |Région|Nom de la région Azure où vous avez déployé votre environnement Azure Virtual Desktop|

1. Dans l’onglet **Ressource** de la page **Créer un point de terminaison privé**, spécifiez les paramètres suivants et sélectionnez **Suivant : Réseau virtuel** :

    |Paramètre|Valeur|
    |---|---|
    |Sous-ressource cible|**connexion**|

1. Dans l’onglet **Réseau virtuel** de la page **Créer un point de terminaison privé**, spécifiez les paramètres suivants et sélectionnez **Suivant : DNS >** (laissez les autres paramètres avec leurs valeurs par défaut) :

    |Paramètre|Valeur|
    |---|---|
    |Réseau virtuel|**az140-vnet11e (az140-11e-RG)**|
    |Sous-réseau|**pe-Subnet**|
    |Stratégie réseau pour les points de terminaison privés|**Disabled**|
    |Configuration d’adresse IP privée|**Allouer dynamiquement l’adresse IP**|

1. Dans l’onglet **DSN** de la page **Créer un point de terminaison privé**, spécifiez les paramètres suivants et sélectionnez **Suivant : Balises >**  :

    |Paramètre|Valeur|
    |---|---|
    |Intégrer à une zone DNS privée|**Oui**|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|**az140-11e-RG**|

    > **Note** : cette étape entraîne la création d’une zone DNS privée nommée **privatelink.wvd.microsoft.com**.

1. Dans l’onglet **Balises** de la page **Créer un point de terminaison privé**, sélectionnez **Suivant : Examiner et créer**.
1. Dans l’onglet **Examiner et créer** de la page **Créer un point de terminaison privé**, sélectionnez **Créer**.

    > **Remarque** : Attendez la fin du déploiement. Le déploiement peut prendre environ 3 minutes.

    > **Note** : vous devez créer un point de terminaison privé pour la sous-ressource de connexion pour chaque pool d’hôtes à utiliser avec Private Link.

#### Tâche 4 : implémenter un point de terminaison privé pour le téléchargement de flux

1. Sur l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop** et, dans la page **Azure Virtual Desktop**, sélectionnez **Espaces de travail**.
1. Dans la page **Espaces de travail Azure Virtual Desktop \|**, sélectionnez **az140-21-ws1**.
1. Dans la page **az140-21-ws1**, dans le menu de navigation vertical, dans la section **Paramètres**, sélectionnez **Mise en réseau**.
1. Dans la page **Mise en réseau az140-21-ws1 \|**, sélectionnez l’onglet **Connexions de point de terminaison privé**, puis sélectionnez **+ Nouveau point de terminaison privé**.
1. Dans l’onglet **Informations de base** de la page **Créer un point de terminaison privé**, spécifiez les paramètres suivants et sélectionnez **Suivant : Ressource >**  :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|**az140-11e-RG**|
    |Nom|**az140-11-pefeeddwnld**|
    |Nom de l’interface réseau|**az140-11-pefeeddwnld-nic**|
    |Région|Nom de la région Azure où vous avez déployé votre environnement Azure Virtual Desktop|

1. Dans l’onglet **Ressource** de la page **Créer un point de terminaison privé**, spécifiez les paramètres suivants et sélectionnez **Suivant : Réseau virtuel** :

    |Paramètre|Valeur|
    |---|---|
    |Sous-ressource cible|**feed**|

1. Dans l’onglet **Réseau virtuel** de la page **Créer un point de terminaison privé**, spécifiez les paramètres suivants et sélectionnez **Suivant : DNS >** (laissez les autres paramètres avec leurs valeurs par défaut) :

    |Paramètre|Valeur|
    |---|---|
    |Réseau virtuel|**az140-vnet11e (az140-11e-RG)**|
    |Sous-réseau|**pe-Subnet**|
    |Stratégie réseau pour les points de terminaison privés|**Disabled**|
    |Configuration d’adresse IP privée|**Allouer dynamiquement l’adresse IP**|

1. Dans l’onglet **DSN** de la page **Créer un point de terminaison privé**, spécifiez les paramètres suivants et sélectionnez **Suivant : Balises >**  :

    |Paramètre|Valeur|
    |---|---|
    |Intégrer à une zone DNS privée|**Oui**|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|**az140-11e-RG**|

    > **Note** : cette étape tire parti de la zone DNS privée nommée **privatelink.wvd.microsoft.com** que vous avez créée dans la tâche précédente.

1. Dans l’onglet **Balises** de la page **Créer un point de terminaison privé**, sélectionnez **Suivant : Examiner et créer**.
1. Dans l’onglet **Examiner et créer** de la page **Créer un point de terminaison privé**, sélectionnez **Créer**.

    > **Remarque** : N’attendez pas que le déploiement se termine, mais passez à la tâche suivante. Le déploiement peut prendre environ une minute.

    > **Note** : vous devez créer un point de terminaison privé pour la sous-ressource de flux pour chaque espace de travail à utiliser avec Private Link.

#### Tâche 5 : implémenter un point de terminaison privé pour la détection de flux initiale

1. Sur l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop** et, dans la page **Azure Virtual Desktop**, sélectionnez **Espaces de travail**.
1. Dans la page **Espaces de travail Azure Virtual Desktop \|**, sélectionnez **az140-21-ws1**.
1. Dans la page **az140-21-ws1**, dans le menu de navigation vertical, dans la section **Paramètres**, sélectionnez **Mise en réseau**.
1. Dans la page **Mise en réseau az140-21-ws1 \|**, sélectionnez l’onglet **Connexions de point de terminaison privé**, puis sélectionnez **+ Nouveau point de terminaison privé**.
1. Dans l’onglet **Informations de base** de la page **Créer un point de terminaison privé**, spécifiez les paramètres suivants et sélectionnez **Suivant : Ressource >**  :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|**az140-11e-RG**|
    |Nom|**az140-11-pefeeddisc**|
    |Nom de l’interface réseau|**az140-11-pefeeddisc-nic**|
    |Région|Nom de la région Azure où vous avez déployé votre environnement Azure Virtual Desktop|

1. Dans l’onglet **Ressource** de la page **Créer un point de terminaison privé**, spécifiez les paramètres suivants et sélectionnez **Suivant : Réseau virtuel** :

    |Paramètre|Valeur|
    |---|---|
    |Sous-ressource cible|**global**|

1. Dans l’onglet **Réseau virtuel** de la page **Créer un point de terminaison privé**, spécifiez les paramètres suivants et sélectionnez **Suivant : DNS >** (laissez les autres paramètres avec leurs valeurs par défaut) :

    |Paramètre|Valeur|
    |---|---|
    |Réseau virtuel|**az140-vnet11e (az140-11e-RG)**|
    |Sous-réseau|**pe-Subnet**|
    |Stratégie réseau pour les points de terminaison privés|**Disabled**|
    |Configuration d’adresse IP privée|**Allouer dynamiquement l’adresse IP**|

1. Dans l’onglet **DSN** de la page **Créer un point de terminaison privé**, spécifiez les paramètres suivants et sélectionnez **Suivant : Balises >**  :

    |Paramètre|Valeur|
    |---|---|
    |Intégrer à une zone DNS privée|**Oui**|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|**az140-11e-RG**|

    > **Note** : cette étape entraîne la création d’une zone DNS privée nommée **privatelink-global.wvd.microsoft.com**.

1. Dans l’onglet **Balises** de la page **Créer un point de terminaison privé**, sélectionnez **Suivant : Examiner et créer**.
1. Dans l’onglet **Examiner et créer** de la page **Créer un point de terminaison privé**, sélectionnez **Créer**.

    > **Remarque** : N’attendez pas que le déploiement se termine, mais passez à la tâche suivante. Le déploiement peut prendre environ une minute.

    > **Note** : vous devez créer un point de terminaison privé pour la sous-ressource globale de chaque espace de travail que vous souhaitez utiliser avec Private Link.

    > **Note** : pour que les modifications réseau prennent effet, vous devez redémarrer les hôtes de session dans le pool d’hôtes cible.

1. À partir de l’ordinateur de labo, dans le navigateur web affichant le portail Azure, accédez à la page **Azure Virtual Desktop**, dans la section **Gérer** du menu de navigation vertical, sélectionnez **Pools d’hôtes** et, dans la page **Pools d’hôtes Azure Virtual Desktop \|**, sélectionnez **az140-21-hp1**.
1. Dans la page **az140-21-hp1**, dans la section **Gérer** du menu de navigation vertical, sélectionnez **Hôtes de session**. 
1. Dans la liste des hôtes de session, cochez toutes les cases à cocher à gauche de chaque hôte de session, puis sélectionnez **Redémarrer** dans la barre d’outils.

    > **Note** : attendez que tous les hôtes de session se trouvent à l’état **En cours d’exécution**. 

#### Tâche 6 : valider la fonctionnalité de point de terminaison privé

> **Note** : par défaut, la connectivité aux espaces de travail Azure Virtual Desktop et aux pools d’hôtes est autorisée à partir de réseaux publics. Vous allez commencer par modifier les paramètres par défaut et appliquer l’accès privé.

1. Sur l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop** et, dans la page **Azure Virtual Desktop**, sélectionnez **Espaces de travail**.
1. Dans la page **Espaces de travail Azure Virtual Desktop \|**, sélectionnez **az140-21-ws1**.
1. Dans la page **az140-21-ws1**, dans le menu de navigation vertical, dans la section **Paramètres**, sélectionnez **Mise en réseau**.
1. Dans la page **Mise en réseau az140-21-ws1\|**, dans l’onglet **Accès public**, sélectionnez l’option **Désactiver l’accès public et utiliser l’accès privé**, puis sélectionnez **Enregistrer**.
1. À partir de l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop**, dans la page **Azure Virtual Desktop**, dans la section **Gérer** du menu de navigation vertical, sélectionnez **Pools d’hôtes** et, dans la page **Pools d’hôtes Azure Virtual Desktop \|**, sélectionnez **az140-21-hp1**. 
1. Dans la page **az140-21-hp1**, dans le menu de navigation vertical, dans la section **Paramètres**, sélectionnez **Mise en réseau**.
1. Dans la page **Mise en réseau az140-21-hp1\|**, dans l’onglet **Accès public**, sélectionnez l’option **Désactiver l’accès public et utiliser l’accès privé**, puis sélectionnez **Enregistrer**.

    > **Note** : pour valider la fonctionnalité de point de terminaison privé, un client RDP doit être connecté à un réseau disposant d’une connectivité privée au réseau virtuel Azure contenant le sous-réseau hébergeant les points de terminaison privés que vous avez créés précédemment dans ce labo. Pour simuler ce scénario, vous allez créer un autre sous-réseau dans le même réseau virtuel utilisé pour créer des points de terminaison privés et déployer une machine virtuelle Azure exécutant Windows 11 dans ce sous-réseau.

1. Sur l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Réseaux virtuels** et, dans la page **Réseaux virtuels**, sélectionnez **az140-vnet11e**.
1. Dans la page **az140-vnet11e**, dans la section **Paramètres** du menu de navigation vertical, sélectionnez **Sous-réseaux**.
1. Dans la page **Sous-réseaux az140-vnet11e \|**, sélectionnez **+ Sous-réseau**.
1. Dans le volet **Ajouter un sous-réseau**, spécifiez les paramètres suivants puis sélectionnez **Ajouter** (conservez les valeurs par défaut des autres paramètres) :

    |Paramètre|Valeur|
    |---|---|
    |Nom|**client-Subnet**|
    |Adresse de début|**10.20.2.0**|
    |Activer le sous-réseau privé (aucun accès sortant par défaut)|Désactivé|

1. À partir de l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Machines virtuelles**, dans la page **Machines virtuelles**, sélectionnez **+ Créer** et, dans la liste déroulante, sélectionnez **Machine virtuelle Azure**.
1. Dans l’onglet **Informations de base** de la page **Créer une machine virtuelle**, spécifiez les paramètres suivants et sélectionnez **Suivant : Disques >** (conservez les valeurs par défaut de tous les autres paramètres) :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|Nom d’un nouveau groupe de ressources **az140-111e-RG**|
    |Nom de la machine virtuelle|**az140-111e-vm0**|
    |Région|Nom de la région Azure où vous avez déployé votre environnement Azure Virtual Desktop|
    |Options de disponibilité|**Aucune redondance de l’infrastructure requise**|
    |Type de sécurité|**Standard**|
    |Image|**Windows 11 Professionnel, version 24H2 - x64 Gen2**|
    |Taille|**Standard D2s v3**|
    |Nom d’utilisateur|Tout nom d’utilisateur valide de votre choix|
    |Mot de passe|Tout mot de passe complexe de votre choix|
    |Aucun port d’entrée public|**Aucun(e)**|
    |Gestion des licences|Activer la case à cocher|

    > **Note** : le mot de passe doit comporter au moins 12 caractères et se composer d’une combinaison de caractères minuscules, de caractères majuscules, de chiffres et de caractères spéciaux. Pour plus d’informations, reportez-vous aux informations relatives aux [exigences de mot de passe lors de la création d’une machine virtuelle Azure](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm-).

1. Dans l’onglet **Disques** de la page **Créer une machine virtuelle**, définissez le **Type de disque du système d’exploitation** sur **HDD Standard (stockage localement redondant)** et sélectionnez **Suivant : Mise en réseau >**.
1. Dans l’onglet **Mise en réseau** de la page **Créer une machine virtuelle**, spécifiez les paramètres suivants (conservez les valeurs par défaut pour les autres) :

    |Paramètre|Valeur|
    |---|---|
    |Réseau virtuel|**az140-vnet11e**|
    |Sous-réseau|**client-Subnet**|
    |Adresse IP publique|**(nouveau) az140-111e-vm0-ip**|
    |Groupe de sécurité réseau de la carte réseau|**Avancée**|

1. Dans l’onglet **Mise en réseau** de la page **Créer une machine virtuelle**, en regard de la liste déroulante **Configurer un groupe de sécurité réseau**, sélectionnez **Créer**.
1. Dans la page **Créer un groupe de sécurité réseau**, supprimez la règle entrante créée au préalable **1000: default-allow-rdp**, puis sélectionnez **+ Ajouter une règle entrante**.
1. Dans le volet **Ajouter une règle de sécurité de trafic entrant**, dans la liste déroulante **Source**, sélectionnez **Mon adresse IP** pour identifier l’adresse IP publique représentant votre connexion à Internet.
1. Dans le volet **Ajouter une règle de sécurité de trafic entrant**, spécifiez les paramètres suivants puis sélectionnez **Ajouter** (conservez les valeurs par défaut des autres paramètres) :

    |Paramètre|Valeur|
    |---|---|
    |Source|**Adresses IP**|
    |Plages d’adresses IP/CIDR sources|Laisser inchangé (cela doit toujours contenir votre adresse IP publique)|
    |Source port ranges|*|
    |Destination|**Any**|
    |Service|**RDP**|
    |Action|**Autoriser**|
    |Priority|**300**|
    |Nom|**AllowCidrBlockRDPInbound**|

1. Dans la page **Créer un groupe de sécurité réseau**, sélectionnez **OK**.
1. De retour dans l’onglet **Mise en réseau** de la page **Créer une machine virtuelle**, sélectionnez **Suivant : Gestion >**  :
1. Dans l’onglet **Gestion** de la page **Créer une machine virtuelle**, spécifiez les paramètres suivants et sélectionnez **Suivant : Surveillance >** (laissez les autres paramètres avec leur valeur par défaut) :

    |Paramètre|Value|
    |---|---|
    |Activer gratuitement le plan De base|disabled|
    |Options d’orchestration des patchs|**Mises à jour manuelles**|

1. Dans l’onglet **Surveillance** de la page **Créer une machine virtuelle**, spécifiez les paramètres suivants et sélectionnez **Examiner et créer** (laissez les autres paramètres avec leur valeur par défaut) :

    |Paramètre|Valeur|
    |---|---|
    |Diagnostics de démarrage|**Disable**|

1. Dans l’onglet **Examiner et créer** de la page **Créer une machine virtuelle**, sélectionnez **Créer**.

    > **Remarque** : Attendez la fin du déploiement. Le déploiement peut prendre environ 5 minutes.

1. Sur l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Machines virtuelles** et, dans la page **Machines virtuelles**, sélectionnez **az140-111e-vm0**.
1. Dans la page **az140-111e-vm0**, sélectionnez **Se connecter** et, dans le menu déroulant, sélectionnez **Se connecter**.
1. Dans la page **Se connecter az140-111e-vm0 \|**, dans la section **Le plus courant**, sélectionnez **Télécharger le fichier RDP**.
1. Dans la fenêtre contextuelle **Télécharger**, sélectionnez **Conserver**, puis sélectionnez **Ouvrir le fichier**.
1. Lorsque vous y êtes invité, sélectionnez **Se connecter**, puis, dans la boîte de dialogue **Sécurité Windows**, entrez le nom d’utilisateur et le mot de passe que vous avez spécifiés lors du déploiement de la machine virtuelle Azure.
1. Lorsque vous êtes invité à confirmer, sélectionnez à nouveau **Se connecter**.
1. Dans la session Bureau à distance vers **az140-111e-vm0**, choisissez et acceptez vos paramètres de confidentialité préférés.
1. Dans la session Bureau à distance vers **az140-111e-vm0**, démarrez Microsoft Edge, accédez à la page [Se connecter à Azure Virtual Desktop à l’aide du client Bureau à distance pour Windows](https://learn.microsoft.com/en-us/azure/virtual-desktop/users/connect-windows), faites défiler jusqu’à la section **Télécharger et installer le client Bureau à distance (MSI)** et sélectionnez le lien [Windows 64 bits](https://go.microsoft.com/fwlink/?linkid=2139369). 
1. Ouvrez Explorateur de fichiers, accédez au dossier **Téléchargements** et lancez l’installation du fichier MSI nouvellement téléchargé. 
1. Lorsque vous y êtes invité, acceptez les conditions du contrat de licence et choisissez l’option **Installer pour tous les utilisateurs de cet ordinateur**. Si vous y êtes invité, acceptez l’invite de contrôle de compte d’utilisateur pour poursuivre l’installation. 
1. Une fois l’installation terminée, vérifiez que la case **Lancer le Bureau à distance à la fin de l’installation** est cochée, puis sélectionnez **Terminer** pour démarrer le client Bureau à distance Microsoft.
1. Dans la session Bureau à distance vers **az140-111e-vm0**, dans la fenêtre du client **Bureau à distance**, sélectionnez **S’abonner** et, lorsque vous y êtes invité, connectez-vous avec les informations d’identification du compte d’utilisateur Entra ID `User2` que vous pouvez trouver dans l’onglet **Ressources** dans le volet droit de la fenêtre d’interface de labo.

   > **Note** : sélectionnez le compte d’utilisateur qui est membre du groupe Entra avec le préfixe **AVD-RemoteApp**.

1. Vérifiez que la page **Bureau à distance** affiche quatre icônes, notamment l’invite de commandes, Microsoft Word, Microsoft Excel et Microsoft PowerPoint. 
1. Double-cliquez sur l’icône d’invite de commandes. 
1. Lorsque vous êtes invité à vous connecter, dans la boîte de dialogue **Sécurité Windows**, entrez le mot de passe du même compte d’utilisateur Microsoft Entra que vous avez utilisé pour vous connecter à l’environnement Azure Virtual Desktop cible.
1. Vérifiez qu’une fenêtre d’**invite de commandes** s’affiche peu de temps après. 
1. À l’invite de commandes, tapez **logoff**, puis appuyez sur l'**Entrée** touche pour vous déconnecter de la session d’application distante actuelle.

   > **Note** : si vous le souhaitez, vous pouvez envisager de tenter de vous abonner au flux et de vous connecter à l’espace de travail Azure Virtual Desktop à partir de l’ordinateur de labo pour vérifier que cette connexion échouera. 

#### Tâche 7 : autoriser l’accès au réseau public à un pool d’hôtes et à un espace de travail

1. Sur l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop** et, dans la page **Azure Virtual Desktop**, sélectionnez **Espaces de travail**.
1. Dans la page **Espaces de travail Azure Virtual Desktop \|**, sélectionnez **az140-21-ws1**.
1. Dans la page **az140-21-ws1**, dans le menu de navigation vertical, dans la section **Paramètres**, sélectionnez **Mise en réseau**.
1. Dans la page **Mise en réseau az140-21-ws1 \|**, dans l’onglet **Accès public**, sélectionnez l’option **Activer l’accès public à partir de tous les réseaux**, puis sélectionnez **Enregistrer**.
1. À partir de l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop**, dans la page **Azure Virtual Desktop**, dans la section **Gérer** du menu de navigation vertical, sélectionnez **Pools d’hôtes** et, dans la page **Pools d’hôtes Azure Virtual Desktop \|**, sélectionnez **az140-21-hp1**. 
1. Dans la page **az140-21-hp1**, dans le menu de navigation vertical, dans la section **Paramètres**, sélectionnez **Mise en réseau**.
1. Dans la page **Mise en réseau az140-21-hp1 \|**, dans l’onglet **Accès public**, sélectionnez l’option **Activer l’accès public à partir de tous les réseaux**, puis sélectionnez **Enregistrer**.
