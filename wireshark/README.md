# Wireshark — Guide d'analyse de trafic pour un audit autorisé

**Documentation outil et grille de lecture des captures.** Aucune capture réelle n'est incluse dans ce dépôt. Les procédures des quatre tests Wireshark sont dans [traffic-analysis.md](traffic-analysis.md).

![Statut](https://img.shields.io/badge/captures-aucune%20capture%20réelle-orange)

---

## Sommaire

1. [Rôle de Wireshark dans l'audit](#1-rôle-de-wireshark-dans-laudit)
2. [Prérequis et installation](#2-prérequis-et-installation)
3. [Capture et filtrage : deux moteurs distincts](#3-capture-et-filtrage--deux-moteurs-distincts)
4. [Préparer une capture propre](#4-préparer-une-capture-propre)
5. [Interface et colonnes](#5-interface-et-colonnes)
6. [Filtres d'affichage utiles](#6-filtres-daffichage-utiles)
7. [ARP : lecture et limites](#7-arp--lecture-et-limites)
8. [DNS : lecture et limites](#8-dns--lecture-et-limites)
9. [TCP : handshake et limites](#9-tcp--handshake-et-limites)
10. [ICMP : lecture et limites du ping](#10-icmp--lecture-et-limites-du-ping)
11. [Outils statistiques](#11-outils-statistiques)
12. [Conservation et anonymisation des preuves](#12-conservation-et-anonymisation-des-preuves)
13. [Dépannage](#13-dépannage)
14. [Checklist de capture](#14-checklist-de-capture)
15. [Ressources](#15-ressources)

---

## 1. Rôle de Wireshark dans l'audit

Wireshark est un analyseur de protocoles qui capture et décode le trafic visible sur une interface réseau. Dans ce projet, il sert à :

- vérifier ce qui se passe réellement sur le réseau, au-delà des déclarations ;
- observer les échanges ARP, DNS, TCP et ICMP ;
- corréler une observation avec un scan Nmap ;
- documenter un comportement réseau avec des preuves horodatées.

Wireshark **ne remplace pas** :

- une analyse de vulnérabilités ;
- une revue de configuration ;
- un IDS/IPS ;
- une analyse de flux chiffrés sans clés ;
- une preuve d'intention malveillante.

> Une capture ne montre que le trafic qui **atteint l'interface de capture**. Elle ne prouve pas qu'un paquet n'a jamais circulé ailleurs, ni qu'un hôte est sûr.

---

## 2. Prérequis et installation

### Prérequis

| Élément | Exigence | Remarque |
|---------|----------|----------|
| Autorisation | Capture explicitement autorisée | La capture peut contenir des données personnelles |
| Interface | Carte réseau compatible | Ethernet, Wi-Fi, loopback |
| Droits | Accès à `dumpcap` ou capabilities | Ne pas lancer Wireshark en root sans nécessité |
| Stockage | Espace suffisant | Une capture longue peut être volumineuse |
| Horodatage | NTP synchronisé | Indispensable pour corréler les preuves |
| Périmètre | Filtre de capture limité | Éviter de capturer tout le réseau |

### Installation

```bash
# Debian / Ubuntu
sudo apt update && sudo apt install wireshark tshark

# Fedora
sudo dnf install wireshark wireshark-cli

# macOS : paquet officiel ou Homebrew
brew install --cask wireshark

# Vérification
wireshark --version
tshark --version
```

### Autoriser un utilisateur non root (Linux)

L'installation Debian/Ubuntu propose le groupe `wireshark`. L'ajout doit être validé par l'administrateur :

```bash
sudo usermod -aG wireshark "$USER"
# Se déconnecter puis se reconnecter pour appliquer le groupe
```

> Sur Windows, l'installation de **Npcap** est requise. Cocher l'option de capture en mode moniteur uniquement si le matériel et l'autorisation le permettent.

---

## 3. Capture et filtrage : deux moteurs distincts

C'est la confusion la plus fréquente en entretien.

| | Filtre de capture | Filtre d'affichage |
|--|-------------------|--------------------|
| Syntaxe | BPF (*Berkeley Packet Filter*) | Syntaxe Wireshark |
| Moment | Avant la capture | Après la capture |
| Effet | **Jette** les paquets non correspondants | **Masque** les paquets non correspondants |
| Exemple | `tcp port 443` | `tcp.port == 443` |
| Avantage | Réduit le volume et l'exposition des données | Permet d'explorer sans recapturer |
| Limite | Un paquet filtré est perdu | La capture complète reste sur disque |

### Exemples comparés

| Objectif | Filtre de capture (BPF) | Filtre d'affichage |
|----------|-------------------------|--------------------|
| ARP seulement | `arp` | `arp` |
| DNS seulement | `port 53` | `dns` |
| TCP seulement | `tcp` | `tcp` |
| ICMP seulement | `icmp` | `icmp` |
| Un hôte | `host 192.168.1.10` | `ip.addr == 192.168.1.10` |
| Un port | `tcp port 443` | `tcp.port == 443` |

> `arp`, `tcp` et `icmp` sont valides dans les **deux** moteurs. Pour DNS, le filtre de capture usuel est `port 53` (UDP et TCP), tandis que `dns` est un filtre d'affichage. Dans Wireshark, la barre de filtre principale est un filtre **d'affichage**.

### Opérateurs de filtre d'affichage

| Opérateur | Signification | Exemple |
|-----------|---------------|---------|
| `==` | Égal | `ip.ttl == 64` |
| `!=` | Différent | `dns.flags.rcode != 0` |
| `&&` / `and` | ET logique | `tcp && ip.addr == 192.168.1.10` |
| `\|\|` / `or` | OU logique | `arp \|\| icmp` |
| `!` / `not` | Négation | `!arp` |
| `contains` | Contient une chaîne | `dns.qry.name contains "lab"` |
| `matches` | Expression régulière | `http.host matches "example\\.lab"` |
| `in { }` | Appartenance | `tcp.port in {80, 443}` |

Utiliser les opérateurs `==` et non `=` pour comparer une valeur. Les noms de champs sont sensibles à la casse.

---

## 4. Préparer une capture propre

### Avant de capturer

| Question | Pourquoi |
|----------|----------|
| Quelle interface ? | Une capture sur la mauvaise interface est vide |
| Quel filtre de capture ? | Limiter le volume et les données sensibles |
| Quelle durée ? | Éviter une capture illimitée |
| Quel volume maximal ? | Prévenir la saturation du disque |
| Où stocker ? | Choisir un emplacement chiffré et autorisé |
| Qui peut lire le fichier ? | Les captures peuvent contenir des données personnelles |
| Comment anonymiser ? | Prévoir la procédure avant diffusion |

### Bonnes pratiques

- Capturer sur **une seule interface** et **un seul scénario** à la fois.
- Utiliser un **filtre de capture** restrictif (hôte, port, protocole).
- Définir une **durée** ou une **taille** maximale.
- Synchroniser l'horloge (NTP) et noter le fuseau.
- Vérifier que le trafic attendu arrive réellement (`ping`, requête DNS contrôlée).
- Ne pas capturer de trafic personnel, médical, bancaire ou authentifiant sans nécessité.
- Arrêter la capture dès que la preuve est suffisante.

### Fichiers de capture

| Format | Usage | Remarque |
|--------|-------|----------|
| `.pcapng` | Format natif Wireshark | Recommandé, métadonnées et interfaces multiples |
| `.pcap` | Compatibilité large | Limité, mais accepté par de nombreux outils |
| `.cap` | Ancien format | À éviter |

Wireshark enregistre par défaut en `.pcapng`. Conserver la capture brute et ne jamais travailler directement sur l'original : ouvrir une copie.

### Capture en ligne de commande

```bash
# Lister les interfaces
tshark -D

# Capture limitée à 60 secondes, filtre de capture ARP
tshark -i <interface> -a duration:60 -f "arp" -w evidence/wireshark/test4_arp.pcapng

# Capture limitée en taille (ring buffer)
tshark -i <interface> -b filesize:10240 -b files:5 -w capture.pcapng
```

`-i` interface, `-a duration:60` durée, `-f` filtre de capture, `-w` fichier de sortie, `-b` conditions de rotation.

---

## 5. Interface et colonnes

### Colonnes par défaut

| Colonne | Contenu | Utilité |
|---------|---------|---------|
| No. | Numéro de trame | Référence unique dans la capture |
| Time | Horodatage | Corrélation |
| Source | Adresse source | Origine |
| Destination | Adresse destination | Cible |
| Protocol | Protocole décodé | Filtrage visuel |
| Length | Taille de trame | Détection d'anomalies |
| Info | Résumé | Lecture rapide |

### Colonnes utiles à ajouter

Clic droit sur un champ → **Apply as Column** :

- `arp.opcode` : requête (1) ou réponse (2) ;
- `dns.qry.name` : nom interrogé ;
- `dns.flags.rcode` : code de retour DNS ;
- `tcp.flags.str` : combinaison des flags TCP ;
- `tcp.stream` : numéro de flux ;
- `icmp.type` : type ICMP ;
- `frame.time_epoch` : horodatage précis.

### Profils et coloration

- Créer un **profil** par scénario (ARP, DNS, TCP, ICMP) pour retrouver ses filtres.
- Utiliser **View → Coloring Rules** pour distinguer les erreurs (`tcp.analysis.flags`, `dns.flags.rcode != 0`).
- Activer **View → Time Display Format → UTC** pour des preuves cohérentes entre machines.

---

## 6. Filtres d'affichage utiles

### ARP

```text
arp
arp.opcode == 1                 # requête ARP
arp.opcode == 2                 # réponse ARP
arp.src.proto_ipv4 == 192.168.1.10
arp.dst.proto_ipv4 == 192.168.1.10
arp.duplicate-address-detected  # si la dissection le signale
```

### DNS

```text
dns
dns.flags.response == 0                    # requêtes
dns.flags.response == 1                    # réponses
dns.qry.name contains "example"
dns.flags.rcode != 0                       # erreurs DNS
dns.flags.truncated == 1                   # réponse tronquée, bascule TCP possible
dns.time > 1                               # temps de réponse supérieur à 1 s
```

### TCP

```text
tcp
tcp.flags.syn == 1 && tcp.flags.ack == 0   # SYN
tcp.flags.syn == 1 && tcp.flags.ack == 1   # SYN/ACK
tcp.flags.reset == 1                       # RST
tcp.flags.fin == 1                         # FIN
tcp.analysis.retransmission                # retransmission
tcp.analysis.zero_window                   # fenêtre nulle
tcp.port == 443
tcp.stream eq 0
```

### ICMP

```text
icmp
icmp.type == 8            # echo request
icmp.type == 0            # echo reply
icmp.type == 3            # destination unreachable
icmp.type == 11           # time exceeded
icmp.code == 3            # port unreachable (avec type 3)
```

### Combinaisons

```text
arp && eth.src == <mac>
dns && ip.addr == 192.168.1.10
tcp.port == 443 && tcp.flags.syn == 1
icmp.type == 8 && ip.dst == 192.168.1.10
```

> Remplacer les adresses et MAC par celles du laboratoire. Un filtre qui ne retourne rien n'est pas une preuve d'absence : vérifier d'abord que la capture contient du trafic.

---

## 7. ARP : lecture et limites

### Ce que fait ARP

ARP (*Address Resolution Protocol*) associe une adresse IPv4 à une adresse MAC sur un même domaine de diffusion.

```text
Hôte A                                               Hôte B
  │── ARP request « Qui a 192.168.1.10 ? » (broadcast) ──▶│
  │◀── ARP reply « 192.168.1.10 est à aa:bb:cc:dd:ee:ff » ─│
```

| Champ | Signification |
|-------|---------------|
| `arp.opcode` | 1 = requête, 2 = réponse |
| `arp.src.hw_mac` | MAC de l'émetteur |
| `arp.src.proto_ipv4` | IP de l'émetteur |
| `arp.dst.proto_ipv4` | IP recherchée |
| `arp.dst.hw_mac` | MAC cible (souvent nulle dans une requête) |

### Ce que l'on peut observer

- quels hôtes se parlent sur le segment local ;
- la correspondance IP/MAC ;
- des requêtes répétées pour une adresse absente ;
- des changements de MAC pour une même IP ;
- des réponses ARP non sollicitées (*gratuitous ARP*).

### Limites d'ARP

| Limite | Conséquence |
|--------|-------------|
| Portée locale | ARP ne traverse pas un routeur |
| Absence d'authentification | N'importe quel hôte peut répondre |
| Cache | Les entrées peuvent être anciennes |
| usurpation possible | Une MAC peut être falsifiée |
| Broadcast | Le volume peut être important |
| IPv6 | ARP ne concerne pas IPv6 (NDP) |

> **Prudence :** une réponse ARP inattendue ou un changement de MAC est un **indice**, pas une preuve d'attaque. Il faut vérifier la table ARP, la table du commutateur, les journaux DHCP et le contexte (remplacement matériel, bascule HA, machine virtuelle).

### Attaques possibles, sans les exécuter

- **ARP spoofing** : usurpation d'une association IP/MAC pour détourner le trafic.
- **ARP flooding** : saturation de la table CAM/ARP.
- **Gratuitous ARP abusif** : perturbation de la résolution.

Ces attaques ne sont **pas** réalisées dans ce projet. Les mentionner sert uniquement à interpréter un événement observé.

---

## 8. DNS : lecture et limites

### Ce que fait DNS

DNS traduit un nom de domaine en adresse IP, ou inversement.

```text
Client                                               Serveur DNS
  │── Query : A? example.lab ─────────────────────────▶│
  │◀── Response : A 192.168.1.10, TTL 300 ─────────────│
```

| Champ | Signification |
|-------|---------------|
| `dns.qry.name` | Nom interrogé |
| `dns.qry.type` | Type d'enregistrement (A, AAAA, MX, TXT, PTR…) |
| `dns.flags.response` | 0 = requête, 1 = réponse |
| `dns.flags.rcode` | 0 = succès, 2 = SERVFAIL, 3 = NXDOMAIN, 5 = REFUSED |
| `dns.time` | Délai de réponse |
| `dns.flags.truncated` | Réponse tronquée, nouveau transfert TCP possible |

### Ce que l'on peut observer

- les noms résolus par les hôtes du segment ;
- les échecs de résolution ;
- les temps de réponse anormaux ;
- l'utilisation de serveurs DNS non prévus ;
- des requêtes répétitives ou volumineuses.

### Limites de DNS

| Limite | Conséquence |
|--------|-------------|
| Chiffrement | DoH/DoT rend le contenu illisible sans clés |
| Cache | Une réponse en cache n'apparaît pas |
| UDP tronqué | Une réponse volumineuse bascule en TCP |
| Serveur intermédiaire | Le résolveur peut masquer la chaîne réelle |
| Absence de contexte | Un nom inconnu n'est pas nécessairement malveillant |
| Données personnelles | Les noms peuvent révéler des usages |
| Faux négatifs | Une requête peut ne pas passer par l'interface capturée |

> Une requête DNS vers un domaine inconnu n'est **pas** une preuve de compromission. Il faut corréler avec l'hôte source, l'historique, la réputation du domaine et les journaux du résolveur.

---

## 9. TCP : handshake et limites

### Rappel du handshake

```text
Client                                   Serveur
  │──── SYN (seq=x) ─────────────────────▶│
  │◀─── SYN/ACK (seq=y, ack=x+1) ────────│
  │──── ACK (ack=y+1) ───────────────────▶│
  │──── données ─────────────────────────▶│
```

| Étape | Flags | Signification |
|-------|-------|---------------|
| 1 | SYN | Demande d'ouverture, numéro de séquence initial |
| 2 | SYN + ACK | Acceptation, séquence du serveur |
| 3 | ACK | Confirmation, connexion établie |
| Fin | FIN ou RST | Fermeture propre ou brutale |

### Ce que l'on peut observer

- ports ouverts (SYN/ACK) et fermés (RST) ;
- connexions établies et fermées ;
- retransmissions, fenêtre nulle, resets ;
- durée et volume par flux ;
- services contactés.

### Limites de TCP

| Limite | Conséquence |
|--------|-------------|
| Point de capture | Seul le trafic passant par l'interface est visible |
| Chiffrement | Le contenu applicatif reste illisible (TLS) |
| Offloading | La carte réseau peut produire des trames de taille inhabituelle (TSO/LRO) |
| Checksum | Les checksums peuvent être marqués incorrects à cause de l'offloading |
| Retransmissions | Peuvent venir du réseau, de la charge ou d'un bug applicatif |
| RST | Peut venir d'un pare-feu, d'un équilibreur ou du serveur |
| NAT | Les adresses observées peuvent être traduites |

> Un `RST` n'est pas nécessairement une attaque : il peut être une fermeture normale, un refus de connexion ou une réponse d'équipement intermédiaire.

---

## 10. ICMP : lecture et limites du ping

### Ce que fait ICMP

ICMP transporte des messages de contrôle et d'erreur. L'echo request/reply est utilisé par `ping`.

| Type | Nom | Usage |
|------|-----|-------|
| 0 | Echo reply | Réponse à un ping |
| 3 | Destination unreachable | Réseau, hôte, port ou protocole inaccessible |
| 8 | Echo request | Ping |
| 11 | Time exceeded | TTL expiré (traceroute) |
| 13/14 | Timestamp | Horodatage (souvent filtré) |

### Ce que l'on peut observer

- hôtes qui répondent au ping ;
- messages d'erreur révélant un filtrage ou une route absente ;
- TTL et taille des paquets ;
- pertes et latences.

### Limites du ping

| Limite | Conséquence |
|--------|-------------|
| ICMP souvent filtré | Un hôte actif peut ne pas répondre |
| Rate limiting | Les réponses peuvent être retardées ou perdues |
| NAT | Le ping peut être bloqué ou réécrit |
| IPv6 | ICMPv6 est distinct d'ICMPv4 |
| Aucune indication de service | Un ping réussi ne dit rien des ports |
| Usurpation | L'adresse source ICMP peut être falsifiée |

> **Le ping ne prouve pas qu'un hôte est absent.** Il prouve seulement qu'aucune réponse ICMP n'a été reçue dans le délai imparti. Inversement, un ping réussi ne prouve pas que l'hôte est sûr.

### ICMP et scan Nmap

Nmap utilise ICMP parmi ses sondes de découverte. Si l'ICMP est filtré, Nmap peut recourir à TCP SYN/ACK ou à l'ARP sur le réseau local. C'est pourquoi `-sn` peut détecter un hôte qui ne répond pas au `ping` système.

---

## 11. Outils statistiques

| Outil | Chemin | Usage | Précautions |
|-------|--------|-------|-------------|
| Protocol Hierarchy | Statistics → Protocol Hierarchy | Répartition des protocoles | Une part élevée d'ARP peut être normale |
| Conversations | Statistics → Conversations | Volume par paire | Corréler avec l'inventaire |
| Endpoints | Statistics → Endpoints | Hôtes les plus actifs | Ne pas conclure à une attaque |
| IO Graph | Statistics → I/O Graph | Évolution temporelle | Échelle trompeuse |
| Expert Information | Analyze → Expert Information | Anomalies détectées | Lire les descriptions, pas seulement les couleurs |
| Follow TCP Stream | Clic droit → Follow → TCP Stream | Contenu d'un flux | Données sensibles, TLS illisible |

> Les statistiques décrivent un **échantillon**. Elles ne mesurent ni la sécurité, ni l'intention, ni l'exhaustivité du trafic.

---

## 12. Conservation et anonymisation des preuves

### Convention de nommage

```text
evidence/wireshark/test4_arp_20261028T140500Z.pcapng
evidence/wireshark/test5_dns_20261028T141000Z.pcapng
```

Format : `test<numéro>_<protocole>_<AAAAMMJJTHHMMSSZ>.pcapng`.

### Empreinte d'intégrité

```bash
sha256sum evidence/wireshark/test4_arp_20261028T140500Z.pcapng
```

Consigner l'empreinte dans le registre des preuves et ne jamais modifier l'original. Travailler sur une copie.

### Anonymisation

Avant toute diffusion :

- remplacer les adresses IP et MAC par des valeurs de documentation ;
- supprimer les noms d'hôtes internes et les noms d'utilisateurs ;
- masquer les identifiants, cookies et jetons ;
- tronquer ou supprimer les charges utiles applicatives ;
- vérifier les métadonnées du fichier (commentaires, interface, système).

Outils possibles : `editcap`, `tracewrangler`, `tcprewrite`. Toute anonymisation doit être **consignée** : une preuve modifiée n'a plus la même valeur et doit être distinguée de l'original.

### Rétention

| Donnée | Durée recommandée | Remarque |
|--------|-------------------|----------|
| Capture brute | Durée du projet + rétention convenue | Stockage chiffré |
| Sortie anonymisée | Durée de publication | Destination explicite |
| Registre des preuves | Durée de l'audit | Traçabilité |
| Identifiants de connexion | Jamais dans le dépôt | Gestion par coffre |

---

## 13. Dépannage

| Symptôme | Cause probable | Action |
|----------|----------------|--------|
| Aucune interface listée | Droits insuffisants, Npcap absent | Vérifier les droits, réinstaller Npcap |
| Capture vide | Mauvaise interface, trafic absent | Vérifier la route, générer un trafic contrôlé |
| `Permission denied` | Utilisateur hors groupe `wireshark` | Ajouter au groupe, se reconnecter |
| Checksums incorrects | Offloading de la carte | Ne pas conclure à une corruption |
| Trames de taille inhabituelle | TSO/LRO | Désactiver l'offloading pour l'analyse |
| Filtre sans résultat | Syntaxe ou capture insuffisante | Vérifier le filtre, élargir la capture |
| Capture saturée | Volume trop important | Filtre de capture, rotation, durée |
| Wi-Fi illisible | Trafic chiffré ou mode non moniteur | Comprendre la limite, ne pas casser le chiffrement |

---

## 14. Checklist de capture

### Avant

- [ ] Autorisation de capture et périmètre vérifiés
- [ ] Interface identifiée
- [ ] Filtre de capture défini
- [ ] Durée et taille maximales fixées
- [ ] Horloge synchronisée
- [ ] Emplacement de stockage chiffré
- [ ] Procédure d'anonymisation prévue
- [ ] Scénario de test reproductible

### Pendant

- [ ] Capture démarrée avant le scénario
- [ ] Trafic attendu vérifié en direct
- [ ] Aucune donnée hors périmètre capturée
- [ ] Heure de début notée

### Après

- [ ] Capture arrêtée et fichier fermé
- [ ] Empreinte SHA-256 calculée
- [ ] Métadonnées consignées (outil, version, interface)
- [ ] Original conservé en lecture seule
- [ ] Analyse réalisée sur une copie
- [ ] Observations séparées des hypothèses
- [ ] Données sensibles supprimées avant diffusion
- [ ] Statut de la preuve mis à jour dans le rapport

---

## 15. Ressources

- Manuel Wireshark : <https://www.wireshark.org/docs/wsug_html_chunked/>
- Référence des filtres d'affichage : <https://www.wireshark.org/docs/dfref/>
- Filtres de capture BPF : <https://wiki.wireshark.org/CaptureFilters>
- Guide de sécurité et vie privée : <https://www.wireshark.org/docs/wsug_html_chunked/ChIntroSecurity.html>
- Documentation `tshark` : <https://www.wireshark.org/docs/man-pages/tshark.html>

> **Rappel** : une capture peut contenir des données personnelles ou sensibles. Elle ne doit être ni publiée, ni partagée, ni conservée sans cadre explicite.
