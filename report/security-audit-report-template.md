# Rapport d'audit de sécurité réseau

> **MODÈLE — À COMPLÉTER APRÈS EXÉCUTION DES TESTS**
> Ce document est un gabarit. Il ne contient **aucun résultat réel**, **aucune vulnérabilité confirmée** et **aucune capture**. Toute section non renseignée doit conserver la mention **« À réaliser »**. Ne jamais publier ce rapport avec des résultats inventés, extrapolés ou non étayés.

---

## 1. Page de garde

| Champ | Valeur |
|-------|--------|
| Référence du rapport | `[AUDIT-AAAA-NNN]` |
| Titre | Audit de sécurité réseau — découverte, exposition et analyse de trafic |
| Client / entité | `[Nom de l'entité]` |
| Site / environnement | `[Site, bâtiment, VLAN, laboratoire]` |
| Type d'audit | Audit technique réseau autorisé (boîte grise / laboratoire) |
| Version | `[0.1 — brouillon]` |
| Date de rédaction | `[AAAA-MM-JJ]` |
| Période des tests | `[AAAA-MM-JJ] au [AAAA-MM-JJ]` |
| Auteur | ISSIFOU Ridwan — Cisco Networking Academy : Networking Basics ; IBM SkillsBuild : Cybersecurity Fundamentals, IT Fundamentals |
| Relecteur | `[Prénom NOM], [fonction]` |
| Destinataire | `[Nom, fonction]` |
| Classification | `[Interne / Confidentiel / Diffusion restreinte]` |
| Référence d'autorisation | `[AUTORISATION-AAAA-NNN]` |
| Statut | **Brouillon — résultats à réaliser** |

### Mentions

> Le présent rapport décrit une méthodologie et, le cas échéant, des observations réalisées dans un périmètre explicitement autorisé. Il ne constitue ni une attestation de conformité, ni une garantie d'absence de vulnérabilités, ni un test d'intrusion complet. Les conclusions sont indissociables du périmètre, de la date et des conditions de test décrits ci-après.

---

## 2. Synthèse exécutive

### 2.1 Objet

`[Décrire en 3 à 5 phrases l'objet de la mission, le périmètre et la question à laquelle le rapport répond.]`

### 2.2 Résultats principaux

| Élément | Statut | Commentaire |
|---------|--------|-------------|
| Tests Nmap réalisés | **À réaliser** | Voir section 6 |
| Captures Wireshark réalisées | **À réaliser** | Voir section 6 |
| Observations validées | **À réaliser** | Voir section 7 |
| Risques confirmés | **À réaliser** | Aucun risque ne doit être listé sans preuve |
| Recommandations prioritaires | **À réaliser** | Voir section 9 |

### 2.3 Message clé

`[Rédiger un message compréhensible par un décideur, sans jargon inutile, et sans affirmer de vulnérabilité non confirmée.]`

**Exemple de formulation prudente :**

> « Les tests ont été réalisés sur [périmètre] le [date]. Ils ont permis d'observer [n] hôtes répondant et [n] ports ouverts. Ces observations décrivent une exposition visible depuis [point de vue] à cette date ; elles ne permettent pas de conclure à la sécurité globale du réseau. [n] observations restent à confirmer et sont détaillées en section 7. »

### 2.4 Niveau de confiance

| Niveau | Signification | Utilisation |
|--------|---------------|-------------|
| Élevé | Fait reproductible, preuve brute conservée, plusieurs sources concordantes | Conclusion autorisée |
| Moyen | Fait observé une fois, preuve conservée, contexte partiel | Conclusion prudente, à confirmer |
| Faible | Indice ou hypothèse, preuve incomplète | **Ne pas présenter comme un résultat** |

---

## 3. Périmètre et objectifs

### 3.1 Périmètre inclus

| Élément | Détail | Référence d'autorisation |
|---------|--------|--------------------------|
| Réseau(x) | `[CIDR autorisé]` | `[Référence]` |
| Hôtes | `[Liste ou plage]` | `[Référence]` |
| Services | `[Ports / protocoles autorisés]` | `[Référence]` |
| Tests | Découverte, scan TCP, détection de services, capture ARP/DNS/TCP/ICMP | `[Référence]` |
| Fenêtre | `[Date et heure de début / fin, fuseau]` | `[Référence]` |
| Point de vue | `[Interne / externe / VLAN]` | `[Référence]` |
| Comptes utilisés | `[Aucun compte privilégié / compte de test]` | `[Référence]` |

