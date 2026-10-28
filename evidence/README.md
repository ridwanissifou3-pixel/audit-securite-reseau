# Preuves d'audit — Convention et registre

**Toute affirmation du rapport doit pouvoir être reliée à une preuve conservée, horodatée et intacte.**

Ce dossier est volontairement vide de résultats : il ne contient que ce README et des fichiers `.gitkeep`. Aucune sortie Nmap, aucune capture Wireshark et aucune capture d'écran n'a été produite dans le cadre de la publication de ce dépôt. Les éléments manquants sont marqués **« À réaliser »**.

![Statut](https://img.shields.io/badge/preuves-aucune%20preuve%20réelle-orange)

---

## Sommaire

1. [Principes](#1-principes)
2. [Arborescence](#2-arborescence)
3. [Autorisation préalable](#3-autorisation-préalable)
4. [Convention de nommage](#4-convention-de-nommage)
5. [Preuves attendues par test](#5-preuves-attendues-par-test)
6. [Capturer une sortie Nmap](#6-capturer-une-sortie-nmap)
7. [Conserver une capture Wireshark](#7-conserver-une-capture-wireshark)
8. [Captures d'écran](#8-captures-décran)
9. [Empreintes et intégrité](#9-empreintes-et-intégrité)
10. [Anonymisation](#10-anonymisation)
11. [Registre des preuves](#11-registre-des-preuves)
12. [Stockage et rétention](#12-stockage-et-rétention)
13. [Checklist des preuves à ajouter](#13-checklist-des-preuves-à-ajouter)

---

## 1. Principes

| Principe | Application |
|----------|-------------|
| **Traçabilité** | Chaque preuve est reliée à un test, une date, un opérateur et une commande |
| **Intégrité** | L'original n'est jamais modifié ; une empreinte SHA-256 est calculée |
| **Minimalisme** | Ne conserver que ce qui est nécessaire à la démonstration |
| **Confidentialité** | Stockage protégé, accès limité, anonymisation avant diffusion |
| **Séparation** | Original, copie de travail et version anonymisée sont distingués |
| **Honnêteté** | Une preuve absente reste « À réaliser » ; elle n'est jamais simulée |

> **Règle absolue :** aucun fichier de preuve ne doit être fabriqué, recréé de mémoire ou complété par une sortie inventée. Une preuve absente est une information utile ; une preuve falsifiée détruit la crédibilité de tout le rapport.

### Ce qui constitue une preuve

| Élément | Exemple | Valeur probante |
|---------|---------|-----------------|
| Sortie brute Nmap | `.nmap`, `.xml` | Élevée si horodatée et hashée |
| Capture réseau | `.pcapng` | Élevée si le point de capture est décrit |
| Journal de commandes | Fichier texte horodaté | Moyenne, à corréler |
| Capture d'écran | PNG de Wireshark | Moyenne, facile à modifier |
| Note manuscrite | Photo ou scan | Faible sans autre preuve |
| Paraphrase dans un rapport | « Le port était ouvert » | **Nulle sans preuve brute** |

---

## 2. Arborescence

```text
evidence/
├── README.md                 # Ce document
├── authorization/            # Autorisations signées (ne pas publier)
│   └── .gitkeep
├── nmap/                     # Sorties Nmap brutes
│   └── .gitkeep
├── wireshark/                # Captures .pcapng brutes
│   └── .gitkeep
└── screenshots/              # Captures d'écran, si nécessaire
    └── .gitkeep
```

### Règles de dépôt

- Les preuves réelles ne doivent **pas** être poussées sur un dépôt public sans anonymisation et validation.
- Les autorisations signées contiennent des noms et des signatures : **ne jamais les publier**.
- Les captures peuvent contenir des données personnelles : **ne jamais les publier brutes**.
- Un fichier `.gitkeep` ne prouve rien : il préserve seulement la structure.

---

## 3. Autorisation préalable

Aucun test ne doit commencer sans un document d'autorisation signé. Le modèle ci-dessous est un **formulaire vierge**, pas une autorisation.

### Modèle de formulaire

| Champ | Contenu attendu | Statut |
|-------|-----------------|--------|
| Référence | `AUTORISATION-AAAA-NNN` | **À réaliser** |
| Entité émettrice | Raison sociale, service | **À réaliser** |
| Représentant autorisé | Nom, fonction, coordonnées | **À réaliser** |
| Bénéficiaire | Nom, organisme, fonction | **À réaliser** |
| Objet | Audit réseau autorisé | **À réaliser** |
| Périmètre inclus | CIDR, hôtes, ports, protocoles | **À réaliser** |
| Périmètre exclu | Équipements critiques, hors CIDR | **À réaliser** |
| Tests autorisés | Découverte, scan TCP, `-sV`, captures ARP/DNS/TCP/ICMP | **À réaliser** |
| Tests interdits | Exploitation, brute force, scripts intrusifs | **À réaliser** |
| Fenêtre | Date/heure de début et de fin, fuseau | **À réaliser** |
| Point de capture | Interface, SPAN/TAP, position | **À réaliser** |
| Source de test | Adresse IP, machine, interface | **À réaliser** |
| Contacts | Technique, escalade, astreinte | **À réaliser** |
| Procédure d'arrêt | Critères et moyen de contact | **À réaliser** |
| Données personnelles | Base légale, minimisation, durée | **À réaliser** |
| Signature émettrice | Nom, date, signature | **À réaliser** |
| Signature bénéficiaire | Nom, date, signature | **À réaliser** |

### Vérifications avant exécution

- [ ] Le document est signé par une personne habilitée.
- [ ] Le périmètre correspond exactement aux adresses qui seront testées.
- [ ] La fenêtre horaire est compatible avec l'activité.
- [ ] Les tests autorisés couvrent explicitement `-sV` et les captures.
- [ ] Le contact d'arrêt est joignable pendant toute la fenêtre.
- [ ] Les données personnelles sont traitées conformément à la base légale.
- [ ] Une copie du document est archivée hors du dépôt public.

---

## 4. Convention de nommage

### Format général

```text
<test>_<protocole>_<AAAAMMJJ>T<HHMMSS>Z_<source>
```

| Segment | Format | Exemple |
|---------|--------|---------|
| Test | `test1` à `test7` | `test3` |
| Protocole / objet | minuscules, sans accent | `services`, `arp`, `dns`, `tcp`, `icmp` |
| Horodatage | UTC, ISO 8601 compact | `20261028T140500Z` |
| Source | identifiant court | `poste-a` |

### Exemples

```text
evidence/nmap/test1_host-discovery_20261028T140000Z_poste-a.nmap
evidence/nmap/test1_host-discovery_20261028T140000Z_poste-a.xml
evidence/nmap/test2_ports_20261028T140500Z_poste-a.nmap
evidence/nmap/test3_services_20261028T141000Z_poste-a.nmap
evidence/wireshark/test4_arp_20261028T141500Z_poste-a.pcapng
evidence/wireshark/test5_dns_20261028T142000Z_poste-a.pcapng
evidence/wireshark/test6_tcp_20261028T142500Z_poste-a.pcapng
evidence/wireshark/test7_icmp_20261028T143000Z_poste-a.pcapng
```

> Ces noms sont des **modèles**. Aucun fichier correspondant n'existe dans ce dépôt à ce jour.

### Règles

- Pas d'espaces, pas d'accents, pas de caractères spéciaux.
- Horodatage en UTC, jamais en heure locale ambiguë.
- Un fichier = un test = une exécution.
- Ne jamais écraser une preuve existante : incrémenter l'horodatage.
- Version anonymisée : suffixe `_anonymise`, avec mention explicite.

---

## 5. Preuves attendues par test

| # | Test | Preuve minimale | Preuve complémentaire | Statut |
|---|------|-----------------|-----------------------|--------|
| 1 | `nmap -sn` | Sortie `.nmap` + `.xml` | Journal de commande, table ARP | **À réaliser** |
| 2 | `nmap <IP>` | Sortie `.nmap` + `.xml` | Capture TCP du scan, `--reason` | **À réaliser** |
| 3 | `nmap -sV` | Sortie `.nmap` + `.xml` | Bannière, capture TCP | **À réaliser** |
| 4 | ARP | `.pcapng` | Table ARP avant/après, table CAM | **À réaliser** |
| 5 | DNS | `.pcapng` | Configuration du résolveur, journal | **À réaliser** |
| 6 | TCP | `.pcapng` | Journal client/serveur, résultat Nmap | **À réaliser** |
| 7 | ICMP | `.pcapng` | Sortie `ping`, résultat Nmap `-sn` | **À réaliser** |

### Éléments communs à chaque preuve

| Élément | Pourquoi |
|---------|----------|
| Date et heure UTC | Situer l'observation |
| Opérateur | Responsabilité |
| Version d'outil | Reproductibilité |
| Commande ou filtre exact | Rejouer le test |
| Source et cible | Interpréter le résultat |
| Point de capture | Comprendre ce qui est visible |
| Empreinte SHA-256 | Intégrité |
| Référence d'autorisation | Cadre légal |
| Limites | Éviter la surinterprétation |

---

## 6. Capturer une sortie Nmap

### Commande recommandée

```bash
mkdir -p evidence/nmap
nmap -sn -n --reason \
  -oA evidence/nmap/test1_host-discovery_$(date -u +%Y%m%dT%H%M%SZ)_poste-a \
  192.168.1.0/24
```

> Remplacer `192.168.1.0/24` par le CIDR autorisé et `poste-a` par l'identifiant de la source. La commande ci-dessus est un **modèle** : elle ne doit être lancée qu'après autorisation.

### Fichiers produits

| Extension | Contenu | Usage |
|-----------|---------|-------|
| `.nmap` | Sortie lisible | Rapport, revue humaine |
| `.xml` | Sortie structurée | Traitement, import |
| `.gnmap` | Sortie grepable | Extraction rapide |

### Vérifications

```bash
ls -lh evidence/nmap/
sha256sum evidence/nmap/test1_host-discovery_*.nmap
sha256sum evidence/nmap/test1_host-discovery_*.xml
```

### Journal de commande

Créer un fichier `evidence/nmap/journal_<date>.txt` contenant :

```text
Date UTC        :
Opérateur       :
Autorisation    :
Version Nmap    :
Commande        :
Source          :
Cible           :
Début           :
Fin             :
Résumé          :
Incidents       :
```

> Ne jamais éditer la sortie brute pour « corriger » un résultat. Une sortie erronée se conserve et s'explique dans le rapport.

---

## 7. Conserver une capture Wireshark

### Capture en ligne de commande

```bash
mkdir -p evidence/wireshark
tshark -i <interface> -f "arp" -a duration:60 \
  -w evidence/wireshark/test4_arp_$(date -u +%Y%m%dT%H%M%SZ)_poste-a.pcapng
```

| Option | Rôle |
|--------|------|
| `-i` | Interface de capture |
| `-f` | Filtre de capture BPF |
| `-a duration:60` | Arrêt automatique après 60 secondes |
| `-w` | Fichier de sortie `.pcapng` |

### Après la capture

```bash
# Vérifier le fichier
capinfos evidence/wireshark/test4_arp_*.pcapng

# Calculer l'empreinte
sha256sum evidence/wireshark/test4_arp_*.pcapng

# Compter les paquets du filtre, sans modifier l'original
tshark -r evidence/wireshark/test4_arp_*.pcapng -Y "arp" | wc -l
```

> `capinfos` fournit la date de capture, la durée, le nombre de paquets et l'interface. Ces métadonnées doivent être consignées.

### Métadonnées à noter

| Métadonnée | Source |
|------------|--------|
| Date et heure de début | `capinfos`, premier paquet |
| Durée | `capinfos` |
| Nombre de paquets | `capinfos` |
| Interface | `capinfos` |
| Filtre de capture | Commande `tshark` |
| Version de `tshark` | `tshark --version` |
| Système | `uname -a` |
| Point de capture | Documentation du lab |

---

## 8. Captures d'écran

Une capture d'écran est une **preuve secondaire** : elle est facile à recadrer, à retoucher ou à sortir de son contexte.

### Règles

- Toujours accompagner la capture d'écran du fichier `.pcapng` ou de la sortie brute.
- Afficher l'horodatage, le filtre appliqué et le nom du fichier.
- Ne pas recadrer au point de masquer le contexte.
- Anonymiser les adresses, noms et données personnelles.
- Nommer le fichier selon la convention.
- Ne pas utiliser une capture d'écran pour « prouver » un résultat absent du fichier brut.

### Nommage

```text
evidence/screenshots/test4_arp_filtre_20261028T141500Z_poste-a.png
```

---

## 9. Empreintes et intégrité

### Calcul

```bash
sha256sum evidence/nmap/*.nmap evidence/nmap/*.xml
sha256sum evidence/wireshark/*.pcapng
```

### Vérification ultérieure

```bash
cd evidence && sha256sum -c SHA256SUMS.txt
```

### Fichier de synthèse

```bash
cd evidence
find . -type f \( -name "*.nmap" -o -name "*.xml" -o -name "*.pcapng" \) \
  -print0 | sort -z | xargs -0 sha256sum > SHA256SUMS.txt
```

> Le fichier `SHA256SUMS.txt` doit être conservé avec les preuves. Toute modification, même minime, change l'empreinte et doit être expliquée.

### Chaîne de conservation

| Étape | Action | Trace |
|-------|--------|-------|
| 1 | Capture ou sortie produite | Fichier brut |
| 2 | Empreinte calculée | `SHA256SUMS.txt` |
| 3 | Original déplacé en lecture seule | Droits du fichier |
| 4 | Copie de travail créée | Suffixe `_copie` |
| 5 | Analyse réalisée sur la copie | Journal |
| 6 | Version anonymisée créée | Suffixe `_anonymise` |
| 7 | Preuve archivée | Registre |

---

## 10. Anonymisation

### Données à traiter

| Donnée | Risque | Action |
|--------|--------|--------|
| Adresses IP publiques | Identification de l'organisation | Remplacer par des plages de documentation |
| Adresses MAC | Identification matérielle | Tronquer ou pseudonymiser |
| Noms d'hôtes | Révélation de l'architecture | Remplacer par `host-a`, `host-b` |
| Noms d'utilisateurs | Données personnelles | Supprimer |
| Identifiants, cookies, jetons | Compromission | Supprimer |
| Requêtes DNS | Usages personnels | Pseudonymiser |
| Charges applicatives | Données métier | Tronquer |
| Horodatages précis | Corrélation | Conserver si nécessaire, sinon arrondir |

### Méthodes

- **Rédaction** : supprimer les champs sensibles du fichier exporté.
- **Pseudonymisation** : remplacer par des identifiants cohérents (`host-a`, `host-b`).
- **Troncature** : ne conserver que les en-têtes nécessaires.
- **Agrégation** : remplacer les valeurs individuelles par des statistiques.

### Traçabilité de l'anonymisation

Consigner :

- l'outil utilisé et sa version ;
- les champs modifiés ;
- le fichier source et le fichier de sortie ;
- l'auteur et la date ;
- la différence d'empreinte entre original et version anonymisée.

> Une preuve anonymisée ne doit jamais être présentée comme l'original. Elle doit être clairement étiquetée.

---

## 11. Registre des preuves

### Modèle

| ID | Test | Type | Fichier | Date UTC | Taille | SHA-256 | Original / Copie | Statut |
|----|------|------|---------|----------|--------|---------|------------------|--------|
| EV-01 | T1 | Nmap `.nmap` | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** | Original | **À réaliser** |
| EV-02 | T1 | Nmap `.xml` | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** | Original | **À réaliser** |
| EV-03 | T2 | Nmap `.nmap` | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** | Original | **À réaliser** |
| EV-04 | T3 | Nmap `.nmap` | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** | Original | **À réaliser** |
| EV-05 | T4 | Wireshark `.pcapng` | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** | Original | **À réaliser** |
| EV-06 | T5 | Wireshark `.pcapng` | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** | Original | **À réaliser** |
| EV-07 | T6 | Wireshark `.pcapng` | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** | Original | **À réaliser** |
| EV-08 | T7 | Wireshark `.pcapng` | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** | Original | **À réaliser** |

### Statuts

| Statut | Signification |
|--------|---------------|
| **À réaliser** | Preuve non produite |
| **Produite** | Fichier existant, non encore hashé |
| **Vérifiée** | Empreinte calculée, fichier relu, cohérent |
| **Anonymisée** | Version publiable, distincte de l'original |
| **Archivée** | Conservée selon la politique de rétention |

---

## 12. Stockage et rétention

| Sujet | Règle |
|-------|-------|
| Emplacement | Dossier chiffré, accès nominatif |
| Sauvegarde | Copie de sécurité avant analyse |
| Droits | Lecture seule pour l'original |
| Durée | Définie avec le commanditaire |
| Suppression | Procédure tracée, preuve de suppression |
| Partage | Uniquement après anonymisation et validation |
| Dépôt public | Aucune preuve brute, aucune autorisation signée |

### Ne jamais publier

- une autorisation signée ;
- une capture contenant des identifiants ;
- une adresse IP publique non anonymisée ;
- un nom d'hôte interne ;
- une capture d'écran avec des données personnelles ;
- un fichier `.pcapng` brut.

---

## 13. Checklist des preuves à ajouter

> **État actuel : toutes les cases sont non cochées. Aucune preuve réelle n'est présente dans ce dépôt.**

### Cadre

- [ ] Autorisation signée archivée hors dépôt public — **À réaliser**
- [ ] Périmètre confirmé et daté — **À réaliser**
- [ ] Contacts et procédure d'arrêt documentés — **À réaliser**
- [ ] Journal de bord créé — **À réaliser**

### Tests Nmap

- [ ] Test 1 : sortie `.nmap` — **À réaliser**
- [ ] Test 1 : sortie `.xml` — **À réaliser**
- [ ] Test 1 : empreinte SHA-256 — **À réaliser**
- [ ] Test 2 : sortie `.nmap` — **À réaliser**
- [ ] Test 2 : sortie `.xml` — **À réaliser**
- [ ] Test 2 : empreinte SHA-256 — **À réaliser**
- [ ] Test 3 : sortie `.nmap` — **À réaliser**
- [ ] Test 3 : sortie `.xml` — **À réaliser**
- [ ] Test 3 : empreinte SHA-256 — **À réaliser**

### Captures Wireshark

- [ ] Test 4 ARP : `.pcapng` — **À réaliser**
- [ ] Test 4 ARP : métadonnées `capinfos` — **À réaliser**
- [ ] Test 4 ARP : SHA-256 — **À réaliser**
- [ ] Test 5 DNS : `.pcapng` — **À réaliser**
- [ ] Test 5 DNS : métadonnées `capinfos` — **À réaliser**
- [ ] Test 5 DNS : SHA-256 — **À réaliser**
- [ ] Test 6 TCP : `.pcapng` — **À réaliser**
- [ ] Test 6 TCP : métadonnées `capinfos` — **À réaliser**
- [ ] Test 6 TCP : SHA-256 — **À réaliser**
- [ ] Test 7 ICMP : `.pcapng` — **À réaliser**
- [ ] Test 7 ICMP : métadonnées `capinfos` — **À réaliser**
- [ ] Test 7 ICMP : SHA-256 — **À réaliser**

### Analyse et restitution

- [ ] Tableau de synthèse des résultats complété — **À réaliser**
- [ ] Registre des observations renseigné — **À réaliser**
- [ ] Captures d'écran contextualisées, si utilisées — **À réaliser**
- [ ] Données personnelles anonymisées — **À réaliser**
- [ ] Versions anonymisées relues — **À réaliser**
- [ ] Rapport final relu par un pair — **À réaliser**
- [ ] Preuves archivées selon la politique de rétention — **À réaliser**

> **Rappel :** tant qu'une case n'est pas cochée avec un fichier réel et une empreinte vérifiable, le rapport doit conserver la mention **« À réaliser »**. Ne jamais remplacer une preuve manquante par une supposition.
