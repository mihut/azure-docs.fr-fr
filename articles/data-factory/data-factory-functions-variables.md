---
title: Variables système et fonctions Data Factory | Microsoft Docs
description: Fournit la liste des variables système et fonctions Azure Data Factory
documentationcenter: ''
author: sharonlo101
manager: jhubbard
editor: monicar
services: data-factory

ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/11/2016
ms.author: shlo

---
# <a name="azure-data-factory---functions-and-system-variables"></a>Azure Data Factory - Variables système et fonctions
Cet article fournit des informations sur les fonctions et variables prises en charge par Azure Data Factory.

## <a name="data-factory-system-variables"></a>Variables système Data Factory
| Nom de la variable | Description | Portée de l’objet | Étendue JSON et cas d’utilisation |
| --- | --- | --- | --- |
| WindowStart |Début de l’intervalle de temps pour l’intervalle d’exécution d’activité en cours |activité |<ol><li>Spécifier des requêtes de sélection de données. Consultez les articles connexes référencés par l’article [Activités de déplacement des données](data-factory-data-movement-activities.md).</li><li>Transmettre des paramètres au script Hive (exemple ci-dessus).</li> |
| WindowEnd |Fin de l’intervalle de temps de l’intervalle d’exécution d’activité en cours |activité |Identique à ce qui précède. |
| SliceStart |Début de l’intervalle de temps pour une tranche de données en cours de génération |activité<br/>jeu de données |<ol><li>Spécifier les chemins d’accès et les noms de fichiers dynamiques tout en travaillant avec [Blob Azure](data-factory-azure-blob-connector.md) et [Jeux de données du système de fichiers](data-factory-onprem-file-system-connector.md).</li><li>Spécifier les dépendances d’entrée avec les fonctions Data Factory dans la collecte d’activité.</li></ol> |
| SliceEnd |Fin de l’intervalle de temps pour une tranche de données en cours de génération |activité<br/>jeu de données |identique à ce qui précède. |

> [!NOTE]
> Actuellement Data Factory exige que le calendrier spécifié dans l’activité corresponde exactement à la planification spécifiée dans la disponibilité du jeu de données de sortie. Cela signifie que WindowStart, WindowEnd et SliceStart et SliceEnd font toujours correspondre la même période de temps et une tranche de sortie unique.
> 
> 

## <a name="data-factory-functions"></a>Fonctions Data Factory
Vous pouvez utiliser des fonctions dans Data Factory avec les variables système mentionnées aux fins suivantes :

1. Spécification de requêtes de sélection de données (consultez les articles connexes référencés par l’article [Activités de déplacement des données](data-factory-data-movement-activities.md) ).
   
   La syntaxe pour appeler une fonction Data Factory est : **$$<function>** pour les requêtes de sélection de données et d’autres propriétés de l’activité et des jeux de données.  
2. Spécification de dépendances d’entrée avec les fonctions Data Factory dans la collecte d’activité (voir exemple ci-dessus).
   
    $$ n’est pas nécessaire pour spécifier des expressions de dépendance d’entrée.   

Dans l’exemple suivant, la propriété **sqlReaderQuery** d’un fichier JSON est affectée à une valeur renvoyée par la fonction **Text.Format**. Cet exemple utilise également une variable système nommée **WindowStart**, qui représente l’heure de début de la fenêtre d’activité à exécuter.

    {
        "Type": "SqlSource",
        "sqlReaderQuery": "$$Text.Format('SELECT * FROM MyTable WHERE StartTime = \\'{0:yyyyMMdd-HH}\\'', WindowStart)"
    }

### <a name="functions"></a>Fonctions
Les tables qui suivent répertorient toutes les fonctions dans Azure Data Factory :

