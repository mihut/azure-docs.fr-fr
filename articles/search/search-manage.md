---
title: Administration des services pour Azure Search dans le portail Azure
description: Gérez Azure Search, un service de recherche cloud hébergé sur Microsoft Azure à l’aide du portail Azure.
services: search
documentationcenter: ''
author: HeidiSteen
manager: jhubbard
editor: ''
tags: azure-portal

ms.service: search
ms.devlang: rest-api
ms.workload: search
ms.topic: article
ms.tgt_pltfrm: na
ms.date: 10/17/2016
ms.author: heidist

---
# <a name="service-administration-for-azure-search-in-the-azure-portal"></a>Administration des services pour Azure Search dans le portail Azure
> [!div class="op_single_selector"]
> * [Portail](search-manage.md)
> * [PowerShell](search-manage-powershell.md)
> * [API REST](search-get-started-management-api.md)
> 
> 

Azure Search est un service de recherche entièrement géré, basé sur le cloud, utilisé pour la création d’une expérience de recherche riche dans des applications personnalisées. Cet article aborde les tâches d’ *administration des services* que vous pouvez effectuer dans le [portail Azure](https://portal.azure.com) pour un service de recherche que vous avez déjà approvisionné. *Service administration* est légère de par sa conception et se limite à ce qui suit :

* Gérer et sécuriser l’accès aux *clés API* utilisées pour l’accès en lecture ou en écriture à votre service.
* Ajustez la capacité de service en modifiant l’allocation des partitions et des réplicas.
* Surveillez l’utilisation des ressources, par rapport aux limites maximales de votre niveau de service.

**Non compris** 

*gestion de contenu* (ou gestion des index) fait référence à des opérations telles que l’analyse de la recherche du trafic pour comprendre le volume de requêtes, détecter les conditions recherchées par les personnes et déterminer dans quelle mesure les résultats de recherche réussissent à guider les clients vers des documents spécifiques de votre index. La gestion de contenu dépasse le cadre de cet article. Pour obtenir des instructions sur la façon d’obtenir un aperçu des opérations internes au niveau de l’index, consultez la page [Fonctionnalité Rechercher l’analyse du trafic pour Azure Search](search-traffic-analytics.md).

*performances des requêtes* dépassent le cadre de cet article. Pour plus d’informations, consultez [Performance and optimization in Azure Search](search-performance-optimization.md)(Performances et optimisation dans Azure Search).

Azure Search ne fournit pas de solutions intégrées pour la sauvegarde et la restauration ou la récupération d’urgence. Pour les clients qui envoient des objets et des données à leur service, le code source pour créer et remplir un index est de fait l’option de restauration si vous supprimez un index par erreur. Pour la récupération d’urgence, les clients choisissent la redondance via un service supplémentaire dans un centre de données régional différent. Pour plus d’informations, consultez [Performance and optimization in Azure Search](search-performance-optimization.md)(Performances et optimisation dans Azure Search).

<a id="admin-rights"></a>

## <a name="administrator-rights-in-azure-search"></a>Droits d’administrateur dans Azure Search
L’approvisionnement ou le retrait du service lui-même peut être effectué par un administrateur d’abonnement Azure ou un coadministrateur.

Au sein d’un service, toute personne ayant accès à l’URL du service et à une clé API d’administration a accès en lecture-écriture au service, avec une possibilité proportionnée d’ajouter, de supprimer ou de modifier des objets de serveur telles que des clés API, des index, des indexeurs, des sources de données, des planifications et des attributions, tels qu’implémentés dans les [rôles définis par RBAC](#rbac).

Toutes les interactions utilisateur avec Azure Search sont soumises à un de ces modes : accès en lecture-écriture au service (droits d’administrateur) ou un accès en lecture seule au service (droits de requête). Pour plus d’informations, consultez la page [Gestion des clés API](#manage-keys).

<a id="sys-info"></a>

## <a name="logging-in-azure-search-and-system-information"></a>Journalisation dans Azure Search et informations système
Azure Search n’expose pas les fichiers journaux d’un service via le portail ou les interfaces de programmation. Au niveau de base et supérieur, Microsoft surveille tous les services Azure Search pour vérifier la disponibilité de 99,9 % par contrat de niveau de service (SLA). Si le service est lent ou si le débit des demandes tombe en dessous des seuils de contrat SLA, les équipes de support passent en revue les fichiers journaux à leur disposition et résolvent le problème.

En termes d’informations générales relatives à votre service, vous pouvez obtenir des informations de plusieurs façons :

* Sur le portail, le tableau de bord du service, via les notifications, les propriétés et les messages d’état.
* À l’aide de [PowerShell](search-manage-powershell.md) ou de [l’API REST de gestion](https://msdn.microsoft.com/library/azure/dn832684.aspx) pour [obtenir les propriétés du service](https://msdn.microsoft.com/library/azure/dn832694.aspx), ou de l’état sur l’utilisation des ressources d’index.
* Via la [recherche du trafic d’analyse](search-traffic-analytics.md), comme indiqué précédemment.

<a id="manage-keys"></a>

## <a name="manage-the-api-keys"></a>Gestion des clés API
Toutes les demandes adressées à votre service de recherche ont besoin d’une clé API générée spécialement pour votre service. Cette clé API constitue le seul mécanisme d’authentification de l’accès au point de terminaison de votre service de recherche. 

Une clé API est une chaîne composée de nombres et de lettres générée de manière aléatoire. Elle est générée uniquement par votre service. Via [les autorisations RBAC](#rbac), vous pouvez supprimer ou lire les clés, mais vous ne pouvez pas remplacer une clé générée avec une chaîne définie par l’utilisateur (en particulier, si vous avez des mots de passe que vous utilisez régulièrement, vous ne pouvez pas remplacer une clé API par un mot de passe défini par l’utilisateur). 

Deux types de clés sont utilisés pour accéder à votre service de recherche :

* Admin (valable pour toute opération de lecture/écriture par rapport au service)
* Requête (valable pour les opérations en lecture seule, telles que les requêtes par rapport à un index)

Une clé API Admin est créée lorsque le service est approvisionné. Bien qu’il existe deux clés d’administration, désignées comme *principale* et *secondaire*, celles-ci sont en fait interchangeables. Chaque service dispose de deux clés Admin que vous pouvez interchanger sans perdre l’accès à votre service. Vous pouvez régénérer l'une des clés Admin, mais vous ne pouvez pas augmenter leur nombre total. Il y a, au maximum, deux clés Admin par service de recherche.

Les clés Requête sont conçues pour les applications clientes qui appellent directement le service Search. Vous pouvez créer, au maximum, 50 clés de ce type. Dans le code d’application, vous spécifiez l’URL de recherche et une clé API de requête pour autoriser l’accès en lecture seule au service. Le code de votre application spécifie également l’index utilisé par votre application. Ensemble, le point de terminaison, une clé API pour un accès en lecture seule et un index cible définissent le niveau de portée et d’accès de la connexion à partir de votre application cliente.

Pour obtenir ou régénérer des clés API, ouvrez le tableau de bord des services. Cliquez sur **CLÉS** pour afficher la page de gestion des clés. Les commandes de régénération ou de création de clés figurent en haut de la page. Par défaut, seules les clés Admin sont créées. Les clés API de requête doivent être créées manuellement.

 ![][9]

<a id="rbac"></a>

## <a name="set-rbac-roles-on-administrative-access-for-azure-search"></a>Définissez des rôles RBAC sur l’accès administratif pour Azure Search
Azure offre un [modèle d’autorisation par rôle global](../active-directory/role-based-access-control-configure.md) pour tous les services gérés via le portail ou les API Resource Manager. Les rôles Propriétaire, Collaborateur et Lecteur définissent le niveau d’administration des services pour les utilisateurs, les groupes et les principaux de sécurité Active Directory que vous assignez à chaque rôle. 

Pour Azure Search, les autorisations RBAC déterminent les tâches administratives suivantes :

| Rôle | Task |
| --- | --- |
| Propriétaire |Création ou suppression du service ou de tout objet sur le service, y compris les clés API, les index, les indexeurs, les sources de données d’indexeur et les planifications de l’indexeur.<p>Afficher l’état du service, notamment des compteurs et la taille du stockage.<p>Ajout ou suppression d'appartenance à un rôle (seul un Propriétaire peut gérer l'appartenance à un rôle).<p>Les administrateurs d’abonnement et de service appartiennent automatiquement au rôle Propriétaire. |
| Collaborateur |Même niveau d’accès que le Propriétaire, à l’exception de la gestion des rôles RBAC. Par exemple, un Collaborateur peut visualiser et régénérer `api-key`, mais il ne peut pas modifier les appartenances aux rôles. |
| Lecteur |Affichage de l'état du service et des clés Requête. Les membres de ce rôle ne peuvent pas modifier la configuration du service, ni afficher des clés Admin. |

Notez que les rôles n'accordent pas de droits d'accès au point de terminaison de service. Les opérations du service Search telles que la gestion ou le remplissage d'index, tout comme les requêtes de données de recherche, sont contrôlées via des clés api, et non par des rôles. Pour en savoir plus, consultez la section « Autorisation pour les opérations de gestion et les opérations de données » de la page [Contrôle d’accès en fonction du rôle](../active-directory/role-based-access-control-what-is.md).

<a id="secure-keys"></a>

## <a name="secure-the-api-keys"></a>Sécurisation des clés API
La sécurité des clés est assurée en limitant l’accès via le portail ou des interfaces Resource Manager (PowerShell ou interface de ligne de commande). Comme indiqué, les administrateurs des abonnements peuvent afficher et régénérer toutes les clés API. Par précaution, passez en revue les affectations de rôle pour comprendre qui a accès aux clés Admin.

1. Dans le tableau de bord du service, cliquez sur l’icône d’accès pour ouvrir le panneau des utilisateurs.
   ![][7]
2. Dans Utilisateurs, vérifiez les affectations de rôles existantes. Normalement, les administrateurs d’abonnement ont déjà un accès complet au service via le rôle Propriétaire.
3. Pour aller plus loin, cliquez sur les **administrateurs d’abonnement** , puis développez la liste d’affectation de rôle pour savoir qui possède des droits d’administration conjoints sur votre service de recherche.

Une autre façon d’afficher les autorisations d’accès consiste à cliquer sur **Rôles** sur le panneau Utilisateurs. Cela affiche les rôles disponibles et le nombre d’utilisateurs ou de groupes affectés à chaque rôle.

<a id="sub-5"></a>

## <a name="monitor-resource-usage"></a>surveiller l’utilisation des ressources ;
Sur le tableau de bord, l’analyse des ressources se limite aux informations affichées dans le tableau de bord des services et à quelques mesures que vous pouvez obtenir en interrogeant le service. Dans la section Utilisation du tableau de bord des services, vous pouvez déterminer rapidement si les niveaux des ressources de partition sont adaptés à votre application.

L'API du service Search vous permet d'obtenir le nombre de documents et d'index. Des limites strictes sont associées à ces valeurs sur la base du niveau de tarification. Pour plus d’informations, voir [Limites de service de recherche](search-limits-quotas-capacity.md). 

* [Obtention de statistiques d'index](http://msdn.microsoft.com/library/dn798942.aspx)
* [Nombre de documents](http://msdn.microsoft.com/library/dn798924.aspx)

> [!NOTE]
> Il arrive qu'une limite soit surévaluée en raison des options de mise en cache. Lors de l'utilisation du service partagé, par exemple, il se peut que le nombre de documents dépasse la limite stricte fixée à 10 000 documents. Cette surévaluation est temporaire et sera détectée lors de la prochaine vérification de l'application des limites. 
> 
> 

<a id="scale"></a>

## <a name="scale-up-or-down"></a>Augmentation ou réduction d'échelle
Au départ, chaque service de recherche comporte, au minimum, un réplica et une partition. Si vous vous êtes inscrit pour bénéficier de ressources dédiées à l’aide des niveaux de tarification [De base et Standard](search-limits-quotas-capacity.md), vous pouvez cliquer sur la vignette **SCALE** (MISE À L’ÉCHELLE) sur le tableau de bord des services pour réajuster le nombre de partitions et de réplicas utilisés par votre service.

Lorsque vous ajoutez des capacités à travers l’une des ressources, le service les utilise automatiquement. Aucune autre action n'est requise de votre part. Il y aura cependant un léger décalage avant que l'effet de la nouvelle ressource ne soit perceptible. L’approvisionnement des ressources supplémentaires demande au moins 15 minutes.

 ![][10]

### <a name="add-replicas"></a>Ajout de réplicas
Pour augmenter le nombre de requêtes par seconde (RPS) ou parvenir à une haute disponibilité, il convient d'ajouter des réplicas. Chaque réplica comporte une copie d’un index. L’ajout d’un réplica se traduit donc par un ou plusieurs index supplémentaires disponibles pour satisfaire les demandes de requête. Au moins 3 réplicas sont nécessaires pour la haute disponibilité (pour plus d’informations, consultez [Planification de la capacité](search-capacity-planning.md)).

Un service de recherche qui comporte davantage de réplicas peut équilibrer la charge des demandes de requête sur un plus grand nombre d'index. Pour un nombre de requêtes donné, le débit sera plus élevé si davantage de copies de l'index sont disponibles pour leur traitement. Si vous constatez une latence des requêtes, la mise en ligne des répliques supplémentaires aura, à n'en pas douter, un effet positif sur les performances.

Bien que l'ajout de réplicas à votre service entraîne une augmentation du débit des requêtes, celui-ci n'est pas exactement multiplié par deux ou par trois. Toutes les applications de recherche sont, en effet, soumises à des facteurs externes susceptibles d'affecter les performances des requêtes. La complexité des requêtes et la latence réseau, notamment, sont deux facteurs qui contribuent à la fluctuation des temps de réponse des requêtes.

### <a name="add-partitions"></a>Ajout de partitions
La plupart des applications de service intègrent le besoin de plusieurs réplicas plutôt que de plusieurs partitions. Vous pouvez ajouter des partitions lorsqu'un plus grand nombre de documents est nécessaire si vous êtes inscrit au service Standard. Le niveau De base ne fournit pas d’autres partitions.

Au niveau Standard, les partitions sont ajoutées par multiples de 12 (notamment 1, 2, 3, 4, 6 ou 12). Il s’agit d’un artefact de partitionnement. Un index est créé dans 12 fragments (ou shards) qui peuvent tous être stockés dans 1 partition ou répartis équitablement dans 2, 3, 4, 6 ou 12 partitions (un fragment par partition).

### <a name="remove-replicas"></a>Suppression de réplicas
Après une période de volume de requêtes intense, il est probable que vous supprimiez des réplicas lorsque la charge des requêtes de recherche est revenue à la normale (à la fin d’une période de vente, par exemple).

Pour ce faire, il vous suffit de faire coulisser le curseur des réplicas sur une valeur plus faible. Rien de plus ! La réduction du nombre de réplicas entraîne l'abandon des machines virtuelles dans le centre de données. Désormais, vos opérations de requête et d'ingestion de données s'exécuteront sur un nombre moins élevé de machines virtuelles. La limite minimale est de 1 réplica.

### <a name="remove-partitions"></a>Suppression de partitions
Contrairement à la suppression de réplicas, qui n'exige aucune opération supplémentaire de votre part, utiliser plus de volume de stockage que la capacité disponible après réduction peut entraîner une surcharge de travail. Par exemple, si votre solution utilise trois partitions, le fait de passer à une ou deux partitions générera une erreur si le nouvel espace de stockage est inférieur à celui requis. Comme vous pouvez vous y attendre, deux solutions s’offrent alors à vous : soit supprimer des index ou documents au sein d’un index associé afin de libérer de l’espace, soit conserver la configuration actuelle.

Aucune méthode de détection ne vous permet d'identifier les fragments d'index qui sont stockés sur des partitions spécifiques. Chaque partition fournit environ 25 Go de stockage. Vous devrez donc réduire l'espace de stockage à une taille pouvant être prise en charge par le nombre de partitions dont vous disposez. Si vous souhaitez revenir à une seule partition, celle-ci devra contenir les 12 fragments.

Pour faciliter la planification, vous pouvez vérifier le stockage (voir la page [Obtenir des statistiques d'index](http://msdn.microsoft.com/library/dn798942.aspx)) afin de connaître l'espace réellement utilisé. 

<a id="advanced-deployment"></a>

## <a name="best-practices-on-scale-and-deployment-(video)"></a>Meilleures pratiques sur l’échelle et le déploiement (vidéo)
Cette vidéo de 30 minutes passe en revue les meilleures pratiques pour les scénarios de déploiement avancés, y compris les charges de travail géolocalisées. Vous pouvez également voir [Performances et optimisation dans Azure Search](search-performance-optimization.md) pour les pages d’aide qui couvrent les mêmes points.

> [!VIDEO https://channel9.msdn.com/Events/Microsoft-Azure/AzureCon-2015/ACON319/player]
> 
> 

<a id="next-steps"></a>

## <a name="next-steps"></a>Étapes suivantes
Une fois que vous avez compris les types d’opérations relatives à l’administration du service, prenez en compte les différentes approches de gestion des services :

* [PowerShell](search-manage-powershell.md)
* [l’API REST de gestion ;](search-get-started-management-api.md)

En outre, si ce n’est pas déjà fait, examinez l’ [article sur les performances et l’optimisation](search-performance-optimization.md), et regardez la vidéo indiquée dans la section précédente pour voir plus de démonstrations des techniques recommandées et les découvrir de manière plus approfondie.

<!--Image references-->
[7]: ./media/search-manage/rbac-icon.png
[8]: ./media/search-manage/Azure-Search-Manage-1-URL.png
[9]: ./media/search-manage/Azure-Search-Manage-2-Keys.png
[10]: ./media/search-manage/Azure-Search-Manage-3-ScaleUp.png






<!--HONumber=Oct16_HO2-->


