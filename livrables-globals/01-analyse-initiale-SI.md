# Partie 2 — Première analyse du SI de NOVA SERVICES

> Conventions : voir [00-README.md](00-README.md). Les identifiants `Axx` renvoient au [registre des actifs](L2-registre-actifs.md).

---

## Exercice 1 — Identifier les actifs

| Actif | Fonction | Criticité estimée |
|---|---|---|
| Keycloak (A14) | Gestion centrale des identités, SSO, MFA, fédération | Élevée |
| Applications Web métier (A16) | Gestion des clients, contrats, facturation, suivi des prestations | Élevée |
| PostgreSQL interne (A18) | Stockage des données de plusieurs applications métier | Élevée |
| Cluster Kubernetes (A22-A24) | Exécution des applications Web, API, microservices | Élevée |
| GitLab + CI/CD (A27-A28) | Code source, pipelines, déploiement en production | Élevée |
| AWS IAM (A15) | Permissions d'accès à toutes les ressources AWS | Élevée |
| HashiCorp Vault (A35) | Stockage des mots de passe, tokens et clés applicatives | Élevée |
| Serveur de sauvegarde + S3 (A36, A33) | Sauvegardes des données et systèmes critiques | Élevée |
| Pare-feu OPNsense (A05) | Filtrage des flux entre Internet et le réseau interne | Élevée |
| VPN fournisseurs (A07) | Accès des prestataires pour la maintenance | Moyenne (exposition forte) |
| VPN collaborateurs (A06) | Accès distant des télétravailleurs | Moyenne |
| Apache Kafka (A20) | Transport des événements entre applications | Moyenne |
| Serveur de fichiers Samba (A21) | Documents internes partagés | Moyenne |
| Microsoft 365 (A37) | Messagerie, stockage et partage documentaire | Moyenne |
| Wazuh / OpenSearch (A42-A43) | Collecte, stockage et analyse des événements de sécurité | Moyenne |
| Postes et portables (A39-A40) | Outils de travail des 180 collaborateurs | Moyenne |
| Wi-Fi visiteurs (A13) | Accès Internet isolé pour les invités | Faible (si l'isolation est vérifiée) |

---

## Exercice 2 — Identifier les actifs exposés

### Exposition directe (joignable depuis Internet)

| Actif | Raison |
|---|---|
| Pare-feu OPNsense (A05) | Interface publique de l'entreprise, reçoit tout le trafic entrant [C] |
| VPN collaborateurs (A06) | Point de terminaison joignable depuis Internet par construction [C] |
| VPN fournisseurs (A07) | Idem, utilisé en plus par des tiers dont l'entreprise ne maîtrise pas les postes [C] |
| Ingress Kubernetes + applications Web / API publiées (A16, A17, A22) | « Plusieurs applications accessibles depuis Internet » [C] |
| AWS Load Balancer (A34) | Distribue les connexions entrantes vers les services AWS [C] |
| Keycloak — pages d'authentification (A14) | Nécessaire pour authentifier les utilisateurs des applications exposées et la fédération [H] |
| Points d'accès aux API AWS / console (A15, A33) | Les API AWS (IAM, S3…) sont joignables depuis Internet par conception ; la protection repose sur les identifiants et les politiques [C]/[V] |
| Microsoft 365, Discord (A37, A38) | Services SaaS accessibles depuis Internet par nature [C] |
| Ordinateurs portables nomades (A40) | Connectés à des réseaux tiers (domicile, clients, Wi-Fi publics) [C] |
| Terminaison de la liaison interne ↔ AWS (A08) | Tunnel VPN site à site établi via Internet [C]/[V] |

### Exposition indirecte (atteignable après un premier rebond)

| Actif | Chemin d'accès indirect |
|---|---|
| PostgreSQL interne et RDS (A18, A19) | Via une application ou une API exposée compromise (injection SQL, vol de secrets) |
| API Kubernetes, RBAC, Secrets (A23, A24) | Via un pod compromis disposant d'un compte de service trop permissif |
| Kafka (A20) | Via une application productrice / consommatrice compromise |
| HashiCorp Vault (A35) | Via les applications ou les pipelines qui y lisent des secrets |
| GitLab et CI/CD (A27, A28) | Via VPN, via un token volé, via une dépendance logicielle piégée ; exposition directe possible [V] |
| Registre d'images (A26) | Via le pipeline CI/CD ou des identifiants de push |
| Serveur de fichiers Samba (A21) | Via un poste compromis ou une session VPN |
| Postes utilisateurs fixes (A39) | Via la messagerie (hameçonnage) et la navigation Web, même filtrée par Squid |
| Serveurs Linux (A41) | Via le VPN fournisseurs / collaborateurs, via SSH depuis un poste d'administration |
| EKS, EC2, S3, RDS (A25, A32, A33, A19) | Via des identifiants IAM volés, la liaison VPN ou les pipelines Terraform |
| Ansible, Terraform (A29, A30) | Via GitLab (code et variables) ou le compte qui les exécute |

### Non exposé directement

| Actif | Remarque |
|---|---|
| Serveur de sauvegarde (A36) | Interne, mais cible privilégiée des rançongiciels une fois dans le réseau |
| DNS interne (A11) | Interne |
| Wazuh, OpenSearch, Prometheus / Grafana / Loki (A42-A44) | Interne — à vérifier pour les tableaux de bord [V] |
| Suricata / Zeek (A45), Greenbone (A46), Velociraptor (A47) | Interne |
| Routeurs / commutateurs Cisco (A09) | Interfaces d'administration supposées internes [V] |
| Wi-Fi entreprise (A12) | Exposition radio locale (parking, voisinage), pas d'exposition Internet |
| Wi-Fi visiteurs (A13) | Accès Internet isolé ; exposition radio locale |

> **À retenir** : « non exposé directement » ne signifie pas « non atteignable ». La plupart des actifs les plus critiques (PostgreSQL, Vault, sauvegardes, API Kubernetes) sont dans la catégorie indirecte.

---

## Exercice 3 — Identifier les données sensibles

| Système | Données sensibles contenues ou accessibles |
|---|---|
| Applications métier + PostgreSQL / RDS | **Données clients** (coordonnées, contrats, facturation, historique), **données de production** (projets, affectations de collaborateurs) |
| Keycloak | **Comptes utilisateurs**, rôles, groupes, empreintes de mots de passe, secrets OTP/MFA, configuration de fédération |
| AWS IAM | Identités et politiques donnant accès à **toutes les données stockées dans AWS** |
| HashiCorp Vault | **Secrets techniques** : mots de passe de bases, tokens, clés d'API, certificats |
| Secrets Kubernetes / variables CI GitLab / état Terraform | **Secrets techniques** potentiellement dupliqués hors de Vault [V] |
| GitLab | Code source (propriété intellectuelle), configurations, IaC, tokens d'accès |
| Serveur de fichiers Samba, SharePoint | **Documents internes** : procédures, documents commerciaux, RH éventuellement [V] |
| Microsoft 365 / Discord | Messagerie professionnelle : échanges avec les clients, pièces jointes |
| Amazon S3 | **Données stockées dans AWS** : fichiers, **sauvegardes**, artefacts |
| Serveur de sauvegarde | Copie de l'ensemble des données critiques → même sensibilité que les originaux |
| Kafka | **Données applicatives** et événements utilisateurs / métier en transit |
| Wazuh / OpenSearch / Loki | **Données de sécurité** : journaux d'authentification, adresses IP, parfois données personnelles dans les logs |
| Greenbone / OpenVAS | Inventaire des vulnérabilités — très utile à un attaquant |
| Postes et portables | Copies locales de documents, sessions ouvertes, identifiants mis en cache |

---

## Exercice 4 — Identifier les sources de données

| Source | Événements intéressants pour la sécurité |
|---|---|
| **OPNsense** | Connexions bloquées / autorisées (volumétrie, pays, ports) ; modification des règles de filtrage ; connexion à l'interface d'administration ; établissement / échec des tunnels VPN |
| **Keycloak** | Échecs d'authentification (rafales, même IP, même compte) ; connexion d'un compte administrateur ; création / suppression de compte ; modification de rôle ou de groupe ; désactivation du MFA ; changement de configuration du realm ou de la fédération |
| **Linux** | Connexions SSH (réussies / échouées, IP source) ; utilisation de `sudo` ; création d'utilisateur ou de clé SSH ; modification de fichiers sensibles (`/etc/passwd`, `sshd_config`, crontab) ; arrêt d'un service ou de l'agent Wazuh |
| **Kubernetes** | Journaux d'audit de l'API (création de pods privilégiés, `exec` dans un pod) ; modification de `Role` / `ClusterRoleBinding` ; lecture de `Secrets` ; déploiement d'une image non issue du registre autorisé ; création de namespace ou de compte de service |
| **Kafka** | Échecs d'authentification des clients ; modification des ACL ; création / suppression de topics ; client inconnu se connectant à un topic sensible ; volumétrie de consommation anormale |
| **GitLab** | Connexion (notamment admin) ; création de token personnel ou de déploiement ; modification des permissions d'un projet ; modification de `.gitlab-ci.yml` ; exécution d'un pipeline de déploiement en production ; désactivation d'une protection de branche |
| **AWS** | Création / modification d'utilisateurs, rôles ou politiques IAM ; création de clés d'accès ; modification des Security Groups / VPC ; création de nouvelle ressource (EC2, bucket) ; changement de la politique d'accès d'un bucket S3 ; connexion console du compte root |
| **PostgreSQL** | Échecs d'authentification ; connexion d'un compte à privilèges hors des horaires / sources habituels ; modification des rôles et des droits (`GRANT`) ; requêtes massives (export) ; suppression de tables ou de bases (`DROP`, `TRUNCATE`) |
| **Suricata** | Alertes de signatures (exploitation, scans) ; flux vers des IP / domaines malveillants (IoC MISP) ; trafic sortant inhabituel (exfiltration, C2) ; protocoles inattendus sur un port |

---

## Exercice 5 — Identifier les outils

| Besoin | Outil |
|---|---|
| Rechercher des vulnérabilités | Greenbone / OpenVAS |
| Détecter des activités réseau suspectes | Suricata / Snort (IDS) — complété par Zeek pour l'analyse des communications |
| Centraliser les événements de sécurité | Wazuh (stockage et recherche dans OpenSearch) |
| Collecter les métriques | Prometheus |
| Visualiser les indicateurs | Grafana (métriques) / OpenSearch Dashboards (événements de sécurité) |
| Analyser le code | SonarQube |
| Analyser les images Docker | Trivy |
| Faire de l'investigation sur un poste | Velociraptor |
| Gérer les indicateurs de compromission | MISP |
| Centraliser les logs applicatifs | Loki |

---

## 2.19 Mise en situation — analyse des 5 événements

### Événement 1 — 35 échecs d'authentification sur le compte admin de Keycloak en 5 minutes

| Question | Réponse |
|---|---|
| Actif concerné | Keycloak (A14), et indirectement toutes les applications qui en dépendent (A16, A17, A27, accès Kubernetes) |
| Menace | Attaquant externe réalisant une attaque par force brute ou par *credential stuffing* ; éventuellement un script interne mal configuré |
| Vulnérabilité possible | Compte d'administration au nom générique et prévisible (`admin`), console d'administration joignable depuis Internet, absence de verrouillage / limitation de débit, absence de MFA sur le compte admin [V] |
| Exposition | **Directe** si la console d'administration est publiée sur Internet [V] ; sinon indirecte (depuis le réseau interne / VPN) |
| Risque | Prise de contrôle de Keycloak → création de comptes, élévation de droits, accès à toutes les applications fédérées : **compromission en cascade** |
| Source de confirmation | Événements Keycloak (`LOGIN_ERROR`, IP source, client), journaux OPNsense / Ingress (origine du trafic), journaux du reverse proxy |
| Outil d'analyse | Wazuh + OpenSearch (corrélation par IP / compte), MISP (réputation de l'IP) |

