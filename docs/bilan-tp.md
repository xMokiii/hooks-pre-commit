# Bilan du TP — NOVA SERVICES

Ce document répond aux sections 10 à 14. La politique et les seuils ci-dessous
sont des propositions pour NOVA SERVICES, à faire valider par les responsables.
Ils ne signifient pas que les réglages distants sont déjà activés.

## 10. Adapter les contrôles aux dépôts

### 1. Contrôles communs

Tous les dépôts doivent versionner la configuration des hooks, détecter les
secrets, vérifier les espaces et fins de fichiers, la syntaxe des configurations
et la taille des fichiers ajoutés. Ils doivent aussi disposer de tests adaptés,
d'un audit des dépendances, d'une revue de code et d'une CI obligatoire avant fusion.

| Dépôt | Socle commun | Compléments adaptés au risque |
| --- | --- | --- |
| API Python | Hooks de qualité, secrets, tests et CI | Bandit, pytest, pip-audit ; tests CI des autorisations, de l'authentification et des entrées de l'API |
| Outil d'administration | Même socle | Analyse adaptée au langage ; tests des droits, commandes dangereuses et modes de simulation ; aucun identifiant de production dans les tests |
| Scripts d'automatisation | Même socle | Validation des entrées, gestion des erreurs, tests d'idempotence ; analyse shell ou PowerShell selon le langage ; tests dans un environnement isolé |

Les outils Python s'appliquent aussi à l'outil d'administration et aux scripts
s'ils sont écrits en Python. Les opérations à privilèges ne sont pas testées sur
des ressources de production.

### 2. Contrôles spécifiques à Python

Bandit analyse les usages Python risqués, pytest vérifie les comportements et
pip-audit recherche les vulnérabilités connues des dépendances déclarées.
Les dépendances applicatives et de développement doivent être recensées ; leurs
versions doivent être verrouillées pour reproduire l'environnement d'équipe.

Dans ce TP, `pip-audit` ne cible que `requirements.txt`. L'audit de
`requirements-dev.txt` et le verrouillage des versions sont des extensions à
prévoir pour les dépôts de NOVA SERVICES, pas des contrôles déjà en place ici.

### 3. Contrôles bloquants avant le commit

Le socle local bloque sur un secret suspect non qualifié, un YAML invalide,
un fichier trop volumineux sans justification, un test rapide en échec ou un
résultat de sécurité atteignant le seuil défini. Une correction automatique
du formatage nécessite une vérification puis un nouvel ajout à l'index Git.

Dans ce dépôt, tous les hooks configurés sont bloquants : qualité, Bandit avec
`-ll` (sévérité moyenne ou élevée), detect-secrets, pytest et pip-audit.
Un échec technique du scanner n'est pas une validation de sécurité.

### 4. Contrôles réservés à la CI

Les suites longues, tests d'intégration, tests sur plusieurs versions de Python,
analyses de conteneurs et contrôles de sécurité dynamiques peuvent rester en CI.
Un audit réseau des dépendances peut aussi être réservé à la CI si sa durée ou
sa disponibilité pénalise les commits ; ce choix doit être documenté.
Dans le TP actuel, cet audit reste exécuté localement et dans GitHub Actions.
La CI reprend également les contrôles rapides, même s'ils ont réussi localement.

### 5. Exceptions à detect-secrets

Chaque détection doit être examinée avant de modifier la baseline ou d'ajouter
une autorisation en ligne. Un faux positif doit être limité à l'occurrence
concernée et accompagné d'un ticket expliquant le contexte, la justification,
le validateur et la date de réexamen. Toute modification de la baseline est revue
dans la pull request ; il est interdit de régénérer la baseline sans examen
ou d'exclure globalement des répertoires pour faire passer les contrôles.

Un véritable secret doit être retiré et, s'il a été exposé, révoqué ou renouvelé.
Il ne devient pas acceptable parce qu'il figure dans une baseline. Les preuves
de traitement ne doivent pas reproduire sa valeur.

### 6. Responsabilité des mises à jour

Le référent DevSecOps propose les mises à jour mensuelles des hooks et outils.
Le mainteneur de chaque dépôt vérifie les changements et les tests ; le référent
sécurité valide les changements de règles et de seuils. Une alerte urgente sur
un outil déclenche une mise à jour hors calendrier. Les mises à jour proposées
par un robot restent soumises à revue.

### 7. Preuve de correction d'une vulnérabilité

Relier le ticket au commit et à la pull request, conserver le résultat initial,
le changement apporté, un test de non-régression pertinent et le rapport du
scanner après correction. La preuve finale doit identifier le commit vérifié
par la CI et la validation du relecteur. Une capture sans version du code ni
commande reproductible ne suffit pas. Pour un secret exposé, joindre la preuve
de révocation sans divulguer le secret.

