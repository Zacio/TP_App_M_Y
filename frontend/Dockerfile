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