### Événement 2 — Connexions SSH sur un serveur Linux depuis une IP inhabituelle

| Question | Réponse |
|---|---|
| Actif concerné | Serveur Linux (A41) et les services qu'il héberge (à identifier : PostgreSQL ? GitLab ?) |
| Menace | Attaquant ayant volé une clé SSH ou un mot de passe ; prestataire légitime utilisant une nouvelle IP ; mouvement latéral depuis une machine interne compromise |
| Vulnérabilité possible | Authentification SSH par mot de passe autorisée, clé SSH partagée ou non révoquée, SSH accessible depuis une zone trop large (VPN fournisseurs, tout le LAN) |
| Exposition | Indirecte (via VPN ou réseau interne) — directe si SSH est publié sur Internet, ce qui serait un écart majeur [V] |
| Risque | Accès non autorisé au serveur, élévation de privilèges (`sudo`), accès aux données ou rebond vers d'autres serveurs |
| Source de confirmation | `auth.log` / journal SSH (réussite ou échec, compte, méthode d'authentification), journaux `sudo`, journaux VPN (attribution de l'IP à un compte), flux OPNsense |
| Outil d'analyse | Wazuh (agent Linux), Zeek (communications de la machine), Velociraptor (investigation si connexion réussie) |

### Événement 3 — Greenbone détecte une vulnérabilité critique sur une application exposée

