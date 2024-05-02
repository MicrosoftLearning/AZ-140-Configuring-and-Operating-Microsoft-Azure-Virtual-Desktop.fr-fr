---
lab:
  title: 'Labo : Déployer et gérer des pools d’hôtes et des hôtes en utilisant PowerShell (AD DS)'
  module: 'Module 2: Implement a WVD Infrastructure'
---

# Labo - Déployer et gérer des pools d’hôtes et des hôtes en utilisant PowerShell
# Manuel de labo de l’étudiant

## Dépendances du labo

- Un abonnement Azure que vous utiliserez dans ce labo.
- Un compte Microsoft ou un compte Azure AD avec le rôle Propriétaire ou Contributeur dans l’abonnement Azure que vous utiliserez dans ce labo et avec le rôle Administrateur général dans le locataire Azure AD associé à cet abonnement Azure.
- Le labo terminé **Préparer le déploiement d’Azure Virtual Desktop (AD DS)**

## Durée estimée

60 minutes

## Scénario du labo

Vous devez automatiser le déploiement des pools d’hôtes et des hôtes Azure Virtual Desktop en utilisant PowerShell dans un environnement Active Directory Domain Services (AD DS).

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

- Déployer des pools d’hôtes et des hôtes Azure Virtual Desktop en utilisant PowerShell
- Ajouter des hôtes au pool d’hôtes Azure Virtual Desktop en utilisant PowerShell

## Fichiers du labo

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.parameters.json

## Instructions

### Exercice 1 : Implémenter des pools d’hôtes et des hôtes de session Azure Virtual Desktop en utilisant PowerShell
  
Les principales tâches de cet exercice sont les suivantes

1. Préparer le déploiement d’un pool d’hôtes Azure Virtual Desktop en utilisant PowerShell
1. Créer un pool d’hôtes Azure Virtual Desktop en utilisant PowerShell
1. Effectuer un déploiement basé sur un modèle d’une machine virtuelle Azure exécutant Windows 11 Entreprise en utilisant PowerShell
1. Ajouter une machine virtuelle Azure exécutant Windows 11 Entreprise en tant qu’hôte de session au pool d’hôtes Azure Virtual Desktop en utilisant PowerShell
1. Vérifier le déploiement de l’hôte de session Azure Virtual Desktop

#### Tâche 1 : Préparer le déploiement d’un pool d’hôtes Azure Virtual Desktop en utilisant PowerShell

