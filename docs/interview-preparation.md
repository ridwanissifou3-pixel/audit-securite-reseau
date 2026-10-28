# Guide d'entretien — stage en réseaux et cybersécurité

**Support de préparation orale fondé sur le projet Network Security Audit Lab.** Ce guide propose des formulations, pas un discours à réciter. Il ne doit jamais servir à revendiquer une expérience, un résultat ou une vulnérabilité qui n'existe pas.

![Statut](https://img.shields.io/badge/expérience-à%20personnaliser-orange)

---

## Sommaire

1. [Comment utiliser ce guide](#1-comment-utiliser-ce-guide)
2. [Présentation orale de 60 à 90 secondes](#2-présentation-orale-de-60-à-90-secondes)
3. [Dix questions techniques](#3-dix-questions-techniques)
4. [Cinq questions difficiles](#4-cinq-questions-difficiles)
5. [Questions à poser au recruteur](#5-questions-à-poser-au-recruteur)
6. [Mise en situation et exercice blanc](#6-mise-en-situation-et-exercice-blanc)
7. [Vocabulaire et formulations](#7-vocabulaire-et-formulations)
8. [Checklist avant l'entretien](#8-checklist-avant-lentretien)
9. [Ce qu'il ne faut pas dire](#9-ce-quil-ne-faut-pas-dire)

---

## 1. Comment utiliser ce guide

### Trois principes

| Principe | Application |
|----------|-------------|
| **Honnêteté** | Dire ce qui a été fait, ce qui reste à faire et ce qui n'a pas été testé |
| **Structure** | Répondre en 3 temps : contexte, action, résultat ou limite |
| **Humilité technique** | Reconnaître une limite est une compétence, pas une faiblesse |

### Le format STAR

Pour les questions comportementales :

| Lettre | Contenu | Exemple de trame |
|--------|---------|------------------|
| **S** — Situation | Contexte | « Dans le cadre de mon projet de lab… » |
| **T** — Tâche | Ce que vous deviez faire | « Je devais documenter une méthode reproductible… » |
| **A** — Action | Ce que vous avez fait | « J'ai rédigé les procédures, défini les preuves… » |
| **R** — Résultat | Ce que cela a produit | « Un dépôt structuré, avec les tests encore à réaliser. » |

> **Attention :** le résultat doit être réel. « J'ai conçu », « j'ai documenté », « j'ai préparé » sont des verbes sûrs. « J'ai audité », « j'ai trouvé », « j'ai exploité » ne doivent être utilisés que si c'est vrai.

---

## 2. Présentation orale de 60 à 90 secondes

### Structure et minutage

| Temps | Séquence | Objectif |
|-------|----------|----------|
| 0–10 s | Accroche | Se présenter et nommer le projet |
| 10–30 s | Problème | Pourquoi un audit réseau méthodique est utile |
| 30–55 s | Réalisation | Ce qui a été construit : 7 tests, outils, rapport |
| 55–75 s | Rigueur | Preuves, éthique, limites |
| 75–90 s | Ouverture | Ce que vous cherchez et ce que vous voulez apprendre |

### Script principal — version « tests documentés, non exécutés »

> **À utiliser uniquement si les tests n'ont pas encore été réalisés.** Durée approximative : 75 secondes.

« Bonjour, je m'appelle ISSIFOU Ridwan et je poursuis une formation Cisco / IBM. Pour montrer ma démarche en cybersécurité, j'ai conçu **Network Security Audit Lab**, un projet qui documente sept tests réseau avec Nmap et Wireshark : découverte d'hôtes, scan de ports, détection de services, puis analyse ARP, DNS, TCP et ICMP.
Le projet part d'un constat simple : un scan ne vaut rien sans périmètre autorisé, sans preuve et sans limites explicites. J'ai donc structuré un cadre d'autorisation, des procédures avec objectif, prérequis, commandes et interprétation, ainsi qu'un modèle de rapport et un registre de preuves avec empreintes SHA-256.
Je distingue volontairement les faits observés, les hypothèses et les recommandations : un port ouvert n'est pas une vulnérabilité, et un scan limité ne prouve pas qu'un réseau est sûr.
Les tests restent à exécuter dans un laboratoire autorisé ; le dépôt est prêt, avec une checklist des preuves à produire. Je cherche un stage où je peux appliquer cette rigueur sur des infrastructures réelles et apprendre aux côtés d'une équipe réseau ou SOC. »

### Variante — version « tests exécutés »

> **À n'utiliser que si les sept tests ont réellement été exécutés et que les preuves sont archivées.** Remplacer les crochets par des faits vérifiés.

« … J'ai conçu **Network Security Audit Lab**, un projet qui documente sept tests réseau avec Nmap et Wireshark.
Je l'ai déroulé sur un laboratoire autorisé : [nombre] hôtes ont répondu à la découverte, [nombre] ports ouverts ont été observés, et j'ai corrélé ces résultats avec des captures ARP, DNS, TCP et ICMP. Chaque résultat est accompagné d'une sortie brute, d'une capture horodatée et d'une empreinte SHA-256.
Ce que j'en retiens, c'est surtout la méthode : distinguer un fait d'une hypothèse, documenter les limites, et ne jamais conclure qu'un port ouvert est une vulnérabilité. J'ai aussi appris à expliquer pourquoi un hôte silencieux n'est pas forcément absent.
Je cherche un stage où je peux approfondir l'analyse réseau et la détection, avec une exigence de traçabilité. »

### Livraison orale

| Point | Conseil |
|-------|---------|
| Débit | Environ 150 mots par minute ; ralentir sur les chiffres |
| Regard | Alterner entre les interlocuteurs, ne pas lire ses notes |
| Mains | Ouvertes, posées, sans agitation |
| Chiffres | Ne citer que ceux que l'on peut prouver |
| Silence | Accepter une pause après une phrase clé |
| Fin | Terminer par une phrase complète, pas par « voilà » |

### Version courte — 30 secondes

> « J'ai conçu un référentiel d'audit réseau avec Nmap et Wireshark : sept tests documentés, un cadre d'autorisation, un modèle de rapport et une méthode de preuve. Mon fil conducteur est la rigueur : un port ouvert n'est pas une vulnérabilité, et une absence de réponse ne prouve pas une absence d'hôte. Les tests sont prêts à être exécutés en laboratoire autorisé, avec une checklist de preuves. Je cherche un stage en réseaux ou cybersécurité pour confronter cette méthode au terrain. »

### Ce que la présentation ne doit pas faire

- Prétendre que les tests ont été exécutés s'ils ne l'ont pas été.
- Citer un nombre d'hôtes ou de ports non vérifié.
- Utiliser le mot « audit » pour un simple scan.
- Promettre une expertise que vous n'avez pas.
- Critiquer un outil sans nuance.

---

## 3. Dix questions techniques

### Question 1 — Quelle est la différence entre un scan SYN et un scan connect ?

**Réponse structurée :**

> Le scan SYN (`-sS`) envoie un segment SYN et lit la réponse sans terminer le handshake. Un SYN/ACK indique un port ouvert ; Nmap envoie alors un RST pour ne pas établir la connexion. Il demande des privilèges élevés et laisse moins de traces applicatives.
> Le scan connect (`-sT`) utilise l'appel système `connect()` : le handshake est complet, la connexion est visible dans les journaux applicatifs, et il fonctionne sans privilèges.
> Les deux restent détectables par un IDS réseau. Le choix dépend des droits, du besoin de discrétion et du contexte autorisé.

**À retenir :** semi-connexion vs connexion complète ; privilèges ; visibilité.

---

### Question 2 — Expliquez les états `open`, `closed` et `filtered`.

**Réponse structurée :**

> `open` : la cible a répondu favorablement, par exemple SYN/ACK. Un processus accepte des connexions.
> `closed` : la cible a répondu négativement, par exemple RST. L'hôte est joignable, mais rien n'écoute à cet instant.
> `filtered` : Nmap n'a pas obtenu de réponse concluante. Un équipement a pu bloquer le paquet, ou celui-ci a été perdu. L'état est indéterminé.
> Un port `filtered` n'est donc pas un port fermé. Et un port `open` n'est pas une vulnérabilité : c'est une surface d'exposition à qualifier.

**À retenir :** réponse reçue ≠ absence de réponse ; indétermination.

---

### Question 3 — Décrivez le handshake TCP et ce qu'il prouve.

**Réponse structurée :**

> Le client envoie SYN avec un numéro de séquence. Le serveur répond SYN/ACK s'il accepte. Le client confirme par ACK. La connexion est établie.
> Un handshake complet prouve qu'un service TCP accepte des connexions sur ce port, depuis ce point du réseau, à cet instant. Il ne prouve ni la version, ni la sécurité, ni la légitimité du service.
> Un RST peut signifier un port fermé ou un rejet par un pare-feu. Une absence de réponse est ambiguë : filtrage, perte ou lenteur.

**À retenir :** trois étapes ; preuve de joignabilité uniquement.

---

### Question 4 — Pourquoi `nmap -sn` peut-il manquer des hôtes actifs ?

**Réponse structurée :**

> `-sn` envoie plusieurs sondes : ARP sur le réseau local, ICMP echo, TCP SYN vers 443, TCP ACK vers 80, parfois ICMP timestamp. Si un hôte filtre tout cela, il ne répond pas.
> L'ARP ne fonctionne que sur le même domaine de diffusion. Un pare-feu peut bloquer l'ICMP et les sondes TCP. Un hôte en veille, derrière un NAT ou dans un VLAN non accessible peut être invisible.
> Enfin, l'absence de réponse n'est pas une preuve d'absence : elle doit être qualifiée d'indéterminée.

**À retenir :** sondes multiples, filtrage, portée locale, incertitude.

---

### Question 5 — Comment fonctionne `-sV` et quelles sont ses limites ?

**Réponse structurée :**

> `-sV` envoie des sondes applicatives, lit les bannières et compare les réponses à une base de signatures. Il propose un service, un produit, une version et un indice de confiance.
> Ses limites : une bannière peut être falsifiée, un reverse proxy peut masquer le service réel, un correctif peut être rétroporté sans changer le numéro de version, et un service authentifié peut ne rien révéler.
> Surtout, `-sV` ne détecte pas les vulnérabilités. Une version ancienne est un indice de risque, pas une preuve d'exploitabilité.

**À retenir :** signatures, indices, pas de conclusion CVE sans vérification.

---

### Question 6 — À quoi sert ARP et quelles sont ses limites ?

**Réponse structurée :**

> ARP associe une adresse IPv4 à une adresse MAC sur un même domaine de diffusion. Il est indispensable pour envoyer une trame Ethernet.
> Ses limites : il ne traverse pas un routeur, il n'a pas d'authentification, les réponses peuvent être usurpées, et le cache peut masquer une résolution.
> Une réponse ARP inattendue est un indice, pas une preuve d'attaque. Il faut corréler avec la table CAM du commutateur, le DHCP et l'inventaire.
> IPv6 utilise NDP, pas ARP : une analyse ARP ne couvre pas IPv6.

**À retenir :** local, sans authentification, indice ≠ preuve.

---

### Question 7 — Comment analyser DNS dans Wireshark, et quelles sont les limites ?

**Réponse structurée :**

> On filtre `dns`, puis on distingue les requêtes (`dns.flags.response == 0`) des réponses. On observe le nom (`dns.qry.name`), le type, le code de retour (`rcode`), le TTL et le délai.
> NXDOMAIN signifie que le nom n'existe pas ; SERVFAIL signifie que le résolveur n'a pas pu répondre ; REFUSED relève d'une politique.
> Les limites : le cache masque les résolutions, DoH/DoT chiffre le contenu, une réponse tronquée bascule en TCP, et le point de capture peut ne pas voir le résolveur.
> Une requête vers un domaine inconnu ne prouve pas une compromission.

**À retenir :** requête/réponse, codes, cache, chiffrement.

---

### Question 8 — Différence entre filtre de capture et filtre d'affichage ?

**Réponse structurée :**

> Le filtre de capture, en syntaxe BPF, s'applique avant l'enregistrement : les paquets non correspondants ne sont pas capturés. Il réduit le volume et l'exposition des données, mais ce qui est exclu est perdu.
> Le filtre d'affichage s'applique après la capture : il masque les paquets sans les supprimer. On peut donc changer d'angle sans recapturer.
> Exemple : `tshark -f "tcp port 443"` filtre à la capture ; `tcp.port == 443` filtre à l'affichage.

**À retenir :** avant/après ; perte vs masquage ; syntaxes différentes.

---

### Question 9 — Que signifient un RST et des retransmissions dans une capture TCP ?

**Réponse structurée :**

> Un RST réinitialise la connexion. Il peut venir d'un port fermé, d'un pare-feu, d'un équilibreur ou d'une application qui coupe la connexion. Ce n'est pas nécessairement une attaque.
> Une retransmission signifie que l'émetteur n'a pas reçu d'acquittement dans le délai prévu. Cela peut venir d'une perte, d'une congestion, d'un filtrage ou d'un problème applicatif.
> Il faut mesurer, comparer avec les journaux et capturer à un autre point avant de conclure.
> Attention aussi à l'offloading : des checksums marqués invalides peuvent être un artefact de la carte réseau, pas une corruption.

**À retenir :** plusieurs causes possibles ; corréler ; offloading.

---

### Question 10 — Comment validez-vous une observation avant de la mettre dans un rapport ?

**Réponse structurée :**

> Je pars de la preuve brute : sortie Nmap, capture `.pcapng`, horodatage, point de capture. Je vérifie qu'elle est complète et reproductible.
> Je formule ensuite un fait, puis une hypothèse, puis l'action qui permettrait de confirmer ou d'infirmer. Je croise plusieurs sources : inventaire, journaux, table ARP, configuration du pare-feu.
> Je qualifie la confiance et la criticité dans le contexte. Si la preuve manque, je marque « À réaliser » plutôt que de compléter par une supposition.
> Enfin, je fais relire par un pair et j'anonymise avant diffusion.

**À retenir :** fait → hypothèse → confirmation → criticité → recommandation.

---

## 4. Cinq questions difficiles

### Question difficile 1 — « Un port ouvert, c'est une vulnérabilité, non ? »

**Ce que le recruteur teste :** la capacité à ne pas confondre exposition et faille.

**Réponse structurée :**

> Non, pas automatiquement. Un port ouvert indique qu'un service accepte des connexions. C'est une surface d'exposition.
> La vulnérabilité dépend du service, de sa version, de sa configuration, de l'authentification, du chiffrement, de l'exposition réseau et des contrôles compensatoires. Un port 443 ouvert avec TLS à jour et une application patchée n'a pas le même risque qu'un service d'administration obsolète accessible à tous.
> Dans un rapport, j'écrirais : « Le port X est ouvert et expose un service Y version Z ; ce constat justifie une vérification de la politique de patch et de la configuration. » Je n'écrirais pas : « Le serveur est vulnérable. »

**Piège à éviter :** minimiser le risque à l'inverse. Un port ouvert reste un point à qualifier.

---

### Question difficile 2 — « Avec un scan limité, pouvez-vous prouver que le réseau est sûr ? »

**Ce que le recruteur teste :** la lucidité et la rigueur.

**Réponse structurée :**

> Non, et je ne le prétendrais pas. Un scan limité ne couvre qu'une partie des ports, des protocoles et des points de vue. Il ne voit ni l'IPv6, ni les services authentifiés, ni les vulnérabilités applicatives, ni les risques organisationnels.
> Ce que je peux dire : « Dans le périmètre testé, à cette date, depuis ce point du réseau, voici ce qui a été observé, avec ces preuves et ces limites. »
> La sécurité globale exige une approche complémentaire : revue de configuration, gestion des correctifs, analyse de vulnérabilités, tests d'intrusion, supervision et revue des processus.
> Un scan négatif est une information, pas une garantie.

**Piège à éviter :** répondre « oui, si le scan ne trouve rien ». C'est faux et dangereux.

---

### Question difficile 3 — « Vous découvrez un serveur non inventorié ou une faille critique pendant l'audit. Que faites-vous ? »

**Ce que le recruteur teste :** l'éthique, l'escalade et le sang-froid.

**Réponse structurée :**

> Je m'arrête au constat. Je ne tente aucune exploitation, aucune authentification et aucune modification.
> Je conserve la preuve brute, je note l'heure, la commande, la source et le point de capture, puis je préviens immédiatement le contact technique et le responsable de l'audit selon la procédure d'escalade.
> Si le périmètre ou l'autorisation ne couvre pas l'élément découvert, je ne l'analyse pas davantage : je le signale sans l'explorer.
> Je documente l'incident dans le rapport, avec le niveau de confiance et les limites. La priorité est la sécurité du système et la traçabilité, pas la démonstration.

**Piège à éviter :** « Je teste pour voir si c'est vraiment exploitable. » Cela sort du cadre.

---

### Question difficile 4 — « Vous n'avez pas d'expérience professionnelle. Pourquoi vous choisir ? »

**Ce que le recruteur teste :** la lucidité, la motivation et la capacité à transférer.

**Réponse structurée :**

> Je n'ai pas encore d'expérience en entreprise, et je ne vais pas l'inventer. Ce que j'apporte, c'est une méthode de travail déjà structurée : un projet documenté, des procédures reproductibles, une exigence de preuve et une conscience claire des limites.
> J'ai appris à distinguer un fait d'une hypothèse, à écrire pour un lecteur non technique et à cadrer un test par une autorisation. Ce sont des réflexes utiles dans un SOC ou une équipe réseau.
> Ce qui me manque, c'est le terrain : la variété des équipements, la pression des incidents, la connaissance de l'organisation. C'est précisément ce que je viens apprendre en stage, avec humilité et envie de contribuer.
> Je peux aussi montrer ce que je sais faire concrètement : expliquer un handshake, lire un `filtered`, ou rédiger une recommandation proportionnée.

**Piège à éviter :** lister des technologies sans preuve, ou prétendre avoir « de l'expérience » via des projets personnels.

---

### Question difficile 5 — « Le client veut un rapport rassurant et rapide. Comment réagissez-vous ? »

**Ce que le recruteur teste :** l'intégrité et la gestion de la pression.

**Réponse structurée :**

> Je comprends la pression, mais je ne peux pas modifier un résultat ni supprimer une limite pour rassurer. Je peux, en revanche, adapter la forme : une synthèse claire, des priorités, un langage non technique, et un plan d'action réaliste.
> Je distingue ce qui est confirmé, ce qui reste à vérifier et ce qui est hors périmètre. Si le délai ne permet pas de tout tester, je réduis le périmètre avec l'accord du client et je documente les tests non réalisés.
> La crédibilité d'un audit repose sur la traçabilité. Un rapport qui masque une incertitude expose l'organisation à une mauvaise décision.
> Je propose donc une restitution honnête, avec des recommandations proportionnées et des échéances négociées.

**Piège à éviter :** céder en promettant « un rapport propre » sans mention des limites.

---

## 5. Questions à poser au recruteur

### Sur la mission

- « Quels types d'infrastructures le stagiaire sera-t-il amené à observer ? »
- « Le stage inclut-il de l'analyse SOC, de la supervision ou de la réponse à incident ? »
- « Quels outils sont utilisés au quotidien : Nmap, Wireshark, Zeek, Suricata, SIEM ? »
- « Comment sont encadrés les tests réseau et les autorisations ? »

### Sur l'équipe et la montée en compétence

- « Comment se passe l'intégration d'un stagiaire dans l'équipe ? »
- « Existe-t-il un mentor ou un référent technique ? »
- « Quels sont les rituels d'équipe : revue de configurations, post-mortem, veille ? »
- « Quelles seraient les premières missions concrètes après un mois ? »

### Sur l'éthique et la sécurité

- « Comment l'entreprise cadre-t-elle les tests de sécurité et les autorisations ? »
- « Existe-t-il une politique de divulgation responsable ? »
- « Comment sont protégées les données collectées pendant les audits ? »

> Poser une question sur le cadre éthique montre une maturité appréciée dans les métiers de la sécurité.

---

## 6. Mise en situation et exercice blanc

### Exercice 1 — Expliquer un état `filtered` à un non-technicien

**Consigne :** 30 secondes, sans jargon.

**Exemple de réponse :**

> « Imaginez une porte dont on ne sait pas si elle est ouverte ou fermée, parce qu'un agent devant elle ne répond pas. On ne peut pas dire qu'elle est fermée : on peut seulement dire qu'on n'a pas pu la tester. En sécurité, on note cet état comme indéterminé et on prévoit une vérification complémentaire. »

### Exercice 2 — Justifier le choix des options

**Situation :** on vous demande pourquoi vous n'utilisez pas `-p-` et `-T5` pour aller plus vite.

**Réponse attendue :**

> `-p-` scanne 65 535 ports : charge plus élevée, durée plus longue, risque d'alerte et d'impact. `-T5` agressif peut provoquer des pertes et de faux `filtered`. Sur un lab sain, `-T4` peut suffire ; sur un réseau sensible, `-T3` est préférable.
> Le bon choix dépend de l'autorisation, de la criticité des cibles et de la question posée. La vitesse n'est pas un critère de qualité.

### Exercice 3 — Corriger une affirmation

**Affirmation :** « Le scan n'a trouvé aucun port ouvert, donc le serveur est sûr. »

**Correction attendue :**

> « Le scan des 1 000 ports TCP les plus courants n'a pas observé de port ouvert depuis [source], à [date]. Ce résultat ne couvre ni les autres ports, ni l'UDP, ni l'IPv6, ni les services filtrés. Il ne permet pas de conclure que le serveur est sûr. »

### Exercice 4 — Décrire une preuve

**Consigne :** citer les éléments minimaux d'une preuve défendable.

**Réponse attendue :**

> Date et heure UTC, opérateur, version d'outil, commande exacte, source, cible, point de capture, fichier brut, empreinte SHA-256, référence d'autorisation et limites.

---

## 7. Vocabulaire et formulations

### Termes à maîtriser

| Terme | Définition courte |
|-------|-------------------|
| Surface d'exposition | Ensemble des services accessibles depuis un point donné |
| Vecteur d'attaque | Moyen par lequel une attaque peut être menée |
| Handshake | Établissement d'une connexion TCP en trois étapes |
| RST | Drapeau de réinitialisation d'une connexion TCP |
| BPF | Syntaxe des filtres de capture |
| TTL | Durée de vie d'un paquet |
| CIDR | Notation d'un réseau, ex. `192.168.1.0/24` |
| NSE | Moteur de scripts de Nmap |
| SPAN | Miroir de port sur un commutateur |
| Faux positif | Détection qui signale à tort un problème |
| Faux négatif | Problème non détecté |
| Chaîne de possession | Traçabilité d'une preuve depuis sa collecte |

### Formulations sûres

| Au lieu de… | Dire… |
|-------------|-------|
| « Le serveur est vulnérable » | « Le service expose une version qui justifie une vérification de patch » |
| « Le réseau est sûr » | « Aucun port ouvert n'a été observé dans le périmètre testé à cette date » |
| « L'hôte est éteint » | « L'hôte n'a pas répondu aux sondes utilisées ; son état est indéterminé » |
| « C'est une attaque » | « Ce comportement est inhabituel et doit être corrélé » |
| « J'ai audité » (si non exécuté) | « J'ai conçu et documenté une méthode d'audit » |
| « J'ai trouvé une faille » (si non confirmé) | « J'ai identifié un point à vérifier » |

---

## 8. Checklist avant l'entretien

### Contenu

- [ ] Je peux expliquer les sept tests en 2 minutes
- [ ] Je connais la différence entre `open`, `closed` et `filtered`
- [ ] Je peux décrire le handshake TCP sans notes
- [ ] Je sais expliquer les limites du ping, d'ARP et de DNS
- [ ] Je peux distinguer filtre de capture et filtre d'affichage
- [ ] Je peux citer les éléments d'une preuve défendable
- [ ] Je connais les limites de `-sV`
- [ ] Je ne revendique aucun résultat non exécuté

### Présentation

- [ ] Pitch de 60 à 90 secondes chronométré
- [ ] Version courte de 30 secondes prête
- [ ] Deux questions à poser au recruteur préparées
- [ ] Exemple de correction d'une affirmation prêt
- [ ] Vocabulaire technique relu

### Logistique

- [ ] Dépôt GitHub accessible et propre
- [ ] Rapport lisible sans les preuves brutes
- [ ] Aucune donnée personnelle dans le dépôt
- [ ] Lien du projet prêt à partager
- [ ] Tenue et ponctualité vérifiées

---

## 9. Ce qu'il ne faut pas dire

| À éviter | Pourquoi | Alternative |
|----------|----------|-------------|
| « J'ai scanné mon réseau domestique » | Le cadre légal et l'autorisation ne sont pas démontrés | « J'ai documenté un protocole pour un laboratoire autorisé » |
| « J'ai trouvé des vulnérabilités » | Aucune preuve ni confirmation | « J'ai identifié des points à vérifier » |
| « Nmap permet de hacker » | Confusion entre audit et attaque | « Nmap permet de cartographier une exposition autorisée » |
| « Wireshark voit tout » | Faux : point de capture, chiffrement | « Wireshark voit le trafic qui atteint l'interface » |
| « Le pare-feu est inutile » | Jugement non étayé | « Le filtrage observé mérite une vérification de configuration » |
| « Je maîtrise tout » | Manque de lucidité | « Je sais faire ceci ; voici ce que je dois apprendre » |
| « C'est facile » | Minimise la complexité | « La difficulté principale est de qualifier sans surinterpréter » |

---

## 10. Clôture de l'entretien

### Message de fin — 20 secondes

> « Merci pour cet échange. Ce qui m'intéresse dans ce stage, c'est de passer d'une méthode documentée à des situations réelles, avec une équipe qui exige de la rigueur. Je suis disponible pour un exercice technique ou une mise en situation si vous le souhaitez. »

### Après l'entretien

- [ ] Envoyer un remerciement personnalisé sous 24 heures
- [ ] Rappeler un point précis de l'échange
- [ ] Joindre le lien du dépôt si demandé
- [ ] Noter les questions posées pour progresser
- [ ] Consigner les retours reçus

---

> **Rappel final :** la crédibilité se construit sur la précision, pas sur la surenchère. Dire « je ne sais pas, voici comment je le vérifierais » est souvent une meilleure réponse que d'inventer.
