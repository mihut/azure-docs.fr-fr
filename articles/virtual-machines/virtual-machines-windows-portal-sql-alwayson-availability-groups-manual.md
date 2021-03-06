---
title: Configuration manuelle d’un groupe de disponibilité Always On dans une machine virtuelle Azure - Microsoft Azure
description: Créez un groupe de disponibilité Always On avec Azure Virtual Machines. Ce didacticiel utilise l'interface utilisateur et des outils, plutôt que des scripts.
services: virtual-machines
documentationcenter: na
author: MikeRayMSFT
manager: timlt
editor: monicar
tags: azure-service-management

ms.service: virtual-machines
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: infrastructure-services
ms.date: 09/22/2016
ms.author: MikeRayMSFT

---
# <a name="configure-always-on-availability-group-in-azure-vm-manually---resource-manager"></a>Configuration manuelle d’un groupe de disponibilité Always On dans une machine virtuelle Azure à l’aide de Resource Manager
> [!div class="op_single_selector"]
> * [Resource Manager : modèle](virtual-machines-windows-portal-sql-alwayson-availability-groups.md)
> * [Resource Manager : mode manuel](virtual-machines-windows-portal-sql-alwayson-availability-groups-manual.md)
> * [Classic : interface utilisateur](virtual-machines-windows-classic-portal-sql-alwayson-availability-groups.md)
> * [Classic : PowerShell](virtual-machines-windows-classic-ps-sql-alwayson-availability-groups.md)
> 
> 

<br/>

Ce didacticiel complet vous montre comment implémenter des groupes de disponibilité SQL Server dans des machines virtuelles Azure Resource Manager. 

Dans ce didacticiel, vous allez créer les éléments suivants :

* un réseau virtuel contenant deux sous-réseaux, notamment un sous-réseau frontal et un sous-réseau principal ;
* deux contrôleurs de domaine dans un groupe à haute disponibilité avec un domaine Active Directory (AD) ; 
* deux machines virtuelles SQL Server dans un groupe à haute disponibilité, déployées dans le sous-réseau principal et jointes au domaine AD ;
* un cluster WSFC à 3 nœuds avec le modèle de quorum Nœud majoritaire ;
* un équilibreur de charge interne pour fournir une adresse IP aux groupes de disponibilité ;
* un groupe de disponibilité avec deux réplicas avec validation synchrone d'une base de données de disponibilité.

La figure suivante est une représentation graphique de la solution.

![Architecture de groupe de disponibilité dans Azure Resource Manager](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/00-EndstateSampleNoELB.png)

Il s’agit d’une configuration possible. Vous pouvez la modifier selon vos besoins. Par exemple, vous pouvez réduire le nombre de machines virtuelles en utilisant un contrôleur de domaine comme témoin de partage de fichiers du quorum. Cela permet de réduire le nombre de machines virtuelles pour un groupe de disponibilité de deux réplicas. Cette méthode permet de se passer d’une machine virtuelle par rapport à la solution.

Ce didacticiel prend quelques heures, car vous devez créer et configurer plusieurs machines virtuelles Azure. Vous pouvez également générer l’ensemble cette solution automatiquement. Dans le portail Azure, il existe une installation de la galerie pour les groupes de disponibilité Always On avec un écouteur. La procédure d’installation de la galerie configure automatiquement tous les éléments nécessaires pour les groupes de disponibilité. Pour plus d’informations, voir [Portail - Resource Manager](virtual-machines-windows-portal-sql-alwayson-availability-groups.md). 

[!INCLUDE [availability-group-template](../../includes/virtual-machines-windows-portal-sql-alwayson-ag-template.md)]

Ce didacticiel part des principes suivants :

