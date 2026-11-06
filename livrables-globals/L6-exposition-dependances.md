# Livrable 6 — Exposition et cartographie des dépendances

> Partie 7 de l'énoncé.
> Rappel : **exposition ≠ risque**. Un actif exposé n'est pas forcément le plus risqué ; un actif interne peut être très critique.

## 1. Cartographie de l'exposition (7.2)

Types d'exposition : **Internet** (direct), **VPN**, **Prestataire**, **Applicative** (via une application / API), **Identité** (via un compte ou un fournisseur d'identité), **Confiance** (relation AWS / SaaS), **Interne**.

### Actifs métier (A01-A04)

Les activités héritent de l'exposition de leurs supports : A01 et A02 sont exposées via les applications et API publiées ; A03 via Microsoft 365 ; A04 via les VPN, les API AWS et les comptes d'administration. Elles ne sont pas analysées ligne à ligne ci-dessous.

### Actifs techniques

| ID | Actif | Exposition principale | Chemins d'accès possibles | Comptes / services susceptibles d'y accéder | Dépend de | Informations manquantes |
|---|---|---|---|---|---|---|
| A05 | OPNsense | Internet | Trafic entrant ; interface d'administration | Administrateurs réseau | A09 | L'interface d'administration est-elle joignable depuis Internet ou le VPN ? |
| A06 | VPN collaborateurs | Internet | Connexion depuis n'importe quelle IP | ~180 collaborateurs | A05, A14 ? | Méthode d'authentification (MFA ? Keycloak ? certificats ?) ; ressources accessibles une fois connecté |
| A07 | VPN fournisseurs | Internet, Prestataire | Postes des prestataires (non maîtrisés) | Comptes des prestataires (nominatifs ou partagés ?) | A05 | Liste des prestataires, comptes, MFA, plages horaires, ressources autorisées, journalisation |
| A08 | Liaison interne ↔ AWS | Internet (tunnel), Confiance | Compromission d'un côté → rebond de l'autre | Services hybrides | A05, A31 | Flux autorisés dans le tunnel, filtrage de chaque côté |
| A09 | Cisco | Interne | Administration depuis le LAN | Administrateurs réseau | — | Réseau d'administration dédié ? |
| A10 | Squid | Interne | Postes | Tous les utilisateurs | A05 | Contournement possible (accès direct à Internet) ? |
| A11 | DNS interne | Interne | Tout le LAN, pods (CoreDNS) | Tous les systèmes | A41 | Résolveur ouvert sur Internet ? |
| A12 | Wi-Fi entreprise | Radio locale | Proximité physique des sites | Collaborateurs | A09 | Authentification (PSK partagée ou 802.1X ?) |
| A13 | Wi-Fi visiteurs | Radio locale, Internet | Proximité physique | Visiteurs anonymes | A05 | **Preuve de l'isolation** (test d'accès au LAN) |
| A14 | Keycloak | Internet (authentification) [H] | Page de connexion, console d'administration, fédération | Tous les utilisateurs, administrateurs, applications clientes | A41, A11, base de données [V] | Console d'administration exposée ? MFA obligatoire pour les admins ? Fédérations configurées |
| A15 | AWS IAM | Internet (API AWS), Identité | Clés d'accès volées (poste, dépôt, CI), console | Administrateurs Cloud, pipeline CI, rôles des services | — | Nombre de comptes / clés, MFA, usage du compte root, clés de longue durée |
| A16 | Applications Web métier | Internet | Ingress / Load Balancer | Utilisateurs internes, clients ? [V] | A22, A17, A14, A18 | Liste des applications publiées et de leur public |
| A17 | API métier | Internet, Applicative | Ingress / Load Balancer, applications | Applications, partenaires ? [V] | A22, A14, A18, A20 | API exposées, contrôle d'autorisation |
| A18 | PostgreSQL interne | Applicative | Applications et API ; administrateurs ; VPN fournisseurs ? | Comptes applicatifs, DBA, sauvegarde | A41, A35 | Quelles applications et quelles bases ? Accès réseau direct depuis le LAN ou le VPN ? |
| A19 | RDS | Applicative, Confiance | EKS / EC2, IAM | Comptes applicatifs, IAM | A31, A15 | Accessible publiquement ? Contenu |
| A20 | Kafka | Applicative | Applications productrices / consommatrices | Comptes clients Kafka | A41 | Authentification et ACL activées ? |
| A21 | Samba | Interne, VPN | Postes, VPN collaborateurs | Collaborateurs | A41 | Droits de partage, accès depuis le VPN |
| A22 | Cluster Kubernetes | Internet (Ingress) | Applications publiées → pod | Utilisateurs des applications | A41, A26 | Isolation des namespaces, *NetworkPolicies* |
| A23 | API Kubernetes / RBAC | Identité, Applicative | Administrateurs, CI/CD, comptes de service des pods | Plateforme, CI, pods | A14 ? | API joignable depuis où ? Droits des comptes de service |
| A24 | Secrets Kubernetes | Identité | Via RBAC (`get secrets`) | Comptes disposant de ce droit | A23 | Chiffrement d'etcd ? Qui peut lire ? |
| A25 | EKS | Internet (Load Balancer), Confiance | ALB, IAM, CI/CD | Utilisateurs, pipeline | A15, A31 | Applications hébergées ; endpoint public de l'API EKS ? |
| A26 | Registre d'images | Applicative | Pipeline CI, développeurs | CI, développeurs, clusters | A28 | Registre utilisé ; qui peut pousser ? Signature des images ? |
| A27 | GitLab | À confirmer (VPN / Internet) | Navigateur, Git (SSH/HTTPS), tokens | Développeurs, prestataires ?, CI | A41, A14 | Exposition Internet ; comptes externes |
| A28 | GitLab CI/CD | Applicative | Modification d'un pipeline, d'une dépendance, d'un runner | Développeurs, mainteneurs | A27 | Runners partagés ? Variables protégées ? Approbation pour la production ? |
| A29 | Terraform | Applicative | Pipeline CI, postes des administrateurs | Administrateurs Cloud, CI | A27, A28, A15 | Emplacement et protection de l'état |
| A30 | Ansible | Interne | Poste / serveur de contrôle, dépôt GitLab | Administrateurs système | A27, A41 | Où s'exécute Ansible ? Stockage des identifiants SSH |
| A31 | VPC / Security Groups | Confiance (API AWS) | IAM | Administrateurs Cloud, Terraform | A15 | Règles ouvertes sur 0.0.0.0/0 ? |
| A32 | EC2 | À confirmer | ALB, IP publiques ?, SSH | Administrateurs | A31, A15 | Services hébergés, IP publiques |
| A33 | S3 | Internet (API S3), Identité | Politique de bucket, clés IAM | Applications, sauvegarde, CI | A15 | Buckets publics ? Versioning / Object Lock ? Qui peut supprimer ? |
| A34 | AWS Load Balancer | Internet | Trafic HTTPS | Tous | A31 | Services publiés ; filtrage applicatif (WAF) ? |
| A35 | Vault | Applicative, Identité | Applications, CI, administrateurs | Comptes / rôles Vault | A41 | Méthodes d'authentification, politiques, journal d'audit activé ? |
| A36 | Serveur de sauvegarde | Interne | Réseau interne, comptes d'administration | Administrateurs, agents de sauvegarde | A41, A33 | Isolation réseau, comptes dédiés, copie hors ligne / immuable |
| A37 | Microsoft 365 | Internet (SaaS) | Connexion depuis n'importe où | Tous les collaborateurs | A14 ? | Fédération / MFA, accès conditionnel |
| A38 | Discord | Internet (SaaS) | Comptes personnels ? | Collaborateurs | — | Statut officiel, contenu échangé |
| A39 | Postes fixes | Applicative (messagerie, Web) | Courriels, navigation, supports amovibles | Utilisateurs | A10, A42 | Niveau de mise à jour (logiciels obsolètes constatés) |
| A40 | Portables | Internet | Réseaux tiers, perte / vol | Collaborateurs nomades | A06 | Chiffrement des disques ? VPN permanent ? |
| A41 | Serveurs Linux | VPN, Prestataire, Interne | SSH depuis LAN / VPN | Administrateurs, prestataires, Ansible | A30 | Qui peut faire du SSH, depuis quelles zones ? |
| A42 | Wazuh | Interne | Agents, console | SOC / RSSI | A43 | Console exposée ? |
| A43 | OpenSearch | Interne | Dashboards | SOC / RSSI | A41 | Authentification sur les tableaux de bord |
| A44 | Prometheus / Loki / Grafana | Interne | Tableaux de bord | Plateforme | A41 | Grafana exposé ? |
| A45 | Suricata / Zeek | Interne | — | SOC | A05 / A09 | Points de capture réels |
| A46 | Greenbone | Interne | Console | RSSI | — | Accès aux rapports |
| A47 | Velociraptor | Interne | Console serveur | SOC / RSSI | Agents | Authentification forte sur la console ? |
| A48 | SonarQube / Trivy / MISP | Interne | CI, console | DevOps, SOC | A28 | Les résultats bloquent-ils le pipeline ? |

