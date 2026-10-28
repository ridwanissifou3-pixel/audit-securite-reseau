# Nmap — Guide d'utilisation pour un audit réseau autorisé

**Documentation outil et grille de lecture des résultats.** Ce fichier ne contient aucun résultat de scan : les procédures concrètes des trois tests Nmap sont dans [scan-methodology.md](scan-methodology.md).

![Statut](https://img.shields.io/badge/résultats-aucun%20scan%20exécuté-orange)

---

## Sommaire

1. [À quoi sert Nmap](#1-à-quoi-sert-nmap)
2. [Prérequis et installation](#2-prérequis-et-installation)
3. [Anatomie d'une commande](#3-anatomie-dune-commande)
4. [Découverte d'hôtes](#4-découverte-dhôtes)
5. [Techniques de scan de ports](#5-techniques-de-scan-de-ports)
6. [Les états de ports](#6-les-états-de-ports)
7. [Détection de services](#7-détection-de-services)
8. [Options utiles et maîtrise de la charge](#8-options-utiles-et-maîtrise-de-la-charge)
9. [Formatage et conservation des preuves](#9-formatage-et-conservation-des-preuves)
10. [Lecture critique des résultats](#10-lecture-critique-des-résultats)
11. [Dépannage](#11-dépannage)
12. [Checklist avant et après un scan](#12-checklist-avant-et-après-un-scan)
13. [Ressources](#13-ressources)

---

## 1. À quoi sert Nmap

Nmap (*Network Mapper*) est un outil libre de cartographie réseau. Il permet notamment de :

- déterminer quels hôtes répondent sur un périmètre donné ;
- identifier les ports TCP et UDP en écoute ;
- deviner le service et sa version derrière un port ;
- compléter ces informations par des scripts NSE (*Nmap Scripting Engine*), dans un cadre strictement autorisé.

Dans ce projet, Nmap sert uniquement à **constater une exposition réseau** : présence d'un hôte, état d'un port, bannière de service. Il ne sert **pas** à exploiter, authentifier, modifier ou déstabiliser un système.

> **Nmap n'est pas un scanner de vulnérabilités.** Un port ouvert indique qu'un processus écoute. Il ne dit rien, à lui seul, de la présence d'une faille, de la sensibilité des données servies ou de la solidité de la configuration.

---

## 2. Prérequis et installation

### Prérequis techniques

| Élément | Exigence | Remarque |
|---------|----------|----------|
| Réseau | Laboratoire isolé ou périmètre autorisé | Jamais un réseau public ou tiers sans accord |
| Autorisation | Document signé, daté, avec périmètre et fenêtre | Voir [evidence/README.md](../evidence/README.md) |
| Droits | Compte local standard ou administrateur selon le type de scan | Certains scans exigent les privilèges racine |
| Connectivité | Route vers la cible, aucun filtrage sortant bloquant | Vérifier avec `ip route` / `traceroute` |
| Journal | Fichier de commandes et horodatage | À conserver avec les sorties |

### Installation

```bash
# Debian / Ubuntu
sudo apt update && sudo apt install nmap

# Fedora
sudo dnf install nmap

# macOS (Homebrew)
brew install nmap

# Windows : installeur officiel depuis nmap.org, puis vérifier dans PowerShell
nmap --version
```

Vérifier également la version du moteur de détection de services :

```bash
nmap --version
nmap --script-updatedb   # uniquement si les scripts NSE sont utilisés et autorisés
```

> La version de Nmap doit être consignée dans le rapport : les signatures de services évoluent d'une release à l'autre.

### Droits et types de scan

| Contexte | Scan TCP par défaut | Remarque |
|----------|---------------------|----------|
| Utilisateur avec privilèges (root / administrateur) | TCP SYN (`-sS`) | Semi-connexion, plus discret et plus rapide |
| Utilisateur sans privilèges | TCP connect (`-sT`) | Handshake complet via l'API système |
| Réseau local Ethernet | Découverte par ARP | Plus fiable que l'ICMP pour les hôtes du même segment |
| Réseau routé | ICMP, TCP et ARP impossible | Dépend du filtrage intermédiaire |

---

## 3. Anatomie d'une commande

```text
nmap -sn 192.168.1.0/24
│    │   └── cible : réseau ou hôte, ici un exemple à remplacer
│    └────── option : -sn = découverte d'hôtes sans scan de ports
└─────────── programme
```

```text
nmap -sV 192.168.1.10
│    │   └── cible : un hôte unique, exemple à remplacer
│    └────── option : -sV = détection de version des services
└─────────── programme
```

Une commande Nmap se lit toujours de gauche à droite : **programme**, **options**, **cible**. Les options peuvent être regroupées (`-sV -Pn` ne s'écrit pas `-sVPn` de façon fiable) ; préférer une option par bloc pour garder la commande lisible et auditable.

### Cibles acceptées

| Forme | Exemple | Usage |
|-------|---------|-------|
| Hôte unique | `192.168.1.10` | Test ciblé |
| Plage | `192.168.1.10-20` | Sous-ensemble autorisé |
| CIDR | `192.168.1.0/24` | Découverte d'un segment |
| Nom DNS | `serveur.lab.local` | Résolution préalable |
| Liste de cibles | `-iL cibles.txt` | Périmètre contractualisé |
| Exclusion | `--exclude 192.168.1.1` | Protéger une passerelle ou un équipement critique |

Toujours vérifier l'expansion réelle d'une plage avant exécution : `192.168.1.0/24` couvre 256 adresses, dont la plupart peuvent être hors périmètre.

---

## 4. Découverte d'hôtes

### Principe

La découverte d'hôtes (*host discovery*, anciennement « ping scan ») cherche à savoir **quelles adresses répondent**, sans scanner les ports. Elle repose sur plusieurs sondes complémentaires.

| Sonde | Protocole | Portée | Contournement |
|-------|-----------|--------|---------------|
| ARP request | ARP | Segment local uniquement | Difficile à filtrer localement |
| ICMP echo request | ICMP | Réseau routé | Souvent bloqué par les pare-feu |
| TCP SYN vers 443 | TCP | Réseau routé | Efficace si le port est ouvert ou le pare-feu permissif |
| TCP ACK vers 80 | TCP | Réseau routé | Peut traverser un pare-feu sans état |
| ICMP timestamp | ICMP | Réseau routé | Souvent bloqué |
| UDP vers port haut (option `-PU`) | UDP | Réseau routé | Réponses rares, utilité limitée |

### Option `-sn`

`-sn` (*no port scan*) désactive le scan de ports et ne conserve que la découverte d'hôtes.

```bash
nmap -sn 192.168.1.0/24
```

| Élément | Signification |
|---------|---------------|
| `-sn` | Découverte d'hôtes seule, sans scan de ports |
| `192.168.1.0/24` | Réseau cible — **exemple à remplacer** |

**Ce que `-sn` produit :** la liste des hôtes ayant répondu à au moins une sonde, avec éventuellement leur nom DNS et leur adresse MAC sur un réseau local.

**Ce que `-sn` ne produit pas :** la liste des ports ouverts, les services, les systèmes d'exploitation, ni la preuve qu'un hôte silencieux est absent.

> **Point clé :** un hôte qui ne répond pas à `-sn` peut être actif mais filtrer toutes les sondes. L'absence de réponse n'est pas une preuve d'absence.

### Options associées

| Option | Effet | Précautions |
|--------|-------|-------------|
| `-n` | Aucune résolution DNS | Réduit le bruit et les fuites de noms |
| `-R` | Résolution DNS systématique | Peut générer du trafic DNS et révéler des noms internes |
| `--dns-servers <IP>` | Serveur DNS dédié | À réserver au laboratoire |
| `-PE`, `-PP`, `-PM` | Choix explicite des sondes ICMP | Utile pour diagnostiquer un filtrage |
| `--disable-arp-ping` | Désactive l'ARP local | À utiliser seulement pour tester un comportement précis |
| `-Pn` | Aucune découverte : considérer la cible comme active | Ne pas confondre avec « la cible est vraiment active » |
| `-v` | Mode verbeux | À consigner dans les preuves |

### Limites de la découverte d'hôtes

- **Filtrage ICMP** : de nombreux systèmes ne répondent pas à l'echo request ; ils peuvent néanmoins être actifs.
- **Pare-feu sans état** : les réponses TCP ACK peuvent être bloquées, ce qui fausse la liste.
- **Segmentation** : un VLAN ou un routeur peuvent interdire les sondes entre segments.
- **ARP local** : fiable sur le même segment, inopérant au-delà d'un routeur.
- **NAT et proxys** : plusieurs hôtes peuvent partager une adresse publique ; un hôte peut apparaître sous une autre identité.
- **Équipements silencieux** : imprimantes, objets connectés ou systèmes durcis peuvent ne jamais répondre.
- **IPv6** : la découverte IPv4 ne dit rien de l'exposition IPv6, souvent oubliée.

---

## 5. Techniques de scan de ports

| Option | Nom | Fonctionnement | Privilèges | Remarque |
|--------|-----|----------------|------------|----------|
| `-sS` | TCP SYN | Envoie SYN, lit SYN/ACK ou RST, n'établit pas la connexion | Root | Rapide, semi-ouvert |
| `-sT` | TCP connect | Handshake complet via `connect()` | Utilisateur standard | Plus visible dans les journaux applicatifs |
| `-sU` | UDP | Envoie des datagrammes, interprète ICMP port unreachable | Root | Lent, résultats souvent `open\|filtered` |
| `-sA` | TCP ACK | Distingue filtré de non filtré | Root | Ne dit pas si un port est ouvert |
| `-sN`, `-sF`, `-sX` | NULL, FIN, Xmas | Contournement de certains filtres simples | Root | Non fiables sur les systèmes modernes |
| `-sI` | Idle scan | Utilise un hôte « zombie » | Root | Avancé, hors périmètre de ce lab |

### Scan par défaut

```bash
nmap 192.168.1.10
```

Sans option de scan, Nmap :

1. effectue une découverte d'hôtes ;
2. scanne les **1 000 ports TCP les plus courants** (ou les 1 000 ports UDP avec `-sU`) ;
3. utilise `-sS` si les privilèges le permettent, sinon `-sT` ;
4. affiche les ports ouverts, fermés ou filtrés selon les réponses.

Ce scan de référence est volontairement limité : il ne couvre ni les 65 535 ports, ni UDP, ni les services exotiques.

### Scan ciblé

```bash
# Ports explicitement autorisés
nmap -p 22,80,443 192.168.1.10

# Plage de ports
nmap -p 1-1024 192.168.1.10

# Tous les ports TCP (coûteux et plus visible)
nmap -p- 192.168.1.10

# UDP ciblé, avec explication des limites
nmap -sU -p 53,123 192.168.1.10
```

> `-p-` scanne les 65 535 ports TCP. Sur un réseau de production, cette commande peut générer un volume important de paquets, déclencher des alertes IDS et allonger considérablement la durée du test. Elle doit être explicitement prévue dans l'autorisation.

---

## 6. Les états de ports

Nmap distingue plusieurs états. Ils décrivent **la réponse observée**, pas la nature du service ni son niveau de sécurité.

| État | Réponse typique | Interprétation | Piège d'analyse |
|------|-----------------|----------------|-----------------|
| `open` | SYN/ACK (TCP), réponse applicative (UDP) | Un processus accepte des connexions | **Ne signifie pas vulnérable** |
| `closed` | RST (TCP), ICMP port unreachable (UDP) | Aucun processus n'écoute, hôte joignable | Un service peut être temporairement arrêté |
| `filtered` | Aucune réponse, ou ICMP administratively prohibited | Un équipement filtre le paquet | Peut aussi venir d'une perte réseau ou d'un délai trop court |
| `unfiltered` | Réponse ACK (scan `-sA`) | Le port est joignable, état ouvert/fermé inconnu | Ne pas confondre avec `open` |
| `open\|filtered` | Aucune réponse distinctive | Nmap ne peut pas trancher | Fréquent en UDP et sur les scans NULL/FIN/Xmas |
| `closed\|filtered` | Réponse ambiguë | État indéterminé | Rare, lié à certains scans idle |

### Pourquoi `filtered` n'est pas `closed`

- `closed` suppose que l'hôte a **répondu** par un RST ou un ICMP unreachable : le paquet est arrivé, la pile a répondu.
- `filtered` signifie que Nmap **n'a pas reçu la réponse attendue** : le paquet a pu être bloqué par un pare-feu, ignoré par la pile, perdu, ou arriver après le délai de retransmission.
- Un port `filtered` peut donc être ouvert derrière un filtrage qui n'autorise pas la source du test.

### Pourquoi `open` n'est pas une vulnérabilité

Un port ouvert est une **surface d'exposition**. La criticité dépend de nombreux facteurs :

| Facteur | Question à poser |
|---------|------------------|
| Service | Quel logiciel écoute réellement ? |
| Version | Est-elle maintenue et patchée ? |
| Authentification | Le service exige-t-il des identifiants robustes ? |
| Chiffrement | Le transport est-il chiffré et correctement configuré ? |
| Exposition | Le port est-il accessible depuis Internet ou seulement depuis un VLAN d'administration ? |
| Données | Quelles données le service traite-t-il ? |
| Contrôles | Un pare-feu, un WAF, une authentification réseau ou une segmentation réduisent-ils le risque ? |

> **Formulation correcte :** « Le port 443/TCP est ouvert et expose un service dont la bannière suggère [service] version [x]. Ce constat justifie une vérification de la politique de patch et de la configuration TLS. »
> **Formulation incorrecte :** « Le port 443 est ouvert, le serveur est vulnérable. »

### Handshake TCP et lecture des états

```text
Client                                   Serveur
  │──── SYN (seq=x) ───────────────────────▶│   port ouvert
  │◀─── SYN/ACK (seq=y, ack=x+1) ───────────│
  │──── ACK (ack=y+1) ─────────────────────▶│   connexion établie

Client                                   Serveur
  │──── SYN ──────────────────────────────▶│   port fermé
  │◀─── RST/ACK ───────────────────────────│

Client                                   Serveur / pare-feu
  │──── SYN ──────────────────────────────▶│   port filtré
  │         (aucune réponse)               │   ou ICMP admin. prohibited
```

| Observation | État Nmap | Ce que l'on peut affirmer |
|-------------|-----------|---------------------------|
| SYN/ACK reçu, puis RST envoyé par Nmap | `open` | Une pile TCP accepte les connexions sur ce port |
| RST reçu | `closed` | L'hôte est joignable, rien n'écoute à cet instant |
| Aucune réponse après retransmissions | `filtered` | Le résultat est indéterminé : filtrage, perte ou lenteur |
| ICMP type 3 code 13 | `filtered` | Un équipement intermédiaire bloque administrativement |
| SYN/ACK sur un scan `-sA` | Non applicable | Le scan ACK ne détermine pas l'ouverture |

Le handshake complet (`-sT`) établit une connexion : le service peut journaliser la connexion, voire déclencher une alerte. Le scan SYN (`-sS`) ne termine pas le handshake, mais reste détectable par un IDS réseau.

---

## 7. Détection de services

### Option `-sV`

```bash
nmap -sV 192.168.1.10
```

`-sV` active la détection de version : Nmap envoie des sondes applicatives, lit les bannières et compare les réponses à sa base de signatures.

| Élément | Signification |
|---------|---------------|
| `-sV` | *Service/version detection* |
| `192.168.1.10` | Hôte cible — **exemple à remplacer** |

**Résultat attendu :** pour chaque port ouvert, un nom de service, éventuellement un produit, une version, un système d'exploitation et un indicateur de confiance (`Service Info`).

### Options de réglage

| Option | Effet | Quand l'utiliser |
|--------|-------|------------------|
| `--version-intensity 0-9` | Intensité des sondes (défaut : 7) | Baisser pour limiter la charge |
| `--version-light` | Équivalent d'intensité 2 | Laboratoire sensible |
| `--version-all` | Essaie toutes les sondes (intensité 9) | Analyse approfondie, plus intrusive |
| `--version-trace` | Trace détaillée des sondes | Débogage uniquement |
| `-sV --open` | N'affiche que les ports ouverts | Rapport synthétique |

### Ce que `-sV` peut et ne peut pas dire

| Peut | Ne peut pas |
|------|-------------|
| Identifier un serveur web, SSH, DNS ou SMTP | Prouver que le service est à jour |
| Proposer un produit et une version | Voir un correctif rétroporté par le distributeur |
| Révéler une bannière personnalisée | Garantir que la bannière est véridique |
| Distinguer plusieurs services sur un même port | Détecter une vulnérabilité applicative |
| Fournir un indice de confiance | Remplacer une revue de configuration |

**Faux positifs classiques :**

- bannière falsifiée ou générique (`Server`, `nginx` sans version) ;
- reverse proxy qui masque le serveur réel ;
- port non standard servant un autre protocole ;
- service encapsulé dans TLS, illisible sans négociation adaptée ;
- version distribuée par un paquet patché sans changement de numéro.

**Faux négatifs classiques :**

- service qui ne répond qu'à un client authentifié ;
- filtrage qui laisse passer le SYN mais bloque les sondes applicatives ;
- délai de réponse supérieur au timeout ;
- service UDP sans réponse distinctive.

> Toute identification de version doit être confrontée, lorsque c'est possible et autorisé, à une source fiable : inventaire, bannière applicative, revue de paquets ou entretien avec l'administrateur. Sans confirmation, la mention reste **« hypothèse »**.

---

## 8. Options utiles et maîtrise de la charge

### Vitesse et fiabilité

| Option | Effet | Risque |
|--------|-------|--------|
| `-T0` à `-T1` | Très lent, furtif | Durée excessive, peu utile en lab |
| `-T2` | Lent | Réseaux sensibles |
| `-T3` | Normal (défaut) | Bon compromis |
| `-T4` | Rapide | Recommandé sur un réseau local sain |
| `-T5` | Agressif | Perte de paquets, faux `filtered`, à éviter |
| `--max-retries <n>` | Limite les retransmissions | Peut augmenter les faux `filtered` |
| `--host-timeout <durée>` | Borne le temps par hôte | Un timeout n'est pas une preuve d'absence |
| `--min-rate` / `--max-rate` | Cadence de paquets | À fixer prudemment, jamais sans accord |
| `--scan-delay <durée>` | Espacement des sondes | Réduit la charge et la détection |

### Résolution et affichage

| Option | Effet |
|--------|-------|
| `-n` | Pas de résolution DNS |
| `-v`, `-vv` | Verbeux, davantage de contexte |
| `--reason` | Indique la raison de chaque état de port |
| `--open` | N'affiche que les ports ouverts |
| `--packet-trace` | Trace des paquets (volume important) |
| `--stats-every 30s` | Progression périodique |

### Scripts NSE

Les scripts NSE peuvent aller de la simple collecte de bannière à des tests intrusifs. Dans ce projet :

- **aucun script NSE n'est utilisé par défaut** ;
- un script ne peut être ajouté que s'il figure dans l'autorisation écrite ;
- les catégories `intrusive`, `brute`, `exploit` et `dos` sont **exclues** ;
- toute sortie de script doit être relue et confrontée à une source indépendante.

```bash
# Exemple de script non intrusif, uniquement si autorisé et pertinent
nmap -sV --script banner 192.168.1.10
```

> Ne jamais interpréter un script NSE comme un audit de vulnérabilité complet. Un script reproduit un test unitaire, avec ses faux positifs.

---

## 9. Formatage et conservation des preuves

### Formats de sortie

| Option | Format | Usage |
|--------|--------|-------|
| `-oN <fichier>` | Texte normal | Lecture humaine |
| `-oX <fichier>` | XML | Traitement automatisé, import |
| `-oG <fichier>` | Grepable | Extraction rapide (format historique) |
| `-oA <préfixe>` | Les trois formats | **Recommandé** pour l'archivage |

```bash
# Exemple de journalisation complète
nmap -sV --reason -oA evidence/nmap/test3_services 192.168.1.10
```

Cette commande crée `test3_services.nmap`, `test3_services.xml` et `test3_services.gnmap`. Le préfixe doit suivre la convention d'[evidence/README.md](../evidence/README.md).

### Métadonnées à consigner

| Métadonnée | Exemple de valeur | Pourquoi |
|------------|-------------------|----------|
| Date et heure | `2026-10-28T14:05:00Z` | Situer l'observation |
| Opérateur | Initiales ou identifiant | Responsabilité |
| Version Nmap | `nmap --version` | Reproductibilité |
| Commande exacte | `nmap -sV 192.168.1.10` | Rejouer le test |
| Périmètre autorisé | Référence du document | Prouver le cadre |
| Source | Adresse IP de la machine de test | Interpréter le filtrage |
| Sortie brute | Fichier `.nmap`, `.xml` | Preuve |
| Empreinte SHA-256 | `sha256sum fichier` | Intégrité |
| Observations | Notes horodatées | Contexte |

### Empreinte d'intégrité

```bash
sha256sum evidence/nmap/test3_services.nmap
```

Consigner l'empreinte dans le registre des preuves. Une capture ou une sortie non hashée peut être contestée.

---

## 10. Lecture critique des résultats

### Grille d'analyse

Pour chaque ligne de résultat, répondre dans cet ordre :

1. **Quel fait a été observé ?** (port, état, réponse, horodatage)
2. **Quelle est la qualité de la preuve ?** (fichier brut, reproductibilité, confiance Nmap)
3. **Quelle hypothèse en découle ?** (service exposé, filtrage, hôte actif)
4. **Quel test confirmerait ou infirmerait l'hypothèse ?**
5. **Quelle recommandation est proportionnée ?**

### Erreurs d'interprétation fréquentes

| Erreur | Correction |
|--------|------------|
| « Port ouvert = vulnérabilité » | Un port ouvert est une exposition ; la vulnérabilité se confirme par analyse |
| « Aucun port ouvert = système sûr » | Le scan est partiel (ports, protocoles, filtrage, source) |
| « `filtered` = fermé » | `filtered` est indéterminé |
| « `-sn` ne trouve rien = réseau vide » | Les hôtes peuvent être silencieux ou filtrés |
| « Version détectée = version installée » | Bannière falsifiable, correctifs rétroportés, proxy possible |
| « Scan depuis un seul point = exposition réelle » | La position réseau change le résultat ; un scan externe et un scan interne ne mesurent pas la même chose |
| « `-T5` plus rapide = plus fiable » | L'agressivité provoque des pertes et de faux `filtered` |

### Ce qu'un scan Nmap ne mesure pas

- la sécurité applicative (injections, contrôle d'accès, logique métier) ;
- les vulnérabilités authentifiées et les mauvaises configurations internes ;
- la sécurité des flux chiffrés ;
- les risques humains, organisationnels et fournisseurs ;
- l'exposition IPv6 si seul IPv4 est scanné ;
- les services non détectés faute de réponse ou de signature.

---

## 11. Dépannage

| Symptôme | Cause probable | Action |
|----------|----------------|--------|
| `nmap: command not found` | Outil absent ou hors `PATH` | Installer, vérifier `which nmap` |
| `Host seems down` | ICMP et TCP filtrés | Vérifier l'autorisation, puis tester `-Pn` de façon encadrée ; ne pas conclure trop vite |
| `QUITTING!` après une demande de privilèges | Scan privilégié sans droits | Relancer avec `sudo` ou choisir `-sT` |
| Tous les ports `filtered` | Pare-feu, mauvaise route, source non autorisée | Vérifier la route, la source, les journaux du pare-feu |
| Résultat incohérent entre deux scans | Équilibrage, filtrage dynamique, timeouts | Répéter, comparer les horodatages, baisser `-T` |
| Scan très long | `-p-`, `-sU`, `-T0`, réseau lent | Réduire le périmètre, borner avec `--host-timeout` |
| Version absente | Service silencieux ou chiffré | Ne pas inventer ; marquer « non déterminée » |

---

## 12. Checklist avant et après un scan

### Avant

- [ ] Autorisation écrite disponible et périmètre vérifié
- [ ] Adresses cibles résolues et confirmées avec le contact technique
- [ ] Fenêtre horaire respectée
- [ ] Source de test autorisée et identifiée
- [ ] Version de Nmap notée
- [ ] Commande relue : pas de `-p-`, `-A`, `--script` non validé
- [ ] Procédure d'arrêt connue
- [ ] Dossier de preuves préparé

### Pendant

- [ ] Sortie redirigée vers un fichier (`-oA`)
- [ ] Heure de début consignée
- [ ] Surveillance de la charge réseau et des alertes
- [ ] Arrêt immédiat si anomalie ou dépassement de périmètre

### Après

- [ ] Heure de fin consignée
- [ ] Sorties brutes conservées sans modification
- [ ] Empreintes SHA-256 calculées
- [ ] Résultats distingués : fait / hypothèse / recommandation
- [ ] Ports non déterminés explicitement marqués
- [ ] Rapport mis à jour, mentions « À réaliser » conservées si preuve absente
- [ ] Données sensibles anonymisées avant tout partage

---

## 13. Ressources

- Documentation officielle Nmap : <https://nmap.org/book/man.html>
- Référence des options : <https://nmap.org/book/man-briefoptions.html>
- Guide des états de ports : <https://nmap.org/book/man-port-scanning-basics.html>
- Base de signatures de services : <https://nmap.org/book/man-version-detection.html>
- Cadre légal français : articles 323-1 à 323-8 du Code pénal (accès et maintien frauduleux dans un système de traitement automatisé de données).

> **Rappel** : la documentation officielle reste la référence. Ce guide est un support pédagogique, pas une garantie de résultat.
