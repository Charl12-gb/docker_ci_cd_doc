## **Guide d'Installation et Configuration de Docker pour un Projet FastAPI avec MySQL/PostgreSQL**

### **1. Introduction**

Dans ce guide, vous apprendrez à installer et configurer **Docker** sur **Windows** et **Linux**, puis à configurer un environnement Docker pour exécuter un projet **FastAPI** avec **MySQL/PostgreSQL**.

---

### **2. Installation de Docker**

#### **2.1. Installation de Docker sur Windows**

1. **Téléchargement de Docker Desktop pour Windows**
   - Allez sur la page officielle de Docker pour télécharger Docker Desktop :  
     [Docker Desktop pour Windows](https://www.docker.com/products/docker-desktop)
   
2. **Installation de Docker Desktop**
   - Lancez l'installateur téléchargé et suivez les étapes d'installation à l'écran.
   - Redémarrez votre ordinateur si cela est nécessaire.

3. **Vérification de l'installation**
   - Une fois l'installation terminée, ouvrez **PowerShell** ou **CMD** et tapez la commande suivante pour vérifier que Docker est correctement installé :
     ```bash
     docker --version
     ```
   - Si la version de Docker est correctement affichée, cela signifie que Docker est installé avec succès.

4. **Configuration de Docker**
   - Docker Desktop utilise **WSL 2** (Windows Subsystem for Linux version 2). Si ce n'est pas déjà activé, vous pouvez activer WSL 2 en suivant les instructions sur [la documentation officielle de Docker](https://docs.docker.com/docker-for-windows/wsl/).

#### **2.2. Installation de Docker sur Linux (Ubuntu)**

1. **Mettre à jour vos paquets**
   - Ouvrez un terminal et mettez à jour vos paquets avec la commande :
     ```bash
     sudo apt update
     ```

2. **Installer les paquets nécessaires**
   - Installez les paquets nécessaires pour installer Docker :
     ```bash
     sudo apt install apt-transport-https ca-certificates curl software-properties-common
     ```

3. **Ajouter la clé GPG de Docker**
   - Ajoutez la clé GPG officielle de Docker :
     ```bash
     curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
     ```

4. **Ajouter le dépôt Docker à APT**
   - Ajoutez le dépôt officiel de Docker à APT :
     ```bash
     sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
     ```

5. **Mettre à jour les sources de paquets**
   - Mettez à jour les sources de paquets :
     ```bash
     sudo apt update
     ```

6. **Installer Docker**
   - Installez Docker avec la commande suivante :
     ```bash
     sudo apt install docker-ce
     ```

7. **Vérification de l'installation**
   - Vérifiez que Docker est bien installé avec la commande suivante :
     ```bash
     docker --version
     ```

8. **Droits d'utilisateur Docker**
   - Pour éviter de devoir utiliser `sudo` à chaque commande Docker, ajoutez votre utilisateur au groupe Docker :
     ```bash
     sudo usermod -aG docker $USER
     ```
   - Déconnectez-vous puis reconnectez-vous ou redémarrez votre machine pour appliquer les modifications.

---

### **3. Configuration d'un Projet FastAPI avec Docker**

#### **3.1. Structure du projet**

Voici la structure de base de votre projet **FastAPI** qui sera déployé avec Docker :

```
/my_project
  ├── Dockerfile
  ├── docker-compose.yml
  ├── .env
  ├── app/
  ├── alembic/
  └── requirements.txt
```

#### **3.2. Dockerfile**

Dans votre **Dockerfile**, vous devez définir les étapes pour créer votre image Docker. Voici un exemple pour un projet **FastAPI** utilisant **Python** :

```dockerfile
# Utilisation d'une image Python officielle comme base
FROM python:3.9-slim

# Installation des dépendances système nécessaires
RUN apt-get update && apt-get install -y \
    gcc \
    libpq-dev

# Définir le répertoire de travail dans le conteneur
WORKDIR /app

# Copier le fichier requirements.txt et installer les dépendances Python
COPY requirements.txt /app/
RUN pip install --no-cache-dir -r requirements.txt

# Copier tout le code source dans le conteneur
COPY . /app/

# Exposer le port sur lequel FastAPI sera exécuté
EXPOSE 8000

# Commande pour démarrer l'application FastAPI avec Uvicorn
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

#### **3.3. docker-compose.yml**

Le fichier **docker-compose.yml** permet de définir et de gérer des services multiples avec Docker. Voici un exemple de fichier **docker-compose.yml** qui configure **FastAPI**, **MySQL**, et **phpMyAdmin** :

```yaml
version: '3.8'

services:
  app:
    build: .
    container_name: fastapi_app
    env_file: .env
    ports:
      - "8000:8000"
    volumes:
      - .:/app
    depends_on:
      - db
    restart: always

  db:
    image: mysql:8
    container_name: mysql_db
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
      MYSQL_DATABASE: ${DB_NAME}
    volumes:
      - db_data:/var/lib/mysql
    ports:
      - "3306:3306"

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadmin
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
    ports:
      - "8080:80"

volumes:
  db_data:
```

- **app** : Le conteneur pour votre application **FastAPI**.
- **db** : Le conteneur MySQL.
- **phpmyadmin** : L'interface Web pour administrer MySQL.

#### **3.4. Fichier `.env`**

Votre fichier `.env` contient les variables d'environnement utilisées par votre projet, comme le mot de passe de la base de données et le nom de la base de données. Voici un exemple :

```
# Variables de la base de données
DB_USER=root
DB_PASSWORD=motdepassesecure
DB_NAME=database_name
DB_HOST=db
DB_PORT=3306

# Mot de passe pour l'utilisateur root (à remplacer par votre propre mot de passe sécurisé)
DB_ROOT_PASSWORD=motdepassesecure
```

---

### **4. Utilisation du fichier `entrypoint.sh`**

Le fichier `entrypoint.sh` est un script qui sera exécuté lorsque le conteneur **Docker** démarre. Il peut être utilisé pour effectuer certaines actions d'initialisation avant le démarrage de votre application **FastAPI**.

Voici un exemple de **`entrypoint.sh`** :

```bash
#!/bin/sh

# Attente que la base de données MySQL soit prête
echo "Waiting for MySQL to be ready..."
until nc -z -v -w30 $DB_HOST $DB_PORT
do
  echo "Waiting for MySQL..."
  sleep 10  # Attente de 10 secondes avant de réessayer
done

# Appliquer les migrations avec Alembic
echo "Applying database migrations..."
alembic upgrade head

# Exécuter le script de seed
echo "Seeding the database..."
python seed.py seed_default_user --email admin@example.com --password password

# Démarrer l'application avec Uvicorn en mode rechargement
echo "Starting FastAPI server with auto-reload..."
exec uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

1. **Ce que fait ce script** :
   - Il attend que MySQL soit prêt à accepter des connexions.
   - Applique les migrations de base de données avec **Alembic**.
   - Démarre **FastAPI** avec **Uvicorn**.

2. **Ajouter ce script à Docker** :
   Dans le fichier **Dockerfile**, vous devez ajouter ce script comme point d'entrée.

```dockerfile
# Utilisation d'une image de base Python
FROM python:3.9-slim

# Installer les dépendances système pour mysqlclient et netcat (en utilisant netcat-openbsd)
RUN apt-get update && apt-get install -y \
    pkg-config \
    libmariadb-dev \
    gcc \
    g++ \
    netcat-openbsd  # Utilisation de netcat-openbsd à la place de netcat \
    && rm -rf /var/lib/apt/lists/*

# Créer et définir le répertoire de travail
WORKDIR /app

# Copier le fichier requirements.txt et installer les dépendances Python
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copier le reste du code de l'application
COPY . .

# Copier le script d'entrée
COPY entrypoint.sh /app/entrypoint.sh
RUN chmod +x /app/entrypoint.sh

# Exposer le port par défaut de FastAPI
EXPOSE 8000

# Utiliser entrypoint.sh comme point d'entrée
ENTRYPOINT ["/app/entrypoint.sh"]
```

---

### **5. Démarrer les services avec Docker**

1. **Construire et démarrer les services avec Docker Compose** :
   À partir du répertoire racine de votre projet, exécutez la commande suivante pour construire et démarrer tous les services définis dans votre fichier **docker-compose.yml** :
   ```bash
   docker-compose up --build
   ```

2. **Accéder à phpMyAdmin** :
   Après que les conteneurs aient démarré, vous pouvez accéder à l'interface **phpMyAdmin** à l'adresse suivante dans votre navigateur :
   ```
   http://localhost:8080
   ```

   Connectez-vous avec les identifiants MySQL :
   - **Serveur** : `db`
   - **Utilisateur** : `root`
   - **Mot de passe** : défini dans `.env`

---

### **6. Conclusion**

Avec Docker, vous avez maintenant un environnement de développement isolé pour votre projet **FastAPI** avec **MySQL** et **phpMyAdmin**. Vous pouvez facilement gérer et déployer votre application avec Docker en utilisant **docker-compose** pour orchestrer les services.

### BD

Si vous voulez voir et interagir avec votre base de données MySQL sans entrer dans le conteneur Docker, vous pouvez utiliser des outils graphiques comme **phpMyAdmin**, **DBeaver**, ou **MySQL Workbench** pour vous connecter directement à votre base de données MySQL qui fonctionne dans Docker. Voici comment procéder pour configurer et visualiser les données dans Docker de manière plus conviviale.

### 1. **Accéder à la base de données MySQL dans Docker avec phpMyAdmin** :

phpMyAdmin est une interface web populaire qui vous permet d'interagir avec MySQL facilement. Vous pouvez le configurer dans votre `docker-compose.yml` pour accéder à votre base de données via un navigateur.

#### Étapes :
1. **Ajouter phpMyAdmin à votre fichier `docker-compose.yml`** :

   Dans votre fichier `docker-compose.yml`, ajoutez un service pour phpMyAdmin :

   ```yaml
   version: '3'
   services:
     db:
       image: mysql:8
       container_name: db
       environment:
         MYSQL_ROOT_PASSWORD: rootpassword
         MYSQL_DATABASE: cica_db
       ports:
         - "3306:3306"
       volumes:
         - db_data:/var/lib/mysql

     phpmyadmin:
       image: phpmyadmin/phpmyadmin
       container_name: phpmyadmin
       environment:
         PMA_HOST: db
         PMA_PORT: 3306
       ports:
         - "8080:80"  # Port pour accéder à phpMyAdmin
       depends_on:
         - db

   volumes:
     db_data:
   ```

   Dans cet exemple, **phpMyAdmin** sera accessible via `http://localhost:8080` sur votre machine locale.

2. **Redémarrez vos conteneurs Docker** :

   Une fois le fichier modifié, vous devez redémarrer votre application Docker pour appliquer les changements :

   ```bash
   docker-compose down
   docker-compose up
   ```

3. **Accéder à phpMyAdmin** :

   Vous pouvez maintenant ouvrir votre navigateur et accéder à phpMyAdmin à l'adresse `http://localhost:8080`.

   **Identifiants de connexion** :
   - **Serveur MySQL** : `db` (nom du conteneur de la base de données)
   - **Utilisateur** : `root`
   - **Mot de passe** : celui défini dans `MYSQL_ROOT_PASSWORD` (par exemple, `rootpassword`)

4. **Explorer vos bases de données** :

   Une fois connecté, vous pourrez visualiser et interagir avec toutes vos bases de données MySQL directement depuis l'interface web de phpMyAdmin.

---

### **Accéder à la base de données MySQL dans Docker avec DBeaver (outil de gestion de base de données)** :

Si vous préférez un outil desktop plus puissant, vous pouvez utiliser **DBeaver** (disponible sur Windows, macOS et Linux).

#### Étapes :
1. **Télécharger et installer DBeaver** :
   - Allez sur [le site de DBeaver](https://dbeaver.io/download/) et téléchargez la version correspondant à votre système d'exploitation.
   - Installez DBeaver.

2. **Configurer la connexion MySQL** :
   - Lancez DBeaver et cliquez sur "Database" > "New Database Connection".
   - Sélectionnez "MySQL" comme type de base de données.
   - Remplissez les informations de connexion :
     - **Host** : `localhost` (ou l'adresse IP du conteneur Docker si elle est différente, vous pouvez obtenir cela via `docker inspect db`).
     - **Port** : `3306` (par défaut pour MySQL).
     - **Username** : `root` (ou tout autre utilisateur que vous avez configuré).
     - **Password** : le mot de passe défini dans votre fichier `.env` ou `docker-compose.yml`.
     - **Database** : Le nom de la base de données, par exemple `cica_db`.

3. **Connecter et explorer** :
   - Une fois la connexion établie, vous pourrez explorer toutes les bases de données, tables, et exécuter des requêtes SQL directement dans DBeaver.

---

### 3. **Accéder à la base de données MySQL avec MySQL Workbench** :

**MySQL Workbench** est un autre outil graphique très populaire pour interagir avec MySQL.

#### Étapes :
1. **Télécharger et installer MySQL Workbench** :
   - Téléchargez et installez **MySQL Workbench** depuis [le site officiel de MySQL](https://dev.mysql.com/downloads/workbench/).

2. **Configurer la connexion MySQL** :
   - Ouvrez MySQL Workbench et créez une nouvelle connexion en cliquant sur le "+" à côté de "MySQL Connections".
   - Entrez les détails suivants pour la connexion :
     - **Hostname** : `localhost` (ou l'adresse IP de votre conteneur MySQL si nécessaire).
     - **Port** : `3306`.
     - **Username** : `root`.
     - **Password** : Le mot de passe root défini dans votre configuration Docker (par exemple `rootpassword`).

3. **Explorer la base de données** :
   - Une fois la connexion établie, vous pouvez voir vos bases de données, exécuter des requêtes, et interagir avec votre base de données MySQL.

---

### Conclusion :

Utiliser phpMyAdmin ou un outil de gestion comme **DBeaver** ou **MySQL Workbench** vous permettra de visualiser et gérer votre base de données MySQL sans avoir à entrer dans le conteneur Docker. Ces outils offrent des interfaces graphiques conviviales et sont très efficaces pour interagir avec vos données MySQL.
