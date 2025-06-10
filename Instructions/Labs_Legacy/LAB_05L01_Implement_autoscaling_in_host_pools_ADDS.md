---
lab:
  title: "Labo : Implémenter la mise à l’échelle automatique dans les pools d’hôtes (AD\_DS)"
  module: 'Module 5: Monitor and Maintain an AVD Infrastructure'
---

# Labo - Implémenter la mise à l’échelle automatique dans les pools d’hôtes (AD DS)
# Manuel de labo de l’étudiant

## Dépendances du labo

- Un abonnement Azure que vous allez utiliser dans ce labo.
- Un compte Microsoft ou Microsoft Entra avec le rôle Propriétaire ou Contributeur dans l’abonnement Azure que vous allez utiliser dans ce labo et avec le rôle Administrateur général dans le locataire Microsoft Entra associé à cet abonnement Azure.
- Le labo terminé **Préparer le déploiement d’Azure Virtual Desktop (AD DS)**
- Le labo terminé **Déployer des pools d’hôtes et des hôtes de session à l’aide du Portail Azure (AD DS)**

## Durée estimée

60 minutes

## Scénario du labo

Vous devez configurer la mise à l’échelle automatique des hôtes de session Azure Virtual Desktop dans un environnement Active Directory Domain Services (AD DS).

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

- Configurer la mise à l’échelle automatique des hôtes de session Azure Virtual Desktop
- Vérifier la mise à l’échelle automatique des hôtes de session Azure Virtual Desktop

## Fichiers du labo

- Aucun

## Instructions

### Exercice 1 : Configurer la mise à l’échelle automatique des hôtes de session Azure Virtual Desktop

Les principales tâches de cet exercice sont les suivantes

1. Préparer la mise à l’échelle des hôtes de session Azure Virtual Desktop
2. Configurer les diagnostics pour suivre la mise à l’échelle automatique d’Azure Virtual Desktop
3. Créer un plan de mise à l’échelle des hôtes de session Azure Virtual Desktop

#### Tâche 1 : Préparer la mise à l’échelle des hôtes de session Azure Virtual Desktop

