# App Todo

## Prérequis
Pour commancer installer les éléments suivants :
- https://www.docker.com/get-started [docker]

## Instalation du projet
1. Clonez le projet :
  ```bash
  git clone https://github.com/Zacio/TP_App_M_Y.git
  ```

2. Construisez le conteneur
  si vous êtes sur windows ouvrez le dossier qui contien l'application et écrivez cmd la barre du chemin du dossier
  pour ouvrir un terminal qui pointe directement sur le chemin voulue , tapez ensuite
  ```bash
  docker-compose up --build
  ```
  dans le terminal

3. Accédez à l'application
  vous pouvez accédez a l'application en tapant
  ```
  http://localhost:3000
  ```
  sur la barre de recherche de votre navigateur

## Structure du dossier de l'application
- `frontend/` - Contient le code front (ReactJS) avec un fichier Dockerfile.
- `backend/` - Contient le code back (Flask) avec un fichier Dockerfile.
- `docker-compose.yml` - Fichier de configuration de Docker Compose.

## Utilisation

- Pour arrêter les conteneur faites :

  ```bash
  docker-compose down
  ```

- après avoir apporté des modifications au code, utilisez :
  ```bash
  docker-compose up --build
  ```
  Pour reconstruire les conteneurs

 ## La base de données
 L'application utilise MySQL pour stocker les données.
  -MySQL et ses configurations est défini dans le fichier `docker-compose.yml`

Pour géré la base de donnée phpMyAdmin est également ajouté au projet
  -phpMyAdmin et ses configurations est défini dans le fichier `docker-compose.yml`

la configuration de MySQL dans flask est fait dans le fichier `config.py` qui se trouve dans le dossier backend
```python
class DevelopmentConfig(Config):
    SQLALCHEMY_DATABASE_URI = (
        f"mysql+pymysql://{os.getenv('MYSQL_USER')}:{os.getenv('MYSQL_PASSWORD')}@{os.getenv('MYSQL_HOST')}/{os.getenv('MYSQL_DATABASE')}"
    )
    # Flask jwt extended
    JWT_ACCESS_TOKEN_EXPIRES = timedelta(minutes=30)
```

### phpMyAdmin

Pour accéder à phpMyAdmin ouvrez votre navigateur et tappez dans la barre des url :
```
http://localhost:8000
```
Utilisez les informations dans le fichier `docker-compose.yml` pour vous connecter.
```yml
mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: mydb
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - app_network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 30s
      timeout: 10s
      retries: 3
```

## Les healthchecks
ils permette de savoir si les conteneur fonctionnent correctement
example :
```yml
healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 30s
      timeout: 10s
      retries: 3
```
## Les dockerfile
Les fichiers Dockerfile doivent être placés dans les répertoires correspondant à leurs services respectifs pour que la configuration docker-compose.yml puisse les trouver facilement

dockerfile back
```dockerfile
# Utiliser une image python comme base
FROM python:3.9-slim

# Définir le répertoire de travail
WORKDIR /app

# Copier les fichiers back de l'application
COPY . /app

# Installer pip et les dépendance de requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Exposer le port utilisé par React qui est 3000 par défaut
EXPOSE 5000

CMD ["flask", "run", "--host=0.0.0.0", "--port=5000"]
```

dockerfile front :
```dockerfile
# Utiliser une image Node.js comme base
FROM node:16-alpine

# Définir le répertoire de travail
WORKDIR /app

# Copier le fichier package.json et package-lock.json
COPY package*.json ./

# Installer les dépendances
RUN npm install

# Copier les fichiers front de l'application
COPY . .

# Exposer le port utilisé par React qui est 3000 par défaut
EXPOSE 3000

# Démarrer l'application en mode développement
CMD ["npm", "run", "dev"]
```

## Le docker compose
Le fichier Docker Compose est un fichier yaml qui permet de définir et d’orchestrer plusieurs conteneurs Docker il doit-etre placer
a la racine du projet
```yml
services:
  back:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: user
      MYSQL_PASSWORD: password
      MYSQL_DATABASE: mydb
    networks:
      - app_network
    depends_on:
      - mysql
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  front:
    build: ./frontend
    ports:
      - "3000:3000"
    networks:
      - app_network
    depends_on:
      - back
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000"]
      interval: 30s
      timeout: 10s
      retries: 3

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: mydb
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - app_network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 30s
      timeout: 10s
      retries: 3

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    environment:
      PMA_HOST: mysql
      MYSQL_ROOT_PASSWORD: rootpassword
    ports:
      - "8000:80"
    networks:
      - app_network

  nginx:
    image: nginx:latest
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    ports:
      - "80:80"
    networks:
      - app_network
    depends_on:
      - front
      - back
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3

networks:
  app_network:
    driver: bridge

volumes:
  mysql_data:
```
### Comment ça marche
voicie un example de fonctionnement
- services: Contien les différents outils utiliser par l'application
  - back : c'est le nom de l'outils 
    - build : c'est le chemin vers l'outil (dans ce cas la le dockerfile du backend)
    - ports : représente le port qu'utilise l'outil (ça s'écrit comme cecie : lePortVoulue:LePortNéssaisere)
    - environment : Définit les variables d'environnement néssaisere (ici les variables d'environnement mysql)
    - depends_on : définis les dépendance de l'outil (dépend de mysql)
    - healthcheck : permette de savoir si l'outil fonctionnent correctement

il existe aussi 
- image : permet de spécifier l'image Docker à utiliser pour un outil 
- networks : Définit le réseau utilisé par les services.
- volumes : Définit le volume pour persister les données MySQL.