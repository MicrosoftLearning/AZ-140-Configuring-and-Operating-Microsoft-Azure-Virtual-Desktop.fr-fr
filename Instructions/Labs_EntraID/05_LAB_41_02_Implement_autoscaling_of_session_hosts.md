---
lab:
  title: "Labo\_: implémenter la mise à l’échelle automatique des hôtes de session"
  module: 'Module 4.1: Monitor and manage Azure Virtual Desktop services'
---

# Labo - Implémenter et surveiller la mise à l’échelle automatique des hôtes de session
# Manuel de labo de l’étudiant

## Dépendances du labo

- Un abonnement Azure que vous allez utiliser dans ce labo.
- Un compte d’utilisateur Microsoft Entra avec le rôle Propriétaire ou Collaborateur dans l’abonnement Azure que vous allez utiliser dans ce labo et avec les autorisations suffisantes pour joindre des appareils au locataire Entra associé à cet abonnement Azure.
- Le labo terminé *Déployer des pools d’hôtes et des hôtes de session à l’aide du portail Azure (Entra ID)*
- Le labo terminé *Gérer des pools d’hôtes et des hôtes de session à l’aide du portail Azure (Entra ID)*
- Le labo *Se connecter aux hôtes de session (Entra ID)* s’est terminé

## Durée estimée

45 minutes

## Scénario de labo

Vous disposez d’un environnement Azure Virtual Desktop qui change régulièrement d’utilisation. Vous souhaitez réduire le coût en tirant parti des fonctionnalités des plans de mise à l’échelle automatique.

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

- Implémenter et évaluer la mise à l’échelle automatique d’Azure Virtual Desktop

## Fichiers du labo

- Aucun

## Instructions

### Exercice 1 : implémenter des plans de mise à l’échelle automatique d’Azure Virtual Desktop
  
Les principales tâches de cet exercice sont les suivantes

1. Attribuer le rôle RBAC requis à un principal de service Azure Virtual Desktop
1. Arrêter et libérer tous les hôtes de session
1. Ajuster les paramètres du pool d’hôtes
1. Créer un plan de mise à l’échelle
1. Évaluer la fonctionnalité de mise à l’échelle automatique
1. Désactiver la mise à l’échelle automatique du pool d’hôtes

#### Tâche 1 : attribuer le rôle RBAC requis au principal de service Azure Virtual Desktop

> **Note** : pour que les plans de mise à l’échelle automatique fonctionnent, vous devez accorder au principal de service Azure Virtual Desktop les autorisations nécessaires pour gérer l’état d’alimentation des machines virtuelles hôtes de session. Ces autorisations peuvent être accordées à l’aide du rôle RBAC intégré **Contributeur de mise sous et hors tension de la virtualisation du Bureau**. Il est important de garder à l’esprit que l’attribution de rôle doit être effectuée au niveau de l’étendue de l’abonnement. L’attribution de ce rôle à n’importe quel niveau inférieur à votre abonnement, tel que le groupe de ressources, le pool d’hôtes ou la machine virtuelle, empêche le bon fonctionnement de la mise à l’échelle automatique. 

> **Note** : ce rôle est différent de celui (**Contributeur de mise sous tension de la virtualisation du Bureau**) utilisé dans le labo *Gérer les pools d’hôtes et les hôtes de session à l’aide du portail Azure (Entra ID)* qui a été nécessaire pour prendre en charge la fonctionnalité *Démarrer la machine virtuelle à la connexion*.

1. Si besoin, à partir de votre ordinateur de labo, démarrez un navigateur web, accédez au portail Azure, puis connectez-vous en fournissant des informations d’identification d’un compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utiliserez dans ce labo.

    > **Note** : utilisez les informations d’identification du compte `User1-` répertorié dans l’onglet Ressources sur le côté droit de la fenêtre de session de labo.

