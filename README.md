# Network Security Audit Lab — Nmap & Wireshark

**Référentiel méthodologique d'audit réseau défensif : découverte d'hôtes, scan de ports, identification de services et analyse de trafic.**

Projet conçu comme support de démonstration pour un stage en **réseaux** ou en **cybersécurité**. Il documente une démarche reproductible, les limites des outils et la traçabilité des preuves — pas des conclusions de sécurité non vérifiées.

![Licence](https://img.shields.io/badge/licence-MIT-blue)
![Statut](https://img.shields.io/badge/preuves-à%20réaliser-orange)
![Outils](https://img.shields.io/badge/outils-Nmap%20%7C%20Wireshark-informational)
![Langue](https://img.shields.io/badge/documentation-français-lightgrey)

> **Statut de ce dépôt : modèle méthodologique.**
> Aucun scan n'a été exécuté dans le cadre de la publication de ces fichiers, aucun résultat réel n'est présenté et aucune vulnérabilité n'est affirmée. Les adresses `192.168.1.0/24` et `192.168.1.10` sont des **exemples de documentation** à remplacer par les adresses d'un laboratoire explicitement autorisé. Toute preuve manquante est marquée **« À réaliser »**.

---

## Table des matières

- [Objectifs](#objectifs)
- [Périmètre des sept tests](#périmètre-des-sept-tests)
- [Compétences mobilisées](#compétences-mobilisées)
- [Arborescence](#arborescence)
- [Démarrage rapide](#démarrage-rapide)
- [Conventions de statut des preuves](#conventions-de-statut-des-preuves)
- [Règles déontologiques](#règles-déontologiques)
- [Limites assumées](#limites-assumées)
- [Feuille de route](#feuille-de-route)
- [Auteur et licence](#auteur-et-licence)

---

## Objectifs

Ce projet poursuit quatre objectifs :

1. **Structurer une reconnaissance réseau autorisée** avec Nmap : découverte d'hôtes, scan de ports, détection de versions de services.
2. **Analyser un trafic capturé** avec Wireshark : ARP, DNS, TCP et ICMP, en reliant chaque trame observée à un protocole et à une hypothèse.
3. **Produire un livrable d'audit professionnel** : périmètre, méthodologie, observations horodatées, niveau de confiance, limites et recommandations proportionnées.
4. **Démontrer une posture défensive et éthique** : autorisation écrite, minimalisme des scans, protection des données capturées, refus de toute exploitation non demandée.

Le dépôt sert de **canevas**. Il ne remplace ni une autorisation, ni une campagne de tests réelle, ni une revue de configuration.

---

## Périmètre des sept tests

| # | Thème | Outil | Commande ou filtre | Fiche détaillée | Statut des preuves |
|---|-------|-------|--------------------|-----------------|--------------------|
| 1 | Découverte d'hôtes | Nmap | `nmap -sn 192.168.1.0/24` | [nmap/scan-methodology.md — Test 1](nmap/scan-methodology.md#test-1--découverte-dhôtes-avec-nmap--sn) | **À réaliser** |
| 2 | Scan de ports | Nmap | `nmap 192.168.1.10` | [nmap/scan-methodology.md — Test 2](nmap/scan-methodology.md#test-2--scan-de-ports-avec-nmap) | **À réaliser** |
| 3 | Détection de services | Nmap | `nmap -sV 192.168.1.10` | [nmap/scan-methodology.md — Test 3](nmap/scan-methodology.md#test-3--détection-de-services-avec-nmap--sv) | **À réaliser** |
| 4 | Analyse ARP | Wireshark | filtre d'affichage `arp` | [wireshark/traffic-analysis.md — Test 4](wireshark/traffic-analysis.md#test-4--analyse-arp-avec-le-filtre-arp) | **À réaliser** |
| 5 | Analyse DNS | Wireshark | filtre d'affichage `dns` | [wireshark/traffic-analysis.md — Test 5](wireshark/traffic-analysis.md#test-5--analyse-dns-avec-le-filtre-dns) | **À réaliser** |
| 6 | Analyse TCP | Wireshark | filtre d'affichage `tcp` | [wireshark/traffic-analysis.md — Test 6](wireshark/traffic-analysis.md#test-6--analyse-tcp-avec-le-filtre-tcp) | **À réaliser** |
| 7 | Analyse ICMP | Wireshark | filtre d'affichage `icmp` | [wireshark/traffic-analysis.md — Test 7](wireshark/traffic-analysis.md#test-7--analyse-icmp-avec-le-filtre-icmp) | **À réaliser** |

### Livrables associés

| Document | Rôle |
|----------|------|
| [nmap/README.md](nmap/README.md) | Fiches outils Nmap, options, états de ports, méthode de validation |
| [nmap/scan-methodology.md](nmap/scan-methodology.md) | Procédure détaillée des tests 1 à 3 |
| [wireshark/README.md](wireshark/README.md) | Prise en main de Wireshark, filtres, pièges d'interprétation |
| [wireshark/traffic-analysis.md](wireshark/traffic-analysis.md) | Procédure détaillée des tests 4 à 7 |
| [report/security-audit-report-template.md](report/security-audit-report-template.md) | Modèle de rapport d'audit avec registre d'observations |
| [evidence/README.md](evidence/README.md) | Convention de nommage, intégrité et checklist des preuves |
| [docs/interview-preparation.md](docs/interview-preparation.md) | Présentation orale de 60 à 90 s, 10 questions techniques, 5 questions difficiles |
| [LICENSE](LICENSE) | Licence MIT du contenu documentaire |

---

## Compétences mobilisées

- **Reconnaissance contrôlée** : choix des options Nmap selon la question posée, limitation de la charge, journalisation des commandes.
- **Lecture de protocoles** : handshake TCP, résolution ARP, requêtes DNS, écho ICMP, distinction entre absence de réponse et service filtré.
- **Rigueur analytique** : séparation explicite entre **fait observé**, **hypothèse** et **recommandation**.
- **Traçabilité** : empreintes SHA-256 des preuves, horodatage, version d'outil, auteur, périmètre.
- **Communication** : rapport lisible par un décideur, annexes techniques, synthèse des risques proportionnée.
- **Éthique opérationnelle** : autorisation préalable, moindre intrusion, arrêt immédiat hors périmètre.

> Un port ouvert n'est **pas** une vulnérabilité : c'est un service exposé, dont la criticité dépend de la version, de la configuration, du contexte et des contrôles compensatoires. Un scan limité ne démontre **jamais** que le réseau est sûr.

---

## Arborescence

```text
audit-securite-reseau/
├── README.md
├── LICENSE
├── nmap/
│   ├── README.md
│   └── scan-methodology.md
├── wireshark/
│   ├── README.md
│   └── traffic-analysis.md
├── report/
│   └── security-audit-report-template.md
├── evidence/
│   ├── README.md
│   ├── authorization/
│   │   └── .gitkeep
│   ├── nmap/
│   │   └── .gitkeep
│   ├── wireshark/
│   │   └── .gitkeep
│   └── screenshots/
│       └── .gitkeep
└── docs/
    └── interview-preparation.md
```

Les dossiers `evidence/*` ne contiennent aucune preuve tant que les tests n'ont pas été réalisés. Les fichiers `.gitkeep` préservent la structure sans simuler de preuve.

---

## Démarrage rapide

### 1. Cloner le dépôt

```bash
git clone https://github.com/ridwanissifou3-pixel/audit-securite-reseau.git
cd audit-securite-reseau
```

### 2. Lire le cadre d'autorisation

Consulter [evidence/README.md](evidence/README.md), puis remplir un formulaire d'autorisation avant toute manipulation. **Sans autorisation écrite, aucun test n'est lancé.**

### 3. Remplacer les adresses d'exemple

Les valeurs `192.168.1.0/24` et `192.168.1.10` sont des placeholders pédagogiques. Utiliser uniquement les adresses d'un laboratoire dont le propriétaire a donné un accord daté et signé.

```bash
# Exemple à adapter — ne pas exécuter tel quel sur un réseau tiers
LAB_NETWORK="<réseau_autorise_CIDR>"
LAB_HOST="<hôte_autorise>"
```

### 4. Vérifier la disponibilité des outils

```bash
nmap --version
wireshark --version   # ou : tshark --version
```

### 5. Exécuter les tests dans l'ordre

Suivre les procédures :

- [nmap/scan-methodology.md](nmap/scan-methodology.md) pour les tests 1 à 3 ;
- [wireshark/traffic-analysis.md](wireshark/traffic-analysis.md) pour les tests 4 à 7 ;
- consigner chaque preuve selon [evidence/README.md](evidence/README.md).

### 6. Renseigner le rapport

Dupliquer [report/security-audit-report-template.md](report/security-audit-report-template.md), remplacer chaque mention **« À réaliser »** par une observation réellement obtenue, ou conserver la mention si la preuve n'existe pas.

---

## Conventions de statut des preuves

| Marqueur | Signification | Usage |
|----------|---------------|-------|
| **Vérifié** | Résultat observé, daté, reproductible et accompagné de sa preuve | À n'utiliser qu'après exécution réelle |
| **À réaliser** | Test prévu mais non exécuté, ou preuve manquante | État par défaut de ce dépôt |
| **Exemple** | Illustration pédagogique, non issue d'une capture réelle | Ne jamais présenter comme un résultat |
| **Hypothèse** | Interprétation plausible restant à confirmer | Toujours accompagnée d'un test de confirmation |
| **Limite** | Ce que la méthode ne permet pas de conclure | À conserver dans le rapport |

Un tableau de résultats ne doit contenir **aucune ligne « Vérifié »** sans fichier de preuve associé dans `evidence/`.

---

## Règles déontologiques

1. **Autorisation écrite préalable** : périmètre, fenêtre horaire, contacts, procédure d'arrêt.
2. **Moindre intrusion** : préférer `-sn`, un scan ciblé et `-sV` léger ; éviter `-A`, `-O`, les scripts `--script` non validés et tout module intrusif.
3. **Pas d'exploitation** : aucune tentative d'authentification, d'élévation de privilèges, de modification ou d'exfiltration.
4. **Protection des données** : les captures peuvent contenir des identifiants, des requêtes DNS nominatives ou des données personnelles. Les anonymiser avant publication.
5. **Arrêt immédiat** si le trafic sort du périmètre, si un service critique réagit anormalement ou si le contact technique le demande.
6. **Séparation des rôles** : l'auteur du test ne valide pas seul la criticité des écarts.

---

## Limites assumées

- Le dépôt est un **modèle** : il ne contient ni résultat, ni capture, ni vulnérabilité vérifiée.
- Un scan Nmap ne remplace pas une revue de configuration, un test d'intrusion, une analyse de vulnérabilités authentifiée ou une revue de code.
- Une capture Wireshark ne voit que le trafic qui atteint l'interface de capture ; le trafic chiffré reste illisible sans clés.
- L'absence de réponse à un ping ne prouve pas l'absence d'hôte ; voir [la section sur les limites du ping](wireshark/traffic-analysis.md#limites-du-ping).
- Les versions de services et les empreintes TCP/IP sont indicatives et peuvent être faussées par les proxys, le NAT ou l'usurpation.

---

## Feuille de route

- [x] Structure documentaire et cadre méthodologique
- [x] Sept procédures de test rédigées
- [x] Modèle de rapport et guide d'entretien
- [ ] Formulaire d'autorisation signé — **À réaliser**
- [ ] Exécution des tests 1 à 7 sur un laboratoire autorisé — **À réaliser**
- [ ] Captures et sorties Nmap archivées avec empreintes SHA-256 — **À réaliser**
- [ ] Rapport d'audit complété avec observations réelles — **À réaliser**
- [ ] Relecture par un pair et anonymisation avant publication — **À réaliser**

---

## Auteur et licence

**Auteur :** **ISSIFOU Ridwan** — en formation Cisco / IBM, candidat à un stage en réseaux / cybersécurité.
**Contact :** +228 71 69 24 01

Le contenu de ce dépôt est distribué sous licence **MIT** — voir [LICENSE](LICENSE). Les marques Nmap et Wireshark appartiennent à leurs détenteurs respectifs ; ce projet n'est affilié ni à Nmap Software LLC, ni à la Wireshark Foundation.

> **Rappel final** : les tests décrits ne doivent être exécutés que sur des réseaux que vous possédez ou pour lesquels vous disposez d'une autorisation écrite explicite. En cas de doute, ne pas lancer le test.