| Question | Réponse |
|---|---|
| Actif concerné | Application Web métier / API exposée (A16 / A17) et son serveur ou pod |
| Menace | Attaquant opportuniste scannant Internet à la recherche de vulnérabilités connues |
| Vulnérabilité | Celle remontée par le scanner (version obsolète, CVE critique) — à confirmer : faux positif possible |
| Exposition | **Directe** (Internet via Ingress / Load Balancer) |
| Risque | Exécution de code ou accès aux données → atteinte aux données clients (PostgreSQL), rebond dans le cluster |
| Source de confirmation | Rapport Greenbone (CVE, preuve), inventaire de versions, journaux applicatifs et d'accès (tentatives d'exploitation déjà observées ?), alertes Suricata |
| Outil d'analyse | Greenbone / OpenVAS, Trivy (si l'application est conteneurisée), Loki / OpenSearch (logs d'accès), Suricata |

### Événement 4 — Un pipeline GitLab déploie une nouvelle image Docker en production

| Question | Réponse |
|---|---|
| Actif concerné | GitLab CI/CD (A28), registre d'images (A26), cluster Kubernetes de production (A22) |
| Menace | Événement **probablement légitime** ; menace possible : développeur malveillant, token CI volé, dépendance piégée (attaque de la chaîne d'approvisionnement) |
| Vulnérabilité possible | Absence de validation / approbation pour la production, branches non protégées, résultat Trivy ignoré, token de déploiement trop permissif |
| Exposition | Indirecte (via GitLab) |
| Risque | Déploiement d'un code ou d'une image non autorisé ou vulnérable en production |
| Source de confirmation | Journaux d'audit GitLab (qui a déclenché, quel commit, approbations), résultat Trivy / SonarQube du pipeline, audit Kubernetes (image déployée, empreinte) |
| Outil d'analyse | GitLab (audit), Trivy, Wazuh / OpenSearch (corrélation avec un changement planifié) |

### Événement 5 — Suricata détecte un comportement réseau suspect depuis un serveur Kubernetes

| Question | Réponse |
|---|---|
| Actif concerné | Nœud ou pod du cluster Kubernetes (A22), potentiellement l'API Kubernetes et les secrets (A23, A24) |
| Menace | Conteneur compromis (via une vulnérabilité applicative ou une image piégée) communiquant avec un serveur de commande, minant de la cryptomonnaie ou scannant le réseau |
| Vulnérabilité possible | Image vulnérable, pod privilégié, absence de *NetworkPolicy* limitant les flux sortants, compte de service trop permissif |
| Exposition | Indirecte (le cluster reçoit du trafic Internet via l'Ingress) |
| Risque | Compromission d'un pod → vol de secrets → accès à PostgreSQL / Vault / API Kubernetes ; exfiltration de données |
| Source de confirmation | Alerte Suricata (signature, IP de destination), Zeek (détail des connexions), audit Kubernetes (`exec`, nouveaux pods), logs du pod, IoC MISP |
| Outil d'analyse | Suricata + Zeek, MISP, Wazuh, Velociraptor (investigation sur le nœud) |