1. À partir de votre ordinateur de labo, démarrez un navigateur web, accédez au [portail Azure](https://portal.azure.com), puis connectez-vous en utilisant les informations d’identification d’un compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utiliserez dans ce labo.
1. Dans le Portail Azure, recherchez et sélectionnez **machines virtuelles** et, dans le panneau **Machines virtuelles**, sélectionnez **az140-dc-vm11**.
1. Dans le panneau **az140-dc-vm11**, sélectionnez **Se connecter**, et dans le menu déroulant, sélectionnez**Se connecter via Bastion**.
1. Lorsque vous y êtes invité, fournissez les informations d’identification suivantes et sélectionnez **Se connecter** :

   |Paramètre|Valeur|
   |---|---|
   |Nom d’utilisateur|**Étudiant**|
   |Mot de passe|**Pa55w.rd1234**|

1. Dans la session Bastion pour **az140-dc-vm11**, démarrez **Windows PowerShell ISE** en tant qu’administrateur.
1. Dans la session Bastion sur **az140-dc-vm11**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour installer le module PowerShell DesktopVirtualization (lorsque vous y êtes invité, cliquez sur **Oui pour tout**) :

   ```powershell
   Install-Module -Name Az.DesktopVirtualization -Force
   ```

   > **Remarque** : Ignorez les avertissements relatifs aux modules PowerShell existants en cours d’utilisation.

1. Dans la session Bastion sur **az140-dc-vm11**, démarrez Microsoft Edge et accédez au [portail Azure](https://portal.azure.com). Si vous y êtes invité, connectez-vous avec les informations d’identification Azure AD du compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utilisez dans ce labo.
1. Dans la session Bastion sur **az140-dc-vm11**, dans le portail Azure, utilisez la zone de texte **Rechercher dans les ressources, services et documents** en haut de la page pour rechercher et accéder à **Réseaux virtuels**, puis dans le panneau **Réseaux virtuels**, sélectionnez **az140-adds-vnet11**. 
1. Dans le panneau **az140-add-vnet11**, sélectionnez **Sous-réseaux**, dans le panneau **Sous-réseaux**, sélectionnez **+ Sous-réseau**, dans le panneau **Ajouter un sous-réseau**, spécifiez les paramètres suivants (laissez tous les autres paramètres avec leurs valeurs par défaut) et cliquez sur **Enregistrer**:

   |Paramètre|Valeur|
   |---|---|
   |Nom|**hp3-Subnet**|
   |Plage d’adresses de sous-réseau|**10.0.3.0/24**|

#### Tâche 2 : Créer un pool d’hôtes Azure Virtual Desktop en utilisant PowerShell

1. Dans la session Bastion sur **az140-dc-vm11**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour identifier la région Azure hébergeant le réseau virtuel Azure **az140-adds-vnet11** :

   ```powershell
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   ```

1. Dans la session Bastion sur **az140-dc-vm11**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour créer un groupe de ressources qui hébergera le pool d’hôtes et ses ressources :

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   ```

1. Dans la session Bastion sur **az140-dc-vm11**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour créer un pool d’hôtes vide :

   ```powershell
   $hostPoolName = 'az140-24-hp3'
   $workspaceName = 'az140-24-ws1'
   $dagAppGroupName = "$hostPoolName-DAG"
   New-AzWvdHostPool -ResourceGroupName $resourceGroupName -Name $hostPoolName -WorkspaceName $workspaceName -HostPoolType Pooled -LoadBalancerType BreadthFirst -Location $location -DesktopAppGroupName $dagAppGroupName -PreferredAppGroupType Desktop 
   ```

   > **Remarque** : La cmdlet **New-AzWvdHostPool** vous permet de créer un pool d’hôtes, un espace de travail et le groupe d’applications de bureau, ainsi que d’inscrire le groupe d’applications de bureau auprès de l’espace de travail. Vous avez la possibilité de créer un nouvel espace de travail ou d’en utiliser un qui existe déjà.

1. Dans la session Bastion sur **az140-dc-vm11**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour récupérer l’attribut objectID du groupe Azure AD nommé **az140-wvd-pooled** :

   ```powershell
   $aadGroupObjectId = (Get-AzADGroup -DisplayName 'az140-wvd-pooled').Id
   ```

1. Dans la session Bastion sur **az140-dc-vm11**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour affecter le groupe Azure AD nommé **az140-wvd-pooled** au groupe d’applications de bureau par défaut du pool d’hôtes qui vient d’être créé :

   ```powershell
   $roleDefinitionName = 'Desktop Virtualization User'
   New-AzRoleAssignment -ObjectId $aadGroupObjectId -RoleDefinitionName $roleDefinitionName -ResourceName $dagAppGroupName -ResourceGroupName $resourceGroupName -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
   ```

#### Tâche 3 : Effectuer un déploiement basé sur un modèle d’une machine virtuelle Azure exécutant Windows 11 Entreprise en utilisant PowerShell

1. Sur votre ordinateur de labo, accédez au compte de stockage déployé. Dans le panneau Partage de fichiers, sélectionnez le partage de fichiers **az140-22-profiles**.

1. Sélectionnez **Charger** et chargez les fichiers de labo **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.json** et **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.parameters.json** dans le partage de fichiers.

1. Dans la session Bastion sur **az140-dc-vm11**, ouvrez l’Explorateur de fichiers et accédez au lecteur Z: précédemment configuré ou à la lettre de lecteur affectée à la connexion au partage de fichiers. Copiez les fichiers de déploiement chargés dans **C:\AllFiles\Labs\02**.

1. Dans la session Bastion vers **az140-dc-vm11**, à partir de la console **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour déployer une machine virtuelle Azure exécutant Windows 11 Entreprise (multisession) qui servira d’hôte de session Azure Virtual Desktop dans le pool d’hôtes que vous avez créé dans la tâche précédente :

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab24hp3Deployment `
     -TemplateFile C:\AllFiles\Labs\02\az140-24_azuredeployhp3.json `
     -TemplateParameterFile C:\AllFiles\Labs\02\az140-24_azuredeployhp3.parameters.json
   ```

   > **Remarque** : Attendez la fin du déploiement avant de passer à la tâche suivante. Ceci peut prendre environ 5-10 minutes. 

   > **Remarque** : Le déploiement utilise un modèle Azure Resource Manager pour provisionner une machine virtuelle Azure et applique une extension de machine virtuelle qui joint automatiquement le système d’exploitation au domaine AD DS **adatum.com**.

1. Dans la session Bastion sur **az140-dc-vm11**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour vérifier que le troisième hôte de session a été correctement joint au domaine AD DS **adatum.com** :

   ```powershell
   Get-ADComputer -Filter "sAMAccountName -eq 'az140-24-p3-0$'"
   ```

#### Tâche 4 : Ajouter une machine virtuelle Azure exécutant Windows 11 Entreprise en tant qu’hôte au pool d’hôtes Azure Virtual Desktop en utilisant PowerShell

1. Dans la session Bastion sur **az140-dc-vm11**, dans la fenêtre de navigateur affichant le portail Azure, recherchez et sélectionnez **Machines virtuelles**, puis dans le panneau **Machines virtuelles**, sélectionnez **az140-24-p3-0** dans la liste des machines virtuelles.
1. Dans le panneau **az140-24-p3-0**, sélectionnez **Se connecter**, dans le menu déroulant, sélectionnez **Se connecter**
1. Vérifier la **connexion en utilisant** **l’adresse IP privée | 10.0.3.4**
1. Dans la section **RDP native**, sélectionnez **Télécharger le fichier RDP**.
1. Si vous y êtes invité, sélectionnez **Conserver**, puis cliquez sur le fichier az140-24-p3-0.rdp **téléchargé**.
1. À l’invite, connectez-vous avec les informations d’identification suivantes :

   |Paramètre|Valeur|
   |---|---|
   |User Name|**ADATUM\\Student**|
   |Mot de passe|**Pa55w.rd1234**|

1. Dans la session Bureau à distance sur **az140-24-p3-0**, démarrez **Windows PowerShell ISE** en tant qu’administrateur.
1. Dans la session Bureau à distance sur **az140-24-p3-0**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour créer un dossier qui hébergera les fichiers requis pour ajouter la machine virtuelle Azure nouvellement déployée en tant qu’hôte de session au pool d’hôtes que vous avez provisionné précédemment dans ce labo :

   ```powershell
   $labFilesFolder = 'C:\AllFiles\Labs\02'
   New-Item -ItemType Directory -Path $labFilesFolder
   ```

   >**Remarque** Utilisez la construction [T] avec prudence pour copier les cmdlets PowerShell. Dans certains cas, le texte copié peut être incorrect, par exemple le signe $ peut s’afficher sous la forme du chiffre 4. Vous devrez les corriger avant d’émettre la cmdlet. Faites une copie dans le volet **Script** de PowerShell ISE, apportez-y les corrections, puis sélectionnez le texte corrigé et appuyez sur **F8** (**Exécuter la sélection**).

1. Dans la session Bureau à distance sur **az140-24-p3-0**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour télécharger les programmes d’installation de l’agent et du chargeur d’amorçage Azure Virtual Desktop qui sont nécessaires pour ajouter l’hôte de session au pool d’hôtes :

   ```powershell
   $webClient = New-Object System.Net.WebClient
   $wvdAgentInstallerURL = 'https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrmXv'
   $wvdAgentInstallerName = 'WVD-Agent.msi'
   $webClient.DownloadFile($wvdAgentInstallerURL,"$labFilesFolder/$wvdAgentInstallerName")
   $wvdBootLoaderInstallerURL = 'https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrxrH'
   $wvdBootLoaderInstallerName = 'WVD-BootLoader.msi'
   $webClient.DownloadFile($wvdBootLoaderInstallerURL,"$labFilesFolder/$wvdBootLoaderInstallerName")
   ```

1. Depuis la console **Administrateur : Console Windows PowerShell ISE** , exécutez ce qui suit pour installer la dernière version du module Az PowerShell, quand vous êtes invité à installer le fournisseur NuGet, appuyez sur **Y**:

   ```powershell
   Install-Module -Name Az -Force
   ```

   > **Remarque** : Vous devrez peut-être attendre trois à cinq minutes avant qu’une sortie de l’installation du module Az s’affiche. Vous devrez peut-être également attendre cinq minutes **après** l’arrêt de la sortie. Ce comportement est normal.

1. Depuis la console **Administrateur : Console Windows PowerShell ISE** , exécutez la commande suivante pour vous connecter à votre abonnement Azure :

   ```powershell
   Connect-AzAccount
   ```

1. Lorsque vous y êtes invité, fournissez les informations d’identification du compte d’utilisateur avec le rôle Propriétaire dans l’abonnement que vous utilisez dans ce labo.
1. Dans la session Bureau à distance sur **az140-24-p3-0**, à partir de la console **Administrateur : Windows PowerShell ISE**, exécutez la commande suivante pour générer le jeton nécessaire pour joindre cet hôte de la session au pool que vous avez provisionné précédemment dans cet exercice :

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   $hostPoolName = 'az140-24-hp3'
   $registrationInfo = New-AzWvdRegistrationInfo -ResourceGroupName $resourceGroupName -HostPoolName $hostPoolName -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```
   > **Remarque** : Un jeton d’inscription est nécessaire pour autoriser un hôte de session à rejoindre le pool d’hôtes. La valeur de la date d’expiration du jeton doit être comprise entre une heure et un mois, à partir de la date et de l’heure actuelles.

1. Dans la session Bureau à distance sur **az140-24-p3-0**, à partir de la console **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour installer l’agent Azure Virtual Desktop :

   ```powershell
   Set-Location -Path $labFilesFolder
   Start-Process -FilePath 'msiexec.exe' -ArgumentList "/i $WVDAgentInstallerName", "/quiet", "/qn", "/norestart", "/passive", "REGISTRATIONTOKEN=$($registrationInfo.Token)", "/l* $labFilesFolder\AgentInstall.log" | Wait-Process
   ```

1. Dans la session Bureau à distance sur **az140-24-p3-0**, à partir de la console **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour installer le chargeur d’amorçage Azure Virtual Desktop :

   ```powershell
   Start-Process -FilePath "msiexec.exe" -ArgumentList "/i $wvdBootLoaderInstallerName", "/quiet", "/qn", "/norestart", "/passive", "/l* $labFilesFolder\BootLoaderInstall.log" | Wait-process
   ```

1. Dans la session Bureau à distance **az140-24-p3-0**, cliquez avec le bouton droit sur **Démarrer**. Dans le menu contextuel, sélectionnez **Arrêter ou déconnecter**, puis cliquez sur **Se déconnecter** dans le menu en cascade.

#### Tâche 5 : Vérifier le déploiement de l’hôte Azure Virtual Desktop

1. Basculez sur l’ordinateur de labo, puis dans le navigateur web affichant le portail Azure, recherchez et sélectionnez **Azure Virtual Desktop**. Dans le panneau **Azure Virtual Desktop**, sélectionnez **Pools d’hôtes**, puis dans le panneau **Azure Virtual Desktop \| Pools d’hôtes**, sélectionnez l’entrée **az140-24-hp3** qui représente le pool nouvellement modifié.
1. Dans le panneau **az140-24-hp3**, dans la section **Gérer** du menu vertical à gauche, cliquez sur **Hôtes de session**. 
1. Dans le panneau **az140-24-hp3 \| Hôtes de session**, vérifiez que le déploiement inclut un seul hôte.

#### Tâche 6 : Gérer des groupes d’applications à l’aide de PowerShell

1. Sur l’ordinateur de labo, basculez vers la session Bastion sur **az140-dc-vm11** et dans la console **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour créer un groupe Application à distance :

   ```powershell
   $subscriptionId = (Get-AzContext).Subscription.Id
   $appGroupName = 'az140-24-hp3-Office365-RAG'
   $resourceGroupName = 'az140-24-RG'
   $hostPoolName = 'az140-24-hp3'
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   New-AzWvdApplicationGroup -Name $appGroupName -ResourceGroupName $resourceGroupName -ApplicationGroupType 'RemoteApp' -HostPoolArmPath "/subscriptions/$subscriptionId/resourcegroups/$resourceGroupName/providers/Microsoft.DesktopVirtualization/hostPools/$hostPoolName"-Location $location
   ```

1. Dans la session Bastion sur **az140-dc-vm11**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour lister les applications du menu **Démarrer** sur les hôtes du pool et examiner la sortie :

   ```powershell
   Get-AzWvdStartMenuItem -ApplicationGroupName $appGroupName -ResourceGroupName $resourceGroupName | Format-List | more
   ```

   > **Remarque** : Pour toute application que vous souhaitez publier, vous avez tout intérêt à enregistrer les informations incluses dans la sortie, notamment les paramètres tels que **FilePath**, **IconPath** et **IconIndex**.

1. Dans la session Bastion sur **az140-dc-vm11**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour publier Microsoft Word :

   ```powershell
   $name = 'Microsoft Word'
   $filePath = 'C:\Program Files\Microsoft Office\root\Office16\WINWORD.EXE'
   $iconPath = 'C:\Program Files\Microsoft Office\Root\VFS\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\wordicon.exe'
   New-AzWvdApplication -GroupName $appGroupName -Name $name -ResourceGroupName $resourceGroupName -FriendlyName $name -Filepath $filePath -IconPath $iconPath -IconIndex 0 -CommandLineSetting 'DoNotAllow' -ShowInPortal:$true
   ```

1. Dans la session Bastion sur **az140-dc-vm11**, à partir du volet de script **Administrateur : Windows PowerShell ISE**, exécutez ce qui suit pour publier Microsoft Word :

   ```powershell
   $aadGroupObjectId = (Get-AzADGroup -DisplayName 'az140-wvd-remote-app').Id
   New-AzRoleAssignment -ObjectId $aadGroupObjectId -RoleDefinitionName 'Desktop Virtualization User' -ResourceName $appGroupName -ResourceGroupName $resourceGroupName -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
   ```

1. Basculez sur l’ordinateur de labo et dans le navigateur web affichant le portail Azure, dans le panneau **az140-24-hp3 \| Hôtes de session**, dans la section **Gérer** du menu vertical à gauche, sélectionnez **Groupes d’applications**.
1. Dans le panneau **az140-24-hp3 \| Groupes d’applications**, dans la liste des groupes d’applications, sélectionnez l’entrée **az140-24-hp3-Office365-RAG**.
1. Dans le panneau **az140-24-hp3-Office365-RAG**, vérifiez la configuration du groupe d’applications, y compris les applications et les affectations.

### Exercice 2 : Arrêter et libérer les machines virtuelles Azure provisionnées dans le labo

Les principales tâches de cet exercice sont les suivantes

1. Arrêter et libérer des machines virtuelles Azure approvisionnées dans le labo

>**Remarque** : Dans cet exercice, vous allez libérer les machines virtuelles Azure approvisionnées dans ce labo pour réduire les frais de calcul correspondants

#### Tâche 1 : Libérer des machines virtuelles Azure approvisionnées dans le labo

1. Basculez vers l’ordinateur labo et, dans la fenêtre du navigateur web affichant le Portail Azure, ouvrez la session de l’interpréteur de commandes **PowerShell** dans le volet **Cloud Shell**.
1. Dans la session PowerShell dans le volet Cloud Shell, exécutez la commande suivante pour répertorier toutes les machines virtuelles Azure créées dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-24-RG'
   ```

1. À partir de la session PowerShell dans le volet Cloud Shell, exécutez ce qui suit pour arrêter et libérer toutes les machines virtuelles Azure que vous avez créées dans ce labo :

   ```powershell
   Get-AzVM -ResourceGroup 'az140-24-RG' | Stop-AzVM -NoWait -Force
   ```

   >**Remarque** : La commande s’exécute de manière asynchrone (telle que déterminée par le paramètre -NoWait). Par conséquent, même si vous pourrez exécuter une autre commande PowerShell immédiatement après la même session PowerShell, il faudra quelques minutes avant que les machines virtuelles Azure ne soient réellement arrêtées et libérées.
