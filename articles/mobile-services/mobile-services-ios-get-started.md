---
title: Prendre en main Azure Mobile Services pour les applications iOS | Microsoft Docs
description: Suivez ce didacticiel pour commencer à utiliser Azure Mobile Services pour le développement iOS.
services: mobile-services
documentationcenter: ios
author: krisragh
manager: dwrede
editor: ''

ms.service: mobile-services
ms.workload: mobile
ms.tgt_pltfrm: mobile-ios
ms.devlang: objective-c
ms.topic: hero-article
ms.date: 07/21/2016
ms.author: krisragh

---
# <a name="getting-started"> </a>Prise en main de Mobile Services
[!INCLUDE [mobile-services-selector-get-started](../../includes/mobile-services-selector-get-started.md)]

&nbsp;

[!INCLUDE [mobile-service-note-mobile-apps](../../includes/mobile-services-note-mobile-apps.md)]

> Pour la version Mobile Apps équivalente de cette rubrique, consultez l’article [Créer une application iOS dans Azure Mobile Apps](../app-service-mobile/app-service-mobile-ios-get-started.md).
> 
> 

Ce didacticiel présente l’ajout d’un service principal cloud à une application iOS à l’aide d’Azure Mobile Services.

Dans ce didacticiel, vous allez créer un service mobile et une simple application *To do list* qui stocke les données d'application dans le nouveau service mobile. Le service mobile à créer utilise du code JavaScript pour la logique métier côté serveur. Pour créer un service mobile avec une logique métier côté serveur dans .NET, consultez la [version de serveur principal .NET] de cette rubrique.

> [!NOTE]
> Pour suivre ce didacticiel, vous avez besoin d'un compte Azure. Si vous n'avez pas de compte, vous pouvez vous inscrire pour une évaluation d'Azure et obtenir des [services mobiles gratuits que vous pourrez conserver après l'expiration de votre période d'évaluation](https://azure.microsoft.com/pricing/details/mobile-services/). Pour plus d'informations, consultez la page [Version d'évaluation gratuite d'Azure](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=AE564AB28&amp;returnurl=http%3A%2F%2Fazure.microsoft.com%2Ffr-FR%2Fdevelop%2Fmobile%2Ftutorials%2Fget-started-ios%2F%20).
> 
> 

## <a name="create-new-service"> </a>Création d'un service mobile
[!INCLUDE [mobile-services-create-new-service](../../includes/mobile-services-create-new-service.md)]

## Création d'une application iOS
Vous pouvez suivre un démarrage rapide facile dans le portail Azure Classic pour créer une application connectée à votre service mobile :

1. Dans le [portail Azure Classic], cliquez sur **Mobile Services**, puis sur le service mobile que vous venez de créer.
2. Dans l’onglet de démarrage rapide, cliquez sur **iOS** sous **Choisir une plateforme**, puis développez **Créer une application iOS**. Cette opération affiche les étapes permettant de créer une application iOS connectée à votre service mobile.
3. Cliquez sur **Create TodoItem table** pour créer une table permettant de stocker les données d'application.
4. Sous **Télécharger et exécuter l'application**, cliquez sur **Télécharger**. Cette opération télécharge le projet de votre exemple d'application *To do list* qui est connectée à votre service mobile, ainsi que le Kit de développement logiciel (SDK) Mobile Services iOS. Enregistrez le fichier projet compressé sur votre ordinateur local et notez l'emplacement où vous l'avez enregistré.

## Exécution de votre nouvelle application iOS
[!INCLUDE [mobile-services-ios-run-app](../../includes/mobile-services-ios-run-app.md)]

<ol start="4"> <li><p>De retour dans le [portail Azure Classic], cliquez sur l’onglet **DONNÉES**, puis cliquez sur la table **TodoItem**. Cela vous permet de parcourir les données insérées par l’application dans la table.<p></li></ol></p>

## <a name="next-steps"> </a>Étapes suivantes
Apprenez à effectuer d’autres tâches importantes dans Mobile Services :

* [Prise en main de la synchronisation des données hors connexion] <br/>Découvrez comment utiliser la synchronisation des données hors connexion pour rendre votre application réactive et robuste.
* [Ajout de l’authentification à une application existante] <br/>Découvrez comment authentifier les utilisateurs de votre application avec un fournisseur d’identité.
* [Ajout de notifications Push à votre application existante] <br/>Découvrez comment envoyer une notification Push très basique à votre application.

[!INCLUDE [app-service-disqus-feedback-slug](../../includes/app-service-disqus-feedback-slug.md)]

<!-- Anchors. -->
[Getting started with Mobile Services]: #getting-started
[Create a new mobile service]: #create-new-service
[Define the mobile service instance]: #define-mobile-service-instance
[Next Steps]: #next-steps

<!-- Images. -->
[6]: ./media/mobile-services-ios-get-started/mobile-portal-quickstart-ios.png
[7]: ./media/mobile-services-ios-get-started/mobile-quickstart-steps-ios.png
[8]: ./media/mobile-services-ios-get-started/mobile-xcode-project.png

[10]: ./media/mobile-services-ios-get-started/mobile-quickstart-startup-ios.png
[11]: ./media/mobile-services-ios-get-started/mobile-data-tab.png
[12]: ./media/mobile-services-ios-get-started/mobile-data-browse.png


<!-- URLs. -->
[Prise en main de la synchronisation des données hors connexion]: mobile-services-ios-get-started-offline-data.md
[Ajout de l’authentification à une application existante]: mobile-services-dotnet-backend-ios-get-started-users.md
[Ajout de notifications Push à votre application existante]: mobile-services-dotnet-backend-ios-get-started-push.md


[Mobile Services iOS SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533
[portail Azure Classic]: https://manage.windowsazure.com/
[XCode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[version de serveur principal .NET]: mobile-services-dotnet-backend-ios-get-started.md

<!---HONumber=AcomDC_0727_2016-->