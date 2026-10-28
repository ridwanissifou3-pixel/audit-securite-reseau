# Analyse de trafic Wireshark — Tests 4 à 7

**Procédure opérationnelle pour un laboratoire autorisé.** Les adresses, noms et MAC sont des **exemples de documentation**. Aucune capture réelle n'est incluse : chaque preuve manquante est marquée **« À réaliser »**.

![Statut](https://img.shields.io/badge/preuves-à%20réaliser-orange)

---

## Sommaire

- [Cadre général](#cadre-général)
- [Scénario de capture](#scénario-de-capture)
- [Test 4 — Analyse ARP avec le filtre `arp`](#test-4--analyse-arp-avec-le-filtre-arp)
- [Test 5 — Analyse DNS avec le filtre `dns`](#test-5--analyse-dns-avec-le-filtre-dns)
- [Test 6 — Analyse TCP avec le filtre `tcp`](#test-6--analyse-tcp-avec-le-filtre-tcp)
- [Test 7 — Analyse ICMP avec le filtre `icmp`](#test-7--analyse-icmp-avec-le-filtre-icmp)
- [Vérification croisée Nmap / Wireshark](#vérification-croisée-nmap--wireshark)
- [Synthèse des quatre tests](#synthèse-des-quatre-tests)
- [Fiche de capture vierge](#fiche-de-capture-vierge)
- [Questions d'entretien transverses](#questions-dentretien-transverses)

---

## Cadre général

### Prérequis communs

| Prérequis | Vérification | Statut |
|-----------|--------------|--------|
| Autorisation de capture | Document signé, interface et périmètre | **À réaliser** |
| Wireshark ou `tshark` installé | `wireshark --version` | **À réaliser** |
| Interface identifiée | `tshark -D` | **À réaliser** |
| Horloge synchronisée | `timedatectl` ou NTP | **À réaliser** |
| Stockage chiffré | Dossier de preuves protégé | **À réaliser** |
| Scénario reproductible | Étapes de génération de trafic écrites | **À réaliser** |
| Anonymisation | Procédure définie | **À réaliser** |

### Règles

1. **Capturer uniquement le trafic autorisé** : filtre de capture restrictif.
2. **Ne jamais capturer de données personnelles** sans nécessité et sans base légale.
3. **Ne pas modifier l'original** : analyse sur copie, empreinte SHA-256 conservée.
4. **Ne pas provoquer d'attaque** : aucune usurpation ARP, aucun empoisonnement DNS, aucun flood.
5. **Conserver l'horodatage UTC** et la version de l'outil.
6. **Séparer fait, hypothèse et recommandation**.

### Rappel : filtre de capture ou d'affichage

| Usage | Syntaxe | Exemple |
|-------|---------|---------|
| Limiter la capture | BPF, avant capture | `tshark -f "arp"` |
| Explorer la capture | Filtre d'affichage, après capture | `arp` dans la barre Wireshark |

Les filtres de capture `arp`, `tcp`, `icmp` et `port 53` sont valides. Le filtre `dns` est un filtre d'affichage ; `tcp` et `icmp` existent dans les deux moteurs. Un paquet exclu par le filtre de capture **n'existe pas** dans le fichier. Pour les quatre tests, il est recommandé de capturer avec un filtre large (`host <IP>` ou aucun filtre sur un lab isolé) puis d'appliquer le filtre d'affichage. Cela permet de réutiliser la même capture pour plusieurs analyses.

---

## Scénario de capture

Le scénario ci-dessous est un **modèle**, pas un compte rendu d'exécution.

| Étape | Action | Trafic attendu | Test |
|-------|--------|----------------|------|
| 0 | Démarrer la capture sur l'interface du lab | — | Tous |
| 1 | Vider ou consulter le cache ARP, puis contacter un hôte local | ARP request/reply | Test 4 |
| 2 | Résoudre un nom du lab avec `nslookup` ou `dig` | DNS query/response | Test 5 |
| 3 | Ouvrir une connexion TCP contrôlée vers un port autorisé | SYN, SYN/ACK, ACK | Test 6 |
| 4 | Envoyer quelques echo requests vers un hôte autorisé | ICMP request/reply | Test 7 |
| 5 | Arrêter la capture, calculer l'empreinte | — | Tous |

### Commandes de génération de trafic (exemples)

```bash
# ARP : forcer une résolution locale vers un hôte autorisé
ping -c 2 192.168.1.10

# DNS : interroger le résolveur du lab pour un nom de test
nslookup host.lab.local 192.168.1.53
# ou
dig @192.168.1.53 host.lab.local A

# TCP : tester l'ouverture d'un port autorisé, sans authentification
nc -vz 192.168.1.10 80

# ICMP : quatre echo requests
ping -c 4 192.168.1.10
```

> Remplacer chaque adresse, nom et port par ceux du périmètre autorisé. `nc`, `dig` et `ping` peuvent être absents selon le système ; vérifier avant le test. Ne pas utiliser `nmap` pour générer le trafic du test TCP : le but est d'observer un handshake complet, pas un scan.

### TOPOLOGIE ET POINT DE CAPTURE

```text
        Poste analyste (capture)                  Hôte cible
       192.168.1.50 (exemple)                   192.168.1.10 (exemple)
              │                                        │
              └──────────── Commutateur ───────────────┘
                              │
                       Résolveur DNS du lab
                         192.168.1.53 (exemple)
```

| Point de capture | Ce que l'on voit | Ce que l'on ne voit pas |
|------------------|------------------|------------------------|
| Poste analyste | Trafic émis et reçu par ce poste | Trafic entre deux autres hôtes |
| Miroir de port (SPAN) | Trafic du port surveillé | Trafic hors du commutateur |
| TAP réseau | Trafic du lien | Trafic d'autres liens |
| Hôte cible | Trafic de l'hôte | Trafic des autres hôtes |

**Le point de capture doit être décrit dans le rapport** : il conditionne tout ce que l'on peut affirmer.

---

## Test 4 — Analyse ARP avec le filtre `arp`

### Objectif

Observer la résolution d'adresses IPv4 vers MAC sur le segment local, afin de :

- comprendre quels hôtes communiquent au niveau 2 ;
- vérifier la correspondance IP/MAC d'un hôte autorisé ;
- repérer des requêtes sans réponse ou des changements inattendus ;
- documenter une preuve de présence locale.

**Question à laquelle le test répond :** « Quels échanges ARP sont visibles sur ce segment pendant la fenêtre de capture ? »

**Question à laquelle il ne répond pas :** « Y a-t-il une attaque ARP en cours ? »

### Prérequis spécifiques

| Prérequis | Détail |
|-----------|--------|
| Position | Même domaine de diffusion que la cible |
| Autorisation | Capture sur le segment autorisé |
| Interface | Ethernet ou Wi-Fi en mode compatible |
| Scénario | Contact contrôlé avec un hôte connu |
| Table ARP | Relevé avant/après avec `ip neigh` ou `arp -a` |
| Privilèges | Lecture de l'interface (groupe `wireshark` ou root) |

### Filtre

**Filtre d'affichage :**

```text
arp
```

**Filtre de capture équivalent (si l'on ne veut que l'ARP) :**

```bash
tshark -i <interface> -f "arp" -a duration:60 -w evidence/wireshark/test4_arp.pcapng
```

**Sous-filtres utiles :**

```text
arp.opcode == 1
arp.opcode == 2
arp.src.proto_ipv4 == 192.168.1.10
```

### Analyse des éléments

| Élément | Rôle | Ce qu'il apporte |
|---------|------|------------------|
| `arp` | Sélectionne les trames ARP | Isole le protocole |
| `arp.opcode` | Type de message | 1 = requête, 2 = réponse |
| `arp.src.hw_mac` | MAC source | Identité matérielle déclarée |
| `arp.src.proto_ipv4` | IP source | Émetteur |
| `arp.dst.proto_ipv4` | IP recherchée | Cible de la résolution |
| `eth.src` / `eth.dst` | MAC de la trame | Peut différer de la MAC ARP déclarée |
| `frame.time` | Horodatage | Chronologie |

### Résultats attendus

| Observation attendue | Forme générale | Remarque |
|----------------------|----------------|----------|
| Requête ARP | Broadcast « Qui a l'IP X ? » | `arp.opcode == 1` |
| Réponse ARP | « IP X est à MAC Y » | `arp.opcode == 2` |
| Requête répétée | Plusieurs tentatives | Peut indiquer une absence de réponse |
| Réponse gratuite | Annonce sans requête préalable | À interpréter avec prudence |
| Aucune trame ARP | Cache déjà rempli ou aucun trafic local | Vérifier le scénario |

**Résultat honnête possible :** « Aucune trame ARP observée dans la fenêtre ; le cache était probablement déjà renseigné ou le point de capture ne voyait pas le segment. »

### Observations à consigner

- date/heure UTC, interface, point de capture ;
- version Wireshark, filtre utilisé ;
- nombre de requêtes et de réponses ;
- couples IP/MAC observés ;
- adresses ayant répondu plusieurs fois ;
- requêtes restées sans réponse ;
- réponses non sollicitées ;
- différences entre la table ARP locale et la capture ;
- présence de plusieurs MAC pour une même IP ;
- nom du fichier de capture et empreinte SHA-256.

### Interprétation

| Observation | Interprétation prudente | Action de confirmation |
|-------------|-------------------------|------------------------|
| Requête + réponse cohérentes | Résolution normale | Comparer à l'inventaire |
| Plusieurs réponses pour une IP | Conflit possible, bascule HA ou usurpation | Vérifier la table du commutateur, le DHCP, les journaux |
| MAC différente de l'inventaire | Remplacement, VM, ou anomalie | Vérifier l'étiquette, le port switch, l'historique |
| Réponse gratuite répétée | Annonce de changement | Corréler avec un événement d'administration |
| Requêtes sans réponse | Hôte absent, filtré ou hors segment | Tester avec un autre protocole |
| Aucune trame | Capture mal placée ou cache | Refaire avec un scénario explicite |

> **Formulation correcte :** « À [heure], la capture montre une réponse ARP associant [IP] à [MAC]. Cette association diffère de l'inventaire [référence] et doit être vérifiée. »
> **Formulation incorrecte :** « Un ARP spoofing est en cours. »

### Limites d'ARP

- **Portée locale** : ARP ne traverse pas un routeur ; il ne voit pas les autres segments.
- **Cache** : une résolution peut ne pas apparaître si l'entrée est déjà en cache.
- **Absence d'authentification** : une réponse ARP peut être falsifiée ; l'observation ne prouve pas la légitimité.
- **Broadcast** : le volume peut être élevé et noyer l'analyse.
- **IPv6** : ARP ne couvre pas IPv6 (NDP) ; une capture ARP ne dit rien de l'exposition IPv6.
- **Point de capture** : sur un commutateur, le trafic entre deux autres hôtes n'est pas visible sans SPAN/TAP.
- **Chiffrement** : sans objet pour ARP, mais le trafic supérieur peut être chiffré.

### Questions d'entretien — Test 4

**1. Pourquoi ARP est-il indispensable sur un réseau local ?**

> *Parce qu'Ethernet achemine les trames par adresse MAC, alors que les applications utilisent des adresses IP. ARP fournit la correspondance IP → MAC sur le segment local.*

**2. Une réponse ARP inattendue prouve-t-elle une attaque ?**

> *Non. Elle peut venir d'un remplacement matériel, d'une machine virtuelle, d'une bascule de haute disponibilité ou d'une erreur d'inventaire. Il faut corréler avec la table du commutateur, le DHCP et les journaux.*

**3. Pourquoi la capture ARP peut-elle être vide alors que le réseau fonctionne ?**

> *Parce que le cache ARP était déjà rempli, parce que le point de capture ne voit pas le segment, ou parce qu'aucun trafic local n'a été généré pendant la fenêtre.*

**4. Comment vérifier une association IP/MAC ?**

> *Comparer la capture, la table ARP de l'hôte, la table CAM du commutateur, le bail DHCP et l'inventaire. Aucune source unique ne suffit.*

---

## Test 5 — Analyse DNS avec le filtre `dns`

### Objectif

Observer les requêtes et réponses DNS d'un scénario contrôlé, afin de :

- vérifier le résolveur utilisé ;
- comprendre le type de requête et le code de réponse ;
- mesurer le délai de résolution ;
- repérer des échecs ou des comportements inattendus.

**Question à laquelle le test répond :** « Quelles requêtes DNS et quelles réponses sont visibles sur l'interface, pour ce scénario ? »

**Question à laquelle il ne répond pas :** « Cet hôte est-il compromis ? » ou « Tout le trafic DNS est-il visible ? »

### Prérequis spécifiques

| Prérequis | Détail |
|-----------|--------|
| Résolveur | Adresse du DNS du lab autorisée |
| Nom de test | Domaine interne ou domaine de test contrôlé |
| Scénario | Une requête unique, reproductible |
| Capture | Interface du poste ou du résolveur |
| Autorisation | Interrogation DNS explicitement couverte |
| Données | Éviter les noms révélant des personnes ou des projets sensibles |

### Filtre

**Filtre d'affichage :**

```text
dns
```

**Filtre de capture (UDP et TCP) :**

```bash
tshark -i <interface> -f "port 53" -a duration:60 -w evidence/wireshark/test5_dns.pcapng
```

**Sous-filtres utiles :**

```text
dns.flags.response == 0
dns.flags.response == 1
dns.qry.name contains "lab"
dns.flags.rcode != 0
dns.flags.truncated == 1
```

### Analyse des éléments

| Élément | Rôle | Ce qu'il apporte |
|---------|------|------------------|
| `dns` | Sélectionne les messages DNS | Isole le protocole |
| `dns.qry.name` | Nom interrogé | Objet de la résolution |
| `dns.qry.type` | Type d'enregistrement | A, AAAA, MX, TXT, PTR… |
| `dns.flags.response` | Sens du message | Requête ou réponse |
| `dns.flags.rcode` | Code de retour | Succès, NXDOMAIN, SERVFAIL, REFUSED |
| `dns.time` | Délai de réponse | Performance |
| `dns.flags.truncated` | Troncature | Bascule possible vers TCP |

### Résultats attendus

| Observation attendue | Forme générale | Remarque |
|----------------------|----------------|----------|
| Requête | Nom + type, vers le résolveur | `dns.flags.response == 0` |
| Réponse | Enregistrement et TTL | `dns.flags.response == 1` |
| Succès | `rcode == 0` | Une ou plusieurs adresses |
| NXDOMAIN | `rcode == 3` | Nom inexistant |
| SERVFAIL | `rcode == 2` | Échec du résolveur |
| REFUSED | `rcode == 5` | Politique du serveur |
| Réponse tronquée | `truncated == 1` | Nouvelle requête en TCP possible |

**Résultat honnête possible :** « Aucune requête DNS visible : le poste utilisait un cache, un résolveur chiffré (DoH/DoT) ou une autre interface. »

### Observations à consigner

- nom interrogé, type et classe ;
- adresse du résolveur et sens de l'échange ;
- code de réponse et TTL ;
- adresse(s) retournée(s) ;
- délai entre requête et réponse ;
- présence de retransmissions ou de bascule TCP ;
- nombre de requêtes pour un même nom ;
- incohérences entre réponse DNS et connexion observée ;
- fichier de capture et empreinte.

### Interprétation

| Observation | Interprétation prudente | Action de confirmation |
|-------------|-------------------------|------------------------|
| Réponse `rcode == 0` | Résolution réussie | Vérifier la cohérence avec la connexion |
| `NXDOMAIN` | Nom inexistant ou zone incomplète | Vérifier l'orthographe et la zone |
| `SERVFAIL` | Résolveur en difficulté ou zone mal déléguée | Consulter les journaux du résolveur |
| `REFUSED` | Politique d'accès | Vérifier la configuration du résolveur |
| Délai élevé | Résolveur lent, réseau chargé, récursion | Mesurer à plusieurs reprises |
| Résolveur inattendu | Configuration ou redirection | Vérifier les paramètres DHCP/DNS |
| Nom inconnu résolu | Peut être normal (domaine interne) | Vérifier l'inventaire des domaines |

> **Prudence :** une requête vers un domaine inconnu n'est pas une preuve de compromission. Un domaine peut être légitime, CDN, télémétrie, mise à jour ou erreur utilisateur. Corréler avec l'hôte, l'historique et la réputation.

### Limites de DNS

- **Cache** : une réponse servie depuis le cache n'apparaît pas.
- **Chiffrement** : DoH et DoT masquent le contenu ; la capture ne montre que le transport.
- **UDP tronqué** : les réponses volumineuses passent en TCP, parfois hors filtre si le filtre de capture est mal choisi.
- **Point de capture** : le trafic peut passer par une autre interface ou un autre résolveur.
- **Résolveur intermédiaire** : la chaîne de délégation n'est pas visible.
- **Données personnelles** : les noms peuvent révéler des usages ; anonymiser avant partage.
- **Absence de contexte** : un échec DNS peut être applicatif, réseau ou serveur.

### Questions d'entretien — Test 5

**1. Quelle différence entre NXDOMAIN et SERVFAIL ?**

> *NXDOMAIN (rcode 3) signifie que le nom n'existe pas dans la zone interrogée. SERVFAIL (rcode 2) signifie que le résolveur n'a pas pu répondre, pour une raison technique ou de délégation. Les causes et les actions diffèrent.*

**2. Pourquoi une capture DNS peut-elle ne rien montrer alors qu'une résolution a eu lieu ?**

> *Cache local, résolveur chiffré, autre interface, capture démarrée trop tard ou requête effectuée par un autre hôte.*

**3. Le DNS est-il chiffré ?**

> *Classiquement non : les requêtes UDP/53 sont en clair. DoH et DoT chiffrent le transport, mais rendent l'analyse réseau plus difficile et déplacent la confiance vers le résolveur.*

**4. Une requête DNS suspecte suffit-elle à conclure à une compromission ?**

> *Non. Il faut corréler avec l'hôte source, le moment, la réputation du domaine, les autres flux et les journaux. Une requête isolée peut être légitime.*

---

## Test 6 — Analyse TCP avec le filtre `tcp`

### Objectif

Observer un handshake TCP contrôlé et le comportement d'un port autorisé, afin de :

- vérifier l'établissement et la fermeture d'une connexion ;
- comparer l'observation à l'état Nmap ;
- distinguer un refus (RST) d'un filtrage (aucune réponse) ;
- repérer retransmissions et resets inattendus.

**Question à laquelle le test répond :** « Que montre la capture pour cette connexion TCP précise, depuis ce point de capture ? »

**Question à laquelle il ne répond pas :** « Le service est-il sûr ? » ou « Tous les ports sont-ils dans cet état ? »

### Prérequis spécifiques

| Prérequis | Détail |
|-----------|--------|
| Cible | Hôte et port autorisés |
| Client | Commande contrôlée, sans authentification |
| Capture | Sur le client, la cible ou un point intermédiaire |
| Scénario | Une connexion unique, horodatée |
| Autorisation | Test de connexion explicitement couvert |
| Filtrage | Éviter de capturer d'autres flux |

### Filtre

**Filtre d'affichage :**

```text
tcp
```

**Filtre de capture :**

```bash
tshark -i <interface> -f "tcp port 80" -a duration:60 -w evidence/wireshark/test6_tcp.pcapng
```

**Sous-filtres utiles :**

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
tcp.flags.syn == 1 && tcp.flags.ack == 1
tcp.flags.reset == 1
tcp.analysis.retransmission
tcp.analysis.zero_window
tcp.stream eq 0
```

### Analyse des éléments

| Élément | Rôle | Ce qu'il apporte |
|---------|------|------------------|
| `tcp` | Sélectionne TCP | Isole le protocole |
| Flags SYN/ACK/RST/FIN | État de la connexion | Handshake, refus, fermeture |
| `tcp.stream` | Regroupe les paquets d'un flux | Lecture chronologique |
| Numéros de séquence | Ordre des données | Détection de retransmissions |
| `tcp.analysis.*` | Anomalies calculées | Retransmission, fenêtre nulle |
| `tcp.time_delta` | Délai entre paquets | Latence |
| Ports source/destination | Identification du service | Corrélation avec Nmap |

### Résultats attendus

| Observation attendue | Forme générale | Interprétation |
|----------------------|----------------|----------------|
| SYN puis SYN/ACK puis ACK | Handshake complet | Port ouvert et service joignable |
| SYN puis RST | Refus | Port fermé ou rejet explicite |
| SYN sans réponse | Silence | Filtrage, perte ou lenteur |
| FIN/ACK des deux côtés | Fermeture propre | Connexion terminée |
| RST en cours de flux | Rupture | Erreur applicative, pare-feu, timeout |
| Retransmissions | Paquets répétés | Perte, congestion ou problème réseau |
| Fenêtre nulle | Le récepteur ne peut plus lire | Application lente ou bloquée |

**Résultat honnête possible :** « Aucun paquet TCP visible : le port était filtré, le point de capture ne voyait pas le flux, ou la connexion a échoué avant émission. »

### Observations à consigner

- adresse source/destination et ports ;
- interface et point de capture ;
- présence et chronologie du handshake ;
- numéros de séquence relatifs et longueur des segments ;
- RTT approximatif et délais ;
- resets, retransmissions, fenêtre nulle ;
- stream number ;
- état final de la connexion ;
- cohérence avec le scan Nmap ;
- fichier de capture et empreinte.

### Interprétation

| Observation | Interprétation prudente | Ce qu'il faut vérifier |
|-------------|-------------------------|------------------------|
| SYN → SYN/ACK → ACK | Un service écoute et répond | Identification du service, version |
| SYN → RST | Rien n'écoute ou rejet | État Nmap, politique du pare-feu |
| SYN sans réponse | Indéterminé | Filtrage, perte, mauvais point de capture |
| Handshake + données chiffrées | Service joignable, contenu illisible | TLS, certificat, configuration |
| RST après quelques paquets | Connexion interrompue | Journal serveur, pare-feu, timeout |
| Retransmissions nombreuses | Perte ou congestion | Métriques réseau, capture sur un autre point |
| Aucun paquet | Scénario ou capture incorrecte | Vérifier le filtre et l'interface |

> **Formulation correcte :** « À [heure], la capture montre un handshake complet vers [IP]:[port], puis une fermeture par FIN. Cela confirme qu'un service acceptait les connexions TCP à cet instant, sans préjuger de sa sécurité. »

### Limites de TCP

- **Point de capture** : seul le trafic de l'interface est visible.
- **Chiffrement** : le contenu applicatif reste illisible.
- **Offloading** : TSO/LRO peut produire des segments de taille inhabituelle et des checksums marqués invalides.
- **NAT** : les adresses et ports observés peuvent être traduits.
- **Équilibrage** : plusieurs serveurs peuvent répondre derrière une même adresse.
- **RST intermédiaire** : un pare-feu peut réinitialiser la connexion sans que le serveur soit en cause.
- **Capture partielle** : début ou fin de connexion manquants.
- **Un flux réussi ne prouve pas** la sécurité du service.

### Questions d'entretien — Test 6

**1. Décrivez le handshake TCP et ce qu'il prouve.**

> *SYN, SYN/ACK, ACK. Il prouve qu'une pile TCP accepte une connexion sur ce port à cet instant. Il ne prouve ni la légitimité, ni la sécurité, ni la version du service.*

**2. Comment distinguer un port fermé d'un port filtré dans une capture ?**

> *Un port fermé répond généralement par RST ; un port filtré ne répond pas ou renvoie une erreur ICMP administrative. L'absence de réponse reste ambiguë (perte, lenteur).*

**3. Que signifient des retransmissions TCP ?**

> *Le sender n'a pas reçu d'acquittement dans le délai prévu. Cela peut venir d'une perte, d'une congestion, d'un pare-feu ou d'un problème applicatif. Il faut mesurer, pas supposer.*

**4. Pourquoi les checksums peuvent-ils apparaître incorrects ?**

> *À cause de l'offloading (checksum offload) : la carte réseau calcule le checksum après la capture. Ce n'est pas nécessairement une corruption.*

---

## Test 7 — Analyse ICMP avec le filtre `icmp`

### Objectif

Observer des échanges ICMP contrôlés, afin de :

- vérifier la joignabilité IP d'un hôte autorisé ;
- mesurer la perte et la latence ;
- comprendre les messages d'erreur ;
- comparer le résultat à la découverte Nmap.

**Question à laquelle le test répond :** « Quelles réponses ICMP sont observées pour ces echo requests, depuis ce point de capture ? »

**Question à laquelle il ne répond pas :** « L'hôte est-il actif et sûr ? » ou « Tous les services sont-ils disponibles ? »

### Prérequis spécifiques

| Prérequis | Détail |
|-----------|--------|
| Cible | Hôte autorisé |
| Commande | `ping` avec un nombre de paquets borné |
| Capture | Sur le client ou la cible |
| Autorisation | Tests ICMP explicitement couverts |
| Filtrage | Savoir qu'ICMP peut être bloqué |
| Horloge | Synchronisée pour comparer les RTT |

### Filtre

**Filtre d'affichage :**

```text
icmp
```

**Filtre de capture :**

```bash
tshark -i <interface> -f "icmp" -a duration:30 -w evidence/wireshark/test7_icmp.pcapng
```

**Sous-filtres utiles :**

```text
icmp.type == 8
icmp.type == 0
icmp.type == 3
icmp.type == 11
```

### Analyse des éléments

| Élément | Rôle | Ce qu'il apporte |
|---------|------|------------------|
| `icmp` | Sélectionne ICMP | Isole le protocole |
| `icmp.type` | 8 = request, 0 = reply, 3 = unreachable, 11 = time exceeded | Nature du message |
| `icmp.code` | Précision de l'erreur | Ex. port unreachable (type 3, code 3) |
| `icmp.seq` | Numéro de séquence | Appariement requête/réponse |
| `icmp.ident` | Identifiant | Appariement |
| `ip.ttl` | Durée de vie | Indice de nombre de sauts |
| `frame.time_delta` | Délai | Latence approximative |

### Résultats attendus

| Observation attendue | Forme générale | Interprétation |
|----------------------|----------------|----------------|
| Request puis reply | Même identifiant et séquence | Hôte joignable en ICMP |
| Aucune reply | Requests seules | ICMP filtré, hôte absent ou perte |
| Destination unreachable (type 3) | Message d'erreur | Réseau, hôte, protocole ou port inaccessible |
| Time exceeded (type 11) | TTL expiré | Boucle de routage ou traceroute |
| Réponses avec TTL variable | Chemins différents | Routage asymétrique ou charge |

**Résultat honnête possible :** « Aucune réponse ICMP reçue ; l'état de l'hôte reste indéterminé. »

### Observations à consigner

- adresse source/destination et interface ;
- nombre de requêtes émises et de réponses reçues ;
- perte (%) et latence min/moy/max si mesurées ;
- identifiants et séquences ;
- TTL observé ;
- messages d'erreur et codes ;
- cohérence avec Nmap `-sn` ;
- fichier de capture et empreinte.

### Interprétation

| Observation | Interprétation prudente | Ce qu'il faut vérifier |
|-------------|-------------------------|------------------------|
| Reply reçue | Hôte joignable en ICMP à cet instant | Ports, services, filtrage |
| Aucune reply | Indéterminé | Autres sondes, journaux, inventaire |
| TTL faible ou variable | Plusieurs sauts ou chemins | Topologie, routage |
| Type 3 code 0 | Réseau inaccessible | Route, passerelle |
| Type 3 code 1 | Hôte inaccessible | ARP, route, pare-feu |
| Type 3 code 3 | Port unreachable | Souvent lié à un datagramme UDP |
| Type 11 | TTL expiré | Boucle ou traceroute |

> **Formulation correcte :** « Les echo requests vers [IP] n'ont pas reçu de reply dans la fenêtre de capture. L'hôte peut être actif mais filtrer ICMP ; son état est indéterminé. »

### Limites du ping

- **ICMP filtré** : de nombreux pare-feu bloquent l'echo request.
- **Rate limiting** : les réponses peuvent être espacées ou supprimées.
- **NAT** : le ping peut être bloqué ou traduit.
- **ICMPv6** : protocole distinct, non couvert par `icmp`.
- **Aucune information de service** : un ping réussi ne prouve pas qu'un port est ouvert.
- **Usurpation** : l'adresse source peut être falsifiée.
- **Point de capture** : une réponse peut emprunter un autre chemin.
- **Absence de réponse ≠ absence d'hôte** : c'est la limite centrale à retenir.

### Questions d'entretien — Test 7

**1. Un ping qui échoue signifie-t-il que l'hôte est éteint ?**

> *Non. ICMP peut être filtré, limité en débit ou perdu. L'hôte peut être actif et répondre sur d'autres protocoles. Le résultat doit être qualifié d'indéterminé.*

**2. À quoi servent les types ICMP 3 et 11 ?**

> *Le type 3 signale une destination inaccessible (réseau, hôte, protocole ou port). Le type 11 signale un TTL expiré, utilisé notamment par traceroute. Ils aident au diagnostic mais ne prouvent pas la sécurité.*

**3. Pourquoi Nmap peut-il détecter un hôte qui ne répond pas au ping système ?**

> *Parce que Nmap combine plusieurs sondes : ARP local, TCP SYN vers 443, TCP ACK vers 80, ICMP. Une sonde peut réussir là où l'echo request échoue.*

**4. Le TTL permet-il d'identifier le système d'exploitation ?**

> *Il donne un indice sur le nombre de sauts et parfois une valeur initiale probable, mais il peut être modifié, et les systèmes modernes ne se distinguent pas de façon fiable par ce seul champ.*

---

## Vérification croisée Nmap / Wireshark

La valeur d'un audit vient de la **corrélation** entre plusieurs sources. Le tableau ci-dessous aide à confronter un état Nmap à une capture.

| État Nmap | Observation Wireshark attendue | Conclusion possible | Piège |
|-----------|--------------------------------|---------------------|-------|
| `open` | SYN → SYN/ACK → ACK | Service TCP joignable | Ne dit rien de la vulnérabilité |
| `closed` | SYN → RST | Rien n'écoute à cet instant | Le service peut être temporairement arrêté |
| `filtered` | SYN sans réponse, ou ICMP admin. | Indéterminé | Perte réseau possible |
| `open\|filtered` | Réponse ambiguë | Indéterminé | Fréquent en UDP |
| `-sn` hôte détecté | Réponse ARP locale ou ICMP/TCP | Hôte actif à cet instant | Peut être un équipement intermédiaire |
| `-sn` hôte non détecté | Aucune réponse | Indéterminé | Hôte silencieux possible |
| Service identifié | Handshake + bannière ou TLS | Service probable | Bannière falsifiable |

### Exemple de démarche de corrélation

1. Nmap indique `443/tcp open` sur `192.168.1.10` — **fait**.
2. La capture montre un SYN/ACK puis un handshake TLS — **fait**.
3. Le certificat présente un nom différent de l'inventaire — **fait**, à vérifier.
4. Hypothèse : le service est un reverse proxy mal documenté — **hypothèse**.
5. Action : demander la configuration à l'administrateur, vérifier l'inventaire — **à réaliser**.
6. Recommandation : documenter le flux et vérifier la politique TLS — **proportionnée**.

> Ne jamais fusionner ces étapes en une seule phrase affirmant une vulnérabilité.

---

## Synthèse des quatre tests

| Critère | Test 4 ARP | Test 5 DNS | Test 6 TCP | Test 7 ICMP |
|---------|-----------|------------|------------|-------------|
| Filtre d'affichage | `arp` | `dns` | `tcp` | `icmp` |
| Portée | Segment local | Résolveur visible | Point de capture | Point de capture |
| Preuve | `.pcapng` + hash | `.pcapng` + hash | `.pcapng` + hash | `.pcapng` + hash |
| Question | IP ↔ MAC | Résolution | Connexion | Joignabilité |
| Limite majeure | Cache et portée locale | Chiffrement et cache | Chiffrement et point de capture | Filtrage ICMP |
| Statut | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** |

### Ce que ces quatre tests ne prouvent pas

- l'absence d'attaques sur le réseau ;
- l'absence de trafic hors du point de capture ;
- la sécurité des services observés ;
- l'intégrité des données applicatives ;
- la conformité à un référentiel ;
- la sécurité de l'IPv6 ;
- la sécurité organisationnelle.

---

## Fiche de capture vierge

```markdown
### Fiche de capture — [Test 4/5/6/7] [Protocole]

| Champ | Valeur |
|-------|--------|
| Identifiant | CAPT-[N]-[AAAAMMJJ]-[NN] |
| Date et heure UTC | À réaliser |
| Opérateur | À réaliser |
| Autorisation (référence) | À réaliser |
| Point de capture | À réaliser |
| Interface | À réaliser |
| Filtre de capture | À réaliser |
| Filtre d'affichage | À réaliser |
| Scénario généré | À réaliser |
| Outil et version | À réaliser |
| Fichier `.pcapng` | À réaliser |
| Taille du fichier | À réaliser |
| Empreinte SHA-256 | À réaliser |
| Statut | À réaliser / Vérifié |

**Faits observés :**
- À réaliser

**Hypothèses :**
- À réaliser

**Limites :**
- À réaliser

**Actions de confirmation :**
- À réaliser

**Anonymisation effectuée :**
- À réaliser
```

---

## Questions d'entretien transverses

**1. Quelle est la différence entre un filtre de capture et un filtre d'affichage ?**

> *Le filtre de capture (BPF) s'applique avant l'enregistrement et jette les paquets non correspondants. Le filtre d'affichage s'applique après et masque seulement les paquets : la capture reste complète.*

**2. Pourquoi une capture ne prouve-t-elle jamais l'absence de trafic ?**

> *Parce qu'elle ne voit que l'interface et la fenêtre capturées. Le trafic peut passer par un autre chemin, être chiffré, ou ne pas être généré pendant la capture.*

**3. Comment protégez-vous les données d'une capture ?**

> *Autorisation, filtre restrictif, stockage chiffré, accès limité, anonymisation avant partage, suppression des données inutiles et durée de rétention définie.*

**4. Comment relier une observation Wireshark à un résultat Nmap ?**

> *En comparant les horodatages, les adresses, les ports, les flags et les numéros de flux. Une corrélation ne vaut que si le point de capture et la source du scan sont décrits.*

**5. Que faites-vous si vous découvrez un trafic que vous n'étiez pas autorisé à capturer ?**

> *J'arrête la capture, je ne diffuse pas le fichier, je le sécurise ou le supprime selon la procédure, je préviens le responsable et je consigne l'incident.*