### Les cinq actifs à l'exposition la plus préoccupante

| Rang | Actif | Justification |
|---|---|---|
| 1 | **VPN fournisseurs (A07)** | Joignable depuis Internet, utilisé par des **tiers** dont les postes et les pratiques ne sont pas maîtrisés, et donnant accès à des ressources d'**administration** (serveurs Linux). Aucune information sur le MFA, les comptes nominatifs ou les ressources autorisées. C'est le chemin le plus court entre un tiers compromis et le cœur du SI. |
| 2 | **Applications Web et API publiées (A16, A17, via Ingress / ALB)** | Exposition directe, vulnérabilité critique déjà détectée sur une application exposée (événement 3, INC-002), et accès direct aux données clients en PostgreSQL. |
| 3 | **Keycloak (A14)** | Page d'authentification joignable depuis Internet [H] ; 35 échecs sur le compte `admin` en 5 minutes (événement 1) montrent qu'il est déjà ciblé. Une compromission ouvre toutes les applications fédérées. |
| 4 | **VPN collaborateurs (A06)** | Joignable depuis Internet, ~180 comptes potentiels, télétravail généralisé, et c'est par ce VPN qu'a eu lieu la connexion inhabituelle d'un compte administrateur (INC-001). Méthode d'authentification inconnue. |
| 5 | **AWS IAM (A15)** | Les API AWS sont joignables depuis Internet par conception : la seule barrière est l'identifiant. Une clé d'accès oubliée dans un dépôt ou un pipeline suffit. Aucune collecte d'audit AWS n'est décrite. |