1. Sur l’ordinateur de labo, dans la fenêtre de navigateur web affichant le portail Azure, ouvrez une session PowerShell dans le volet Azure Cloud Shell.

    > **Note** : si vous y êtes invité, dans le volet **Mise en route**, dans la liste déroulante **Abonnement**, sélectionnez le nom de l’abonnement Azure que vous utilisez dans ce labo, puis sélectionnez **Appliquer**.

1. Dans la session PowerShell dans le volet Azure Cloud Shell, exécutez la commande suivante pour récupérer la valeur de la propriété ID de l’abonnement Azure que vous utilisez dans ce labo et stockez-la dans une variable `$subId` :

    ```powershell
    $subId = (Get-AzSubscription).Id
    ```

1. Exécutez la commande suivante pour créer une variable $parameters, qui stocke une table de hachage qui contient les valeurs du nom de définition de rôle RBAC, l’application Microsoft Entra représentant le principal de service **Azure Virtual Desktop** et l’étendue de l’abonnement :

    ```powershell
    $parameters = @{
        RoleDefinitionName = "Desktop Virtualization Power On Off Contributor"
        ApplicationId = "9cdead84-a844-4324-93f2-b2e6bb768d07"
        Scope = "/subscriptions/$subId"
    }
    ```

1. Exécutez la commande suivante pour créer une attribution de rôle RBAC :

    ```powershell
    New-AzRoleAssignment @parameters
    ```

1. Fermez le volet Cloud Shell.

#### Tâche 2 : arrêter et libérer tous les hôtes de session

> **Note** : pour évaluer la fonctionnalité de mise à l’échelle automatique, vous allez arrêter et libérer tous les hôtes de session dans l’environnement Azure Virtual Desktop. 

1. À partir de l’ordinateur de labo, dans la fenêtre du navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop** et, dans la page **Azure Virtual Desktop**, dans la barre de menus verticale, dans la section **Gérer**, sélectionnez **Pools d’hôtes**.
1. Dans la page **Pool d’hôtes Azure Virtual Desktop \|**, dans la liste des pools d’hôtes, sélectionnez **az140-21-hp1**.
1. Dans la page **az140-21-hp1**, dans la barre de menus verticale, dans la section **Gérer**, sélectionnez **Hôtes de session**.
1. Dans la page **Hôtes de session az140-21-hp1 \|**, cochez la case en regard de chaque nom d’hôte de session, puis sélectionnez **Arrêter**.
1. Lorsque vous êtes invité à confirmer, dans la fenêtre contextuelle **Arrêter les hôtes de session**, sélectionnez **Arrêter**.

    > **Note** : vous devrez peut-être sélectionner l’icône de points de suspension (`...`) dans la barre d’outils pour afficher le bouton **Arrêter**.

    > **Note** : n’attendez pas que les hôtes de session soient arrêtés et libérés, mais passez à la tâche suivante. L’arrêt et la libération des hôtes de session peuvent prendre environ 2 minutes.

#### Tâche 3 : ajuster les paramètres du pool d’hôtes

> **Note** : lorsque vous utilisez la mise à l’échelle automatique pour les pools d’hôtes groupés, vous devez disposer d’un paramètre MaxSessionLimit configuré pour ce pool d’hôtes. Dans ce labo, vous allez le définir à un niveau artificiellement bas afin de faciliter l’illustration de la fonctionnalité de mise à l’échelle automatique.

1. À partir de l’ordinateur de labo, dans le navigateur web affichant le portail Azure, sur la page **az140-21-hp1**, dans la section **Paramètres**, sélectionnez **Propriétés**.
1. Dans la page **Propriétés az140-21-hp1 \|**, dans la zone de texte **Limite de session maximale**, entrez **1**.
1. Dans la page **Propriétés az140-21-hp1 \|**, sélectionnez **Enregistrer**.

#### Tâche 4 : créer un plan de mise à l’échelle

