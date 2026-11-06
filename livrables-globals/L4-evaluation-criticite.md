# Livrable 4 — Évaluation de la criticité des actifs

> Partie 6 de l'énoncé. Échelle 1 (faible) à 4 (critique).
> **Règle de décision** : la criticité retenue est la note la plus élevée parmi les quatre critères (la conséquence la plus grave l'emporte) — **pas de moyenne**. Les notes sont provisoires et seront révisées lorsque la DSI aura levé les points [V].

Critères :
- **C** — Confidentialité : conséquences d'un accès non autorisé.
- **I** — Intégrité : conséquences d'une modification ou destruction non autorisée.
- **D** — Disponibilité : conséquences d'une interruption de service.
- **M** — Impact métier : conséquences sur les activités, les clients, les obligations, la continuité.

## 1. Actifs proposés par l'énoncé — évaluation détaillée

| Actif | C | I | D | M | Criticité retenue | Justification |
|---|---|---|---|---|---|---|
| **Keycloak** (A14) | 4 | 4 | 4 | 4 | **4 — Critique** | C : contient comptes, rôles et secrets MFA ; un accès admin permet d'usurper n'importe quel utilisateur. I : modifier un rôle donne accès à toutes les applications fédérées. D : point unique d'authentification — panne = applications, API et GitLab inaccessibles. M : touche les quatre activités simultanément. |
| **PostgreSQL** (A18) | 4 | 4 | 4 | 4 | **4 — Critique** | La criticité vient des **données hébergées** : données clients et de production de plusieurs applications métier. C : fuite de données personnelles (RGPD). I : facturation / affectations faussées. D : arrêt des activités A et B. *À réévaluer base par base quand l'inventaire des bases sera fourni [V].* |
| **GitLab** (A27 dépôts / A28 CI/CD) | 3 / 4 | 4 / 4 | 3 / 3 | 4 / 4 | **4 — Critique** | I : une modification du code ou du pipeline est déployée automatiquement en production (Kubernetes, AWS via Terraform). C : code source et, côté CI, variables contenant des secrets (d'où C = 4 pour A28). D : indisponibilité = plus de déploiement ni de correctif rapide, mais la production continue de tourner (D = 3). |
| **Amazon S3** (A33) | 4 | 4 | 3 | 4 | **4 — Critique** | La criticité dépend des objets stockés : S3 contient des **sauvegardes** [C], des fichiers et des artefacts. C : copie des données sensibles ; un bucket rendu public est une fuite massive. I : suppression / chiffrement des sauvegardes = plus de reprise possible. D : 3 car l'indisponibilité ponctuelle n'arrête pas l'activité courante. |
| **Wi-Fi visiteurs** (A13) | 1 | 1 | 1 | 1 | **1 — Faible** *(conditionnel)* | Réseau qui ne fournit qu'un accès Internet isolé [C] : aucune donnée interne, solution de remplacement simple (partage de connexion). **Cette note n'est valable que si l'isolation est effective** (aucune route vers le LAN, le VPN ou les interfaces d'administration). Si l'isolation n'est pas vérifiée → réévaluer à 3, car le réseau deviendrait un point d'entrée physique anonyme. |

## 2. Évaluation de l'ensemble du registre

| ID | Actif | C | I | D | M | Crit. | Justification synthétique |
|---|---|---|---|---|---|---|---|
| A01 | Gestion des clients | 4 | 4 | 3 | 4 | **4** | Données personnelles clients ; facturation = chiffre d'affaires |
| A02 | Production des services | 3 | 4 | 4 | 4 | **4** | Cœur de métier ; arrêt = prestations non pilotées, pénalités |
| A03 | Communication et collaboration | 3 | 3 | 3 | 3 | **3** | Mode dégradé possible ; vecteur d'hameçonnage |
| A04 | Administration du SI | 4 | 4 | 3 | 4 | **4** | Conditionne la sécurité et la reprise de toutes les activités |
| A05 | Pare-feu OPNsense | 2 | 4 | 4 | 4 | **4** | Point de passage unique ; règles modifiées = exposition générale |
| A06 | VPN collaborateurs | 3 | 3 | 3 | 3 | **3** | Télétravail bloqué si indisponible ; accès au LAN si abusé |
| A07 | VPN fournisseurs | 3 | 3 | 2 | 3 | **3** | Accès d'administration à des tiers ; indisponibilité = maintenance différée |
| A08 | Liaison interne ↔ AWS | 3 | 3 | 3 | 3 | **3** | Rupture des applications hybrides ; pont entre environnements |
| A09 | Routeurs / commutateurs Cisco | 2 | 3 | 3 | 3 | **3** | Coupure d'un site ; segmentation contournable (redondance à confirmer) |
| A10 | Proxy Squid | 2 | 2 | 2 | 2 | **2** | Contournable temporairement |
| A11 | DNS interne | 2 | 3 | 3 | 3 | **3** | Panne = services internes injoignables ; falsification = redirection |
| A12 | Wi-Fi entreprise | 2 | 2 | 2 | 2 | **2** | Alternative filaire |
| A13 | Wi-Fi visiteurs | 1 | 1 | 1 | 1 | **1** | Conditionnel à l'isolation (voir ci-dessus) |
| A14 | Keycloak | 4 | 4 | 4 | 4 | **4** | Voir §1 |
| A15 | AWS IAM | 4 | 4 | 3 | 4 | **4** | Contrôle d'accès à toutes les ressources Cloud, sauvegardes S3 comprises |
| A16 | Applications Web métier | 4 | 4 | 4 | 4 | **4** | Supportent directement A01 et A02 |
| A17 | API métier / microservices | 4 | 4 | 3 | 4 | **4** | Accès programmatique massif aux données |
| A18 | PostgreSQL interne | 4 | 4 | 4 | 4 | **4** | Voir §1 |
| A19 | Amazon RDS | 3 | 3 | 3 | 3 | **3** | Provisoire — contenu des bases à confirmer |
| A20 | Apache Kafka | 3 | 3 | 3 | 3 | **3** | Désynchronisation / falsification d'événements entre applications |
| A21 | Serveur de fichiers Samba | 3 | 3 | 2 | 3 | **3** | Documents internes ; cible rançongiciel |
| A22 | Cluster Kubernetes interne | 3 | 4 | 4 | 4 | **4** | Exécute les applications métier |
| A23 | API Kubernetes / RBAC | 3 | 4 | 3 | 4 | **4** | Contrôle total du cluster |
| A24 | Secrets Kubernetes | 4 | 3 | 2 | 4 | **4** | Identifiants d'accès aux bases et API |
| A25 | Amazon EKS | 3 | 3 | 3 | 3 | **3** | Provisoire — applications hébergées à confirmer |
| A26 | Registre d'images | 2 | 3 | 3 | 3 | **3** | Images piégées ; conditionné à un déploiement |
| A27 | GitLab — dépôts | 3 | 4 | 3 | 4 | **4** | Voir §1 |
| A28 | GitLab CI/CD | 4 | 4 | 3 | 4 | **4** | Voir §1 |
| A29 | Terraform | 3 | 3 | 2 | 3 | **3** | Risque porté par A28 / A15 ; état à localiser |
| A30 | Ansible | 3 | 4 | 2 | 4 | **4** | Privilèges sur tous les serveurs Linux |
| A31 | Amazon VPC / Security Groups | 2 | 3 | 3 | 3 | **3** | Règle ouverte = exposition directe des ressources AWS |
| A32 | Amazon EC2 | 2 | 2 | 2 | 2 | **2** | Provisoire — services hébergés non identifiés |
| A33 | Amazon S3 | 4 | 4 | 3 | 4 | **4** | Voir §1 |
| A34 | AWS Load Balancer | 1 | 2 | 3 | 3 | **3** | Indisponibilité des services AWS exposés |
| A35 | HashiCorp Vault | 4 | 4 | 4 | 4 | **4** | Clés de toutes les ressources ; indisponibilité = applications à l'arrêt |
| A36 | Serveur de sauvegarde | 4 | 4 | 3 | 4 | **4** | Capacité de reprise ; D = 4 au moment d'une restauration |
| A37 | Microsoft 365 | 3 | 3 | 3 | 3 | **3** | Messagerie et documents ; mode dégradé possible |
| A38 | Discord | 2 | 2 | 1 | 2 | **2** | Provisoire — contenu échangé à confirmer |
| A39 | Postes fixes | 3 | 3 | 2 | 3 | **3** | Point d'entrée ; impact individuel sauf propagation |
| A40 | Portables nomades | 3 | 3 | 2 | 3 | **3** | Perte / vol ; réseaux non maîtrisés |
| A41 | Serveurs Linux (socle) | 3 | 3 | 3 | 3 | **3** | Criticité propre au socle ; les services critiques sont évalués séparément |
| A42 | Wazuh | 3 | 3 | 3 | 3 | **3** | Perte de la capacité de détection |
| A43 | OpenSearch / Dashboards | 3 | 3 | 3 | 3 | **3** | Preuves d'investigation ; données personnelles dans les logs |
| A44 | Prometheus / Loki / Grafana | 2 | 2 | 2 | 2 | **2** | Diagnostic dégradé |
| A45 | Suricata / Zeek | 2 | 2 | 2 | 2 | **2** | Perte de visibilité réseau |
| A46 | Greenbone / OpenVAS | 2 | 2 | 1 | 2 | **2** | Rapports à protéger ; indisponibilité supportable |
| A47 | Velociraptor | 3 | 4 | 2 | 3 | **4** | Capacité d'exécution sur tous les postes équipés |
| A48 | SonarQube / Trivy / MISP | 2 | 2 | 2 | 2 | **2** | Contrôles contournés, sans impact direct sur la production |

## 3. Observations

1. **Les outils d'administration sont aussi critiques que les données** : Ansible (A30), Velociraptor (A47), GitLab CI (A28), Vault (A35) n'hébergent pas de données métier mais donnent accès à tout. Leur criticité vient de leur **pouvoir**, pas de leur contenu.
2. **La criticité technique ≠ criticité métier** : Kubernetes est complexe, mais sa note 4 se justifie uniquement parce qu'il exécute les applications des activités A et B. Un cluster qui n'hébergerait que des outils internes serait noté plus bas.
3. **Notes provisoires** (dépendent d'informations manquantes) : A13, A19, A25, A32, A38 — à réévaluer dès que la DSI aura précisé les contenus et l'isolation.
