# Kubernetes Security Platform

> Plateforme pédagogique de sécurisation d’un cluster Kubernetes local avec Minikube et des outils open source.

Ce dépôt présente une démarche progressive de sécurisation d’un environnement Kubernetes local. Il regroupe les manifestes, les politiques, les scénarios de test, les rapports d’analyse et la documentation produits au cours du projet.

Le projet a été réalisé dans un contexte d’apprentissage. Il permet d’explorer les principes de validation des manifestes, de **Policy as Code**, de sécurité des conteneurs, d’audit de conformité et de segmentation réseau. Il ne constitue pas une configuration prête pour une infrastructure de production.

## Objectifs

La plateforme couvre les principaux objectifs suivants :

| Domaine | Objectif | Outils ou ressources |
|---|---|---|
| Cluster local | Créer et administrer un cluster de démonstration | Minikube, Docker, kubectl |
| Déploiement | Déployer une application Nginx avec des ressources déclaratives | Namespace, ConfigMap, Secret, Deployment, Service, Ingress |
| Qualité des manifestes | Vérifier la syntaxe et les bonnes pratiques | kubeconform, kube-linter, kube-score |
| Policy as Code | Contrôler, modifier et générer des ressources Kubernetes | Kyverno |
| Vulnérabilités | Analyser une image de conteneur | Trivy |
| Conformité | Comparer la configuration aux recommandations CIS | kube-bench |
| Sécurité réseau | Restreindre les communications entre Pods | NetworkPolicy |
| Exposition | Identifier des faiblesses d’exposition du cluster | kube-hunter |

## Architecture pédagogique

Le projet suit dix phases. Les premières phases installent l’environnement et déploient l’application de démonstration. Les phases suivantes analysent les manifestes, introduisent Kyverno, testent les politiques, analysent les vulnérabilités et évaluent la configuration du cluster. Les dernières phases traitent la segmentation réseau, la détection des expositions et la documentation finale.

```text
kubernetes-security-platform/
├── phase2/                       # Déploiement de l’application Nginx
├── phase3/                       # Rapports kubeconform et kube-linter
├── phase4/                       # 13 politiques Kyverno
├── phase5/                       # Manifestes conformes et non conformes
├── phase6/                       # Rapports Trivy
├── phase7/                       # Job et rapports kube-bench
├── phase8/                       # NetworkPolicies et scénarios d’isolation
├── phase9/                       # Rapport kube-hunter
├── phase10/                      # Synthèse et copies des livrables principaux
├── README.md
└── .gitignore
```

## Prérequis

Pour reproduire les expérimentations localement, installer Docker, kubectl et Minikube. Les outils d’analyse peuvent être installés localement ou exécutés dans des conteneurs selon leur documentation officielle.

Les commandes suivantes permettent de vérifier les composants principaux :

```bash
docker --version
kubectl version --client
minikube version
```

## Démarrage du cluster

```bash
minikube start
kubectl cluster-info
kubectl get nodes
kubectl get pods -A
```

## Déploiement de l’application de démonstration

Les manifestes de la phase 2 sont appliqués de manière déclarative :

```bash
kubectl apply -f phase2/namespace.yaml
kubectl apply -f phase2/configmap.yaml
kubectl apply -f phase2/secret.yaml
kubectl apply -f phase2/deployment.yaml
kubectl apply -f phase2/service.yaml
kubectl apply -f phase2/ingress.yaml
```

Vérifier ensuite les ressources :

```bash
kubectl get all -n demo-app
kubectl get ingress -n demo-app
```

Le fichier `phase2/secret.yaml` contient uniquement des **placeholders de démonstration**. Remplacer ces valeurs localement si l’expérimentation nécessite d’autres paramètres. Ne jamais utiliser ce manifeste tel quel pour de vraies informations d’authentification.

## Installation de Kyverno et application des politiques

Après l’installation de Kyverno dans le cluster, appliquer les politiques dans l’ordre suivant :

```bash
kubectl apply -f phase4/00-namespace-security-tools.yaml
kubectl apply -f phase4/
kubectl get clusterpolicy
kubectl get clusterpolicy -o wide
```

La phase 4 contient des politiques de validation, de mutation et de génération. Elles couvrent notamment l’interdiction du tag `latest`, l’exécution non root, les conteneurs privilégiés, les contextes de sécurité, le système de fichiers racine en lecture seule, les requests/limits, les probes, les labels, les annotations, le profil seccomp, les NetworkPolicies et les LimitRanges.

## Scénarios de validation

Les manifestes de la phase 5 servent à comparer un cas conforme avec plusieurs cas volontairement non conformes : image utilisant `latest`, exécution root, conteneur privilégié, absence de limites de ressources, absence de probes et labels manquants.

```bash
kubectl apply -f phase5/01-pod-conforme.yaml
kubectl apply -f phase5/02-pod-latest.yaml
kubectl apply -f phase5/03-pod-root.yaml
kubectl apply -f phase5/04-pod-privilegie.yaml
kubectl apply -f phase5/05-pod-sans-limites.yaml
kubectl apply -f phase5/06-pod-sans-probes.yaml
kubectl apply -f phase5/07-pod-sans-labels.yaml
```

Les messages retournés par Kyverno doivent être comparés aux rapports et consignés dans la documentation de test.

## Rapports disponibles

Les résultats générés pendant les expérimentations sont conservés dans les dossiers correspondants. Le rapport Trivy est disponible en formats texte et JSON. Les rapports kube-bench comprennent une version détaillée et une version nettoyée. Les rapports kube-hunter et kube-score complètent l’évaluation de l’exposition et des bonnes pratiques.

Les résultats sont liés à un environnement local et à une date d’exécution donnée. Ils ne doivent pas être interprétés comme une certification permanente de la sécurité d’un cluster.

## Sécurité et limites

Ce dépôt est destiné à l’apprentissage et à la démonstration. Il ne fournit pas une architecture de production complète. Avant toute utilisation réelle, il faudrait notamment revoir la gestion des secrets, les droits RBAC, le chiffrement des données, la configuration du control plane, la journalisation, la supervision, la supply chain des images et la stratégie de réponse aux incidents.

Aucun mot de passe réel, token ou clé privée ne doit être ajouté au dépôt. Les valeurs du manifeste Secret ont été remplacées par des placeholders non sensibles avant publication.

## Contexte du projet

Ce travail a été réalisé dans le cadre d’un stage d’été chez Proxym Group. Il a permis de découvrir les pratiques DevSecOps appliquées à Kubernetes et de produire une documentation technique structurée autour de dix phases d’expérimentation.

## Licence

Sauf indication contraire dans un fichier dédié, ce dépôt est publié à des fins pédagogiques. Une licence open source explicite peut être ajoutée selon les conditions de diffusion souhaitées par l’auteur.