1. À partir de l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop**, dans la page **Azure Virtual Desktop**, dans la section **Gérer** du menu de navigation vertical, sélectionnez **Plans de mise à l’échelle**.
1. Dans la page **Plans de mise à l’échelle Azure Virtual Desktop \|**, sélectionnez **+ Créer**.
1. Dans l’onglet **Informations de base** de la page **Créer un plan de mise à l’échelle**, spécifiez les paramètres suivants et sélectionnez **Suivant : Planifications** :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|Nom d’un nouveau groupe de ressources **az140-412e-RG**|
    |Nom du plan de mise à l’échelle|**az140-scalingplan412e**|
    |Région|Nom d’une région Azure dans laquelle vous avez déployé l’environnement Azure Virtual Desktop|
    |Nom convivial|**az140-scalingplan412e**|
    |Fuseau horaire|Fuseau horaire local de la région Azure dans laquelle vous avez déployé l’environnement Azure Virtual Desktop|
    |Type de pool d’hôtes|**Groupé**|
    |Méthode de mise à l'échelle|**Mise à l’échelle automatique de la gestion de l’alimentation**|

    > **Note** : laissez la propriété **Étiquette d’exclusion** non définie. En général, vous pouvez utiliser cette fonctionnalité pour exclure les machines virtuelles Azure avec des étiquettes définies arbitrairement à partir de la mise à l’échelle automatique.

1. Dans l’onglet **Planification**, sélectionnez **+ Ajouter une planification**.

    > **Note** : les planifications vous permettent de définir des heures d’accélération, des heures de pointe, des heures de ralentissement et des heures creuses pendant des jours de semaine et de spécifier des déclencheurs de mise à l’échelle automatique. Le plan de mise à l’échelle doit inclure une planification associée pour au moins un jour de la semaine. 

1. Dans l’onglet **Général** du volet **Ajouter une planification**, ajustez la configuration par défaut pour qu’elle corresponde aux paramètres suivants, puis sélectionnez **Suivant** :

    |Paramètre|Valeur|
    |---|---|
    |Fuseau horaire|Fuseau horaire local de votre environnement Azure Virtual Desktop (en fonction de la région que vous avez sélectionnée précédemment dans cette tâche)|
    |Nom de la planification|**week_schedule**|
    |Répéter le|**7 sélectionné** (sélectionner tous les jours de la semaine)|

    > **Note** : la planification couvre efficacement chaque jour de la semaine, ce qui facilitera l’évaluation des résultats de la mise à l’échelle automatique.

1. Dans l’onglet **Accélération** du volet **Ajouter une planification**, ajustez la configuration par défaut pour qu’elle corresponde aux paramètres suivants, puis sélectionnez **Suivant** :

    |Paramètre|Valeur|
    |---|---|
    |Heure de début (format 12 heures)|Votre heure actuelle moins 1 heure|
    |Algorithme d’équilibrage de charge|**À largeur prioritaire**|
    |Pourcentage minimal d’hôtes (%)|**30**|
    |Seuil de capacité (%)|**60**|

    > **Note** : pour les pools d’hôtes groupés, la mise à l’échelle automatique ignore les algorithmes d’équilibrage de charge existants dans les paramètres de votre pool d’hôtes et applique à la place un équilibrage de charge en fonction de la configuration de votre planification.

    > **Note** : le paramètre **Pourcentage minimal d’hôtes** désigne le pourcentage minimal de machines virtuelles hôtes de session à démarrer pour les heures d’accélération et de pointe. Par exemple, si la valeur **Pourcentage minimal d’hôtes** est spécifiée sur 30 % et que le nombre total d’hôtes de session dans votre pool d’hôtes est de 3, la mise à l’échelle automatique garantit qu’au minimum 1 hôte de session est disponible pour accepter les connexions utilisateur.

    > **Note** : la mise à l’échelle automatique arrondit vers le haut jusqu’au nombre entier le plus proche.

    > **Note** : le paramètre **Seuil de capacité** est le pourcentage de capacité du pool d’hôtes utilisé qui sera considéré pour évaluer s’il faut activer/désactiver des machines virtuelles pendant les heures d’accélération et de pointe. Par exemple, si le seuil de capacité spécifié est 60 % et que votre pool d’hôtes a une capacité totale de 1 session (avec un hôte en cours d’exécution), la mise à l’échelle automatique active des hôtes de session supplémentaires dès que le pool d’hôtes dépasse une charge de 60 % (100 % dans ce cas).

