---
title: Approvisionnement de l’application web avec le cache Redis
description: Utilisez un modèle Azure Resource Manager pour déployer une application web Azure avec le cache Redis.
services: app-service
documentationcenter: ''
author: steved0x
manager: erickson-doug
editor: ''

ms.service: app-service
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/09/2016
ms.author: sdanie

---
# Création d’une application web avec le cache Redis à l’aide d’un modèle
Dans cette rubrique, vous allez apprendre à créer un modèle Azure Resource Manager qui déploie une application web Azure avec le cache Redis. Vous allez apprendre comment définir les ressources à déployer et configurer les paramètres qui sont spécifiés lors de l’exécution du déploiement. Vous pouvez utiliser ce modèle pour vos propres déploiements, ou le personnaliser afin qu’il réponde à vos besoins.

Pour en savoir plus sur la création de modèles, voir [Création de modèles Azure Resource Manager](../resource-group-authoring-templates.md).

Pour le modèle complet, consultez la page [Modèle d’application web avec le cache Redis](https://github.com/Azure/azure-quickstart-templates/blob/master/201-web-app-with-redis-cache/azuredeploy.json).

## Ce que vous allez déployer
Dans ce modèle, vous allez déployer :

* Application web Azure
* Cache Redis Azure.

Pour exécuter automatiquement le déploiement, cliquez sur le bouton ci-dessous :

[![Déploiement sur Azure](./media/cache-web-app-arm-with-redis-cache-provision/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-web-app-with-redis-cache%2Fazuredeploy.json)

## Paramètres à spécifier
[!INCLUDE [app-service-web-deploy-web-parameters](../../includes/app-service-web-deploy-web-parameters.md)]

[!INCLUDE [cache-deploy-parameters](../../includes/cache-deploy-parameters.md)]

## Variables pour les noms
Ce modèle utilise des variables afin de créer des noms pour les ressources. Il utilise la fonction [uniqueString](../resource-group-template-functions.md#uniquestring) pour construire une valeur basée sur l’identifiant du groupe de ressources.

    "variables": {
      "hostingPlanName": "[concat('hostingplan', uniqueString(resourceGroup().id))]",
      "webSiteName": "[concat('webSite', uniqueString(resourceGroup().id))]",
      "cacheName": "[concat('cache', uniqueString(resourceGroup().id))]"
    },


## Ressources à déployer
[!INCLUDE [app-service-web-deploy-web-host](../../includes/app-service-web-deploy-web-host.md)]

### Cache Redis
Crée le cache Redis Azure utilisé avec l'application web. Le nom du cache est spécifié dans la variable **cacheName**.

Le modèle crée le cache au même emplacement que le groupe de ressources.

    {
      "name": "[variables('cacheName')]",
      "type": "Microsoft.Cache/Redis",
      "location": "[resourceGroup().location]",
      "apiVersion": "2015-08-01",
      "dependsOn": [ ],
      "tags": {
        "displayName": "cache"
      },
      "properties": {
        "sku": {
          "name": "[parameters('cacheSKUName')]",
          "family": "[parameters('cacheSKUFamily')]",
          "capacity": "[parameters('cacheSKUCapacity')]"
        }
      }
    }


### Application web
Crée l’application web avec le nom spécifié dans la variable **webSiteName**.

Notez que l’application web est configurée selon des paramètres qui lui permettent d’utiliser le cache Redis. Ces paramètres sont créés de façon dynamique en fonction des valeurs fournies lors du déploiement.

    {
      "apiVersion": "2015-08-01",
      "name": "[variables('webSiteName')]",
      "type": "Microsoft.Web/sites",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[concat('Microsoft.Web/serverFarms/', variables('hostingPlanName'))]",
        "[concat('Microsoft.Cache/Redis/', variables('cacheName'))]"
      ],
      "tags": {
        "[concat('hidden-related:', resourceGroup().id, '/providers/Microsoft.Web/serverfarms/', variables('hostingPlanName'))]": "empty",
        "displayName": "Website"
      },
      "properties": {
        "name": "[variables('webSiteName')]",
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]"
      },
      "resources": [
        {
          "apiVersion": "2015-08-01",
          "type": "config",
          "name": "appsettings",
          "dependsOn": [
            "[concat('Microsoft.Web/Sites/', variables('webSiteName'))]",
            "[concat('Microsoft.Cache/Redis/', variables('cacheName'))]"
          ],
          "properties": {
            "CacheConnection": "[concat(variables('cacheName'),'.redis.cache.windows.net,abortConnect=false,ssl=true,password=', listKeys(resourceId('Microsoft.Cache/Redis', variables('cacheName')), '2015-08-01').primaryKey)]"
          }
        }
      ]
    }

## Commandes pour l’exécution du déploiement
[!INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

### PowerShell
    New-AzureRmResourceGroupDeployment -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-web-app-with-redis-cache/azuredeploy.json -ResourceGroupName ExampleDeployGroup

### Interface de ligne de commande Azure
    azure group deployment create --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-web-app-with-redis-cache/azuredeploy.json -g ExampleDeployGroup

<!---HONumber=AcomDC_0810_2016-->