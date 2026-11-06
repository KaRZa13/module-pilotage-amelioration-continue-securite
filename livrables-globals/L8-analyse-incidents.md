# Livrable 8 — Analyse des incidents

> Partie 13 de l'énoncé. Aucune règle de détection n'est construite ici : il s'agit de **qualifier** les situations.

---

## 1. User Story US-SEC-001 — vérification des critères d'acceptation

> *En tant que responsable sécurité de NOVA SERVICES, je veux disposer d'une vision structurée des actifs critiques, de leurs dépendances et de leur exposition, afin de déterminer les priorités de sécurité et les besoins de surveillance du système d'information.*

| Critère d'acceptation | Statut | Où le trouver |
|---|---|---|
| Activités métier principales identifiées | ✅ | [L1](L1-analyse-activites-metier.md) |
| Données importantes et sensibles recensées | ✅ (12 catégories) | [L3](L3-inventaire-donnees.md) |
| Actifs métier et techniques distingués | ✅ (A01-A04 métier, A05-A48 techniques) | [L2](L2-registre-actifs.md) |
| Propriétaires identifiés ou signalés comme inconnus | ✅ (tous « à désigner », rôle pressenti indiqué) | [L2](L2-registre-actifs.md) |
| Criticité justifiée | ✅ | [L4](L4-evaluation-criticite.md) |
| Expositions et dépendances documentées | ✅ | [L6](L6-exposition-dependances.md) |
| Cartographie du SI produite | ✅ | [L5](L5-cartographie-SI.md) |
| Premiers besoins de sécurité et de surveillance exprimés | ✅ | [L7](L7-besoins-et-matrice-pilotage.md) |
| Décisions reliées aux enjeux métier | ✅ | [L9](L9-synthese-decisionnelle.md) |
| Hypothèses et informations manquantes signalées | ✅ (marqueurs [H] / [V]) | [00-README](00-README.md) + chaque livrable |

---

## 2. INC-001 — Connexion inhabituelle à un compte administrateur

**Faits communiqués** : compte `admin-jdupont` · service : administration du SI · connexion via VPN · source Internet · actif administré : serveur de base de données · heure : 02 h 37.

> **Point de méthode** : une connexion inhabituelle est un **signal à examiner**, pas une preuve de compromission.

### Quels actifs et services sont concernés ?

| Actif | Rôle dans l'événement |
|---|---|
| Compte `admin-jdupont` (identité à privilèges, D04) | Identité utilisée — référentiel à préciser : Keycloak ? compte VPN local ? compte système / PostgreSQL ? [V] |
| VPN collaborateurs (A06) — ou fournisseurs (A07) ? [V] | Point d'entrée utilisé |
| Pare-feu OPNsense (A05) | Point de passage de la connexion |
| Serveur de base de données : serveur Linux (A41) + PostgreSQL (A18) — ou RDS (A19) ? [V] | Cible de l'administration |
| Données clients et de production (D01, D02) | Données potentiellement atteintes |
| Activités A (Gestion des clients), B (Production), D (Administration du SI) | Activités métier exposées |

### Pourquoi la base de données peut-elle être critique ?

- Elle concentre les **données clients** (personnelles, contrats, facturation) et les **données de production** de plusieurs applications (criticité 4, cf. [L4](L4-evaluation-criticite.md)).
- Un compte administrateur peut **lire tout**, **modifier tout** (falsification de factures, de RIB) et **supprimer tout**, sans passer par les contrôles applicatifs.
- Plusieurs applications en dépendent : une indisponibilité arrête les activités A et B.
- Elle peut héberger la base de Keycloak [V] : l'impact s'étendrait alors aux identités.

### Quels risques peuvent être associés à cette connexion ?

| Hypothèse | Risque |
|---|---|
| Identifiants (et éventuellement MFA) volés par hameçonnage ou réutilisation de mot de passe | Accès illégitime à privilèges : exfiltration des données clients, falsification, sabotage |
| Poste de l'administrateur compromis (session VPN détournée) | Idem, plus persistance sur le poste |
| Rançongiciel en phase de préparation (heure nocturne typique) | Suppression des sauvegardes puis chiffrement de la base |
| **Activité légitime** : intervention planifiée, astreinte, administrateur en déplacement à l'étranger, nouveau fournisseur d'accès | Aucun — mais révèle peut-être un défaut de traçabilité des interventions |
| Compte partagé ou utilisé par un prestataire | Défaut de gouvernance des comptes (traçabilité impossible) |

### Quelles informations supplémentaires permettraient de qualifier l'événement ?

