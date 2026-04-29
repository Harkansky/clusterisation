1. Liste complète des tâches (sans bonus)
1️⃣ Préparer l’infrastructure du cluster (2 points)
Objectif : 1 manager + 2 workers
Tâches :
    1. Installer Docker sur les 3 machines
    2. Initialiser Docker Swarm
docker swarm init
    3. Ajouter les workers
docker swarm join --token TOKEN IP_MANAGER:2377
    4. Vérifier le cluster
docker node ls
Résultat attendu :
MANAGER
WORKER
WORKER

2️⃣ Containeriser les applications (5 points)
Architecture minimale :
Frontend
Backend
Database
Tâches :
Frontend
Créer :
frontend/
Dockerfile
Exemple :
FROM node:20
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build
CMD ["npm","run","preview"]

Backend
Créer :
backend/
Dockerfile
Exemple :
FROM node:20
WORKDIR /app
COPY . .
RUN npm install
CMD ["node","server.js"]

Database
Utiliser image officielle :
postgres
ou
mysql

3️⃣ Créer la stack Docker Swarm
Créer :
stack.yml
Structure minimale :
version: "3.8"
services:
  frontend:
    image: frontend
    deploy:
      replicas: 3
    ports:
      - "80:3000"
  backend:
    image: backend
    deploy:
      replicas: 2
  db:
    image: postgres
    volumes:
      - db_data:/var/lib/postgresql/data
volumes:
  db_data:
Déployer :
docker stack deploy -c stack.yml app
Vérifier :
docker service ls

4️⃣ Persistance des données (2 points)
Objectif :
La DB survit à la suppression d’un container.
Tâches :
Créer volume :
volumes:
  db_data:
Test :
1️⃣ Supprimer container DB
docker service rm app_db
2️⃣ Le redéployer
Les données doivent toujours être là.

5️⃣ Mise en réseau et exposition (2 points)
Objectif :
Accéder à l’application via navigateur.
Créer réseau overlay :
docker network create --driver overlay app-network
Dans stack :
networks:
  app-network:
Services :
frontend
backend
db
connectés à ce réseau.

6️⃣ Reverse Proxy + HTTPS (2 points)
Utiliser :
Traefik
ou
Nginx
Le plus simple : Traefik
Services :
traefik
frontend
backend
Fonctions :
    • reverse proxy
    • génération HTTPS

7️⃣ Secrets (sécurité) (2 points)
Créer secrets :
echo "mypassword" | docker secret create db_password -
Dans stack :
secrets:
  db_password:
    external: true
Utilisation :
POSTGRES_PASSWORD_FILE=/run/secrets/db_password

8️⃣ Test haute disponibilité
Test obligatoire pour capture :
1️⃣ scale service
docker service scale app_backend=4
2️⃣ kill container
docker ps
docker kill containerID
Docker doit recréer automatiquement un container.

9️⃣ DNS local
Modifier :
/etc/hosts
Exemple :
127.0.0.1 app.local
Accès :
http://app.local

10️⃣ Documentation finale (2 points)
Le repo doit contenir :
README.md
architecture.png
stack.yml
dockerfiles
Documentation :
1️⃣ Installation Docker
2️⃣ Création cluster
3️⃣ Déploiement stack
4️⃣ Tests haute disponibilité
5️⃣ Accès application
Captures :
    • docker node ls
    • docker service ls
    • scaling
    • kill container
    • application fonctionnelle

2. Répartition des tâches pour 2 personnes
👨‍💻 Personne A — Démarrage du projet
Objectif : tout préparer pour que l’app fonctionne dans le cluster
Tâches :
Infrastructure
    • installer Docker sur 3 machines
    • créer cluster swarm
    • connecter workers
Commandes :
docker swarm init
docker swarm join
docker node ls

Containerisation
Créer :
frontend/Dockerfile
backend/Dockerfile
Tester localement :
docker build
docker run

Stack Docker
Créer :
stack.yml
Services :
frontend (3 replicas)
backend (2 replicas)
db (1 replica)

Réseau overlay
Créer réseau :
docker network create --driver overlay app-network

Déployer la stack
docker stack deploy -c stack.yml app

Vérifications
Tester :
docker service ls
docker service ps

Livrables personne A
Repo contenant :
frontend/
backend/
stack.yml
dockerfiles
    • stack fonctionnelle sans HTTPS.

👨‍💻 Personne B — Finalisation du projet
Objectif : sécurité, exposition et documentation

Reverse proxy
Installer :
Traefik
Configurer :
routing
https
domain

HTTPS
Configurer :
certificat auto-signé
ou
letsencrypt

Secrets
Créer :
docker secret create
Secrets :
db_password
api_key

Persistance DB
Configurer :
volume postgres
Tester :
supprimer container
verifier data

Tests haute disponibilité
Faire captures :
scale services
kill container
recreation auto

DNS
Configurer :
/etc/hosts
app.local

Documentation
Créer :
README.md
Contenu :
architecture
installation
commandes
tests
captures

3. Structure finale du repo
project-cluster/
frontend/
Dockerfile
backend/
Dockerfile
stack.yml
docs/
architecture.png
tests.png
README.md

4. Planning conseillé (super efficace)
Jour 1
Personne A
Dockerfiles
stack.yml
cluster
deployment

Jour 2
Personne B
traefik
https
secrets
tests
documentation

5. Résultat final attendu
Application :
https://app.local
Architecture :
           Traefik
              |
      -----------------
      |               |
   Frontend x3    Backend x2
                      |
                   Database
                   (volume)
