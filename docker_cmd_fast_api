Voici une liste de commandes Docker pour exécuter votre projet **FastAPI** dans un conteneur Docker, ainsi que des explications sur leur rôle.

### **1. Commandes Docker pour gérer l'image et le conteneur**

#### **1.1. Construire l'image Docker**
- **Commande :**
  ```bash
  docker build -t fastapi-app .
  ```
  - **Rôle :** Cette commande construit l'image Docker à partir du `Dockerfile` présent dans le répertoire courant (`.`). L'option `-t fastapi-app` permet de taguer l'image avec le nom `fastapi-app`.

#### **1.2. Lister les images Docker**
- **Commande :**
  ```bash
  docker images
  ```
  - **Rôle :** Affiche toutes les images Docker présentes sur votre machine, y compris l'image `fastapi-app` que vous venez de créer.

#### **1.3. Lancer le conteneur Docker**
- **Commande :**
  ```bash
  docker run -d -p 8000:8000 --name fastapi_app-container fastapi-app
  ```
  - **Rôle :** Lance un conteneur en arrière-plan (`-d`) à partir de l'image `fastapi-app`, tout en mappant le port local 8000 au port 8000 du conteneur (`-p 8000:8000`). Le conteneur est nommé `fastapi_app-container`.

#### **1.4. Lancer un conteneur Docker avec volume monté**
- **Commande :**
  ```bash
  docker run -d -p 8000:8000 -v $(pwd):/app --name fastapi_app-container fastapi-app
  ```
  - **Rôle :** Lance un conteneur avec le code source de votre projet monté dans le conteneur. `$(pwd)` permet de monter le répertoire courant du système hôte dans `/app` dans le conteneur.

#### **1.5. Lister les conteneurs Docker**
- **Commande :**
  ```bash
  docker ps
  ```
  - **Rôle :** Affiche les conteneurs en cours d'exécution, avec leur nom, ID, ports exposés, etc.

#### **1.6. Lister tous les conteneurs, y compris ceux qui sont arrêtés**
- **Commande :**
  ```bash
  docker ps -a
  ```
  - **Rôle :** Affiche tous les conteneurs, qu'ils soient en cours d'exécution ou arrêtés.

#### **1.7. Accéder à un conteneur Docker en mode interactif**
- **Commande :**
  ```bash
  docker exec -it fastapi_app-container /bin/bash
  ```
  - **Rôle :** Permet d'ouvrir un terminal interactif (`-it`) dans le conteneur `fastapi_app-container` pour exécuter des commandes directement à l'intérieur du conteneur.

#### **1.8. Arrêter un conteneur Docker**
- **Commande :**
  ```bash
  docker stop fastapi_app-container
  ```
  - **Rôle :** Arrête le conteneur nommé `fastapi_app-container`.

#### **1.9. Supprimer un conteneur Docker**
- **Commande :**
  ```bash
  docker rm fastapi_app-container
  ```
  - **Rôle :** Supprime le conteneur nommé `fastapi_app-container`, après l'avoir arrêté.

#### **1.10. Supprimer une image Docker**
- **Commande :**
  ```bash
  docker rmi fastapi-app
  ```
  - **Rôle :** Supprime l'image `fastapi-app` de votre machine. Assurez-vous que le conteneur basé sur cette image est arrêté avant de supprimer l'image.

---

### **2. Commandes Docker Compose (si utilisé)**

#### **2.1. Démarrer tous les services avec Docker Compose**
- **Commande :**
  ```bash
  docker-compose up
  ```
  - **Rôle :** Lance tous les services définis dans le fichier `docker-compose.yml`. Si les images n'existent pas encore, elles seront automatiquement construites.

#### **2.2. Démarrer les services en arrière-plan**
- **Commande :**
  ```bash
  docker-compose up -d
  ```
  - **Rôle :** Lance tous les services en arrière-plan, ce qui permet de continuer à utiliser le terminal pendant l'exécution.

#### **2.3. Arrêter les services Docker Compose**
- **Commande :**
  ```bash
  docker-compose down
  ```
  - **Rôle :** Arrête tous les services définis dans le fichier `docker-compose.yml` et supprime les conteneurs associés.

#### **2.4. Voir les logs des services Docker Compose**
- **Commande :**
  ```bash
  docker-compose logs
  ```
  - **Rôle :** Affiche les logs de tous les services Docker Compose. Vous pouvez ajouter le nom du service pour afficher les logs d'un seul service (par exemple, `docker-compose logs app`).

#### **2.5. Lancer des commandes spécifiques dans un conteneur via Docker Compose**
- **Commande :**
  ```bash
  docker-compose exec app /bin/bash
  ```
  - **Rôle :** Exécute une commande dans le conteneur d'un service spécifique défini dans `docker-compose.yml` (par exemple, accéder à un conteneur `app` en mode interactif).

#### **2.6. Construire l'image avec Docker Compose**
- **Commande :**
  ```bash
  docker-compose build
  ```
  - **Rôle :** Construit les images Docker pour tous les services définis dans le fichier `docker-compose.yml`.

---

### **3. Exécution des commandes FastAPI dans Docker**

#### **3.1. Exécuter les migrations Alembic (Base de données)**
Si vous utilisez Alembic pour les migrations de base de données dans votre projet **FastAPI**, exécutez la commande suivante après avoir accédé au conteneur Docker :

- **Commande :**
  ```bash
  docker exec -it fastapi_app-container alembic upgrade head
  ```
  - **Rôle :** Applique les migrations de base de données jusqu'à la révision la plus récente (`head`).

#### **3.2. Lancer les tests FastAPI**
Si vous avez des tests définis dans votre projet (par exemple avec `pytest`), vous pouvez exécuter les tests dans le conteneur :

- **Commande :**
  ```bash
  docker exec -it fastapi_app-container pytest
  ```
  - **Rôle :** Lance les tests définis dans votre projet à l'intérieur du conteneur Docker.

---

### **4. Commandes utiles pour MySQL (si utilisé avec Docker)**

#### **4.1. Accéder au conteneur MySQL**
Si vous avez un conteneur MySQL en cours d'exécution et souhaitez y accéder :

- **Commande :**
  ```bash
  docker exec -it mysql-db mysql -u root -p
  ```
  - **Rôle :** Ouvre un terminal MySQL dans le conteneur `mysql-db` en tant qu'utilisateur `root`. Vous devrez entrer le mot de passe root pour MySQL.

#### **4.2. Vérifier les bases de données MySQL**
Une fois connecté à MySQL, vous pouvez lister toutes les bases de données :

- **Commande :**
  ```sql
  SHOW DATABASES;
  ```
  - **Rôle :** Affiche toutes les bases de données présentes dans MySQL.

---

### **5. Commandes de gestion des volumes Docker**

#### **5.1. Lister les volumes Docker**
- **Commande :**
  ```bash
  docker volume ls
  ```
  - **Rôle :** Affiche tous les volumes Docker existants.

#### **5.2. Supprimer un volume Docker**
- **Commande :**
  ```bash
  docker volume rm <volume_name>
  ```
  - **Rôle :** Supprime un volume Docker spécifique par son nom.

---

Avec ces commandes, vous pouvez gérer, exécuter et tester votre projet **FastAPI** dans un environnement Docker. Assurez-vous que votre fichier `Dockerfile` et `docker-compose.yml` sont correctement configurés pour éviter les erreurs et optimiser la gestion des conteneurs.
