---
title: "Réplication de données dans le stockage Azure | Microsoft Docs"
description: "Les données de votre compte Stockage Microsoft Azure sont répliquées à des fins de durabilité et de haute disponibilité. Les options de réplication incluent le stockage localement redondant (LRS), le stockage redondant dans une zone (ZRS), le stockage géo-redondant (GRS) et le stockage géo-redondant avec accès en lecture (RA-GRS)."
services: storage
documentationcenter: 
author: mmacy
manager: timlt
editor: tysonn
ms.assetid: 86bdb6d4-da59-4337-8375-2527b6bdf73f
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/15/2017
ms.author: marsma
ms.translationtype: HT
ms.sourcegitcommit: 83f19cfdff37ce4bb03eae4d8d69ba3cbcdc42f3
ms.openlocfilehash: c35f53c5c80a67cb33b457e9e3235d0e745536cf
ms.contentlocale: fr-fr
ms.lasthandoff: 08/21/2017

---
# <a name="azure-storage-replication"></a>Réplication Azure Storage

Les données de votre compte de stockage Microsoft Azure sont toujours répliquées pour en garantir la durabilité et la haute disponibilité. La réplication copie vos données dans le même centre de données ou dans un second centre de données, selon l’option de réplication que vous choisissez. La réplication protège vos données et maintient votre application en bon état de fonctionnement en cas de panne matérielle temporaire. Si vos données sont répliquées vers un deuxième centre de données, elles sont protégées en cas de défaillance irrémédiable de l’emplacement primaire.

