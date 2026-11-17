### Modification et validation du Workflow GitHub Actions

#### 1. Détection des nouvelles vulnérabilités avec Semgrep
**Choix de conception et d'implémentation :**
Pour intégrer Semgrep, nous avons choisi d'utiliser l'action officielle `semgrep/action@v1` au lieu d'une installation manuelle, ce qui rend le workflow plus maintenable. 
Nous avons défini la variable d'environnement `SEMGREP_BASELINE_REF` pour trouver que les nouvelles vulnérabilités.

Pour faire cela:
1. Nous avons ajouté `fetch-depth: 0` à l'étape `actions/checkout` pour que l'action télécharge l'historique complet des commits, pour que Semgrep puisse faire un `git diff`.
2. Nous avons configuré l'action avec `config: "p/java, p/findsecbugs"`, car plusieurs de nos vulnérabilités injectées dépendent des règles plus avancées de FindSecBugs.

**Validation :**
Nous avons validé cette modification en exécutant le workflow sur un nouveau commit. Le système a comparé l'état actuel à l'état précédent et a correctement fait échouer le build en identifiant exactement nos 7 nouvelles vulnérabilités.

#### 2. Implémentation du "Rickroll" (Humour)
**Choix de conception :**
Pour satisfaire le critère d'humour, nous avons intégré l'action existante recommandée `TejasvOnly/random-rickroll@v1.0` à la fin de notre bloc de jobs. Nous avons configuré l'étape avec la condition `if: failure()` pour qu'elle s'exécute si le build Maven échoue ou si Semgrep détecte une nouvelle faille.
Le paramètre `percentage: 100` a été forcé pour s'assurer que le Rickroll s'affiche à 100% des cas d'échec sous forme de commentaire automatique sur la Pull Request (grâce aux permissions `pull-requests: write` accordées au `GITHUB_TOKEN`).