1. **Intervention prévue ?** Ticket de changement, astreinte, maintenance planifiée à 02 h 37.
2. **Confirmation hors bande** avec le titulaire du compte (appel téléphonique, pas par courriel).
3. **Origine de l'IP** : géolocalisation, fournisseur, réputation (MISP), déjà utilisée par ce compte ?
4. **Authentification** : MFA utilisé ? Échecs préalables ? Plusieurs sessions simultanées depuis des IP différentes ?
5. **Historique du compte** : horaires habituels, IP habituelles, machines habituellement administrées.
6. **Actions réalisées pendant la session** : commandes, `sudo`, requêtes SQL, volume de données lues, modifications de droits, accès au serveur de sauvegarde.
7. **Durée de la session** et éventuelles connexions vers d'autres serveurs (mouvement latéral).
8. **Signaux corrélés** : alertes Suricata / Zeek, nouvelle clé SSH, création de compte, changements dans Keycloak.

### Quels journaux ou sources de données pourraient être utiles ?

| Source | Information recherchée | Collectée aujourd'hui ? |
|---|---|---|
| Journaux VPN (WireGuard / OpenVPN) | Compte, IP publique, heure, durée, IP attribuée | [V] |
| OPNsense | Flux depuis l'IP VPN attribuée vers le serveur | Oui (vers Wazuh) [C] |
| Keycloak | Authentification du compte, MFA, échecs préalables | Oui [C] |
| Serveur Linux : `auth.log`, SSH, `sudo` | Connexion SSH, élévation de privilèges, commandes | Oui (agent Wazuh) [C] |
| Journaux PostgreSQL | Connexions, requêtes, modifications de rôles, volumes | **Non décrit** [V] |
| Serveur de sauvegarde | Accès ou suppressions de sauvegardes | **Non décrit** [V] |
| Suricata / Zeek | Volumes sortants, connexions vers des IP suspectes | Oui [C] |
| Velociraptor | Investigation du poste de l'administrateur / du serveur si doute | Outil disponible [C] |
| Outil de gestion des changements / tickets | Intervention prévue ? | [V] |

### Qui devrait être informé selon la procédure interne ?

**La procédure interne de gestion des incidents n'est pas fournie [V]**. En l'absence de procédure, les interlocuteurs logiques sont :
- le **responsable sécurité / RSSI** (qualification et décision) ;
- le **responsable informatique / équipe infrastructure** (propriétaire technique du serveur et du VPN) ;
- le **titulaire du compte** `admin-jdupont` (confirmation hors bande) et son responsable hiérarchique ;
- le **propriétaire métier des données** (direction commerciale / opérations) si des données ont été consultées ou modifiées ;
- le **DPO** si une violation de données personnelles est suspectée (délai de 72 h pour la notification à la CNIL si la violation est avérée) ;
- la **direction** en cas d'incident confirmé à fort impact.

> Action de pilotage : formaliser cette procédure (mesure ANSSI « procédure de gestion des incidents »), voir [10-referentiel-ANSSI.md](10-referentiel-ANSSI.md).

### Quelles informations manquent pour conclure à un incident de sécurité ?

- La confirmation ou l'infirmation par le titulaire du compte et l'existence d'une intervention planifiée.
- La preuve d'une **action illégitime** (accès à des données hors du périmètre de la mission, modifications, exfiltration) — aujourd'hui difficile faute de journaux PostgreSQL.
- La comparaison avec le **comportement habituel** du compte (aucune référence établie).
- La méthode d'authentification réellement utilisée (MFA ou non).

**Conclusion provisoire** : à traiter comme un **événement de sécurité à qualifier en priorité élevée** (compte à privilèges + actif de criticité 4 + horaire atypique + source Internet), sans le déclarer incident tant que les vérifications 1 à 6 n'ont pas été réalisées. Si le titulaire ne confirme pas la connexion, des mesures conservatoires s'imposent (désactivation du compte, coupure de la session, préservation des journaux).

---

## 3. INC-002 — Vulnérabilité critique sur une application métier

**Faits** : le scanner signale une vulnérabilité critique sur un serveur hébergeant une application métier. Le responsable métier : *« Nous ne pouvons pas arrêter l'application aujourd'hui. Elle est utilisée par tous les commerciaux. »*

### La criticité technique de la vulnérabilité suffit-elle pour décider de la priorité ?

**Non.** Le score (CVSS) mesure la gravité **intrinsèque** de la faille, indépendamment du contexte. La priorité dépend aussi :
- de la **criticité métier** de l'actif ;
- de son **exposition** réelle (Internet, interne, conditions d'exploitation) ;
- de l'**exploitabilité** (code d'exploitation public, exploitation active constatée) ;
- des **mesures compensatoires** déjà en place ;
- des **conséquences** d'une exploitation (données atteintes, rebonds possibles).

