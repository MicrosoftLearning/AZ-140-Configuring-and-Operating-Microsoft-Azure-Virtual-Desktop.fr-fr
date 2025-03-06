---
lab:
  title: "Labo\_: créer des images d’hôte de session personnalisées à l’aide de modèles d’images"
  module: 'Module 1.5: Create and manage session host images'
---

# Labo - Créer des images d’hôte de session personnalisées à l’aide de modèles d’images
# Manuel de labo de l’étudiant

## Dépendances du labo

- Un abonnement Azure que vous allez utiliser dans ce labo.
- Un compte d’utilisateur Microsoft Entra avec le rôle Propriétaire dans l’abonnement Azure que vous allez utiliser dans ce labo et avec les autorisations suffisantes pour joindre des appareils au locataire Entra associé à cet abonnement Azure.

## Durée estimée

90 minutes (le temps nécessaire à la génération de l’image est de 45 minutes)

## Scénario de labo

Vous souhaitez implémenter un environnement Azure Virtual Desktop. Vous devez utiliser des images de machine virtuelle personnalisées lors du déploiement d’hôtes de session Azure Virtual Desktop.

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

- Utiliser des modèles d’images personnalisés pour créer des images personnalisées d’hôtes de session dans Azure Virtual Desktop

## Fichiers du labo

- Aucun

## Instructions

### Exercice 1 : créer des images d’hôtes de session personnalisées à l’aide de modèles d’image

Les principales tâches de cet exercice sont les suivantes

1. Inscrire les fournisseurs de ressources requis
1. Créer une identité managée attribuée par l’utilisateur
1. Créer un rôle personnalisé de contrôle d’accès en fonction du rôle (RBAC) Azure
1. Définir des autorisations sur les ressources associées à l’approvisionnement d’images hôtes
1. Créer une définition d’image dans une instance Azure Compute Gallery
1. Créer un modèle d’image personnalisée
1. Générer une image personnalisée
1. Déployer des hôtes de session à l’aide d’une image personnalisée

> **Note** : avant de pouvoir créer un modèle d’image personnalisé, vous devez satisfaire un certain nombre de conditions préalables, notamment :

- Inscrire tous les fournisseurs de ressources requis
- Créer une identité managée attribuée par l’utilisateur
- Octroyer les autorisations requises par l’identité managée affectée par l’utilisateur à l’aide d’un rôle de contrôle d’accès en fonction du rôle (RBAC) Azure.
- Si vous envisagez de distribuer l’image à l’aide d’Azure Compute Gallery, vous devez créer son instance avec une définition d’image

#### Tâche 1 : inscrire les fournisseurs de ressources requis

1. Si besoin, à partir de votre ordinateur de labo, démarrez un navigateur web, accédez au portail Azure, puis connectez-vous en fournissant des informations d’identification d’un compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utiliserez dans ce labo.

    > **Note** : utilisez les informations d’identification du compte `User1-` répertorié dans l’onglet Ressources sur le côté droit de la fenêtre de session de labo.

1. Dans le portail Azure, démarrez une session PowerShell dans Azure Cloud Shell.

    > **Note** : si vous y êtes invité, dans le volet **Mise en route**, dans la liste déroulante **Abonnement**, sélectionnez le nom de l’abonnement Azure que vous utilisez dans ce labo, puis sélectionnez **Appliquer**.

