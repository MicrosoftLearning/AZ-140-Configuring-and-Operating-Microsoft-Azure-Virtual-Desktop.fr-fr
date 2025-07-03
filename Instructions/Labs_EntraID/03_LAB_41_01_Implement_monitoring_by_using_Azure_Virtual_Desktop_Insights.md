---
lab:
  title: "Labo\_: implémenter la surveillance à l’aide d’Azure\_Virtual\_Desktop\_Insights"
  module: 'Module 4.1: Monitor and manage Azure Virtual Desktop services'
---

# Labo - Implémenter la surveillance à l’aide d’Azure Virtual Desktop Insights
# Manuel de labo de l’étudiant

## Dépendances du labo

- Un abonnement Azure que vous allez utiliser dans ce labo.
- Un compte d’utilisateur Microsoft Entra avec le rôle Propriétaire ou Collaborateur dans l’abonnement Azure que vous allez utiliser dans ce labo et avec les autorisations suffisantes pour joindre des appareils au locataire Entra associé à cet abonnement Azure.
- Le labo terminé *Déployer des pools d’hôtes et des hôtes de session à l’aide du portail Azure (Entra ID)*
- Le labo terminé *Gérer des pools d’hôtes et des hôtes de session à l’aide du portail Azure (Entra ID)*

## Durée estimée

25 minutes

## Scénario de labo

Vous disposez d’un environnement Azure Virtual Desktop existant. Vous souhaitez surveiller l’état et les activités de l’environnement.

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

- Implémenter la surveillance d’un environnement Azure Virtual Desktop

## Fichiers du labo

- Aucun

## Instructions

### Exercice 1 : implémenter la surveillance d’un environnement Azure Virtual Desktop
  
Les principales tâches de cet exercice sont les suivantes

1. Inscrire l’abonnement Azure auprès du fournisseur de ressources Microsoft.Insights
1. Créer un espace de travail Azure Log Analytics
1. Configurer le classeur de configuration Virtual Desktop Insights

#### Tâche 1 : enregistrer l’abonnement Azure auprès du fournisseur de ressources Microsoft.Insights

> **Note** : Azure Virtual Desktop Insights s’appuie sur le fournisseur de ressources Microsoft.Insights. Vous devez donc d’abord l’inscrire dans l’abonnement Azure que vous utilisez pour ce labo. Dans le labo *Déployer des pools d’hôtes et des hôtes de session à l’aide du portail Azure (Entra ID),* vous avez effectué cette tâche à l’aide d’Azure PowerShell. Dans ce labo, vous allez l’accomplir à l’aide du portail Azure (les deux méthodes sont prises en charge et disponibles).

1. Si besoin, à partir de votre ordinateur de labo, démarrez un navigateur web, accédez au portail Azure, puis connectez-vous en fournissant des informations d’identification d’un compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utiliserez dans ce labo.

    > **Note** : utilisez les informations d’identification du compte `User1-` répertorié dans l’onglet Ressources sur le côté droit de la fenêtre de session de labo.

1. À partir de l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Abonnements**, dans la page **Abonnements**, sélectionnez l’abonnement Azure que vous utilisez dans ce labo, puis, dans le menu de navigation vertical, dans la section **Paramètres**, sélectionnez **Fournisseurs de ressources**.
1. Dans l’onglet **Fournisseurs de ressources**, dans la zone de texte de recherche, entrez **Microsoft.Insights**, dans la liste des résultats, sélectionnez le petit cercle à gauche de l’entrée **Microsoft.Insights**, puis sélectionnez **Enregistrer**.

    > **Note** : attendez la fin du processus d’inscription. Cela prend généralement environ une minute. Utilisez le bouton **Actualiser** de la barre d’outils pour afficher la valeur à jour de l’état d’inscription.

#### Tâche 2 : Créer un espace de travail Azure Log Analytics

> **Note** : Azure Virtual Desktop Insights est un tableau de bord basé sur les classeurs Azure Monitor qui aide les professionnels de l’informatique à surveiller leurs environnements Azure Virtual Desktop. 

> **Note** : vous pouvez surveiller les opérations de mise à l’échelle automatique avec Insights uniquement avec des pools d’hôtes groupés. 

