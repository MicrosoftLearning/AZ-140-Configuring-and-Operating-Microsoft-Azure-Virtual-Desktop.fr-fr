---
lab:
  title: "Labo\_: se connecter aux hôtes de session (Entra\_ID)"
  module: 'Module 3.2: Plan and implement user experience and client settings'
---

# Labo - Se connecter aux hôtes de session (Entra ID)
# Manuel de labo de l’étudiant

## Dépendances du labo

- Un abonnement Azure que vous allez utiliser dans ce labo.
- Un compte d’utilisateur Microsoft Entra avec le rôle Propriétaire ou Collaborateur dans l’abonnement Azure que vous allez utiliser dans ce labo et avec les autorisations suffisantes pour joindre des appareils au locataire Entra associé à cet abonnement Azure.
- Le labo terminé *Déployer des pools d’hôtes et des hôtes de session à l’aide du portail Azure (Entra ID)*
- Le labo terminé *Gérer des pools d’hôtes et des hôtes de session à l’aide du portail Azure (Entra ID)*

## Durée estimée

20 minutes

## Scénario de labo

Vous disposez d’un environnement Azure Virtual Desktop existant contenant des hôtes de session joints à Entra. Vous devez valider leurs fonctionnalités en vous y connectant à partir d’un client Windows 11 qui n’est pas joint ou inscrit à Microsoft Entra.

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

- Validez les fonctionnalités des hôtes de session Azure Virtual Desktop joints à Microsoft Entra en vous y connectant à partir d’un client Windows qui n’est pas joint ou inscrit à Microsoft Entra.

## Fichiers du labo

- Aucun

## Instructions

### Exercice 1 : valider les fonctionnalités des hôtes de session Azure Virtual Desktop joints à Microsoft Entra en vous y connectant à partir d’un client Windows 11
  
Les principales tâches de cet exercice sont les suivantes

1. Ajuster les propriétés RDP du pool d’hôtes Azure Virtual Desktop
1. Installer le client Bureau à distance Microsoft sur un ordinateur Windows 11
1. S’abonner à un espace de travail Azure Virtual Desktop
1. Tester les applications Azure Virtual Desktop


#### Tâche 1 : ajuster les propriétés RDP du pool d’hôtes Azure Virtual Desktop