1. Dans la session PowerShell du volet Azure Cloud Shell, exécutez la commande suivante pour inscrire le fournisseur de ressources **Microsoft.DesktopVirtualization** :

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
    Register-AzResourceProvider -ProviderNamespace Microsoft.VirtualMachineImages
    Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
    Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
    Register-AzResourceProvider -ProviderNamespace Microsoft.Network
    Register-AzResourceProvider -ProviderNamespace Microsoft.KeyVault
    Register-AzResourceProvider -ProviderNamespace Microsoft.ContainerInstance
    ```

    > **Note** : n’attendez pas la fin de l’inscription. Ceci peut prendre environ 5 minutes.

1. Fermez le volet Azure Cloud Shell.

#### Tâche 2 : créer une identité managée affectée par l’utilisateur

1. À partir de l’ordinateur de labo, dans la fenêtre du navigateur web affichant le portail Azure, recherchez et sélectionnez **Identités managées**.
1. Dans la page **Identités managés**, sélectionnez **+ Créer**.
1. Dans l’onglet **Informations de base** de la page **Créer une identité managée affectée par l’utilisateur**, spécifiez les paramètres suivants, puis sélectionnez **Examiner et créer** :

    > **Note** : lorsque vous définissez la valeur **Name** , basculez vers l’onglet Ressources situé à droite de la fenêtre de session de labo et identifiez la chaîne de caractères entre *User1-* et le caractère *@*. Utilisez cette chaîne pour remplacer l’espace réservé *aléatoire*.

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|Nom d’un nouveau groupe de ressources **az140-15a-RG**|
    |Région|Nom de la région Azure dans laquelle déployer votre environnement Azure Virtual Desktop|
    |Nom|**az140**-*random*-**uami**|

1. Sous l’onglet **Review + create (Vérifier + créer)** , sélectionnez **Créer**.

    >**Note** : attendez la fin de l’approvisionnement de l’identité managée affectée par l’utilisateur. Cela devrait prendre juste quelques secondes.

#### Tâche 3 : créer un rôle de contrôle d’accès en fonction du rôle Azure personnalisé (RBAC)

>**Note** : le rôle de contrôle d’accès en fonction du rôle Azure personnalisé (RBAC) sera utilisé pour attribuer les autorisations appropriées à l’identité managée affectée par l’utilisateur créée dans la tâche précédente.

1. Sur l’ordinateur de labo, dans la fenêtre de navigateur web affichant le portail Azure, ouvrez une session PowerShell dans le volet Azure Cloud Shell.

1. Dans la session PowerShell du volet Azure Cloud Shell, exécutez la commande suivante pour identifier la valeur de la propriété **ID** de l’abonnement Azure utilisée pour ce labo et stockez-la dans la variable **$subscriptionId** :

    ```powershell
    $subscriptionId = (Get-AzSubscription).Id
    ```

1. Exécutez la commande suivante pour créer la définition de rôle du nouveau rôle personnalisé, y compris sa valeur d’étendue assignable, et stockez-la dans la variable **$jsonContent** (veillez à remplacer l’espace réservé *aléatoire* par la même chaîne que celle que vous avez identifiée dans la tâche précédente) :

    ```powershell
    $jsonContent = @"
    {
      "Name": "Desktop Virtualization Image Creator (random)",
      "IsCustom": true,
      "Description": "Create custom image templates for Azure Virtual Desktop images.",
      "Actions": [
        "Microsoft.Compute/galleries/read",
        "Microsoft.Compute/galleries/images/read",
        "Microsoft.Compute/galleries/images/versions/read",
        "Microsoft.Compute/galleries/images/versions/write",
        "Microsoft.Compute/images/write",
        "Microsoft.Compute/images/read",
        "Microsoft.Compute/images/delete"
      ],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/subscriptions/$subscriptionId",
        "/subscriptions/$subscriptionId/resourceGroups/az140-15b-RG"
      ]
    }
    "@
    ```

1. Exécutez la commande suivante pour stocker le contenu de la variable **$jsonContent** dans un fichier nommé **CustomRole.json** :

    ```powershell
    $jsonContent | Out-File -FilePath 'CustomRole.json'
    ```

1. Exécutez la commande suivante pour créer le nouveau rôle personnalisé :

    ```powershell
    New-AzRoleDefinition -InputFile ./CustomRole.json
    ```

1. Fermez le volet Azure Cloud Shell.

#### Tâche 4 : définir des autorisations sur les ressources associées à l’approvisionnement d’images hôtes

1. À partir de l’ordinateur du labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Groupes de ressources**, puis, dans la page **Groupes de ressources**, sélectionnez **+ Créer**.
1. Sous l’onglet **Informations de base** de la page **Créer un groupe de ressources**, spécifiez les paramètres suivants et sélectionnez **Examiner et créer** :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|Nom d’un nouveau groupe de ressources **az140-15b-RG**|
    |Région|Nom de la région Azure dans laquelle déployer votre environnement Azure Virtual Desktop|

1. Sous l’onglet **Review + create (Vérifier + créer)** , sélectionnez **Créer**.
1. Actualisez la page **Groupes de ressources** et, dans la liste des groupes de ressources, sélectionnez **az140-15b-RG**.
1. Dans la page **az140-15b-RG**, dans le menu de navigation vertical, sélectionnez **Contrôle d’accès (IAM)**.
1. Dans la page **Contrôle d’accès (IAM) az140-15b-RG \|**, sélectionnez **+ Ajouter**, puis dans le menu déroulant, sélectionnez **Ajouter une attribution de rôle**.
1. Dans l’onglet **Rôle** de la page **Ajouter une attribution de rôle**, vérifiez que l’onglet **Rôles de fonction de tâche** est sélectionné, dans la zone de texte de recherche, entrez **Desktop Virtualization Image Creator** (*random*), dans la liste des résultats, sélectionnez **Desktop Virtualization Image Creator** (*aléatoire*), puis sélectionnez **Suivant**.

    >**Note** : veillez à remplacer l’espace réservé *random* par la même chaîne que celle que vous avez utilisée lors de la définition du nouveau rôle RBAC personnalisé.

1. Dans l’onglet **Membres** de la page **Ajouter une attribution de rôle**, sélectionnez l’option **Identité managée**, cliquez sur **+ Sélectionner des membres**, dans le volet **Sélectionner des identités managées**, dans la liste déroulante **Identité managée**, sélectionnez **Identité managée affectée par l’utilisateur**, dans la liste des identités managées affectées par l’utilisateur, sélectionnez **az140**-*random*-**uami** (où l’espace réservé *random* représente la même chaîne que celle que vous avez utilisée lors de la définition du nouveau rôle RBAC personnalisé), puis cliquez sur **Sélectionner**.
1. De retour dans l’onglet **Membres** de la page **Ajouter une attribution de rôle**, sélectionnez **Vérifier + attribuer**.
1. Sous l’onglet **Vérifier + attribuer**, sélectionnez **Vérifier + attribuer**. 

#### Tâche 5 : créer une définition d’image dans une instance Azure Compute Gallery

1. À partir de l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez les **galeries Azure Compute**, puis, dans la page **galeries Azure Compute**, sélectionnez **+ Créer**.
1. Dans l’onglet **Informations de base** de la page **Créer une galerie Azure Compute**, spécifiez les paramètres suivants, puis sélectionnez **Suivant : Méthode de partage** :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|**az140-15b-RG**|
    |Nom|**az14015computegallery**|
    |Région|Nom de la région Azure dans laquelle déployer votre environnement Azure Virtual Desktop|

1. Dans l’onglet **Partage** de la page **Créer une galerie Azure Compute**, conservez l’option par défaut **Contrôle d’accès en fonction du rôle (RBAC)**, puis sélectionnez **Examiner et créer**.
1. Sous l’onglet **Review + create (Vérifier + créer)** , sélectionnez **Créer**.

    >**Remarque** : Attendez la fin du processus de provisionnement. Cela devrait prendre moins d’une minute.

1. À partir de l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez les **galeries Azure Compute** et, dans la page des **galeries Azure Compute**, sélectionnez **az14015computegallery**. 
1. Dans la page **az14015computegallery**, sélectionnez **+ Ajouter** et, dans le menu déroulant, sélectionnez **+ Définition d’image de machine virtuelle**. 
1. Dans l’onglet **Informations de base** de la page **Créer une définition d’image de machine virtuelle**, spécifiez les paramètres suivants et sélectionnez **Suivant : Version** (conservez les valeurs par défaut de tous les autres paramètres) :

    |Paramètre|Valeur|
    |---|---|
    |Région|Nom de la région Azure dans laquelle déployer votre environnement Azure Virtual Desktop|
    |Nom de la définition d’image de machine virtuelle|**az14015imagedefinition**|
    |Type de système d’exploitation|**Windows**|
    |Type de sécurité|**Lancement fiable pris en charge**|
    |État du système d’exploitation|**Généralisé**|
    |Éditeur|**MicrosoftWindowsDesktop**|
    |Offre|**Windows 11**|
    |Référence (SKU)|**win11-23h2-avd-m365**|

    > **Note** : la génération de machine virtuelle est automatiquement définie sur Gen2, car les machines virtuelles Gen 1 ne sont pas prises en charge avec le type de sécurité confidentiel et avec lancement fiable.

1. Dans l’onglet **Version** de la page **Créer une définition d’image de machine virtuelle**, conservez les paramètres inchangés et sélectionnez **Suivant : Options de publication**.

    > **Note** : vous ne devez pas créer la version de l’image de machine virtuelle à ce stade. Azure Virtual Desktop s’en chargera.

1. Dans l’onglet **Options de publication** de la page **Créer une définition d’image de machine virtuelle**, laissez les paramètres inchangés et sélectionnez **Examiner et créer**.
1. Dans l’onglet **Examiner et créer** de la page **Créer une définition d’image de machine virtuelle**, sélectionnez **Créer**.

    > **Remarque** : Attendez la fin du processus de provisionnement. Cette étape prend généralement moins d’une minute.

#### Tâche 6 : créer un modèle d’image personnalisé

1. À partir de l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop**, dans la page **Azure Virtual Desktop**, puis dans la section **Gérer** du menu de navigation vertical, sélectionnez **Modèles d’image personnalisés**, et dans la page **Modèles d’images personnalisés Azure Virtual Desktop \|**, sélectionnez **+ Ajouter un modèle d’image personnalisé**. 
1. Dans l’onglet **Informations de base** du panneau **Créer un modèle d’image personnalisé**, spécifiez les paramètres suivants et sélectionnez **Suivant** :

    > **Note** : lorsque vous définissez la propriété **Identité managée**, veillez à remplacer l’espace réservé *aléatoire* par la même chaîne que celle que vous avez identifiée précédemment dans cet exercice.

    |Paramètre|Valeur|
    |---|---|
    |Nom du modèle|**az140-15b-imagetemplate**|
    |Importer depuis un modèle existant|**Non**|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|**az140-15b-RG**|
    |Emplacement|Nom de la région Azure dans laquelle déployer votre environnement Azure Virtual Desktop|
    |Identité managée|**az140**-*random*-**uami**|

1. Dans l’onglet **Image source** de la page **Créer un modèle d’image personnalisé**, spécifiez les paramètres suivants, puis sélectionnez **Suivant** :

    |Paramètre|Valeur|
    |---|---|
    |Type de source|**Image de plateforme (place de marché)**|
    |Sélectionner une image|**Windows 11 Entreprise multisession, version 23H2 + Microsoft 365 Apps**|

1. Dans l’onglet **Informations de base** de la page **Créer un modèle d’image personnalisé**, spécifiez les paramètres suivants (conservez les valeurs par défaut de tous les autres paramètres) et sélectionnez **Suivant** :

    |Paramètre|Valeur|
    |---|---|
    |Azure Compute Gallery|activé|
    |Nom de la galerie|**az14015computegallery**|
    |Définition d’image de galerie|**az14015imagedefinition**|
    |Version de l’image de galerie|**1.0.0**|
    |Nom de la sortie d’exécution|**az140-15-image-1.0.0**|
    |Régions de réplication|Nom de la région Azure dans laquelle déployer votre environnement Azure Virtual Desktop|
    |Exclure de la plus récente|**Non**|
    |Type de compte de stockage|**Standard_LRS**|

    > **Note** : vous pouvez utiliser la propriété **Régions de réplication** pour prendre en charge les builds multirégions. Définir l’option **Exclure de la dernière version** sur **Oui** empêcherait l’utilisation de cette version d’image lorsque **la dernière version** est spécifiée comme version de l’élément **ImageReference** lors de la création de la machine virtuelle.

1. Dans l’onglet **Propriétés de build** de la page **Créer un modèle d’image personnalisé**, spécifiez les paramètres suivants (conservez les valeurs par défaut de tous les autres paramètres) et sélectionnez **Suivant** :

    |Paramètre|Valeur|
    |---|---|
    |Délai d’expiration de la génération|**120**|
    |Taille de la machine virtuelle de génération du modèle|**Standard_DC2s_v3**|
    |Taille du disque du système d’exploitation (Go)|**127**|
    |Groupe intermédiaire|**az140-15c-RG**|
    |Réseau virtuel|Ne pas définir|

    > **Note** : le **groupe intermédiaire** est le groupe de ressources utilisé pour effectuer une copie intermédiaire des ressources pour générer l’image et stocker les journaux d’activité. Si vous n’en fournissez pas, le nom est généré automatiquement. Si le nom du **réseau virtuel** n’est pas défini, un nom temporaire est créé, ainsi qu’une adresse IP publique pour la machine virtuelle utilisée pour créer le build.

    > **Important** : vérifiez que vous disposez d’un nombre suffisant de processeurs virtuels disponibles pour la taille de machine virtuelle de build que vous avez spécifiée. Si ce n’est pas le cas, choisissez une taille différente ou demandez une augmentation du quota.

1. Dans l’onglet **Personnalisation** de la page **Créer un modèle d’image personnalisé**, sélectionnez **+ Ajouter un script intégré**. 
1. Dans le volet **Sélectionner des scripts intégrés**, passez en revue les options disponibles regroupées dans des scripts spécifiques au système d’exploitation, des scripts Azure Virtual Desktop, des scripts MSIX App Attach, des scripts d’application et des scripts liés aux mises à jour Windows, puis sélectionnez les entrées suivantes :

   - **Redirection de fuseau horaire** : permet aux clientx d’utiliser leur fuseau horaire dans une session sur les hôtes de session
   - **Désactiver l’assistant Stockage** : empêche l’assistant Stockage d’affecter négativement les hôtes de session en détectant de manière erronée les conditions d’espace disque libre faible
   - **Activez la protection contre la capture d’écran** grâce au **bloage de capture d’écran sur le client et le serveur** : bloque ou masque le contenu distant dans les captures d’écran et le partage d’écran

1. Dans le volet **Sélectionner des scripts intégrés**, sélectionnez **Enregistrer**.

    > **Note** : vous pouvez ajouter vos propres scripts. Pour obtenir des exemples, envisagez de référencer les scripts intégrés, tels que [Redirection de fuseau horaire](https://raw.githubusercontent.com/Azure/RDS-Templates/master/CustomImageTemplateScripts/CustomImageTemplateScripts_2024-03-27/TimezoneRedirection.ps1), [Désactiver l’assistant Stockage](https://raw.githubusercontent.com/Azure/RDS-Templates/master/CustomImageTemplateScripts/CustomImageTemplateScripts_2024-03-27/DisableStorageSense.ps1) ou [Activez la protectioncontre la capture d’écran](https://raw.githubusercontent.com/Azure/RDS-Templates/master/CustomImageTemplateScripts/CustomImageTemplateScripts_2024-03-27/ScreenCaptureProtection.ps1).

1. De retour dans l’onglet **Personnalisation** de la page **Créer un modèle d’image personnalisé**, sélectionnez **Suivant**.
1. Dans l’onglet **Balises** de la page **Créer un modèle d’image personnalisé**, sélectionnez **Suivant**.
1. Dans l’onglet **Examiner et créer** de la page **Créer un modèle d’image personnalisé**, sélectionnez **Créer**.

    > **Note** : attendez que le modèle soit créé. Cette opération peut prendre quelques minutes. Actualisez la page **Modèles d’images personnalisés Azure Virtual Desktop \|** pour passer en revue le statut du modèle.

#### Tâche 7 : créer une image personnalisée

> **Note** : les tâches restantes de ce labo sont facultatives, car elles impliquent un temps d’attente plutôt important. 

1. À partir de l’ordinateur de labo, dans le navigateur web affichant le portail Azure, dans la page **Modèle d’image personnalisé Azure Virtual Desktop \|**, sélectionnez **az140-15b-imagetemplate**.
1. Dans la page **az140-15b-imagetemplate**, sélectionnez **Démarrer le build**.

    > **Note** : attendez que le build soit créé. Le temps réel d’exécution du processus de génération de build peut varier, mais avec les paramètres fournis dans les instructions du labo, il doit se terminer dans les 45 minutes. Actualisez la page toutes les quelques minutes et surveillez la valeur **État d’exécution de build** dans la section **Essentials** de la page **az140-15b-imagetemplate**. 

    > **Note** : l’état de l’exécution de build doit changer à un moment donné de **En cours d’exécution - Génération** à **En cours d’exécution - Distribution** et enfin à **Opération réussie**.

    > **Note** : en attendant la fin de la génération de build, passez en revue le contenu du groupe de ressources intermédiaire **az140-15c-RG**, où les ressources de build, y compris la machine virtuelle du build, un réseau virtuel, un groupe de sécurité réseau, le coffre de clés, l’instantané, l’instance de conteneur et le compte de stockage sont automatiquement provisionnés. 

1. À partir de l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Groupes de ressources**, puis, dans la page **Groupes de ressources**, sélectionnez **az140-15c-RG**.
1. Dans la page **az140-15c-RG**, dans la section **Ressources**, prenez note des ressources provisionnées automatiquement.
1. Revenez à la page **az140-15b-imagetemplate** et surveillez la progression de la génération de build. 

    > **Note** : vous pouvez également utiliser le **Journal d’activité** pour suivre l’achèvement du processus de génération de build. L’action sur laquelle vous devez vous concentrer est **Exécuter un modèle d’image de machine virtuelle pour produire sa sortie**. Son état doit changer à un moment donné entre **Accepté** et **Opération réussie**.

1. Une fois le build terminé, à partir de l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez les **galeries Azure Compute** et, dans la page des **galeries Azure Compute**, sélectionnez **az14015computegallery**. 
1. Dans **az14015computegallery**, dans l’onglet **Définitions**, sélectionnez **az14015imagedefinition**.
1. Dans la page **az14015imagedefinition**, dans l’onglet **Versions**, passez en revue les informations sur l’image **1.0.0 (dernière version)**.

#### Tâche 8 : déployer des hôtes de session à l’aide d’une image personnalisée

> **Note** : si vous le souhaitez, envisagez de passer en revue les étapes initiales du déploiement d’hôtes de session Azure Virtual Desktop à l’aide de l’image personnalisée que vous avez créée. 

1. Sur l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Réseaux virtuels** et, dans la page **Réseaux virtuels**, sélectionnez **Créer +**.
1. Sous l’onglet **Informations de base** du panneau **Créer un réseau virtuel**, spécifier les paramètres suivants et sélectionner **Suivant** :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|Nom d’un nouveau groupe de ressources **az140-15d-RG**|
    |Nom du réseau virtuel|**az140-vnet15d**|
    |Région|Nom de la région Azure dans laquelle vous souhaitez déployer l’environnement Azure Virtual Desktop|

1. Sous l’onglet **Sécurité**, acceptez les paramètres par défaut et sélectionnez **Suivant**.
1. Dans l’onglet **Adresses IP**, spécifiez les paramètres suivants :

    |Paramètre|Valeur|
    |---|---|
    |Espace d’adressage IP|**10.30.0.0/16**|

1. Sélectionnez l’icône Modifier (crayon) en regard de l’entrée de sous-réseau **par défaut**, dans le volet **Modifier**, spécifiez les paramètres suivants (laissez les autres avec leurs valeurs existantes) et sélectionnez **Enregistrer** :

    |Paramètre|Valeur|
    |---|---|
    |Nom|**hp1-Subnet**|
    |Adresse de début|**10.30.1.0**|
    |Activer le sous-réseau privé (aucun accès sortant par défaut)|Désactivé|

1. De retour dans l’onglet **Adresses IP**, sélectionnez **Examiner et créer**, puis choisissez **Créer**.

    > **Remarque** : Attendez la fin du processus de provisionnement. Cette étape prend généralement moins d’une minute.

1. Sur l’ordinateur de labo, dans la fenêtre de navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop**, dans la page **Azure Virtual Desktop**, dans la section **Gérer**, sélectionnez **Pools d’hôtes**, puis, dans la page **Pools d’hôtes Azure Virtual Desktop \|**, sélectionnez **+ Créer**. 
1. Dans l’onglet **Informations de base** de la page **Créer un pool d’hôtes**, spécifiez les paramètres suivants et sélectionnez **Suivant : Hôtes de session >** (laissez les autres paramètres avec leurs valeurs par défaut) :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|**az140-15d-RG**|
    |Nom du pool d’hôtes|**az140-15-hp1**|
    |Emplacement|Nom de la région Azure dans laquelle déployer votre environnement Azure Virtual Desktop|
    |Environnement de validation|**Aucun**|
    |Type de groupe d’applications préféré|**Bureau**|
    |Type de pool d’hôtes|**Groupé**|
    |Créer une configuration d’hôte de session|**Non**|
    |Algorithme d’équilibrage de charge|**À largeur prioritaire**|

    > **Note** : lors de l’utilisation de l’algorithme d’équilibrage de charge en largeur d’abord, le paramètre de limite de session maximale est facultatif.

1. Dans l’onglet **Hôtes de session** de la page **Créer un pool d’hôtes**, spécifiez les paramètres suivants (laissez les autres paramètres avec leurs valeurs par défaut) :

    > **Note** : lorsque vous définissez la valeur du **préfixe de nom**, basculez vers l’onglet Ressources situé à droite de la fenêtre de session de labo et identifiez la chaîne de caractères entre *User1-* et le caractère *@*. Utilisez cette chaîne pour remplacer l’espace réservé *aléatoire*.

    |Paramètre|Valeur|
    |---|---|
    |Ajouter des machines virtuelles|**Oui**|
    |Resource group|**Par défaut, identique à celui du pool d’hôtes**|
    |Préfixe de nom|**sh0**_random_|
    |Type de machine virtuelle|**Machine virtuelle Azure**|
    |Emplacement des machines virtuelles|Nom de la région Azure dans laquelle déployer votre environnement Azure Virtual Desktop|
    |Options de disponibilité|**Aucune redondance de l’infrastructure requise**|
    |Type de sécurité|**Lancement fiable des machines virtuelles**|

1. Dans l’onglet **Machines virtuelles** de la page **Créer un pool d’hôtes**, dans la liste déroulante **Image**, sélectionnez **Voir toutes les images**.
1. Dans la page **Sélectionner une image**, sélectionnez **Images partagées** et, dans la liste des images, sélectionnez **az14015imagedefinition**. 
1. De retour dans l’onglet **Machines virtuelles** de la page **Créer un pool d’hôtes**, spécifiez les paramètres suivants et sélectionnez **Suivant : Espace de travail >** (laissez les autres paramètres avec leurs valeurs par défaut) :

    |Paramètre|Valeur|
    |---|---|
    |Taille de la machine virtuelle|**Standard D2s v3**|
    |Nombre d'ordinateurs virtuels|**1**|
    |Type de disque du système d’exploitation|**SSD Standard**|
    |Taille du disque de système d’exploitation|**Taille par défaut (128 Gio)**|
    |Diagnostics de démarrage|**Activer avec le compte de stockage managé (recommandé)**|
    |Réseau virtuel|az140-vnet15d|
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

    > **Remarque** : Attendez la fin du déploiement. Ceci peut prendre environ de 10 à 15 minutes.
