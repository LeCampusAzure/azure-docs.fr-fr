---
title: "Approvisionner un appareil simulé vers Azure IoT Hub | Microsoft Docs"
description: "Démarrage rapide d’Azure : Créer et approvisionner un appareil simulé à l’aide du service d’approvisionnement d’appareil Azure IoT Hub"
services: iot-dps
keywords: 
author: dsk-2015
ms.author: dkshir
ms.date: 09/05/2017
ms.topic: hero-article
ms.service: iot-dps
documentationcenter: 
manager: timlt
ms.devlang: na
ms.custom: mvc
ms.translationtype: HT
ms.sourcegitcommit: eeed445631885093a8e1799a8a5e1bcc69214fe6
ms.openlocfilehash: d4eeb7a77d6336e241c196e4ad48af52d57af1d4
ms.contentlocale: fr-fr
ms.lasthandoff: 09/07/2017

---

# <a name="create-and-provision-a-simulated-device-using-iot-hub-device-provisioning-service-preview"></a>Créer et approvisionner un appareil simulé à l’aide du service d’approvisionnement d’appareil Azure IoT Hub (préversion)

Ces étapes indiquent comment créer un appareil simulé sur votre ordinateur de développement exécutant le système d’exploitation Windows, comment exécuter le simulateur Windows TPM en tant que [Module de sécurité matériel (HSM)](https://azure.microsoft.com/blog/azure-iot-supports-new-security-hardware-to-strengthen-iot-security/) de l’appareil et comment utiliser l’exemple de code pour connecter cet appareil au service d’approvisionnement d’appareil et à votre IoT hub. 

Veillez à compléter les étapes décrites dans la section relative à la [configuration du service d’approvisionnement d’appareil Azure IoT Hub avec le portail Azure](./quick-setup-auto-provision.md) avant de continuer.

<a id="setupdevbox"></a>
## <a name="prepare-the-development-environment"></a>Préparer l’environnement de développement 

1. Assurez-vous que Visual Studio 2015 ou [Visual Studio 2017](https://www.visualstudio.com/vs/) est installé sur votre ordinateur. 

2. Téléchargez et installez le [système de génération de CMake](https://cmake.org/download/).

3. Assurez-vous que l’élément `git` est installé sur votre machine et est ajouté aux variables d’environnement accessibles à la fenêtre de commande. Consultez la section relative aux [outils clients de Software Freedom Conservancy](https://git-scm.com/download/) pour accéder à la dernière version des outils `git` à installer, qui inclut **Git Bash**, l’application de ligne de commande que vous pouvez utiliser pour interagir avec votre référentiel Git local. 

4. Ouvrez une invite de commandes ou Git Bash. Clonez le référentiel GitHub pour l’exemple de code de simulation d’appareil :
    
    ```cmd/sh
    git clone https://github.com/Azure/azure-iot-sdk-c.git --recursive
    ```

5. Créez un dossier dans votre copie locale de ce référentiel GitHub pour le processus de génération de CMake. 

    ```cmd/sh
    cd azure-iot-device-auth
    mkdir cmake
    cd cmake
    ```

6. L’exemple de code utilise un simulateur Windows TPM. Exécutez la commande suivante pour activer l’authentification de jeton SAS. Elle génère également une solution Visual Studio pour l’appareil simulé.

    ```cmd/sh
    cmake -Ddps_auth_type=tpm_simulator ..
    ```

7. Dans une invite de commandes distincte, accédez au dossier racine GitHub et exécutez le simulateur [TPM](https://docs.microsoft.com/windows/device-security/tpm/trusted-platform-module-overview) . Il écoute un socket sur les ports 2321 et 2322. Ne fermez pas cette fenêtre de commande ; vous devez laisser ce simulateur s’exécuter jusqu’à la fin de ce guide de démarrage rapide. 

    ```cmd/sh
    .\azure-iot-device-auth\dps_client\deps\utpm\tools\tpm_simulator\Simulator.exe
    ```

## <a name="create-a-device-enrollment-entry-in-the-device-provisioning-service"></a>Créer une entrée d’inscription d’appareil dans le service d’approvisionnement d’appareil

1. Ouvrez la solution générée dans le dossier *cmake* nommé `azure_iot_sdks.sln` générez-la dans Visual Studio.

2. Cliquez avec le bouton droit sur le projet **tpm_device_provision** et sélectionnez **Définir comme projet de démarrage**. Exécutez la solution. La fenêtre Sortie affiche la **_paire de clés de type EK (Endorsement Key)_** et **_l’ID d’inscription_** nécessaires à l’inscription de l’appareil. Notez ces valeurs. 

3. Connectez-vous au portail Azure, cliquez sur le bouton **Toutes les ressources** dans le menu de gauche et ouvrez votre service d’approvisionnement d’appareil.

4. Dans le panneau de résumé du service d’approvisionnement d’appareil, sélectionnez **Gérer les inscriptions**. Sélectionnez l’onglet **Inscriptions individuelles** et cliquez sur le bouton **Ajouter** dans la partie supérieure. Sélectionnez **TPM** en tant que *Mécanisme*de l’attestation d’identité et entrez *l’ID d’inscription* et la *paire de clés de type EK (Endorsement Key)* comme exigé par le panneau. Cela fait, cliquez sur le bouton **Enregistrer**. 

    ![Saisir les informations d’inscription d’appareil dans le panneau du portail](./media/quick-create-simulated-device/enter-device-enrollment.png)  

   Lorsque l’inscription aboutit, *l’ID d’inscription* de votre appareil s’affiche dans la liste sous l’onglet *Inscriptions individuelles*. 

<a id="firstbootsequence"></a>
## <a name="simulate-first-boot-sequence-for-the-device"></a>Simuler la première séquence de démarrage de l’appareil

1. Dans le portail Azure, sélectionnez le panneau **Vue d’ensemble** de votre service d’approvisionnement d’appareil et notez les valeurs de **_point de terminaison global de l’appareil_** et **_d’étendue de l’ID_**.

    ![Extraire des informations du point de terminaison DPS à partir du panneau du portail](./media/quick-create-simulated-device/extract-dps-endpoints.png) 

2. Sur votre machine, dans Visual Studio, sélectionnez l’exemple de projet nommé **dps_client_sample** et ouvrez le fichier **dps_client_sample.c**.

3. Assignez la valeur _d’étendue de l’ID_ à la variable `dps_scope_id`. Notez que la variable `dps_uri` a la même valeur que le _point de terminaison global de l’appareil_. 

    ```c
    static const char* dps_uri = "global.azure-devices-provisioning.net";
    static const char* dps_scope_id = "[DPS Id Scope]";
    ```

4. Cliquez avec le bouton droit sur le projet **dps_client_sample** et sélectionnez **Définir comme projet de démarrage**. Exécutez l’exemple. Notez les messages qui simulent le démarrage et la connexion de l’appareil au service d’approvisionnement d’appareil pour obtenir des informations concernant votre IoT Hub. En cas de réussite de l’approvisionnement de votre appareil simulé au IoT Hub lié à votre service d’approvisionnement, l’ID d’appareil s’affiche sur le panneau **Explorateur d’appareils** du concentrateur. 

    ![L’appareil est inscrit avec le hub IoT](./media/quick-create-simulated-device/hub-registration.png) 


## <a name="clean-up-resources"></a>Supprimer des ressources

Si vous envisagez de continuer à manipuler et explorer l’exemple de client d’appareil, ne nettoyez pas les ressources créées lors de ce démarrage rapide. Sinon, procédez aux étapes suivantes pour supprimer toutes les ressources créées lors de ce démarrage rapide.

1. Fermez la fenêtre de sortie de l’exemple de client d’appareil sur votre machine.
1. Fermez la fenêtre du simulateur TPM sur votre machine.
1. Dans le menu de gauche du portail Azure, cliquez sur **Toutes les ressources**, puis sélectionnez votre service d’approvisionnement d’appareil. Dans la partie supérieure du panneau **Toutes les ressources**, cliquez sur **Supprimer**.  
1. À partir du menu de gauche, dans le portail Azure, cliquez sur **Toutes les ressources**, puis sélectionnez votre IoT Hub. Dans la partie supérieure du panneau **Toutes les ressources**, cliquez sur **Supprimer**.  

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un appareil simulé TPM sur votre machine et l’avez approvisionné vers votre IoT Hub à l’aide du service d’approvisionnement d’appareil Azure IoT Hub. Pour en savoir plus sur l’approvisionnement de l’appareil en profondeur, référez-vous au didacticiel relatif à l’installation du service d’approvisionnement d’appareil dans le portail Azure. 

> [!div class="nextstepaction"]
> [Didacticiels relatifs au service d’approvisionnement d’appareil Azure IoT Hub](./tutorial-set-up-cloud.md)