| Catégorie | Fonction | Paramètres | Description |
| --- | --- | --- | --- |
| Time |AddHours(X,Y) |X: DateTime  <br/><br/>Y: int |Ajoute Y heures à l’heure donnée X. <br/><br/>Exemple : 9/5/2013 12:00:00 PM + 2 heures = 9/5/2013 2:00:00 PM |
| Temps |AddMinutes(X,Y) |X: DateTime  <br/><br/>Y: int |Ajoute Y minutes à X.<br/><br/>Exemple : 9/15/2013 12:00:00 PM + 15 minutes = 9/15/2013 12:15:00 PM |
| Time |StartOfHour(X) |X: DateTime |Obtient l’heure de début de l’heure représentée par le composant heure de X. <br/><br/>Exemple : StartOfHour 9/15/2013 05:10:23 PM est 9/15/2013 05:00:00 PM |
| Date |AddDays(X,Y) |X: DateTime <br/><br/>Y: int |Ajoute Y jours à X.<br/><br/>Exemple : 9/15/2013 12:00:00 PM + 2 jours = 9/17/2013 12:00:00 PM |
| Date |AddMonths(X,Y) |X: DateTime <br/><br/>Y: int |Ajoute Y mois à X.<br/><br/>Exemple : 9/15/2013 12:00:00 PM + 1 mois = 10/15/2013 12:00:00 PM |
| Date |AddQuarters(X,Y) |X: DateTime  <br/><br/>Y: int |Ajoute Y * 3 mois à X.<br/><br/>Exemple : 9/15/2013 12:00:00 PM + 1 trimestre = 12/15/2013 12:00:00 PM |
| Date |AddWeeks(X,Y) |X: DateTime <br/><br/>Y: int |Ajoute Y * 7 jours à X<br/><br/>Exemple : 15/9/2013 12:00:00 PM + 1 semaine = 22/9/2013 12:00:00 PM |
| Date |AddYears(X,Y) |X: DateTime <br/><br/>Y: int |Ajoute Y années à X.<br/><br/>Exemple : 9/15/2013 12:00:00 PM + 1 an = 9/15/2014 12:00:00 PM |
| Date |Day(X) |X: DateTime |Obtient le composant jour de X.<br/><br/>Exemple : le jour du 9/15/2013, 12:00:00 PM est 9. |
| Date |DayOfWeek(X) |X: DateTime |Obtient le composant semaine de X.<br/><br/>Exemple : DayOfWeek du 9/15/2013, 12:00:00 PM est dimanche. |
| Date |DayOfYear(X) |X: DateTime |Permet d’obtenir le jour de l’année représenté par le composant année de X.<br/><br/>Exemples :<br/>12/1/2015 : jour 335 de 2015<br/>12/31/2015 : jour 365 de 2015<br/>12/31/2016 : jour 366 de 2016 (année bissextile) |
| Date |DaysInMonth(X) |X: DateTime |Permet d’obtenir les jours du mois représentés par le composant mois du paramètre X.<br/><br/>Exemple : DaysInMonth du 9/15/2013 s’élève à 30 puisqu’il y a 30 jours dans le mois de septembre. |
| Date |EndOfDay(X) |X: DateTime |Obtient la valeur date-heure qui représente la fin de la journée (composant jour) de X.<br/><br/>Exemple : EndOfDay du 9/15/2013 05:10:23 PM est 9/15/2013 11:59:59 PM. |
| Date |EndOfMonth(X) |X: DateTime |Permet d’obtenir la fin du mois représentée par le composant mois du paramètre X. <br/><br/>Exemple : EndOfMonth 9/15/2013 05:10:23 PM est 9/30/2013 11:59:59 PM (date-heure qui représente la fin du mois de septembre) |
| Date |StartOfDay(X) |X: DateTime |Permet d’obtenir le début de la journée représenté par le composant jour du paramètre X.<br/><br/>Exemple : la valeur StartOfDay de 9/15/2013 05:10:23 PM est 9/15/2013 12:00:00 AM. |
| DateTime |From(X) |X: String |Analyser la chaîne X à une heure de date. |
| DateTime |Ticks(X) |X: DateTime |Permet d’obtenir la propriété de graduation du paramètre X. Un cycle est égal à 100 nanosecondes. La valeur de cette propriété représente le nombre de graduations écoulées depuis 12:00:00 minuit, le 1er janvier 0001. |
| Texte |Format(X) |X : variable de chaîne |Met en forme le texte. |

#### <a name="text.format-example"></a>Exemple Text.Format
    "defines": { 
        "Year" : "$$Text.Format('{0:yyyy}',WindowStart)",
        "Month" : "$$Text.Format('{0:MM}',WindowStart)",
        "Day" : "$$Text.Format('{0:dd}',WindowStart)",
        "Hour" : "$$Text.Format('{0:hh}',WindowStart)"
    }

Consultez la rubrique [Chaînes de format de date et d’heure personnalisées](https://msdn.microsoft.com/library/8kb3ddd4.aspx) , qui décrit les différentes options de formatage que vous pouvez utiliser (par exemple : aa et aaaa). 

> [!NOTE]
> Lorsque vous utilisez une fonction au sein d’une autre fonction, vous n’avez pas besoin d’utiliser le préfixe **$$** de la fonction interne. Par exemple : $$Text.Format('PartitionKey eq \\'my_pkey_filter_value\\' et RowKey ge \\'{0:yyyy-MM-dd HH:mm:ss}\\'', Time.AddHours(SliceStart, -6)). Dans cet exemple, notez que le préfixe **$$** n’est pas utilisé pour la fonction **Time.AddHours**. 
> 
> 

<!--HONumber=Oct16_HO2-->