### 3.2 Périmètre exclu

| Élément exclu | Raison |
|---------------|--------|
| `[Adresses hors CIDR]` | Hors autorisation |
| `[Équipements critiques]` | Risque opérationnel |
| `[Tests d'exploitation]` | Non demandés |
| `[Ingénierie sociale]` | Hors périmètre |
| `[Audit applicatif]` | Hors périmètre |
| `[IPv6]` | Non couvert par les tests |

### 3.3 Objectifs

1. Cartographier les hôtes répondant dans le périmètre autorisé.
2. Identifier les ports TCP ouverts parmi les 1 000 plus courants.
3. Qualifier les services exposés et leur version probable.
4. Observer et documenter les échanges ARP, DNS, TCP et ICMP.
5. Produire des recommandations proportionnées et traçables.
6. **Ne pas** exploiter, modifier ou interrompre les systèmes.

### 3.4 Critères de succès

- [ ] Tous les tests prévus ont été exécutés ou explicitement marqués « non réalisé ».
- [ ] Chaque résultat est associé à une preuve horodatée et hashée.
- [ ] Les faits, hypothèses et recommandations sont séparés.
- [ ] Les limites sont documentées.
- [ ] Le rapport a été relu par un pair.

---

## 4. Cadre contractuel et éthique

| Élément | Référence / statut |
|---------|--------------------|
| Autorisation écrite signée | **À réaliser** |
| Périmètre annexé à l'autorisation | **À réaliser** |
| Contacts techniques et escalade | **À réaliser** |
| Procédure d'arrêt d'urgence | **À réaliser** |
| Engagement de confidentialité | **À réaliser** |
| Assurance / responsabilité | **À réaliser** |
| Base légale du traitement des données | **À réaliser** |

### Règles appliquées

- Aucun test hors périmètre.
- Aucune exploitation de vulnérabilité.
- Aucune tentative d'authentification ou de brute force.
- Aucun script NSE intrusif.
- Aucune capture de données personnelles non nécessaires.
- Arrêt immédiat en cas d'incident ou de dépassement.

> En France, l'accès ou le maintien frauduleux dans un système de traitement automatisé de données est réprimé par les articles 323-1 à 323-8 du Code pénal. Une autorisation écrite ne dispense pas du respect des autres obligations légales et contractuelles.

---

## 5. Méthodologie

### 5.1 Approche

| Phase | Activité | Livrable |
|-------|----------|----------|
| Cadrage | Périmètre, autorisation, contacts, fenêtre | Document d'autorisation |
| Préparation | Vérification des outils, journal, sauvegarde | Journal de bord |
| Découverte | `nmap -sn` | Sortie Nmap + hash |
| Exposition | `nmap <IP>` | Sortie Nmap + hash |
| Services | `nmap -sV <IP>` | Sortie Nmap + hash |
| Trafic | Captures ARP, DNS, TCP, ICMP | `.pcapng` + hash |
| Analyse | Corrélation, qualification, limites | Registre d'observations |
| Restitution | Rapport, synthèse, recommandations | Ce document |

### 5.2 Outils et versions

| Outil | Version | Système | Remarque |
|-------|---------|---------|----------|
| Nmap | `[version exacte]` | `[OS]` | `nmap --version` |
| Wireshark | `[version exacte]` | `[OS]` | `wireshark --version` |
| `tshark` | `[version exacte]` | `[OS]` | Si utilisé |
| Système d'exploitation | `[version]` | — | — |
| Horloge | `[NTP / UTC]` | — | Synchronisation |

### 5.3 Tests prévus

| # | Test | Commande / filtre | Exécuté | Preuve |
|---|------|-------------------|---------|--------|
| 1 | Découverte d'hôtes | `nmap -sn 192.168.1.0/24` | **À réaliser** | **À réaliser** |
| 2 | Scan de ports | `nmap 192.168.1.10` | **À réaliser** | **À réaliser** |
| 3 | Détection de services | `nmap -sV 192.168.1.10` | **À réaliser** | **À réaliser** |
| 4 | ARP | filtre `arp` | **À réaliser** | **À réaliser** |
| 5 | DNS | filtre `dns` | **À réaliser** | **À réaliser** |
| 6 | TCP | filtre `tcp` | **À réaliser** | **À réaliser** |
| 7 | ICMP | filtre `icmp` | **À réaliser** | **À réaliser** |

