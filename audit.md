# Exercice 1 : Configuration locale de l’API et audit

## Étape 1 : lancement de l’API en local (verification)

* [x] Le dépôt de l’API a bien été cloné
* [x] Les dépendances principales sont installées (backend avec `uv`, frontend avec `npm`)
* [x] Les serveurs de développement tournent en local
* [x] L’API est accessible sur `http://localhost:8000` et l’interface sur `http://localhost:3000`
* [x] Quelques endpoints ont été testés (`/api/v1/stations/` et `/api/v1/horaire/`) et renvoient des réponses correctes

L’application peut donc être lancée en local sans problème particulier. L’environnement de dev fonctionne globalement bien même si certaines étapes restent un peu manuelles.

---

## Étape 2 : audit du système

Voici un audit du projet organisé par plusieurs catégories importantes.

### 1. Qualité du code

**Ce qui manque :**

* Un peu plus d’outils pour analyser la sécurité du code (par exemple un scanner Python type **Bandit**)
* Un contrôle plus strict de la couverture de tests pour éviter qu’elle baisse avec le temps

**Manuel vs automatisé :**

* **Automatisé** : un système de `pre-commit` vérifie certaines règles de formatage ou de linting avant les commits, et une vérification similaire est lancée sur les Pull Requests avec **GitHub Actions**
* **Manuel** : la cohérence globale de l’architecture ou de la logique métier repose surtout sur les revues de code

**Points de défaillance possibles :**

* Les hooks locaux peuvent être contournés avec certaines options git
* Si les règles dans la CI ne sont pas assez strictes, du code non conforme peut quand même être mergé

---

### 2. Tests

**Ce qui manque :**

* Les tests ne semblent pas être exécutés automatiquement dans la CI
* Peu ou pas de tests d’intégration complets
* Aucun vrai test de charge sur la base de données ou l’API

**Manuel vs automatisé :**

* **Manuel** : les tests unitaires peuvent être lancés avec `pytest` pour le backend ou via les scripts `npm`, mais cela dépend surtout de la discipline des développeurs

**Points de défaillance possibles :**

* Des régressions peuvent être introduites sans être détectées
* La CI peut construire et déployer une image même si certains tests échoueraient localement.

---

### 3. Processus de déploiement

**Ce qui manque :**

* Une stratégie pour éviter les interruptions pendant les mises à jour
* Un système de rollback automatique si une nouvelle version ne démarre pas

**Manuel vs automatisé :**

* **Automatisé** : le déploiement semble déclenché lorsqu’un tag de version est publié et les images Docker sont construites dans la pipeline
* **Manuel** : certaines opérations côté serveur restent assez simples (arrêt puis relance des conteneurs)

**Points de défaillance possibles :**

* Une courte interruption de service peut arriver pendant le redémarrage des conteneurs
* Si une commande échoue pendant le déploiement, l’environnement peut rester dans un état incomplet

---

### 4. Surveillance (Monitoring)

**Ce qui manque :**

* Un système pour remonter automatiquement les erreurs du backend ou du frontend
* Une centralisation des logs pour éviter d’avoir à les consulter directement sur le serveur

**Manuel vs automatisé :**

* **Manuel** : en cas de problème il faut se connecter au serveur et regarder les logs Docker pour comprendre ce qui se passe

**Points de défaillance possibles :**

* Les pannes peuvent être découvertes tardivement
* L’équipe technique n’a pas de vision claire de l’état de l’application ou des performances.

---

### 5. Sécurité

**Ce qui manque :**

* Un scan de vulnérabilités des dépendances dans la CI (par exemple avec **Trivy** ou un outil similaire)
* Une limitation du nombre de requêtes vers certains endpoints de l’API
* Une gestion plus stricte des privilèges dans les conteneurs Docker

**Manuel vs automatisé :**

* **Manuel** : la mise à jour des dépendances vulnérables repose surtout sur la surveillance des alertes GitHub ou sur `npm audit`

**Points de défaillance possibles :**

* Certaines variables sensibles pourraient avoir des valeurs par défaut peu sécurisées
* Les conteneurs exécutés avec l’utilisateur `root` augmentent les risques en cas de faille
* Sans limitation de requêtes, l’API peut être plus exposée aux abus ou au scraping

---

### 6. Documentation

**Ce qui manque :**

* Une documentation expliquant certains choix d’architecture
* Une documentation plus détaillée pour les composants frontend

**Manuel vs automatisé :**

* **Automatisé** : une partie de la documentation de l’API peut être générée automatiquement via la spec **OpenAPI**
* **Manuel** : les autres documents doivent être maintenus à la main

**Points de défaillance possibles :**

* La documentation peut rapidement devenir obsolète
* Certains fichiers peuvent ne plus correspondre exactement à l’implémentation réelle.

---

## Étape 3 : Plan d’amélioration (priorités)

### 1. Sécurité : conteneurs exécutés avec trop de privilèges

**Solution à court terme :**

* Modifier les Dockerfiles pour utiliser un utilisateur non privilégié dans les conteneurs

**Solution à long terme :**

* Mettre en place des règles plus strictes dans l’infrastructure pour limiter les permissions par défaut

**Technologies possibles :**

* Configuration Docker et outils de scan comme **Trivy**

---

### 2. Tests : absence d’exécution automatique dans la CI

**Solution à court terme :**

* Ajouter une étape dans **GitHub Actions** pour lancer les tests `pytest` et les tests frontend avant le build

**Solution à long terme :**

* Bloquer les merges si la CI échoue ou si la couverture de tests descend sous un certain seuil

**Technologies possibles :**

* GitHub Actions, pytest, outils de couverture de tests

---

### 3. Déploiement : interruptions possibles pendant la mise à jour

**Solution à court terme :**

* Améliorer le script de déploiement pour redémarrer uniquement les conteneurs nécessaires plutôt que tout arrêter

**Solution à long terme :**

* Utiliser une plateforme d’orchestration capable de faire des mises à jour progressives

**Technologies possibles :**

* Docker Compose amélioré ou solutions comme **Kubernetes**

---

### 4. Sécurité : absence de rate limiting

**Solution à court terme :**

* Activer la limitation de requêtes fournie par **Django REST Framework**

**Solution à long terme :**

* Ajouter une protection supplémentaire côté reverse proxy ou via un service cloud

**Technologies possibles :**

* Middleware DRF, configuration **NGINX** ou WAF.

---

### 5. Observabilité : manque de monitoring centralisé

**Solution à court terme :**

* Ajouter un outil simple pour remonter les erreurs backend et frontend (par exemple **Sentry**)

**Solution à long terme :**

* Mettre en place une solution plus complète avec métriques, logs centralisés et alertes

**Technologies possibles :**

* Sentry, Grafana ou Prometheus pour les métriques

---

Dans l’ensemble le projet est fonctionnel et bien structuré, mais plusieurs points pourraient être améliorés pour renforcer la fiabilité, la sécurité et le suivi de l’application sur le long terme. Certaines améliorations restent assez simples à mettre en place mais peuvent avoir un impact important sur la stabilité du système.
