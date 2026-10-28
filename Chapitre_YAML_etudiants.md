# YAML : écrire, lire et valider des fichiers de configuration

**Faculté des Sciences et Techniques de Settat, Université Hassan Ier**
**Module : Cloud Computing et DevOps**
**Public : étudiants de Licence ou de Master en informatique (niveau débutant)**

---

## Table des matières

1. [Partie 1. Introduction à YAML](#partie-1-introduction-à-yaml)
2. [Partie 2. Structure et syntaxe fondamentales](#partie-2-structure-et-syntaxe-fondamentales)
3. [Partie 3. Les structures de données en YAML](#partie-3-les-structures-de-données-en-yaml)
4. [Partie 4. Comprendre l’indentation et les erreurs](#partie-4-comprendre-lindentation-et-les-erreurs)
5. [Partie 5. YAML et JSON](#partie-5-yaml-et-json)
6. [Partie 6. Création et organisation des fichiers YAML](#partie-6-création-et-organisation-des-fichiers-yaml)
7. [Partie 7. Validation et débogage](#partie-7-validation-et-débogage)
8. [Partie 8. Application complète avec Docker Compose](#partie-8-application-complète-avec-docker-compose)
9. [Partie 9. Introduction à YAML dans Kubernetes](#partie-9-introduction-à-yaml-dans-kubernetes)
10. [Partie 10. Bonnes pratiques](#partie-10-bonnes-pratiques)
11. [Partie 11. Exercices et travaux pratiques](#partie-11-exercices-et-travaux-pratiques)
12. [Partie 12. Synthèse et évaluation](#partie-12-synthèse-et-évaluation)


---

## Objectifs d’apprentissage

À la fin de ce chapitre, vous serez capable de :

- définir YAML et situer ce format par rapport à HTML, JSON et aux langages de programmation ;
- lire et écrire des fichiers YAML (scalaires, listes, mappings, chaînes multilignes, ancres) ;
- identifier et corriger les erreurs d’indentation, de syntaxe et de typage ;
- distinguer un fichier **syntaxiquement valide**, un fichier **conforme au schéma d’un outil** et une configuration **fonctionnelle** ;
- valider un fichier avec des outils adaptés et le lire depuis un programme Python ;
- écrire un fichier `compose.yaml` (Nginx + PostgreSQL) et un manifeste Kubernetes simple ;
- appliquer les bonnes pratiques de rédaction et de sécurité.

## Conventions utilisées dans ce document

| Étiquette | Signification |
|---|---|
| **[Exécutable]** | L’exemple peut être testé tel quel (sous réserve des prérequis indiqués). |
| **[Illustratif]** | L’exemple sert à comprendre une idée ; il n’est pas destiné à être exécuté tel quel. |
| **[Incorrect]** | L’exemple contient volontairement une erreur. |

> **Remarque importante.** YAML est un **format** : il décrit des données. Ce sont des **outils** (Docker Compose, Kubernetes, Ansible, GitHub Actions…) qui lisent ces données et leur donnent un sens. Un même fichier YAML peut donc être valide pour YAML et invalide pour l’outil qui le consomme. Cette distinction revient tout au long du chapitre.

### Outils nécessaires pour les parties pratiques

| Outil | Utilité | Parties concernées |
|---|---|---|
| Un éditeur de texte (Visual Studio Code recommandé) | Écrire les fichiers | Toutes |
| Python 3 avec `pyyaml` et `yamllint` | Lire et valider du YAML | 6, 7, 11 |
| Docker avec Docker Compose v2 (`docker compose`) | Parties pratiques Compose | 8, 11 |
| `kubectl` et un cluster local (minikube, kind, Docker Desktop…) | Partie Kubernetes (facultative) | 9, 11 |

---

## Partie 1. Introduction à YAML

### 1.1 Définition et signification du nom

**YAML** est un **format de sérialisation de données** lisible par l’humain. Sérialiser signifie représenter des données (nombres, textes, listes, objets…) sous forme de texte, afin de les stocker dans un fichier ou de les échanger entre programmes.

Le nom est un **acronyme récursif** (il se contient lui-même) :

> **YAML Ain’t Markup Language**, soit « YAML n’est pas un langage de balisage ».

À l’origine, YAML signifiait *Yet Another Markup Language* (« encore un langage de balisage »). Le nom a été changé pour insister sur le fait que YAML sert à représenter des **données** et non à mettre en forme des documents.

Quelques repères sur les versions de la spécification :

| Version | Remarque |
|---|---|
| YAML 1.1 (2005) | Très répandue historiquement. Reconnaît `yes`, `no`, `on`, `off` comme booléens. |
| YAML 1.2 (2009, révision 1.2.2 en 2021) | Version actuelle de la spécification. Booléens limités à `true` et `false` dans le schéma de base. |

> **Attention.** Beaucoup d’outils et de bibliothèques n’implémentent pas exactement la même version ni exactement les mêmes fonctionnalités. Par exemple, la bibliothèque Python PyYAML suit en grande partie YAML 1.1. Nous signalerons ces différences au fil du chapitre.

### 1.2 Format de données, langage de balisage, langage de programmation

Ces trois notions sont souvent confondues. Voici comment les distinguer.

| Critère | Format de données (YAML, JSON) | Langage de balisage (HTML, XML, Markdown) | Langage de programmation (Python, C, Java) |
|---|---|---|---|
| Rôle principal | Décrire des **données** | Décrire la **structure et la présentation** d’un document | Décrire des **traitements** (algorithmes) |
| Contient des instructions (boucles, conditions) ? | Non | Non (HTML seul) | Oui |
| Est exécuté ? | Non, il est **lu** par un programme | Non, il est **interprété** par un navigateur ou un outil de rendu | Oui, par un interpréteur ou après compilation |
| Exemple de contenu | `port: 8080` | `<h1>Titre</h1>` | `for i in range(3): print(i)` |

**YAML et HTML.** HTML utilise des balises (`<p>…</p>`) pour indiquer à un navigateur comment afficher un document. YAML n’affiche rien : il décrit des données (« le port est 8080 », « la liste des serveurs est… »). Il n’y a pas de balises en YAML, la structure est exprimée par l’**indentation** (le décalage des lignes vers la droite).

**YAML et langages de programmation.** Un fichier YAML ne contient ni boucle, ni condition, ni fonction. Il dit **quoi** configurer, pas **comment** le calculer. Certains outils ajoutent de la logique autour de YAML (expressions `${{ ... }}` dans GitHub Actions, modèles Jinja2 dans Ansible, modèles Helm pour Kubernetes), mais cette logique appartient à l’outil, pas à YAML.

### 1.3 Objectifs et avantages

YAML a été conçu pour être :

- **lisible** par un humain, sans outil particulier ;
- **facile à écrire** à la main (peu de symboles, pas d’accolades obligatoires) ;
- **adapté aux fichiers de configuration** (commentaires autorisés) ;
- **expressif** : listes, objets imbriqués, textes sur plusieurs lignes, réutilisation de blocs.

### 1.4 Principaux domaines d’utilisation

| Outil / domaine | Rôle de YAML | Nom de fichier courant |
|---|---|---|
| **Docker Compose** | Décrire des applications multi-conteneurs | `compose.yaml` |
| **Kubernetes** | Décrire les ressources du cluster (manifestes) | `deployment.yaml`, `service.yaml` |
| **Ansible** | Décrire des automatisations (playbooks) | `playbook.yml` |
| **GitHub Actions** | Décrire des workflows CI/CD | `.github/workflows/ci.yml` |
| **GitLab CI** | Décrire des pipelines CI/CD | `.gitlab-ci.yml` |
| **OpenAPI** | Décrire des API REST | `openapi.yaml` |

Un **pipeline CI/CD** est une suite d’étapes automatisées (compiler, tester, déployer) déclenchée à chaque modification du code.

**[Illustratif]** Un extrait de workflow GitHub Actions :

```yaml
name: CI
on:
  push:
    branches: [main]
jobs:
  tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Lancer les tests
        run: python -m pytest
```

Lecture : le workflow s’appelle `CI` ; il se déclenche à chaque `push` sur la branche `main` ; il contient un travail (`job`) nommé `tests`, exécuté sur une machine Ubuntu, avec deux étapes : récupérer le code, puis lancer les tests.

**[Illustratif]** Un extrait de playbook Ansible :

```yaml
- name: Installer Nginx
  hosts: webservers
  become: true
  tasks:
    - name: Installer le paquet nginx
      ansible.builtin.apt:
        name: nginx
        state: present
```

Lecture : sur le groupe de machines `webservers`, avec les droits administrateur (`become: true`), on s’assure que le paquet `nginx` est installé.

### 1.5 Pourquoi YAML est utile : un exemple concret

Imaginons que l’on veuille décrire un serveur. En texte libre :

```text
Le serveur web-01 a 2 processeurs, 4 Go de RAM, il est actif,
et il écoute sur les ports 80 et 443.
```

Un programme ne peut pas lire cette phrase de façon fiable. En YAML, la même information devient **structurée**, donc exploitable par un programme **et** lisible par un humain :

```yaml
serveur:
  nom: web-01
  cpu: 2
  ram_go: 4
  actif: true
  ports:
    - 80
    - 443
```

Avantages concrets : on peut versionner ce fichier avec Git, le relire six mois plus tard, le comparer avec `git diff`, et le faire lire par un outil qui déploiera automatiquement ce serveur. C’est le principe de l’**infrastructure as code** : l’infrastructure est décrite par des fichiers plutôt que configurée à la main.

### 1.6 Avantages et limites

| Avantages | Limites |
|---|---|
| Très lisible | L’indentation est **significative** : une erreur d’espace change le sens ou casse le fichier |
| Commentaires autorisés (`#`) | Certaines valeurs sont interprétées de façon inattendue (`NO`, `1.10`, `0755`…) |
| Peu de symboles | Spécification longue et complexe ; fonctionnalités rarement utilisées |
| Structures riches (listes, objets, ancres, multilignes) | Comportement variable selon le parseur et la version de YAML |
| Standard de fait dans l’écosystème DevOps | Pas de schéma intégré : la validité fonctionnelle dépend de l’outil |
| Facile à versionner avec Git | Risque de sécurité si on charge du YAML non fiable avec une fonction non sécurisée (voir partie 6) |

### Points clés de la partie 1

- YAML est un format de données, pas un langage de programmation ni un langage de balisage de documents.
- Il structure les données par l’indentation.
- Il est omniprésent en DevOps : Compose, Kubernetes, Ansible, CI/CD.
- Sa simplicité apparente cache des pièges (indentation, typage implicite).

---

## Partie 2. Structure et syntaxe fondamentales

Pour chaque notion, nous suivrons toujours le même plan : **(1)** explication, **(2)** exemple minimal, **(3)** exemple réaliste, **(4)** explication ligne par ligne, **(5)** erreur fréquente et correction.

### 2.1 Structure générale d’un document YAML

**1. Explication.** Un document YAML est un fichier texte (de préférence encodé en **UTF-8**) qui représente **une seule valeur racine**. Cette racine est le plus souvent un **mapping** (ensemble de paires clé-valeur) ou une **liste**. Les valeurs peuvent elles-mêmes contenir d’autres mappings ou listes : c’est ce qui forme une structure arborescente.

**2. Exemple minimal** **[Exécutable]**

```yaml
nom: serveur-web
```

**3. Exemple réaliste** **[Exécutable]**

```yaml
# Description d'une machine virtuelle
machine:
  nom: vm-web-01
  cpu: 2
  ram_go: 4
  systeme: Ubuntu 24.04
```

**4. Explication ligne par ligne**

- Ligne 1 : un commentaire, ignoré par le parseur.
- Ligne 2 : la clé `machine` n’a pas de valeur sur sa ligne ; sa valeur est le bloc indenté qui suit.
- Lignes 3 à 6 : quatre paires clé-valeur appartenant à `machine`, car elles sont décalées de deux espaces par rapport à `machine`.
- `Ubuntu 24.04` est une chaîne (elle contient des lettres), il n’y a pas besoin de guillemets.

**5. Erreur fréquente** : mélanger un mapping et une liste au même niveau.

```yaml
nom: vm-web-01
- cpu
```

**[Incorrect]** La racine ne peut pas être à la fois un mapping (`nom: …`) et une liste (`- …`). Le parseur signale une erreur.

Correction : rattacher la liste à une clé.

```yaml
nom: vm-web-01
caracteristiques:
  - cpu
```

### 2.2 Les paires clé-valeur

**1. Explication.** Une paire clé-valeur associe un **nom** (la clé) à une **valeur**. Syntaxe : `clé: valeur`, avec **un espace obligatoire après les deux-points**. Les clés d’un même mapping doivent être **uniques**.

**2. Exemple minimal** **[Exécutable]**

```yaml
hote: localhost
```

**3. Exemple réaliste** **[Exécutable]**

```yaml
serveur_base_donnees:
  hote: db.exemple.local
  port: 5432
  utilisateur: app_user
  ssl: true
```

**4. Explication ligne par ligne**

- `serveur_base_donnees:` : clé dont la valeur est un mapping (le bloc indenté).
- `hote: db.exemple.local` : valeur de type chaîne.
- `port: 5432` : valeur de type entier (aucun guillemet).
- `utilisateur: app_user` : chaîne ; les underscores sont autorisés.
- `ssl: true` : booléen.

**5. Erreur fréquente** : oublier l’espace après les deux-points.

```yaml
hote: localhost
port:5432
```

**[Incorrect]** `port:5432` n’est pas lu comme une paire clé-valeur (les deux-points doivent être suivis d’un espace). Selon le contexte, le parseur signale une erreur ou produit un résultat inattendu.

Correction :

```yaml
hote: localhost
port: 5432
```

### 2.3 Les chaînes de caractères, guillemets simples et doubles

**1. Explication.** Une **chaîne de caractères** est un texte. En YAML, on peut l’écrire :

| Forme | Exemple | Particularités |
|---|---|---|
| Sans guillemets (*plain*) | `nom: Alice` | La plus courante ; restrictions sur certains caractères |
| Guillemets simples | `nom: 'Alice'` | Texte pris **littéralement** ; une apostrophe s’écrit en la doublant : `'l''école'` |
| Guillemets doubles | `nom: "Alice"` | Les **séquences d’échappement** sont interprétées : `\n` (saut de ligne), `\t` (tabulation), `\"`, `\\`, `\u00e9` |

Les guillemets deviennent **nécessaires** lorsque la valeur :

- contient `: ` (deux-points suivi d’un espace) ou ` #` (espace suivi de dièse) ;
- commence par un caractère spécial : `@`, `` ` ``, `*`, `&`, `!`, `%`, `{`, `[`, `|`, `>`, `-` suivi d’un espace, `?` ;
- ressemble à un autre type (`"true"`, `"123"`, `"null"`) alors qu’on veut une chaîne.

**2. Exemple minimal** **[Exécutable]**

```yaml
nom: Alice
ville: 'Settat'
message: "Bonjour\nle monde"
```

**3. Exemple réaliste** **[Exécutable]**

```yaml
application:
  titre: "Portail étudiant : FST Settat"
  chemin_windows: 'C:\Users\etudiant\projet'
  phrase: 'l''ordinateur est allumé'
  separateur: "colonne1\tcolonne2"
  version: "1.10"
```

**4. Explication ligne par ligne**

- `titre` : les guillemets sont nécessaires car la valeur contient `: `.
- `chemin_windows` : guillemets **simples** pour que les antislashs (`\`) restent littéraux.
- `phrase` : l’apostrophe est doublée dans une chaîne à guillemets simples.
- `separateur` : en guillemets doubles, `\t` devient une vraie tabulation.
- `version` : les guillemets forcent le type chaîne (sans eux, `1.10` serait lu comme le nombre `1.1`).

**5. Erreur fréquente** : deux-points dans une chaîne non protégée.

```yaml
titre: Portail étudiant : FST Settat
```

**[Incorrect]** Le deuxième `: ` est interprété comme un nouveau séparateur clé-valeur. Erreur du type « mapping values are not allowed here ».

Correction :

```yaml
titre: "Portail étudiant : FST Settat"
```

Autre piège : `chemin: "C:\Users\etudiant"` (guillemets doubles) provoque une erreur, car `\U` est lu comme une séquence d’échappement Unicode invalide. Utiliser des guillemets simples.

### 2.4 Les nombres entiers et décimaux

**1. Explication.** Un nombre écrit sans guillemets est interprété comme un nombre. Les entiers s’écrivent `42`, `-7`. Les décimaux utilisent le **point** comme séparateur : `3.14`. Le séparateur décimal **n’est jamais la virgule**.

**2. Exemple minimal** **[Exécutable]**

```yaml
port: 8080
temperature: 36.6
```

**3. Exemple réaliste** **[Exécutable]**

```yaml
cluster:
  noeuds: 3
  cpu_par_noeud: 4
  charge_max: 0.85
  decalage_horaire: -1
```

**4. Explication ligne par ligne**

- `noeuds: 3` et `cpu_par_noeud: 4` : entiers.
- `charge_max: 0.85` : décimal (flottant).
- `decalage_horaire: -1` : entier négatif.

**5. Erreur fréquente** : virgule décimale.

```yaml
charge_max: 0,85
```

**[Incorrect]** (au sens sémantique) Ce n’est pas une erreur de syntaxe : `0,85` est lu comme une **chaîne** et non comme un nombre. Un programme qui attend un nombre échouera plus tard.

Correction :

```yaml
charge_max: 0.85
```

> **Attention.** La notation scientifique n’est pas gérée de façon identique partout : `1e3` ou `1.5e3` sont des flottants en YAML 1.2, mais PyYAML les lit comme des chaînes (il exige par exemple `1.0e+3`). En cas de doute, écrire `1500.0`.

### 2.5 Les valeurs booléennes et nulles

**1. Explication.** Un **booléen** vaut vrai ou faux : on écrit `true` ou `false` (en minuscules). Une valeur **nulle** représente « pas de valeur » : on écrit `null`, `~`, ou on laisse la valeur vide.

**2. Exemple minimal** **[Exécutable]**

```yaml
actif: true
commentaire_admin: null
```

**3. Exemple réaliste** **[Exécutable]**

```yaml
service:
  nom: sauvegarde
  active: true
  debug: false
  derniere_execution: null
  proprietaire: ~
  description:
```

**4. Explication ligne par ligne**

- `active: true`, `debug: false` : booléens.
- `derniere_execution: null` : valeur nulle explicite.
- `proprietaire: ~` : le tilde est une autre écriture de `null`.
- `description:` : valeur vide, donc nulle aussi (ce n’est **pas** une chaîne vide).

**5. Erreur fréquente** : écrire le booléen dans une autre langue ou une autre forme.

```yaml
debug: Faux
actif: Oui
```

**[Incorrect]** `Faux` et `Oui` sont des **chaînes** (non vides, donc « vraies » dans beaucoup de langages !). Correction :

```yaml
debug: false
actif: true
```

### 2.6 Les valeurs ambiguës

**1. Explication.** YAML **devine** le type d’une valeur écrite sans guillemets. Cette interprétation implicite réserve des surprises, d’autant plus qu’elle varie selon la **version de YAML** et le **parseur**.

| Valeur écrite | Interprétation YAML 1.1 (ex. PyYAML) | Interprétation YAML 1.2 (schéma de base) | Pour obtenir une chaîne |
|---|---|---|---|
| `NO`, `yes`, `on`, `off`, `y`, `n` | Booléen | Chaîne | `"NO"` |
| `1.10` | Nombre `1.1` | Nombre `1.1` | `"1.10"` |
| `0755` | Entier **octal** (493) | Entier 755 | `"0755"` |
| `22:22` ou `12:30` | Entier en base 60 (« sexagésimal ») : 1342 et 750 | Chaîne | `"22:22"` |
| `2026-10-28` | Date (type timestamp, selon le parseur) | Chaîne (pas de type date dans le schéma de base) | `"2026-10-28"` |
| `1e3` | Chaîne (PyYAML) | Flottant | `"1e3"` ou `1000.0` |
| *(valeur vide)* | `null` | `null` | `""` |

**2. Exemple minimal** **[Exécutable]**

```yaml
pays: NO
```

Avec un parseur YAML 1.1 comme PyYAML, `pays` vaut le booléen `False` : c’est le célèbre « problème de la Norvège » (le code pays `NO`).

**3. Exemple réaliste** **[Exécutable]**

```yaml
release:
  version: "3.10"
  pays: "NO"
  permissions: "0755"
  mapping_ssh: "22:22"
  date: "2026-10-28"
```

**4. Explication ligne par ligne**

- Chaque valeur est entre guillemets : le parseur la lit **toujours** comme chaîne, quelle que soit la version de YAML.
- Sans guillemets, `3.10` deviendrait `3.1`, `NO` deviendrait `False`, `0755` deviendrait `493`, `22:22` deviendrait `1342`.

**5. Erreur fréquente** : correspondance de ports dans un fichier de configuration.

```yaml
ports:
  - 22:22
```

**[Incorrect]** Avec un parseur YAML 1.1, `22:22` est lu comme le nombre `1342`. C’est pourquoi la documentation de Docker Compose recommande de **toujours mettre les ports entre guillemets**.

Correction :

```yaml
ports:
  - "22:22"
```

> **Remarque.** Le fichier de workflow GitHub Actions contient une clé nommée `on`. Un parseur YAML 1.1 strict la lirait comme le booléen `true` ; l’outil GitHub la traite correctement comme une clé. Ce genre de cas illustre pourquoi il faut connaître le comportement de l’**outil cible**.

### 2.7 Les commentaires

**1. Explication.** Un commentaire commence par `#` et se termine à la fin de la ligne. Pour être un commentaire, le `#` doit être **en début de ligne** ou **précédé d’un espace**. YAML **n’a pas** de commentaires multilignes (`/* … */` n’existe pas). Un `#` à l’intérieur d’une chaîne entre guillemets est du texte.

**2. Exemple minimal** **[Exécutable]**

```yaml
# Port du serveur
port: 8080
```

**3. Exemple réaliste** **[Exécutable]**

```yaml
# Configuration de la sauvegarde nocturne
sauvegarde:
  heure: "23:30"      # format HH:MM, entre guillemets
  retention_jours: 14 # durée de conservation
  couleur_statut: "#2E8B57"
  site: http://exemple.ma/page#section
```

**4. Explication ligne par ligne**

- Ligne 1 : commentaire sur toute la ligne.
- `heure` : le commentaire suit la valeur, précédé d’au moins un espace.
- `couleur_statut` : le `#` est dans une chaîne entre guillemets, c’est du **texte**.
- `site` : le `#` n’est pas précédé d’un espace, donc il fait partie de l’URL.

**5. Erreur fréquente** : `#` non protégé dans une valeur.

```yaml
couleur: #2E8B57
```

**[Incorrect]** Tout ce qui suit `#` (précédé d’un espace) est un commentaire : `couleur` vaut `null`, sans message d’erreur.

Correction :

```yaml
couleur: "#2E8B57"
```

Autre erreur : utiliser `//` ou `/* */` comme dans d’autres langages. Ces symboles sont lus comme du texte et provoquent des erreurs ou des valeurs fausses. Seul `#` est valide.

### 2.8 Espaces, tabulations et indentation

**1. Explication.** L’**indentation** est le décalage d’une ligne par rapport à la précédente. En YAML :

- on indente **uniquement avec des espaces** ; les **tabulations sont interdites** pour l’indentation ;
- tous les éléments d’un même niveau ont **exactement la même indentation** ;
- un élément enfant est **plus indenté** que son parent ;
- la convention courante est **2 espaces** par niveau (Compose, Kubernetes) ;
- il faut un **espace** après `:` et après `-`.

**2. Exemple minimal** **[Exécutable]**

```yaml
parent:
  enfant: valeur
```

**3. Exemple réaliste** **[Exécutable]**

```yaml
projet:
  nom: tp-cloud
  equipe:
    responsable: Sara
    membres:
      - Amine
      - Nadia
  actif: true
```

**4. Explication ligne par ligne**

- `projet:` est au niveau 0 (aucun espace).
- `nom`, `equipe`, `actif` sont au niveau 1 (2 espaces) : ce sont les enfants de `projet`.
- `responsable` et `membres` sont au niveau 2 (4 espaces) : enfants de `equipe`.
- Les éléments `- Amine` et `- Nadia` sont au niveau 3 (6 espaces) : éléments de la liste `membres`.
- `actif: true` revient au niveau 1 : il appartient à `projet`, pas à `equipe`.

**5. Erreur fréquente** : indentation incohérente.

```yaml
projet:
  nom: tp-cloud
   version: 1
```

**[Incorrect]** `version` est indentée de 3 espaces alors que `nom` l’est de 2. Le parseur ne sait pas à quel niveau la rattacher (erreur de syntaxe).

Correction :

```yaml
projet:
  nom: tp-cloud
  version: 1
```

Autre erreur : une **tabulation** en début de ligne. Elle est invisible, d’où des erreurs difficiles à repérer (voir partie 4). Configurer l’éditeur pour afficher les espaces et insérer des espaces à la touche `Tab` (voir partie 6).

### 2.9 Les erreurs de syntaxe les plus courantes

| # | Erreur | Exemple incorrect | Correction |
|---|---|---|---|
| 1 | Tabulation pour indenter | `<TAB>nom: x` | Utiliser des espaces |
| 2 | Indentation incohérente | `nom: a` puis `␣␣␣port: 1` | Aligner les clés d’un même niveau |
| 3 | Espace manquant après `:` | `port:8080` | `port: 8080` |
| 4 | Espace manquant après `-` | `-web` | `- web` |
| 5 | `: ` dans une valeur non protégée | `titre: Cours : YAML` | `titre: "Cours : YAML"` |
| 6 | Valeur commençant par un caractère spécial | `mdp: @dmin` | `mdp: "@dmin"` |
| 7 | `#` pris pour du texte | `couleur: #fff` | `couleur: "#fff"` |
| 8 | Clés dupliquées | deux fois `port:` au même niveau | Garder une seule clé |
| 9 | Guillemets non fermés | `nom: "Alice` | `nom: "Alice"` |
| 10 | Type implicite non voulu | `version: 1.10`, `pays: NO` | `"1.10"`, `"NO"` |

### Points clés de la partie 2

- Une paire clé-valeur s’écrit `clé: valeur`, avec un espace après les deux-points.
- On indente avec des espaces, jamais avec des tabulations.
- Les guillemets protègent les caractères spéciaux et forcent le type chaîne.
- Les types implicites (`NO`, `1.10`, `0755`, `22:22`) sont une source d’erreurs : en cas de doute, utiliser des guillemets.
- Seul `#` introduit un commentaire.

---

## Partie 3. Les structures de données en YAML

YAML permet de représenter trois grandes familles de données : les **scalaires** (valeurs simples), les **listes** (séquences ordonnées) et les **mappings** (objets, ensembles de paires clé-valeur). On les combine ensuite pour construire des structures complexes.

### 3.A Les scalaires

Un **scalaire** est une valeur simple, non décomposable : chaîne, entier, décimal, booléen, nul. Leur syntaxe a été détaillée dans la partie 2 ; voici un récapitulatif.

| Type | Exemple | Remarque |
|---|---|---|
| Chaîne | `nom: Alice`, `nom: "Alice"` | Guillemets si caractères spéciaux ou type ambigu |
| Entier | `age: 20` | Pas de séparateur de milliers |
| Décimal | `moyenne: 14.5` | Point décimal |
| Booléen | `actif: true` | `true` / `false` en minuscules |
| Nul | `tel: null`, `tel: ~`, `tel:` | Valeur absente |

**[Exécutable]**

```yaml
etudiant: Salma
age: 20
moyenne: 14.5
inscrite: true
telephone: null
```

Lecture : cinq paires, une par type de scalaire. Chaque valeur est lue avec le type correspondant.

> **Mini-exercice 3.1.** Donner le type de chaque valeur : `a: 7`, `b: "7"`, `c: 7.0`, `d: false`, `e: ~`, `f: sept`.

### 3.B Les listes

Une **liste** (ou séquence) est une suite ordonnée d’éléments. Deux écritures existent : le **style bloc** (un `- ` par élément) et le **style flux** (entre crochets, sur une ligne).

#### Listes simples

**[Exécutable]**

```yaml
# Style bloc
outils:
  - Docker
  - Git
  - Ansible

# Style flux (équivalent)
outils_flux: [Docker, Git, Ansible]
```

Lecture : `outils` est une liste de 3 chaînes. Chaque élément commence par `- ` (tiret + espace), tous alignés. `outils_flux` donne exactement la même liste.

#### Listes de nombres

```yaml
ports_ouverts:
  - 22
  - 80
  - 443
temperatures: [21.5, 22.0, 19.8]
```

Les éléments sont des entiers ou des décimaux, car aucun guillemet n’est présent.

#### Listes d’objets

Chaque élément de la liste est un mapping. Le `- ` introduit l’élément et la **première clé** suit sur la même ligne ; les autres clés s’alignent dessous.

**[Exécutable]**

```yaml
serveurs:
  - nom: web-01
    ip: 10.0.0.11
  - nom: web-02
    ip: 10.0.0.12
```

Lecture : `serveurs` contient deux objets, chacun avec les clés `nom` et `ip`. Notez que `ip` est alignée sous `nom` (4 espaces), et non sous le tiret. `10.0.0.11` est une chaîne (deux points ou plus : ce n’est pas un nombre).

#### Listes imbriquées

```yaml
matrice:
  - [1, 2, 3]
  - [4, 5, 6]

groupes_tp:
  - - alice
    - bob
  - - chloe
```

Lecture : `matrice` est une liste de deux listes de trois nombres. `groupes_tp` utilise l’écriture bloc compacte `- -` : le premier élément est la liste `[alice, bob]`, le second est `[chloe]`.

#### Listes vides

```yaml
taches: []
```

> **Attention.** `taches:` (sans rien) vaut `null`, **pas** une liste vide. Pour une liste vide, écrire `[]` explicitement. De même, un mapping vide s’écrit `{}`.

> **Mini-exercice 3.2.** Écrire la liste des ports 80, 443 et 8080 en style bloc puis en style flux.

### 3.C Les objets ou mappings

Un **mapping** est un ensemble de paires clé-valeur. Une valeur peut elle-même être un mapping ou une liste.

#### Paires clé-valeur et objets imbriqués

**[Exécutable]**

```yaml
base_donnees:
  moteur: postgresql
  connexion:
    hote: db.local
    port: 5432
```

Lecture : `base_donnees` contient `moteur` et `connexion`, qui est un mapping imbriqué. On accède à une valeur en suivant le chemin : `base_donnees` → `connexion` → `port`.

#### Objets contenant des listes

```yaml
application:
  nom: portail
  environnements:
    - dev
    - test
    - prod
```

#### Structure complexe sur plusieurs niveaux

**[Exécutable]**

```yaml
infrastructure:
  region: eu-west-3
  reseau:
    cidr: 10.0.0.0/16
    sous_reseaux:
      - nom: public
        cidr: 10.0.1.0/24
      - nom: prive
        cidr: 10.0.2.0/24
  instances:
    - nom: web-01
      type: t3.micro
      tags:
        role: web
        env: production
```

Lecture :

- Niveau 1 : `infrastructure` (mapping).
- Niveau 2 : `region` (chaîne), `reseau` (mapping), `instances` (liste).
- `reseau.sous_reseaux` est une liste de deux mappings.
- `instances[0].tags` est un mapping de deux paires.
- `10.0.0.0/16` est une chaîne (notation CIDR : adresse réseau et masque).

Un mapping vide s’écrit `{}` ; le style flux existe aussi pour les mappings : `tags: {role: web, env: production}`.

> **Mini-exercice 3.3.** Écrire un objet `vm` avec les clés `nom`, `cpu` et `disques` (liste de deux objets ayant `nom` et `taille_go`).

### 3.D Les structures avancées

#### Chaînes multilignes : `|` et `>`

- `|` (**style littéral**) : conserve les retours à la ligne tels qu’écrits.
- `>` (**style replié**) : remplace les retours à la ligne simples par des espaces ; une **ligne vide** crée un vrai saut de ligne.

**[Exécutable]**

```yaml
script_litteral: |
  echo "Début"
  apt-get update
  echo "Fin"

description_repliee: >
  Ce texte est écrit sur
  plusieurs lignes mais sera
  lu comme une seule ligne.
```

Valeurs obtenues :

- `script_litteral` → `echo "Début"\napt-get update\necho "Fin"\n` (3 lignes, retour final conservé).
- `description_repliee` → `Ce texte est écrit sur plusieurs lignes mais sera lu comme une seule ligne.\n`.

Avec le style replié, une ligne vide sépare des paragraphes :

```yaml
paragraphes: >
  Premier paragraphe
  sur deux lignes.

  Second paragraphe.
```

Résultat : `Premier paragraphe sur deux lignes.\nSecond paragraphe.\n`.

> Usage typique : écrire un script shell dans une étape de pipeline CI (style `|`) ou une longue description (style `>`).

#### Gestion des fins de ligne : `|-`, `|+`, `>-`, `>+`

Un **indicateur de chomping** (gestion des retours à la ligne finaux) se place après `|` ou `>` :

| Indicateur | Nom | Effet sur les retours à la ligne **finaux** |
|---|---|---|
| (aucun) | *clip* | Conserve **un seul** retour final |
| `-` | *strip* | **Supprime** tous les retours finaux |
| `+` | *keep* | **Conserve** tous les retours finaux (y compris les lignes vides) |

**[Exécutable]**

```yaml
a: |
  texte
b: |-
  texte
c: |+
  texte

d: fin
```

Valeurs : `a` → `"texte\n"`, `b` → `"texte"`, `c` → `"texte\n\n"` (la ligne vide avant `d` est conservée). Les indicateurs `>-` et `>+` fonctionnent de la même façon avec le style replié.

> **Remarque.** `|-` est très utile quand on veut une valeur **sans** saut de ligne final, par exemple pour une clé ou un mot de passe lu depuis un fichier.

#### Ancres et alias : `&` et `*`

Une **ancre** (`&nom`) marque une valeur ; un **alias** (`*nom`) la réutilise ailleurs. Cela évite de répéter des blocs identiques.

**[Exécutable]**

```yaml
ville_ecole: &ville Settat
ville_stage: *ville

limites_defaut: &limites
  cpu: 2
  ram_go: 4

vm_web:
  limites: *limites
vm_db:
  limites: *limites
```

Lecture : `ville_stage` vaut `Settat`. `vm_web.limites` et `vm_db.limites` valent tous deux `{cpu: 2, ram_go: 4}`. L’ancre doit être définie **avant** l’alias. Après lecture, l’information « c’était un alias » est perdue : le programme reçoit simplement les valeurs.

#### Fusion de mappings avec `<<`

La **clé de fusion** `<<` insère les paires d’un mapping dans un autre ; les clés écrites localement **remplacent** celles fusionnées.

**[Exécutable avec PyYAML]**

```yaml
base_service: &base_service
  restart: unless-stopped
  environnement: production

web:
  <<: *base_service
  port: 8080

worker:
  <<: *base_service
  environnement: test
```

Résultat : `web` = `{restart: unless-stopped, environnement: production, port: 8080}` ; `worker` = `{restart: unless-stopped, environnement: test}`.

> **Attention : fonctionnalité historique et non portable de manière identique.** La clé de fusion `<<` provient de la spécification **YAML 1.1** ; elle ne fait pas partie du schéma de base de YAML 1.2. De nombreuses bibliothèques et certains outils la prennent en charge (PyYAML, Docker Compose, GitLab CI…), mais pas tous, et pas forcément avec exactement le même comportement. Avant de l’utiliser, vérifier la documentation de l’outil cible. Les ancres et alias simples (`&` et `*`) sont, eux, définis par YAML 1.2.

#### Documents multiples : `---` et `...`

Un fichier peut contenir **plusieurs documents** YAML. `---` marque le début d’un document ; `...` marque (facultativement) sa fin.

```yaml
---
nom: document-1
valeur: 10
---
nom: document-2
valeur: 20
...
```

Kubernetes utilise couramment `---` pour mettre **plusieurs ressources dans un même fichier** (voir partie 9). En Python, on lit plusieurs documents avec `yaml.safe_load_all(...)`.

#### Clés complexes

Par défaut, une clé est une chaîne. YAML permet aussi des clés « complexes » (listes, mappings) avec l’indicateur `?` :

```yaml
? [nord, sud]
: axe_vertical
```

Ici, la clé est la liste `[nord, sud]`. Cette fonctionnalité est **rarement nécessaire** dans les configurations courantes, car :

- JSON, le format d’échange le plus répandu, n’accepte que des clés de type chaîne ;
- la plupart des langages ne peuvent pas utiliser une liste comme clé de dictionnaire (en Python, une liste n’est pas hachable ; PyYAML lève une erreur) ;
- les outils DevOps attendent des clés simples (noms de champs).

Pour une clé contenant des espaces ou des caractères spéciaux, un simple guillemet suffit : `"clé avec : deux-points": valeur`.

> **Mini-exercice 3.4.** Écrire un champ `message` sur deux lignes avec `|`, puis deux clés `region_a` et `region_b` réutilisant la même valeur `eu-west-3` grâce à une ancre.

### Points clés de la partie 3

- Trois familles : scalaires, listes (`- `), mappings (`clé: valeur`).
- Les structures s’emboîtent librement ; l’indentation indique l’appartenance.
- `|` conserve les sauts de ligne ; `>` les replie ; `-` et `+` règlent les fins de ligne.
- Ancres et alias évitent les répétitions ; `<<` est historique et non portable de manière identique.
- Les clés complexes existent mais sont rares.

---

## Partie 4. Comprendre l’indentation et les erreurs

### 4.1 Pourquoi l’indentation est fondamentale

Dans JSON, la structure est indiquée par des accolades `{}` et des crochets `[]`. Dans YAML, elle est indiquée par l’**indentation**. Déplacer une ligne de deux espaces peut :

- produire une erreur de syntaxe (cas favorable : on le voit tout de suite) ;
- ou, plus grave, **changer le sens** du fichier sans erreur (la donnée change de parent).

Illustration de ce second cas, avec deux fichiers valides mais différents :

```yaml
# Fichier A : debug appartient à "service"
service:
  nom: api
  debug: true
```

```yaml
# Fichier B : debug appartient à la racine
service:
  nom: api
debug: true
```

### 4.2 Un exemple correctement indenté

**[Exécutable]**

```yaml
application:
  nom: portail
  serveurs:
    - nom: web-01
      ports:
        - 80
        - 443
    - nom: web-02
      ports:
        - 80
```

Lecture : chaque niveau ajoute 2 espaces. Les clés d’un même objet (`nom`, `ports`) sont alignées. Les `- ` d’une même liste sont alignés.

> **Remarque.** YAML autorise aussi à écrire les éléments d’une liste **au même niveau** que la clé parente :
>
> ```yaml
> serveurs:
> - web-01
> - web-02
> ```
>
> C’est valide, mais moins lisible. Nous recommandons de toujours indenter les listes de 2 espaces, comme dans les exemples du chapitre.

### 4.3 Indentation incohérente

**[Incorrect]**

```yaml
base:
  hote: localhost
   port: 5432
```

**Problème.** `hote` est indentée de 2 espaces, `port` de 3. Le parseur ne peut ni la considérer comme sœur de `hote`, ni comme un enfant valide (`hote` a déjà une valeur). Il signale une erreur.

**Corrigé :**

```yaml
base:
  hote: localhost
  port: 5432
```

**[Incorrect]** Une tabulation (la ligne `hote` ci-dessous commence par un caractère de tabulation, invisible à l’écran) :

```yaml
base:
	hote: localhost
  port: 5432
```

**Problème.** La première indentation est une tabulation, ce qui est **interdit** par YAML. Comme la tabulation est invisible dans la plupart des éditeurs, l’erreur est difficile à repérer.

**Corrigé :** remplacer la tabulation par deux espaces.

### 4.4 Listes mal positionnées

**[Incorrect]**

```yaml
serveurs:
  - nom: web-01
  ip: 10.0.0.11
```

**Problème.** `ip` est indentée comme le `-`, pas comme `nom`. Elle ne peut donc pas appartenir à l’objet de la liste, et elle n’est pas non plus au bon niveau pour être une clé de la racine. Le parseur signale une erreur.

**Corrigé :** aligner `ip` sous `nom`.

```yaml
serveurs:
  - nom: web-01
    ip: 10.0.0.11
```

**[Incorrect, erreur de sens]**

```yaml
ports:
  - 80
  - 443
nom: web
  - 8080
```

**Problème.** L’élément `- 8080` est indenté sous `nom: web` : il ne peut pas être ajouté à la liste `ports`, et un scalaire ne peut pas contenir de liste. Erreur de syntaxe.

**Corrigé :**

```yaml
ports:
  - 80
  - 443
  - 8080
nom: web
```

### 4.5 Deux-points et caractères spéciaux mal utilisés

**[Incorrect]**

```yaml
message: Erreur: disque plein
commande: echo: bonjour
```

**Problème.** La séquence `: ` (deux-points suivi d’un espace) est toujours un séparateur clé-valeur. Une valeur ne peut donc pas en contenir sans protection.

**Corrigé :**

```yaml
message: "Erreur : disque plein"
commande: 'echo "bonjour"'
```

### 4.6 Types de données involontairement interprétés

**[Incorrect, au sens sémantique]** (aucune erreur de syntaxe, mais des valeurs inattendues avec un parseur YAML 1.1 comme PyYAML)

```yaml
projet:
  version: 3.10
  pays: NO
  permissions: 0755
  heure_sauvegarde: 23:30
  actif: yes
```

**Problème.** `version` devient le nombre `3.1`, `pays` devient `False`, `permissions` devient l’entier octal `493`, `heure_sauvegarde` devient l’entier `1410` (base 60), `actif` devient `True`. Aucune erreur n’est signalée.

**Corrigé :** forcer les chaînes avec des guillemets et utiliser `true` pour les booléens.

```yaml
projet:
  version: "3.10"
  pays: "NO"
  permissions: "0755"
  heure_sauvegarde: "23:30"
  actif: true
```

### 4.7 Chaînes contenant des caractères spéciaux

**[Incorrect]**

```yaml
mot_de_passe: @dmin2026
motif: *.log
modele: {nom}
priorite: !haute
```

**Problème.** Chaque valeur commence par un caractère réservé : `@` est réservé, `*` introduit un alias, `{` ouvre un mapping en style flux, `!` introduit une balise (*tag*). Le parseur signale une erreur ou interprète la valeur autrement.

**Corrigé :**

```yaml
mot_de_passe: "@dmin2026"
motif: "*.log"
modele: "{nom}"
priorite: "!haute"
```

> **Remarque.** Un mot de passe ne doit évidemment pas être stocké en clair dans un fichier versionné : l’exemple n’illustre que la syntaxe (voir parties 8 et 10).

### Points clés de la partie 4

- L’indentation exprime la structure : un décalage peut casser le fichier ou changer son sens.
- Jamais de tabulation ; 2 espaces par niveau.
- Les clés d’un même niveau sont alignées ; dans une liste d’objets, les clés s’alignent sous la première clé.
- Protéger par des guillemets les valeurs contenant `: ` ou commençant par un caractère spécial.
- Les erreurs de **typage implicite** sont silencieuses : ce sont les plus dangereuses.

---

## Partie 5. YAML et JSON

**JSON** (*JavaScript Object Notation*) est un autre format de données, très utilisé dans les API web. Les deux formats représentent des structures similaires (objets, listes, nombres, chaînes, booléens, valeurs nulles), mais leur écriture et leurs possibilités diffèrent.

### 5.1 Un même exemple en YAML puis en JSON

**YAML** **[Exécutable]**

```yaml
# Configuration d'un service
service:
  nom: api-notes
  port: 8000
  actif: true
  replicas: 3
  tags:
    - python
    - fastapi
  base_donnees:
    hote: db.local
    port: 5432
  limites: null
```

**JSON équivalent** **[Exécutable]**

```json
{
  "service": {
    "nom": "api-notes",
    "port": 8000,
    "actif": true,
    "replicas": 3,
    "tags": ["python", "fastapi"],
    "base_donnees": {
      "hote": "db.local",
      "port": 5432
    },
    "limites": null
  }
}
```

Observations : le commentaire a disparu (JSON n’en admet pas) ; les accolades, crochets, virgules et guillemets sur les clés sont obligatoires en JSON.

### 5.2 Tableau comparatif

| Critère | YAML | JSON |
|---|---|---|
| Syntaxe | Indentation, `:`, `- ` | Accolades, crochets, virgules, guillemets doubles |
| Lisibilité | Très lisible pour de la configuration | Plus verbeux, moins agréable à lire à la main |
| Commentaires | Oui (`#`) | Non (dans la norme) |
| Types de base | Chaînes, nombres, booléens, nul, listes, mappings ; types supplémentaires selon le schéma et le parseur (dates…) | Chaînes, nombres, booléens, `null`, tableaux, objets |
| Listes et objets | Style bloc ou style flux | `[]` et `{}` uniquement |
| Chaînes multilignes | Oui (`\|`, `>`) | Non (il faut écrire `\n`) |
| Réutilisation (ancres, alias) | Oui | Non |
| Documents multiples dans un fichier | Oui (`---`) | Non (un seul document) |
| Écriture manuelle | Facile, mais sensible à l’indentation | Moins facile (virgules, guillemets), plus strict |
| Typage implicite | Oui (source de pièges) | Non : les types se voient directement |
| Usage typique | Fichiers de configuration (Compose, Kubernetes, CI/CD) | Échanges d’API, stockage de données, configuration machine |

### 5.3 Utilisation dans les API, les applications et les fichiers de configuration

- **API web** : JSON est le format d’échange dominant (réponses HTTP, requêtes). Il est simple à analyser et strict.
- **Fichiers de configuration écrits par des humains** : YAML est privilégié grâce aux commentaires et à sa lisibilité.
- **Outils DevOps** : beaucoup d’outils acceptent YAML et JSON ; Kubernetes, par exemple, convertit en interne les manifestes YAML en JSON.

### 5.4 Ce qu’il faut retenir sur leur relation

- Les deux formats peuvent représenter des **structures de données similaires**. La spécification YAML 1.2 a été conçue pour que la plupart des documents JSON soient aussi des documents YAML valides.
- Ils ne sont **pas interchangeables sans conversion** : un fichier écrit en YAML idiomatique (indentation, commentaires, ancres) n’est pas du JSON valide.
- La conversion peut **perdre ou modifier des informations** : les commentaires disparaissent ; les ancres et alias sont remplacés par les valeurs dupliquées ; les documents multiples ne rentrent pas dans un seul document JSON ; l’ordre des clés et le style de présentation ne sont pas conservés ; les types implicites YAML (dates, octal…) peuvent produire des valeurs différentes selon le parseur.

### 5.5 Convertir avec Python

**[Exécutable]** (prérequis : Python 3 et PyYAML, voir partie 6)

```python
import json
import yaml

# YAML -> JSON
with open("service.yaml", encoding="utf-8") as f:
    donnees = yaml.safe_load(f)

print(json.dumps(donnees, indent=2, ensure_ascii=False))

# JSON -> YAML
texte_json = '{"serveur": {"hote": "10.0.0.5", "port": 22}}'
objet = json.loads(texte_json)
print(yaml.safe_dump(objet, sort_keys=False, allow_unicode=True))
```

Explication :

- `yaml.safe_load` lit le YAML et produit des structures Python (dictionnaires, listes…).
- `json.dumps` transforme ces structures en texte JSON ; `ensure_ascii=False` conserve les accents.
- `json.loads` fait l’opération inverse ; `yaml.safe_dump` produit du YAML ; `sort_keys=False` conserve l’ordre des clés.

### Points clés de la partie 5

- YAML et JSON décrivent des structures proches, avec des syntaxes différentes.
- YAML ajoute commentaires, multilignes, ancres et documents multiples ; JSON est plus strict et plus simple à analyser.
- Une conversion YAML vers JSON perd les commentaires et les ancres.

---

## Partie 6. Création et organisation des fichiers YAML

### 6.1 Créer un fichier `.yaml` ou `.yml`

Un fichier YAML est un **fichier texte brut**. On peut le créer avec n’importe quel éditeur de texte, ou en ligne de commande :

```bash
mkdir -p mon-projet/config
touch mon-projet/config/app.yaml
```

Explication : `mkdir -p` crée le dossier (et les dossiers parents s’il le faut) ; `touch` crée un fichier vide.

> **Attention.** Ne pas utiliser un traitement de texte (Word, LibreOffice Writer) : ils ajoutent une mise en forme qui rend le fichier invalide. Utiliser un éditeur de texte ou de code, avec l’encodage **UTF-8**.

### 6.2 Différence entre `.yaml` et `.yml`

Les deux extensions désignent **le même format**. L’extension `.yaml` est celle recommandée par la communauté YAML ; `.yml` est une abréviation héritée des anciens systèmes limités à trois lettres. Le contenu est identique.

En pratique, **l’outil impose parfois un nom** :

| Outil | Noms reconnus (exemples) |
|---|---|
| Docker Compose | `compose.yaml` (nom préféré), `compose.yml`, `docker-compose.yaml`, `docker-compose.yml` |
| GitHub Actions | tout fichier `.yml` ou `.yaml` dans `.github/workflows/` |
| GitLab CI | `.gitlab-ci.yml` (nom par défaut) |
| Kubernetes (`kubectl apply -f`) | l’extension n’a pas d’importance |

Règle simple : **choisir une extension et s’y tenir dans un même projet**, sauf si l’outil impose un nom précis.

### 6.3 Écrire du YAML avec Visual Studio Code

1. Créer le fichier (`Fichier > Nouveau fichier`) et l’enregistrer avec l’extension `.yaml`. VS Code détecte le langage YAML.
2. Installer l’extension **YAML** (éditée par Red Hat) : elle propose la coloration, la validation de syntaxe, l’autocomplétion et la validation par schéma pour de nombreux formats.
3. Configurer l’éditeur pour éviter les erreurs d’indentation. Ouvrir `settings.json` (palette de commandes : « Preferences: Open User Settings (JSON) ») et ajouter :

```json
{
  "[yaml]": {
    "editor.insertSpaces": true,
    "editor.tabSize": 2,
    "editor.detectIndentation": false,
    "editor.autoIndent": "advanced"
  },
  "editor.renderWhitespace": "boundary",
  "files.insertFinalNewline": true,
  "files.trimTrailingWhitespace": true
}
```

Explication :

- `insertSpaces` et `tabSize` : la touche `Tab` insère 2 espaces, jamais un caractère de tabulation.
- `detectIndentation: false` : évite que VS Code « devine » une autre indentation.
- `renderWhitespace` : rend les espaces et tabulations visibles.
- `insertFinalNewline` et `trimTrailingWhitespace` : retour à la ligne final et suppression des espaces inutiles en fin de ligne.

### 6.4 Organiser les fichiers dans un projet

**[Illustratif]**

```text
projet-cloud/
├── README.md
├── compose.yaml              # application multi-conteneurs
├── .env.example              # modèle de variables (sans secret)
├── .gitignore                # contient notamment ".env"
├── config/
│   ├── app.yaml              # configuration de l'application
│   └── logging.yaml
├── k8s/
│   ├── deployment.yaml       # manifestes Kubernetes
│   └── service.yaml
└── .github/
    └── workflows/
        └── ci.yml            # pipeline CI/CD
```

Principes : un fichier = une responsabilité ; des noms explicites ; les fichiers propres à un outil dans le dossier attendu par cet outil ; les secrets hors du dépôt (voir partie 10).

### 6.5 Reconnaître les erreurs de syntaxe

Dans VS Code, les erreurs apparaissent de plusieurs façons : soulignement rouge ou orange, panneau **Problèmes** (`Ctrl+Maj+M`), guides d’indentation. Les messages mentionnent souvent un **numéro de ligne et de colonne**.

> **Conseil.** La ligne indiquée par le parseur est celle où il a **constaté** le problème. La cause réelle se trouve souvent **juste au-dessus** (indentation de la ligne précédente, guillemet non fermé…).

### 6.6 Utiliser un validateur YAML

Un **validateur** (ou *linter*) analyse un fichier et signale des erreurs ou des mauvaises pratiques. Mettons en place un environnement Python isolé, puis installons deux bibliothèques.

```bash
python3 -m venv .venv
source .venv/bin/activate        # Windows (PowerShell) : .venv\Scripts\Activate.ps1
pip install pyyaml yamllint
```

Explication :

- `python3 -m venv .venv` crée un **environnement virtuel** : un espace isolé pour installer des bibliothèques sans toucher au reste du système ;
- `source .venv/bin/activate` l’active ;
- `pyyaml` permet de **lire** du YAML en Python ; `yamllint` est un **linter** en ligne de commande.

Valider un fichier :

```bash
yamllint config/app.yaml
```

`yamllint` signale des erreurs de syntaxe (`error`) et des problèmes de style (`warning`) : indentation, lignes trop longues, clés dupliquées, valeurs « truthy » ambiguës (`yes`, `on`)… Ses règles se configurent dans un fichier `.yamllint` (voir partie 10).

### 6.7 Choisir un outil de validation fiable

| Critère | Pourquoi |
|---|---|
| Logiciel libre, maintenu, largement utilisé | Moins de risques de bugs ou d’abandon |
| Exécutable **en local** | Évite d’envoyer des données sensibles à un site tiers |
| Version de YAML prise en charge indiquée | Comportements différents entre 1.1 et 1.2 |
| Détecte les clés dupliquées | Beaucoup de parseurs les acceptent silencieusement |
| Utilisable en **intégration continue** | Contrôle automatique à chaque modification |
| Messages d’erreur clairs (ligne, colonne) | Diagnostic rapide |

> **Attention.** Ne **collez jamais** un fichier contenant des mots de passe, jetons ou clés dans un validateur en ligne.

> **Remarque importante.** Un éditeur ou un validateur YAML vérifie en général la **syntaxe**, éventuellement la conformité à un **schéma** si on lui en fournit un. Il **ne garantit pas** que la configuration fonctionnera : images absentes, ports occupés, droits manquants, mauvaises valeurs métier… (voir partie 7). Pour valider un fichier destiné à un outil, utiliser aussi **les commandes de validation de cet outil** (`docker compose config`, `kubectl apply --dry-run`, `ansible-lint`, `actionlint`…).

### 6.8 Lire un fichier YAML depuis Python

Fichier `config/app.yaml` **[Exécutable]** :

```yaml
service:
  nom: api-notes
  port: 8000
  actif: true
  tags:
    - python
    - fastapi
```

Script `lire_config.py` **[Exécutable]** (prérequis : `pyyaml` installé) :

```python
from pathlib import Path

import yaml

chemin = Path("config/app.yaml")

try:
    with chemin.open(encoding="utf-8") as fichier:
        config = yaml.safe_load(fichier)
except FileNotFoundError:
    raise SystemExit(f"Fichier introuvable : {chemin}")
except yaml.YAMLError as erreur:
    raise SystemExit(f"YAML invalide : {erreur}")

print(config["service"]["nom"])        # api-notes
print(config["service"]["port"] + 1)   # 8001 (le port est un entier)
print(config["service"]["tags"][0])    # python
```

Explication :

- `Path` représente un chemin de fichier ; `open(encoding="utf-8")` ouvre le fichier en lecture.
- `yaml.safe_load` lit le YAML et renvoie des structures Python : mapping → `dict`, liste → `list`, `true` → `True`, `null` → `None`.
- Les `try/except` gèrent les deux erreurs classiques : fichier absent et YAML invalide (`yaml.YAMLError`).
- On accède aux valeurs avec des clés et des indices : `config["service"]["tags"][0]`.

> **Attention : sécurité.** Utiliser **`yaml.safe_load`**, jamais `yaml.load` sans chargeur sûr sur des données qui ne sont pas de confiance : certaines formes de chargement peuvent construire des objets arbitraires et exécuter du code.

> **Remarque.** PyYAML suit en grande partie YAML 1.1. Pour YAML 1.2, d’autres bibliothèques existent (par exemple `ruamel.yaml`). Vérifier toujours la version prise en charge par l’outil cible.

### Points clés de la partie 6

- `.yaml` et `.yml` sont équivalents ; l’outil peut imposer un nom.
- Configurer l’éditeur : espaces uniquement, tabulations visibles.
- Un validateur vérifie la syntaxe et le style, pas le bon fonctionnement.
- Lire du YAML en Python : `yaml.safe_load`.

---

## Partie 7. Validation et débogage

### 7.1 Catégories d’erreurs

| Catégorie | Description | Exemple |
|---|---|---|
| Erreur de syntaxe | Le fichier n’est pas du YAML | Guillemet non fermé, `: ` dans une valeur |
| Erreur d’indentation | Mauvais niveau, tabulation | Clé décalée d’un espace |
| Type de données incorrect | Le type lu n’est pas celui attendu | `replicas: "3"` au lieu de `3` |
| Clés dupliquées | Même clé deux fois dans un mapping | Deux `port:` au même niveau |
| Erreur de structure | YAML valide mais structure inattendue | Liste à la place d’un mapping, clé mal placée |
| Erreur propre à l’application | Valide pour YAML, rejeté ou inefficace pour l’outil | Clé inconnue, port déjà utilisé, image introuvable |

### 7.2 Trois niveaux de validité

Un fichier peut être « correct » à trois niveaux **différents et successifs** :

| Niveau | Question | Qui vérifie ? | Exemple d’échec |
|---|---|---|---|
| 1. **Syntaxiquement valide** | Est-ce du YAML bien formé ? | Un parseur YAML, `yamllint` | Indentation incohérente |
| 2. **Conforme au schéma de l’application** | Les clés, types et structures sont-ils ceux que l’outil attend ? | L’outil lui-même (`docker compose config`, `kubectl --dry-run`), un schéma (JSON Schema) | Clé `prots` au lieu de `ports` |
| 3. **Fonctionnel à l’exécution** | La configuration produit-elle le résultat attendu dans l’environnement réel ? | Un test d’exécution, les journaux | Port 8080 déjà occupé, image inexistante, mot de passe incorrect |

Chaque niveau suppose le précédent, mais **le réussir ne garantit pas le suivant**.

### 7.3 Fichiers erronés à analyser

Essayez de diagnostiquer chaque cas **avant** d’ouvrir le diagnostic.

#### Cas A

```yaml
application:
  nom: API Notes
  description: Service de notes: version 2
```

<details>
<summary>Diagnostic et correction du cas A</summary>

- **Niveau :** 1 (syntaxe).
- **Cause :** la valeur de `description` contient `: ` (deux-points suivi d’un espace), lu comme un second séparateur clé-valeur.
- **Correction :** `description: "Service de notes : version 2"`.

</details>

#### Cas B

```yaml
services:
  web:
    image: nginx:stable-alpine
     ports:
      - "8080:80"
```

<details>
<summary>Diagnostic et correction du cas B</summary>

- **Niveau :** 1 (indentation).
- **Cause :** `ports` est indentée de 5 espaces alors que `image`, sa sœur, l’est de 4. Le parseur ne peut pas la rattacher.
- **Correction :** aligner `ports` sur `image` (4 espaces) ; les éléments de liste sont alors indentés de 6 espaces.

```yaml
services:
  web:
    image: nginx:stable-alpine
    ports:
      - "8080:80"
```

</details>

#### Cas C

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: "3"
```

*(extrait volontairement incomplet : on ne s’intéresse qu’au champ `replicas`)*

<details>
<summary>Diagnostic et correction du cas C</summary>

- **Niveau :** 1 valide (c’est du YAML correct), 2 invalide.
- **Cause :** `replicas` doit être un **entier** pour Kubernetes ; `"3"` est une chaîne. L’API Kubernetes rejette la ressource avec une erreur de type.
- **Correction :** `replicas: 3`.
- **Leçon :** les guillemets ne sont pas « plus sûrs » : ils changent le type.

</details>

#### Cas D

```yaml
services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: app_user
      POSTGRES_USER: admin
```

<details>
<summary>Diagnostic et correction du cas D</summary>

- **Niveau :** 1 (clé dupliquée).
- **Cause :** `POSTGRES_USER` apparaît deux fois dans le même mapping. La spécification exige des clés **uniques**. Selon l’outil, la duplication est **rejetée** ou la **dernière valeur est silencieusement retenue** (c’est le cas de PyYAML) : le résultat peut donc varier d’un outil à l’autre.
- **Détection :** `yamllint` signale la règle `key-duplicates`.
- **Correction :** ne garder qu’une seule clé, avec la valeur voulue.

</details>

#### Cas E

```yaml
services:
  web:
    image: nginx:stable-alpine
    prots:
      - "8080:80"
```

<details>
<summary>Diagnostic et correction du cas E</summary>

- **Niveau :** 1 valide, 2 invalide.
- **Cause :** faute de frappe : `prots` n’est pas une propriété connue de Compose. YAML ne le sait pas, car il n’a aucun schéma ; `docker compose config` signale une propriété non autorisée.
- **Correction :** `ports`.
- **Leçon :** `yamllint` n’aurait **pas** détecté cette erreur ; c’est l’outil cible qui la détecte.

</details>

#### Cas F

```yaml
services:
  web:
    image: nginx:stable-alpine
    ports:
      - "8080:80"
```

Ce fichier est valide aux niveaux 1 et 2, mais `docker compose up -d` échoue avec un message indiquant que le port est déjà alloué (ou qu’une adresse est déjà utilisée).

<details>
<summary>Diagnostic et correction du cas F</summary>

- **Niveau :** 3 (exécution).
- **Cause :** un autre programme (ou un autre conteneur) utilise déjà le port 8080 de la machine hôte.
- **Diagnostic :** lire le message d’erreur ; identifier le processus ou conteneur qui occupe le port (`docker ps`, ou un outil système).
- **Correction :** arrêter l’autre programme, ou choisir un autre port : `"8081:80"`.

</details>

### 7.4 Méthode de diagnostic

1. **Lire le message d’erreur en entier** : numéro de ligne, colonne, nature du problème.
2. **Regarder autour de la ligne signalée**, surtout la ligne précédente (indentation, guillemets).
3. **Rendre les espaces visibles** et vérifier l’absence de tabulation.
4. **Lancer `yamllint`** pour confirmer la validité syntaxique et repérer les clés dupliquées.
5. **Regarder ce que le parseur a compris** :

   ```bash
   python3 -c "import sys, json, yaml; print(json.dumps(yaml.safe_load(open(sys.argv[1], encoding='utf-8')), indent=2, ensure_ascii=False, default=str))" fichier.yaml
   ```

   Cette commande charge le fichier, puis l’affiche en JSON : on voit les **types réellement interprétés** (`"NO"` devenu `false`, `3.10` devenu `3.1`…). Le paramètre `default=str` évite une erreur pour les valeurs que JSON ne sait pas représenter (comme une date).
6. **Valider avec l’outil cible** (`docker compose config`, `kubectl apply --dry-run=client -f`…).
7. **Tester à l’exécution** et lire les journaux (`docker compose logs`, `kubectl logs`).

### Points clés de la partie 7

- Trois niveaux : syntaxe, conformité au schéma, fonctionnement réel.
- Un fichier peut être valide pour YAML et rejeté par l’outil, ou accepté par l’outil et ne pas fonctionner.
- Afficher ce que le parseur comprend est le moyen le plus sûr de détecter les erreurs de typage.

---

## Partie 8. Application complète avec Docker Compose

### 8.1 Rappels et prérequis

**Docker** est un outil qui exécute des applications dans des **conteneurs** (environnements isolés et légers). Une **image** est le modèle à partir duquel un conteneur est créé (par exemple l’image `nginx`). **Docker Compose** est un outil qui lit un fichier YAML décrivant plusieurs conteneurs (appelés **services**) et les démarre ensemble.

> **Attention.** Distinguons bien : **YAML** est le format du fichier ; **Docker Compose** est l’outil qui l’interprète ; **Nginx** et **PostgreSQL** sont les logiciels exécutés dans les conteneurs. Un fichier `compose.yaml` ne « crée » rien tout seul : il n’a d’effet que si Docker Compose est installé, si les images peuvent être téléchargées et si les fichiers qu’il référence existent.

**Prérequis**

- Docker Engine ou Docker Desktop, avec le plugin Compose v2. Vérification :

  ```bash
  docker --version
  docker compose version
  ```

- Un accès à Internet pour télécharger les images (`nginx` et `postgres`) la première fois.
- Le port `8080` de la machine hôte libre.

> **Remarque.** La commande moderne est `docker compose` (avec un espace). L’ancienne `docker-compose` (avec un tiret) correspond à l’ancienne version 1, désormais abandonnée. Le champ `version:` en tête de fichier est **obsolète** dans la spécification Compose actuelle : Compose l’ignore et affiche un avertissement ; nous ne l’utilisons donc pas.

**Arborescence du TP**

```text
tp-compose/
├── compose.yaml
├── .env              # secrets locaux (NE JAMAIS versionner)
├── .env.example      # modèle sans secret (versionné)
├── .gitignore        # contient la ligne : .env
└── site/
    └── index.html
```

### 8.2 Étape 1 : un seul service Nginx

**[Exécutable]** Créer `site/index.html` :

```html
<!doctype html>
<html lang="fr">
  <head><meta charset="utf-8"><title>TP YAML</title></head>
  <body><h1>Bonjour depuis Nginx</h1></body>
</html>
```

Créer `compose.yaml` :

```yaml
services:
  web:
    image: nginx:stable-alpine
    ports:
      - "8080:80"
    volumes:
      - ./site:/usr/share/nginx/html:ro
```

Explication ligne par ligne :

- `services:` : clé racine ; chaque enfant est un service.
- `web:` : nom choisi pour le service (il servira aussi de nom réseau).
- `image: nginx:stable-alpine` : image `nginx`, étiquette (*tag*) `stable-alpine`, une version stable sur base Alpine (légère). On évite l’étiquette `latest`, qui change dans le temps.
- `ports: - "8080:80"` : redirige le port `8080` de la machine hôte vers le port `80` du conteneur. Format `HÔTE:CONTENEUR`. Les guillemets évitent toute interprétation numérique (voir partie 2).
- `volumes: - ./site:/usr/share/nginx/html:ro` : monte le dossier local `site` dans le conteneur, à l’endroit où Nginx cherche ses pages ; `:ro` (*read-only*) interdit au conteneur de le modifier.

Test : `docker compose up -d`, puis ouvrir `http://localhost:8080`.

### 8.3 Étape 2 : ajout de PostgreSQL

On ajoute un service `db`. À ce stade, les variables sont écrites en clair, ce qui est acceptable **uniquement** pour comprendre la syntaxe :

**[Illustratif]**

```yaml
services:
  web:
    image: nginx:stable-alpine
    ports:
      - "8080:80"

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: app_user
      POSTGRES_PASSWORD: mot_de_passe_en_clair   # à éviter !
      POSTGRES_DB: app_db
```

Les variables `POSTGRES_USER`, `POSTGRES_PASSWORD` et `POSTGRES_DB` sont lues par l’image officielle PostgreSQL pour créer l’utilisateur, son mot de passe et la base au premier démarrage. `POSTGRES_PASSWORD` est obligatoire : sans elle, le conteneur s’arrête.

### 8.4 Étape 3 : sortir les secrets du fichier YAML

Compose peut **substituer** des variables écrites sous la forme `${NOM}` à partir d’un fichier `.env` placé à côté de `compose.yaml`.

Fichier `.env` **[Exécutable]** (ce n’est pas du YAML : une ligne `NOM=valeur` par variable) :

```text
POSTGRES_USER=app_user
POSTGRES_PASSWORD=ChangezMoi_Un_Mot_De_Passe_Long_Et_Unique
POSTGRES_DB=app_db
```

Fichier `.env.example` (modèle, versionné) :

```text
POSTGRES_USER=app_user
POSTGRES_PASSWORD=
POSTGRES_DB=app_db
```

Fichier `.gitignore` :

```text
.env
```

### 8.5 Fichier complet `compose.yaml`

**[Exécutable]**

```yaml
# compose.yaml : Nginx + PostgreSQL (exemple pédagogique)
name: tp-yaml            # nom du projet Compose (préfixe des ressources créées)

services:
  # --- Serveur web ---
  web:
    image: nginx:stable-alpine
    restart: unless-stopped            # redémarre sauf arrêt manuel
    ports:
      - "8080:80"                      # HÔTE:CONTENEUR
    volumes:
      - ./site:/usr/share/nginx/html:ro
    networks:
      - frontend
      - backend
    depends_on:
      db:
        condition: service_healthy     # attend que PostgreSQL soit prêt

  # --- Base de données ---
  db:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER:-app_user}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:?POSTGRES_PASSWORD manquant dans .env}
      POSTGRES_DB: ${POSTGRES_DB:-app_db}
    volumes:
      - db_data:/var/lib/postgresql/data   # volume persistant nommé
    networks:
      - backend                            # pas de port publié vers l'hôte
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s

networks:
  frontend:
  backend:
    internal: true       # réseau sans accès à l'extérieur

volumes:
  db_data:               # déclare le volume nommé (stockage géré par Docker)
```

### 8.6 Explication des sections et des lignes importantes

**Niveau racine**

| Clé | Rôle |
|---|---|
| `name` | Nom du projet Compose ; sert de préfixe aux réseaux, volumes et conteneurs |
| `services` | Les conteneurs à exécuter |
| `networks` | Les réseaux virtuels déclarés |
| `volumes` | Les volumes nommés déclarés |

**Service `web`**

- `restart: unless-stopped` : Docker redémarre le conteneur après un plantage ou un redémarrage de la machine, sauf si on l’a arrêté volontairement.
- `networks: [frontend, backend]` : le service est connecté aux deux réseaux.
- `depends_on` avec `condition: service_healthy` : Compose démarre `web` **après** que `db` soit déclaré « en bonne santé » par son `healthcheck`. Ici, c’est surtout pédagogique : Nginx n’utilise pas encore la base.

**Service `db`**

- `${POSTGRES_USER:-app_user}` : valeur de la variable `POSTGRES_USER` du fichier `.env`, ou `app_user` si elle n’est pas définie (`:-` fournit une valeur par défaut).
- `${POSTGRES_PASSWORD:?message}` : valeur de la variable, ou **échec immédiat** avec le message si elle est absente (`:?`). Aucun mot de passe n’est écrit dans le fichier YAML.
- `volumes: - db_data:/var/lib/postgresql/data` : associe le volume nommé `db_data` au dossier où PostgreSQL stocke ses données. Les données **survivent** à la suppression du conteneur.
- `networks: [backend]` et **absence de `ports`** : la base n’est pas accessible depuis l’extérieur de la machine ; seuls les conteneurs du réseau `backend` la voient.
- `healthcheck` : commande exécutée régulièrement pour savoir si le service est prêt. `pg_isready` est un utilitaire de PostgreSQL. `$$` est la façon d’écrire un littéral `$` dans Compose : la variable est ainsi lue **dans le conteneur**, et non remplacée par Compose sur la machine hôte.
- `interval`, `timeout`, `retries`, `start_period` : fréquence des tests, durée maximale d’un test, nombre d’échecs avant de déclarer le service non sain, délai de grâce au démarrage.

**Réseaux et volumes**

- `frontend:` et `db_data:` n’ont pas de valeur : en YAML, c’est `null`. Compose accepte cette forme pour dire « créer avec les options par défaut ».
- `internal: true` : les conteneurs de ce réseau ne peuvent pas joindre l’extérieur. Comme `web` est aussi sur `frontend`, il reste joignable via le port publié.
- **Communication entre services** : sur un même réseau, un service est joignable par son **nom** : depuis `web`, la base est atteignable à l’adresse `db`, port `5432`.

### 8.7 Commandes à utiliser

Toutes les commandes s’exécutent dans le dossier contenant `compose.yaml`.

| But | Commande | Explication |
|---|---|---|
| Valider et afficher la configuration finale | `docker compose config` | Vérifie le fichier (niveau 2 de la partie 7) et montre les variables substituées |
| Démarrer les services | `docker compose up -d` | `-d` (*detached*) : en arrière-plan |
| Vérifier leur état | `docker compose ps` | Liste les services, leur état et leurs ports |
| Afficher les journaux | `docker compose logs` | Journaux de tous les services |
| Suivre les journaux d’un service | `docker compose logs -f --tail=50 db` | `-f` : en continu ; `--tail=50` : 50 dernières lignes |
| Tester la page web | `curl http://localhost:8080` | Doit renvoyer le HTML de `index.html` |
| Tester la communication | `docker compose exec web ping -c 1 db` | Le nom `db` est résolu par le réseau Compose |
| Interroger PostgreSQL | `docker compose exec db sh -c 'psql -U "$POSTGRES_USER" -d "$POSTGRES_DB" -c "SELECT version();"'` | `psql` est le client PostgreSQL ; les variables sont lues dans le conteneur |
| Arrêter sans supprimer | `docker compose stop` | Les conteneurs sont arrêtés mais conservés |
| **Supprimer les conteneurs et réseaux, en conservant les données** | `docker compose down` | Les **volumes nommés ne sont pas supprimés** |
| Lister les volumes | `docker volume ls` | Doit montrer `tp-yaml_db_data` |

> **Danger.** `docker compose down -v` (ou `--volumes`) **supprime aussi les volumes nommés**, donc **les données de PostgreSQL**. À n’utiliser que si l’on veut volontairement repartir de zéro.

### 8.8 Limites de l’exemple

- Nginx et PostgreSQL ne sont **pas réellement reliés** par une application : aucun programme n’utilise la base. L’exemple illustre la syntaxe Compose, le réseau et la persistance.
- Aucun chiffrement (HTTPS) n’est configuré.
- Un seul serveur, sans sauvegarde automatique ni supervision : ce n’est pas une architecture de production.
- Le comportement dépend des images utilisées. Pour les images PostgreSQL **18 et suivantes**, l’emplacement recommandé du volume a changé : la documentation de l’image indique de monter le volume sur `/var/lib/postgresql` (et non `/var/lib/postgresql/data`). Notre exemple fixe la version **16** pour rester cohérent ; en changeant de version majeure, **relire la documentation de l’image**.
- Le changement de version majeure de PostgreSQL ne se fait pas en modifiant seulement l’étiquette de l’image : il nécessite une procédure de migration des données.

### 8.9 Bonnes pratiques de sécurité

- Ne jamais écrire de mots de passe dans `compose.yaml` ni les versionner ; utiliser `.env` ignoré par Git (ou un gestionnaire de secrets).
- Utiliser des mots de passe **longs et uniques**, jamais ceux de l’exemple.
- **Ne pas publier** le port de la base vers l’extérieur sans nécessité. Pour un débogage local, restreindre à la machine : `"127.0.0.1:5432:5432"`.
- Fixer des **versions précises** d’images plutôt que `latest`.
- Monter les fichiers en **lecture seule** (`:ro`) quand c’est possible.
- Pour aller plus loin : l’image PostgreSQL accepte `POSTGRES_PASSWORD_FILE` avec les **secrets** Docker, ce qui évite de passer le mot de passe par variable d’environnement.

### Points clés de la partie 8

- Un `compose.yaml` décrit des services, réseaux et volumes ; Docker Compose l’exécute.
- Les valeurs sensibles se placent dans `.env`, pas dans le YAML versionné.
- `docker compose down` conserve les volumes nommés ; `down -v` les supprime.
- Le YAML n’est qu’une description : images, ports libres et fichiers référencés doivent exister.

---

## Partie 9. Introduction à YAML dans Kubernetes

### 9.1 Kubernetes et manifestes

**Kubernetes** est un système d’**orchestration de conteneurs** : il déploie, surveille et met à l’échelle des conteneurs sur un ensemble de machines appelé **cluster**. On décrit l’**état souhaité** (par exemple « 3 copies de Nginx ») dans des fichiers YAML appelés **manifestes** ; Kubernetes travaille ensuite pour faire correspondre la réalité à cette description.

> **Prérequis et limites.** Les exemples de cette partie nécessitent un **cluster Kubernetes opérationnel** (minikube, kind, Docker Desktop avec Kubernetes, ou un cluster distant) et l’outil `kubectl`. Cette partie est une **introduction** : YAML ne remplace pas la connaissance de Kubernetes.

### 9.2 Les quatre champs de premier niveau

| Champ | Rôle |
|---|---|
| `apiVersion` | Version de l’API Kubernetes à laquelle appartient la ressource (`v1`, `apps/v1`…) |
| `kind` | Type de ressource (`Pod`, `Deployment`, `Service`…) |
| `metadata` | Identité : `name`, `labels` (étiquettes clé-valeur), `namespace` |
| `spec` | **Spécification** : l’état souhaité (son contenu dépend du `kind`) |

### 9.3 Pod, Deployment et Service

| Ressource | Rôle | Remarque |
|---|---|---|
| **Pod** | La plus petite unité déployable : un ou plusieurs conteneurs qui partagent réseau et stockage | Un Pod créé seul n’est pas recréé s’il disparaît |
| **Deployment** | Gère un ensemble de Pods identiques (réplicas), assure les mises à jour progressives et recrée les Pods défaillants | Utilisé pour les applications sans état |
| **Service** | Donne une adresse réseau **stable** et répartit le trafic vers des Pods choisis par leurs `labels` | Les Pods changent, le Service reste |

### 9.4 Un manifeste simple : un Pod

**[Exécutable avec un cluster]**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nginx
  labels:
    app: demo
spec:
  containers:
    - name: nginx
      image: nginx:1.27-alpine
      ports:
        - containerPort: 80
```

Explication :

- `apiVersion: v1` et `kind: Pod` : un Pod fait partie de l’API de base.
- `metadata.name` : nom du Pod ; `labels` : étiquette libre.
- `spec.containers` : **liste** (note : `- name:`) de conteneurs ; ici un seul, nommé `nginx`.
- `containerPort: 80` : information indiquant le port d’écoute du conteneur.

### 9.5 Un Deployment et un Service dans un même fichier

Le séparateur `---` (partie 3) permet de placer **deux ressources** dans le même fichier `k8s/web.yaml`.

**[Exécutable avec un cluster]**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  labels:
    app: web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx:1.27-alpine
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 50m
              memory: 64Mi
            limits:
              memory: 128Mi
---
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
```

Explication :

- `apps/v1` / `Deployment` : le type qui gère les réplicas.
- `replicas: 2` : on veut **2 Pods** identiques.
- `selector.matchLabels` : comment le Deployment retrouve ses Pods ; **doit correspondre** aux `labels` du `template`.
- `template` : le modèle de Pod (son propre `metadata` et `spec`).
- `resources` : `requests` = ressources réservées ; `limits` = plafond. `50m` signifie 50 millièmes de cœur, `64Mi` 64 mébioctets.
- Deuxième document : un `Service` de type `ClusterIP` (accessible à l’intérieur du cluster). Son `selector` (`app: web`) désigne les Pods à atteindre ; `port` est le port du Service, `targetPort` celui du conteneur.

### 9.6 La commande `kubectl apply -f`

```bash
kubectl apply -f k8s/web.yaml
```

Explication : `kubectl` envoie le contenu du fichier au cluster ; `apply` crée les ressources ou les met à jour pour qu’elles correspondent au fichier ; `-f` désigne le fichier. Commandes utiles :

| But | Commande |
|---|---|
| Vérifier la connexion au cluster | `kubectl cluster-info` |
| Tester sans rien créer (côté client) | `kubectl apply --dry-run=client -f k8s/web.yaml` |
| Tester avec validation par le serveur | `kubectl apply --dry-run=server -f k8s/web.yaml` |
| Voir les Pods, Deployments, Services | `kubectl get pods`, `kubectl get deployments`, `kubectl get svc` |
| Accéder au Service depuis la machine locale | `kubectl port-forward svc/web 8080:80` |
| Consulter les journaux d’un Pod | `kubectl logs <nom-du-pod>` |
| Supprimer les ressources du fichier | `kubectl delete -f k8s/web.yaml` |
| Obtenir la documentation d’un champ | `kubectl explain deployment.spec.replicas` |

### 9.7 Validité YAML et validité Kubernetes

Ce sont deux niveaux distincts (voir partie 7).

**[Incorrect]** Ce fichier est du YAML parfaitement valide :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replica: 2
  container:
    - name: nginx
      image: nginx:1.27-alpine
```

Mais Kubernetes le **rejette** : le champ s’appelle `replicas` (avec un `s`), `containers` (au pluriel) se trouve dans `spec.template.spec`, et les champs `selector` et `template` obligatoires sont absents. Selon la version de `kubectl`, les champs inconnus provoquent une erreur ou un avertissement.

Autre exemple d’erreur de schéma : des `labels` du `template` qui **ne correspondent pas** au `selector` du Deployment sont refusés par le cluster.

> **À retenir.** Un fichier syntaxiquement valide n’est pas une ressource Kubernetes valide. Et une ressource valide ne garantit pas que l’application fonctionne (image introuvable, ressources insuffisantes, etc.) : on vérifie avec `kubectl get pods` et `kubectl describe pod`.

### Points clés de la partie 9

- Un manifeste décrit l’état souhaité avec `apiVersion`, `kind`, `metadata`, `spec`.
- Pod = unité de base ; Deployment = gestion de réplicas ; Service = adresse stable.
- `kubectl apply -f` applique le manifeste à un cluster opérationnel.
- Validité YAML ≠ validité Kubernetes ≠ application fonctionnelle.

---

## Partie 10. Bonnes pratiques

1. **Indentation cohérente.** 2 espaces par niveau, jamais de tabulations ; l’éditeur est configuré en conséquence (partie 6).
2. **Noms explicites.** Des clés et des services parlants (`base_donnees`, `retention_jours`) ; une convention unique (`snake_case` ou `kebab-case`) dans un même projet, sauf si l’outil impose la sienne.
3. **Commentaires utiles.** Expliquer le **pourquoi** (un choix inhabituel, une contrainte) plutôt que paraphraser la ligne.
4. **Éviter la complexité excessive.** Limiter la profondeur des niveaux ; découper en plusieurs fichiers ; réserver ancres, alias et fusions aux vraies répétitions.
5. **Vérifier les types.** Mettre entre guillemets les valeurs ambiguës (`"NO"`, `"3.10"`, `"0755"`, `"22:22"`) ; utiliser `true` et `false`.
6. **Éviter les clés dupliquées.** Les détecter avec `yamllint`.
7. **Ne pas stocker de secrets dans des fichiers publics ou versionnés.** Utiliser des variables d’environnement, un fichier `.env` ignoré par Git, ou un gestionnaire de secrets. Si un secret a été publié par erreur, le **révoquer** et le remplacer : le supprimer du dépôt ne suffit pas, car l’historique Git le conserve.
8. **Contrôler avant déploiement.** Valider la syntaxe, puis avec l’outil cible (`docker compose config`, `kubectl apply --dry-run=server`), idéalement automatiquement en intégration continue.
9. **Utiliser des outils de validation et de formatage.** `yamllint`, extension YAML de l’éditeur, hooks de pré-commit.
10. **Documenter les configurations.** Un `README.md` qui explique les fichiers, les variables attendues et les commandes de démarrage ; un `.env.example` à jour.
11. **Tenir compte de la version de YAML prise en charge par l’outil cible.** Ne pas supposer que `yes`, `on`, `<<` ou les dates se comportent partout de la même façon ; consulter la documentation de l’outil.
12. **Fixer les versions.** Étiquettes d’images précises, actions CI avec une version, afin d’obtenir des déploiements reproductibles.

**Exemple de configuration `yamllint`** **[Exécutable]** (fichier `.yamllint` à la racine du projet) :

```yaml
extends: default

rules:
  line-length:
    max: 120
  document-start: disable
  truthy:
    allowed-values: ["true", "false"]
    check-keys: false
  indentation:
    spaces: 2
    indent-sequences: true
```

Explication : `extends: default` reprend les règles par défaut ; on y ajuste la longueur maximale des lignes, on n’exige pas `---` en début de fichier, on n’accepte que `true` et `false` comme booléens (sans contrôler les clés, utile pour `on:` des workflows GitHub), et on impose 2 espaces avec listes indentées.

**Exemple de contrôle automatique** **[Illustratif]** dans un workflow GitHub Actions :

```yaml
name: Validation YAML
on:
  pull_request:
jobs:
  yamllint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Installer yamllint
        run: pip install yamllint
      - name: Contrôler les fichiers
        run: yamllint .
```

### Points clés de la partie 10

- Cohérence, lisibilité et simplicité d’abord.
- Typage explicite pour les valeurs ambiguës.
- Pas de secrets dans les dépôts.
- Valider à chaque niveau : syntaxe, schéma, exécution.

---

## Partie 11. Exercices et travaux pratiques

**Mode d’emploi.** Les exercices sont classés par série, puis du plus facile au plus difficile (★ facile, ★★ intermédiaire, ★★★ avancé). Pour chaque exercice, tester votre réponse avec `yamllint` et en affichant ce que Python comprend (partie 7.4). Les corrigés sont dans l’**annexe A**.

### Série A : paires clé-valeur et types (5 exercices)

**Exercice 1 (★) : fiche d’étudiant**

*Énoncé.* Écrire un fichier `etudiant.yaml` contenant : le nom `Salma Benali`, l’âge `20`, la filière `Informatique`, l’inscription en cours (`oui`, sous forme de booléen) et la moyenne `14.5`.
*Résultat attendu.* Cinq paires clé-valeur ; types : chaîne, entier, chaîne, booléen, décimal.

**Exercice 2 (★) : configuration d’un serveur**

*Énoncé.* Écrire une configuration contenant : l’hôte `localhost`, le port `3306`, un mot de passe **non défini** (valeur nulle), et ajouter un commentaire avant chaque clé expliquant son rôle.
*Résultat attendu.* Trois paires, trois commentaires, une valeur nulle.

**Exercice 3 (★★) : guillemets**

*Énoncé.* Écrire en YAML valide, sous les clés indiquées, les valeurs suivantes (chaque valeur doit être lue comme une **chaîne**) :

| Clé | Valeur à représenter |
|---|---|
| `titre` | `Atelier : Introduction au Cloud` |
| `chemin` | `C:\Cours\TP1` |
| `phrase` | `l'étudiant a dit "salut"` |
| `couleur` | `#1E90FF` |

*Résultat attendu.* Quatre chaînes exactement égales aux valeurs du tableau.

**Exercice 4 (★★) : prédire les types**

*Données de départ.* Fichier `types.yaml` :

```yaml
a: 42
b: "42"
c: 4.0
d: yes
e: ~
f: 3.10
g: "true"
h: 0755
i: 2026-10-28
j: ''
```

*Énoncé.* Pour chaque clé, indiquer le type et la valeur obtenus **avec PyYAML** (YAML 1.1), puis indiquer les clés dont l’interprétation changerait avec le schéma de base de YAML 1.2.
*Résultat attendu.* Un tableau à 10 lignes (clé, type, valeur) et la liste des clés qui diffèrent.

**Exercice 5 (★★★) : vérifier avec Python**

*Énoncé.* Écrire un script Python `types.py` qui charge `types.yaml` (exercice 4) avec `yaml.safe_load` et affiche, pour chaque clé, la valeur et son type. Comparer avec vos prédictions.
*Résultat attendu.* Une sortie de 10 lignes et la liste des prédictions erronées éventuelles.

### Série B : listes et objets imbriqués (5 exercices)

**Exercice 6 (★) : liste simple**

*Énoncé.* Écrire la liste des outils DevOps `Docker`, `Git`, `Ansible`, `Terraform`, d’abord en style bloc (clé `outils`), puis en style flux (clé `outils_flux`).
*Résultat attendu.* Deux clés représentant la même liste de 4 chaînes.

**Exercice 7 (★) : objet imbriqué**

*Énoncé.* Décrire un étudiant : `nom` = `Omar`, `adresse` (objet avec `ville` = `Settat` et `code_postal` = `26000`, à lire comme **chaîne**), `notes` (liste `12`, `15.5`, `14`).
*Résultat attendu.* Un objet avec un sous-objet et une liste de nombres.

**Exercice 8 (★★) : liste d’objets**

*Énoncé.* Décrire trois conteneurs sous la clé `conteneurs`. Chaque conteneur a `nom`, `image`, `port` :

| nom | image | port |
|---|---|---|
| web | `nginx:stable-alpine` | 80 |
| db | `postgres:16-alpine` | 5432 |
| cache | `redis:7-alpine` | 6379 |

*Résultat attendu.* Une liste de trois mappings ; `port` est un entier.

**Exercice 9 (★★) : listes imbriquées**

*Énoncé.* Décrire un planning sous la clé `planning` : un objet dont les clés sont `lundi`, `mardi`, `mercredi`. Chaque jour est une **liste** de créneaux ; chaque créneau est un objet avec `heure` (chaîne, par exemple `"08:30"`) et `matiere`. Lundi : `08:30` Cloud, `10:30` DevOps. Mardi : `14:00` Réseaux. Mercredi : **aucun** créneau (liste vide).
*Résultat attendu.* Trois jours, dont un avec la liste vide `[]`.

**Exercice 10 (★★★) : structure complexe**

*Énoncé.* Écrire un fichier `infra.yaml` décrivant :

- un projet `tp-cloud` situé dans la région `eu-west-3` ;
- deux réseaux : `public` (CIDR `10.0.1.0/24`) et `prive` (CIDR `10.0.2.0/24`) ;
- deux machines virtuelles : `web-01` (2 CPU, réseau `public`, disques : `systeme` de 20 Go) et `db-01` (4 CPU, réseau `prive`, disques : `systeme` de 20 Go et `donnees` de 100 Go).

Ensuite, écrire l’expression Python qui donne la taille du disque `donnees` de `db-01`.
*Résultat attendu.* Un fichier valide à trois niveaux et une expression d’accès.

### Série C : correction de fichiers erronés (5 exercices)

**Exercice 11 (★) : indentation**

*Énoncé.* Corriger le fichier suivant (attention : la ligne `nom` commence par une **tabulation**, invisible ici) :

```yaml
serveur:
	nom: web-01
   port: 8080
  actif: true
```

*Résultat attendu.* Un fichier valide, indenté de 2 espaces, sans tabulation.

**Exercice 12 (★★) : espaces manquants**

*Énoncé.* Corriger et expliquer chaque erreur :

```yaml
application:
  nom:Portail
  langages:
    -Python
    -Java
```

*Résultat attendu.* Un fichier valide où `langages` est une liste de deux éléments ; une explication de chaque correction.

**Exercice 13 (★★) : caractères spéciaux**

*Énoncé.* Corriger le fichier pour que toutes les valeurs soient lues comme des chaînes :

```yaml
message: Erreur: disque plein
mot_de_passe: @dmin2026
filtre: *.log
```

*Résultat attendu.* Un fichier valide avec trois chaînes.

**Exercice 14 (★★) : clé dupliquée**

*Énoncé.* Ce fichier est un `compose.yaml` partiel. Trouver le problème et le corriger **sans supprimer de service** :

```yaml
services:
  web:
    image: nginx:stable-alpine
    ports:
      - "8080:80"
  db:
    image: postgres:16-alpine
  web:
    image: httpd:2.4
```

*Résultat attendu.* Un fichier avec trois services aux noms uniques.

**Exercice 15 (★★★) : types implicites**

*Énoncé.* Ce fichier est valide, mais plusieurs valeurs ne sont pas celles que l’on croit avec PyYAML. Les identifier et corriger pour que **toutes** les valeurs soient des chaînes :

```yaml
projet:
  version: 1.10
  pays: NO
  couleur: #1E90FF
  permissions: 0755
  heure_sauvegarde: 23:30
```

*Résultat attendu.* Cinq chaînes ; pour chaque valeur, indiquer ce que le parseur lisait avant correction.

### Série D : conversion YAML et JSON (3 exercices)

**Exercice 16 (★) : YAML vers JSON**

*Énoncé.* Convertir en JSON :

```yaml
projet: cloud-tp
equipe:
  - nom: Amine
    role: dev
  - nom: Nadia
    role: ops
public: false
budget: null
```

*Résultat attendu.* Un document JSON valide, équivalent.

**Exercice 17 (★★) : JSON vers YAML**

*Énoncé.* Convertir en YAML, puis ajouter un commentaire expliquant la clé `delai` :

```json
{
  "serveur": {"hote": "10.0.0.5", "port": 22, "options": ["compression", "keepalive"]},
  "actif": true,
  "delai": 2.5
}
```

*Résultat attendu.* Un fichier YAML équivalent avec un commentaire (impossible en JSON).

**Exercice 18 (★★★) : ancres et conversion**

*Énoncé.* Convertir en JSON le fichier ci-dessous (en supposant un parseur qui gère `<<`), puis indiquer **ce qui est perdu** dans la conversion :

```yaml
defauts: &defauts
  cpu: 2
  ram_go: 4
vm1:
  <<: *defauts
  nom: web
vm2:
  <<: *defauts
  nom: db
  ram_go: 8
```

*Résultat attendu.* Un JSON avec trois objets (`defauts`, `vm1`, `vm2`) et une explication des pertes.

### Série E : chaînes multilignes, ancres et alias (3 exercices)

**Exercice 19 (★) : `|` et `|-`**

*Énoncé.* **(a)** Donner la valeur exacte (avec `\n`) de chaque clé :

```yaml
script: |
  echo "a"
  echo "b"
sans_fin: |-
  echo "a"
  echo "b"
```

**(b)** Écrire une clé `commande` dont la valeur est exactement `docker compose up -d` **sans retour à la ligne final**, avec un bloc multiligne.
*Résultat attendu.* Deux chaînes (a) et une écriture YAML (b).

**Exercice 20 (★★) : `>` et `>-`**

*Énoncé.* Donner la valeur exacte de chaque clé :

```yaml
texte: >
  Le cloud
  computing

  est une
  approche.
court: >-
  Un seul
  paragraphe
```

*Résultat attendu.* Deux chaînes (avec `\n` explicites).

**Exercice 21 (★★★) : ancres et fusion**

*Énoncé.* Décrire trois environnements `dev`, `test` et `prod` sous la clé `environnements`. Tous ont `region: eu-west-3`, `replicas: 1` et un objet `logging` avec `niveau: info`. Seul `prod` diffère : `replicas: 3` et `logging.niveau: warning`. Utiliser **une ancre**, **une fusion** et écrire la configuration sans répéter le bloc commun. Signaler la limite de portabilité de la fusion.
*Résultat attendu.* Un fichier avec une seule définition du bloc commun ; une remarque sur la fusion (superficielle) et sur sa portabilité.

### Série F : configurations inspirées de situations réelles (3 exercices)

**Exercice 22 (★★) : workflow CI**

*Énoncé.* Écrire un workflow GitHub Actions `ci.yml` qui : se déclenche sur chaque `push` vers `main` ; contient un job `tests` sur `ubuntu-latest` ; récupère le code (`actions/checkout@v4`), installe Python `3.12` (`actions/setup-python@v5`), installe les dépendances avec `pip install -r requirements.txt` puis lance `python -m pytest`.
*Résultat attendu.* Un fichier valide en YAML ; il n’est pas nécessaire de l’exécuter (il faudrait un dépôt GitHub).

**Exercice 23 (★★) : playbook Ansible**

*Énoncé.* Écrire un playbook qui, sur le groupe d’hôtes `web`, avec les droits administrateur, installe les paquets `git` et `nginx` (module `ansible.builtin.apt`, `update_cache: true`), puis démarre et active le service `nginx` (module `ansible.builtin.service`).
*Résultat attendu.* Un playbook valide en YAML (exécutable seulement avec Ansible et des machines Debian/Ubuntu accessibles).

**Exercice 24 (★★★) : fichier de supervision**

*Énoncé.* Écrire un fichier `supervision.yaml` décrivant : un `intervalle_defaut_secondes` de `60` ; une liste `sondes` de trois sondes (`nom`, `url`, `delai_max_secondes`) : `site` (`https://exemple.ma`, 5), `api` (`https://exemple.ma/api/sante`, 3), `base` (`tcp://db.exemple.local:5432`, 2) ; une liste `notifications.emails` de deux adresses ; un champ `message_alerte` sur plusieurs lignes (texte replié avec `>`).
*Résultat attendu.* Un fichier valide ; toutes les URL sont des chaînes ; `message_alerte` forme un seul paragraphe.

### Série G : travaux pratiques Docker Compose (2 TP)

**TP 1 (★★) : serveur web statique**

*Prérequis.* Docker avec Compose v2 ; port `8080` libre.
*Énoncé.*

1. Créer `tp1/site/index.html` avec un titre de votre choix.
2. Écrire `tp1/compose.yaml` avec un service `web` (image `nginx:stable-alpine`, port `8080:80`, dossier `site` monté en lecture seule).
3. Valider avec `docker compose config`, démarrer, vérifier l’état, tester avec `curl`.
4. Modifier `index.html` **sans redémarrer** et vérifier que la page change.
5. Provoquer volontairement une erreur (remplacer `ports` par `prots`), observer le message de `docker compose config`, puis corriger.
6. Arrêter et supprimer les conteneurs.

*Résultat attendu.* Une page accessible sur `http://localhost:8080`, un message d’erreur observé et expliqué (niveau 2 de la partie 7).

**TP 2 (★★★) : Nginx, PostgreSQL et persistance**

*Prérequis.* TP 1 réussi.
*Énoncé.*

1. Créer `tp2/` avec `compose.yaml`, `.env`, `.env.example`, `.gitignore`, `site/index.html`.
2. Écrire le `compose.yaml` de la partie 8 **en le réécrivant sans le copier** (service `web`, service `db`, variables dans `.env`, volume `db_data`, réseaux, `healthcheck`).
3. Démarrer et vérifier : `docker compose ps` doit montrer `db` en bonne santé (*healthy*).
4. Se connecter à PostgreSQL et créer une table `notes` contenant une ligne.
5. Exécuter `docker compose down`, puis `docker compose up -d` : vérifier que la table existe encore.
6. Supprimer volontairement le `POSTGRES_PASSWORD` de `.env` et observer l’erreur de `docker compose config`. Le remettre.
7. Exécuter `docker compose down -v` et constater que les données ont disparu.

*Résultat attendu.* Données conservées après l’étape 5, perdues après l’étape 7 ; un court rapport expliquant la différence entre `down` et `down -v`.

### Série H : introduction à Kubernetes (1 exercice)

**Exercice 27 (★★★)**

*Prérequis.* Cluster local opérationnel (minikube, kind ou Docker Desktop) et `kubectl`.
*Énoncé.* Écrire `k8s/web.yaml` contenant un Deployment `web` (3 réplicas, image `nginx:1.27-alpine`, port conteneur 80) et un Service `web` de type `ClusterIP`. Puis :

1. Valider avec `kubectl apply --dry-run=server -f k8s/web.yaml`, puis appliquer.
2. Vérifier avec `kubectl get deployments,pods,svc`.
3. Modifier `app: web` en `app: webb` **uniquement** dans le `selector` du Deployment et observer l’erreur ; corriger.
4. Passer à 2 réplicas dans le fichier, réappliquer, et vérifier le résultat.
5. Supprimer les ressources.

*Résultat attendu.* 3 puis 2 Pods en état `Running` ; une erreur de cohérence `selector`/`labels` observée ; explication de la différence entre fichier YAML valide et ressource Kubernetes valide.

---

## Partie 12. Synthèse et évaluation

### 12.1 Résumé des notions essentielles

- **YAML** est un format de données lisible, pas un langage de programmation ni un langage de balisage de documents. Il décrit **quoi** configurer ; l’outil qui lit le fichier décide **comment** l’utiliser.
- La **structure** repose sur l’**indentation** (espaces uniquement, 2 par niveau par convention).
- Trois familles de données : **scalaires**, **listes** (`- `), **mappings** (`clé: valeur`). On les emboîte librement.
- Le **typage est implicite** : `NO`, `1.10`, `0755`, `22:22` peuvent surprendre. Les guillemets forcent une chaîne.
- Fonctions avancées : chaînes multilignes (`|`, `>`, `-`, `+`), ancres et alias (`&`, `*`), fusion `<<` (historique, non portable de manière identique), documents multiples (`---`, `...`).
- **YAML et JSON** représentent des structures proches ; la conversion fait perdre commentaires et ancres.
- Trois niveaux de validité : **syntaxe**, **schéma de l’outil**, **exécution réelle**.
- Outils : éditeur configuré, `yamllint`, `yaml.safe_load` en Python, commandes de validation de l’outil cible.
- **Docker Compose** : `services`, `networks`, `volumes`, variables d’environnement, `docker compose down` (conserve les volumes) et `down -v` (supprime les volumes).
- **Kubernetes** : `apiVersion`, `kind`, `metadata`, `spec` ; Pod, Deployment, Service ; `kubectl apply -f`.
- **Sécurité** : jamais de secrets dans un dépôt ; versions d’images fixées ; ports non publiés inutilement.

### 12.2 Tableau récapitulatif de la syntaxe YAML

| Élément | Syntaxe | Exemple |
|---|---|---|
| Paire clé-valeur | `clé: valeur` | `port: 8080` |
| Commentaire | `# texte` | `# Port du serveur` |
| Chaîne | sans guillemets, `'…'` ou `"…"` | `nom: "Alice"` |
| Entier / décimal | chiffres, point décimal | `age: 20`, `pi: 3.14` |
| Booléen | `true` / `false` | `actif: true` |
| Nul | `null`, `~` ou vide | `tel: null` |
| Liste (bloc) | `- élément` | `- Docker` |
| Liste (flux) | `[a, b, c]` | `[80, 443]` |
| Liste vide | `[]` | `taches: []` |
| Mapping imbriqué | indentation | `db:` puis `  hote: x` |
| Mapping (flux) | `{clé: valeur}` | `{role: web}` |
| Mapping vide | `{}` | `options: {}` |
| Chaîne littérale | `\|` | conserve les retours à la ligne |
| Chaîne repliée | `>` | remplace les retours simples par des espaces |
| Retour final | `\|-`, `\|+`, `>-`, `>+` | supprime / conserve les retours finaux |
| Ancre | `&nom` | `base: &base` |
| Alias | `*nom` | `copie: *base` |
| Fusion (historique) | `<<: *nom` | `<<: *base` |
| Début / fin de document | `---` / `...` | documents multiples |
| Clé complexe | `? clé` puis `: valeur` | rarement utile |

### 12.3 Glossaire

| Terme | Définition |
|---|---|
| **Alias** | Référence (`*nom`) à une valeur marquée par une ancre. |
| **Ancre** | Marqueur (`&nom`) placé sur une valeur pour la réutiliser. |
| **CI/CD** | Intégration continue et déploiement continu : automatisation des tests et des livraisons. |
| **Cluster** | Ensemble de machines gérées ensemble (Kubernetes). |
| **Conteneur** | Environnement isolé et léger qui exécute une application. |
| **Deployment** | Ressource Kubernetes qui gère des réplicas de Pods. |
| **DevOps** | Démarche qui rapproche développement et exploitation, avec beaucoup d’automatisation. |
| **Docker Compose** | Outil qui lit un fichier YAML pour exécuter plusieurs conteneurs. |
| **Document (YAML)** | Un ensemble de données complet dans un flux ; un fichier peut en contenir plusieurs. |
| **Image** | Modèle à partir duquel un conteneur est créé. |
| **Indentation** | Décalage des lignes par des espaces ; indique la structure en YAML. |
| **Infrastructure as code** | Description de l’infrastructure par des fichiers versionnés. |
| **JSON** | Format de données à accolades et crochets, très utilisé dans les API. |
| **Linter** | Outil qui analyse un fichier pour signaler erreurs et mauvaises pratiques. |
| **Liste (séquence)** | Suite ordonnée d’éléments (`- a`). |
| **Manifeste** | Fichier décrivant l’état souhaité de ressources (Kubernetes). |
| **Mapping (objet)** | Ensemble de paires clé-valeur. |
| **Parseur** | Programme qui lit un texte et construit des données structurées. |
| **Pod** | Plus petite unité déployable de Kubernetes. |
| **Scalaire** | Valeur simple : chaîne, nombre, booléen, nul. |
| **Schéma** | Description formelle de la structure et des types attendus. |
| **Sérialisation** | Représentation de données sous forme de texte ou d’octets. |
| **Service (Compose)** | Définition d’un conteneur dans un `compose.yaml`. |
| **Service (Kubernetes)** | Adresse réseau stable donnant accès à un ensemble de Pods. |
| **Typage implicite** | Détermination du type d’une valeur à partir de son écriture. |
| **Volume** | Espace de stockage qui survit à la suppression d’un conteneur. |
| **YAML 1.1 / 1.2** | Versions de la spécification ; elles diffèrent sur certains types (booléens, octal…). |

### 12.4 Questionnaire à choix multiples (15 questions)

*Une seule réponse correcte par question. Réponses justifiées dans l’annexe B.*

**Question 1.** Que signifie YAML ?
- A. *Yet Another Markup Language*, uniquement
- B. *YAML Ain’t Markup Language*
- C. *Yielding Advanced Markup Language*
- D. *Your Application Markup Language*

**Question 2.** Quelle proposition décrit le mieux YAML ?
- A. Un langage de programmation compilé
- B. Un langage de balisage pour pages web
- C. Un format de sérialisation de données lisible par l’humain
- D. Un système de gestion de bases de données

**Question 3.** Quel caractère est interdit pour l’indentation en YAML ?
- A. L’espace
- B. Le saut de ligne
- C. Le point-virgule
- D. La tabulation

**Question 4.** Quelle ligne est un commentaire YAML valide ?
- A. `// commentaire`
- B. `/* commentaire */`
- C. `# commentaire`
- D. `<!-- commentaire -->`

**Question 5.** Quelle ligne est une paire clé-valeur correcte ?
- A. `port = 8080`
- B. `port -> 8080`
- C. `port:8080`
- D. `port: 8080`

**Question 6.** Quelle écriture est équivalente à la liste suivante ?

```yaml
- web
- db
```

- A. `[web, db]`
- B. `(web, db)`
- C. `<web, db>`
- D. `web: db`

**Question 7.** Que fait l’indicateur `>` dans `texte: >` ?
- A. Il conserve tous les retours à la ligne
- B. Il remplace les retours à la ligne simples par des espaces
- C. Il supprime les commentaires
- D. Il transforme le texte en liste

**Question 8.** Quel est l’effet de `|-` ?
- A. Il conserve les retours à la ligne internes et supprime le retour final
- B. Il conserve toutes les lignes vides finales
- C. Il transforme le texte en liste
- D. Il est interdit en YAML

**Question 9.** À quoi servent `&` et `*` ?
- A. `&` fait référence à une valeur et `*` la définit
- B. Les deux servent à écrire des commentaires
- C. `&` définit une ancre et `*` y fait référence
- D. `&` fusionne deux fichiers et `*` supprime une clé

**Question 10.** Que peut-on dire de la clé de fusion `<<` ?
- A. C’est un élément standard de YAML 1.2, géré à l’identique partout
- B. Elle sert à fusionner deux fichiers
- C. Elle introduit un commentaire
- D. C’est une fonctionnalité historique (YAML 1.1) prise en charge par de nombreux outils, mais non portable de manière identique

**Question 11.** Avec un parseur YAML 1.1 comme PyYAML, que vaut `pays: NO` ?
- A. La chaîne `"NO"`
- B. Le booléen `False`
- C. La valeur nulle
- D. Une erreur de syntaxe

**Question 12.** Un manifeste Kubernetes syntaxiquement valide en YAML…
- A. est forcément accepté par `kubectl`
- B. respecte forcément le schéma Kubernetes
- C. ne garantit pas la conformité au schéma Kubernetes
- D. est forcément valide pour JSON Schema

**Question 13.** Quelle commande supprime les conteneurs et réseaux d’un projet Compose en **conservant** les volumes nommés ?
- A. `docker compose down -v`
- B. `docker system prune -a --volumes`
- C. `docker volume prune`
- D. `docker compose down`

**Question 14.** Dans un manifeste Kubernetes, que décrit le champ `spec` ?
- A. L’état souhaité de la ressource
- B. La version de l’API
- C. Le nom et les étiquettes de la ressource
- D. Le type de la ressource

**Question 15.** Quelle affirmation sur YAML et JSON est exacte ?
- A. JSON autorise les commentaires
- B. YAML permet les commentaires, alors que JSON standard ne le permet pas
- C. YAML interdit les listes imbriquées
- D. YAML et JSON sont interchangeables sans aucune conversion

### 12.5 Questions d’analyse de code

**Analyse 1.** Donner la structure obtenue (équivalent JSON) pour :

```yaml
a: &x
  - 1
  - 2
b: *x
c: [*x, 3]
```

**Analyse 2.** Donner la valeur exacte (avec `\n` si nécessaire) de `msg` :

```yaml
msg: >-
  Bonjour
  tout le monde

  Fin
```

**Analyse 3.** Ce fichier est incorrect. Expliquer pourquoi et le corriger :

```yaml
services:
 web:
    image: nginx:stable-alpine
   ports:
    - 8080:80
```

**Analyse 4.** On charge le fichier suivant en Python avec `data = yaml.safe_load(...)`.

```yaml
cluster:
  noeuds:
    - nom: n1
      cpu: 4
    - nom: n2
      cpu: 8
```

(a) Quelle est la valeur de `data["cluster"]["noeuds"][1]["cpu"]` ? (b) Quel est le type de `data["cluster"]["noeuds"]` ? (c) Écrire l’expression qui donne la somme des CPU de tous les nœuds.

### 12.6 Exercice de synthèse

**Partie 1 : écrire.** Écrire un fichier `plateforme.yaml` qui contient :

- une clé `projet` avec : `nom` = `Plateforme TP Cloud`, `version` = `2.0` (**chaîne**), `actif` = vrai, `accueil` = un message de deux lignes à conserver tel quel (`Bienvenue sur la plateforme de TP.` puis `Merci de respecter la charte d'utilisation.`) ;
- une liste `responsables` de deux objets (`nom`, `email`) : `Sara` (`sara@exemple.ma`) et `Omar` (`omar@exemple.ma`) ;
- un bloc commun `commun` (`region: eu-west-3`, `replicas: 1`, `debug: true`, `logging` avec `niveau: info`) ;
- une clé `environnements` avec `dev` et `prod` qui **réutilisent** `commun` par ancre et fusion ; `prod` a `replicas: 3` et `debug: false`.

**Partie 2 : corriger.** Corriger le fichier suivant (la ligne `nom` de `projet` commence par une **tabulation**) ; indiquer chaque erreur, sa cause, et si elle est silencieuse ou bloquante. L’objectif est que `version` soit une chaîne, `actif` un booléen vrai, `couleur` une chaîne, et que la liste d’équipe soit valide.

```yaml
projet:
	nom: Plateforme TP
  version: 2.0
  actif: Oui
  equipe:
  - nom: Sara
    email: sara@exemple.ma
  - nom: Omar
   email: omar@exemple.ma
  note: Attention: dépend de 'Docker'
  couleur: #00AAFF
```

**Partie 3 : vérifier.** Écrire la commande `yamllint` et le script Python (3 à 5 lignes) permettant de contrôler que votre fichier corrigé est valide et de lire `projet.version`.

*Résultat attendu.* Deux fichiers valides, une liste d’erreurs justifiées, une commande et un script fonctionnels.