> Les adresses ci-dessus sont des **exemples**. Elles doivent être remplacées par celles du périmètre autorisé dans la version finale.

### 5.4 Méthode de qualification d'une observation

1. **Constat** : que montre la preuve brute ?
2. **Vérification** : la preuve est-elle complète, reproductible, cohérente ?
3. **Hypothèse** : quelle explication propose-t-on ?
4. **Confirmation** : quel test ou quelle source permettrait de trancher ?
5. **Criticité** : quel impact et quelle probabilité, dans quel contexte ?
6. **Recommandation** : quelle action proportionnée ?

Une observation sans preuve reste au statut **« À réaliser »** et ne peut pas être notée comme risque confirmé.

---

## 6. Résultats des tests

> **Aucun résultat n'est prérempli.** Remplir uniquement après exécution réelle. Conserver les sorties brutes dans `evidence/` et les référencer.

### 6.1 Test 1 — Découverte d'hôtes

| Champ | Valeur |
|-------|--------|
| Statut | **À réaliser** |
| Date / heure UTC | **À réaliser** |
| Commande | `nmap -sn [CIDR autorisé]` |
| Source | **À réaliser** |
| Preuve | **À réaliser** |
| Empreinte SHA-256 | **À réaliser** |
| Hôtes ayant répondu | **À réaliser** |
| Hôtes attendus silencieux | **À réaliser** |
| Limites | ICMP filtrable, portée ARP locale, instantané |

**Faits observés :** À réaliser
**Hypothèses :** À réaliser
**Actions :** À réaliser

### 6.2 Test 2 — Scan de ports

| Champ | Valeur |
|-------|--------|
| Statut | **À réaliser** |
| Date / heure UTC | **À réaliser** |
| Commande | `nmap [IP autorisée]` |
| Type de scan effectif | **À réaliser** (`-sS` ou `-sT`) |
| Preuve | **À réaliser** |
| Empreinte SHA-256 | **À réaliser** |
| Ports ouverts | **À réaliser** |
| Ports fermés | **À réaliser** |
| Ports filtrés | **À réaliser** |
| Limites | 1 000 ports TCP, point de vue unique, instantané |

**Faits observés :** À réaliser
**Hypothèses :** À réaliser
**Actions :** À réaliser

### 6.3 Test 3 — Détection de services

| Champ | Valeur |
|-------|--------|
| Statut | **À réaliser** |
| Date / heure UTC | **À réaliser** |
| Commande | `nmap -sV [IP autorisée]` |
| Intensité | **À réaliser** |
| Preuve | **À réaliser** |
| Empreinte SHA-256 | **À réaliser** |
| Services identifiés | **À réaliser** |
| Services non identifiés | **À réaliser** |
| Limites | Bannière falsifiable, proxy, correctifs rétroportés |

**Faits observés :** À réaliser
**Hypothèses :** À réaliser
**Actions :** À réaliser

### 6.4 Test 4 — ARP

| Champ | Valeur |
|-------|--------|
| Statut | **À réaliser** |
| Date / heure UTC | **À réaliser** |
| Filtre d'affichage | `arp` |
| Point de capture | **À réaliser** |
| Preuve `.pcapng` | **À réaliser** |
| Empreinte SHA-256 | **À réaliser** |
| Requêtes / réponses | **À réaliser** |
| Couples IP/MAC | **À réaliser** |
| Anomalies apparentes | **À réaliser** |
| Limites | Segment local, cache, absence d'authentification |

**Faits observés :** À réaliser
**Hypothèses :** À réaliser
**Actions :** À réaliser

### 6.5 Test 5 — DNS

| Champ | Valeur |
|-------|--------|
| Statut | **À réaliser** |
| Date / heure UTC | **À réaliser** |
| Filtre d'affichage | `dns` |
| Résolveur | **À réaliser** |
| Preuve `.pcapng` | **À réaliser** |
| Empreinte SHA-256 | **À réaliser** |
| Requêtes / réponses | **À réaliser** |
| Codes de réponse | **À réaliser** |
| Limites | Cache, DoH/DoT, UDP tronqué |

**Faits observés :** À réaliser
**Hypothèses :** À réaliser
**Actions :** À réaliser

### 6.6 Test 6 — TCP