* vous disposez déjà d’un compte Azure ;
* vous savez déjà comment configurer une machine virtuelle SQL Server dans la galerie de machines virtuelles avec l'interface graphique utilisateur. Pour plus d'informations, consultez [Configuration d'une machine virtuelle SQL Server dans Azure](virtual-machines-windows-portal-sql-server-provision.md)
* Vous disposez déjà d’une connaissance approfondie des groupes de disponibilité. Pour plus d’informations, voir [Groupes de disponibilité AlwaysOn (SQL Server)](https://msdn.microsoft.com/library/hh510230.aspx).

> [!NOTE]
> Si l’utilisation des groupes de disponibilité avec SharePoint vous intéresse, consultez également [Configurer des groupes de disponibilité AlwaysOn SQL Server 2012 pour SharePoint 2013](https://technet.microsoft.com/library/jj715261.aspx).
> 
> 

## <a name="create-resource-group"></a>Créer un groupe de ressources
1. Connectez-vous au [portail Azure](http://portal.azure.com). 
2. Cliquez sur **+Nouveau**, puis saisissez **Groupe de ressources** dans la fenêtre de recherche **Marketplace**.
   
   ![Groupe de ressources](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/01-resourcegroupsymbol.png)
3. Cliquez sur **Groupe de ressources** 
   
   ![Nouveau groupe de ressources](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/01-newresourcegroup.png)
4. Cliquez sur **Create**. 
5. Dans le panneau **Groupe de ressources**, sous **Nom du groupe de ressources**, saisissez **SQL-HA-RG**.
6. Si vous avez plusieurs abonnements Azure, vérifiez qu’il s’agit bien de celui dans lequel vous souhaitez créer le groupe de disponibilité. 
7. Sélectionnez un emplacement. L’emplacement est la région Azure où vous souhaitez créer le groupe de disponibilité. Pour ce didacticiel, nous allons générer toutes les ressources dans un même emplacement Azure. 
8. Assurez-vous que l’option **Épingler au tableau de bord** est activée. Ce paramètre facultatif place un raccourci vers le groupe de ressources sur le tableau de bord du Portail Azure. 
9. Cliquez sur **Créer** pour créer le groupe de ressources.

Azure crée le groupe de ressources et épingle un raccourci vers celui-ci dans le portail.

## <a name="create-network-and-subnets"></a>Créer un réseau et des sous-réseaux
L’étape suivante consiste à créer les réseaux et les sous-réseaux dans le groupe de ressources Azure.

La solution utilise un réseau virtuel avec deux sous-réseaux. Pour plus d’informations sur les réseaux dans Azure, consultez la page [Présentation du réseau virtuel](../virtual-network/virtual-networks-overview.md) .

Pour créer le réseau virtuel :

1. Dans le portail Azure, cliquez sur le nouveau groupe de ressources, puis sur **+** pour ajouter un nouvel élément au groupe de ressources. Azure ouvre le panneau **Tout** . 
   
   ![Nouvel élément](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/02-newiteminrg.png)
2. Recherchez **Réseau virtuel**.
   
   ![Recherche d’un réseau virtuel](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/04-findvirtualnetwork.png)
3. Cliquez sur **Réseau virtuel**.
4. Dans le panneau **Réseau virtuel**, cliquez sur le modèle de déploiement **Resource Manager**, puis sur **Créer**.
   
   ![Créer un réseau virtuel](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/05-createvirtualnetwork.png)
5. Dans le panneau **Créer un réseau virtuel** , configurez le réseau virtuel. 
   
   Le tableau suivant indique les paramètres relatifs au réseau virtuel. 
   
   | **Champ** | Valeur |
   | --- | --- |
   | **Nom** |autoHAVNET |
   | **Espace d’adressage** |10.0.0.0/16 |
   | **Nom du sous-réseau** |Subnet-1 |
   | **Plage d’adresses de sous-réseau** |10.0.0.0/24 |
   | **Abonnement** |Spécifiez l’abonnement à utiliser. Si vous ne disposez que d’un abonnement, cette valeur peut être vide. |
   | **Emplacement** |Spécifiez l’emplacement Azure. |
   
   Votre espace d’adressage et votre plage d’adresses de sous-réseau peuvent être différents de ceux indiqués dans ce tableau. En fonction de votre abonnement, Azure spécifie un espace d’adressage disponible et la plage d’adresses de sous-réseau correspondante. Si vous ne disposez pas d’un espace d’adressage suffisant, utilisez un autre abonnement. 
6. Cliquez sur **Créer**
   
   ![Configuration d’un réseau virtuel](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/06-configurevirtualnetwork.png)

Vous revenez au tableau de bord du portail et Azure vous avertit lorsque le réseau est créé.

### <a name="create-the-second-subnet"></a>Créer le deuxième sous-réseau
À ce stade, votre réseau virtuel contient un sous-réseau, nommé Subnet-1. Les contrôleurs de domaine utilisent ce sous-réseau. Les serveurs SQL utilisent un deuxième sous-réseau nommé **Subnet-2**. Pour configurer Subnet-2 :

1. Dans votre tableau de bord, cliquez sur le groupe de ressources que vous avez créé, **SQL-HA-RG**. Recherchez le réseau dans le groupe de ressources, sous **Ressources**.
   
   Si **SQL-HA-RG** n’est pas visible, cliquez sur **Groupes de ressources** et filtrez en fonction du nom du groupe de ressources.
2. Cliquez sur **autoHAVNET** dans la liste des ressources. Azure ouvre le panneau de configuration de réseau.
3. Dans le réseau virtuel **autoHAVNET** , cliquez sur **Tous les paramètres*.
4. Dans le panneau **Paramètres**, cliquez sur **Sous-réseaux**.
   
   Vous pouvez voir le sous-réseau que vous avez déjà créé. 
   
   ![Configuration d’un réseau virtuel](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/07-addsubnet.png)
5. Créez un deuxième sous-réseau. Cliquez sur **+ Sous-réseau**. 
   
   Dans le panneau **Ajouter un sous-réseau**, configurez le sous-réseau en saisissant **subnet-2** sous **Nom**. Azure spécifie automatiquement une **plage d’adresses**valide. Vérifiez que cette plage d’adresses comporte au moins 10 adresses. Dans un environnement de production, vous pouvez avoir besoin de davantage d’adresses. 
6. Cliquez sur **OK**.

![Configuration d’un réseau virtuel](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/08-configuresubnet.png)

Voici un résumé des paramètres de configuration relatifs au réseau virtuel et aux deux sous-réseaux.

| **Champ** | Valeur |
| --- | --- |
| **Nom** |**autoHAVNET** |
| **Espace d’adressage** |Dépend des espaces d’adressage disponibles dans votre abonnement. 10.0.0.0/16 est une valeur courante. |
| **Nom du sous-réseau** |**Subnet-1** |
| **Plage d’adresses de sous-réseau** |Dépend des plages d’adresses disponibles dans votre abonnement. 10.0.0.0/24 est une valeur courante. |
| **Nom du sous-réseau** |**Subnet-2** |
| **Plage d’adresses de sous-réseau** |Dépend des plages d’adresses disponibles dans votre abonnement. 10.0.1.0/24 est une valeur courante. |
| **Abonnement** |Spécifiez l’abonnement à utiliser. |
| **Groupe de ressources** |**SQL-HA-RG** |
| **Emplacement** |Spécifiez le même emplacement que celui choisi pour le groupe de ressources. |

## <a name="create-availability-sets"></a>Créer des groupes à haute disponibilité
Avant de créer des machines virtuelles, vous devez créer des groupes à haute disponibilité. Les groupes à haute disponibilité réduisent le temps d’arrêt lors des événements de maintenance planifiée et non planifiée. Un groupe à haute disponibilité Azure est un groupe de ressources logique qu’Azure place dans des domaines de mise à jour et d’erreur physiques. Un domaine d’erreur garantit que les membres du groupe à haute disponibilité disposent de ressources réseau et d’une alimentation distinctes. Un domaine de mise à jour permet de s’assurer que les membres du groupe à haute disponibilité ne sont pas arrêtés pour maintenance simultanément. [Gestion de la disponibilité des machines virtuelles](virtual-machines-windows-manage-availability.md).

Vous avez besoin de deux groupes à haute disponibilité. L’un pour les contrôleurs de domaine. L’autre pour les serveurs SQL.

Pour créer un groupe à haute disponibilité, accédez au groupe de ressources, puis cliquez sur **Ajouter**. Filtrez les résultats en tapant **Groupe à haute disponibilité**. Cliquez sur **Groupe à haute disponibilité** dans les résultats. Cliquez sur **Create**.

Configurez deux groupes à haute disponibilité en vous basant sur les paramètres du tableau suivant.

| **Champ** | Groupe à haute disponibilité du contrôleur de domaine | Groupe à haute disponibilité SQL Server |
| --- | --- | --- |
| **Nom** |adavailabilitySet |sqlAvailabilitySet |
| **Groupe de ressources** |SQL-HA-RG |SQL-HA-RG |
| **Domaines d'erreur** |3 |3 |
| **Domaines de mise à jour** |5 |3 |

Après avoir créé les groupes à haute disponibilité, revenez au groupe de ressources dans le Portail Azure.

## <a name="create-domain-controllers"></a>Création de contrôleurs de domaine
Une fois que vous avez créé le réseau, les sous-réseaux, les groupes à haute disponibilité et un équilibreur de charge accessible sur Internet, vous êtes prêt à créer les machines virtuelles pour les contrôleurs de domaine.

### <a name="create-the-virtual-machines-for-the-domain-controllers"></a>Créer les machines virtuelles pour les contrôleurs de domaine
Pour créer et configurer les contrôleurs de domaine, revenez au groupe de ressources **SQL-HA-RG** .

1. Cliquez sur Ajouter. Le panneau **Tout** s’affiche.
2. Tapez **Windows Server 2012 R2 Datacenter**. 
3. Cliquez sur **Windows Server 2012 R2 Datacenter**. Dans le panneau **Windows Server 2012 R2 Datacenter**, vérifiez que le modèle de déploiement est défini sur **Resource Manager**, puis cliquez sur **Créer**. Azure ouvre le panneau **Créer une machine virtuelle** . 

Procédez comme suit pour créer deux machines virtuelles. Nommez les deux machines virtuelles comme suit :

* ad-primary-dc
* ad-secondary-dc
  
  > [!NOTE]
  > **ad-secondary-dc** est un composant facultatif utilisé pour fournir une haute disponibilité aux services de domaine Active Directory. 
  > 
  > 

Le tableau suivant indique les paramètres relatifs à ces deux machines.

| **Champ** | Valeur |
| --- | --- |
| **Nom d'utilisateur** |DomainAdmin |
| **Mot de passe** |Contoso!0000 |
| **Abonnement** |*votre abonnement* |
| **Groupe de ressources** |SQL-HA-RG |
| **Emplacement** |*votre emplacement* |
| **Taille** |D1_V2 (Standard) |
| **Type de stockage** |standard |
| **Compte de stockage** |*Créé automatiquement* |
| **Réseau virtuel** |autoHAVNET |
| **Sous-réseau** |Subnet-1 |
| **Adresse IP publique** |*Même nom que la machine virtuelle* |
| **Groupe de sécurité réseau** |*Même nom que la machine virtuelle* |
| **Diagnostics** |Activé |
| **Compte de stockage de diagnostics** |*Créé automatiquement* |
| **Groupe à haute disponibilité** |adavailabilitySet |

> [!NOTE]
> Vous ne pouvez pas modifier le groupe à haute disponibilité sur une machine virtuelle après sa création.
> 
> 

Azure crée les machines virtuelles.

Une fois les machines virtuelles créées, configurez le contrôleur de domaine.

### <a name="configure-the-domain-controller"></a>Configurer le contrôleur de domaine
Dans les étapes suivantes, vous allez configurer la machine **ad-primary-dc** comme contrôleur de domaine pour corp.contoso.com.

1. Dans le portail, ouvrez le groupe de ressources **SQL-HA-RG**, puis sélectionnez la machine **ad-primary-dc**. Dans le panneau **ad-primary-dc**, cliquez sur **Se connecter** pour ouvrir un fichier RDP pour l’accès au Bureau à distance.
   
    ![Se connecter à la machine virtuelle](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/20-connectrdp.png)
2. Connectez-vous avec votre compte administrateur configuré (**\DomainAdmin**) et votre mot de passe(**Contoso!0000**).
3. Par défaut, le tableau de bord **Gestionnaire de serveur** doit être affiché.
4. Cliquez sur le lien du tableau de bord **Ajouter des rôles et fonctionnalités** .
   
    ![Ajout de rôles Explorateur de serveurs](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC784623.png)
5. Sélectionnez **Suivant** jusqu’à ce que vous atteigniez la section **Rôles de serveur**.
6. Sélectionnez les rôles **Services de domaine Active Directory** et **Serveur DNS**. Lorsque vous y êtes invité, ajoutez les fonctionnalités supplémentaires requises par ces rôles.
   
   > [!NOTE]
   > Windows vous avertit qu’il n’y a aucune adresse IP statique. Si vous testez la configuration, cliquez sur Continuer. Pour les scénarios de production, définissez l’adresse IP comme statique dans le portail Azure ou [utilisez PowerShell pour définir l’adresse IP statique de la machine contrôleur de domaine](../virtual-network/virtual-networks-reserved-private-ip.md).
   > 
   > 
   
    ![Boîte de dialogue Ajouter des rôles](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC784624.png)
7. Cliquez sur **Suivant** jusqu’à ce que vous atteigniez la section **Confirmation**. Cochez la case **Redémarrer automatiquement le serveur de destination, si nécessaire** .
8. Cliquez sur **Installer**.
9. Une fois les fonctionnalités installées, retournez dans le tableau de bord **Gestionnaire de serveur** .
10. Sélectionnez la nouvelle option **AD DS** dans le volet gauche.
11. Cliquez sur le lien **Plus** dans la barre d'avertissement jaune.
    
     ![Boîte de dialogue AD DS sur une machine virtuelle du serveur DNS](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC784625.png)
12. Dans la colonne **Action** de la boîte de dialogue **Tous les détails de la tâche serveur**, cliquez sur **Promouvoir ce serveur en contrôleur de domaine**.
13. Dans l' **Assistant de configuration des services de domaine Active Directory**, utilisez les valeurs suivantes :

| **Page** | Paramètre |
| --- | --- |
| **Configuration du déploiement** |**Ajouter une nouvelle forêt** = sélectionné<br/>**Nom de domaine racine** = corp.contoso.com |
| **Options de contrôleur de domaine :** |**Mot de passe DSRM** = Contoso!0000<br/>**Confirmer le mot de passe** = Contoso!0000 |

1. Cliquez sur **Suivant** pour parcourir les autres pages de l'Assistant. Sur la page **Vérification de la configuration requise**, vérifiez que vous voyez le message suivant : **Toutes les vérifications de la configuration requise ont donné satisfaction**. Notez que vous devez examiner les messages d'avertissement applicables, mais il est possible de poursuivre l'installation.
2. Cliquez sur **Installer**. La machine virtuelle **ad-primary-dc** redémarre automatiquement.

### <a name="configure-the-second-domain-controller"></a>Configuration du second contrôleur de domaine
Une fois le contrôleur de domaine principal redémarré, vous pouvez configurer le second contrôleur de domaine. Cette étape facultative intervient pour les scénarios de haute disponibilité. Pour effectuer cette étape, vous devez connaître l’adresse IP privée du contrôleur de domaine. Vous pouvez l’obtenir à partir du Portail Azure. Suivez ces étapes pour configurer le second contrôleur de domaine.

1. Connectez-vous à la machine **ad-primary-dc** . 
2. Ouvrez le Bureau à distance et connectez-vous au contrôleur de domaine secondaire à l’aide de l’adresse IP. Si vous ne connaissez pas l’adresse IP du contrôleur de domaine secondaire, accédez au Portail Azure et vérifiez l’adresse affectée à l’interface réseau du contrôleur de domaine secondaire. 
3. Remplacez l’adresse du serveur DNS préféré par celle du contrôleur de domaine. 
4. Lancez le fichier RDP sur le contrôleur de domaine principal (**ad-primary-dc**) et connectez-vous à la machine virtuelle à l’aide de votre compte d’administrateur configuré (**BUILTIN\DomainAdmin**) et de votre mot de passe (**Contoso!0000**).
5. À partir du contrôleur de domaine principal, lancez une session Bureau à distance sur **ad-secondary-dc** à l’aide de l’adresse IP. Utilisez le même compte et le même mot de passe.
6. Une fois connecté, vous devez voir le tableau de bord **Gestionnaire de serveur** . Dans le volet gauche, cliquez sur **Serveur local** .
7. Sélectionnez le lien **Adresse IPv4 attribuée par DHCP, Compatible IPv6** .
8. Dans la fenêtre **Connexions réseau** , sélectionnez l’icône du réseau.
   
    ![Modification du serveur DNS préféré de la machine virtuelle](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC784629.png)
9. Dans la barre de commandes, cliquez sur **Modifier les paramètres de cette connexion** (selon la taille de la fenêtre, vous devrez peut-être cliquer sur la double flèche droite pour afficher cette commande).
10. Cliquez sur **Internet Protocol Version 4 (TCP/IPv4)** , puis sur Propriétés.
11. Sélectionnez Utiliser l’adresse de serveur DNS suivante et spécifiez l’adresse du contrôleur de domaine principal sous **Serveur DNS préféré**.
12. L’adresse est celle affectée à une machine virtuelle dans le sous-réseau subnet-1 du réseau virtuel Azure, et cette machine virtuelle est **ad-primary-dc**. Pour vérifier l’adresse IP **ad-primary-dc**, exécutez **nslookup ad-primary-dc** dans l’invite de commandes, comme indiqué ci-dessous.
    
     ![Utilisation de NSLOOKUP pour trouver l'adresse IP du contrôleur de domaine](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC664954.png)
    
    > [!NOTE]
    > Après avoir défini le DNS, vous risquez de perdre la session RDP avec le serveur membre. Dans ce cas, redémarrez la machine virtuelle à partir du Portail Azure.
    > 
    > 
13. Cliquez sur **OK**, puis sur **Fermer** pour valider les modifications. Vous pouvez maintenant joindre la machine virtuelle à **corp.contoso.com**.
14. Répétez les étapes que vous avez suivies pour créer le premier contrôleur de domaine, mais utilisez les valeurs suivantes dans l’ **Assistant Configuration des services de domaine Active Directory**:

| Page | Paramètre |
| --- | --- |
| **Configuration du déploiement** |**Ajouter un contrôleur de domaine à un domaine existant** = sélectionné<br/>**Racine** = corp.contoso.com |
| **Options de contrôleur de domaine :** |**Mot de passe DSRM** = Contoso!0000<br/>**Confirmer le mot de passe** = Contoso!0000 |

### <a name="configure-domain-accounts"></a>Configuration des comptes de domaine
Les étapes suivantes configurent les comptes Active Directory (AD) pour une utilisation ultérieure.

1. Reconnectez-vous à la machine **ad-primary-dc** .
2. Dans **Gestionnaire de serveur**, sélectionnez **Outils**, puis cliquez sur **Centre d’administration Active Directory**.
   
    ![Centre d'administration Active Directory](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC784626.png)
3. Dans le panneau **Centre d'administration Active Directory** select **corp (local)** dans le volet gauche.
4. Dans le volet droit **Tâches**, sélectionnez **Nouveau**, puis cliquez sur **Utilisateur**. Utilisez les paramètres suivants :

| Paramètre | Valeur |
| --- | --- |
| **Prénom** |Installer |
| **SamAccountName de l'utilisateur** |Installer |
| **Mot de passe** |Contoso!0000 |
| **Confirmer le mot de passe** |Contoso!0000 |
| **Autres options de mot de passe** |Sélectionné |
| **Le mot de passe n'expire jamais** |Activé |

1. Cliquez sur **OK** pour créer l’utilisateur **d’Installation**. Utilisez ce compte pour configurer le cluster de basculement et le groupe de disponibilité.
2. Créez deux utilisateurs supplémentaires en suivant les mêmes étapes : **CORP\SQLSvc1** et **CORP\SQLSvc2**. Les services SQL Server utilisent ces comptes. Ensuite, donnez à **CORP\Install** les autorisations nécessaires pour configurer le clustering de basculement Windows Server (WSFC, Windows Server Failover Clustering).
3. Dans le **Centre d’administration Active Directory**, sélectionnez **corp (local)** dans le volet gauche. Ensuite, dans le volet droit **Tâches**, cliquez sur **Propriétés**.
   
    ![Propriétés de l'utilisateur CORP](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC784627.png)
4. Sélectionnez **Extensions**, puis cliquez sur le bouton **Avancé** de l’onglet **Sécurité**.
5. Dans la fenêtre **Paramètres de sécurité avancés pour corp** . Cliquez sur **Ajouter**.
6. Cliquez sur **Sélectionner un principal**. Recherchez **CORP\Install**. Cliquez sur **OK**.
7. Sélectionnez les autorisations **Lire toutes les propriétés** et **Créer des objets ordinateur**.
   
    ![Autorisations de l’utilisateur Corp](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC784628.png)
8. Cliquez sur **OK**, puis sur **OK** à nouveau. Fermez la fenêtre des propriétés corp.

Maintenant que vous avez fini de configurer Active Directory et les objets utilisateur, créez les deux machines virtuelles SQL Server et une machine virtuelle de serveur témoin. Joignez les trois machines virtuelles au domaine.

## <a name="create-sql-servers"></a>Création de serveurs SQL
### <a name="create-and-configure-the-sql-server-vms"></a>Créer et configurer les machines virtuelles SQL Server
Créez ensuite trois machines virtuelles, dont deux machines virtuelles SQL Server et un nœud de cluster WSFC. Pour créer chaque machine virtuelle, revenez au groupe de ressources **SQL-HA-RG**, cliquez sur **Ajouter**, recherchez l’élément de galerie approprié, **Machine virtuelle**, puis **À partir de la galerie**. Utilisez ensuite les modèles dans le tableau suivant pour vous aider à créer les machines virtuelles.

| Page | MV1 | MV2 | MV3 |
| --- | --- | --- | --- |
| Sélectionnez l’élément de la galerie approprié. |**Windows Server 2012 R2 Datacenter** |**SQL Server 2014 SP1 Enterprise sur Windows Server 2012 R2** |**SQL Server 2014 SP1 Enterprise sur Windows Server 2012 R2** |
| **Notions** |**Nom** = cluster-fsw<br/>**Nom d’utilisateur** = DomainAdmin<br/>**Mot de passe** = Contoso!0000<br/>**Abonnement** = votre abonnement<br/>**Groupe de ressources** = SQL-HA-RG<br/>**Emplacement** = Votre emplacement Azure |**Nom** = sqlserver-0<br/>**Nom d’utilisateur** = DomainAdmin<br/>**Mot de passe** = Contoso!0000<br/>**Abonnement** = votre abonnement<br/>**Groupe de ressources** = SQL-HA-RG<br/>**Emplacement** = Votre emplacement Azure |**Nom** = sqlserver-1<br/>**Nom d’utilisateur** = DomainAdmin<br/>**Mot de passe** = Contoso!0000<br/>**Abonnement** = votre abonnement<br/>**Groupe de ressources** = SQL-HA-RG<br/>**Emplacement** = Votre emplacement Azure |
| Configuration de la machine virtuelle - **Taille** |DS1 (1 cœur, 3,5 Go de mémoire) |**TAILLE** = DS 2 (2 cœurs, 7 Go de mémoire) |**TAILLE** = DS 2 (2 cœurs, 7 Go de mémoire) |
| Configuration de la machine virtuelle - **Paramètres** |**Stockage** = Premium (SSD)<br/>**SOUS-RÉSEAUX DU RÉSEAU** = autoHAVNET<br/>**COMPTE DE STOCKAGE** : utilisez un compte de stockage généré automatiquement<br/>**Sous-réseau** = subnet-2(10.1.1.0/24)<br/>**Adresse IP publique** = aucune<br/>**Groupe de sécurité réseau** = aucun<br/>**Diagnostics de surveillance** = activés<br/>**Compte de stockage de diagnostics** = utilisez un compte de stockage généré automatiquement<br/>**GROUPE À HAUTE DISPONIBILITÉ** = sqlAvailabilitySet<br/> |**Stockage** = Premium (SSD)<br/>**SOUS-RÉSEAUX DU RÉSEAU** = autoHAVNET<br/>**COMPTE DE STOCKAGE** : utilisez un compte de stockage généré automatiquement<br/>**Sous-réseau** = subnet-2(10.1.1.0/24)<br/>**Adresse IP publique** = aucune<br/>**Groupe de sécurité réseau** = aucun<br/>**Diagnostics de surveillance** = activés<br/>**Compte de stockage de diagnostics** = utilisez un compte de stockage généré automatiquement<br/>**GROUPE À HAUTE DISPONIBILITÉ** = sqlAvailabilitySet<br/> |**Stockage** = Premium (SSD)<br/>**SOUS-RÉSEAUX DU RÉSEAU** = autoHAVNET<br/>**COMPTE DE STOCKAGE** : utilisez un compte de stockage généré automatiquement<br/>**Sous-réseau** = subnet-2(10.1.1.0/24)<br/>**Adresse IP publique** = aucune<br/>**Groupe de sécurité réseau** = aucun<br/>**Diagnostics de surveillance** = activés<br/>**Compte de stockage de diagnostics** = utilisez un compte de stockage généré automatiquement<br/>**GROUPE À HAUTE DISPONIBILITÉ** = sqlAvailabilitySet<br/> |
| Configuration de la machine virtuelle - **Paramètres SQL Server** |Non applicable |**Connectivité SQL** = privée (dans le réseau virtuel)<br/>**Port** = 1433<br/>**Authentification SQL** = désactivée<br/>**Configuration du stockage** = général<br/>**Mise à jour corrective automatisée** : dimanche à 2h00<br/>**Sauvegarde automatisée** = désactivée</br>**Intégration Azure Key Vault** = Désactivée |**Connectivité SQL** = privée (dans le réseau virtuel)<br/>**Port** = 1433<br/>**Authentification SQL** = désactivée<br/>**Configuration du stockage** = général<br/>**Mise à jour corrective automatisée** : dimanche à 2h00<br/>**Sauvegarde automatisée** = désactivée</br>**Intégration Azure Key Vault** = Désactivée |

<br/>

> [!NOTE]
> La configuration précédente suggère des machines virtuelles de niveau STANDARD, car les machines de niveau DE BASE ne prennent pas en charge les points de terminaison à équilibrage de charge requis par l’écouteur de groupe de disponibilité. De plus, les tailles de machines suggérées ici sont destinées au test des groupes de disponibilité dans les machines virtuelles Azure. Pour optimiser les performances des charges de travail de production, consultez les recommandations relatives à la taille des ordinateurs SQL Server et à la configuration dans [Meilleures pratiques relatives aux performances de SQL Server dans Azure Virtual Machines](virtual-machines-windows-sql-performance.md).
> 
> 

Une fois les trois machines virtuelles entièrement configurées, vous devez les attacher au domaine **corp.contoso.com** et accorder à CORP\Install des droits d’administration sur les machines.

Pour simplifier la procédure, notez l’adresse IP virtuelle Azure pour chaque machine virtuelle. Obtenez l’adresse IP pour chaque serveur. Dans le groupe de ressources Azure SQL-HA-RG, cliquez sur la ressource **autohaVNET** . Le panneau **autohaVNET** affiche les adresses IP de chaque machine de votre réseau.

Consignez les adresses IP des appareils suivants : 

| Rôle de la machine virtuelle | Appareil | Adresse IP |
| --- | --- | --- |
| Contrôleur de domaine principal |ad-primary-dc | |
| Contrôleur de domaine secondaire |ad-secondary-dc | |
| Témoin de partage de fichiers de cluster |cluster-fsw | |
| SQL Server |sqlserver-0 | |
| SQL Server |sqlserver-1 | |

Utilisez ces adresses pour configurer le service DNS pour chaque machine virtuelle. Procédez comme suit pour chacune des trois machines virtuelles.

1. Tout d’abord, modifiez l’adresse du serveur DNS préféré pour chaque serveur membre. 
2. Lancez le fichier RDP sur le contrôleur de domaine principal (**ad-primary-dc**) et connectez-vous à la machine virtuelle à l’aide de votre compte d’administrateur configuré (**BUILTIN\DomainAdmin**) et de votre mot de passe (**Contoso!0000**).
3. À partir du contrôleur de domaine principal, lancez une session Bureau à distance sur **sqlserver-0** avec l’adresse IP. Utilisez le même compte et le même mot de passe.
4. Une fois connecté, vous devez voir le tableau de bord **Gestionnaire de serveur** . Dans le volet gauche, cliquez sur **Serveur local** .
5. Sélectionnez le lien **Adresse IPv4 attribuée par DHCP, Compatible IPv6** .
6. Dans la fenêtre **Connexions réseau** , sélectionnez l’icône du réseau.
   
    ![Modification du serveur DNS préféré de la machine virtuelle](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC784629.png)
7. Dans la barre de commandes, cliquez sur **Modifier les paramètres de cette connexion** (selon la taille de la fenêtre, vous devrez peut-être cliquer sur la double flèche droite pour afficher cette commande).
8. Cliquez sur **Internet Protocol Version 4 (TCP/IPv4)** , puis sur Propriétés.
9. Sélectionnez Utiliser l’adresse de serveur DNS suivante et spécifiez l’adresse du contrôleur de domaine principal sous **Serveur DNS préféré**.
10. L’adresse est celle affectée à une machine virtuelle dans le sous-réseau subnet-1 du réseau virtuel Azure, et cette machine virtuelle est **ad-primary-dc**. Pour vérifier l’adresse IP **ad-primary-dc**, exécutez **nslookup ad-primary-dc** dans l’invite de commandes, comme indiqué ci-dessous.
    
     ![Utilisation de NSLOOKUP pour trouver l'adresse IP du contrôleur de domaine](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC664954.png)
    
    > [!NOTE]
    > Après avoir défini le DNS, vous risquez de perdre la session RDP avec le serveur membre. Dans ce cas, redémarrez la machine virtuelle à partir du Portail Azure.
    > 
    > 
11. Cliquez sur **OK**, puis sur **Fermer** pour valider les modifications. Vous pouvez maintenant joindre la machine virtuelle à **corp.contoso.com**.
12. De retour dans la fenêtre **Serveur local**, cliquez sur le lien **WORKGROUP**.
13. Dans la section **Nom de l’ordinateur**, cliquez sur **Modifier**.
14. Cochez la case **Domaine** et saisissez **corp.contoso.com** dans la zone de texte. Cliquez sur **OK**.
15. Dans la boîte de dialogue contextuelle **Sécurité Windows**, spécifiez les informations d’identification du compte d’administrateur de domaine par défaut (**CORP\DomainAdmin**) et le mot de passe (**Contoso!0000**).
16. Lorsque le message « Bienvenue dans le domaine corp.contoso.com » s’affiche, cliquez sur **OK**.
17. Cliquez sur **Fermer**, puis sur **Redémarrer maintenant** dans la boîte de dialogue contextuelle.
18. Répétez ces étapes pour le serveur témoin de partage de fichiers et chaque serveur SQL Server. 

### <a name="add-the-corp\install-user-as-an-administrator-on-each-cluster-vm:"></a>Ajoutez l’utilisateur Corp\Install en tant qu’administrateur sur chaque machine virtuelle du cluster :
1. Patientez jusqu’à ce que la machine virtuelle ait redémarré, puis lancez à nouveau le fichier RDP à partir du contrôleur de domaine principal pour vous connecter à **sqlserver-0** à l’aide du compte **BUILTIN\DomainAdmin**.
2. Dans **Gestionnaire de serveur**, sélectionnez **Outils**, puis cliquez sur **Gestion de l’ordinateur**.
   
    ![Gestion de l'ordinateur](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC784630.png)
3. Dans la fenêtre **Gestion de l’ordinateur**, développez **Utilisateurs et groupes locaux**, puis sélectionnez **Groupes**.
4. Double-cliquez sur le groupe **Administrateurs** .
5. Dans la boîte de dialogue **Propriétés de Administrateurs**, cliquez sur le bouton **Ajouter**.
6. Entrez l’utilisateur **CORP\Install**, puis cliquez sur **OK**. Lorsque vous êtes invité à spécifier vos informations d’identification, utilisez le compte **DomainAdmin** avec le mot de passe **Contoso!0000**.
7. Cliquez sur **OK** pour fermer la boîte de dialogue **Propriétés de Administrateurs**.
8. Répétez les étapes ci-dessus avec **sqlserver-1** et **cluster-fsw**.

## <a name="create-the-cluster"></a>Création du cluster
### <a name="add-the-**failover-clustering**-feature-to-each-cluster-vm."></a>Ajoutez la fonctionnalité de **Clustering de basculement** à chaque machine virtuelle du cluster.
1. Établissez une connexion RDP avec **sqlserver-0**.
2. Dans le tableau de bord **Gestionnaire de serveur**, cliquez sur **Ajouter des rôles et fonctionnalités**.
3. Dans **l’Assistant d’ajout de rôles et fonctionnalités**, cliquez sur **Suivant** jusqu’à ce que vous atteigniez la page **Fonctionnalités**.
4. Sélectionnez **Clustering de basculement**. Lorsque vous y êtes invité, ajoutez toutes les autres fonctionnalités.
   
    ![Ajout de la fonctionnalité de Clustering de basculement à la machine virtuelle](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC784631.png)
5. Cliquez sur **Suivant**, puis sur **Installer** dans la page **Confirmation**.
6. Lorsque l’installation de la fonctionnalité **Clustering de basculement** est terminée, cliquez sur **Fermer**.
7. Déconnectez-vous de la machine virtuelle.
8. Répétez les étapes de cette section avec **sqlserver-1** et **cluster-fsw**.

Les machines virtuelles SQL Server sont maintenant configurés et exécutées, mais elles sont installées avec les options par défaut de SQL Server.

### <a name="create-the-wsfc-cluster"></a>Création du cluster WSFC
Dans cette section, vous allez créer le cluster WSFC qui héberge le groupe de disponibilité. À ce stade, vous devriez avoir effectué les opérations suivantes pour chacune des trois machines virtuelles dans le cluster WSFC :

* configuration complète dans Azure
* machine virtuelle jointe au domaine
* Ajout de **CORP\Install** au groupe Administrateurs local
* Ajout de la fonctionnalité de Clustering de basculement à chaque machine virtuelle

Il s'agit des composants requis sur chaque machine virtuelle avant sa liaison au cluster WSFC.

Notez également que le réseau virtuel Azure ne se comporte pas de la même manière qu'un réseau local. Vous devez créer le cluster dans l'ordre suivant :

1. Créez un cluster à nœud unique sur l’un des nœuds (**sqlserver-0**).
2. Remplacez l’adresse IP du cluster par une adresse IP inutilisée dans **sqlsubnet**.
3. Mise en ligne du nom de cluster.
4. Ajoutez les autres nœuds (**sqlserver-1** et **cluster-fsw**).

Suivez les étapes ci-dessous pour configurer le cluster.

1. Lancez le fichier RDP pour **sqlserver-0** et connectez-vous en utilisant le compte de domaine **CORP\Install**.
2. Dans le tableau de bord **Gestionnaire de serveur**, sélectionnez **Outils**, puis cliquez sur **Gestionnaire du cluster de basculement**.
3. Dans le volet gauche, cliquez sur **Gestionnaire du cluster de basculement**, puis sur **Créer un cluster**, comme illustré ci-dessous.
   
    ![Créer un cluster](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC784632.png)
4. Dans l'Assistant de création de cluster, créez un cluster à un nœud en complétant les pages avec les paramètres ci-dessous :

| Page | Paramètres |
| --- | --- |
| Avant de commencer |Utilisation des valeurs par défaut |
| Sélection des serveurs |Saisissez **sqlserver-0** dans **Entrer un nom de serveur**, puis cliquez sur **Ajouter**. |
| Avertissement de validation |Cliquez sur **Non. Je n’ai pas besoin de l’assistance de Microsoft pour ce cluster, et par conséquent, je ne souhaite pas exécuter les tests de validation. Lorsque je clique sur Suivant, poursuivre la création du cluster**. |
| Point d'accès pour l'administration du cluster |Saisissez **Cluster1** dans **Nom du cluster**. |
| Confirmation |Utilisez les valeurs par défaut, sauf si vous utilisez des espaces de stockage. Consultez la remarque qui suit ce tableau. |

> [!NOTE]
> Si vous utilisez des [espaces de stockage](https://technet.microsoft.com/library/hh831739), qui regroupent plusieurs disques dans des pools de stockage, vous devez décocher la case **Ajouter la totalité du stockage disponible au cluster** sur la page **Confirmation**. Si vous ne décochez pas cette option, Windows détache les disques virtuels pendant le processus de clustering. Par conséquent, ils n’apparaissent pas dans le Gestionnaire de disque ou dans l’explorateur, jusqu’à ce que les espaces de stockage soient supprimés du cluster et rattachés à l’aide de PowerShell.
> 
> 

Maintenant que vous avez créé le cluster, vérifiez la configuration et ajoutez les nœuds restants. 

1. Dans le volet central, faites défiler jusqu’à la section **Ressources principales du cluster** et développez les détails de **Nom : Cluster1**. Les ressources **Nom** et **Adresse IP** doivent toutes deux être à l’état **Échec**. La ressource d’adresse IP ne peut pas être mise en ligne, car le cluster a la même adresse IP que la machine elle-même, qui est une adresse dupliquée.
2. Cliquez avec le bouton droit sur la ressource **Adresse IP** en échec, puis cliquez sur **Propriétés**.
   
    ![Propriétés du cluster](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC784633.png)
3. Sélectionnez **Adresse IP statique** et spécifiez une adresse de subnet-2 disponible dans la zone de texte Adresse. Cliquez ensuite sur **OK**.
4. Dans la section **Principales ressources de cluster**, cliquez avec le bouton droit sur **Nom : Cluster1**, puis cliquez sur **Mettre en ligne**. Attendez que les deux ressources soient en ligne. Lorsque la ressource du cluster apparaît en ligne, elle met à jour le serveur de contrôleur de domaine avec un nouveau compte d’ordinateur Active Directory (AD). Utilisez ce compte AD pour exécuter le service en cluster du groupe de disponibilité ultérieurement.
5. Enfin, ajoutez les nœuds restants au cluster. Dans l’arborescence du navigateur, cliquez avec le bouton droit sur **Cluster1.corp.contoso.com**, puis cliquez sur **Ajouter un nœud**, comme illustré ci-dessous.
   
    ![Ajout d’un nœud au cluster](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC784634.png)
6. Dans **l’Assistant d’ajout de nœud**, cliquez sur **Suivant**. Dans la page **Sélectionner des serveurs**, ajoutez **sqlserver-1** et **cluster-fsw** à la liste en saisissant le nom du serveur dans **Entrer un nom de serveur**, puis en cliquant sur **Ajouter**. Une fois ces opérations effectuées, cliquez sur **Suivant**.
7. Sur la page **Avertissement de validation**, cliquez sur **Non** (dans un scénario de production, vous devez effectuer les essais de validation). Cliquez ensuite sur **Suivant**.
8. Dans la page **Confirmation**, cliquez sur **Suivant** pour ajouter les nœuds.
   
   > [!WARNING]
   > Si vous utilisez des [espaces de stockage](https://technet.microsoft.com/library/hh831739), qui regroupent plusieurs disques dans des pools de stockage, vous devez décocher la case **Ajouter la totalité du stockage disponible au cluster** . Si vous ne décochez pas cette option, les disques virtuels sont détachés pendant le processus de clustering. Par conséquent, ils n’apparaissent pas non plus dans le Gestionnaire de disque ou dans l'explorateur, jusqu'à ce que les espaces de stockage soient supprimés du cluster et rattachés à l'aide de PowerShell.
   > 
   > 
9. Une fois les nœuds ajoutés au cluster, cliquez sur **Terminer**. Le Gestionnaire du cluster de basculement doit maintenant indiquer que votre cluster a trois nœuds, répertoriés dans le conteneur **Nœuds** .
10. Déconnectez-vous de la session Bureau à distance.

## <a name="configure-availability-groups"></a>Configuration des groupes de disponibilité
Dans cette section, vous effectuerez les opérations suivantes sur **sqlserver-0** et **sqlserver-1** :

* Ajouter **CORP\Install** en tant que rôle sysadmin à l’instance SQL Server par défaut
* Ouvrez le pare-feu pour l’accès à distance au serveur SQL Server pour le processus SQL Server, le point de terminaison de mise en miroir de base de données et le port de sonde de l’équilibreur de charge Azure. 
* Activation de la fonctionnalité de groupes de disponibilité
* Modifier le compte de service SQL Server pour **CORP\SQLSvc1** et **CORP\SQLSvc2**, respectivement

Ces actions peuvent être effectuées dans n'importe quel ordre. Néanmoins, les étapes ci-dessous les traitent dans l'ordre. Suivez les étapes pour **sqlserver-0** et **sqlserver-1** :

### <a name="add-installation-account-as-sysadmin-fixed-server-role-on-each-sql-server"></a>Ajouter un compte d’installation en tant que rôle serveur fixe sysadmin sur chaque serveur SQL Server
1. Si vous ne vous êtes pas déconnecté de la session Bureau à distance pour la machine virtuelle, faites-le maintenant.
2. Lancez les fichiers RDP pour **sqlserver-0** et **sqlserver-1** et connectez-vous en tant que **BUILTIN\DomainAdmin**.
3. Lancez **SQL Server Management Studio** et ajoutez **CORP\Install** comme rôle **sysadmin** dans l’instance SQL Server par défaut. Dans **l’Explorateur d’objets**, cliquez avec le bouton droit sur **Connexions**, puis cliquez sur **Nouvelle connexion**.
4. Saisissez **CORP\Install** dans **Nom de connexion**.
5. Dans la page **Rôles de serveur**, cliquez sur **sysadmin**. Cliquez ensuite sur **OK**. Une fois la connexion créée, vous pouvez l'afficher en développant **Connexions** in **Explorateur d'objets**.

### <a name="open-the-firewall-for-remote-access-to-sql-server-and-the-probe-port-on-each-sql-server"></a>Ouvrir le pare-feu pour l’accès à distance au serveur SQL Server et au port de sonde sur chaque serveur SQL Server
Cette solution requiert deux règles de pare-feu sur chaque serveur SQL Server. La première règle fournit un accès entrant au serveur SQL Server. La seconde fournit un accès entrant à l’équilibreur de charge et à l’écouteur. 

1. Puis, créez une règle de pare-feu pour un serveur SQL. Lancez le **Pare-feu Windows avec fonctions de sécurité avancées** à partir de l’écran **d’accueil**.
2. Dans le volet gauche, sélectionnez **Règles de trafic entrant**. Dans le volet droit, cliquez sur **Nouvelle règle**.
3. Dans la page **Type de règle**, sélectionnez **Programme**, puis cliquez sur **Suivant**.
4. Dans la page **Programme**, sélectionnez **Ce chemin d’accès au programme** et saisissez **%ProgramFiles%\Microsoft SQL Server\MSSQL12.MSSQLSERVER\MSSQL\Binn\sqlservr.exe** dans la zone de texte (si vous suivez ces instructions avec SQL Server 2012, le répertoire de SQL Server est **MSSQL11.MSSQLSERVER**). Cliquez ensuite sur **Suivant**.
5. Dans la page **Action**, sélectionnez **Autoriser la connexion** et cliquez sur **Suivant**.
6. Dans la page **Profil**, acceptez les paramètres par défaut et cliquez sur **Suivant**.
7. Dans la page **Nom**, spécifiez un nom pour la règle, par exemple **SQL Server (règle de programme)** dans la zone de texte **Nom**, puis cliquez sur **Terminer**.
8. Créez une autre règle de pare-feu de trafic entrant pour le port de sonde. Cette règle est une règle de trafic entrant vers le port TCP 59999, utilisée dans le cadre de ce didacticiel. Nommez la règle **Écouteur SQL Server**.

Effectuez toutes les étapes pour les deux serveurs SQL Server.

### <a name="enable-availability-groups-feature-on-each-sql-server"></a>Activation de la fonctionnalité de groupes de disponibilité sur chaque serveur SQL Server
Effectuez ces étapes pour les deux serveurs SQL Server. 

1. Ensuite, activez la fonctionnalité **Groupes de disponibilité AlwaysOn** . À partir de l’écran **d’accueil**, lancez le **Gestionnaire de configuration SQL Server**.
2. Dans l’arborescence du navigateur, cliquez sur **Services SQL Server**, cliquez avec le bouton droit sur **SQL Server (MSSQLSERVER)**, puis cliquez sur **Propriétés**.
3. Cliquez sur l’onglet **Haute disponibilité AlwaysOn**, sélectionnez **Activer les groupes de disponibilité AlwaysOn**, comme illustré ci-dessous, puis cliquez sur **Appliquer**. Cliquez sur **OK** dans la boîte de dialogue contextuelle mais ne fermez pas encore la fenêtre Propriétés. Redémarrez le service SQL Server après avoir modifié le compte de service.
   
    ![Activation des groupes à haute disponibilité AlwaysOn](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC665520.gif)

### <a name="set-the-sql-server-service-account-on-each-sql-server"></a>Définir le compte de service SQL Server sur chaque serveur SQL Server
Effectuez ces étapes pour les deux serveurs SQL Server.

1. Ensuite, modifiez le compte de service SQL Server. Cliquez sur l’onglet **Ouverture de session**, saisissez **CORP\SQLSvc1** (pour **sqlserver-0**) ou **CORP\SQLSvc2** (pour **sqlserver-1**) dans **Nom de compte**, saisissez et confirmez le mot de passe, puis cliquez sur **OK**.
2. Dans la fenêtre contextuelle, cliquez sur **Oui** pour redémarrer le service SQL Server. Une fois que le service SQL Server est redémarré, les modifications apportées dans la fenêtre Propriétés sont actives.
3. Déconnectez-vous des machines virtuelles.

### <a name="create-the-availability-group"></a>Création du groupe de disponibilité
Vous pouvez maintenant configurer le groupe de disponibilité. Voici une présentation des opérations à effectuer :

* Création d’une base de données (**MyDB1**) sur **sqlserver-0**
* Sauvegarde complète et sauvegarde du journal des transactions de la base de données
* Restauration de la sauvegarde complète et de la sauvegarde du journal vers **sqlserver-1** avec l’option **NORECOVERY**
* Création du groupe de disponibilité (**AG1**) avec validation synchrone, basculement automatique et réplicas secondaires lisibles

### <a name="create-the-mydb1-database-on-sqlserver-0:"></a>Créer la base de données MyDB1 sur sqlserver-0 :
1. Si vous ne vous êtes pas encore déconnecté des sessions Bureau à distance pour **sqlserver-0** et **sqlserver-1**, faites-le maintenant.
2. Lancez le fichier RDP pour **sqlserver-0** et connectez-vous en tant que **CORP\Install**.
3. Dans **l’Explorateur de fichiers**, sous **C:\**, créez un répertoire appelé **sauvegarde**. Vous utiliserez ce répertoire pour sauvegarder et restaurer votre base de données.
4. Cliquez avec le bouton droit sur le nouveau répertoire, pointez sur **Partager avec**, puis cliquez sur **Des personnes spécifiques**, comme illustré ci-dessous.
   
    ![Créer un dossier de sauvegarde](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC665521.gif)
5. Ajoutez **CORP\SQLSvc1** et donnez-lui l’autorisation **Lecture/écriture**, ajoutez **CORP\SQLSvc2** et donnez-lui l’autorisation **Lecture/écriture**, comme illustré ci-dessous, puis cliquez sur **Partager**. Une fois le processus de partage de fichiers terminé, cliquez sur **Terminer**.
   
    ![Accorder des autorisations pour le dossier de sauvegarde](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC665522.gif)
6. Créez ensuite la base de données. À partir du menu **d’accueil**, lancez **SQL Server Management Studio**, puis cliquez sur **Connexion** pour vous connecter à l’instance SQL Server par défaut.
7. Dans **l’Explorateur d’objets**, cliquez avec le bouton droit sur **Bases de données**, puis cliquez sur **Nouvelle base de données**.
8. Dans **Nom de base de données**, saisissez **MyDB1**, puis cliquez sur **OK**.

### <a name="take-a-full-backup-of-mydb1-and-restore-it-on-sqlserver-1:"></a>Effectuer une sauvegarde complète de MyDB1 et la restaurer sur sqlserver-1 :
1. Puis, effectuez une sauvegarde complète de la base de données. Dans **l’Explorateur d’objets**, développez **Bases de données**, cliquez avec le bouton droit sur **MyDB1**, pointez sur **Tâches**, puis cliquez sur **Sauvegarder**.
2. Dans la section **Source**, laissez la valeur de **Type de sauvegarde** sur **Complet**. Dans la section **Destination**, cliquez sur **Supprimer** afin de supprimer le chemin d’accès de fichier par défaut correspondant au fichier de sauvegarde.
3. Dans la section **Destination**, cliquez sur **Ajouter**.
4. Dans la zone de texte **Nom de fichier**, saisissez **\\\\sqlserver-0\backup\MyDB1.bak**. Cliquez ensuite sur **OK**, puis à nouveau sur **OK** pour sauvegarder la base de données. Une fois l’opération de sauvegarde terminée, cliquez à nouveau sur **OK** pour fermer la boîte de dialogue.
5. Puis, effectuez une sauvegarde du journal des transactions de la base de données. Dans **l’Explorateur d’objets**, développez **Bases de données**, cliquez avec le bouton droit sur **MyDB1**, pointez sur **Tâches**, puis cliquez sur **Sauvegarder**.
6. Dans **Type de sauvegarde**, sélectionnez **Journal des transactions**. Conservez le chemin d’accès des fichiers **Destination** que vous avez spécifié précédemment et cliquez sur **OK**. Une fois l’opération de sauvegarde terminée, cliquez à nouveau sur **OK** .
7. Ensuite, restaurez la sauvegarde complète et la sauvegarde du journal des transactions sur **sqlserver-1**. Lancez le fichier RDP pour **sqlserver-1** et connectez-vous en tant que **CORP\Install**. Laissez la session Bureau à distance de **sqlserver-0** ouverte.
8. À partir du menu **d’accueil**, lancez **SQL Server Management Studio**, puis cliquez sur **Connexion** pour vous connecter à l’instance SQL Server par défaut.
9. Dans **l’Explorateur d’objets**, cliquez avec le bouton droit sur **Bases de données**, puis cliquez sur **Restaurer la base de données**.
10. Dans la section **Source**, sélectionnez **Unité**, puis cliquez sur le bouton **…** .
11. Dans **Sélectionner les unités de sauvegarde**, cliquez sur **Ajouter**.
12. Dans l’emplacement du fichier de sauvegarde, saisissez **\\\\sqlserver-0\backup**, cliquez sur Actualiser, sélectionnez MyDB1.bak, puis cliquez sur OK et de nouveau sur OK. La sauvegarde complète et la sauvegarde des journaux doivent désormais apparaître dans le volet Jeux de sauvegarde à restaurer.
13. Accédez à la page Options, puis sélectionnez RESTORE WITH NORECOVERY dans l'état de récupération, puis cliquez sur OK pour restaurer la base de données. Lorsque l'opération de restauration est terminée, cliquez à nouveau sur OK.

### <a name="create-the-availability-group:"></a>Création du groupe de disponibilité :
1. Retournez à la session Bureau à distance de **sqlserver-0**. Dans **l’Explorateur d’objets** dans SSMS, cliquez sur **Haute disponibilité AlwaysOn**, puis sur **Assistant Nouveau groupe de disponibilité**, comme illustré ci-dessous.
   
    ![Lancer l'Assistant Nouveau groupe de disponibilité](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC665523.gif)
2. Sur la page **Introduction**, cliquez sur **Suivant**. Dans la page **Spécifier le nom du groupe de disponibilité**, saisissez **AG1** dans **Nom du groupe de disponibilité**, puis cliquez à nouveau sur **Suivant**.
   
    ![Assistant Nouveau groupe de disponibilité, spécifier le nom du groupe de disponibilité](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC665524.gif)
3. Dans la page **Sélectionner des bases de données**, sélectionnez **MyDB1**, puis cliquez sur **Suivant**. Cette base de données répond à la configuration requise pour un groupe de disponibilité, car vous avez effectué au moins une sauvegarde complète sur le réplica principal prévu.
   
    ![Assistant Nouveau groupe de disponibilité, sélectionner les bases de données](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC665525.gif)
4. Sur la page **Spécifier les réplicas**, cliquez sur **Ajouter un réplica**.
   
    ![Assistant Nouveau groupe de disponibilité, spécifier les réplicas](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC665526.png)
5. La boîte de dialogue **Se connecter au serveur** apparaît. Saisissez **sqlserver-1** dans **Nom du serveur**, puis cliquez sur **Connexion**.
   
    ![Assistant Nouveau groupe de disponibilité, se connecter au serveur](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC665527.png)
6. De retour sur la page **Spécifier les réplicas**, vous devez maintenant voir **sqlserver-1** répertorié dans les **Réplicas de disponibilité**. Configurez les réplicas comme indiqué ci-dessous. 
   
    ![Assistant Nouveau groupe de disponibilité, spécifier les réplicas (Terminé)](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC665528.png)
7. Cliquez sur **Points de terminaison** pour afficher le point de terminaison de mise en miroir de la base de données qui sera utilisé par ce groupe de disponibilité. Chaque instance SQL Server doit posséder un point de terminaison de mise en miroir de la base de données. Notez le port TCP que cet Assistant spécifie pour ce point de terminaison. Créez une règle de pare-feu entrante sur chaque serveur pour ce port TCP.
   
    Quand vous avez terminé, cliquez sur **Suivant**.
8. Sur la page **Sélectionner la synchronisation de données initiale**, cliquez sur **Joindre uniquement**, puis sur **Suivant**. Vous avez déjà effectué la synchronisation des données manuellement lorsque vous avez effectué les sauvegardes complète et de transaction sur **sqlserver-0** avant de les restaurer sur **sqlserver-1**. Vous pouvez donc choisir de ne pas effectuer les opérations de sauvegarde et de restauration sur votre base de données et sélectionner **Complète** pour laisser l’Assistant Nouveau groupe de disponibilité effectuer la synchronisation des données pour vous. Toutefois, cela n’est pas recommandé pour les grandes bases de données de certaines entreprises.
   
    ![Assistant Nouveau groupe de disponibilité, sélectionner la synchronisation initiale des données](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC665529.png)
9. Dans la page **Validation**, cliquez sur **Suivant**. Cette page doit ressembler à l’illustration ci-dessous. Un avertissement concernant la configuration de l’écouteur s’affiche, car aucun écouteur du groupe de disponibilité n’est configuré. Vous pouvez ignorer cet avertissement, étant donné que ce didacticiel ne configure pas d’écouteur. Vous créerez l’écouteur ultérieurement dans le cadre de ce didacticiel.
   
    ![Assistant Nouveau groupe de disponibilité, validation](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC665530.gif)
10. Dans la page **Résumé**, cliquez sur **Terminer**, puis patientez pendant que l’assistant configure le nouveau groupe de disponibilité. Dans la page **Progression**, vous pouvez cliquer sur **Plus de détails** pour afficher la progression détaillée. Une fois l’assistant terminé, consultez la page **Résultats** pour vérifier que le groupe de disponibilité a bien été créé, comme illustré ci-dessous, puis cliquez sur **Fermer** pour quitter l’assistant.
    
     ![Assistant Nouveau groupe de disponibilité, résultats](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC665531.png)
11. Dans **l’Explorateur d’objets**, développez **Haute disponibilité AlwaysOn**, puis **Groupes de disponibilité**. Vous devez maintenant voir le nouveau groupe de disponibilité dans ce conteneur. Cliquez avec le bouton droit sur **AG1 (principal)**, puis cliquez sur **Afficher le tableau de bord**.
    
     ![Afficher le tableau de bord de groupe de disponibilité](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC665532.png)
12. Votre **tableau de bord AlwaysOn** doit ressembler à celui illustré ci-dessous. Vous pouvez voir les réplicas, le mode de basculement de chaque réplica et l'état de synchronisation.
    
     ![Tableau de bord de groupe de disponibilité](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC665533.png)
13. Revenez au **Gestionnaire de serveur**, sélectionnez **Outils**, puis lancez le **Gestionnaire du cluster de basculement**.
14. Développez **Cluster1.corp.contoso.com**, puis **Services et applications**. Sélectionnez **Rôles** et notez que le rôle du groupe de disponibilité **AG1** a été créé. Notez que AG1 n'a pas d’adresse IP permettant aux clients de base de données de se connecter au groupe de disponibilité, car vous n'avez pas configuré d’écouteur. Vous pouvez vous connecter directement au nœud principal pour les opérations de lecture-écriture et au nœud secondaire pour les requêtes en lecture seule.
    
     ![Groupe de disponibilité dans le Gestionnaire du cluster de basculement](./media/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual/IC665534.png)

> [!WARNING]
> N'essayez pas de basculer le groupe de disponibilité à partir du Gestionnaire du Cluster de basculement. Vous devez effectuer toutes les opérations de basculement à partir du **tableau de bord AlwaysOn** dans SSMS. Pour plus d’informations, consultez [Restrictions d’utilisation du Gestionnaire du cluster de basculement WSFC avec des groupes de disponibilité](https://msdn.microsoft.com/library/ff929171.aspx).
> 
> 

## <a name="configure-internal-load-balancer"></a>Configuration de l’équilibreur de charge interne
Pour vous connecter directement au groupe de disponibilité, vous devez configurer un équilibreur de charge. Celui-ci dirige le trafic client vers la machine virtuelle liée à l’adresse IP de l’écouteur et sur le port de la sonde. Ce didacticiel utilise un équilibreur de charge interne. L’équilibreur de charge interne autorise le trafic au sein du même réseau virtuel à se connecter à SQL Server. Les applications qui doivent se connecter à SQL Server via Internet nécessitent un équilibreur de charge externe (accessible sur Internet). Pour plus d’informations, consultez [Vue d’ensemble de Azure Load Balancer](../load-balancer/load-balancer-overview.md).

> [!NOTE]
> Actuellement, le portail Azure vous permet uniquement d’utiliser un port frontal spécifique une seule fois sur un équilibreur de charge. Pour utiliser le même port pour tous les écouteurs, utilisez PowerShell afin d’associer les adresses IP de l’écouteur à l’équilibreur de charge. Pour obtenir des instructions, consultez [Créer l’écouteur du groupe de disponibilité et l’équilibreur de charge | Azure](virtual-machines-windows-portal-sql-ps-alwayson-int-listener.md). 
> 
> 

### <a name="create-the-load-balancer-in-azure"></a>Créer l’équilibreur de charge dans Azure
1. Dans le portail Azure, accédez à **SQL-HA-RG** et cliquez sur **+ Ajouter**.
2. Recherchez **Équilibrage de charge**. Choisissez l’équilibreur de charge publié par Microsoft et cliquez sur **Créer**.
3. Configurez les paramètres suivants pour l’équilibreur de charge.

| Paramètre | Champ |
| --- | --- |
| **Nom** |sqlLB |
| **Schéma** |Interne |
| **Réseau virtuel** |autoHAVNET |
| **Sous-réseau** |subnet-2. Il s’agit de l’adresse IP que vous définirez pour l’écouteur dans la ressource de cluster. |
| **Affectation d’adresses IP** |Statique |
| **Adresse IP** |Utilisez une adresse de subnet-2 disponible. |
| **Abonnement** |Utilisez le même abonnement que pour toutes les autres ressources de cette solution. |
| **Emplacement** |Utilisez le même emplacement que pour toutes les autres ressources de cette solution. |

Cliquez sur **Create**.

Définissez les paramètres suivants sur l’équilibreur de charge :

| Paramètre | Champ |
| --- | --- |
| **Pool principal** - Nom |sqlLBBE |
| **Groupe à haute disponibilité SQLLBBE** |sqlAvailabilitySet |
| **Machines virtuelles SQLLBBE** |sqlserver-0, sqlserver-1 |
| **SQLLBBE utilisé par** |SQLAlwaysOnEndPointListener |
| **Sonde** - Nom |SQLAlwaysOnEndPointProbe |
| **Protocole de la sonde** |TCP |
| **Port de la sonde** |59999 - Notez que vous pouvez utiliser n’importe quel port inutilisé. |
| **Intervalle d’analyse** |5 |
| **Seuil de défaillance de la sonde** |2 |
| **Sonde utilisée par** |SQLAlwaysOnEndPointListener |
| **Règles d’équilibrage de charge** - Nom |SQLAlwaysOnEndPointListener |
| **Règles d’équilibrage de charge - Protocole** |TCP |
| **Règles d’équilibrage de charge - Port** |1433 * |
| **Règles d’équilibrage de charge - Port principal** |1433 |
| **Règles d’équilibrage de charge - Sonde** |SQLAlwaysOnEndPointProbe |
| **Règles d’équilibrage de charge - Persistance de session** |Aucun |
| **Règles d’équilibrage de charge - Délai d’inactivité** |4 |
| **Règles d’équilibrage de charge - IP flottante (retour direct du serveur)** |Activé |

> * 1433 est le port SQL Server par défaut. À utiliser pour le port frontal de l’instance par défaut. Si vous avez besoin de plusieurs groupes de disponibilité, vous devez créer des adresses IP supplémentaires pour chacun d’eux. Chaque groupe de disponibilité doit posséder son propre port frontal. Consultez [Créer un écouteur du groupe de disponibilité et un équilibreur de charge | Azure](virtual-machines-windows-portal-sql-ps-alwayson-int-listener.md).
> 
> [!NOTE]
> Vous devez activer le retour direct du serveur dans les règles d’équilibrage de charge au moment de la création.
> 
> 

Après avoir configuré l’équilibreur de charge, configurez l’écouteur sur le cluster de basculement. 

## <a name="configure-the-listener"></a>Configurer l’écouteur
La prochaine étape consiste à configurer un écouteur de groupe de disponibilité sur le cluster de basculement. 

1. Établissez une connexion RDP avec le serveur SQL Server d’ad-primary-dc à sqlserver-0.
2. Dans le Gestionnaire du cluster de basculement, notez le nom du réseau en cluster. Pour déterminer le nom du réseau en cluster dans le **Gestionnaire du cluster de basculement**, cliquez sur **Réseaux** dans le volet de gauche. Vous utiliserez ce nom dans la variable `$ClusterNetworkName` du script PowerShell.
3. Dans le Gestionnaire du cluster de basculement, développez le nom du cluster et cliquez sur **Rôles**.
4. Sous **Rôles**, cliquez avec le bouton droit sur le nom du groupe de disponibilité, puis sélectionnez **Ajouter une ressource** > **Point d’accès client**. 
5. Pour **Nom**, saisissez **aglistener**. Cliquez deux fois sur **Suivant**, puis sur **Terminer**. Ne mettez pas l'écouteur ou la ressource en ligne à ce stade.
6. Cliquez sur l'onglet **Ressources** , puis développez le Point d'accès Client vous venez de créer. Cliquez avec le bouton droit sur la ressource IP, puis cliquez sur Propriétés. Notez le nom de l’adresse IP. Utilisez ce nom dans la variable `$IPResourceName` du script PowerShell.
7. Sous **Adresse IP**, cliquez sur **Adresse IP statique** et définissez comme adresse IP statique la même adresse que celle utilisée dans le portail Azure pour l’équilibreur de charge **sqlLB**. Utilisez cette même adresse IP dans la variable `$ILBIP` du script PowerShell. 
8. Désactivez NetBIOS pour cette adresse et cliquez sur **OK**. 
9. Sur le nœud de cluster qui héberge actuellement le réplica principal, ouvrez une fenêtre PowerShell ISE avec élévation de privilèges et collez les commandes suivantes dans un nouveau script.
   
    ```
    $ClusterNetworkName = "<MyClusterNetworkName>" # the cluster network name (Use Get-ClusterNetwork on Windows Server 2012 of higher to find the name)
    $IPResourceName = "<IPResourceName>" # the IP Address resource name
    $ILBIP = "<X.X.X.X>" # the IP Address of the Internal Load Balancer (ILB). This is the static IP address for the load balancer you configured in the Azure portal.
    [int]$ProbePort = <nnnnn> # In this sample we've using 59999 for the probe port. 
   
    Import-Module FailoverClusters
   
    Get-ClusterResource $IPResourceName | Set-ClusterParameter -Multiple @{"Address"="$ILBIP";"ProbePort"=$ProbePort;"SubnetMask"="255.255.255.255";"Network"="$ClusterNetworkName";"EnableDhcp"=0}
    ```
10. Mettez à jour les variables et exécutez le script PowerShell pour configurer l’adresse IP et le port du nouvel écouteur.
11. Dans le **Gestionnaire du cluster de basculement**, cliquez avec le bouton droit sur la ressource du groupe de disponibilité, puis cliquez sur **Propriétés**. Dans l’onglet **Dépendances** , définissez le groupe de ressources qui doit dépendre du nom de réseau de l’écouteur. 
12. Affectez à la propriété du port de l’écouteur la valeur 1433. Pour ce faire, ouvrez SQL Server Management Studio, cliquez avec le bouton droit sur l’écouteur du groupe de disponibilité, puis sélectionnez Propriétés. Affectez au **Port** la valeur 1433. Vous devez utiliser le même numéro de port que celui configuré pour SQL Server. 
13. À ce stade, vous pouvez mettre l’écouteur en ligne. Cliquez avec le bouton droit sur l'écouteur, puis cliquez sur **Mettre en ligne**.

### <a name="test-the-connection-to-the-listener"></a>Tester la connexion à l’écouteur
Pour tester la connexion :

1. Établissez une connexion RDP avec le serveur SQL Server qui ne possède pas le réplica.
2. Utilisez l’utilitaire sqlcmd pour tester la connexion. Par exemple, le script suivant établit une connexion sqlcmd avec le réplica principal via l’écouteur avec une authentification Windows :
   
    sqlcmd -S "<listenerName>" -E
   
   Si l’écouteur utilise un port en plus du port 1433, vous devez spécifier le numéro de port dans le test. Par exemple, la requête suivante teste la connectivité avec le nom de l’écouteur utilisant le port 1435 :
   
        sqlcmd -S "<listenerName>",1435 -E

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur l’utilisation de SQL Server dans Azure, consultez [SQL Server sur Azure Virtual Machines](virtual-machines-windows-sql-server-iaas-overview.md).

<!--HONumber=Oct16_HO2-->


