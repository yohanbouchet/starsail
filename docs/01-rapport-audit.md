# Rapport d'audit et diagnostic

Amélioration des pratiques DevOps — Projet StarMap

> Statut : à rédiger (structure du modèle fourni par StarSail).

## Synthèse

<!-- À rédiger en dernier : 4 constats et 4 recommandations triées par
priorité, une demi-page. -->

## 1. Introduction et contexte

### 1.1 Présentation de l'entreprise

StarSail Inc. est une entreprise technologique fondée il y a cinq ans,
qui compte une trentaine de collaborateurs. Spécialisée dans la
géolocalisation par traitement d'images astronomiques, elle vise à rendre
la navigation possible dans les zones dépourvues de signal satellite.

Elle est organisée en quatre pôles : Recherche et Développement (R&D),
Développement logiciel (Dev), Opérations et infrastructure (Ops),
Qualité et Amélioration continue (QA).

Son projet phare, StarMap, détermine la position d'un utilisateur à
partir d'une photographie du ciel nocturne. Il est en phase de
pré-production.

StarSail identifie elle-même plusieurs limites dans ses processus :
livraisons manuelles, tests partiels, communication perfectible entre
Dev et Ops, absence d'automatisation complète de la chaîne CI/CD. C'est
dans ce contexte que la Responsable Qualité et Amélioration Continue a
confié cet audit au Responsable DevOps, avec un objectif : améliorer la
qualité, la sécurité et la rapidité des déploiements.

### 1.2 Périmètre de l'audit

L'audit porte sur la chaîne de livraison de StarMap, depuis le poste du
développeur jusqu'à la mise en production : construction, tests,
contrôles de sécurité, déploiement et supervision.

**Périmètre technique**

| Élément | Description |
|---|---|
| Environnement de développement | Postes des développeurs, Docker Compose, PostgreSQL |
| Environnement de test / préproduction (staging) | Serveur Debian « iso-production » ; il est supposé héberger aussi l'environnement de démonstration utilisé avec le client, absent du schéma d'infrastructure |
| Environnement de production | Serveur Debian, conteneurs déployés par Ansible et des scripts Bash, supervision des journaux et des métriques HTTP |
| Outils transverses | Registre Docker interne, Trivy, Git |

**Périmètre organisationnel**

| Acteur | Rôle dans la chaîne |
|---|---|
| Équipe Dev (4 personnes) | Développement, tests, construction des images, rédaction de la procédure de déploiement |
| Équipe Ops (2 personnes) | Analyse de sécurité, tests, déploiement, supervision |
| Client | Priorisation, campagnes de test, choix de la date de mise en production |

**Hors périmètre** : la version mobile de StarMap, citée sans aucune
description ; le modèle d'intelligence artificielle et le pôle R&D ;
l'architecture de l'application (par exemple l'ajout d'un cache Redis) ;
les processus internes du client.

## 2. Méthodologie d'audit

### 2.1 Sources d'information

- Le document « Projet StarMap – Équipes et processus DevOps actuels » :
  présentation de l'entreprise et de l'infrastructure, questionnaires des
  équipes Dev et Ops, schéma d'infrastructure, frise du cycle DevOps et
  schéma des échanges entre le client, les Dev et les Ops (étapes E1 à
  E14).
- La demande de la Responsable Qualité et Amélioration Continue et les
  modèles de rapport fournis.
- Pour la veille : documentations officielles des outils, comparatifs,
  retours d'expérience publiés et dates de fin de support
  (endoflife.date).
- L'expérience de l'auditeur sur des projets antérieurs : analyse des
  images avec Trivy, contrôle des Dockerfile avec Hadolint, audit des
  dépendances, tests de bout en bout avec Cypress et tests de charge avec
  k6 ; puis chaîne CI/CD sous GitHub Actions avec SonarQube, Trivy,
  Gitleaks, Dependabot et mesure des indicateurs DORA.

### 2.2 Démarche d'analyse

L'audit a suivi cinq étapes :

1. Analyse des documents et définition du périmètre.
2. Cartographie du pipeline actuel (diagramme AS-IS).
3. Analyse SWOT, puis priorisation des constats selon leur impact sur la
   qualité, la sécurité et les délais.
