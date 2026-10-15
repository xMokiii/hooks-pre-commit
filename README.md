# hooks-pre-commit

Projet Python avec controles de qualite et de securite avant les commits et dans la CI.

Le [bilan du TP](docs/bilan-tp.md) contient les réponses aux sections 10 à 14,
la politique de sécurisation proposée, trois indicateurs et l'analyse des corrections.
Le [résultat des contrôles locaux](docs/verification.txt) accompagne les livrables.

## Installation locale

Avec Python 3.13, sous PowerShell :

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements-dev.txt
Copy-Item .env.example .env
pre-commit install
pre-commit run --all-files
```

Renseigner `APP_PASSWORD` dans `.env` pour les fonctionnalites qui en ont besoin.
Ce fichier reste local. Les tests de calcul et la CI ne necessitent aucun secret.

## CI GitHub Actions (section 9)

Le workflow `.github/workflows/ci.yml` utilise Python 3.13 et s'execute sur les
pull requests vers `main`, sur les pushes vers `main` et sur lancement manuel.
Il installe `requirements-dev.txt`, puis lance `pre-commit run --all-files`.
Les hooks executent le nettoyage des fichiers, la validation YAML, Bandit,
detect-secrets, les tests `pytest -q` et `pip-audit -r requirements.txt`.
Un echec bloque la reussite du controle `Controles Python`.

Les actions utilisent des versions majeures. Pour un usage en production,
epingler leurs SHA apres validation et maintenir ces references a jour.

## Protection de main sur GitHub (section 9.1)

La configuration locale ne protege pas la branche distante. Reglages a activer
par un administrateur apres avoir pousse le workflow et obtenu sa premiere execution :

1. Ouvrir les [regles de protection du depot](https://github.com/xMokiii/hooks-pre-commit/settings/branches)
   et creer une regle pour `main`.
2. Activer **Require a pull request before merging**, avec une approbation requise.
3. Activer **Dismiss stale pull request approvals when new commits are pushed**.
4. Activer **Require status checks to pass before merging**, selectionner
   `Controles Python` et exiger une branche a jour avant la fusion.
5. Activer **Do not allow bypassing the above settings** pour appliquer les regles
   aux administrateurs. Laisser les force pushes et les suppressions interdits.
6. Enregistrer la regle et verifier qu'un push direct vers `main` est refuse.

Une approbation exige un autre collaborateur : l'auteur d'une pull request ne
peut pas approuver sa propre demande. La disponibilite des protections depend
de la visibilite du depot et de l'offre GitHub.

Voir la [documentation GitHub sur les branches protegees](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches).

## pre-commit.ci (section 9.2)

Configuration preparee dans `.pre-commit-config.yaml` : corrections automatiques
des pull requests et propositions mensuelles de mise a jour des hooks.

Pour activer le service, se connecter sur [pre-commit.ci](https://pre-commit.ci/)
avec GitHub et autoriser son application sur ce depot. Le fichier de
configuration seul n'active pas le service.

Les hooks locaux `pytest` et `pip-audit`, qui utilisent `language: system`, sont
ignores uniquement par pre-commit.ci. GitHub Actions continue de les executer.
Le controle `Controles Python` doit donc rester obligatoire dans la protection
de `main`, meme si pre-commit.ci est active. Apres sa premiere execution, son
controle peut egalement etre ajoute aux verifications obligatoires.