| Champ | Valeur |
|-------|--------|
| Statut | **À réaliser** |
| Date / heure UTC | **À réaliser** |
| Filtre d'affichage | `tcp` |
| Point de capture | **À réaliser** |
| Preuve `.pcapng` | **À réaliser** |
| Empreinte SHA-256 | **À réaliser** |
| Handshake observé | **À réaliser** |
| RST / retransmissions | **À réaliser** |
| Flux analysés | **À réaliser** |
| Limites | Point de capture, chiffrement, offloading |

**Faits observés :** À réaliser
**Hypothèses :** À réaliser
**Actions :** À réaliser

### 6.7 Test 7 — ICMP

| Champ | Valeur |
|-------|--------|
| Statut | **À réaliser** |
| Date / heure UTC | **À réaliser** |
| Filtre d'affichage | `icmp` |
| Point de capture | **À réaliser** |
| Preuve `.pcapng` | **À réaliser** |
| Empreinte SHA-256 | **À réaliser** |
| Requêtes / réponses | **À réaliser** |
| Perte / latence | **À réaliser** |
| Limites | ICMP filtrable, rate limiting, absence de réponse non concluante |

**Faits observés :** À réaliser
**Hypothèses :** À réaliser
**Actions :** À réaliser

---

## 7. Registre des observations

> Une ligne = une observation atomique, rattachée à **une** preuve. Ne pas regrouper plusieurs constats dans une seule ligne. Ne pas inscrire de vulnérabilité sans confirmation.

| ID | Date (UTC) | Test | Actif | Fait observé | Preuve | Confiance | Hypothèse | Criticité | Recommandation | Statut |
|----|------------|------|-------|--------------|--------|-----------|-----------|-----------|----------------|--------|
| OBS-01 | **À réaliser** | T1–T7 | **À réaliser** | **À réaliser** | **À réaliser** | Faible / Moyen / Élevé | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** |
| OBS-02 | **À réaliser** | T1–T7 | **À réaliser** | **À réaliser** | **À réaliser** | Faible / Moyen / Élevé | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** |
| OBS-03 | **À réaliser** | T1–T7 | **À réaliser** | **À réaliser** | **À réaliser** | Faible / Moyen / Élevé | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** |

### 7.1 Exemple de formulation — observation factuelle

> **OBS-EX-01 — Exemple pédagogique, non issu d'une observation réelle.**
> **Fait :** « Le port 22/TCP de 192.168.1.10 (exemple) est ouvert et Nmap identifie un service SSH avec un indice de confiance moyen. »
> **Preuve :** « `evidence/nmap/test3_services.nmap`, empreinte SHA-256 [valeur]. »
> **Hypothèse :** « Un service d'administration SSH est exposé au segment testé. »
> **Action :** « Vérifier la politique d'accès, l'authentification et la version auprès de l'administrateur. »
> **Ce qui n'est pas affirmé :** « Aucune vulnérabilité n'est confirmée. »

### 7.2 Règles de rédaction d'une observation

| À faire | À éviter |
|---------|----------|
| Décrire ce qui est observé, avec date et preuve | Affirmer une intention ou une attaque |
| Indiquer la confiance et les limites | Extrapoler à tout le réseau |
| Relier à un test et à un actif | Mélanger plusieurs constats |
| Proposer une action de confirmation | Conclure « vulnérable » sans analyse |
| Conserver la mention « À réaliser » si la preuve manque | Remplir par une supposition |

---

## 8. Matrice de risque

### 8.1 Échelle de probabilité

| Niveau | Définition |
|--------|------------|
| 1 — Très faible | Exploitation très improbable dans le contexte |
| 2 — Faible | Nécessite des conditions particulières |
| 3 — Moyenne | Exploitation possible avec des compétences courantes |
| 4 — Élevée | Exploitation probable, vecteur accessible |

### 8.2 Échelle d'impact

| Niveau | Définition |
|--------|------------|
| 1 — Mineur | Gêne limitée, aucune donnée sensible |
| 2 — Modéré | Atteinte limitée à un service non critique |
| 3 — Majeur | Atteinte à des données sensibles ou à un service important |
| 4 — Critique | Atteinte à la sécurité, à la conformité ou à la continuité |

### 8.3 Matrice

| Probabilité ↓ / Impact → | 1 Mineur | 2 Modéré | 3 Majeur | 4 Critique |
|--------------------------|----------|----------|----------|------------|
| 4 Élevée | Moyen | Élevé | Critique | Critique |
| 3 Moyenne | Faible | Moyen | Élevé | Critique |
| 2 Faible | Faible | Faible | Moyen | Élevé |
| 1 Très faible | Faible | Faible | Moyen | Moyen |