4. Veille technologique ciblée sur les besoins identifiés.
5. Formulation de recommandations stratégiques.

Deux règles ont été appliquées tout au long de l'audit :

- **Règle de preuve** : chaque constat cite le document et la page dont
  il provient ; ce qui n'est pas documenté est présenté comme une
  hypothèse.
- **Traçabilité** : chaque point de friction reçoit un numéro (F1, F2…)
  repris dans l'analyse SWOT, les constats et les recommandations.

Les risques de sécurité sont analysés à l'aide de référentiels publics :
l'OWASP Top 10 (risques des applications web) et l'OWASP Top 10 des
risques CI/CD (risques des chaînes d'intégration et de déploiement).

Une IA (Claude, d'Anthropic) a été utilisée comme assistant pour relire
les documents, proposer des formulations et vérifier la cohérence.
L'analyse, les choix et le contenu final ont été décidés par l'auditeur.

## 3. État des lieux technique

### 3.1 Infrastructure et stack technologique

StarMap est hébergé sur deux serveurs Debian, l'un de test et
préproduction, l'autre de production, auxquels s'ajoutent les postes des
développeurs. Les images Docker sont stockées dans un registre interne
partagé et analysées avec Trivy avant chaque déploiement.

Le tableau suivant reprend les technologies citées par les équipes, leur
rôle dans la chaîne, leur niveau déclaré dans les questionnaires et l'état
de leur fin de support (source : endoflife.date, consulté le 17/11/2026).

| Technologie | Rôle dans la chaîne | Équipe | Niveau déclaré | Version documentée | Fin de support |
|---|---|---|---|---|---|
| PHP / Symfony | Langage et framework du backend | Dev | Très bon | « PHP 8 » ; Symfony non précisé | PHP 8.0 et 8.1 terminés ; 8.2 jusqu'au 31/12/2026 ; 8.3 à 8.5 maintenus |
| TypeScript / Stimulus | Interface web (front-end) | Dev | Bon | non précisée | — |
| Composer / NPM | Gestion des dépendances PHP et JavaScript | Dev | Bon | non précisée | Node.js 22, 24 et 26 maintenus |
| PHPUnit / Jest | Tests unitaires et d'intégration | Dev | Très bon / Moyen | non précisée | — |
| Docker Compose / Docker | Environnement de développement ; images et conteneurs en préproduction et en production | Dev / Ops | Moyen / Bon | non précisée | Docker Engine 29 maintenu ; 25.0 jusqu'au 04/12/2026 |
| PostgreSQL | Base de données (catalogue d'étoiles) | Dev et Ops | Moyen | non précisée | 14 terminé le 12/11/2026 ; 15 à 18 maintenus |
| Debian | Système des deux serveurs | Ops | Bon | « stable » | 13 (stable) maintenue ; 12 en support étendu jusqu'au 30/06/2028 |
| Apt | Gestion des paquets des serveurs Debian | Ops | Bon | — | — |
| Bash / Ansible | Configuration des serveurs et déploiement des conteneurs | Ops | Très bon / Bon | non précisée | — |
| TestInfra / BashUnit | Tests d'infrastructure | Ops | Bon | non précisée | — |
| Trivy | Analyse des failles (CVE) de l'image avant déploiement | Ops | non indiqué | non précisée | — |
| Git, VS Code | Gestion des versions du code ; éditeur | Dev | non indiqué | — | — |

### 3.2 Processus de développement et déploiement

```mermaid
---
config:
  flowchart:
    wrappingWidth: 200
    nodeSpacing: 30
    rankSpacing: 70
    subGraphTitleMargin:
      top: 8
      bottom: 14
---
flowchart TB
  subgraph CLIENT["Client"]
    direction LR
    C1["Priorise les évolutions"] ~~~ C2["Teste la démonstration avec les Dev"]
  end
  subgraph DEV["Équipe Dev"]
    direction LR
    D1["Développement et tests unitaires<br/><i>Docker Compose, PHPUnit, Jest</i>"]
    D2["Livraison de démonstration<br/><i>environnement de démonstration</i>"]
    D3["F1 · Construction de l'image sur le poste<br/><i>Docker</i>"]
    D4["F2 · Envoi au registre et procédure écrite<br/><i>registre interne</i>"]
    D1 --> D2 --> D3 --> D4
  end
  subgraph OPS["Équipe Ops"]
    direction LR
    O1["F4 · Analyse de l'image<br/><i>Trivy</i>"]
    O2["F5 · Préproduction et tests de la procédure<br/><i>Ansible, Bash</i>"]
    O3["F6 · Mise en production, accès coupé<br/><i>Ansible, Bash</i>"]
    O4["Supervision<br/><i>journaux, métriques HTTP</i>"]
    O1 --> O2 -->|date fixée avec le client| O3 --> O4
  end
  CLIENT <-->|"évolutions, campagne de test"| DEV
  DEV ==>|"F3 · e-mail : tag et procédure"| OPS
  OPS -.->|"F7 · e-mail : bugs, erreurs 5xx, CVE"| DEV

  classDef manuel fill:#ffe0b2,stroke:#e65100,color:#000
  classDef friction fill:#ffcdd2,stroke:#b71c1c,stroke-width:3px,color:#000
  classDef neutre fill:#eeeeee,stroke:#616161,color:#000
  class D1,D2,D4,O1,O2,O4 manuel
  class D3,O3 friction
  class C1,C2 neutre
  linkStyle 8,9 stroke:#b71c1c,stroke-width:3px
```

*Figure 1 – Pipeline actuel de StarMap (AS-IS).*

| Repère | Signification |
|---|---|
| Orange | Étape manuelle |
| Rouge (cadre épais) | Étape manuelle et point de friction majeur |
| Flèche rouge | Passage de relais par e-mail entre Dev et Ops |
| Gris | Intervention du client |
| F1 à F7 | Points de friction, repris dans les sections 4 à 6 |

Une évolution suit le chemin suivant. Le client priorise les évolutions
avec l'équipe Dev, qui développe et lance les tests unitaires sur ses
postes, puis livre une démonstration testée avec le client. L'image
Docker est ensuite construite sur le poste d'un développeur (F1), envoyée
au registre interne et accompagnée d'une procédure écrite (F2). Le tag de
l'image et la procédure sont transmis aux Ops par e-mail (F3). Les Ops
analysent l'image avec Trivy (F4), la déploient en préproduction et
rejouent les tests de la procédure (F5), puis fixent une date avec le
client et mettent en production en coupant l'accès (F6). Les anomalies
détectées par la supervision, les bugs et les alertes de sécurité
reviennent aux Dev par e-mail (F7).

### 3.3 Analyse des workflows Dev et Ops

**Passages de relais entre équipes**

| Échange | Sens | Canal | Source |
|---|---|---|---|
| Tag de l'image et procédure de déploiement | Dev → Ops | e-mail | p. 3 ; questionnaire Ops p. 7 |
| Bugs ou questions sur la procédure de déploiement | Ops → Dev | e-mail | questionnaires Dev p. 5 et Ops p. 7 |
| Erreurs HTTP 5xx constatées en production | Ops → Dev | e-mail | questionnaire Ops p. 7 |
| Alertes de sécurité (CVE dans l'image en production) | Ops → Dev | e-mail | p. 3 ; questionnaires Dev p. 5 et Ops p. 7 |
| Date de mise en production | Ops ↔ client | non précisé | questionnaire Ops p. 7 |
| Notification de mise en production | Ops → client et Dev | e-mail | questionnaire Ops p. 7 |

Les équipes Dev et Ops citent les trois mêmes types d'échanges : bugs liés
au déploiement, anomalies après la mise en production et alertes de
sécurité. Elles s'accordent donc sur les difficultés. Ces retours passent
par e-mail, « souvent avec retard » (p. 3).

**Tâches manuelles ou réalisées deux fois**

- Les tests sont menés deux fois à la main : d'abord par le client et
  les Dev sur l'environnement de démonstration, puis par les Ops en
  préproduction.
- La procédure de déploiement est rédigée par les Dev, puis rejouée pas
  à pas par les Ops.
- Le tag de l'image est recopié à la main dans un e-mail.

**Aspects humains**

- Le problème de nommage des images est signalé par les Ops dans une
  question réservée aux Dev, avec cette remarque : « Je ne sais pas si les
  développeurs en ont discuté » (questionnaire Ops, p. 6).
- L'équipe Ops compte deux personnes : le Lead Ops, spécialiste de Bash et
  d'Ansible, et un Senior Ops, responsable de la sécurité et de la
  supervision. Côté Dev, l'automatisation (Docker, NPM) est confiée au
  stagiaire (questionnaires Dev p. 4 et Ops p. 6).

## 4. Analyse SWOT

La matrice croise ce qui dépend de StarSail (forces et faiblesses) et ce
qui vient de son environnement (opportunités et menaces). Chaque point est
numéroté et renvoie, quand il y a lieu, aux points de friction F1 à F7 de
la figure 1. Les risques de sécurité figurent dans les faiblesses et les
menaces.

| | Utile | Nuisible |
|---|---|---|
| **Interne** | **Forces**<br>S1 Application conteneurisée, registre interne<br>S2 Préproduction identique à la production<br>S3 Analyse Trivy avant chaque déploiement<br>S4 Ansible et tests d'infrastructure<br>S5 Très bon niveau PHP/Symfony et PHPUnit<br>S6 Équipes coordonnées, démonstration au client, propositions d'outils<br>S7 Supervision de la production | **Faiblesses**<br>W1 Images construites sur les postes (F1)<br>W2 Nommage des images non respecté (F2)<br>W3 Relais et retours par e-mail, avec retard (F3, F7)<br>W4 Tests manuels, partiels et refaits deux fois (F5)<br>W5 Résultats de Trivy non suivis, CVE en production (F4)<br>W6 Mise en production manuelle avec coupure (F6)<br>W7 Aucune preuve de SAST, SCA, signature ni gestion des secrets<br>W8 Versions non inventoriées<br>W9 Savoirs portés par peu de personnes |
| **Externe** | **Opportunités**<br>O1 Socle existant (Docker, Ansible, registre)<br>O2 Outils CI/CD et de sécurité gratuits ou open source<br>O3 Pratiques DevSecOps et indicateurs DORA<br>O4 Notifications et tickets automatiques<br>O5 Équipes demandeuses d'automatisation | **Menaces**<br>T1 Faille exploitée en production<br>T2 Fin de support d'une brique non inventoriée<br>T3 Attaque par la chaîne d'approvisionnement<br>T4 Déploiement d'une mauvaise version<br>T5 Départ d'une personne clé<br>T6 Perte de réactivité face à la concurrence |

*Figure 2 – Matrice SWOT de la chaîne de livraison de StarMap.*

### 4.1 Forces

- **S1** : l'application est livrée en images Docker, stockées dans un
  registre interne partagé (p. 2).
- **S2** : les Ops testent chaque version sur un environnement
  « iso-production » avant la production (questionnaire Ops, p. 7).
- **S3** : chaque image est analysée avec Trivy avant d'être déployée
  (p. 3).
- **S4** : l'infrastructure est déjà décrite avec Ansible, et les Ops
  maîtrisent des outils de test d'infrastructure, TestInfra et BashUnit
  (questionnaire Ops, p. 6).
- **S5** : l'équipe Dev déclare un très bon niveau en PHP/Symfony et en
  PHPUnit (questionnaire Dev, p. 4).
- **S6** : les équipes travaillent de manière coordonnée (p. 3), le
  client voit une démonstration à mi-itération (questionnaire Dev, p. 5),
  et chaque équipe propose elle-même des outils (Cypress, Redis,
  Kubernetes).
- **S7** : la production est supervisée (journaux et métriques HTTP) et
  les erreurs 5xx sont détectées (schéma p. 8, questionnaire Ops p. 7).

### 4.2 Faiblesses

- **W1** : les images sont construites sur les postes des développeurs,
  sans chaîne d'intégration continue (p. 3) ; leur contenu dépend donc du
  poste qui les a produites.
- **W2** : le nommage (tag) des images « n'est pas toujours respecté »,
  ce qui provoque des erreurs de déploiement (questionnaire Ops, p. 6-7).
- **W3** : le travail et les retours (bugs, alertes de sécurité, erreurs)
  circulent par e-mail, « souvent avec retard » (p. 3) ; StarSail juge
  elle-même la communication Dev/Ops « perfectible » (p. 2).
- **W4** : les tests sont qualifiés de « partiels » (p. 2), lancés à la
  main et refaits deux fois (questionnaires Dev p. 5 et Ops p. 7) ; le
  niveau déclaré en Jest est « moyen » (p. 4).
- **W5** : les résultats de Trivy ne sont pas suivis dans un outil (p. 3),
  et des CVE sont signalées dans l'image en production (questionnaires Dev
  p. 5 et Ops p. 7).
- **W6** : la mise en production est manuelle et coupe l'accès à
  l'application pendant l'opération (questionnaire Ops, p. 7).
- **W7** : aucun élément fourni ne montre d'analyse du code (SAST), des
  dépendances Composer et NPM (SCA), de signature des images ni de
  gestion des secrets ; les développeurs envoient eux-mêmes leurs images
  dans le registre partagé (schéma p. 8).
- **W8** : les versions ne sont pas inventoriées (« PHP 8 », « Debian
  stable », PostgreSQL et Symfony sans numéro), ce qui empêche de vérifier
  leur fin de support (section 3.1).
- **W9** : l'équipe Ops compte deux personnes, et l'automatisation côté
  Dev est confiée au stagiaire (questionnaires Dev p. 4 et Ops p. 6).

### 4.3 Opportunités

- **O1** : Docker, Ansible et le registre existent déjà ; l'automatisation
  peut s'appuyer sur ce socle au lieu de le remplacer.
- **O2** : de nombreux outils de CI/CD, de test et de sécurité sont
  gratuits ou open source, adaptés à une petite équipe (comparés en
  annexe A).
- **O3** : les pratiques DevSecOps intègrent les contrôles de sécurité dès
  le début de la chaîne, et les indicateurs DORA permettent de mesurer
  les progrès de la livraison.
- **O4** : les alertes peuvent déclencher des notifications (téléphone,
  messagerie) et créer des tickets automatiquement, à la place des
  e-mails.
- **O5** : les équipes demandent déjà des outils d'automatisation : tests
  de bout en bout avec Cypress côté Dev, orchestration avec Kubernetes
  côté Ops (questionnaires Dev p. 5 et Ops p. 7).

### 4.4 Menaces

- **T1** : des CVE étant déjà présentes en production, une faille connue
  peut y être exploitée avant d'être corrigée.
- **T2** : une brique non inventoriée peut avoir atteint sa fin de
  support sans que l'équipe le sache : par exemple, PHP 8.2 n'est plus
  maintenu après le 31/12/2026, et PostgreSQL 14 ne l'est plus depuis le
  12/11/2026 (endoflife.date, consulté le 17/11/2026).
- **T3** : les outils de la chaîne peuvent eux-mêmes être compromis : en
  mars 2026, des versions piégées de Trivy ont volé les secrets de
  chaînes CI (CVE-2026-33634). Ce risque correspond à la catégorie A03 de
  l'OWASP Top 10:2025, « défaillances de la chaîne d'approvisionnement
  logicielle ».
- **T4** : un tag erroné peut conduire à déployer une mauvaise version en
  production.
- **T5** : le départ du stagiaire ou d'un des deux Ops ferait perdre une
  partie des savoirs nécessaires aux livraisons.
- **T6** : des livraisons lentes et des coupures de service réduisent la
  réactivité de StarSail face à ses concurrents.

## 5. Constats et besoins identifiés

### 5.1 Problématique 1

### 5.2 Problématique 2

### 5.3 Problématique 3

### 5.4 Problématique 4

### 5.5 Autres constats

## 6. Recommandations stratégiques

### 6.1 Axes d'amélioration prioritaires

### 6.2 Solutions proposées par axe

### 6.3 Justification des orientations

## Conclusion

## Annexes

### Annexe A – Tableau de veille technologique

### Annexe B – Glossaire

### Annexe C – Sources et références