1. Dans l’onglet **Heures de pointe** du volet **Ajouter une planification**, ajustez la configuration par défaut pour qu’elle corresponde aux paramètres suivants, puis sélectionnez **Suivant** :

    |Paramètre|Valeur|
    |---|---|
    |Heure de début (format 12 heures)|Votre heure actuelle plus 1 heure|
    |Algorithme d’équilibrage de charge|**À profondeur prioritaire**|
    |Seuil de capacité (%)|**60**|

    > **Note** : le paramètre **Seuil de capacité (%)** est partagé entre les paramètres **Accélération** et **Heures de pointe**.

1. Dans l’onglet **Ralentissement** du volet **Ajouter une planification**, ajustez la configuration par défaut pour qu’elle corresponde aux paramètres suivants, puis sélectionnez **Suivant** :

    |Paramètre|Valeur|
    |---|---|
    |Heure de début (format 12 heures)|Votre heure actuelle plus 2 heures|
    |Algorithme d’équilibrage de charge|**À profondeur prioritaire**|
    |Pourcentage minimal d’hôtes actifs (%)|**10**|
    |Seuil de capacité (%)|**80**|
    |Forcer la déconnexion des utilisateurs|**Non**|
    |Arrêtez les machines virtuelles dans les cas suivants :|**Les machines virtuelles n’ont pas de sessions actives ou déconnectées**|

    > **Note** : le paramètre **Pourcentage minimal d’hôtes actifs (%)** désigne le pourcentage minimal de machines virtuelles hôtes de session auxquelles vous souhaitez accéder pour les heures de ralentissement et creuses. Par exemple, si la valeur **Pourcentage minimal d’hôtes actifs (%)** est défini sur 10 % et que le nombre total d’hôtes de session dans votre pool d’hôtes est de 3, la mise à l’échelle automatique garantit qu’au minimum 1 hôte de session est disponible pour prendre en charge des connexions utilisateur.

    > **Note** : le paramètre **Seuil de capacité (%)** désigne le pourcentage de capacité du pool d’hôtes utilisé qui sera considéré pour évaluer s’il faut désactiver les machines virtuelles pendant les heures de ralentissement et creuses. Par exemple, avec 1 connexion utilisateur et 3 hôtes en cours d’exécution, si le seuil de capacité est spécifié comme étant de 80 %, la mise à l’échelle automatique désactive 1 hôte (ce qui signifie une valeur de 50 % de capacité du pool d’hôtes utilisé).

    > **Note** : en général, la mise à l’échelle automatique arrête et libère les hôtes de session en fonction des règles suivantes :

    - La capacité du pool d’hôtes utilisée est inférieure au seuil de capacité.
    - La désactivation des hôtes de session n’entraîne pas de dépassement du seuil de capacité.
    - La mise à l’échelle automatique désactive seulement les hôtes de session sans aucune session utilisateur (sauf si le plan de mise à l’échelle est en phase de ralentissement et que vous avez activé le paramètre de fermeture de session forcée). 
    - La mise à l’échelle automatique groupée ne désactive pas les hôtes de session dans la phase d’accélération pour éviter une mauvaise expérience utilisateur.

1. Dans l’onglet **Heures creuses** du volet **Ajouter une planification**, ajustez la configuration par défaut pour qu’elle corresponde aux paramètres suivants, puis sélectionnez **Ajouter** :

    |Paramètre|Valeur|
    |---|---|
    |Heure de début (format 12 heures)|Votre heure actuelle plus 3 heures|
    |Algorithme d’équilibrage de charge|**À profondeur prioritaire**|
    |Seuil de capacité (%)|**80**|

    > **Note** : le paramètre **Seuil de capacité** est partagé entre les paramètres **Ralentissement** et **Heures creuses**.