1. À partir de l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Espaces de travail Log Analytics**, dans la page **Espaces de travail Log Analytics**, sélectionnez **+ Créer**.
1. Dans l’onglet **Informations de base** de la page **Créer un espace de travail Log analytics**, spécifiez les paramètres suivants et sélectionnez **Examiner et créer** :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|Nom d’un nouveau groupe de ressources **az140-411e-RG**|
    |Nom|**az140-laworkspace41e**|
    |Région|Nom d’une région Azure dans laquelle vous avez déployé l’environnement Azure Virtual Desktop|

1. Dans la page **Vérifier + créer**, sélectionnez **Créer**.

    > **Remarque** : Attendez la fin du processus de provisionnement. Cela prend généralement environ une minute.

    > **Note** : ensuite, vous devez activer la collecte de données dans l’espace de travail Log Analytics nouvellement approvisionné des diagnostics à partir de l’environnement Azure Virtual Desktop, des compteurs de performances des hôtes de session et des journaux des événements Windows à partir des hôtes de session Azure Virtual Desktop.

#### Tâche 3 : configurer le classeur de configuration Virtual Desktop Insights

> **Note** : la première fois que vous ouvrez Azure Virtual Desktop Insights, vous devez en effectuer la configuration pour votre environnement Azure Virtual Desktop.

1. À partir de l’ordinateur de labo, dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop** et, dans la page **Azure Virtual Desktop**, dans le menu de navigation vertical, dans la section **Surveillance**, sélectionnez **Classeurs**.
1. Dans la liste des classeurs **Windows Virtual Desktop**, dans la section **Windows Virtual Desktop**, sélectionnez le classeur **Insights**.
1. Dans la page **Classeurs \| Insights Azure Virtual Desktop \|**, passez en revue les messages d’avertissement indiquant que l’espace de travail et les hôtes de session n’envoient pas de données à l’espace de travail, puis sélectionnez le lien **Classeur de configuration** pour résoudre le problème.
1. Dans la page **CheckAMAConfiguration**, dans l’onglet **Paramètres de diagnostics des ressources**, dans la liste déroulante **Espace de travail Log Analytics**, sélectionnez **az140-laworkspace41e**.
1. Sur la page **CheckAMAConfiguration**, dans l’onglet **Paramètres de diagnostics des ressources**, dans la section **Pool d’hôtes az140-21-hp1**, examinez le message d’avertissement indiquant qu’aucune configuration de diagnostic existante n’a été trouvée pour le pool d’hôtes sélectionné, puis sélectionnez **Configurer le pool d’hôtes**.
1. Dans le volet **Déployer un modèle**, sélectionnez **Déployer**.

    > **Note** : cela active efficacement les tables de diagnostic suivantes dans l’espace de travail Log Analytics cible :
    - Activités de gestion
    - Flux
    - Connexions
    - Erreurs
    - Points de contrôle
    - HostRegistration
    - AgentHealthStatus

    > **Remarque** : Attendez la fin du déploiement. Cette étape prend généralement moins d’une minute.

1. Dans la page **CheckAMAConfiguration**, dans l’onglet **Paramètres de diagnostics des ressources**, sélectionnez l’icône **Actualiser** (flèche circulaire) dans la barre d’outils.
1. Passez en revue la section **Pool d’hôtes az140-21-hp1** et vérifiez que les paramètres de diagnostic sont activés pour **allLogs**.
1. Dans l’onglet **Paramètres de diagnostics des ressources**, faites défiler jusqu’à la section **Espace de travail az140-21-ws1**, puis sélectionnez **Configurer l’espace de travail**.
1. Dans le volet **Déployer un modèle**, sélectionnez **Déployer**.

    > **Note** : cela configure efficacement l’espace de travail pour **allLogs**.

    > **Remarque** : Attendez la fin du déploiement. Cette étape prend généralement moins d’une minute.

1. Dans la page **CheckAMAConfiguration**, dans l’onglet **Paramètres de diagnostics des ressources**, sélectionnez l’icône **Actualiser** (flèche circulaire) dans la barre d’outils.
1. Passez en revue la section **Espace de travail az140-21-ws1** et vérifiez que les paramètres de diagnostic sont activés pour **allLogs** et qu’il ne reste pas de messages d’avertissement.
1. Accédez au haut de la page **CheckAMAConfiguration** et basculez vers l’onglet **Paramètres des données de l’hôte de la session**.
1. Dans l’onglet **Paramètres des données de l’hôte de la session**, dans la section **Créer une DCR**, dans la liste déroulante **Destination de l’espace de travail**, sélectionnez **az140-laworkspace41e**, puis **Créer une règle de collecte de données**.
1. Dans le volet **Déployer un modèle**, sélectionnez **Déployer**.

    > **Remarque** : Attendez la fin du déploiement. Cette étape prend généralement moins d’une minute.

