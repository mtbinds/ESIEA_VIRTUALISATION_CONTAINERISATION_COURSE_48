# LAB - Docker images

Dans ce LAB, vous vous familiariserez avec les ***Dockerfile*** : un outil vous permettant de `packager` vos applications dans des images docker réutilisables.

Grâce aux `Dockerfiles`, vous serez en mesure de figer votre application à un instant T, dans une image docker qui vous servira par la suite à créer des containers.

Jusqu'ici, vous avez appris à utiliser les différentes ressources `docker`, pour créer des environnements en adressant les différentes problématiques de :

- Configuration (des containers)
- Stockage (volume)
- Réseau Network

Tout cela, en utilisant des `images docker` proposées par d'autres utilisateurs (`nginx`, `node:alpine`, `madjidtaoualit/uno`). Maintenant, il est temps d'apprendre à créer vos propres images.

L'objectif de ce LAB est de vous apprendre à utiliser le fichier `Dockerfile` pour construire des images docker à partir d'une application, vous permettant ainsi de porter cette application en tout simplicité vers de multiples environnements.

Ainsi, le jour où vous serez amené face à des problématiques de `mise en production` d'une application, vous aurez les cartes en main pour livrer votre produit sous la forme la plus `portable` possible : Une image Docker. Et ainsi potentiellement la déployer sur de l'infrastructure, avec les connaissances que vous avez accumulées jusqu'à présent sur Docker.

**Différents exemples sont disponibles ici**:

- [Application Golang](./examples/go)
- [Application Python](./examples/python)

## Exercice: Partie 1

Pour ce LAB:

- [Utilisez le Dockerfile ici](./Dockerfile)
- L'application utilisée pour créer l'image docker [se trouve ici](./app): Simplement une ***API Back-end*** `express.js` (node.js)

Ici, les objectifs à atteindre à travers l'exercice:

- Votre nouvelle image (***Dockerfile***) devra étendre l'image `node:alpine`
- L'image doit spécifier un dossier courant `/home/node`
- Uniquement les **dépendances de production** devront être installées dans votre image
- L'ensemble du [code présent dans le dossier app](./app) doit être présent dans le dossier `/home/node` du container
- Par défaut, la commande exécutée au démarrage d'un container doit être `npm run start`

Une fois ces critères remplis dans votre fichier Dockerfile, pour vérifier que tout fonctionne correctement:

- Buildez l'image docker:

  ```bash
  # Il faut être dans le même dossier que Dockerfile
  docker build -t lab-images:latest .
  ```
  Le build doit se faire sans erreur.

- Testez votre image avec la commande suivante:
  
  ```bash
  docker run -p 3000:3000 lab-images:latest
  ```
  Le serveur node.js devrait démarrer sans problème **Vérifier ensuite dans votre navigateur que le résultat est le suivant**:

  ![express website](./assets/express.png)

- L'application node utilisée dans ce LAB utilise la variable d'environnement `PORT`, pour configurer le serveur Web. Testez à nouveau avec la commande suivante (en modifiant la variable PORT, pour s'assurer que l'image buildée est bien sensible à ce paramètre):
  ```bash
  docker run --rm -e PORT=2000 -p 3000:2000 lab-images:latest
  ```
  Ouvrez votre navigateur sur http://localhost:3000 à nouveau, vous devriez observer le même résultat que précédemment.

## Exercice: Partie 2

Dans cette seconde partie, vous allez construire l'image `madjidtaoualit/uno`.

Utiliser la commande `npm run build` pour générer les fichiers statiques (`html`, `js`, `png`, `css`) dans le dossier `build`.

Pour cet exercice, vous aurez besoins d'utiliser deux `stages` dans votre Dockerfile:

- Une première (étape), basée sur l'image `node:alpine`, car le site est développé avec des technologies `node.js`, il va falloir dans un premier temps:

  - Installer les dépendances avec la commande `npm install`
  - Builder les dépendances avec la commande `npm run build`. (**Le dossier `build` sera créer suite à cette commande, contenant les fichiers statiques)**.

Pour rappel, pour déclarer deux étapes dans un `Dockerfile`:

```Dockerfile
# Exemple: première étape, on build un application en go
# Le "as build" va nous servir pour copier le binaire produit dans la seconde étape
FROM golang:alpine AS build

WORKDIR /home/app

COPY . .

# On build l'application go, et on place le binaire produit à /home/application, pour le réutiliser plus tard dans la seconde étape
RUN go build -o /home/application 

# On donne le droit d'exécuter le binaire produit
RUN chmod +x /home/application

# Seconde étape - pour exécuter le binaire, on souhaite utiliser un environnement alpine léger
FROM alpine

WORKDIR /home/app

# On copie le binaire produit dans la première phase grâce au flag --from
# Ici, on place le binaire dans notre image finale à l'emplacement /home/app/application
COPY --from=build /home/application ./application

# On déclare que la commande à utiliser est le déclenchement du binaire produit, à présent copier à l'emplacement /home/app/application
CMD ["/home/app/application"]
```

J'ai décidé d'utiliser un exemple d'une application en `golang` pour ne pas vous spoiler le fonctionnement avec `node` pour cet exercice, mais vous pouvez facilement adapter le fonctionnement de ce Dockerfile aux besoins de cet exercice : Utiliser `node:alpine` pour construire l'application, et exposer les fichiers buildés dans une image nginx.

### Les consignes:

- Téléchargez ou installez le projet qui contient le code du site uno [est disponible sur ce repo Github](https://github.com/mtbinds/DOCKER_EXAMPLE_2)
- `cd` dans le dossier contenant le projet uno
- Créez un fichier `Dockerfile`, et complétez afin de pouvoir créer l'image `madjidtaoualit/uno`.

Pour lancer le build à partir d'un Dockerfile, placez vous dans le même dossier et lancez:

```bash
docker build -t <nom-de-votre-image> .
```
Si aucun message d'erreur ne s'affiche, l'image est construite correctement.

Vous pouvez ensuite tester l'image simplement:

```bash
docker run --rm -p 3000:3000 <nom-de-votre-image>
```
Allez ensuite dans votre navigateur pour vérifier que le site s'affiche correctement (Pensez à supprimer le cache !), vous devriez voir le résultat suivant:

![website uno](./assets/website-image.png)