**Écartés du top 5, mais à suivre** : OPNsense (exposé mais c'est un équipement de filtrage ; l'enjeu est son interface d'administration [V]), Microsoft 365 (exposé par nature, protégé par le fournisseur, mais l'hameçonnage reste le vecteur n°1), portables nomades.

> Ces cinq actifs ne sont **pas** tous les plus critiques : PostgreSQL, Vault ou les sauvegardes sont plus critiques mais moins exposés. Le croisement criticité × exposition est fait dans la [synthèse (L9)](L9-synthese-decisionnelle.md).

---

## 2. Dépendances des cinq actifs les plus critiques (7.3)

Actifs retenus (criticité 4, forte centralité dans la cartographie, cohérence avec la synthèse) : **Keycloak, PostgreSQL, Kubernetes (cluster + API/RBAC + secrets), GitLab (dépôts + CI/CD), AWS IAM.**

### 2.1 Keycloak (A14)

```
Keycloak
 ├── Utilisé par
 │    ├── Applications Web métier (A16) ── authentification SSO
 │    ├── API (A17) ───────────────────── validation des jetons
 │    ├── GitLab (A27) ─────────────────── SSO
 │    ├── Kubernetes (A23) ─────────────── OIDC des administrateurs [V]
 │    ├── Microsoft 365 (A37) ──────────── fédération [V]
 │    └── VPN (A06/A07) ────────────────── authentification [V]
 ├── A besoin de
 │    ├── Serveurs Linux (A41) ─────────── hébergement
 │    ├── Base de données Keycloak ─────── emplacement [V] (PostgreSQL A18 ?)
 │    ├── DNS interne (A11)
 │    └── Sauvegardes (A36) ────────────── restauration du realm [V]
 └── Collecte : événements → Wazuh (A42) [C]
```