1. De retour dans l’onglet **Planification** de la page **Créer un plan de mise à l’échelle**, sélectionnez **Suivant : Affectations de pool d’hôtes**.
1. Dans l’onglet **Affectations de pool d’hôtes**, dans la liste déroulante **Sélectionner un pool d’hôtes**, sélectionnez **az140-21-hp1**, vérifiez que la case **Activer la mise à l’échelle automatique** est cochée et sélectionnez **Examiner et créer**.
1. Dans la page **Vérifier + créer**, sélectionnez **Créer**.

    > **Note** : attendez la fin de la configuration de la mise à l’échelle automatique. Cela prend généralement seulement quelques secondes.

#### Tâche 5 : évaluer la fonctionnalité de mise à l’échelle automatique

> **Note** : vous allez commencer par évaluer les paramètres **Accélération**.

1. À partir de l’ordinateur de labo, dans la fenêtre du navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop** et, dans la page **Azure Virtual Desktop**, dans la barre de menus verticale, dans la section **Gérer**, sélectionnez **Pools d’hôtes**.
1. Dans la page **Pool d’hôtes Azure Virtual Desktop \|**, dans la liste des pools d’hôtes, sélectionnez **az140-21-hp1**.
1. Dans la page **az140-21-hp1**, dans la barre de menus verticale, dans la section **Gérer**, sélectionnez **Hôtes de session**.
1. Sur la page **Hôtes de session az140-21-hp1 \|**, passez en revue les valeurs du paramètre **État d’alimentation** des hôtes de session et vérifiez que l’un de ces hôtes est répertorié comme étant **En cours d’exécution**.

    > **Note** : vous devrez peut-être attendre quelques minutes avant que les premiers hôtes de session atteignent l’état **En cours d’exécution**.

    > **Note** : ce comportement est attendu, étant donné que, selon les paramètres **Accélération** du plan de mise à l’échelle nouvellement créé, au moins un hôte de session doit toujours être en ligne. À ce stade, la capacité du pool d’hôtes est 1 (car il n’y a qu’1 hôte en cours d’exécution), mais la capacité du pool d’hôtes utilisée est de 0 %, car il n’existe aucune connexion utilisateur.

    > **Note** : ensuite, vous allez évaluer le paramètre de seuil de capacité pour les options **Accélération** et **Heures de pointe** en lançant une session utilisateur unique. Nous pouvons évaluer cela même en dehors de la fenêtre **Heures de pointe**, car les deux étapes partagent le même seuil de capacité.

1. À partir de l’ordinateur de labo, démarrez le client Bureau à distance Microsoft.
1. Sur l’ordinateur de labo, dans la fenêtre du client **Bureau à distance**, sélectionnez **S’abonner**. Lorsque vous y êtes invité, connectez-vous avec les informations d’identification du compte d’utilisateur Entra ID `User2` que vous pouvez trouver dans l’onglet **Ressources** dans le volet droit de la fenêtre d’interface du labo.
1. Vérifiez que la page **Bureau à distance** affiche quatre icônes, notamment Microsoft Word, Microsoft Excel, Microsoft PowerPoint et invite de commandes. 
1. Double-cliquez sur l’icône d’invite de commandes. 
1. Lorsque vous êtes invité à vous connecter, dans la boîte de dialogue **Sécurité Windows**, entrez le mot de passe du compte d’utilisateur Microsoft Entra que vous avez utilisé pour vous connecter à l’environnement Azure Virtual Desktop cible.
1. Vérifiez qu’une fenêtre d’**invite de commandes** s’affiche peu de temps après. 

    > **Note** : la capacité du pool d’hôtes utilisée est maintenant de 100 %, ce qui est supérieur au seuil de capacité (60 %). Cela doit entraîner une mise à l’échelle automatique sur un autre hôte, ce qui amènera la capacité du pool d’hôtes utilisée à 50 %. Étant donné que cette valeur est inférieure au seuil de capacité, le troisième hôte reste arrêté/libéré. Vous allez le vérifier ensuite.