La réplication garantit que votre compte de stockage répond aux exigences du [contrat de niveau de service (SLA) pour le stockage](https://azure.microsoft.com/support/legal/sla/storage/) même en cas de panne. Pour plus d’informations sur les garanties de durabilité et de disponibilité du stockage Azure, consultez le contrat de niveau de service.

Lorsque vous créez un compte de stockage, vous pouvez sélectionner une des options de réplication suivantes :

* [Stockage localement redondant (LRS)](#locally-redundant-storage)
* [Stockage redondant dans une zone (ZRS)](#zone-redundant-storage)
* [Stockage géo-redondant (GRS)](#geo-redundant-storage)
* [Stockage géo-redondant avec accès en lecture (RA-GRS)](#read-access-geo-redundant-storage)

Le stockage géoredondant avec accès en lecture (RA-GRS) est l’option par défaut lorsque vous créez un compte de stockage.

Le tableau suivant fournit une vue d’ensemble rapide des différences entre LRS, ZRS, GRS et RA-GRS, tandis que les sections suivantes décrivent chaque type de réplication plus en détail.

| Stratégie de réplication | LRS | ZRS | GRS | RA-GRS |
|:--- |:--- |:--- |:--- |:--- |
| Les données sont répliquées entre plusieurs centres de données. |Non |Oui |Oui |Oui |
| Les données peuvent être lues tant à partir d’un emplacement secondaire que de l’emplacement principal. |Non |Non |Non |Oui |
| Nombre de copies de données conservées sur des nœuds distincts. |3 |3 |6 |6 |

Consultez [Tarification Azure Storage](https://azure.microsoft.com/pricing/details/storage/) pour connaître les informations de tarification des différentes options de redondance.

> [!NOTE]
> Stockage Premium prend en charge uniquement un stockage localement redondant (LRS). Pour plus d’informations sur Stockage Premium, consultez [Stockage Premium : stockage hautes performances pour les charges de travail des machines virtuelles Azure](../storage-premium-storage.md).
>

## <a name="locally-redundant-storage"></a>Stockage localement redondant
Le stockage localement redondant (LRS) réplique vos données trois fois à l’intérieur d’une unité d’échelle de stockage hébergée dans un centre de données situé dans la région dans laquelle vous avez créé votre compte de stockage. Une demande d’écriture est retournée avec succès uniquement après son écriture dans les trois réplicas. Ces trois réplicas se trouvent dans des domaines d’erreur et de mise à niveau distincts dans une unité d’échelle de stockage.

Une unité d’échelle de stockage est un ensemble de racks de nœuds de stockage. Un domaine d’erreur est un groupe de nœuds qui représentent une unité physique d’incident et peuvent être considérés comme des nœuds appartenant au même rack physique. Un domaine de mise à niveau est un groupe de nœuds qui sont mis à jour ensemble au cours d’un processus de mise à niveau de service (déploiement). Les trois réplicas sont répartis sur des domaines d’erreur et de mise à jour au sein d’une unité d’échelle de stockage afin de garantir que les données sont disponibles même en cas de défaillance matérielle d’un rack ou lorsque les nœuds sont mis à niveau pendant un déploiement.

Le stockage LRS est l’option la moins coûteuse et offre une durabilité moindre par rapport aux autres options. En cas d’incident au niveau du centre de données (incendie, inondation, etc.) les trois réplicas peuvent être perdus ou irrécupérables. Pour atténuer ce risque, un stockage géoredondant (GRS) est recommandé pour la plupart des applications.

Le stockage localement redondant peut toujours être adapté dans certains scénarios :

* Fournit la bande passante maximale la plus élevée de toutes les options de réplication Stockage Azure.
* Si votre application stocke des données qui peuvent être recréées facilement, vous pouvez opter pour un stockage LRS.
* Certaines applications sont limitées à la réplication des données dans un pays en raison des exigences de gouvernance des données. Une région jumelée peut être située dans un autre pays. Pour plus d’informations sur les régions jumelées, voir [Régions Azure](https://azure.microsoft.com/regions/).

## <a name="zone-redundant-storage"></a>Stockage redondant dans une zone
Le stockage redondant dans une zone (ZRS) réplique vos données de façon asynchrone entre plusieurs centres de données situés dans une ou deux régions, et stocke trois réplicas, comme c’est le cas pour le stockage LRS. Cela fournit donc une durabilité supérieure à celle du stockage LRS. Les données stockées par stockage ZRS sont durables même si le centre de données principal est indisponible ou irrécupérable.
Les clients qui envisagent d’utiliser le stockage ZRS doivent tenir compte des points suivants :

* Le stockage ZRS n’est disponible que pour les objets blob de blocs dans les comptes de stockage à usage général et est pris en charge uniquement dans les versions de service de stockage 2014-02-14 et ultérieures.
* Étant donné que la réplication asynchrone implique un délai, il est possible que, en cas de sinistre local, les modifications n’ayant pas encore été répliquées dans la région secondaire soient perdues, si les données ne peuvent pas être récupérées à partir de la région primaire.
* Il est possible que le réplica ne soit disponible que si Microsoft lance le basculement vers la région secondaire.
* Les comptes de stockage ZRS ne peuvent pas être convertis ultérieurement en stockage LRS ou GRS. De même, un compte LRS ou GRS ne peut pas être converti en un compte ZRS.
* Les comptes ZRS n’ont pas de métriques ni de fonctionnalités de journalisation.

## <a name="geo-redundant-storage"></a>Stockage géo-redondant
Le stockage géo-redondant (GRS) réplique vos données vers une région secondaire à des centaines de kilomètres de la région primaire. Si le GRS est activé pour votre compte de stockage, vos données restent durables, même en cas de panne régionale totale ou d’incident empêchant la récupération depuis la région primaire.

Pour un compte de stockage avec GRS activé, une mise à jour est d’abord appliquée dans la région primaire, où elle est répliquée trois fois. La mise à jour est ensuite répliquée trois fois également de manière asynchrone dans la région secondaire.

Avec le stockage GRS, les régions primaire et secondaire gèrent les réplicas sur plusieurs domaines d’erreur et domaines de mise à niveau distincts, au sein d’une unité d’échelle de stockage comme décrit pour le stockage LRS.

Considérations :

* Étant donné que la réplication asynchrone implique un délai, il est possible que, en cas de sinistre régional, les modifications n’ayant pas encore été répliquées dans la région secondaire soient perdues, si les données ne peuvent pas être récupérées à partir de la région primaire.
* Le réplica n’est disponible que si Microsoft lance le basculement vers la région secondaire. Si Microsoft initie un basculement vers la région secondaire, vous obtiendrez un accès en lecture et écriture à ces données une fois le basculement effectué. Pour plus d’informations, consultez [Conseils sur la récupération d’urgence](../storage-disaster-recovery-guidance.md). 
* Si une application souhaite lire du contenu à partir de la région secondaire, l’utilisateur doit activer le stockage RA-GRS.

Lorsque vous créez un compte de stockage, vous sélectionnez la région primaire pour le compte. La région secondaire est déterminée en fonction de la région primaire et ne peut pas être modifiée. Le tableau suivant montre les paires de régions primaires et secondaires.

| Primaire | Secondaire |
| --- | --- |
| États-Unis - partie centrale septentrionale |Centre-Sud des États-Unis |
| Centre-Sud des États-Unis |États-Unis - partie centrale septentrionale |
| Est des États-Unis |Ouest des États-Unis |
| Ouest des États-Unis |Est des États-Unis |
| Est des États-Unis 2 |Centre des États-Unis |
| Centre des États-Unis |Est des États-Unis 2 |
| Europe du Nord |Europe de l'Ouest |
| Europe de l'Ouest |Europe du Nord |
| Asie du Sud-Est |Est de l'Asie |
| Est de l'Asie |Asie du Sud-Est |
| Chine orientale |Chine du Nord |
| Chine du Nord |Chine orientale |
| Est du Japon |Ouest du Japon |
| Ouest du Japon |Est du Japon |
| Sud du Brésil |Centre-Sud des États-Unis |
| Est de l’Australie |Sud-est de l’Australie |
| Sud-est de l’Australie |Est de l’Australie |
| Sud de l'Inde |Inde-Centre |
| Inde-Centre |Sud de l'Inde |
| Inde-Ouest |Sud de l'Inde |
| Gouvernement américain - Iowa |Gouvernement américain - Virginie |
| Gouvernement américain - Virginie |Gouvernement des États-Unis – Texas |
| Gouvernement des États-Unis – Texas |Gouvernement des États-Unis – Arizona |
| Gouvernement des États-Unis – Arizona |Gouvernement des États-Unis – Texas |
| Centre du Canada |Est du Canada |
| Est du Canada |Centre du Canada |
| Ouest du Royaume-Uni |Sud du Royaume-Uni |
| Sud du Royaume-Uni |Ouest du Royaume-Uni |
| Centre de l’Allemagne |Nord-Est de l’Allemagne |
| Nord-Est de l’Allemagne |Centre de l’Allemagne |
| Ouest des États-Unis 2 |Centre-Ouest des États-Unis |
| Centre-Ouest des États-Unis |Ouest des États-Unis 2 |

Pour obtenir des informations à jour sur les régions prises en charge par Azure, voir [Régions Azure](https://azure.microsoft.com/regions/).

>[!NOTE]  
> La région secondaire du Gouvernement des États-Unis - Virginie est le Gouvernement des États-Unis - Texas. Auparavant, il s’agissait du Gouvernement des États-Unis - Iowa. Les comptes de stockage utilisant toujours la région secondaire Gouvernement des États-Unis - Iowa sont en cours de migration vers la région secondaire Gouvernement des États-Unis - Texas. 
> 
> 

## <a name="read-access-geo-redundant-storage"></a>Stockage géo-redondant avec accès en lecture
Le stockage géo-redondant avec accès en lecture (RA-GRS) optimise la disponibilité de votre compte de stockage, en fournissant un accès en lecture seule aux données dans l’emplacement secondaire, en plus de la réplication entre les deux régions offerte par GRS.

Lorsque vous activez l’accès en lecture seule à vos données dans la région secondaire, vos données sont disponibles sur un point de terminaison secondaire, en plus du point de terminaison primaire pour votre compte de stockage. Le point de terminaison secondaire est similaire au point de terminaison principal, mais ajoute le suffixe `–secondary` au nom du compte. Par exemple, si votre point de terminaison principal pour le service d’objets blob est `myaccount.blob.core.windows.net`, votre point de terminaison secondaire est `myaccount-secondary.blob.core.windows.net`. Les clés d’accès pour votre compte de stockage sont les mêmes pour le point de terminaison primaire et secondaire.

Considérations :

* Votre application doit gérer le point de terminaison avec lequel l’interaction se fait lors de l’utilisation du stockage RA-GRS.
* Étant donné que la réplication asynchrone implique un délai, il est possible que, en cas de sinistre régional, les modifications n’ayant pas encore été répliquées dans la région secondaire soient perdues, si les données ne peuvent pas être récupérées à partir de la région primaire.
* Si Microsoft initie un basculement vers la région secondaire, vous obtiendrez un accès en lecture et écriture à ces données une fois le basculement effectué. Pour plus d’informations, consultez [Conseils sur la récupération d’urgence](../storage-disaster-recovery-guidance.md). 
* Le stockage RA-GRS est destiné à des fins de haute disponibilité. Pour obtenir des conseils sur l’évolutivité, veuillez consulter la [liste de vérification de performances](../storage-performance-checklist.md).

## <a name="frequently-asked-questions"></a>Forum Aux Questions

<a id="howtochange"></a>
#### <a name="1-how-can-i-change-the-geo-replication-type-of-my-storage-account"></a>1. Comment modifier le type de géoréplication de mon compte de stockage ?

   Pour modifier le type de géoréplication de votre compte de stockage entre LRS, GRS et RA-GRS, vous pouvez utiliser le [portail Azure](https://portal.azure.com/), [Azure PowerShell](storage-powershell-guide-full.md) ou encore utiliser par programme l’une de nos nombreuses bibliothèques clientes de stockage. Notez que les comptes de stockage ZRS ne peuvent pas être convertis en comptes de stockage LRS ni GRS. De même, un compte LRS ou GRS ne peut pas être converti en un compte ZRS.

<a id="changedowntime"></a>
#### <a name="2-will-there-be-any-down-time-if-i-change-the-replication-type-of-my-storage-account"></a>2. Si je modifie le type de réplication de mon compte de stockage, le système connaît-il un temps d’arrêt ?

   Non, il n’y a pas de temps d’arrêt.

<a id="changecost"></a>
#### <a name="3-will-there-be-any-additional-cost-if-i-change-the-replication-type-of-my-storage-account"></a>3. Si je modifie le type de réplication de mon compte de stockage, des coûts supplémentaires sont-ils ajoutés ?

   Oui. Si vous remplacez votre compte de stockage LRS par un compte GRS (ou RA-GRS), cette opération induit des frais supplémentaires pour la sortie impliquée par la copie des données existantes de l’emplacement principal vers l’emplacement secondaire. Une fois les données initiales copiées, il n’existe aucuns frais d’entrée supplémentaires pour la géoréplication des données de l’emplacement principal vers l’emplacement secondaire. Vous trouverez les détails sur les frais liés à la bande passante dans la [page de tarification du stockage Azure](https://azure.microsoft.com/pricing/details/storage/blobs/). Si vous remplacez un compte de stockage GRS par un compte LRS, il n’existe aucun coût supplémentaire, mais vos données sont supprimées de l’emplacement secondaire.

<a id="ragrsbenefits"></a>
#### <a name="4-how-can-ra-grs-help-me"></a>4. En quoi un stockage RA-GRS peut-il m’aider ?
   
   Le stockage GRS assure la réplication de vos données d’une région primaire vers une région secondaire distante de centaines de kilomètres de la région primaire. Dans une telle situation, vos données restent durables, même en cas de panne régionale totale ou d’incident empêchant la récupération de la région primaire. Le stockage RA-GRS dispose de cette fonctionnalité, et permet par ailleurs de lire les données de l’emplacement secondaire. Pour obtenir quelques idées sur la façon de tirer parti de cette capacité, voir [Conception d’applications hautement disponibles à l’aide du stockage RA-GRS](../storage-designing-ha-apps-with-ragrs.md). 

<a id="lastsynctime"></a>
#### <a name="5-is-there-a-way-for-me-to-figure-out-how-long-it-takes-to-replicate-my-data-from-the-primary-to-the-secondary-region"></a>5. Existe-t-il un moyen me permettant de déterminer la durée de la réplication de mes données de la région primaire vers la région secondaire ?
   
   Si vous utilisez un stockage RA-GRS, vous pouvez vérifier l’heure de la dernière synchronisation de votre compte de stockage. Il s’agit d’une valeur de date/heure (GMT) ; toutes les écritures principales effectuées avant l’heure de la dernière synchronisation sont correctement écrites à l’emplacement secondaire, ce qui signifie qu’elles peuvent être lues à partir de l’emplacement secondaire. Les écritures principales effectuées après l’heure de la dernière synchronisation peuvent ou non être encore disponibles pour les lectures. Vous pouvez interroger cette valeur à l’aide du [portail Azure](https://portal.azure.com/), [d’Azure PowerShell](storage-powershell-guide-full.md) ou encore par programme à l’aide de l’API REST ou de l’une des bibliothèques clientes de stockage. 

<a id="outage"></a>
#### <a name="6-how-can-i-switch-to-the-secondary-region-if-there-is-an-outage-in-the-primary-region"></a>6. Comment basculer vers la région secondaire en cas de panne dans la région primaire ?
   
   Pour plus d’informations, reportez-vous à l’article [Que faire en cas de panne du stockage Azure](../storage-disaster-recovery-guidance.md).

<a id="rpo-rto"></a>
#### <a name="7-what-is-the-rpo-and-rto-with-grs"></a>7. Que sont l’objectif de point de récupération (RPO) et l’objectif de délai de récupération (RTO) avec GRS ?
   
   Objectif de point de récupération (RPO, Recovery Point Objective) : dans les stockages GRS et RA-GRS, le service de stockage effectue de façon asynchrone la géoréplication des données de l’emplacement principal vers l’emplacement secondaire. S’il se produit un incident régional majeur et qu’un basculement doit être effectué, les dernières modifications delta n’ayant pas été géorépliquées risquent d’être perdues. Le nombre de minutes de perte de données potentielle correspond au point RPO (en d’autres termes, il s’agit du point dans le temps auquel les données peuvent être récupérées). En général, notre point RPO est inférieur à 15 minutes, même s’il n’existe actuellement aucun contrat de niveau de service sur la durée de la géoréplication.

   Objectif de délai de récupération (RTO, Recovery Time Objective) : il s’agit d’une mesure de la durée qui nous est nécessaire pour effectuer le basculement et replacer le compte de stockage en ligne si nous devons procéder à un basculement. Le temps nécessaire à l’exécution du basculement inclut les éléments suivants :
    * temps qui nous est nécessaire pour examiner et déterminer si nous pouvons récupérer les données à l’emplacement principal ou si nous devons effectuer un basculement ;
    * Basculement du compte en modifiant les entrées DNS principales pour pointer vers l’emplacement secondaire.

   Nous prenons très au sérieux la responsabilité de conserver vos données. S’il existe la moindre chance de les récupérer, nous repousserons donc le basculement pour nous concentrer sur la récupération des données de l’emplacement principal. À l’avenir, nous envisageons de fournir une API vous permettant de déclencher un basculement au niveau du compte, ce qui vous permettrait de contrôler le point RTO vous-même. Cette fonctionnalité n’est pas encore disponible.
   
## <a name="next-steps"></a>Étapes suivantes
* [Conception d’applications hautement disponibles à l’aide du stockage RA-GRS](../storage-designing-ha-apps-with-ragrs.md)
* [Tarification du stockage Azure](https://azure.microsoft.com/pricing/details/storage/)
* [À propos des comptes de stockage Azure](../storage-create-storage-account.md)
* [Objectifs de performance et d’extensibilité du Stockage Azure](storage-scalability-targets.md)
* [Options de redondance et stockage géo-redondant avec accès en lecture de Stockage Microsoft Azure ](http://blogs.msdn.com/b/windowsazurestorage/archive/2013/12/11/introducing-read-access-geo-replicated-storage-ra-grs-for-windows-azure-storage.aspx)
* [Document SOSP - Stockage Azure : service de stockage cloud à haute disponibilité et à cohérence forte](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx)