| Élément | Détail |
|---|---|
| Applications qui l'utilisent | A16, A17, A27 ; probablement A23, A37, VPN [V] |
| Identités / rôles nécessaires | Administrateurs du realm (compte `admin` générique constaté), comptes de service des clients confidentiels |
| Flux réseau importants | Utilisateurs → Keycloak (HTTPS, Internet [H]) ; applications → Keycloak (jetons) ; Keycloak → sa base |
| Données échangées | Identifiants, jetons OIDC/SAML, rôles, attributs utilisateurs |
| Conséquences d'une indisponibilité | Plus aucune connexion aux applications métier, aux API, à GitLab ; administration bloquée si Kubernetes / VPN dépendent de Keycloak |
| Effets en cascade d'une compromission | Usurpation de n'importe quel utilisateur → A16/A17 (données clients) → GitLab (code, CI) → Kubernetes → AWS via le pipeline |

### 2.2 PostgreSQL (A18)

```
PostgreSQL
 ├── Utilisé par
 │    ├── Applications Web métier (A16) ── données clients, facturation
 │    ├── API / microservices (A17) ────── données de production
 │    └── Keycloak (A14) ───────────────── base du realm [V]
 ├── A besoin de
 │    ├── Serveurs Linux (A41)
 │    ├── Secrets (A35 / A24) ──────────── identifiants des comptes applicatifs
 │    └── Réseau interne (A09, A11)
 ├── Protégé par
 │    └── Serveur de sauvegarde (A36) ──── sauvegarde → externalisation S3 [V]
 └── Collecte : non décrite [V]
```

| Élément | Détail |
|---|---|
| Applications qui l'utilisent | Applications de gestion des clients et de production (liste exacte [V]) |
| Identités / rôles nécessaires | Comptes applicatifs (un par application ?), DBA, compte de sauvegarde, accès éventuels des prestataires [V] |
| Flux réseau importants | Pods Kubernetes → PostgreSQL (5432) ; serveur de sauvegarde → PostgreSQL ; administration (SSH / psql) |
| Données échangées | D01 données clients, D02 données de production |
| Conséquences d'une indisponibilité | Arrêt simultané des applications A16 / A17 → activités A et B à l'arrêt |
| Effets en cascade d'une compromission | Fuite ou falsification des données clients ; si la base de Keycloak y est hébergée, compromission des identités |

### 2.3 Kubernetes (A22, A23, A24)

```
Cluster Kubernetes
 ├── Exécute
 │    ├── Applications Web (A16), API (A17), services internes
 │    └── Traitements automatisés
 ├── A besoin de
 │    ├── Nœuds Linux (A41)
 │    ├── Registre d'images (A26) ──────── images à déployer
 │    ├── Secrets (A24) / Vault (A35) ──── identifiants applicatifs
 │    ├── CoreDNS (A11)
 │    └── Keycloak (A14) ───────────────── authentification des admins [V]
 ├── Piloté par
 │    ├── GitLab CI/CD (A28) ───────────── déploiements
 │    └── Administrateurs (RBAC A23)
 ├── Accède à
 │    ├── PostgreSQL (A18), Kafka (A20), Vault (A35)
 └── Collecte : audit API → Wazuh [C] ; métriques → Prometheus [C] ; logs → Loki [C]
```

| Élément | Détail |
|---|---|
| Applications qui l'utilisent | Toutes les applications conteneurisées (A16, A17) |
| Identités / rôles nécessaires | Administrateurs du cluster, compte de déploiement CI, comptes de service des pods (droits RBAC à inventorier [V]) |
| Flux réseau importants | Internet → Ingress → pods ; pods → PostgreSQL / Kafka / Vault ; CI → API Kubernetes ; nœuds → registre |
| Données échangées | Données métier en transit, secrets montés dans les pods, images |
| Conséquences d'une indisponibilité | Arrêt des applications métier exposées et internes |
| Effets en cascade d'une compromission | Pod compromis → compte de service → lecture des Secrets → accès PostgreSQL / Vault ; si droits élevés → contrôle du cluster et des nœuds |

