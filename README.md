 VPROFILE-ACTION – CI/CD pour l’application VProfile (Java, Docker, AWS ECR)

Ce dépôt contient le **workflow GitHub Actions** destiné à automatiser le cycle de vie de l’application Java VProfile : tests, analyse de style, build Docker et déploiement sur AWS ECR. Il s’intègre parfaitement avec l'infrastructure EKS provisionnée par le projet [`iac-vprofile`](https://github.com/ton-repo/iac-vprofile).

---

## 🎯 Objectifs du pipeline

- Vérifier la qualité du code (tests unitaires + checkstyle)
-  Exécuter une analyse SonarQube (désactivée dans ce repo)
- Construire une image Docker pour l’application
- Publier l’image dans un **AWS Elastic Container Registry (ECR)**

---

## 📁 Structure du dépôt

```

vprofile-action/
├── .github/workflows/
│   └── main.yml               # Pipeline GitHub Actions
├── src/                       # Code source Java
├── files/                     # Fichiers de configuration Tomcat
├── kubernetes/                # YAML de déploiement Kubernetes
├── Dockerfile                 # Dockerfile pour l'application
├── pom.xml                    # Build Maven
└── README.md

````

---

## ⚙️ Outils utilisés

| Outil                  | Rôle                                |
|------------------------|-------------------------------------|
| Maven                  | Build Java + tests + checkstyle     |
| Docker                 | Construction d’image                |
| AWS ECR                | Registry d’images Docker            |
| GitHub Actions         | Orchestration du pipeline CI/CD     |

---

## 🔁 Pipeline GitHub Actions – `.github/workflows/main.yml`

### 📌 Déclencheur

```yaml
on:
  workflow_dispatch
````

Le pipeline s’exécute **manuellement** depuis GitHub (menu Actions → bouton "Run workflow").

---

### 🧪 Job 1 : `Testing`

| Étape        | Description                                   |
| ------------ | --------------------------------------------- |
| ✅ Checkout   | Récupération du code source                   |
| ✅ Maven test | Exécution des tests unitaires                 |
| ✅ Checkstyle | Vérification des conventions de code          |
| 🔒 SonarQube | Désactivé car non utilisé ici (code commenté) |

---

### 🏗️ Job 2 : `BUILD_AND_PUBLISH`

Ce job dépend de `Testing` (`needs: Testing`) et :

| Étape                  | Description                                                                  |
| ---------------------- | ---------------------------------------------------------------------------- |
| ✅ Checkout             | Code source                                                                  |
| 🐳 Docker build & push | Publie l’image vers **ECR** avec tags `latest` et `${{ github.run_number }}` |

> Utilise l'action [`appleboy/docker-ecr-action`](https://github.com/appleboy/docker-ecr-action)

---

## 🔐 Secrets GitHub nécessaires

Ajoute ces secrets dans **Settings > Secrets > Actions** :

| Secret                                    | Utilité                          |
| ----------------------------------------- | -------------------------------- |
| `AWS_ACCESS_KEY_ID`                       | Authentification AWS             |
| `AWS_SECRET_ACCESS_KEY`                   | Authentification AWS             |
| `REGISTRY`                                | URL de ton registre ECR          |
| `ECR_REPOSITORY`                          | Nom du repo (ex: `vprofileapp`)  |
| *(facultatif)* `SONAR_TOKEN`, `SONAR_URL` | Pour activer SonarQube plus tard |

---

## 📦 Déploiement sur EKS

Une fois l’image buildée et publiée, elle peut être utilisée dans ton cluster Kubernetes via les fichiers de déploiement présents dans `kubernetes/` :

```bash
kubectl apply -f kubernetes/vproappdep.yml
kubectl apply -f kubernetes/vproapp-service.yml
```

> Ces commandes sont exécutées **depuis le cluster provisionné avec `iac-vprofile`**.

---

## 🧼 Nettoyage

Pour supprimer les pods et services :

```bash
kubectl delete -f kubernetes/
```

---

## 🧠 À venir (améliorations possibles)

* Ajout d’un déploiement automatique sur EKS
* Analyse de couverture avec Jacoco
* Intégration SonarQube (via SonarCloud ou instance locale)

---

## 👤 Auteur

**Jouneid Guefif**
CI/CD DevOps Java avec GitHub Actions, Maven, Docker & AWS


```
