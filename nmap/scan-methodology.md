# Méthodologie de scan Nmap — Tests 1 à 3

**Procédure opérationnelle pour un laboratoire autorisé.** Les adresses utilisées (`192.168.1.0/24`, `192.168.1.10`) sont des **exemples de documentation**. Elles doivent être remplacées par les adresses du périmètre autorisé.

![Statut](https://img.shields.io/badge/preuves-à%20réaliser-orange)

---

## Sommaire

- [Cadre général](#cadre-général)
- [Topologie de référence](#topologie-de-référence)
- [Ordre d'exécution](#ordre-dexécution)
- [Test 1 — Découverte d'hôtes avec `nmap -sn`](#test-1--découverte-dhôtes-avec-nmap--sn)
- [Test 2 — Scan de ports avec `nmap`](#test-2--scan-de-ports-avec-nmap)
- [Test 3 — Détection de services avec `nmap -sV`](#test-3--détection-de-services-avec-nmap--sv)
- [Synthèse des trois tests](#synthèse-des-trois-tests)
- [Fiche de test vierge](#fiche-de-test-vierge)
- [Questions d'entretien transverses](#questions-dentretien-transverses)

---

## Cadre général

### Prérequis communs

| Prérequis | Vérification | Statut |
|-----------|--------------|--------|
| Autorisation écrite | Document signé, périmètre et fenêtre explicites | **À réaliser** |
| Périmètre validé | Adresses confirmées avec le responsable technique | **À réaliser** |
| Laboratoire isolé | Aucun accès vers un réseau tiers | **À réaliser** |
| Nmap installé | `nmap --version` | **À réaliser** |
| Machine de test identifiée | Adresse source et interface notées | **À réaliser** |
| Journal de bord | Fichier horodaté créé | **À réaliser** |
| Procédure d'arrêt | Contact et critère d'interruption connus | **À réaliser** |

### Règles d'exécution

1. **Une commande à la fois**, consignée avant exécution.
2. **Aucune option non prévue** dans l'autorisation (pas de `-A`, `-O`, `-p-`, `--script`).
3. **Aucune conclusion de vulnérabilité** à partir d'un seul scan.
4. **Aucune modification** des sorties brutes.
5. **Arrêt immédiat** si le trafic dépasse le périmètre ou si un équipement réagit anormalement.
6. **Anonymisation** avant publication : noms d'hôtes, adresses MAC, identifiants.

### Conventions de statut

| Statut | Signification |
|--------|---------------|
| **À réaliser** | Test prévu, non exécuté ou preuve non archivée |
| **Vérifié** | Résultat observé, daté et accompagné d'une preuve brute |
| **Hypothèse** | Interprétation à confirmer |
| **Limite** | Ce que le test ne permet pas de conclure |

---

## Topologie de référence

Le schéma ci-dessous est un **modèle pédagogique**, pas la description d'un laboratoire existant.

```text
                     Réseau autorisé : 192.168.1.0/24 (exemple)
                     ┌───────────────────────────────────────┐
                     │                                       │
              ┌──────┴──────┐                        ┌───────┴────────┐
              │ Poste       │                        │ Hôte cible     │
              │ analyste    │                        │ 192.168.1.10   │
              │ 192.168.1.50│                        │ (exemple)      │
              └──────┬──────┘                        └───────┬────────┘
                     │                                       │
                     └───────────────┬───────────────────────┘
                                     │
                             ┌───────┴────────┐
                             │ Commutateur /  │
                             │ routeur du lab │
                             │ 192.168.1.1    │
                             └────────────────┘
```

| Rôle | Adresse exemple | Remarque |
|------|-----------------|----------|
| Réseau cible | `192.168.1.0/24` | Remplacer par le CIDR autorisé |
| Poste analyste | `192.168.1.50` | Adresse source à consigner |
| Hôte cible | `192.168.1.10` | Remplacer par l'hôte autorisé |
| Passerelle | `192.168.1.1` | À exclure si elle n'est pas dans le périmètre |

> **À faire avant le test :** lister les adresses du segment et faire confirmer par le responsable du laboratoire celles qui sont **dans** le périmètre. Un `/24` n'est pas automatiquement autorisé dans son intégralité.

---

## Ordre d'exécution

| Étape | Test | Commande | Dépend de | Durée indicative |
|-------|------|----------|-----------|------------------|
| 1 | Découverte | `nmap -sn <CIDR>` | Autorisation | Secondes à quelques minutes |
| 2 | Scan de ports | `nmap <IP>` | Test 1 | Quelques secondes à quelques minutes |
| 3 | Détection de services | `nmap -sV <IP>` | Test 2 | Quelques minutes |

L'ordre est important : la découverte identifie les hôtes actifs, le scan de ports réduit l'analyse aux ports accessibles, la détection de services qualifie l'exposition. Chaque étape doit être **validée avant de passer à la suivante**.

---

## Test 1 — Découverte d'hôtes avec `nmap -sn`

### Objectif

Identifier les hôtes qui répondent sur le segment autorisé, afin de :

- constituer la liste des cibles potentiellement actives ;
- repérer des adresses non inventoriées ou inattendues ;
- préparer un scan de ports ciblé ;
- documenter la couverture de la découverte et ses angles morts.

**Question à laquelle le test répond :** « Quelles adresses du périmètre ont répondu à au moins une sonde de découverte à l'instant T ? »

**Question à laquelle il ne répond pas :** « Quels hôtes sont réellement présents et quels services exposent-ils ? »

### Prérequis spécifiques

| Prérequis | Détail |
|-----------|--------|
| Périmètre | CIDR exact autorisé, taille connue |
| Position réseau | Idéalement sur le même segment pour bénéficier de l'ARP |
| Droits | Droits élevés recommandés pour l'ARP et les sondes brutes |
| Fenêtre | Période creuse, accord du responsable |
| Exclusions | Passerelle, équipements critiques, adresses hors périmètre |
| Outils | `nmap`, `ip` / `ifconfig`, `ip neigh` pour vérifier la table ARP |

### Commande

```bash
nmap -sn 192.168.1.0/24
```

Variante journalisée, recommandée pour la preuve :

```bash
nmap -sn -n --reason -oA evidence/nmap/test1_host-discovery 192.168.1.0/24
```

### Analyse des options

| Option | Rôle | Effet concret | Remarque |
|--------|------|---------------|----------|
| `-sn` | Désactive le scan de ports | Envoie uniquement des sondes de découverte | Anciennement `-sP` |
| `-n` | Pas de résolution DNS | Évite des requêtes DNS et des noms parasites | Les noms ne sont pas indispensables ici |
| `--reason` | Affiche la raison | Indique la sonde ayant provoqué la réponse | Très utile pour comprendre un silence |
| `-oA <préfixe>` | Triple sortie | Produit `.nmap`, `.xml`, `.gnmap` | Format recommandé pour l'archive |
| `--exclude <IP>` | Exclut une adresse | Protège un équipement hors périmètre | À utiliser si nécessaire |
| `-v` | Verbeux | Affiche la progression | Utile pour le journal |

### Résultats attendus

| Élément attendu | Forme générale | Remarque |
|-----------------|----------------|----------|
| En-tête de version | Version de Nmap et de la base de signatures | À recopier dans le rapport |
| Liste d'hôtes | Une ligne par adresse ayant répondu | Peut être vide |
| Nom DNS | Si résolution activée | Absent avec `-n` |
| Adresse MAC et constructeur | Sur le même segment, avec privilèges | Preuve d'une réponse ARP |
| Latence | Délai de réponse | Peut varier |
| Bilan | Nombre d'hôtes détectés | Ne pas confondre avec le nombre d'hôtes réels |

**Résultat honnête possible :** « N hôtes ont répondu ; M adresses connues de l'inventaire n'ont pas répondu ; la couverture est partielle. »

### Observations à consigner

- date, heure de début et de fin, fuseau horaire ;
- version de Nmap et commande exacte ;
- interface et adresse source ;
- CIDR autorisé et référence du document d'autorisation ;
- nombre d'adresses interrogées et nombre de réponses ;
- liste des adresses ayant répondu, avec MAC et constructeur si disponibles ;
- adresses attendues mais silencieuses, d'après l'inventaire ;
- adresses inconnues ou hors inventaire ;
- toute variation par rapport à un test antérieur ;
- présence éventuelle de doublons ou d'adresses en conflit.

### Interprétation

| Observation | Interprétation prudente | Interprétation interdite |
|-------------|-------------------------|--------------------------|
| Une adresse répond | Un hôte (ou un équipement) est actif à cet instant | « C'est un ordinateur Windows » |
| Une adresse ne répond pas | Silence : filtrage, absence, veille ou perte possible | « Il n'y a pas d'hôte » |
| MAC d'un constructeur connu | Indice sur le matériel | « C'est forcément le bon équipement » |
| Adresse inconnue | Écart d'inventaire à vérifier | « C'est un équipement malveillant » |
| Réponse ARP mais pas ICMP | Hôte actif local, ICMP filtré | « L'hôte est hors ligne » |

**Formulation recommandée pour le rapport :**

> « À [horodatage], la découverte `nmap -sn` sur [CIDR autorisé] a recueilli N réponses. Les adresses [X, Y] figurent dans l'inventaire ; l'adresse [Z] n'y figure pas et doit être rapprochée du registre DHCP et de la table du commutateur. Les adresses [A, B] n'ont pas répondu malgré leur présence attendue : leur état est indéterminé. »

### Limites

- **ICMP filtré** : un hôte actif peut ne pas répondre à l'echo request.
- **Pare-feu TCP** : les sondes SYN et ACK peuvent être bloquées.
- **Portée ARP** : l'ARP ne traverse pas un routeur ; hors segment, la découverte dépend des sondes IP.
- **Sondes UDP** : seules les sondes `-PU` sont concernées ; les réponses sont rares et l'apport reste limité.
- **NAT** : plusieurs hôtes peuvent partager une adresse ; un hôte peut être invisible.
- **VLAN et ACL** : segmentation pouvant masquer des hôtes.
- **IPv6** : non couvert par un scan IPv4.
- **Hôtes en veille** : un poste éteint ou en veille ne répond pas.
- **Instantané** : le résultat est valable à l'instant du test, pas de façon permanente.
- **Pas de ports** : aucune conclusion sur les services ou les vulnérabilités.

### Questions d'entretien — Test 1

**1. Pourquoi `-sn` est-il préférable à un scan complet pour commencer ?**

> *Pistes :* il limite la charge et l'intrusion, il identifie les cibles actives, il permet de justifier un scan ciblé ensuite, et il respecte le principe de minimisation. Il ne dispense pas d'une autorisation.

**2. Un hôte ne répond pas à `-sn`. Est-il absent ?**

> *Non.* Il peut filtrer l'ICMP et les sondes TCP, être en veille, derrière un NAT, ou avoir perdu les paquets. La bonne formulation est « non détecté à cet instant », pas « absent ».

**3. Pourquoi l'ARP est-il plus fiable que l'ICMP sur un réseau local ?**

> *Parce que l'ARP est nécessaire pour communiquer sur le segment et est rarement filtré. Il ne fonctionne toutefois que sur le même domaine de diffusion et ne prouve pas l'état des ports.*

**4. Comment vérifier qu'une adresse inconnue n'est pas un simple bail DHCP dynamique ?**

> *Comparer avec la table DHCP, le bail, la table ARP du commutateur, l'inventaire et le journal de l'équipement. Aucune conclusion ne doit reposer sur la seule découverte Nmap.*

---

## Test 2 — Scan de ports avec `nmap`

### Objectif

Déterminer, sur un hôte autorisé, quels ports TCP parmi les 1 000 plus courants sont **ouverts**, **fermés** ou **filtrés**, afin de :

- cartographier la surface d'exposition visible depuis la source de test ;
- hiérarchiser les services à qualifier ;
- distinguer une absence de service d'un filtrage ;
- préparer la détection de versions.

**Question à laquelle le test répond :** « Quels états Nmap sont observés sur les 1 000 ports TCP les plus courants de cet hôte, depuis cette source, à cet instant ? »

**Question à laquelle il ne répond pas :** « Cet hôte est-il sécurisé ? » ou « Quels sont tous les ports ouverts ? »

### Prérequis spécifiques

| Prérequis | Détail |
|-----------|--------|
| Hôte cible | Adresse unique confirmée dans le périmètre |
| Autorisation | Scan de ports explicitement couvert |
| Droits | Root pour `-sS`, sinon `-sT` |
| Réseau | Route stable, pas de proxy interceptant |
| Fenêtre | Accord de l'équipe responsable de l'hôte |
| Tests préalables | Test 1 réalisé ou hôte confirmé actif |

### Commande

```bash
nmap 192.168.1.10
```

Variante journalisée et explicite :

```bash
nmap -n --reason -oA evidence/nmap/test2_ports 192.168.1.10
```

Variantes ciblées, uniquement si l'autorisation le prévoit :

```bash
# Ports explicitement listés
nmap -p 22,80,443 192.168.1.10

# Hôte supposé actif mais filtrant la découverte
nmap -Pn -p 22,80,443 192.168.1.10
```

### Analyse des options

| Élément | Rôle | Effet | Remarque |
|---------|------|-------|----------|
| `192.168.1.10` | Cible | Aucune option de scan : comportement par défaut | Remplacer par l'hôte autorisé |
| *(défaut)* | Découverte + scan TCP | 1 000 ports TCP les plus courants | Ni UDP, ni tous les ports |
| `-sS` | TCP SYN | Semi-connexion si privilèges | Souvent utilisé par défaut |
| `-sT` | TCP connect | Handshake complet | Visible côté application |
| `-Pn` | Pas de découverte | Traite la cible comme active | Ne prouve pas qu'elle l'est |
| `-p` | Sélection des ports | Limite ou étend le périmètre | `-p-` = 65 535 ports, à éviter sans accord |
| `-n` | Pas de DNS | Réduit le bruit | — |
| `--reason` | Justification de l'état | Affiche la réponse ou l'absence | Essentiel pour l'analyse |
| `-oA` | Archivage | Trois formats | Preuve |
| `-T3` / `-T4` | Temporisation | Normale / rapide en lab | `-T5` déconseillé |

### Résultats attendus

| Élément attendu | Forme générale | Remarque |
|-----------------|----------------|----------|
| En-tête | Version, cible, heure | Vérifier l'heure |
| Tableau des ports | `PORT STATE SERVICE REASON` | Lignes pour les ports ouverts et, selon les options, les ports filtrés ; les fermés peuvent être résumés |
| États | `open`, `closed`, `filtered` | Voir la signification ci-dessous |
| Bilan | Nombre de ports par état | Ne pas extrapoler |
| Avertissements | DNS, route, privilèges | À consigner |

**Résultat honnête possible :** « Aucun port ouvert observé sur les 1 000 ports TCP testés ; des ports filtrés subsistent ; l'absence d'ouverture n'exclut pas un service sur un port non testé ou en UDP. »

### Lire les états

| État | Ce que Nmap a observé | Ce que l'on peut dire | Ce que l'on ne peut pas dire |
|------|-----------------------|-----------------------|------------------------------|
| `open` | SYN/ACK ou équivalent | Un processus accepte des connexions | Qu'il est vulnérable |
| `closed` | RST ou ICMP unreachable | L'hôte répond, rien n'écoute | Que le service n'existe jamais |
| `filtered` | Pas de réponse ou ICMP admin. | Le résultat est indéterminé | Que le port est fermé |
| `open\|filtered` | Réponse ambiguë | Indéterminé | Qu'il est ouvert |

### Observations à consigner

- commande exacte, version Nmap, privilèges utilisés ;
- adresse source, interface, route vers la cible ;
- date et heure, durée du scan ;
- type de scan effectif (`-sS` ou `-sT`) ;
- liste des ports avec état, service et **raison** ;
- ports filtrés et hypothèses de filtrage ;
- ports habituellement attendus mais absents ;
- messages d'erreur ou timeouts ;
- comparaison éventuelle avec un scan de référence.

### Interprétation

| Observation | Interprétation | Action de confirmation |
|-------------|----------------|------------------------|
| Port 22 `open` | Service SSH exposé | Test 3 : version, bannière, politique d'authentification |
| Port 80 `open` | Service HTTP exposé | Vérifier redirection, en-têtes, version |
| Port 443 `open` | Service HTTPS exposé | Vérifier certificat, TLS, version |
| Port 3389 `open` | Bureau à distance exposé | Vérifier restriction d'accès et authentification |
| Port `closed` | Rien n'écoute à cet instant | Ne pas conclure à une absence permanente |
| Port `filtered` | Filtrage ou perte | Vérifier la politique du pare-feu |
| Aucun port ouvert | Rien d'observé sur le périmètre testé | Ne pas conclure à la sécurité |

**Exemple de formulation :**

> « Depuis [source], le scan des 1 000 ports TCP les plus courants de [hôte] a observé [n] ports ouverts, [m] fermés et [k] filtrés. Les ports [liste] sont exposés. La criticité ne peut être établie sans identification des services (test 3) ni connaissance de la politique d'accès. »

### Limites

- **Périmètre partiel** : 1 000 ports TCP seulement ; les ports hauts et l'UDP ne sont pas couverts.
- **Position de la source** : un scan interne ne reflète pas l'exposition depuis Internet.
- **Filtrage dynamique** : un pare-feu peut réagir différemment selon la source ou la charge.
- **Timeouts** : un port lent peut apparaître `filtered` à tort.
- **Services éphémères** : un service peut démarrer ou s'arrêter entre deux scans.
- **Load balancing** : plusieurs serveurs derrière une adresse peuvent répondre différemment.
- **NAT** : le port observé peut être traduit.
- **Aucune preuve de vulnérabilité** : l'état ne décrit que l'accessibilité.

### Questions d'entretien — Test 2

**1. Quelle est la différence entre un scan SYN et un scan connect ?**

> *Le scan SYN (`-sS`) envoie un SYN et interprète la réponse sans terminer le handshake ; il exige des privilèges et laisse moins de traces applicatives. Le scan connect (`-sT`) utilise `connect()` et établit une connexion complète, visible dans les journaux applicatifs ; il fonctionne sans privilèges.*

**2. Un port `filtered` est-il fermé ?**

> *Non.* `filtered` signifie que Nmap n'a pas obtenu de réponse concluante : le paquet a pu être bloqué, perdu ou retardé. L'état est indéterminé.

**3. Pourquoi un scan par défaut ne suffit-il pas à cartographier l'exposition ?**

> *Il ne couvre que 1 000 ports TCP courants, pas l'UDP, pas les ports hauts, et dépend de la position de la source. Il donne une vue partielle à un instant donné.*

**4. Un port ouvert est-il une vulnérabilité ?**

> *Non.* C'est une surface d'exposition. La vulnérabilité dépend du service, de sa version, de sa configuration, de l'authentification, du chiffrement, de l'exposition et des contrôles compensatoires. Elle doit être confirmée par une analyse dédiée.

---

## Test 3 — Détection de services avec `nmap -sV`

### Objectif

Qualifier les services associés aux ports ouverts détectés lors du test 2 :

- identifier le protocole applicatif ;
- proposer un produit et une version ;
- évaluer le niveau de confiance de l'identification ;
- préparer une vérification de politique de patch et de configuration.

**Question à laquelle le test répond :** « Quel service Nmap pense-t-il identifier derrière chaque port ouvert, avec quel produit, quelle version et quel indice de confiance ? »

**Question à laquelle il ne répond pas :** « Ce service est-il vulnérable ? »

### Prérequis spécifiques

| Prérequis | Détail |
|-----------|--------|
| Résultats du test 2 | Liste des ports ouverts à qualifier |
| Autorisation | Sondes applicatives explicitement couvertes |
| Fenêtre | Éviter les heures de pointe |
| Équipement | Ne pas cibler un équipement fragile ou critique sans accord |
| Droits | Root recommandé, mais `-sV` peut fonctionner sans |
| Base de signatures | Version de Nmap notée |

### Commande

```bash
nmap -sV 192.168.1.10
```

Variante journalisée, limitée aux ports ouverts connus :

```bash
nmap -sV --version-light --reason -p 22,80,443 \
  -oA evidence/nmap/test3_services 192.168.1.10
```

> Remplacer la liste `-p` par les ports réellement observés au test 2. Ne pas recopier une liste d'exemple sans vérification.

### Analyse des options

| Option | Rôle | Effet | Remarque |
|--------|------|-------|----------|
| `-sV` | Détection de version | Envoie des sondes applicatives | Peut être journalisé côté cible |
| `-p` | Restreint les ports | Réduit la charge | À aligner sur le test 2 |
| `--version-intensity <0-9>` | Intensité | Défaut 7 | Baisser pour un lab sensible |
| `--version-light` | Intensité 2 | Moins de sondes | Moins précis |
| `--version-all` | Intensité 9 | Toutes les sondes | Plus intrusif, à éviter par défaut |
| `--reason` | Justification | Contexte des états | Utile au rapport |
| `-oA` | Archivage | Trois formats | Preuve |
| `-n` | Pas de DNS | Réduit le bruit | — |

### Résultats attendus

| Élément attendu | Forme générale | Remarque |
|-----------------|----------------|----------|
| Port et état | `22/tcp open` | Reprendre l'état du test 2 |
| Service | `ssh`, `http`, `https` | Nom générique |
| Produit | `OpenSSH`, `nginx`, `Apache httpd` | Peut être absent |
| Version | Numéro ou indication | Peut être partiel |
| Info complémentaire | Système, module, protocole | Selon les signatures |
| Indice de confiance | Valeur dans la sortie XML | À consigner |
| CPE | Identifiant de plateforme | Aide à la comparaison avec un inventaire |

**Résultat honnête possible :** « Le service n'a pas pu être identifié avec certitude ; la version est inconnue. » Cette mention est préférable à une affirmation non fondée.

### Observations à consigner

- commande, version Nmap, intensité utilisée ;
- ports réellement testés ;
- service, produit, version et informations complémentaires ;
- indice de confiance et éventuel CPE ;
- bannière brute si elle est collectée ;
- incohérences entre le port attendu et le service observé ;
- services non identifiés ;
- erreurs, timeouts, refus de connexion ;
- captures ou sorties brutes associées.

### Interprétation

| Observation | Interprétation | Suite |
|-------------|----------------|-------|
| `ssh` + `OpenSSH` + version | Service SSH identifié | Vérifier la politique de patch et l'authentification |
| `http` + `nginx` + version | Serveur web identifié | Vérifier TLS, en-têtes, version maintenue |
| Service inconnu | Identification non concluante | Analyse manuelle autorisée ou revue de configuration |
| Port 443 avec service `http` | Peut-être un proxy ou un service mal configuré | Vérifier avec une requête applicative encadrée |
| Version très ancienne | Indice de risque | Confirmer la fin de support auprès de l'éditeur |
| Version identique à l'inventaire | Cohérence | Vérifier tout de même les correctifs |

**Ce qu'il est permis d'écrire :**

> « Le port 22/TCP est ouvert et Nmap identifie un service SSH, produit [X], version [Y], avec un indice de confiance [Z]. Ce constat doit être rapproché de l'inventaire et de la politique de patch. Aucune vulnérabilité n'est affirmée à ce stade. »

**Ce qu'il est interdit d'écrire sans analyse dédiée :**

> « OpenSSH [Y] est vulnérable à CVE-XXXX-YYYY. »

Une correspondance version/CVE doit être vérifiée sur une source officielle (éditeur, NVD), en tenant compte des correctifs rétroportés et de la configuration réelle.

### Limites

- **Bannière falsifiable** : un service peut annoncer une version fausse.
- **Correctifs rétroportés** : la version affichée peut contenir des correctifs non reflétés dans le numéro.
- **Reverse proxy** : le service réel peut être masqué.
- **TLS** : sans négociation adaptée, le contenu reste chiffré.
- **Authentification** : les services authentifiés ne révèlent pas leur version.
- **Sondes** : l'intensité et le timeout influencent la détection.
- **Faux positifs** : une signature peut correspondre à plusieurs produits.
- **Pas d'analyse de vulnérabilité** : `-sV` ne teste aucune faille.

### Questions d'entretien — Test 3

**1. `-sV` détecte-t-il les vulnérabilités ?**

> *Non.* Il identifie un service et une version probable. La vulnérabilité doit être confirmée par une analyse dédiée, une revue de configuration ou un test autorisé spécifique.

**2. Pourquoi la version détectée peut-elle être trompeuse ?**

> *Bannière personnalisée, proxy, version rétroportée par la distribution, service encapsulé ou signature imprécise. Il faut croiser avec l'inventaire et l'éditeur.*

**3. Comment réduire l'impact de `-sV` sur un équipement sensible ?**

> *Restreindre les ports avec `-p`, utiliser `--version-light`, éviter les heures de pointe, prévoir une fenêtre d'arrêt et surveiller les journaux. En cas de doute, ne pas lancer le test.*

**4. Quelle différence entre `-sV` et `-O` ?**

> *`-sV` identifie les services et versions ; `-O` tente de deviner le système d'exploitation à partir de la pile TCP/IP. Les deux sont indicatifs et peuvent être faussés. `-O` n'est pas utilisé dans ce projet.*

---

## Synthèse des trois tests

| Critère | Test 1 | Test 2 | Test 3 |
|---------|--------|--------|--------|
| Question | Quels hôtes répondent ? | Quels ports sont accessibles ? | Quels services sont exposés ? |
| Commande | `nmap -sn CIDR` | `nmap IP` | `nmap -sV IP` |
| Couverture | Hôtes du segment | 1 000 ports TCP | Ports ouverts identifiés |
| Preuve attendue | Sortie `.nmap` + `.xml` | Sortie `.nmap` + `.xml` | Sortie `.nmap` + `.xml` |
| Risque | Faible | Modéré | Modéré à plus élevé |
| Statut dans ce dépôt | **À réaliser** | **À réaliser** | **À réaliser** |

### Ce que ces trois tests ne prouvent pas

- l'absence de vulnérabilités ;
- l'absence de services sur les ports non testés ou en UDP ;
- la sécurité de la configuration applicative ;
- la conformité à un référentiel ;
- l'absence d'exposition depuis un autre point du réseau ;
- la sécurité de l'IPv6 ;
- la sécurité organisationnelle.

---

## Fiche de test vierge

À dupliquer pour chaque test et à joindre au rapport.

```markdown
### Fiche de test — [Numéro et titre]

| Champ | Valeur |
|-------|--------|
| Identifiant | TEST-[N]-[AAAAMMJJ]-[NN] |
| Date et heure (UTC) | À réaliser |
| Opérateur | À réaliser |
| Autorisation (référence) | À réaliser |
| Périmètre testé | À réaliser |
| Source (IP, interface) | À réaliser |
| Cible | À réaliser |
| Outil et version | À réaliser |
| Commande exacte | À réaliser |
| Sortie brute (fichier) | À réaliser |
| Empreinte SHA-256 | À réaliser |
| Statut | À réaliser / Vérifié |

**Faits observés :**
- À réaliser

**Hypothèses :**
- À réaliser

**Limites du test :**
- À réaliser

**Actions de confirmation :**
- À réaliser

**Recommandations proportionnées :**
- À réaliser
```

---

## Questions d'entretien transverses

**1. Comment garantissez-vous que le scan reste dans le périmètre autorisé ?**

> *Document signé, liste d'adresses confirmée, exclusions explicites, fenêtre horaire, journal de commandes, surveillance pendant l'exécution et arrêt immédiat en cas d'écart.*

**2. Comment différenciez-vous un fait d'une hypothèse dans un rapport ?**

> *Un fait est directement observable et daté (sortie brute, trame). Une hypothèse est une interprétation, explicitement étiquetée, avec le test qui permettrait de la confirmer.*

**3. Pourquoi un scan négatif ne suffit-il pas à rassurer ?**

> *Parce qu'il est partiel : ports, protocoles, position de la source, filtrage, délais. La sécurité ne se déduit pas de l'absence de réponse.*

**4. Que faites-vous si un scan déclenche une alerte ou ralentit un service ?**

> *J'arrête le test, je préviens le contact technique, je consigne l'incident, je conserve les preuves et je ne reprends qu'après accord explicite.*

**5. Peut-on scanner un réseau public pour « vérifier » ?**

> *Non.* Un scan non autorisé peut constituer une atteinte à un système de traitement automatisé de données. Seuls les réseaux possédés ou explicitement autorisés peuvent être testés.
