# Livrable 9 — Synthèse décisionnelle

> Parties 12 et 15 de l'énoncé. La section 1 est la **synthèse d'une page** destinée à la direction ; la section 2 détaille la justification de chaque choix (partie 12).

---

## 1. Synthèse (1 page)

**Question** : quels sont les cinq actifs que NOVA SERVICES devrait examiner en priorité pour piloter sa sécurité, et pourquoi ?

### Critères de sélection

Un actif est prioritaire s'il cumule : **(1) criticité 4** (impact majeur sur une ou plusieurs activités), **(2) exposition** directe ou indirecte avérée, **(3) position centrale** dans les dépendances (sa compromission se propage), **(4) lien avec les signaux déjà observés** (connexions suspectes, vulnérabilités, alertes), **(5) déficit de visibilité** (pas de collecte décrite).

### Les cinq actifs recommandés

| # | Actif | Pourquoi en priorité |
|---|---|---|
| 1 | **Keycloak** | Porte d'entrée unique de toutes les applications ; déjà ciblé (35 échecs sur `admin`) ; compromission = accès à toutes les activités. |
| 2 | **PostgreSQL** (+ ses sauvegardes) | Concentre les données clients et de production ; impact RGPD et financier direct ; **aucun journal de base collecté** aujourd'hui. |
| 3 | **Applications et API métier exposées sur Kubernetes** (cluster, RBAC, secrets) | Exposées sur Internet avec une vulnérabilité critique déjà détectée ; un pod compromis peut atteindre les secrets et la base. |
| 4 | **GitLab et la chaîne CI/CD** | Tout ce qui y est modifié part en production (Kubernetes, AWS via Terraform) ; détient des identifiants à privilèges. |
| 5 | **AWS IAM** | Contrôle tout le Cloud, **y compris les sauvegardes dans S3** ; joignable depuis Internet par conception ; **aucun audit AWS collecté**. |

**Juste en dessous, à traiter rapidement** : le **VPN fournisseurs** (exposition la plus préoccupante, tiers non maîtrisés — vérification immédiate), les **sauvegardes** (capacité de reprise non démontrée) et **Vault**.

### Pourquoi pas les autres ?

OPNsense est exposé mais c'est un équipement de protection, surveillé par Wazuh ; les postes sont nombreux mais leur compromission individuelle a un impact contenu (sauf propagation, qui passe justement par les cinq actifs ci-dessus) ; les outils de supervision sont importants mais ne portent pas directement les activités métier.

### Principales incertitudes

1. **Aucun propriétaire désigné** pour les 48 actifs.
2. Répartition des applications et des bases entre l'**interne et AWS**.
3. Emplacement de **tous les secrets** (Vault seul, ou aussi Kubernetes, CI, Terraform ?).
4. **MFA** sur Keycloak (admin), VPN et AWS : non documenté.
5. **Sauvegardes** : isolation, immuabilité et tests de restauration non documentés.
6. **Collecte absente** pour PostgreSQL, AWS, Vault, VPN, sauvegardes, Microsoft 365.
7. Procédure de gestion des incidents non fournie.

### Premières actions à envisager

| Action | Actif | Échéance suggérée |
|---|---|---|
| Désigner un propriétaire pour chaque actif de criticité 4 | Tous | Immédiat |
| Vérifier le MFA et l'exposition de la console d'administration Keycloak ; supprimer l'usage du compte générique `admin` | Keycloak | Immédiat |
| Qualifier et traiter la vulnérabilité critique de l'application exposée (mitigation puis correctif) | Applications / Kubernetes | Immédiat |
| Auditer le VPN fournisseurs (comptes nominatifs, MFA, périmètre) | VPN fournisseurs | Court terme |
| Activer la collecte des journaux PostgreSQL et de l'audit AWS vers Wazuh | PostgreSQL, AWS IAM | Court terme |
| Réaliser un test de restauration documenté et vérifier l'immuabilité des sauvegardes S3 | Sauvegardes | Court terme |
| Inventorier les comptes à privilèges et les tokens (Keycloak, GitLab, AWS, Kubernetes, Vault) | Identités | Court terme |
| Formaliser la procédure de gestion des incidents | Organisation | Court terme |

---

## 2. Justification détaillée des priorités (partie 12)

### 1 — Keycloak (A14)