Il faut aussi vérifier qu'il ne s'agit pas d'un **faux positif** (version détectée par bannière, correctif rétroporté par la distribution Debian / Ubuntu).

### Quelle est la criticité métier de l'application ?

**Élevée (4)** : application utilisée par **tous les commerciaux** → support direct de l'activité A — Gestion des clients (données clients, contrats, facturation). Les deux dimensions s'opposent : elle est **trop critique pour être arrêtée**, mais aussi **trop critique pour rester vulnérable**. C'est précisément pourquoi la décision doit être instruite et non subie.

### Le serveur ou le service est-il accessible depuis Internet ou par d'autres chemins ?

À confirmer [V]. Questions à poser :
- L'application est-elle publiée via l'Ingress ou le Load Balancer (exposition directe) ou seulement accessible en interne / via VPN ?
- Le **composant vulnérable** est-il celui qui est exposé (ex. le serveur Web) ou un service secondaire non joignable (ex. un port d'administration filtré) ?
- Quels autres chemins : VPN collaborateurs (commerciaux nomades), VPN fournisseurs, autres applications du même réseau ?

Si l'application est exposée sur Internet (cas de l'événement 3), la priorité devient très élevée.

### Existe-t-il un correctif, un contournement ou une mitigation ?

À vérifier auprès de l'éditeur / des sources de sécurité. Options à étudier **sans arrêt prolongé** :
- **Correctif** appliqué dans une fenêtre courte et planifiée (soir, week-end), après test en préproduction ; si l'application est sur Kubernetes, déploiement progressif (*rolling update*) **sans interruption** du service.
- **Mitigations temporaires** : restreindre l'accès (filtrage OPNsense / Security Group aux seules IP ou au VPN des commerciaux), règle de filtrage applicatif (WAF / reverse proxy) bloquant le motif d'exploitation, désactivation de la fonctionnalité vulnérable, durcissement de la configuration.
- **Surveillance renforcée** : signatures Suricata dédiées, examen des journaux d'accès à la recherche de tentatives d'exploitation.

### Quelles dépendances et contraintes d'exploitation faut-il connaître ?

- Hébergement : serveur dédié, Kubernetes interne ou EKS ? Redondance possible (plusieurs instances) ?
- Dépendances : PostgreSQL (A18), Keycloak (A14), API (A17), Kafka (A20).
- Existence d'un environnement de **préproduction** pour tester le correctif.
- Fenêtres de faible usage (horaires des commerciaux, périodes de clôture commerciale).
- Contraintes contractuelles (engagements de disponibilité, éditeur à solliciter).
- Propriétaire métier habilité à **accepter formellement un risque résiduel**.

### Quel risque résiduel subsisterait si la correction était différée ?

- Exploitation de la faille → **exécution de code ou accès aux données clients** → fuite (impact RGPD, perte de confiance), falsification, voire rebond vers PostgreSQL, le cluster ou le réseau interne.
- Le risque croît avec le temps (publication d'un code d'exploitation, scans automatisés sur Internet).
- Si des mitigations sont en place, le risque résiduel est réduit mais pas nul : il doit être **documenté, accepté par le propriétaire métier, borné dans le temps** (date limite de correction) et **suivi**.

### Quelles informations devront être collectées dans le TP2 pour prioriser ce risque ?

| Information | Source |
|---|---|
| Identifiant de la vulnérabilité (CVE), score CVSS, vecteur d'attaque | Rapport Greenbone |
| Exploitabilité : code public, exploitation active connue (catalogues de vulnérabilités exploitées), probabilité d'exploitation | Veille, MISP |
| Exposition réelle de l'application et du composant | Cartographie (L5), règles OPNsense / Ingress / Security Groups |
| Criticité métier et données traitées | Registre (L2), inventaire des données (L3) |
| Disponibilité d'un correctif, d'une mitigation, d'un environnement de test | Éditeur, équipe applicative |
| Tentatives d'exploitation déjà observées | Journaux d'accès (Loki / OpenSearch), Suricata |
| Délai acceptable de correction et décideur de l'acceptation du risque | Propriétaire métier, RSSI |

**Recommandation provisoire** : ne pas arrêter l'application aujourd'hui, mais **appliquer dès maintenant des mitigations** (restriction d'accès, filtrage, surveillance renforcée), **planifier le correctif** dans la fenêtre la plus proche avec un test préalable, et **formaliser l'acceptation temporaire du risque** par le responsable métier avec une échéance.