### 2.4 GitLab (A27, A28)

```
GitLab + CI/CD
 ├── Utilisé par
 │    ├── Développeurs, administrateurs (code, IaC, playbooks)
 │    └── Prestataires ? [V]
 ├── A besoin de
 │    ├── Keycloak (A14) ───────────────── SSO
 │    ├── Serveurs Linux (A41), runners
 │    └── Sauvegardes (A36) [V]
 ├── Agit sur (privilèges)
 │    ├── Registre d'images (A26) ──────── push
 │    ├── API Kubernetes (A23) ─────────── déploiement en production
 │    ├── AWS via Terraform (A29 → A15) ── création / modification de ressources
 │    └── Ansible (A30) ─────────────────── playbooks [H]
 ├── Contrôles intégrés : SonarQube, Trivy (A48)
 └── Collecte : journaux d'audit → Wazuh [C]
```

| Élément | Détail |
|---|---|
| Applications qui l'utilisent | Chaîne de développement de toutes les applications, IaC Terraform, playbooks Ansible |
| Identités / rôles nécessaires | Comptes développeurs / mainteneurs / administrateurs, tokens personnels et de déploiement, identifiants AWS et Kubernetes du pipeline |
| Flux réseau importants | Postes → GitLab ; runners → registre, API Kubernetes, API AWS ; runners → Internet (dépendances) |
| Données échangées | Code, IaC, images, secrets de déploiement |
| Conséquences d'une indisponibilité | Plus de déploiement ni de correctif rapide (y compris pour corriger une vulnérabilité critique) ; la production continue de fonctionner |
| Effets en cascade d'une compromission | Porte dérobée dans une image → production Kubernetes ; modification Terraform → infrastructure AWS ; vol des identifiants AWS du pipeline |

### 2.5 AWS IAM (A15)

```
AWS IAM
 ├── Contrôle l'accès à
 │    ├── EKS (A25), EC2 (A32), RDS (A19)
 │    ├── S3 (A33) ──────────────────────── fichiers, SAUVEGARDES, artefacts
 │    ├── ECR (A26) ─────────────────────── images [V]
 │    └── VPC / Security Groups (A31)
 ├── Utilisé par
 │    ├── Administrateurs Cloud (console / CLI)
 │    ├── Pipeline CI/CD + Terraform (A28, A29)
 │    ├── Rôles des services (EKS, EC2)
 │    └── Serveur de sauvegarde (A36) ──── écriture vers S3 [H]
 └── Collecte : non décrite [V] (audit AWS à mettre en place)
```

| Élément | Détail |
|---|---|
| Applications qui l'utilisent | Toutes les ressources AWS ; pipeline ; sauvegardes vers S3 |
| Identités / rôles nécessaires | Compte racine, utilisateurs / rôles administrateurs, rôle du pipeline Terraform, rôles des workloads |
| Flux réseau importants | Appels aux API AWS depuis Internet, depuis le pipeline, depuis les ressources AWS |
| Données échangées | Politiques, clés d'accès, jetons temporaires |
| Conséquences d'une indisponibilité | (Rare côté AWS) impossibilité d'administrer ou de déployer ; les services en cours continuent |
| Effets en cascade d'une compromission | Accès à toutes les données AWS, ouverture de règles réseau, création de ressources, **suppression des sauvegardes S3** → perte simultanée des données et de leur copie |

## 3. Synthèse des effets en cascade

```
                ┌──────────── Keycloak ────────────┐
                │ (identités des applications)     │
                ▼                                  ▼
        Applications / API ──────────────►  GitLab + CI/CD
                │                                  │
                ▼                                  ├──► Kubernetes (production)
           PostgreSQL                              │
          (données clients)                        └──► Terraform ──► AWS IAM
                │                                                       │
                ▼                                                       ▼
        Serveur de sauvegarde ─────────────────────────────────►  S3 (sauvegardes)
```

Deux chaînes de compromission majeures apparaissent :
1. **Identité → application → données** : Keycloak → applications → PostgreSQL.
2. **Chaîne logicielle → infrastructure** : GitLab CI → Kubernetes / AWS IAM → S3, qui peut atteindre **à la fois la production et les sauvegardes**.