### 8.4 Règles d'usage

- La criticité est **contextuelle** : un port ouvert sur un lab isolé n'a pas la même valeur que sur un serveur exposé.
- Une criticité ne peut être attribuée qu'à une observation **confirmée** et documentée.
- Une hypothèse peut être mentionnée, mais pas notée comme risque confirmé.
- La matrice ne remplace pas une analyse métier ni une validation par le responsable.

---

## 9. Recommandations

> Ne renseigner que des recommandations découlant d'observations réelles. Les lignes ci-dessous sont des **modèles** à supprimer si elles ne sont pas justifiées.

| ID | Recommandation | Observation liée | Priorité | Effort | Responsable | Échéance | Statut |
|----|----------------|------------------|----------|--------|-------------|----------|--------|
| REC-01 | `[Action concrète et vérifiable]` | OBS-01 | P1 / P2 / P3 | Faible / Moyen / Élevé | **À réaliser** | **À réaliser** | **À réaliser** |
| REC-02 | `[Action concrète et vérifiable]` | OBS-02 | P1 / P2 / P3 | Faible / Moyen / Élevé | **À réaliser** | **À réaliser** | **À réaliser** |

### 9.1 Priorisation

| Priorité | Délai indicatif | Critère |
|----------|-----------------|---------|
| P1 — Immédiate | 0 à 7 jours | Risque critique ou exposition directe confirmée |
| P2 — Court terme | 1 à 4 semaines | Risque élevé ou contrôle essentiel absent |
| P3 — Planifiée | 1 à 6 mois | Amélioration de posture, durcissement |

### 9.2 Familles de recommandations possibles

À n'utiliser que si une observation les justifie :

- restreindre l'accès aux services d'administration par ACL ou bastion ;
- désactiver les services non nécessaires ;
- maintenir les services à jour selon la politique de l'éditeur ;
- renforcer l'authentification et le chiffrement ;
- segmenter les flux ;
- journaliser et superviser les accès ;
- documenter les flux dans l'inventaire ;
- revoir la politique de filtrage.

### 9.3 Ce qu'une recommandation ne doit pas être

- « Fermer tous les ports » sans analyse d'impact ;
- « Installer un antivirus » sans lien avec l'observation ;
- « Le réseau est sûr » sans preuve ;
- une recommandation copiée d'un modèle sans adaptation au contexte.

---

## 10. Limites de l'audit

| Limite | Conséquence sur les conclusions |
|--------|---------------------------------|
| Périmètre restreint | Les résultats ne concernent que les adresses et ports testés |
| Point de vue unique | L'exposition peut différer depuis un autre segment |
| Instantané | L'état peut changer après le test |
| Nmap limité aux 1 000 ports TCP | Les ports hauts et l'UDP ne sont pas couverts |
| Détection de version indicative | Bannières falsifiables, proxys, correctifs rétroportés |
| Capture sur une interface | Le trafic hors interface n'est pas visible |
| Chiffrement | Le contenu TLS/DoH/DoT reste illisible |
| Pas d'exploitation | Aucune vulnérabilité n'est confirmée par exploitation |
| Pas d'audit applicatif | La logique métier et le code ne sont pas couverts |
| IPv6 non couvert | L'exposition IPv6 n'est pas évaluée |
| Facteurs humains et organisationnels | Non évalués |

> **Conclusion type :** « Les tests réalisés décrivent une exposition observée dans un périmètre et à une date donnés. Ils ne permettent pas de conclure que le réseau est sûr, ni qu'il est vulnérable dans son ensemble. »

---

## 11. Conclusion

### 11.1 Rappel des objectifs

`[Rappeler les objectifs et le périmètre.]`

### 11.2 Réponse synthétique

`[Répondre aux questions posées, en distinguant ce qui est vérifié, ce qui reste à réaliser et ce qui est hors périmètre.]`

### 11.3 Prochaines étapes

| Étape | Responsable | Échéance | Statut |
|-------|-------------|----------|--------|
| Compléter les tests manquants | **À réaliser** | **À réaliser** | **À réaliser** |
| Confirmer les observations | **À réaliser** | **À réaliser** | **À réaliser** |
| Mettre en œuvre les recommandations P1 | **À réaliser** | **À réaliser** | **À réaliser** |
| Rejouer les tests de contrôle | **À réaliser** | **À réaliser** | **À réaliser** |
| Archiver les preuves | **À réaliser** | **À réaliser** | **À réaliser** |