> **Note** : les paramètres RDP que vous avez implémentés dans le labo précédent offrent l’expérience utilisateur optimale (via la prise en charge de l’authentification unique), mais cela nécessite des modifications supplémentaires décrites dans [Configurer l’authentification unique pour Azure Virtual Desktop à l’aide de l’authentification Microsoft Entra ID](https://learn.microsoft.com/en-us/azure/virtual-desktop/configure-single-sign-on). Sans ces modifications, par défaut, l’authentification est prise en charge afin que l’ordinateur client réponde à l’un des critères suivants :

- L’ordinateur client est joint à Microsoft Entra au même locataire Microsoft Entra que l’hôte de session
- L’ordinateur client dispose d’une jonction hybride à Microsoft Entra au même locataire Microsoft Entra que l’hôte de session
- L’ordinateur client est inscrit auprès de Microsoft Entra au même locataire Microsoft Entra que l’hôte de session

Étant donné qu’aucun de ces critères ne s’applique à l’ordinateur de labo, il est nécessaire d’ajouter `targetisaadjoined:i:1` en tant que propriété RDP personnalisée au pool d’hôtes.

1. Si besoin, à partir de votre ordinateur de labo, démarrez un navigateur web, accédez au portail Azure, puis connectez-vous en fournissant des informations d’identification d’un compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utiliserez dans ce labo.

    > **Note** : utilisez les informations d’identification du compte `User1-` répertorié dans l’onglet Ressources sur le côté droit de la fenêtre de session de labo.

1. Dans le navigateur web affichant le portail Azure, dans la page **az140-21-hp1**, dans la barre de menus verticale, dans la section **Paramètres**, sélectionnez l’entrée **Propriétés RDP**.
1. Dans la page **Propriétés RDP az140-21-hp1 \|**, sélectionnez l’onglet **Avancé**. 
1. Dans l’onglet **Avancé** de la page **Propriétés RDP az140-21-hp1 \|**, dans la zone de texte **Propriétés RDP**, ajoutez la chaîne suivante au contenu existant (veillez à ajouter un point-virgule de début (`;`) si nécessaire pour séparer cette chaîne de celle qui la précède :

    ```txt
    targetisaadjoined:i:1
    ```

1. Dans la zone de texte **Propriétés RDP**, supprimez la chaîne suivante (le cas échéant) du contenu existant (avec son point-virgule de fin) :

    ```txt
    enablerdsaadauth:i:value
    ```

1. Dans la page **Propriétés RDP az140-21-hp1 \|**, sélectionnez **Enregistrer**.

#### Tâche 2 : Installer le client Bureau à distance Microsoft sur un ordinateur Windows 11

1. À partir de l’ordinateur de labo, démarrez un navigateur web, accédez à [Se connecter à Azure Virtual Desktop avec le client Bureau à distance pour Windows](https://learn.microsoft.com/en-us/azure/virtual-desktop/users/connect-windows), faites défiler jusqu’à la section **Télécharger et installer le client Bureau à distance (MSI)** et sélectionnez le lien [Windows 64 bits](https://go.microsoft.com/fwlink/?linkid=2139369). 
1. Ouvrez Explorateur de fichiers, accédez au dossier **Téléchargements** et lancez l’installation du fichier MSI nouvellement téléchargé. 
1. Dans la fenêtre **Installation du Bureau à distance**, lorsque vous y êtes invité, acceptez les conditions du contrat de licence et choisissez l’option **Installer pour tous les utilisateurs de cet ordinateur**. Si vous y êtes invité, acceptez l’invite de contrôle de compte d’utilisateur pour poursuivre l’installation.
1. Une fois l’installation terminée, vérifiez que la case **Lancer le Bureau à distance à la fin de l’installation** est cochée, puis sélectionnez **Terminer** pour démarrer le client Bureau à distance Microsoft.

   > **Note** : l’[application Bureau à distance du Store](https://learn.microsoft.com/en-us/azure/virtual-desktop/users/connect-windows?pivots=rd-store) pour Windows ne prend pas en charge la connexion aux hôtes de session joints à Microsoft Entra.

#### Tâche 3 : s’abonner à un espace de travail Azure Virtual Desktop

1. Sur l’ordinateur de labo, basculez vers la fenêtre du client **Bureau à distance**, sélectionnez **S’abonner** et, lorsque vous y êtes invité, connectez-vous avec les informations d’identification du compte d’utilisateur Entra ID `User1` que vous pouvez trouver dans l’onglet **Ressources** dans le volet droit de la fenêtre d’interface du labo.

   > **Note** : sélectionnez le compte d’utilisateur qui est membre du groupe Entra avec le préfixe **AVD-DAG**.

   > **Note** : sinon, dans la fenêtre du client **Bureau à distance**, sélectionnez **S’abonner avec l’URL**, dans le volet **S’abonner à un espace de travail**, dans l’**Adresse e-mail ou URL de l’espace de travail**, tapez **https://client.wvd.microsoft.com/api/arm/feeddiscovery**, sélectionnez **Suivant** et, une fois que vous y êtes invité, connectez-vous avec les informations d’identification Microsoft Entra.

1. Vérifiez que la page **Bureau à distance** affiche uniquement l’icône **SessionDesktop**.

   > **Note** : ce comportement est attendu, car le compte d’utilisateur Microsoft Entra que vous avez sélectionné a été affecté dans le premier labo *Déployer des pools d’hôtes et des hôtes de session à l’aide du Portail Azure (Entra ID)* au groupe d’applications de bureau généré automatiquement **az140-21-hp1-DAG desktop**.

1. Dans la page **Bureau à distance**, cliquez avec le bouton droit sur l’icône **SessionDesktop** et, dans le menu contextuel, sélectionnez **Paramètres**.
1. Dans le volet **SessionDesktop**, désactivez le commutateur **Utiliser les paramètres par défaut**.
1. Dans la section **Paramètres d’affichage**, dans le menu déroulant, sélectionnez l’entrée **Sélectionner des affichages** et choisissez les affichages que vous souhaitez utiliser pour la session.
1. Dans le volet **SessionDesktop**, passez en revue les options restantes, notamment **Adapter aux affichages actuels (agrandir)**, **Un seul affichage en mode fenêtré** et **Ajuster la session à la fenêtre**, sans apporter de modifications. 
1. Fermez le volet **SessionDesktop**. 
1. Dans la page **Bureau à distance**, double-cliquez sur l’icône **SessionDesktop**.
1. Lorsque vous êtes invité à vous connecter, dans la boîte de dialogue **Sécurité Windows**, entrez le mot de passe du premier compte d’utilisateur Microsoft Entra que vous avez utilisé dans cette tâche pour vous connecter à l’environnement Azure Virtual Desktop cible.

   > **Note** : Azure Virtual Desktop ne prend pas en charge la connexion à Microsoft Entra ID en utilisant un compte d’utilisateur, puis la connexion à Windows avec un compte d’utilisateur distinct. La connexion simultanée avec deux comptes différents peut entraîner une reconnexion des utilisateurs à l’hôte de session incorrect, des informations incorrectes ou manquantes dans le Portail Azure et des messages d’erreur qui s’affichent lors de l’utilisation de l’attachement d’application ou de l’attachement d’application MSIX.

   > **Note** : la fenêtre **SessionDesktop** vous sera automatiquement présentée.

1. Dans la fenêtre de session Bureau à distance, vérifiez que vous disposez d’un accès administratif complet dans la session (par exemple, sélectionnez l’icône du logo **Windows** dans la barre des tâches, puis sélectionnez l’élément **Windows PowerShell(Admin)** dans le menu contextuel.
1. Dans la fenêtre de session Bureau à distance, sélectionnez l’icône du logo Windows dans la barre des tâches, sélectionnez l’icône d’avatar représentant le compte d’utilisateur Microsoft Entra que vous avez utilisé pour vous connecter et, dans le menu contextuel, sélectionnez **Se déconnecter**.

   > **Note** : cette opération met automatiquement fin à la session Bureau à distance. 

1. De retour dans la fenêtre **Bureau à distance**, sélectionnez l’icône de points de suspension (`...`) à droite de l’entrée d’espace de travail **az140-21-ws1**, sélectionnez **Se désabonner** et, quand vous êtes invité à confirmer, sélectionnez **Continuer**.
1. Dans la fenêtre du client **Bureau à distance**, sélectionnez **S’abonner** et, lorsque vous y êtes invité, connectez-vous avec les informations d’identification du deuxième compte d’utilisateur Entra ID que vous pouvez trouver dans l’onglet **Ressources** dans le volet droit de la fenêtre de l’interface de labo.

   > **Note** : sélectionnez le compte d’utilisateur qui est membre du groupe Entra avec le préfixe **AVD-RemoteApp**.

1. Vérifiez que la page **Bureau à distance** affiche quatre icônes, notamment l’invite de commandes, Microsoft Word, Microsoft Excel et Microsoft PowerPoint. 

   > **Note** : ce comportement est attendu, car le compte d’utilisateur Microsoft Entra que vous avez sélectionné a été affecté dans le premier labo *Déployer des pools d’hôtes et des hôtes de session à l’aide du portail Azure (Entra ID)* aux groupes d’applications **az140-21-hp1-Office365-RAG** et **az140-21-hp1-Utilities-RAG**.

1. Double-cliquez sur l’icône d’invite de commandes. 
1. Lorsque vous êtes invité à vous connecter, dans la boîte de dialogue **Sécurité Windows**, entrez le mot de passe du deuxième compte d’utilisateur Microsoft Entra que vous avez utilisé pour vous connecter à l’environnement Azure Virtual Desktop cible.
1. Vérifiez qu’une fenêtre d’**invite de commandes** s’affiche peu de temps après. 
1. Dans la fenêtre d’invite de commandes, tapez **hostname**, puis appuyez sur la touche **Entrée** pour afficher le nom de l’ordinateur sur lequel l’invite de commandes s’exécute.

   > **Note** : vérifiez que le nom affiché commence par le préfixe **sh-**.

1. À l’invite de commandes, tapez **logoff**, puis appuyez sur l'**Entrée** touche pour vous déconnecter de la session d’application distante actuelle.
1. Double-cliquez sur les icônes restantes de la page **Bureau à distance** pour lancer Microsoft Word, Microsoft Excel et Microsoft PowerPoint.
1. Fermez chaque fenêtre de session.
