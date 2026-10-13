# Journal de bord - Introduction au DevOps

Dépôt : `victorriv42/visites-Mathis` - binôme : Victor (A) et Mathis (B).

## TP01 - Forge, dépôt et flux de travail

**Q1. Délai entre mon premier commit et sa fusion sur main**
Le délai vient surtout de l'attente de la relecture de Mathis : nous ne travaillons pas en même temps.

**Q2. Ce qui ne dépend pas de GitHub**
Marcherait tel quel sur GitLab, Gitea ou Forgejo : Git (branches, rebase, conflits, tags), signature SSH, pre-commit, commits conventionnels. Changerait : les commandes `gh` et les règles de protection.

**Q3. Pourquoi signer si la forge vérifie qui pousse**
La forge vérifie qui pousse, pas qui a écrit le commit : l'auteur se falsifie facilement. La signature prouve l'auteur, même si un autre pousse ou si le dépôt migre vers une autre forge.

**Questions des étapes**
- `allowed_signers` : liste `e-mail clé-publique` des signataires de confiance, utilisée par `git log --show-signature` pour afficher `Good "git" signature`.
- Modèle ou fork : un modèle crée un dépôt neuf sans lien avec l'original, un fork garde le lien.
- `git branch -d` refuse après un squash : le squash crée un nouveau commit, Git croit la branche non fusionnée. Il faut `-D`.
- `--force-with-lease` : refuse de pousser si la branche distante a bougé depuis mon dernier `fetch`. `--force` écrase sans vérifier.
- pre-commit : un fichier mal formaté fait échouer le commit (ruff reformate, je refais `git add`). `--no-verify` saute les hooks.

**Ce qui m'a pris le plus de temps**
1. La clé SSH : mauvais nom de fichier, HTTPS au lieu de SSH, session root.
2. L'accès au dépôt modèle (404, dépôt non marqué comme modèle). [À COMPLÉTER : solution retenue]
3. L'invitation de Mathis : lien en 404 tant qu'il n'était pas connecté au bon compte.
4. Un `readme.md` créé en minuscules à côté de `README.md`.

**Observation** : Mathis a demandé un test sur `GET /` quand Redis est indisponible. Aujourd'hui l'exception remonte (500) ; un 503 comme sur `/health` serait plus propre.

## TP02 - Protéger le dépôt

**Ruleset `main`** : suppression et force push interdits, PR avec 1 approbation et "dismiss stale approvals", historique linéaire, commits signés. Le check "test" viendra au TP03.
- Push direct refusé
- PR #4 avec commit non signé : `Unverified`, fusion bloquée.

**Pourquoi "dismiss stale approvals" ?** Sans cette option, on peut faire approuver un petit changement puis pousser du code non relu. Avec elle, tout nouveau push annule les approbations.

**CODEOWNERS** : moi sur `.github/` et `infra/`, Mathis sur `app/` et `monitoring/`, nous deux sur `docs/`.
**Propriétaire seul et absent ?** La PR reste bloquée, un auteur ne pouvant pas approuver sa propre PR. Il faut au moins deux propriétaires par zone sensible ou une procédure de remplacement.

**Dependabot** : dependabot.yml est fusionné et pris en compte. Les trois écosystèmes configurés sont détectés : pyproject.toml (pip), Dockerfile (docker) et actions (github-actions)
**Qui approuve la PR d'un robot ?** Un membre humain, avec les mêmes règles que pour tout le monde. C'est une bonne chose : une mise à jour de dépendance exécute du code externe, donc un humain la vérifie et les tests passent avant fusion.

**gitleaks**
**Nom de la variable ou valeur ?** Les règles génériques s'appuient sur un mot-clé dans le nom, les règles par fournisseur sur le format de la valeur.

**Secret historique** : commit [À COMPLÉTER : SHA], auteur [À COMPLÉTER], date [À COMPLÉTER], fichier `app/old_settings.py`.
**Réécrire l'historique ?** [À COMPLÉTER : décision avec Mathis]. `git filter-repo` change tous les SHA, donc tout le monde doit recloner.
**Rotation** : le secret a été poussé, donc considéré comme compromis. Le fichier est supprimé par PR et un vrai jeton serait révoqué chez le fournisseur. L'empreinte du faux positif est dans `.gitleaksignore` pour un rapport propre.

**Q1. Quelles règles auraient bloqué le secret ?** La revue (humaine) peut laisser passer. Le contrôle requis gitleaks en CI (automatique) est le plus fiable, car il ne se contourne pas depuis le poste.
**Q2. pre-commit protège-t-il un collègue qui ne l'a pas installé ?** Non, et `--no-verify` le contourne. Il faut gitleaks en CI comme contrôle requis (TP03), et la push protection de GitHub.
**Q3. Combien de temps un secret reste-t-il privé ?** Quelques minutes au plus, des robots scrutent les dépôts publics. GitHub Secret Scanning fait révoquer certains jetons de fournisseurs partenaires. [À COMPLÉTER : types couverts, selon la doc GitHub]