| Critère | Analyse |
|---|---|
| Criticité | 4 (C4 / I4 / D4 / M4) |
| Exposition | Internet pour l'authentification [H] ; console d'administration à vérifier [V] |
| Données traitées | Comptes, rôles, groupes, secrets MFA (D04) |
| Activités dépendantes | A, B, C (fédération [V]), D |
| Conséquences d'un incident | Panne : plus aucun accès aux applications. Compromission : usurpation de tout utilisateur, création de comptes administrateurs → propagation vers applications, GitLab, Kubernetes |
| Besoins de surveillance | Échecs d'authentification, connexions admin, changements de rôles, MFA, configuration |
| Incertitudes | Exposition de la console, MFA des admins, emplacement de la base Keycloak, utilisation par le VPN |
| Pourquoi avant les autres | C'est la **racine de confiance** : protéger les applications sans protéger Keycloak est inutile. Signal d'attaque déjà observé. |

### 2 — PostgreSQL (A18) et ses sauvegardes

| Critère | Analyse |
|---|---|
| Criticité | 4 (C4 / I4 / D4 / M4) |
| Exposition | Indirecte via applications et API ; accès d'administration via VPN (INC-001) |
| Données traitées | Données clients (personnelles), contrats, facturation, production (D01, D02) |
| Activités dépendantes | A — Gestion des clients, B — Production |
| Conséquences d'un incident | Fuite (RGPD, confiance), falsification (fraude), destruction (arrêt de deux activités) |
| Besoins de surveillance | Accès à privilèges, accès inhabituels, opérations destructrices, état des sauvegardes |
| Incertitudes | Liste des bases et applications, accès directs depuis le LAN / VPN, journalisation activée |
| Pourquoi avant les autres | C'est l'actif dont la perte a **le plus de valeur métier**, et c'est un **angle mort** de la collecte actuelle. |

### 3 — Applications / API métier sur Kubernetes (A16, A17, A22-A24)

| Critère | Analyse |
|---|---|
| Criticité | 4 |
| Exposition | **Directe** (Ingress / Load Balancer) ; vulnérabilité critique déjà détectée |
| Données traitées | Données clients et de production en transit ; secrets montés dans les pods |
| Activités dépendantes | A, B |
| Conséquences d'un incident | Exploitation → accès aux données, rebond vers la base, les secrets et potentiellement le cluster |
| Besoins de surveillance | Changements RBAC, lecture de secrets, `exec`, images déployées, trafic sortant des pods |
| Incertitudes | Répartition interne / EKS, droits des comptes de service, *NetworkPolicies* |
| Pourquoi avant les autres | C'est le **point de contact direct** entre Internet et les données métier, avec une faille connue. |

### 4 — GitLab et CI/CD (A27, A28)

| Critère | Analyse |
|---|---|
| Criticité | 4 |
| Exposition | À confirmer (VPN / Internet) ; tokens et dépendances logicielles |
| Données traitées | Code, IaC, secrets de déploiement (D05, D06, D07) |
| Activités dépendantes | D (administration), B (déploiement des applications) |
| Conséquences d'un incident | Porte dérobée en production, modification de l'infrastructure AWS, vol des identifiants AWS / Kubernetes |
| Besoins de surveillance | Permissions, tokens, modifications de pipelines, déploiements en production |
| Incertitudes | Exposition, runners partagés, approbations pour la production, variables protégées |
| Pourquoi avant les autres | **Pont à privilèges** vers Kubernetes et AWS : un seul point de compromission touche plusieurs zones. |

### 5 — AWS IAM (A15)

| Critère | Analyse |
|---|---|
| Criticité | 4 |
| Exposition | Internet (API AWS), protection uniquement par les identifiants |
| Données traitées | Identités, politiques, clés d'accès ; accès à S3 (sauvegardes), RDS, EKS |
| Activités dépendantes | D, et toutes les activités hébergées dans AWS |
| Conséquences d'un incident | Accès à toutes les données Cloud ; **suppression des sauvegardes S3** ; ouverture de flux réseau |
| Besoins de surveillance | Changements de politiques et d'identités, compte racine, création de clés, politiques S3 |
| Incertitudes | Nombre de comptes et clés, MFA, séparation des droits sur les sauvegardes, **absence de collecte d'audit** |
| Pourquoi avant les autres | Un compromis IAM peut détruire **à la fois** la production Cloud et la capacité de reprise ; il n'est aujourd'hui pas observé. |

### Traçabilité de la décision

Ce choix s'appuie sur : la criticité ([L4](L4-evaluation-criticite.md)), l'exposition et les dépendances ([L6](L6-exposition-dependances.md)), les besoins de surveillance ([L7](L7-besoins-et-matrice-pilotage.md)) et les signaux de la mise en situation ([01](01-analyse-initiale-SI.md), [L8](L8-analyse-incidents.md)). Il sera **révisé** dès que la DSI aura levé les incertitudes listées ci-dessus — en particulier si l'audit du VPN fournisseurs ou des sauvegardes révèle un écart majeur, ces actifs remonteraient dans le top 5.
