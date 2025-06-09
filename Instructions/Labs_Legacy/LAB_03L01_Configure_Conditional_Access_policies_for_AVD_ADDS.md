---
lab:
  title: 'Labo : Configurer des stratégies d’accès conditionnel pour AVD (AD DS)'
  module: 'Module 3: Manage Access and Security'
---

# Labo : Configurer des stratégies d’accès conditionnel pour AVD (AD DS)
# Manuel de labo de l’étudiant

## Dépendances du labo

- Un abonnement Azure
- Un compte Microsoft ou un compte Microsoft Entra disposant du rôle Administrateur général pour le locataire Microsoft Entra qui est associé à l’abonnement Azure, et disposant du rôle Propriétaire ou Contributeur dans l’abonnement Azure
- Le labo terminé **Préparer le déploiement d’Azure Virtual Desktop (AD DS)**
- Le labo terminé **Déployer des pools d’hôtes et des hôtes de session à l’aide du Portail Azure (AD DS)**

## Durée estimée

60 minutes

## Scénario du labo

Vous devez contrôler l’accès au déploiement d’Azure Virtual Desktop dans un environnement Active Directory Domain Services (AD DS) à l’aide de l’accès conditionnel Microsoft Entra.

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

- Préparer pour l’accès conditionnel basé sur Microsoft Entra pour Azure Virtual Desktop
- Implémenter l’accès conditionnel basé sur Microsoft Entra pour Azure Virtual Desktop

## Fichiers du labo

- Aucun 

## Instructions

