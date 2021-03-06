---
title: "Créer une application web Node.js et MongoDB dans Azure | Microsoft Docs"
description: "Apprenez comment faire fonctionner une application Node.js dans Azure, avec une connexion à une base de données Cosmos DB, via une chaîne de connexion MongoDB."
services: app-service\web
documentationcenter: nodejs
author: cephalin
manager: erikre
editor: 
ms.assetid: 0b4d7d0e-e984-49a1-a57a-3c0caa955f0e
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 05/04/2017
ms.author: cephalin
ms.custom: mvc
ms.translationtype: Human Translation
ms.sourcegitcommit: f7479260c7c2e10f242b6d8e77170d4abe8634ac
ms.openlocfilehash: 14e399c06b21d9cabab28bdb195b02eb422f2d00
ms.contentlocale: fr-fr
ms.lasthandoff: 06/21/2017

---
# <a name="build-a-nodejs-and-mongodb-web-app-in-azure"></a>Créer une application web Node.js et MongoDB dans Azure

Azure Web Apps offre un service d’hébergement web hautement évolutif appliquant des mises à jour correctives automatiques. Ce didacticiel vous montre comment créer une application web Node.js dans Azure et comment la connecter à une base de données. Lorsque vous aurez terminé, vous disposerez d’une application MEAN (MongoDB, Express, AngularJS et Node.js) exécutée sous [Azure App Service](app-service-web-overview.md). Pour plus de simplicité, l’exemple d’application utilise [l’infrastructure de développement web MEAN.js](http://meanjs.org/).

![Application MEAN.js exécutée dans Azure App Service](./media/app-service-web-tutorial-nodejs-mongodb-app/meanjs-in-azure.png)

Ce que vous allez apprendre :

> [!div class="checklist"]
> * Créer une base de données MongoDB dans Azure
> * Connecter une application Node.js à MongoDB
> * Déploiement de l’application dans Azure
> * Mise à jour du modèle de données et redéploiement de l’application
> * Diffusion des journaux de diagnostic à partir d’Azure
> * Gestion de l’application dans le portail Azure

## <a name="prerequisites"></a>Composants requis

Pour suivre ce didacticiel :

1. [Installez Git](https://git-scm.com/)
1. [Installez Node.js et NPM](https://nodejs.org/)
1. [Installer Gulp.js](http://gulpjs.com/) (requis par [MEAN.js](http://meanjs.org/docs/0.5.x/#getting-started))
1. [Installez et exécutez MongoDB Community Edition](https://docs.mongodb.com/manual/administration/install-community/) 

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si vous choisissez d’installer et d’utiliser l’interface de ligne de commande localement, vous devez exécuter Azure CLI version 2.0 ou une version ultérieure pour poursuivre la procédure décrite dans cet article. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, consultez [Installation d’Azure CLI 2.0]( /cli/azure/install-azure-cli). 

## <a name="test-local-mongodb"></a>Tester la base de données MongoDB locale

Ouvrez la fenêtre du terminal et `cd` dans le répertoire `bin` de votre installation MongoDB. Vous pouvez utilisez cette fenêtre de terminal pour exécuter toutes les commandes de ce didacticiel.

Exécutez `mongo` dans le terminal pour vous connecter à votre serveur MongoDB local.

```bash
mongo
```

Si la connexion est établie, cela signifie que votre base de données MongoDB est déjà en cours d’exécution. Dans le cas contraire, assurez-vous que votre base de données MongoDB locale est démarrée en suivant les étapes de la procédure [d’installation de MongoDB Community Edition](https://docs.mongodb.com/manual/administration/install-community/). Souvent, MongoDB est installé, mais vous devez néanmoins le démarrer en exécutant `mongod`. 

Lorsque vous avez fini de tester votre base de données MongoDB, tapez `Ctrl+C` dans le terminal. 

## <a name="create-local-nodejs-app"></a>Créer une application Node.js locale

Cette étape consiste à configurer le projet Node.js local.

### <a name="clone-the-sample-application"></a>Clonage de l’exemple d’application

Dans la fenêtre de terminal, `cd` vers un répertoire de travail.  

Exécutez la commande suivante pour cloner l’exemple de référentiel. 

```bash
git clone https://github.com/Azure-Samples/meanjs.git
```

Cet exemple de référentiel contient une copie du référentiel [MEAN.js](https://github.com/meanjs/mean). Il est modifié pour s’exécuter sous App Service (pour plus d’informations, consultez le fichier [Lisez-moi](https://github.com/Azure-Samples/meanjs/blob/master/README.md) du référentiel MEAN.js).

### <a name="run-the-application"></a>Exécution de l'application

Exécutez la commande suivante pour installer les packages requis et démarrer l’application.

```bash
cd meanjs
npm install
npm start
```

Lorsque l’application est entièrement chargée, vous obtenez un message similaire à celui-ci :

```
--
MEAN.JS - Development Environment

Environment:     development
Server:          http://0.0.0.0:3000
Database:        mongodb://localhost/mean-dev
App version:     0.5.0
MEAN.JS version: 0.5.0
--
```

Accédez à http://localhost:3000 dans un navigateur. Cliquez sur **S’inscrire** dans le menu supérieur et créez un utilisateur test. 

L’exemple d’application MEAN.js stocke les données utilisateur dans la base de données. Si vous parvenez à créer un utilisateur et à vous connecter, votre application écrit les données dans la base de données MongoDB locale.

![MEAN.js se connecte correctement à MongoDB](./media/app-service-web-tutorial-nodejs-mongodb-app/mongodb-connect-success.png)

Sélectionnez **Admin > Manage Articles** (Administrateur > Gérer les articles) pour ajouter des articles.

Pour arrêter Node.js à tout moment, appuyez sur `Ctrl+C` dans le terminal. 

## <a name="create-production-mongodb"></a>Créer une base de données MongoDB de production

Dans cette étape, vous allez créer une base de données MongoDB dans Azure. Lorsque votre application est déployée sur Azure, elle utilise cette base de données cloud.

Pour MongoDB, ce didacticiel utilise [Azure Cosmos DB](/azure/documentdb/). Cosmos DB prend en charge les connexions client MongoDB.

### <a name="log-in-to-azure"></a>Connexion à Azure

Azure CLI 2.0 vous permettra de créer les ressources nécessaires pour héberger votre application dans Azure. Connectez-vous à votre abonnement Azure avec la commande [az login](/cli/azure/#login) et suivez les instructions à l’écran.

```azurecli-interactive
az login
```   

### <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group#create).

[!INCLUDE [Resource group intro](../../includes/resource-group.md)]

L’exemple suivant crée un groupe de ressources dans la région Europe de l’Ouest.

```azurecli-interactive
az group create --name myResourceGroup --location "West Europe"
```

Utilisez la commande d’interface de ligne de commande Azure [az appservice list-locations](/cli/azure/appservice#list-locations) pour répertorier les emplacements disponibles. 

### <a name="create-a-cosmos-db-account"></a>Création d’un compte Cosmos DB

Créez un compte Cosmos DB à l’aide de la commande [az cosmosdb create](/cli/azure/cosmosdb#create).

Dans la commande suivante, remplacez l’espace réservé *\<nom_cosmosdb>* par un nom unique Cosmos DB. Ce nom est utilisé en tant que point de terminaison Cosmos DB, `https://<cosmosdb_name>.documents.azure.com/`. Pour cette raison, le nom doit être unique sur l’ensemble des comptes Cosmos DB dans Azure. Le nom ne peut contenir que des minuscules, des chiffres, le tiret -) et doit compter entre 3 et 50 caractères.

```azurecli-interactive
az cosmosdb create \
    --name <cosmosdb_name> \
    --resource-group myResourceGroup \
    --kind MongoDB
```

Le paramètre *--kind MongoDB* prend en charge les connexions clientes MongoDB.

Une fois le compte Cosmos DB créé, Azure CLI affiche des informations similaires à celles de l’exemple suivant :

```json
{
  "consistencyPolicy":
  {
    "defaultConsistencyLevel": "Session",
    "maxIntervalInSeconds": 5,
    "maxStalenessPrefix": 100
  },
  "databaseAccountOfferType": "Standard",
  "documentEndpoint": "https://<cosmosdb_name>.documents.azure.com:443/",
  "failoverPolicies": 
  ...
  < Output truncated for readability >
}
```

## <a name="connect-app-to-production-mongodb"></a>Connecter l’application à la production MongoDB

Pendant cette étape, vous connectez votre exemple d’application MEAN.js à la base de données Cosmos DB que vous venez de créer, en utilisant une chaîne de connexion MongoDB. 

### <a name="retrieve-the-database-key"></a>Récupérer la clé de la base de données

Pour se connecter à la base de données Cosmos DB, vous avez besoin de la clé de la base de données. Utilisez la commande [az documentdb list-keys](/cli/azure/cosmosdb#list-keys) pour récupérer la clé primaire.

```azurecli-interactive
az cosmosdb list-keys --name <cosmosdb_name> --resource-group myResourceGroup
```

L’interface de ligne de commande Azure montre des informations semblables à ce qui suit :

```json
{
  "primaryMasterKey": "RS4CmUwzGRASJPMoc0kiEvdnKmxyRILC9BWisAYh3Hq4zBYKr0XQiSE4pqx3UchBeO4QRCzUt1i7w0rOkitoJw==",
  "primaryReadonlyMasterKey": "HvitsjIYz8TwRmIuPEUAALRwqgKOzJUjW22wPL2U8zoMVhGvregBkBk9LdMTxqBgDETSq7obbwZtdeFY7hElTg==",
  "secondaryMasterKey": "Lu9aeZTiXU4PjuuyGBbvS1N9IRG3oegIrIh95U6VOstf9bJiiIpw3IfwSUgQWSEYM3VeEyrhHJ4rn3Ci0vuFqA==",
  "secondaryReadonlyMasterKey": "LpsCicpVZqHRy7qbMgrzbRKjbYCwCKPQRl0QpgReAOxMcggTvxJFA94fTi0oQ7xtxpftTJcXkjTirQ0pT7QFrQ=="
}
```

Copiez la valeur de `primaryMasterKey`. Vous aurez besoin de ces informations dans l’étape suivante.

<a name="devconfig"></a>
### <a name="configure-the-connection-string-in-your-nodejs-application"></a>Configurer la chaîne de connexion dans votre application Node.js

Dans votre référentiel MEAN.js, ouvrez _config/env/production.js_.

Dans l’objet `db`, mettez à jour la valeur de `uri` :

* Remplacez les deux espaces réservés  *\<cosmosdb_name>* par votre nom de base de données Cosmos DB.
* Remplacez l’espace réservé  *\<primary_master_key>* par la clé copiée à l’étape précédente.

Le code suivant montre l’objet `db` :

```javascript
db: {
  uri: 'mongodb://<cosmosdb_name>:<primary_master_key>@<cosmosdb_name>.documents.azure.com:10250/mean?ssl=true&sslverifycertificate=false',
  ...
},
```

L’option `ssl=true` est requise, car [Cosmos DB nécessite SSL](../cosmos-db/connect-mongodb-account.md#connection-string-requirements). 

Enregistrez vos modifications.

### <a name="test-the-application-in-production-mode"></a>Tester l’application en mode de production 

Exécutez la commande suivante pour réduire et regrouper des scripts pour l’environnement de production. Ce processus génère les fichiers nécessaires à l’environnement de production.

```bash
gulp prod
```

Exécutez la commande suivante pour utiliser la chaîne de connexion que vous avez configurée dans _config/env/production.js_.

```bash
NODE_ENV=production node server.js
```

`NODE_ENV=production` définit la variable d’environnement qui indique à Node.js de s’exécuter dans l’environnement de production.  `node server.js` démarre le serveur Node.js avec `server.js` à la racine du référentiel. Voici comment est chargée votre application Node.js dans Azure. 

Lorsque l’application est chargée, assurez-vous qu’elle s’exécute dans l’environnement de production :

```
--
MEAN.JS

Environment:     production
Server:          http://0.0.0.0:8443
Database:        mongodb://<cosmosdb_name>:<primary_master_key>@<cosmosdb_name>.documents.azure.com:10250/mean?ssl=true&sslverifycertificate=false
App version:     0.5.0
MEAN.JS version: 0.5.0
```

Accédez à http://localhost:8443 dans un navigateur. Cliquez sur **S’inscrire** dans le menu supérieur et créez un utilisateur test. Si vous parvenez à créer un utilisateur et à vous connecter, votre application écrit les données dans la base de données Cosmo DB dans Azure. 

Dans le terminal, arrêtez Node.js en tapant `Ctrl+C`. 

## <a name="deploy-app-to-azure"></a>Déployer des applications dans Azure

Dans cette étape, vous allez déployer dans Azure App Service votre application Node.js connectée à MongoDB.

### <a name="create-an-app-service-plan"></a>Créer un plan App Service

Créez un plan App Service avec la commande [az appservice plan create](/cli/azure/appservice/plan#create). 

[!INCLUDE [app-service-plan](../../includes/app-service-plan.md)]

L’exemple suivant crée un plan App Service nommé _myAppServicePlan_ en indiquant le niveau tarifaire **Gratuit** :

```azurecli-interactive
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku FREE
```

Lorsque le plan App Service est créé, l’interface Azure CLI affiche des informations similaires à l’exemple suivant :

```json 
{ 
  "adminSiteName": null,
  "appServicePlanName": "myAppServicePlan",
  "geoRegion": "North Europe",
  "hostingEnvironmentProfile": null,
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Web/serverfarms/myAppServicePlan", 
  "kind": "app",
  "location": "North Europe",
  "maximumNumberOfWorkers": 1,
  "name": "myAppServicePlan",
  ...
  < Output has been truncated for readability >
} 
```

### <a name="create-a-web-app"></a>Créer une application web

Créez une application web dans le plan App Service `myAppServicePlan` avec la commande [az webapp create](/cli/azure/webapp#create). 

L’application web vous offre un espace d’hébergement pour déployer votre code, et fournit une URL pour vous permettre d’afficher l’application déployée. Utilisez  pour créer l’application web. 

Dans la commande suivante, remplacez l’espace réservé *\<app_name>* par le nom unique de votre application. Ce nom est utilisé dans l’URL par défaut de l’application web. Pour cette raison, ce nom doit être unique sur l’ensemble des applications dans Azure App Service. 

```azurecli-interactive
az webapp create --name <app_name> --resource-group myResourceGroup --plan myAppServicePlan
```

Une fois l’application web créée, Azure CLI affiche des informations similaires à celles de l’exemple suivant : 

```json 
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "<app_name>.azurewebsites.net",
  "enabled": true,
  ...
  < Output has been truncated for readability >
}
```

### <a name="configure-an-environment-variable"></a>Configurer une variable d’environnement

Précédemment dans ce didacticiel, vous avez codé en dur la chaîne de connexion de la base de données dans _config/env/prodution.js_. Conformément aux meilleures pratiques de sécurité, vous allez vouloir conserver ces données sensibles en dehors de votre référentiel Git. Pour l’application en cours d’exécution dans Azure, vous utiliserez plutôt une variable d’environnement.

Dans App Service, vous définissez les variables d’environnement en tant que _paramètres d’application_ à l’aide de la commande [az webapp config appsettings update](/cli/azure/webapp/config/appsettings#update). 

L’exemple suivant configure un paramètre d’application `MONGODB_URI` dans votre application web Azure. Remplacez les espaces réservés *\<app_name>*, *\<cosmosdb_name>*, et *\<primary_master_key>*.

```azurecli-interactive
az webapp config appsettings update \
    --name <app_name> \
    --resource-group myResourceGroup \
    --settings MONGODB_URI="mongodb://<cosmosdb_name>:<primary_master_key>@<cosmosdb_name>.documents.azure.com:10250/mean?ssl=true"
```

Dans le code Node.js, vous accédez à ce paramètre d’application avec `process.env.MONGODB_URI`, comme vous accéderiez à n’importe quelle variable d’environnement. 

À présent, annulez vos modifications apportées à _config/env/production.js_ à l’aide de la commande suivante :

```bash
git checkout -- .
```

Ouvrez à nouveau _config/env/production.js_. Notez que l’application MEAN.js par défaut est déjà configurée pour utiliser la variable d’environnement `MONGODB_URI` que vous avez créée.

```javascript
db: {
  uri: ... || process.env.MONGODB_URI || ...,
  ...
},
```

### <a name="configure-local-git-deployment"></a>Configurer le déploiement Git local 

Utilisez la commande [az webapp deployment user set](/cli/azure/webapp/deployment/user#set) pour créer des informations d’identification de déploiement.

Vous pouvez déployer votre application dans Azure App Service de plusieurs façons, notamment FTP, Git local ainsi que GitHub, Visual Studio Team Services et Bitbucket. Pour les sites FTP et Git locaux, il est nécessaire de disposer d’un utilisateur de déploiement configuré sur le serveur pour authentifier votre déploiement. Cet utilisateur de déploiement se situe au niveau des comptes et est différent de votre compte d’abonnement Azure. Vous n’avez à configurer cet utilisateur de déploiement qu’une seule fois.

Dans la commande suivante, remplacez les valeurs *\<user-name>* et *\<password>* par le nom et le mot de passe du nouvel utilisateur. Le nom d’utilisateur doit être unique. Le mot de passe doit comporter au moins huit caractères et inclure deux des trois éléments suivants : lettres, chiffres et symboles. Si vous obtenez une erreur ` 'Conflict'. Details: 409`, modifiez le nom d’utilisateur. Si vous obtenez une erreur ` 'Bad Request'. Details: 400`, utilisez un mot de passe plus fort.

```azurecli-interactive
az appservice web deployment user set --user-name <username> --password <password>
```

Enregistrez le nom d’utilisateur et le mot de passe pour les utiliser dans les étapes ultérieures du déploiement de l’application.

Utilisez la commande [az webapp deployment source config-local-git](/cli/azure/webapp/deployment/source#config-local-git) pour configurer l’accès Git local à l’application web Azure. 

```azurecli-interactive
az webapp deployment source config-local-git --name <app_name> --resource-group myResourceGroup
```

Lorsque l’utilisateur du déploiement est configuré, l’interface Azure CLI affiche l’URL de déploiement pour votre application web Azure au format suivant :

```bash 
https://<username>@<app_name>.scm.azurewebsites.net:443/<app_name>.git 
``` 

Copiez la sortie du terminal, car elle sera utilisée à l’étape suivante. 

### <a name="push-to-azure-from-git"></a>Effectuer une transmission de type push vers Azure à partir de Git

Ajoutez un référentiel distant Azure dans votre référentiel Git local. 

```bash
git remote add azure <paste_copied_url_here> 
```

Effectuez une transmission de type push vers le référentiel distant Azure pour déployer votre application Node.js. Le mot de passe que vous avez fourni précédemment dans le cadre de la création de l’utilisateur du déploiement vous sera demandé. 

```bash
git push azure master
```

Au cours du déploiement, Azure App Service communique sa progression avec Git.

```bash
Counting objects: 5, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (5/5), 489 bytes | 0 bytes/s, done.
Total 5 (delta 3), reused 0 (delta 0)
remote: Updating branch 'master'.
remote: Updating submodules.
remote: Preparing deployment for commit id '6c7c716eee'.
remote: Running custom deployment command...
remote: Running deployment command...
remote: Handling node.js deployment.
.
.
.
remote: Deployment successful.
To https://<app_name>.scm.azurewebsites.net/<app_name>.git
 * [new branch]      master -> master
``` 

Vous remarquerez peut-être que le processus de déploiement exécute [Gulp](http://gulpjs.com/) après `npm install`. App Service n’exécute pas les tâches Gulp ou Grunt pendant le déploiement ; cet exemple de référentiel possède donc deux fichiers supplémentaires dans son répertoire racine pour l’activer : 

- _.deployment_ : ce fichier indique à App Service d’exécuter `bash deploy.sh` en tant que script de déploiement personnalisé.
- _deploy.sh_ : le script de déploiement personnalisé. Si vous examinez le fichier, vous verrez qu’il exécute `gulp prod` après `npm install` et `bower install`. 

Vous pouvez utiliser cette approche pour ajouter une étape à votre déploiement Git. Notez que si vous redémarrez votre application web Azure à tout moment, App Service ne réexécute pas ces tâches d’automatisation.

### <a name="browse-to-the-azure-web-app"></a>Rechercher l’application web Azure 

Accédez à l’application web déployée à l’aide de votre navigateur web. 

```bash 
http://<app_name>.azurewebsites.net 
``` 

Cliquez sur **S’inscrire** dans le menu supérieur et créez un utilisateur fictif. 

Si vous réussissez, et si l’application se connecte automatiquement à l’utilisateur créé, cela signifie que votre application MEAN.js dans Azure dispose d’une connectivité à la base de données MongoDB (Cosmos DB). 

![Application MEAN.js exécutée dans Azure App Service](./media/app-service-web-tutorial-nodejs-mongodb-app/meanjs-in-azure.png)

Sélectionnez **Admin > Manage Articles** (Administrateur > Gérer les articles) pour ajouter des articles. 

**Félicitations !** Vous exécutez une application Node.js orientée données dans Azure App Service.

## <a name="update-data-model-and-redeploy"></a>Mettre à jour le modèle de données et redéployer

Dans cette étape, vous allez changer le modèle de données `article` et publier vos modifications dans Azure.

### <a name="update-the-data-model"></a>Mettre à jour le modèle de données

Ouvrez _modules/articles/server/models/article.server.model.js_.

Dans `ArticleSchema`, ajoutez un type `String` appelé `comment`. Lorsque vous aurez terminé, votre code de schéma doit ressembler à ceci :

```javascript
var ArticleSchema = new Schema({
  ...,
  user: {
    type: Schema.ObjectId,
    ref: 'User'
  },
  comment: {
    type: String,
    default: '',
    trim: true
  }
});
```

### <a name="update-the-articles-code"></a>Mettre à jour le code d’articles

Mettez à jour le reste de votre code `articles` afin d’utiliser `comment`.

Vous avez cinq fichiers à modifier : le contrôleur de serveur et les quatre vues clientes. 

Ouvrez _modules/articles/server/controllers/articles.server.controller.js_.

Dans la fonction `update`, ajoutez une affectation de `article.comment`. Le code suivant illustre la fonction `update` achevée :

```javascript
exports.update = function (req, res) {
  var article = req.article;

  article.title = req.body.title;
  article.content = req.body.content;
  article.comment = req.body.comment;

  ...
};
```

Ouvrez _modules/articles/client/views/view-article.client.view.html_.

Juste au-dessus de la balise de fermeture `</section>`, ajoutez la ligne suivante pour afficher `comment` avec le reste des données de l’article :

```HTML
<p class="lead" ng-bind="vm.article.comment"></p>
```

Ouvrez _modules/articles/client/views/list-articles.client.view.html_.

Juste au-dessus de la balise de fermeture `</a>`, ajoutez la ligne suivante pour afficher `comment` avec le reste des données de l’article :

```HTML
<p class="list-group-item-text" ng-bind="article.comment"></p>
```

Ouvrez _modules/articles/client/views/admin/list-articles.client.view.html_.

À l’intérieur de l’élément `<div class="list-group">`, juste au-dessus de la balise de fermeture `</a>`, ajoutez la ligne suivante pour afficher `comment` avec le reste des données de l’article :

```HTML
<p class="list-group-item-text" data-ng-bind="article.comment"></p>
```

Ouvrez _modules/articles/client/views/admin/form-article.client.view.html_.

Recherchez l’élément `<div class="form-group">` qui contient le bouton Envoyer, qui ressemble à ceci :

```HTML
<div class="form-group">
  <button type="submit" class="btn btn-default">{{vm.article._id ? 'Update' : 'Create'}}</button>
</div>
```

Juste au-dessus de cette balise, ajoutez un autre élément `<div class="form-group">` permettant aux utilisateurs de modifier le champ `comment`. Votre nouvel élément doit avoir l’aspect suivant :

```HTML
<div class="form-group">
  <label class="control-label" for="comment">Comment</label>
  <textarea name="comment" data-ng-model="vm.article.comment" id="comment" class="form-control" cols="30" rows="10" placeholder="Comment"></textarea>
</div>
```

### <a name="test-your-changes-locally"></a>Tester vos modifications en local

Enregistrez toutes vos modifications.

Testez de nouveau vos modifications en mode de production.

```bash
gulp prod
NODE_ENV=production node server.js
```

> [!NOTE]
> N’oubliez pas que votre _config/env/production.js_ a été rétabli et que la variable d’environnement `MONGODB_URI` est définie uniquement dans votre application web Azure, et non sur votre ordinateur local. Si vous examinez le fichier de configuration, vous verrez que la configuration de production utilise par défaut une base de données MongoDB. Cela vous évite de toucher aux données de production lorsque vous testez vos changements de code en local.

Accédez à `http://localhost:8443` dans un navigateur et assurez-vous que vous êtes connecté.

Sélectionnez **Admin > Manage Articles** (Administrateur > Gérer les articles), puis ajoutez un article en cliquant sur le bouton **+**.

Vous voyez maintenant la nouvelle zone de texte `Comment`.

![Champ de commentaire ajouté aux Articles](./media/app-service-web-tutorial-nodejs-mongodb-app/added-comment-field.png)

Dans le terminal, arrêtez Node.js en tapant `Ctrl+C`. 

### <a name="publish-changes-to-azure"></a>Publier les modifications dans Azure

Validez vos modifications dans Git, puis envoyez les modifications de code à Azure.

```bash
git commit -am "added article comment"
git push azure master
```

Une fois le `git push` terminé, accédez à votre application web Azure et essayez la nouvelle fonctionnalité.

![Modifications du modèle et de la base de données publiées dans Azure](media/app-service-web-tutorial-nodejs-mongodb-app/added-comment-field-published.png)

Si vous avez ajouté des articles précédemment, vous pouvez toujours les visualiser. Les données existantes dans votre Cosmos DB ne sont pas perdues. De plus, les mises à jour que vous appliquez au schéma de données n’affectent pas vos données existantes.

## <a name="stream-diagnostic-logs"></a>Diffuser les journaux de diagnostic 

Pendant l’exécution de votre application Node.js dans Azure App Service, vous pouvez acheminer les journaux de la console vers votre terminal. De cette façon, vous pouvez obtenir les mêmes messages de diagnostic pour vous aider à déboguer les erreurs d’application.

Pour démarrer la diffusion de journaux, utilisez la commande [az webapp log tail](/cli/azure/webapp/log#tail).

```azurecli-interactive
az webapp log tail --name <app_name> --resource-group myResourceGroup
``` 

Une fois que la diffusion a démarré, actualisez votre application web Azure dans le navigateur pour générer un trafic web. Vous voyez maintenant les journaux de la console acheminés vers votre terminal.

Pour arrêter la diffusion de journaux à tout moment, tapez `Ctrl+C`. 

## <a name="manage-your-azure-web-app"></a>Gérer votre application web Azure

Accédez au [portail Azure](https://portal.azure.com) pour voir l’application web que vous avez créée.

Dans le menu de gauche, cliquez sur **App Services**, puis cliquez sur le nom de votre application web Azure.

![Navigation au sein du portail pour accéder à l’application web Azure](./media/app-service-web-tutorial-nodejs-mongodb-app/access-portal.png)

Par défaut, le portail affiche la page **Vue d’ensemble** de l’application web. Cette page propose un aperçu de votre application. Ici, vous pouvez également effectuer des tâches de gestion de base (parcourir, arrêter, démarrer, redémarrer et supprimer des éléments, par exemple). Les onglets figurant sur le côté gauche de la page affichent les différentes pages de configuration que vous pouvez ouvrir.

![Page App Service du Portail Azure](./media/app-service-web-tutorial-nodejs-mongodb-app/web-app-blade.png)

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

<a name="next"></a>
## <a name="next-steps"></a>Étapes suivantes

Vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer une base de données MongoDB dans Azure
> * Connecter une application Node.js à MongoDB
> * Déploiement de l’application dans Azure
> * Mettre à jour le modèle de données et redéployer l’application
> * Diffuser des journaux à partir d’Azure vers votre terminal
> * Gérer l’application dans le portail Azure

Passez au didacticiel suivant pour découvrir comment mapper un nom DNS personnalisé à votre application web.

> [!div class="nextstepaction"] 
> [Mapper un nom DNS personnalisé existant à des applications web Azure](app-service-web-tutorial-custom-domain.md)

