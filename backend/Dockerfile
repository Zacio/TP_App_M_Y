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