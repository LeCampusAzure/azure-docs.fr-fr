---
title: "Limitations d’Azure Cloud Shell (version préliminaire) | Microsoft Docs"
description: "Vue d’ensemble des limites d’Azure Cloud Shell"
services: 
documentationcenter: 
author: jluk
manager: timlt
tags: azure-resource-manager
ms.assetid: 
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 08/21/2017
ms.author: juluk
ms.translationtype: HT
ms.sourcegitcommit: 83f19cfdff37ce4bb03eae4d8d69ba3cbcdc42f3
ms.openlocfilehash: e42841b126a9df9240bec3f489589d5ce4a6db80
ms.contentlocale: fr-fr
ms.lasthandoff: 08/21/2017

---

# <a name="limitations-of-azure-cloud-shell"></a>Limitations d’Azure Cloud Shell
Azure Cloud Shell a les limitations connues suivantes :

## <a name="system-state-and-persistence"></a>État du système et persistance
La machine qui fournit votre session Cloud Shell est temporaire. En effet, elle est recyclée lorsque votre session reste inactive pendant 20 minutes. Cloud Shell requiert qu’un partage de fichiers soit monté. Par conséquent, votre abonnement doit être en mesure de configurer des ressources de stockage pour accéder à Cloud Shell. Autres éléments à prendre en compte :
* Avec le stockage monté, seules les modifications apportées à votre répertoire `$Home` ou `clouddrive` sont conservées.
* Les partages de fichiers peuvent être montés uniquement depuis votre [région affectée](persisting-shell-storage.md#mount-a-new-clouddrive).
* Azure Files prend uniquement en charge les comptes de stockage localement redondant et les comptes de stockage géoredondant.

## <a name="user-permissions"></a>Autorisations utilisateur
Les autorisations sont définies en tant qu’utilisateurs standards sans accès sudo. Les installations en dehors du répertoire `$Home` ne sont pas conservées.
Même si certaines commandes situées dans le répertoire `clouddrive` (par exemple, `git clone`) ne disposent pas des autorisations nécessaires, votre répertoire `$Home`, lui, dispose de ces autorisations.

## <a name="browser-support"></a>Prise en charge des navigateurs
Cloud Shell prend en charge les dernières versions de Microsoft Edge, Microsoft Internet Explorer, Google Chrome, Mozilla Firefox et Apple Safari. Safari en mode privé n’est pas pris en charge.

## <a name="copy-and-paste"></a>Copier et coller
Étant donné que les commandes Ctrl + C et Ctrl + V ne fonctionnent pas comme des raccourcis permettant de copier et coller dans Cloud Shell sur les ordinateurs Windows, utilisez les commandes Ctrl + Inser et Maj + Inser pour copier et coller respectivement.

Des options de copier-coller du bouton droit sont également disponibles, mais sont soumises à l’accès au Presse-papiers du navigateur.

## <a name="editing-bashrc"></a>Modifier .bashrc
Faites attention lorsque vous modifiez .bashrc : cela peut entraîner des erreurs inattendues dans Cloud Shell.

## <a name="bashhistory"></a>.bash_history
Votre historique de commandes de l’interpréteur de commandes risque d’être incohérent en raison d’interruptions de sessions Cloud Shell ou de sessions simultanées.

## <a name="usage-limits"></a>Limites d’utilisation
Cloud Shell est destiné aux cas d’usage interactif. De fait, les sessions non interactives longues se terminent sans avertissement.

## <a name="network-connectivity"></a>Connectivité réseau
La latence dans Cloud Shell est soumise à la connectivité Internet locale, Cloud Shell continue d’essayer de traiter les instructions envoyées.

## <a name="next-steps"></a>Étapes suivantes
[Démarrage rapide de Cloud Shell](quickstart.md)