---

## 12. Annexes

### Annexe A — Journal des commandes

| Date (UTC) | Opérateur | Commande | Source | Cible | Sortie | Statut |
|------------|-----------|----------|--------|-------|--------|--------|
| **À réaliser** | **À réaliser** | `nmap -sn [CIDR]` | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** |
| **À réaliser** | **À réaliser** | `nmap [IP]` | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** |
| **À réaliser** | **À réaliser** | `nmap -sV [IP]` | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** |
| **À réaliser** | **À réaliser** | Filtre `arp` | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** |
| **À réaliser** | **À réaliser** | Filtre `dns` | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** |
| **À réaliser** | **À réaliser** | Filtre `tcp` | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** |
| **À réaliser** | **À réaliser** | Filtre `icmp` | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** |

### Annexe B — Inventaire des preuves

| ID preuve | Type | Fichier | Date | Taille | SHA-256 | Statut |
|-----------|------|---------|------|--------|---------|--------|
| EV-01 | Sortie Nmap | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** |
| EV-02 | Sortie Nmap | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** |
| EV-03 | Sortie Nmap | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** |
| EV-04 | Capture ARP | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** |
| EV-05 | Capture DNS | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** |
| EV-06 | Capture TCP | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** |
| EV-07 | Capture ICMP | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** | **À réaliser** |

### Annexe C — Glossaire

| Terme | Définition |
|-------|------------|
| ARP | Protocole de résolution d'adresse IPv4 vers MAC sur un réseau local |
| BPF | Filtre de capture utilisé par `libpcap`/`dumpcap` |
| CVE | Identifiant public d'une vulnérabilité |
| DNS | Système de noms de domaine |
| DoH / DoT | DNS over HTTPS / DNS over TLS |
| ICMP | Protocole de messages de contrôle et d'erreur |
| NSE | Nmap Scripting Engine |
| RST | Drapeau TCP de réinitialisation |
| SHA-256 | Fonction de hachage utilisée pour l'intégrité |
| SPAN | Miroir de port sur un commutateur |
| SYN / SYN-ACK / ACK | Drapeaux du handshake TCP |
| TTL | Durée de vie d'un paquet |
| VLAN | Réseau local virtuel |

### Annexe D — Références

- Documentation Nmap : <https://nmap.org/book/man.html>
- Documentation Wireshark : <https://www.wireshark.org/docs/>
- NVD : <https://nvd.nist.gov/>
- ANSSI — recommandations réseaux : <https://cyber.gouv.fr/>
- Code pénal, articles 323-1 à 323-8.

---

## 13. Validation

| Rôle | Nom | Date | Signature / visa |
|------|-----|------|------------------|
| Auteur | **À réaliser** | **À réaliser** | **À réaliser** |
| Relecteur technique | **À réaliser** | **À réaliser** | **À réaliser** |
| Responsable du périmètre | **À réaliser** | **À réaliser** | **À réaliser** |
| Destinataire | **À réaliser** | **À réaliser** | **À réaliser** |

### Déclaration de l'auteur

> **À ne signer qu'après exécution réelle des tests et vérification des preuves.**
>
> Je certifie que les résultats présentés dans ce rapport correspondent à des observations réellement effectuées dans le périmètre autorisé, que les preuves associées sont conservées et que les limites sont documentées. Les éléments non réalisés sont explicitement marqués **« À réaliser »**. Aucune vulnérabilité n'est affirmée sans confirmation.

**Nom :** ISSIFOU Ridwan
**Date :** `[AAAA-MM-JJ]`
**Signature :** `[Signature]`

---

## 14. Checklist de finalisation

- [ ] Toutes les mentions « À réaliser » ont été soit complétées par une preuve, soit conservées
- [ ] Chaque observation possède un identifiant unique et une preuve
- [ ] Les empreintes SHA-256 sont reportées
- [ ] Les adresses d'exemple ont été remplacées par celles du périmètre
- [ ] Les données personnelles ont été anonymisées
- [ ] Les faits, hypothèses et recommandations sont distincts
- [ ] Aucune vulnérabilité n'est affirmée sans confirmation
- [ ] Les limites sont explicites
- [ ] Les recommandations sont proportionnées et attribuées
- [ ] Le rapport a été relu par un pair
- [ ] La classification et la diffusion sont définies
- [ ] La version finale est archivée avec les preuves