1. Sur la page **CheckAMAConfiguration**, dans l’onglet **Sélectionner les paramètres des données de l’hôte de session**, sélectionnez l’icône **Actualiser** (la flèche circulaire) dans la barre d’outils.

    > **Note** : avant de continuer, assurez-vous que la DCR nouvellement créée est répertoriée dans la sous-section **DCR disponibles** de la section **Créer une DCR**. Si ce n’est pas le cas, attendez encore une minute et actualisez à nouveau la page.

1. Dans l’onglet **Sélectionner les paramètres des données de l’hôte de session**, dans la liste déroulante **DCR sélectionnée**, sélectionnez l’entrée commençant par le préfixe **microsoft-avdi-**.
1. Si nécessaire, dans l’onglet **Paramètres des données de l’hôte de session**, dans la section **Associations DCR**, sélectionnez **Déployer une association** et, dans le volet **Déployer un modèle**, sélectionnez **Déployer**.

    > **Note** : cela associe efficacement la DCR nouvellement créée aux hôtes de session dans le pool d’hôtes **az140-21-hp1**.

    > **Remarque** : Attendez la fin du déploiement. Cette étape prend généralement moins d’une minute.

1. Sur la page **CheckAMAConfiguration**, dans l’onglet **Sélectionner les paramètres des données de l’hôte de session**, sélectionnez l’icône **Actualiser** (la flèche circulaire) dans la barre d’outils.
1. Dans l’onglet **Sélectionner les paramètres des données de l’hôte de session**, dans la section **Hôtes de session dont l’extension Azure Monitor est manquante**, sélectionnez **Ajouter une extension**.
1. Dans le volet **Déployer un modèle**, sélectionnez **Déployer**.

    > **Note** : cette opération installe efficacement l’extension Azure Monitor sur les hôtes de session dans le pool d’hôtes **az140-21-hp1**.

    > **Remarque** : Attendez la fin du déploiement. Cela peut prendre environ 1 minute.

1. Sur la page **CheckAMAConfiguration**, dans l’onglet **Sélectionner les paramètres des données de l’hôte de session**, sélectionnez l’icône **Actualiser** (la flèche circulaire) dans la barre d’outils.
1. Vérifiez qu’aucune erreur ou message d’avertissement n’est affiché. 
1. Accédez au haut de la page **CheckAMAConfiguration**, sélectionnez l’onglet **Données générées**, puis sélectionnez l’icône **Actualiser** (flèche circulaire) dans la barre d’outils.
1. Passez en revue les sections affichant des graphiques représentant des données collectées, y compris **Données facturées au cours des 24 dernières heures**, **Compteurs de performances** et **Événements**.

    > **Note** : utilisez la section **Données facturées au cours des dernières 24 heures** pour surveiller l’ingestion des données. Vous êtes responsable des frais inhérents à Log Analytics pour le stockage et l’ingestion des données.

1. Dans le navigateur web affichant le portail Azure, revenez à la page **Azure Virtual Desktop** et, dans la section **Surveillance** du menu de navigation vertical, sélectionnez **Insights**.
1. Dans la page **Insights Azure Virtual Desktop \|**, passez en revue le contenu de l’onglet **Vue d’ensemble**, notamment la section **Capacité**, **Diagnostics de connexion : % des utilisateurs en mesure de se connecter**, **Performances de connexion : temps pour se connecter (nouvelles sessions)** et télémétrie d’**Utilisation**. 
1. Ensuite, passez en revue tous les onglets restants de la page **Insights Azure Virtual Desktop \|**, notamment **Fiabilité de la connexion**, **Diagnostics de connexion**, **Performances de connexion**, **Utilisateurs **, **Utilisation**, **Clients** et **Alertes**.

    > **Note** : envisagez de revoir ces onglets de la page Insights une fois que vous avez terminé les labos suivants pour passer en revue les graphiques représentant les données de télémétrie collectées.