1. Sur votre ordinateur de labo, démarrez un navigateur web, accédez au [portail Azure](https://portal.azure.com), puis connectez-vous en utilisant les informations d’identification d’un compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utiliserez dans ce labo.
1. Sur l’ordinateur de labo, dans la fenêtre de navigateur web affichant le portail Azure, ouvrez une session **PowerShell** dans le volet **Cloud Shell**.

   >**Remarque** : Les pools d’hôtes que vous envisagez d’utiliser avec la mise à l’échelle automatique doivent être configurés avec une valeur autre que la valeur par défaut du paramètre **MaxSessionLimit**. Vous pouvez définir cette valeur dans les paramètres du pool d’hôtes dans le portail Azure ou en exécutant la cmdlet Azure PowerShell **Update-AzWvdHostPool**, comme dans cet exemple. Vous pouvez également la définir explicitement lors de la création d’un pool dans le portail Azure ou lors de l’exécution de la cmdlet Azure PowerShell **New-AzWvdHostPool**.

1. À partir de la session PowerShell du volet Cloud Shell, exécutez la commande suivante pour définir la valeur du paramètre **MaxSessionLimit** du pool d’hôtes **az140-21-hp1** sur **2** : 

   ```powershell
   Update-AzWvdHostPool -ResourceGroupName 'az140-21-RG' `
   -Name az140-21-hp1 `
   -MaxSessionLimit 2
   ```

   >**Remarque** : Dans ce labo, la valeur du paramètre **MaxSessionLimit** est volontairement basse afin de faciliter le déclenchement du comportement de mise à l’échelle automatique.

   >**Remarque** : Avant de créer votre premier plan de mise à l’échelle, vous devez affecter le rôle RBAC **Contributeur de mise hors tension de la virtualisation de bureau** à Azure Virtual Desktop avec votre abonnement Azure comme étendue cible. 

1. Dans la fenêtre de navigateur affichant le portail Azure, fermez le volet Cloud Shell.
1. Dans le portail Azure, recherchez et sélectionnez **Abonnements**, puis dans la liste des abonnements, sélectionnez celui qui contient les ressources Azure Virtual Desktop. 
1. Dans la page des abonnements, sélectionnez **Contrôle d’accès (IAM)**.
1. Dans la page **Contrôle d’accès (IAM)**, dans la barre d’outils, sélectionnez le bouton **+ Ajouter**, puis sélectionnez **Ajout de l’attribution de rôle** dans le menu déroulant.
1. Dans le panneau **Ajouter une attribution de rôle**, sous l’onglet **Rôle**, spécifiez les paramètres suivants et sélectionnez **Suivant** :

   |Paramètre|Valeur|
   |---|---|
   |Rôle de fonction de travail|**Contributeur de mise hors tension de la virtualisation de bureau**|

1. Dans le panneau **Ajouter une attribution de rôle**, sous l’onglet **Membres**, cliquez sur **+ Sélectionner des membres**, spécifiez les paramètres suivants, puis cliquez sur **Sélectionner**. 

   |Paramètre|Valeur|
   |---|---|
   |Select|**Azure Virtual Desktop** ou **Windows Virtual Desktop**|

1. Dans le panneau **Ajouter une attribution de rôle**, sélectionnez **Vérifier + attribuer**

   >**Remarque** : La valeur dépend du moment où le fournisseur de ressources **Microsoft.DesktopVirtualization** a été inscrit pour la première fois dans votre locataire Azure.

1. Sous l’onglet **Vérifier + attribuer**, sélectionnez **Vérifier + attribuer**.

#### Tâche 2 : Configurer les diagnostics pour suivre la mise à l’échelle automatique d’Azure Virtual Desktop

1. Sur l’ordinateur de labo, dans la fenêtre de navigateur web affichant le portail Azure, ouvrez une session **PowerShell** dans le volet **Cloud Shell**.

   >**Remarque** : Vous allez utiliser un compte Stockage Azure pour stocker les événements de mise à l’échelle automatique. Vous pouvez le créer directement dans le portail Azure ou utiliser Azure PowerShell, comme illustré dans cette tâche.

1. Dans la session PowerShell du volet Cloud Shell, exécutez les commandes suivantes pour créer un compte Stockage Azure :

   ```powershell
   $resourceGroupName = 'az140-51-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName 'az140-11-RG').Location
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   $suffix = Get-Random
   $storageAccountName = "az140st51$suffix"
   New-AzStorageAccount -Location $location -Name $storageAccountName -ResourceGroupName $resourceGroupName -SkuName Standard_LRS
   ```

   >**Remarque** : Attendez que le compte de stockage soit provisionné.

1. Dans la fenêtre de navigateur affichant le portail Azure, fermez le volet Cloud Shell.
1. Sur votre ordinateur de labo, dans le navigateur affichant le portail Azure, accédez à la page du pool d’hôtes **az140-21-hp1**.
1. Dans la page **az140-21-hp1**, sélectionnez **Paramètres de diagnostic**, puis **+ Ajouter un paramètre de diagnostic**.
1. Dans la page **Paramètres de diagnostic**, dans la zone de texte **Nom du paramètre de diagnostic**, entrez **az140-51-scaling-plan-diagnostics**, puis dans la section **Groupes de catégories**, sélectionnez **Journaux de mise à l’échelle automatique pour les pools de hôte groupé**. 
1. Dans la même page, dans la section **Détails de la destination**, sélectionnez **Archiver dans un compte de stockage**, puis dans la liste déroulante **Compte de stockage**, sélectionnez le nom du compte de stockage qui commence par le préfixe **az140st51**.
1. Sélectionnez **Enregistrer**.

#### Tâche 3 : Créer un plan de mise à l’échelle des hôtes de session Azure Virtual Desktop

1. Sur votre ordinateur de labo, dans le navigateur affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop**. 
1. Dans la page **Azure Virtual Desktop**, sélectionnez **Plans de mise à l’échelle**, puis **+ Créer**.
1. Dans l’onglet **Informations de base** de l’assistant **Créer un plan de mise à l’échelle**, spécifiez les informations suivantes et sélectionnez **Suivant : Planifications >** (laissez les autres avec leurs valeurs par défaut) :

   |Paramètre|Valeur|
   |---|---|
   |Groupe de ressources|**az140-51-RG**|
   |Nom|**az140-51-scaling-plan**|
   |Emplacement|la même région Azure dans laquelle vous avez déployé les hôtes de session dans les labos précédents|
   |Nom convivial|**az140-51 scaling plan**|
   |Fuseau horaire|votre fuseau horaire local|

   >**Remarque** : Les étiquettes d’exclusion vous permettent de désigner un nom d’étiquette pour les hôtes de session que vous souhaitez exclure des opérations de mise à l’échelle. Par exemple, vous pouvez baliser les machines virtuelles configurées en mode maintenance afin que la mise à l’échelle automatique ne remplace pas le mode de maintenance pendant la maintenance à l’aide de l’étiquette d’exclusion « excludeFromScaling ». 

1. Sous l’onglet **Planifications** de l’Assistant **Création d’un plan de mise à l’échelle**, sélectionnez **+ Ajouter une planification**.
1. Sous l’onglet **Général** de l’Assistant **Ajouter une planification**, spécifiez les informations suivantes et cliquez sur **Suivant**.

   |Paramètre|Valeur|
   |---|---|
   |Nom de la planification|**az140-51-schedule**|
   |Répéter le|**7 sélectionné** (sélectionner tous les jours de la semaine)|

1. Sous l’onglet **Accélération** de l’Assistant **Ajouter une planification**, spécifiez les informations suivantes et cliquez sur **Suivant**.

   |Paramètre|Valeur|
   |---|---|
   |Heure de début (format 24 heures)|votre heure actuelle moins 9 heures|
   |Algorithme d’équilibrage de charge|**Largeur d’abord**|
   |Pourcentage minimal d’hôtes (%)|**20**|
   |Seuil de capacité (%)|**60**|

   >**Remarque** : La préférence d’équilibrage de charge que vous sélectionnez ici remplace celle que vous avez sélectionnée dans les paramètres de votre pool d’hôtes d’origine.

   >**Remarque** : Le pourcentage minimal d’hôtes désigne le pourcentage d’hôtes de session que vous souhaitez toujours conserver. Si le pourcentage que vous entrez n’est pas un nombre entier, il est arrondi au nombre entier supérieur le plus proche. 

   >**Remarque** : Le seuil de capacité représente le pourcentage de capacité disponible du pool d’hôtes qui déclenchera une action de mise à l’échelle. Par exemple, si deux hôtes de session dans le pool d’hôtes présentant une limite de session maximale de 20 sont activés, la capacité du pool d’hôtes disponible est de 40. Si vous définissez le seuil de capacité sur 75 % et que les hôtes de session possèdent plus de 30 sessions utilisateur, la mise à l’échelle automatique active un troisième hôte de session. Cela permet de modifier la capacité du pool d’hôtes disponible pour passer de 40 à 60.

1. Sous l’onglet **Heures de pointe** de l’Assistant **Ajouter une planification**, spécifiez les informations suivantes et cliquez sur **Suivant**.

   |Paramètre|Valeur|
   |---|---|
   |Heure de début (format 24 heures)|votre heure actuelle moins 8 heures|
   |Algorithme d’équilibrage de charge|**À profondeur prioritaire**|

   >**Remarque** : L’heure de début désigne l’heure de fin de la phase d’accélération.

   >**Remarque** : La valeur du seuil de capacité de cette phase est déterminée par la valeur de seuil de capacité d’accélération.

1. Sous l’onglet **Ralentissement** de l’Assistant **Ajouter une planification**, spécifiez les informations suivantes et cliquez sur **Suivant**.

   |Paramètre|Valeur|
   |---|---|
   |Heure de début (format 24 heures)|votre heure actuelle moins 2 heures|
   |Algorithme d’équilibrage de charge|**À profondeur prioritaire**|
   |Pourcentage minimal d’hôtes (%)|**10**|
   |Seuil de capacité (%)|**90**|
   |Forcer la déconnexion des utilisateurs|**Oui**|
   |Délai avant de déconnecter les utilisateurs et d’arrêter les machines virtuelles (min)|**30**|

   >**Remarque** : Si l’option **Forcer la déconnexion des utilisateurs** est activée, la mise à l’échelle automatique attribue à l’hôte de session le nombre le plus bas de sessions utilisateur en mode maintenance, envoie à toutes les sessions utilisateur actives une notification d’arrêt imminent et force leur déconnexion une fois que le délai spécifié est passé. Une fois que la mise à l’échelle automatique a déconnecté toutes les sessions utilisateur, elle libère la machine virtuelle. 

   >**Remarque** : Si vous n’avez pas activé la déconnexion forcée lors d’un ralentissement, les hôtes de session sans sessions actives ou déconnectées sont libérés.

1. Sous l’onglet **Heures creuses** de l’Assistant **Ajouter une planification**, spécifiez les informations suivantes et cliquez sur **Ajouter**.

   |Paramètre|Valeur|
   |---|---|
   |Heure de début (format 24 heures)|votre heure actuelle moins 1 heure|
   |Algorithme d’équilibrage de charge|**À profondeur prioritaire**|

   >**Remarque** : La valeur du seuil de capacité de cette phase est déterminée par la valeur du seuil de capacité du ralentissement.

1. De retour dans l’onglet **Planifications** de l’assistant **Créer un plan de mise à l’échelle**, sélectionnez **Suivant : Affectations de pool d’hôtes >**  :
1. Dans la page **Attributions de pool d’hôtes**, dans la liste déroulante **Sélectionner un pool d’hôtes**, sélectionnez **az140-21-hp1**, vérifiez que la case **Activer la mise à l’échelle automatique** est cochée et sélectionnez **Vérifier + créer**, puis **Créer**.


### Exercice 2 : Vérifier la mise à l’échelle automatique des hôtes de session Azure Virtual Desktop

Les principales tâches de cet exercice sont les suivantes

1. Vérifier la mise à l’échelle automatique des hôtes de session Azure Virtual Desktop


#### Tâche 1 : Vérifier la mise à l’échelle automatique des hôtes de session Azure Virtual Desktop

1. Sur l’ordinateur de labo, dans la fenêtre de navigateur web affichant le portail Azure, ouvrez une session **PowerShell** dans le volet **Cloud Shell**.
1. À partir de la session PowerShell du volet Cloud Shell, exécutez la commande suivante pour démarrer les machines virtuelles Azure hôtes de session Azure Virtual Desktop que vous utiliserez dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**Remarque** : Attendez que les machines virtuelles Azure hôtes de session s’exécutent.

1. Sur l’ordinateur de labo, dans la fenêtre de navigateur web affichant le portail Azure, accédez à la page du pool d’hôtes **az140-21-hp1**.
1. Dans la page **az140-21-hp1**, sélectionnez **Hôtes de session**.
1. Attendez qu’au moins un hôte de session soit répertorié avec l’état **Arrêt**.

   > **Remarque** : Vous devrez probablement actualiser la page pour mettre à jour l’état des hôtes de session.

   > **Remarque** : Si tous les hôtes de session sont toujours disponibles après 15 minutes, revenez à la page **az140-51-scaling-plan** et réduisez la valeur du **Pourcentage minimal d’hôtes (%)** du paramètre **Ralentissement**.

   > **Remarque** : Une fois que l’état d’un ou plusieurs hôtes de session a changé, les journaux de mise à l’échelle automatique devraient être disponibles dans le compte Stockage Azure. 

1. Dans le portail Azure, recherchez et sélectionnez **Comptes de stockage**, puis dans la page **Comptes de stockage**, sélectionnez l’entrée représentant le compte de stockage créé précédemment dans cet exercice (dont le nom commence par le préfixe **az140st51**).
1. Dans la page des comptes de stockage, sélectionnez **Conteneurs**.
1. Dans la liste des conteneurs, sélectionnez **insights-logs-autoscaleevaluationpooled**.
1. Dans la page **insights-logs-autoscaleevaluationpooled**, parcourez la hiérarchie des dossiers jusqu’à atteindre l’entrée représentant un blob au format JSON stocké dans le conteneur.
1. Sélectionnez l’entrée du blob, sélectionnez l’icône des points de suspension à l’extrême droite de la page, puis dans le menu déroulant, sélectionnez **Télécharger**.
1. Sur votre ordinateur de labo, ouvrez le blob téléchargé dans un éditeur de texte de votre choix et examinez son contenu. Vous devriez être en mesure de trouver des références aux événements de mise à l’échelle automatique. Dans ce cas, nous pouvons rechercher « désalloué » pour faciliter l’identification.

   >**Remarque** : Voici un exemple de contenu de blob qui inclut des références aux événements de mise à l’échelle automatique :

   ```json
   host_Ring    "R0"
   Level    4
   ActivityId   "00000000-0000-0000-0000-000000000000"
   time "2023-03-26T19:35:46.0074598Z"
   resourceId   "/SUBSCRIPTIONS/AAAAAAAE-0000-1111-2222-333333333333/RESOURCEGROUPS/AZ140-51-RG/PROVIDERS/MICROSOFT.DESKTOPVIRTUALIZATION/SCALINGPLANS/AZ140-51-SCALING-PLAN"
   operationName    "ScalingEvaluationSummary"
   category "AutoscaleEvaluationPooled"
   resultType   "Succeeded"
   level    "Informational"
   correlationId    "ddd3333d-90c2-478c-ac98-b026d29e24d5"
   properties   
   Message  "Active session hosts are at 0.00% capacity (0 sessions across 3 active session hosts). This is below the minimum capacity threshold of 90%. 2 session hosts can be drained and deallocated."
   HostPoolArmPath  "/subscriptions/aaaaaaaa-0000-1111-2222-333333333333/resourcegroups/az140-21-rg/providers/microsoft.desktopvirtualization/hostpools/az140-21-hp1"
   ScalingEvaluationStartTime   "2023-03-26T19:35:43.3593413Z"
   TotalSessionHostCount    "3"
   UnhealthySessionHostCount    "0"
   ExcludedSessionHostCount "0"
   ActiveSessionHostCount   "3"
   SessionCount "0"
   CurrentSessionOccupancyPercent   "0"
   CurrentActiveSessionHostsPercent "100"
   Config.ScheduleName  "az140-51-schedule"
   Config.SchedulePhase "OffPeak"
   Config.MaxSessionLimitPerSessionHost "2"
   Config.CapacityThresholdPercent  "90"
   Config.MinActiveSessionHostsPercent  "5"
   DesiredToScaleSessionHostCount   "-2"
   EligibleToScaleSessionHostCount  "1"
   ScalingReasonType    "DeallocateVMs_BelowMinSessionThreshold"
   BeganForceLogoffOnSessionHostCount   "0"
   BeganDeallocateVmCount   "1"
   BeganStartVmCount    "0"
   TurnedOffDrainModeCount  "0"
   TurnedOnDrainModeCount   "1"
   ```


### Exercice 3 : Arrêter et libérer les machines virtuelles Azure provisionnées dans le labo

Les principales tâches de cet exercice sont les suivantes

1. Arrêter et libérer les machines virtuelles Azure provisionnées dans le labo

>**Remarque** : Dans cet exercice, vous allez libérer les machines virtuelles Azure utilisées dans ce labo pour réduire les frais de calcul correspondants.

#### Tâche 1 : Libérer les machines virtuelles Azure provisionnées dans le labo

1. Basculez sur l’ordinateur de labo, puis dans la fenêtre de navigateur web affichant le portail Azure, ouvrez une session **PowerShell** dans le volet **Cloud Shell**.
1. À partir de la session PowerShell du volet Cloud Shell, exécutez la commande suivante pour lister toutes les machines virtuelles Azure utilisées dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. À partir de la session PowerShell du volet Cloud Shell, exécutez la commande suivante pour arrêter et libérer toutes les machines virtuelles Azure que vous avez utilisées dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Remarque** : La commande s’exécute de manière asynchrone (comme l’indique le paramètre -NoWait), donc même si vous pouvez exécuter une autre commande PowerShell dans la même session PowerShell immédiatement après, il faut quelques minutes avant que les machines virtuelles Azure ne soient réellement arrêtées et libérées.
