---
title: "Présentation de Service Bus Relay | Microsoft Docs"
description: "Présentation de Service Bus Relay."
services: service-bus
documentationcenter: .net
author: sethmanheim
manager: timlt
editor: 
ms.assetid: 1038a2d8-5def-4f48-8703-cb0070fc5f10
ms.service: service-bus
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: get-started-article
ms.date: 09/01/2016
ms.author: sethm
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 0482096cbec6a5e4b7b13ea662a180cd9b96e85f


---
# <a name="overview-of-service-bus-relay"></a>Présentation de Service Bus Relay
Composant majeur de Service Bus, un service *relay* centralisé (mais dont la charge est hautement équilibrée) vous permet de créer des applications hybrides qui s’exécutent à la fois dans un centre de données Azure et dans votre propre environnement d’entreprise local.  Service Bus Relay prend en charge une grande variété de protocoles de transport et normes de services web. Cela inclut SOAP, WS-* et même REST. Le service relay facilite vos applications hybrides en offrant la possibilité d’exposer les services WCF (Windows Communication Foundation) qui résident dans un réseau d’entreprise sur le cloud public en toute sécurité, sans avoir à ouvrir une connexion de pare-feu ni à exiger des modifications intrusives dans une infrastructure de réseau d’entreprise. 

![Concepts du service WCF Relay](./media/service-bus-relay-overview/sb-relay-01.png)

Le service de relais prend en charge la messagerie unidirectionnelle standard, la messagerie demande/réponse et la messagerie d’homologue à homologue. Il prend également en charge la distribution des événements sur Internet pour activer les scénarios de publication/abonnement et la communication par socket bidirectionnelle pour une efficacité accrue de point à point. 

Dans le modèle de messagerie par relais, un service local se connecte au service de relais via un port sortant et crée un socket bidirectionnel lié à des adresses de rendez-vous spécifiques. Le client peut ensuite communiquer avec le service local en envoyant des messages au service de relais en ciblant l'adresse de rendezvous. Le service de relais « relayera » ensuite les messages au service local via le socket bidirectionnel déjà en place. Le client n’a pas besoin d’une connexion directe au service local ni de savoir où se trouve le service, et le service local n’a pas besoin d’ouvrir de ports entrants sur le pare-feu.

Vous lancez la connexion entre votre service local et le service de relais à l'aide d'une suite de liaisons de « relais » WCF. En coulisses, les liaisons de relais se mappent à de nouveaux éléments de liaison de transport destinés à créer des composants de canal WCF qui s'intègrent à Service Bus dans le cloud. 

## <a name="next-steps"></a>Étapes suivantes
Pour plus de détails sur Service Bus Relay, consultez les rubriques suivantes.

* [Présentation de l’architecture d’Azure Service Bus](../service-bus-messaging/service-bus-fundamentals-hybrid-solutions.md)
* [Guide pratique pour utiliser le service Service Bus WCF Relay](service-bus-dotnet-how-to-use-relay.md)




<!--HONumber=Nov16_HO2-->