1. Sur l’ordinateur de labo, basculez vers la fenêtre de navigateur web affichant le portail Azure. 
1. Dans la page **Hôtes de session az140-21-hp1 \|**, sélectionnez **Actualiser**. Passez en revue les valeurs du paramètre **État d’alimentation** des hôtes de session et vérifiez que maintenant deux d’entre eux sont répertoriés comme étant **En cours d’exécution**.

    > **Note** : ensuite, vous allez évaluer le paramètre de seuil de capacité **Ralentissement** en ajustant sa fenêtre de temps. 

1. Sur la page **Hôtes de session az140-21-hp1 \|**, dans le menu de navigation vertical, dans la section **Gérer**, sélectionnez **Plans de mise à l’échelle**, puis, sur la page **Plans de mise à l’échelle**, sélectionnez **az140-scaleplan412e**.
1. Dans la page **az140-scaleplan412e**, dans le menu de navigation vertical, dans la section **Paramètres**, sélectionnez **Planifications**, puis sélectionnez **week_schedule**.
1. Dans le volet **week_schedule**, accédez à l’onglet **Ralentissement** et ajustez la valeur du paramètre **Heure de début (format 12 heures)** à n’importe quel moment entre l’**Heure de début (format 12 heures)** de la phase **Heures de pointe** et votre heure actuelle.

    > **Note** : vous devrez peut-être ajuster la valeur de l’**Heure de début (format 12 heures)** de la phase d’**Heures de pointe**.

1. Dans le volet **week_schedule**, accédez à l’onglet **Heures creuses**, puis sélectionnez **Enregistrer**.
1. Basculez vers la fenêtre d’**invite de commandes** représentant la seule session RDP dans le pool d’hôtes et, dans l’invite de commandes, entrez les éléments suivants et appuyez sur la touche **Entrée** :

    ```cmd
    logoff
    ```

1. À partir de l’ordinateur de labo, dans la fenêtre du navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop** et, dans la page **Azure Virtual Desktop**, dans la barre de menus verticale, dans la section **Gérer**, sélectionnez **Pools d’hôtes**.
1. Dans la page **Pool d’hôtes Azure Virtual Desktop \|**, dans la liste des pools d’hôtes, sélectionnez **az140-21-hp1**.
1. Dans la page **az140-21-hp1**, dans la barre de menus verticale, dans la section **Gérer**, sélectionnez **Hôtes de session**.
1. Dans la page **Hôtes de session az140-21-hp1 \|**, passez en revue les valeurs du paramètre **État d’alimentation** des hôtes de session et vérifiez qu’un seul de ces hôtes est répertorié comme étant **En cours d’exécution**.

    > **Note** : l’arrêt de l’hôte de session peut prendre 1 à 2 minutes.

#### Tâche 6 : désactiver la mise à l’échelle automatique du pool d’hôtes

> **Note** : pour vous assurer que la configuration de la mise à l’échelle automatique n’affecte pas d’autres labos, vous allez supprimer l’affectation du pool d’hôtes du plan de mise à l’échelle que vous avez implémenté dans ce labo.

1. À partir de l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop**, dans la page **Azure Virtual Desktop**, dans la section **Gérer** du menu de navigation vertical, sélectionnez **Plans de mise à l’échelle**, puis, dans la page **Plans de mise à l’échelle**, sélectionnez **az140-scaleplan412e**.
1. Dans la page **az140-scaleplan412e**, dans la section **Gérer**, sélectionnez **Affectations de pool d’hôtes**.
1. Dans la page **Affectations de pool d’hôtes az140-scaleplan412e \|**, sélectionnez **az140-21-hp1**, puis sélectionnez **Annuler l’affectation**. Quand vous êtes invité à confirmer, dans la boîte de dialogue **Annuler l’affectation d’un pool d’hôtes**, sélectionnez **Annuler l’affectation**.