### 8. Dépendance vulnérable sans correctif

Qualifier l'exposition réelle, rechercher une version non affectée, retirer ou
remplacer la dépendance, ou désactiver la fonctionnalité exposée. Si le composant
doit être conservé temporairement, définir des mesures compensatoires et ouvrir
une exception avec responsable, échéance et suivi du correctif amont.
Le référent sécurité et le responsable métier approuvent le risque résiduel.
Une vulnérabilité critique exploitable sans mesure efficace bloque la livraison.
Une exclusion d'audit permanente et non justifiée n'est pas une solution.

## Politique courte de sécurisation du code proposée

- **Obligations :** configuration des hooks versionnée, recherche de secrets,
  validation des configurations, analyse adaptée au langage, tests et audit des
  dépendances. Aucun secret dans les fichiers versionnés ; `.env` reste local et
  `.env.example` ne contient que les noms de variables et des valeurs non secrètes.
- **Local et CI :** exécuter les contrôles rapides avant commit. GitHub Actions
  relance tous les hooks sur les fichiers versionnés, tests et audit inclus.
  Les contrôles longs et les environnements multiples sont pris en charge en CI.
- **Blocage :** un échec local empêche le commit normal. Toute CI obligatoire en
  échec ou non terminée interdit la fusion. Les secrets non traités et les risques
  critiques sans protection efficace interdisent la livraison. Une panne de
  scanner doit être résolue ou faire l'objet d'une dérogation explicite et limitée.
- **Branche principale :** imposer côté serveur une pull request, au moins une
  approbation indépendante, les contrôles CI réussis et l'absence de contournement
  par les administrateurs. La configuration locale seule ne suffit pas.
- **Faux positifs :** justifier chaque occurrence, limiter la portée de
  l'autorisation et conserver le ticket ainsi que la revue de la modification.
- **Exceptions :** validation par le référent sécurité, avec accord du
  responsable métier pour une acceptation de risque. Le mainteneur applique la
  décision. Documenter le périmètre, les mesures compensatoires, le propriétaire
  et une expiration proposée de 30 jours maximum, sans renouvellement automatique.
  Le développeur concerné ne valide pas seul sa propre exception.
- **Mises à jour :** chaque mois, le référent DevSecOps lance
  `pre-commit autoupdate` dans une branche dédiée, examine les notes de version,
  actualise les dépendances et références des actions, puis ouvre une pull request.
  Les contrôles complets et une revue précèdent la fusion. En production, épingler
  les actions par SHA validé et conserver des versions de dépendances reproductibles.

## 11. Bonnes pratiques et limites

Documenter l'installation, vérifier les changements des hooks et conserver les
résultats des analyses permet de reproduire les contrôles. Le scanner ne détecte
pas tous les secrets et Bandit ne prouve pas l'absence de vulnérabilité.
Un audit sans résultat ne couvre que les dépendances analysées et les
vulnérabilités connues à sa date d'exécution.

Les hooks restent contournables. Ils complètent la revue humaine, les tests
fonctionnels et de sécurité, la gestion des secrets de production et les règles
serveur. Les scanners de conteneurs et analyses approfondies doivent être ajoutés
selon les composants réellement utilisés.

## 12. Trois indicateurs de suivi proposés

Les seuils sont des objectifs initiaux à valider, puis à ajuster selon le risque.

| Indicateur et calcul | Source | Propriétaire | Fréquence | Seuil d'alerte proposé | Action |
| --- | --- | --- | --- | --- | --- |
| Taux de dépôts configurés : dépôts actifs avec hooks versionnés et CI obligatoire / dépôts actifs du périmètre × 100 | Inventaire des dépôts, configurations et règles serveur | Référent DevSecOps | Hebdomadaire | Moins de 100 % des dépôts du périmètre | Affecter un responsable et un délai de mise en conformité ; traiter les dépôts sensibles en priorité |
| Délai médian de correction : médiane de la durée entre première détection qualifiée et vérification de correction des vulnérabilités closes sur 30 jours | Tickets horodatés, commits et rapports CI | Référent sécurité | Hebdomadaire, sur fenêtre glissante de 30 jours | Médiane supérieure à 7 jours ; alerte immédiate pour une vulnérabilité critique ouverte depuis plus de 24 h | Prioriser les corrections avec l'équipe ; examiner aussi l'âge des tickets ouverts pour éviter un indicateur trompeur |
| Nombre d'exceptions actives : dérogations approuvées non closes et non expirées à la date du relevé | Registre des exceptions et tickets liés | Référent sécurité | Hebdomadaire ; contrôle quotidien des expirations | Plus de 3 exceptions actives par dépôt, ou au moins une dérogation expirée encore appliquée | Réexaminer les justifications ; retirer les dérogations expirées et remettre les contrôles en blocage |

