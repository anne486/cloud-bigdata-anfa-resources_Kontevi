# Rendu - Séance 2
**Nom et prénom :** KONTEVI Akossiwa Anne
**Identifiant GitHub :** anne486
**Date de soumission :** 25/06/2026



## Résumé de la séance

Au cours de cette séance, j'ai conteneurisé une application PySpark d'analyse du référentiel d'Anfa à l'aide d'un Dockerfile, en appliquant les bonnes pratiques essentielles comme l'utilisation d'un `.dockerignore` et l'optimisation du cache. J'ai ensuite orchestré une stack complète de 3 services (MinIO, Jupyter et l'image custom) avec Docker Compose. Enfin, j'ai exploré les données du bucket `anfa-raw` depuis un notebook Jupyter via `boto3` et `pandas`, démontrant ainsi l'interopérabilité des services conteneurisés.

## Étapes principales

1. **Écriture du Dockerfile et construction de l'image** : Création du script `analyse_referentiel.py`, du `requirements.txt` et du `Dockerfile` pour conteneuriser l'application PySpark. Construction de l'image `anfa-analyse:v1` avec une taille observée de **808 Mo**.

2. **Mise en place du `.dockerignore` et observation du cache** : Ajout du fichier `.dockerignore` pour exclure les fichiers inutiles du contexte de build. Observation du mécanisme de cache de Docker : lors de modifications du code uniquement, l'étape `pip install` est réutilisée (CACHED) grâce à l'ordre stratégique des instructions dans le Dockerfile.

3. **Orchestration avec Docker Compose** : Écriture du fichier `docker-compose.yml` définissant trois services :
   - **MinIO** : stockage S3-compatible pour les données
   - **Jupyter** : environnement de notebooks scientifiques
   - **anfa-app** : application batch PySpark (qui s'exécute et se termine)

4. **Exploration des données** : Création du notebook `exploration_minio.ipynb` qui :
   - Se connecte à MinIO via `boto3`
   - Liste les objets du bucket `anfa-raw`
   - Lit le fichier `referentiel/lignes.csv` avec `pandas`
   - Affiche les données et effectue une analyse exploratoire (Top 3 des lignes les plus longues)

## Captures d'écran

### docker compose ps
![docker compose ps](captures/docker-ps.png)

### Notebook Jupyter
![Notebook Jupyter](captures/jupyter-pandas.png)

### Bonus multi-stage (optionnel)
*Non réalisé.*

### Réponses aux exercices d'application

**Question 1 : Pourquoi est-il important de copier `requirements.txt` avant le reste du code ?**

Cela permet d'exploiter le mécanisme de cache de Docker. Les dépendances sont installées dans une couche séparée. Tant que `requirements.txt` ne change pas, Docker réutilise la couche en cache et ne réinstalle pas PySpark (300+ Mo) à chaque modification du code. Cela accélère considérablement les rebuilds en développement.

**Question 2 : Quelle est la différence entre `docker run` et `docker compose up` ?**

- `docker run` : permet de lancer un seul conteneur à partir d'une image avec des options spécifiques (ports, volumes, etc.). Il est utilisé pour des cas simples ou des tests unitaires.

- `docker compose up` : orchestre un ensemble de services définis dans un fichier YAML. Il crée, démarre et connecte les conteneurs entre eux sur un réseau dédié, avec gestion des dépendances (healthcheck, ordre de démarrage). C'est la solution adaptée pour des applications multi-conteneurs complexes.

**Question 3 : Pourquoi `anfa-app` est-il dans l'état `Exited (0)` dans `docker compose ps` ?**

`anfa-app` est une application batch (script PySpark) qui s'exécute, effectue son analyse et se termine proprement. Le code de sortie `0` indique une terminaison réussie. Contrairement aux services web (Jupyter, MinIO), ce conteneur n'a pas besoin de tourner en permanence. C'est pourquoi le paramètre `restart: "no"` est utilisé dans le `docker-compose.yml` pour éviter qu'il ne redémarre sans cesse.

### Difficultés rencontrées

**Aucune difficulté majeure.** L'ensemble du TP s'est déroulé sans erreur notable. Les captures d'écran ont été réalisées conformément aux consignes. Les services ont démarré correctement et le notebook Jupyter a permis d'explorer avec succès les données stockées dans MinIO.