>**Important** : Microsoft a renommé **Azure Active Directory** (**Azure AD**) **Microsoft Entra ID**. Pour plus d’informations sur cette modification, consultez [Le nouveau nom d’Azure Active Directory](https://learn.microsoft.com/en-us/entra/fundamentals/new-name). Cette modification étant en cours d’application, vous rencontrerez peut-être encore des cas d’incohérences entre l’instruction du labo et les éléments de l’interface à mesure que vous parcourez chaque exercice. Merci de bien vouloir tenir compte de cet état de fait (en particulier, dans ce laboratoire, **Microsoft Entra Connect** désigne le nouveau nom d’**Azure Active Directory Connect** et le terme **Azure Active Directory** est toujours utilisé lors de la configuration du point de connexion de service dans la tâche 4 de l’exercice 1).

>**Important** : L’activation d’un essai gratuit de Microsoft Entra ID P2 nécessite la fourniture d’informations de carte de crédit. Pour cette raison, cet exercice est entièrement facultatif. Au lieu de cela, les instructeurs de cours pourront choisir de faire la démonstration de cette fonctionnalité aux étudiants.

### Exercice 1 : Préparer pour l’accès conditionnel basé sur Microsoft Entra pour Azure Virtual Desktop

Les principales tâches de cet exercice sont les suivantes

1. Configurer les licences Microsoft Entra Premium P2
1. Configurer l’authentification multifacteur (MFA) Microsoft Entra
1. Inscrire un utilisateur pour l’authentification multifacteur Microsoft Entra
1. Configurer la jointure hybride Microsoft Entra
1. Déclencher la synchronisation delta Microsoft Azure Active Directory Connect

#### Tâche 1 : Configurer les licences Microsoft Entra Premium P2

>**Remarque** : Les licences Microsoft Entra Premium P1 ou P2 sont requises pour implémenter l’accès conditionnel Microsoft Entra. Vous allez utiliser un essai gratuit de 30 jours pour ce labo.

1. À partir de votre ordinateur de labo, démarrez un navigateur web, accédez au [portail Azure](https://portal.azure.com), puis connectez-vous en fournissant les informations d’identification Microsoft Entra d’un compte d’utilisateur disposant du rôle Propriétaire dans l’abonnement que vous allez utiliser dans ce labo et du rôle Administrateur général dans le locataire Microsoft Entra associé à cet abonnement.

    >**Important** : Vérifiez que vous utilisez un compte professionnel ou scolaire, et **non** un compte Microsoft.

1. Dans le Portail Azure, recherchez et sélectionnez **Azure Active Directory** pour accéder au locataire Microsoft Entra associé à l’abonnement Azure utilisé pour ce labo.
1. Dans le panneau Azure Active Directory, dans la barre verticale du menu de gauche, dans la section **Gérer**, cliquez sur **Utilisateurs**. 
1. Dans le panneau **Utilisateurs | Tous les utilisateurs (préversion)**, sélectionnez **aduser5**.
1. Dans le panneau **aduser5 | Profil**, dans la barre d’outils, cliquez sur **Modifier**. Dans la **section Paramètres** de la liste déroulante **Emplacement d’utilisation**, sélectionnez le pays où se trouve l’environnement du labo et cliquez sur **Enregistrer** dans la barre d’outils.
1. Dans le panneau **aduser5 | Profil**, dans la section **Identité**, identifiez le nom d’utilisateur principal du compte **aduser5**.

    >**Remarque** : Enregistrez cette valeur. Vous en aurez besoin plus tard dans ce labo.

1. Dans le panneau **Utilisateurs | Tous les utilisateurs (préversion)**, sélectionnez le compte d’utilisateur utilisé pour vous connecter au début de cette tâche et répétez l’étape précédente si votre compte n’a pas l’**Emplacement d’utilisation** affecté. 

    >**Remarque** : La propriété **Emplacement d’utilisation** doit être définie pour affecter des licences Microsoft Entra Premium P2 aux comptes d’utilisateur.

1. Dans le panneau **utilisateurs | Tous les utilisateurs (préversion)**, sélectionnez le compte d’utilisateur **aadsyncuser** et identifiez son nom d’utilisateur principal.

    >**Remarque** : Enregistrez cette valeur. Vous en aurez besoin plus tard dans ce labo.

1. Dans le Portail Azure, revenez au panneau **Vue d’ensemble** du locataire Microsoft Entra et, dans la section **Gérer** de la barre verticale du menu de gauche, cliquez sur **Licences**.
1. Dans le panneau **Licenses \| Vue d’ensemble**, dans la section **Gérer** de dans la barre verticale du menu de gauche, cliquez sur **Tous les produits**.
1. Dans le panneau **Licenses \| Tous les produits**, dans la barre d’outils, cliquez sur **+ Essayer/Acheter**.
1. Dans le panneau **Activer**, cliquez sur **Essai gratuit** dans la section **MICROSOFT ENTRA ID P2**, puis cliquez sur **Activer** et suivez les invites pour terminer le processus d’activation.
1. Dans le panneau **Licences – Tous les produits**, sélectionnez l’entrée **Enterprise Mobility + Security E5**. 
1. Dans le panneau **Enterprise Mobility + Security E5**, dans la barre d’outils, cliquez sur **+ Attribuer**.
1. Dans le panneau **Attribuer une licence**, cliquez sur **Ajouter des utilisateurs et des groupes**. Dans le panneau **Ajouter des utilisateurs et des groupes**, sélectionnez **aduser5**, ainsi que votre compte d’utilisateur, puis cliquez sur **Sélectionner**.
1. Revenu dans le panneau **Attribuer une licence**, cliquez sur **Options d’affectation**. Dans le panneau **Options d’affectation**, assurez-vous de l’activation de toutes les options, cliquez sur **Vérifier + attribuer**, cliquez sur **Attribuer**.

#### Tâche 2 : Configurer l’authentification multifacteur (MFA) Microsoft Entra

1. Sur votre ordinateur de labo, dans le navigateur web affichant le Portail Azure, revenez au panneau **Vue d’ensemble** du locataire Microsoft Entra et, dans le menu vertical de gauche, dans la section **Gérer**, cliquez sur **Sécurité**.
1. Dans le panneau **Sécurité | Bien démarrer**, dans la section **Protéger** du menu vertical de gauche, cliquez sur **Protection de l’identité**.
1. Dans le panneau **Protection de l’identité | Vue d’ensemble**, dans la section **Protéger** du menu vertical de gauche, cliquez sur **Stratégie d’inscription d’authentification multifacteur** (si nécessaire, actualisez la page du navigateur web).
1. Dans le panneau **Protection de l’identité | Stratégie d’inscription à l’authentification multifacteur**, dans la section **Affectations** de la **Stratégie d’inscription d’authentification multifacteur**, cliquez sur **Tous les utilisateurs**. Sous l’onglet **Inclure**, cliquez sur l’option **Sélectionner des individus et des groupes**, sur **Sélectionner des utilisateurs**, sur **Aduser5** et sur **Sélectionner**. En bas du panneau, définissez le commutateur **Appliquer la stratégie** sur **Activé** et cliquez sur **Enregistrer**.

#### Tâche 3 : Inscrire un utilisateur pour l’authentification multifacteur Microsoft Entra

1. Sur votre ordinateur de labo, ouvrez une session de navigateur web **InPrivate**, accédez au [Portail Azure](https://portal.azure.com) et connectez-vous en fournissant le nom d’utilisateur principal **aduser5** identifié précédemment dans cet exercice, ainsi que le mot de passe défini lors de la création de ce compte d’utilisateur.
1. Lorsque vous recevez le message **Plus d’informations requises**, cliquez sur **Suivant**. Cela redirige automatiquement votre navigateur vers la page **Microsoft Authenticator**.
1. Dans la page **Vérification de sécurité supplémentaire**, dans la section **Étape 1 : Comment devons-nous vous contacter ?**, sélectionnez votre méthode d’authentification préférée et suivez les instructions pour terminer le processus d’inscription. 
1. Dans la page Portail Azure, dans le coin supérieur droit, cliquez sur l’icône représentant l’avatar de l’utilisateur, puis sur **Se déconnecter** et fermez la fenêtre **Inprivate** du navigateur. 

#### Tâche 4 : Configurer la jointure hybride Microsoft Entra

> **Remarque** : Il est possible de tirer profit de cette fonctionnalité pour implémenter une sécurité supplémentaire lors de la configuration de l’accès conditionnel pour les appareils en fonction de leur état de jointure Microsoft Entra.

1. Sur l’ordinateur de labo, dans le navigateur web affichant le Portail Azure, recherchez et sélectionnez **machines virtuelles** et, dans le panneau **Machines virtuelles**, sélectionnez **az140-dc-vm11**.
1. Dans le panneau **az140-dc-vm11**, sélectionnez **Se connecter**, et dans le menu déroulant, sélectionnez **Se connecter via Bastion**.
1. Lorsque vous y êtes invité, fournissez les informations d’identification suivantes et sélectionnez **Connecter** :

   |Paramètre|Valeur|
   |---|---|
   |Nom d’utilisateur|**Étudiant**|
   |Mot de passe|**Pa55w.rd1234**|

1. Dans la session Bastion à **az140-dc-vm11**, dans le menu **Démarrer**, développez le dossier **Azure AD Connect** et sélectionnez **Azure AD Connect**.

   > **Remarque** Si vous obtenez une fenêtre d’erreur de défaillance indiquant que le service de synchronisation n’est pas en cours d’exécution, accédez à la fenêtre de commande PowerShell et entrez **Start-Service "ADSync"**, puis réessayez l’étape précédente.

1. Dans la page **Bienvenue dans Azure AD Connect** de la fenêtre **Microsoft Azure Active Directory Connect**, sélectionnez **Configurer**.
1. Dans la page **Tâches supplémentaires** de la fenêtre **Microsoft Azure Active Directory Connect**, sélectionnez **Configurer les options de l’appareil** et sélectionnez **Suivant**.
1. Dans la page **Vue d’ensemble** de la fenêtre **Microsoft Azure Active Directory Connect**, évaluez les informations relatives à la **jointure hybride Microsoft Entra** et la **réécriture d’appareil**, puis sélectionnez **Suivant**.
1. Dans la page **Connexion à Microsoft Entra** de la fenêtre **Microsoft Entra Connect**, authentifiez-vous à l’aide des informations d’identification du compte d’utilisateur **aadsyncuser** créé dans le labo précédent, puis sélectionnez **Suivant**.  
1. Dans la page **Options de l’appareil** de la fenêtre **Microsoft Azure Active Directory Connect**, assurez-vous de la sélection de l’option **Configurer la jointure Hybrid Azure AD** et sélectionnez **Suivant**. 
1. Dans la page **Systèmes d’exploitation de l’appareil** de la fenêtre **Microsoft Azure Active Directory Connect**, sélectionnez la case à cocher **Appareils joints à un domaine Windows 10 ou ultérieur**, puis **Suivant**. 
1. Dans la page **Configuration SCP** de la fenêtre **Microsoft Azure Active Directory Connect**, sélectionnez la case à cocher en regard de l’entrée **adatum.com**. Dans la liste déroulante **Service d’authentification**, sélectionnez l’entrée **Azure Active Directory**, puis **Ajouter**. 
1. Lorsque vous y êtes invité, dans la boîte de dialogue **Informations d’identification d’administrateur d’entreprise**, spécifiez les informations d’identification suivantes et sélectionnez **OK** :

   |Paramètre|Valeur|
   |---|---|
   |Nom d’utilisateur|**ADATUM\Student**|
   |Mot de passe|**Pa55w.rd1234**|

1. De retour sur la **page de configuration** SCP dans la **fenêtre Microsoft Azure Active Directory Connect** , sélectionnez **Suivant**.
1. Dans la page **Prêt à configurer** de la fenêtre **Microsoft Entra Microsoft Azure Active Directory Connect**, sélectionnez **Configurer**, puis **Quitter** une fois la configuration terminée.
1. Dans la session Bastion pour **az140-dc-vm11**, démarrez **Windows PowerShell ISE** en tant qu’administrateur.
1. Dans la session Bastion vers **az140-dc-vm11**, à partir de la console **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour déplacer le compte d’ordinateur **az140-cl-vm11** vers l’unité d’organisation (UO) **WVDClients** :

   ```powershell
   Move-ADObject -Identity "CN=az140-cl-vm11,CN=Computers,DC=adatum,DC=com" -TargetPath "OU=WVDClients,DC=adatum,DC=com"
   ```

1. Dans la session Bastion à **az140-dc-vm11**, dans le menu **Démarrer**, développez le dossier **Azure AD Connect** et sélectionnez **Azure AD Connect**.
1. Dans la page **Bienvenue dans Azure AD Connect** de la fenêtre **Microsoft Azure Active Directory Connect**, sélectionnez **Configurer**.
1. Dans la page **Tâches supplémentaires** de la fenêtre **Microsoft Azure Active Directory Connect**, sélectionnez **Personnaliser les options de synchronisation** et sélectionnez **Suivant**.
1. Dans la page **Connexion à Microsoft Entra** de la fenêtre **Microsoft Azure Active Directory Connect**, authentifiez-vous à l’aide des informations d’identification du compte d’utilisateur **aadsyncuser** créé dans l’exercice précédent, puis sélectionnez **Suivant**. 
1. Dans la page **Connecter vos annuaires** de la fenêtre **Microsoft Azure Active Directory Connect**, sélectionnez **Suivant**.
1. Dans la page **Filtrage du domaine et de l’unité d’organisation** de la fenêtre **Microsoft Azure Active Directory Connect**, assurez-vous de la sélection de l’option **Synchroniser les domaines et les unités d’organisation sélectionnés**, développez le nœud **adatum.com**, assurez-vous de la sélection de la case à cocher en regard de l’unité d’organisation **ToSync**, sélectionnez la case à cocher en regard de l’unité d’organisation **WVDClients**, puis **Suivant**.
1. Dans la page **Fonctionnalités facultatives** de la fenêtre **Microsoft Azure Active Directory Connect**, acceptez les paramètres par défaut et sélectionnez **Suivant**.
1. Dans la page **Prêt à configurer** de la fenêtre **Microsoft Azure Active Directory Connect**, vérifiez que la case **Démarrez le processus de synchronisation une fois la configuration terminée** est cochée et sélectionnez **Configurer**.
1. Vérifiez les informations de la page **Configuration terminée**, puis cliquez sur **Quitter** pour fermer la fenêtre **Microsoft Azure Active Directory Connect**.

#### Tâche 5 : Déclencher la synchronisation complète Microsoft Azure Active Directory Connect

1. À partir de votre ordinateur de labo, dans le Portail Azure, recherchez et sélectionnez **Machines virtuelles**, puis, dans le panneau **Machines virtuelles**, sélectionnez l’entrée**az140-cl-vm11**. Cette opération ouvre le panneau **az140-cl-vm11**.
1. Dans le panneau **az140-cl-vm11** , sélectionnez **Redémarrer** , puis attendez que la notification de **machine** virtuelle redémarrée avec succès s’affiche.
1. Dans la session Bastion vers **az140-dc-vm11**, basculez vers la fenêtre **Administrateur : Windows PowerShell ISE**.
1. Dans la session Bastion vers **az140-dc-vm11**, à partir de la console **Administrateur : Volet de la console Windows PowerShell ISE** , exécutez ce qui suit pour déclencher la synchronisation complète de Microsoft Azure Active Directory Connect :

   ```powershell
   Import-Module -Name "C:\Program Files\Microsoft Azure AD Sync\Bin\ADSync"
   Start-ADSyncSyncCycle -PolicyType Initial
   ```

1. Dans la session Bastion pour **az140-dc-vm11**, démarrez Microsoft Edge et accédez au [portail Azure](https://portal.azure.com). Lorsque vous y êtes invité, connectez-vous à l’aide des informations d’identification Microsoft Entra du compte d’utilisateur avec le rôle Administrateur général dans le locataire Microsoft Entra associé à l’abonnement Azure utilisé dans ce labo.
1. Dans la session Bastion à **az140-dc-vm11**, dans la fenêtre Microsoft Edge affichant le portail Azure, recherchez et sélectionnez **Microsoft Entra ID** pour accéder au locataire Microsoft Entra associé à l’abonnement Azure que vous utilisez pour ce labo.
1. Dans le panneau Microsoft Entra ID, dans la barre de menus verticale située à gauche, dans la section **Gérer**, cliquez sur **Appareils**. 
1. Dans le panneau **Appareils | Tous les appareils**, évaluez la liste des appareils et assurez-vous que l’appareil **az140-cl-vm11** est répertorié avec l’entrée **jointure Microsoft Entra Hybrid** dans la colonne **Type de jointure**.

   > **Remarque** : Quelques minutes peuvent être nécessaires avant que l’appareil n’apparaisse dans le Portail Azure.

### Exercice 2 : Implémenter l’accès conditionnel basé sur Microsoft Entra pour Azure Virtual Desktop

Les principales tâches de cet exercice sont les suivantes

1. Créer une stratégie d’accès conditionnel basée sur Microsoft Entra pour toutes les connexions Azure Virtual Desktop
1. Tester la stratégie d’accès conditionnel basée sur Microsoft Entra pour toutes les connexions Azure Virtual Desktop
1. Modifier la stratégie d’accès conditionnel basé sur Microsoft Entra pour exclure les ordinateurs à jointure hybride à Microsoft Entra de l’exigence d’authentification multifacteur
1. Tester la stratégie d’accès conditionnel modifiée et basée sur Microsoft Entra

#### Tâche 1 : Créer une stratégie d’accès conditionnel basée sur Microsoft Entra pour toutes les connexions Azure Virtual Desktop

>**Remarque** : Dans cette tâche, vous allez configurer une stratégie d’accès conditionnel basée sur Microsoft Entra qui nécessite une authentification multifacteur pour vous connecter à une session Azure Virtual Desktop. La stratégie applique également une réauthentification 4 premières heures après une authentification réussie.

1. Sur votre ordinateur de labo, dans le navigateur web affichant le Portail Azure, revenez au panneau **Vue d’ensemble** du locataire Microsoft Entra et, dans le menu vertical de gauche, dans la section **Gérer**, cliquez sur **Sécurité**.
1. Dans le panneau **Sécurité \| Bien démarrer**, dans la section **Protéger** du menu vertical de gauche, cliquez sur **Accès conditionnel**.
1. Dans le panneau **Accès conditionnel \| Stratégies**, dans la barre d’outils, cliquez sur **+ Nouvelle stratégie**.
1. Dans le volet **Nouveau**, configurez les éléments suivants :

   - Dans la zone de texte **Nom**, tapez **az140-31-wvdpolicy1**
   - Dans la section **Affectations**, sélectionnez l’option **Utilisateurs ou identités de charge de travail**. Dans la liste déroulante **À quoi cette stratégie s’applique-t-elle ?**, assurez-vous de la sélection des **Utilisateurs et les groupes**. Dans la section **Sélectionner des utilisateurs et des groupes**, sélectionnez la case à cocher **Utilisateurs et groupes**. Dans le panneau **Sélectionner**, cliquez sur **aduser5** et cliquez sur **Sélectionner**.
   - Dans la section **Affectations**, cliquez sur **Applications ou actions cloud**. Vérifiez que, dans le commutateur **Sélectionner ce à quoi cette stratégie s’applique**, l’option **Applications cloud** est sélectionnée, et cliquez sur l’option **Sélectionner des applications**. Dans le panneau **Sélectionner**, dans la zone de texte **Rechercher**, entrez **Azure Virtual Desktop**. Dans la liste des résultats, cochez la case en regard de l’entrée **Azure Virtual Desktop**. Dans la zone de texte **Rechercher**, entrez **Bureau à distance Microsoft**, cochez la case en regard de l’entrée **Bureau à distance Microsoft** et cliquez sur **Sélectionner**. 

   > **Remarque** : Azure Virtual Desktop (ID de l’application 9cdead84-a844-4324-93f2-b2e6bb768d07) est utilisé lorsque l’utilisateur s’abonne à un flux et s’authentifie auprès de la passerelle Azure Virtual Desktop pendant une connexion. Bureau à distance Microsoft (ID de l’application a4a365df-50f1-4397-bc59-1a1564b8bb9c) est utilisé lorsque l’utilisateur s’authentifie auprès de l’hôte de la session, avec l’authentification unique activée.

   - Dans la section **Affectations**, cliquez sur **Conditions**, puis **Applications clientes**. Dans le panneau **Applications clientes**, définissez le commutateur **Configurer** sur **Oui**. Assurez-vous de la sélection des cases à cocher **Navigateur** et **Applications mobiles et clients de bureau** et cliquez sur **Terminé**.
   - Dans la section **Contrôles d’accès**, cliquez sur **Octroi**. Dans le panneau **Octroi**, assurez-vous de la sélection de l’option **Accorder l’accès**, cochez la case **Exiger l’authentification multifacteur** et cliquez sur **Sélectionner**.
   - Dans la section **Contrôles Access**, cliquez sur **Session**. Dans le panneau **Session**, cochez la case **Fréquence de connexion**, dans la première zone de texte, tapez **4**, dans la liste déroulante **Sélectionner des unités**, sélectionnez **Heures**. Laissez la case **Session de navigateur persistante** décochée et cliquez sur **Sélectionner**.
   - Définissez le commutateur **Activer la stratégie** sur **Activé**.

1. Dans le panneau **Nouveau**, cliquez sur **Créer**. 

#### Tâche 2 : Tester la stratégie d’accès conditionnel basée sur Microsoft Entra pour toutes les connexions Azure Virtual Desktop

1. Sur l’ordinateur de labo et, dans la fenêtre du navigateur web affichant le Portail Azure, ouvrez la session de l’interpréteur de commandes **PowerShell** dans le panneau **Cloud Shell**.
1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez ce qui suit pour démarrer la session Azure Virtual Desktop héberge des machines virtuelles Azure que vous utiliserez dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**Remarque** : Attendez la fin de la commande et l’exécution de toutes les machines virtuelles Azure du groupe de ressources **az140-21-RG**. 

1. Sur votre ordinateur de labo, ouvrez une session de navigateur web **InPrivate**, accédez au [Portail Azure](https://portal.azure.com) et connectez-vous en fournissant le nom d’utilisateur principal **aduser5** identifié précédemment dans cet exercice, ainsi que le mot de passe défini lors de la création de ce compte d’utilisateur.

   > **Remarque** : Vérifiez que vous n’êtes pas invité à vous authentifier par une authentification multifacteur.

1. Dans la session de navigateur **InPrivate**, accédez à la page du client web HTML5 d’Azure Virtual Desktop sur [https://rdweb.wvd.microsoft.com/arm/webclient](https://rdweb.wvd.microsoft.com/arm/webclient).

   > **Remarque** : Vérifiez que cela déclenche automatiquement l’authentification par l’authentification multifacteur.

1. Dans le panneau **Entrer un code**, tapez le code à partir du SMS ou de l’application d’authentificateur inscrite et sélectionnez **Vérifier**.
1. Dans la page **Toutes les ressources**, cliquez sur **Invite de commandes**. Dans le panneau **Accéder aux ressources locales**, désactivez la case à cocher de l’**Imprimante** et cliquez sur **Autoriser**.
1. Lorsque vous y êtes invité, dans la zone de texte **Nom d’utilisateur** de la section **Entrer vos informations d’identification**, renseignez le nom d’utilisateur principal d’**aduser5**, puis saisissez le mot de passe défini lors de la création de ce compte d’utilisateur dans la zone de texte **Mot de passe** et cliquez sur **Envoyer**.
1. Assurez-vous que l’application distante de l’**invite de commandes** a bien démarré.
1. Dans la fenêtre d’**invite de commandes** de l’Application distante, tapez **déconnexion** et appuyez sur **Entrer**.
1. De retour dans la page **Toutes les ressources**, cliquez sur **aduser5** dans le coin supérieur droit. Dans le menu déroulant, cliquez sur **Se déconnecter** et fermez la fenêtre **InPrivate** du navigateur.

#### Tâche 3 : Modifier la stratégie d’accès conditionnel basé sur Microsoft Entra pour exclure les ordinateurs à jointure hybride à Microsoft Entra de l’exigence d’authentification multifacteur

>**Remarque** : Dans cette tâche, vous allez modifier la stratégie d’accès conditionnel basée sur Microsoft Entra qui nécessite l’authentification multifacteur pour se connecter à une session Azure Virtual Desktop, afin que les connexions provenant d’ordinateurs joints à Microsoft Entra ne nécessitent pas d’authentification multifacteur.

1. Sur votre ordinateur de labo, dans la fenêtre du navigateur affichant le Portail Azure, dans le panneau **Accès conditionnel | Stratégies**, cliquez sur l’entrée représentant la stratégie **az140-31-wvdpolicy1**.
1. Dans le panneau **az140-31-wvdpolicy1**, dans la section **Contrôles d’accès**, cliquez sur **Octroi**. Dans le panneau **Octroi**, sélectionnez les cases à cocher **Exiger l’authentification multifacteur** et **Exiger un appareil à jointure hybride Microsoft Entra**, assurez-vous de l’activation de l’option **Demander un des contrôles sélectionnés** et cliquez sur **Sélectionner**.
1. Dans le panneau **az140-31-wvdpolicy1**, cliquez sur **Enregistrer**.

>**Remarque** : La mise en œuvre de la stratégie peut prendre 15 minutes.

#### Tâche 4 : Tester la stratégie d’accès conditionnel modifiée et basée sur Microsoft Entra

1. Sur votre ordinateur de labo, dans la fenêtre du navigateur affichant le Portail Azure, recherchez et sélectionnez **machines virtuelles** et, dans le panneau **Machines virtuelles**, sélectionnez l’entrée **az140-cl-vm11**.
1. Dans le panneau **az140-cl-vm11**, sélectionnez **Connecter**, dans le menu déroulant, sélectionnez **Bastion**, sous l’onglet **Bastion** du panneau **az140-cl-vm11\| Connecter**, sélectionnez **Utiliser bastion**.
1. Lorsque vous y êtes invité, fournissez les informations d’identification suivantes et sélectionnez **Connecter** :

   |Paramètre|Valeur|
   |---|---|
   |Nom d’utilisateur|**Student@adatum.com**|
   |Mot de passe|**Pa55w.rd1234**|

1. Dans la session Bastion vers **az140-cl-vm11**, démarrez Microsoft Edge et accédez à la page du client web HTML5 d’Azure Virtual Desktop à l’adresse [https://rdweb.wvd.microsoft.com/arm/webclient](https://rdweb.wvd.microsoft.com/arm/webclient).

   > **Remarque** : Vérifiez cette fois, que vous n’allez pas être invité à vous authentifier par l’authentification multifacteur. Cela est dû au fait que **az140-cl-vm11** à une jointure hybride à Microsoft Entra.

1. Dans la page **Toutes les ressources**, double-cliquez sur **Invite de commandes**, dans le **volet Accéder aux ressources** locales, désactivez la **zone d’case activée de l’imprimante**, puis cliquez sur **Autoriser**.
1. Lorsque vous y êtes invité, dans la zone de texte **Nom d’utilisateur** de la section **Entrer vos informations d’identification**, renseignez le nom d’utilisateur principal d’**aduser5**, puis saisissez le mot de passe défini lors de la création de ce compte d’utilisateur dans la zone de texte **Mot de passe** et cliquez sur **Envoyer**.
1. Assurez-vous que l’application distante de l’**invite de commandes** a bien démarré.
1. Dans la fenêtre d’**invite de commandes** de l’Application distante, tapez **déconnexion** et appuyez sur **Entrer**.
1. De retour dans la page **Toutes les ressources**, dans le coin supérieur droit, cliquez sur **aduser5**, puis cliquez sur **Se déconnecter** dans le menu déroulant.
1. Dans la session Bastion vers **az140-cl-vm11**, cliquez sur **Démarrer**. Dans la barre verticale située directement au-dessus du bouton **Démarrer**, cliquez sur l’icône représentant le compte d’utilisateur connecté, puis cliquez sur **Se déconnecter** dans le menu contextuel.

### Exercice 3 : Arrêter et libérer les machines virtuelles Azure approvisionnées et utilisées dans le labo

Les principales tâches de cet exercice sont les suivantes

1. Arrêter et libérer les machines virtuelles Azure approvisionnées et utilisées dans le labo

>**Remarque** : Dans cet exercice, vous allez libérer les machines virtuelles Azure approvisionnées et utilisées dans ce labo pour réduire les frais de calcul correspondants

#### Tâche 1 : Libérer des machines virtuelles Azure approvisionnées et utilisées dans le labo

1. Basculez vers l’ordinateur labo et, dans la fenêtre du navigateur web affichant le Portail Azure, ouvrez la session de l’interpréteur de commandes **PowerShell** dans le volet **Cloud Shell**.
1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez la commande suivante pour répertorier toutes les machines virtuelles Azure créées et utilisées dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez ce qui suit pour arrêter et libérer toutes les machines virtuelles Azure que vous avez créées et utilisées dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Remarque** : La commande s’exécute de manière asynchrone (telle que déterminée par le paramètre -NoWait). Par conséquent, même si vous pourrez exécuter une autre commande PowerShell immédiatement après la même session PowerShell, il faudra quelques minutes avant que les machines virtuelles Azure ne soient réellement arrêtées et libérées.