Une absence de vulnérabilités closes rend la médiane non calculable : indiquer
« non disponible », jamais zéro. Ventiler les résultats par dépôt et gravité,
afficher le nombre de tickets concernés et suivre séparément les cas ouverts.
Une dérogation expirée sort du nombre d'exceptions actives mais déclenche son
alerte propre. L'augmentation des détections peut venir d'une meilleure couverture ;
elle ne prouve pas à elle seule une dégradation.

## 13. Réponses aux questions de compréhension

1. **Hook pre-commit :** un contrôle lancé avant la création d'un commit ; avec
   le framework pre-commit, plusieurs outils sont orchestrés depuis un même fichier.
2. **Local et CI :** le contrôle local donne un retour rapide sur le poste ; la CI
   vérifie le code sur un environnement distant et peut conditionner la fusion.
3. **Configuration versionnée :** partager les mêmes règles, suivre leur évolution
   et permettre leur reproduction par l'équipe et la CI.
4. **Faux positifs :** les motifs et heuristiques peuvent prendre une donnée fictive
   ou publique pour un secret ; une qualification humaine reste nécessaire.
5. **Baseline :** elle référence des détections connues ; elle ne retire pas un secret
   et ne révoque pas ses droits. Y inscrire un vrai secret peut masquer son maintien.
6. **eval() :** une entrée non fiable peut exécuter du code arbitraire avec les droits
   du processus. Une liste explicite d'opérations réduit ce risque.
7. **Exclusion de .env :** éviter de publier les valeurs locales et les secrets.
   `.gitignore` ne retire pas un fichier déjà suivi ni son historique.
8. **Variable d'environnement :** elle évite le secret dans le code mais ne gère
   ni les droits, ni la rotation, ni les fuites par logs ou environnement compromis.
   Un gestionnaire de secrets reste nécessaire selon l'environnement de production.
9. **Analyse et audit :** l'analyse de code cherche des pratiques risquées dans le
   programme ; l'audit compare les dépendances à des vulnérabilités connues.
10. **Tests en CI :** les hooks peuvent être absents ou contournés ; un environnement
    indépendant vérifie aussi que le code fonctionne sans dépendre du poste local.
11. **Protection de main :** règles côté hébergeur exigeant pull request, revue et
    vérifications CI réussies, sans autorisation de contourner ces règles.
12. **Indicateurs :** couverture des dépôts, délai de correction et exceptions
    actives, détaillés dans la section 12 ; le taux de CI réussies et les
    vulnérabilités ouvertes peuvent compléter ce suivi.

## 14. Bilan des livrables

| Livrable | Fichier ou état |
| --- | --- |
| Hooks partagés | `../.pre-commit-config.yaml` |
| Fichiers locaux exclus et configuration exemple | `../.gitignore` et `../.env.example` |
| Code corrigé | `../app.py` |
| Tests unitaires | `../test_app.py` : addition, soustraction, division par zéro, opération inconnue |
| Dépendances | `../requirements.txt` et `../requirements-dev.txt` |
| Résultat des contrôles | `verification.txt` : sortie de la vérification locale, à distinguer d'un rapport GitHub Actions |
| Workflow CI | `../.github/workflows/ci.yml` ; exécution distante à vérifier après publication |
| Politique et réponses | Ce document |
| Trois indicateurs | Section 12 de ce document |
| Protection de main et activation pre-commit.ci | Procédure dans le README ; activation distante non vérifiée |

### Analyse de deux problèmes et de leurs corrections

| Problème initial | Risque | Correction | Vérification et limite |
| --- | --- | --- | --- |
| Mot de passe fictif écrit directement dans app.py | Publication involontaire d'un secret si cette pratique est reprise avec une vraie valeur | Lecture de APP_PASSWORD avec python-dotenv et get_password(), .env ignoré, exemple sans valeur | Vérifier l'absence de secret en dur et exécuter detect-secrets ; si une valeur réelle a été exposée, son retrait ne dispense pas de la révoquer |
| eval() appliqué à une saisie utilisateur | Exécution de code arbitraire | Conversion des entrées en nombres et appel à calculate(), qui autorise seulement quatre opérations et refuse la division par zéro | Bandit et tests pytest ; les tests existants couvrent notamment le refus d'une opération inconnue et de la division par zéro, pas toutes les entrées interactives |

Le problème de lecture de `.secrets.baseline` a aussi été corrigé par sa conversion
de UTF-16 vers UTF-8. Une baseline illisible est une erreur de configuration,
pas une absence de secret.
