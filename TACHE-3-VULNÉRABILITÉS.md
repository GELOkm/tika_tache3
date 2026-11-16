### Vulnérabilités introduites:

#### 1. Exécution de commandes arbitraire

Fichier modifié : `ImagePreprocessor.java` (module `tika-parser-ocr-module`)

Type de vulnérabilité : Exécution de commandes arbitraire

**Justification de l'introduction de la vulnérabilité** :
Une recherche globale sur le répertoire de Tika a été faite pour chercher des occurences de mots clés entourant les
utilitaires d'exécution de commandes. En utilisant un filtre pour ne chercher qu'à l'intérieur des modules authorisés
pour la tâche 3, nous avons trouvé une occurence de `.execute(...)` dans la classe `ImagePreprocessor.java`.
En analysant la logique de gestion des arguments de `DefaultExecutor.execute(...)`, nous avons conclu qu'il serait un
bon candidat pour introduire une entrée controlée par utilisateur à l'aide d'une variable d'environnement.

**Mise en scénario** :

_Développeur_ : Ajoute la fonctionnalité de pouvoir fournir des arguments additionnels à la commande de prétraitement
d'image via une variable d'environnement. Afin de satisfaire les procédés qui nécessitent des prétraitements d'image
spéciaux.

_Attaquant_ : Assigne une valeur spécifique à la variable d'environnement afin d'obtenir la capacité à exécuter une
commande arbitraire.

**Remarque** :

La première version de la vulnérabilité introduite n'a introduit que l'ajout d'une entrée vulnérable via `getenv()`.
Lors de l'exécution de semgrep, nous avons remarqué que cette vulnérabilité n'avait pas été identifié. Après avoir
appris à interpréter les règles de semgrep, nous avons compris que la règle de détection ne prenait pas en charge la
librairie d'Apache (`org.apache.commons.exec`) ainsi que d'autres différences qui ont fait passer la vulnérabilité sous
le radar. Nous l'avons donc modifiée pour rendre cette vulnérabilité détectable.

---

#### 2. Déni de service

Fichier modifié : `CleanPhoneText.java` (module `tika-core`)

Type de vulnérabilité : Déni de service

**Justification de l'introduction de la vulnérabilité** :
À l'aide d'une recherche globale par regex sur le répertoire, nous avons identifié plusieurs définitions de regex.
Nous avons ensuite analysé rapidement celles-ci afin d'identifier les regex qui pourraient potentiellement être mal
implémentées. Dans `CleanPhoneText.java`, nous avons trouvé le regex `cleanPhoneRegex` qui est un simple regex pour
valider les numéros de téléphone. Nous l'avons modifié (avec l'aide d'un LLM) pour le rendre vulnérable à une attaque
de déni de service par regex (ReDoS).

**Mise en scénario** :

_Développeur_ : Modifie le regex pour supporter des formats de numéros de téléphone plus complexes, mais introduit
involontairement une vulnérabilité ReDoS.

_Attaquant_ : Fournit une entrée spécialement conçue pour exploiter la vulnérabilité ReDoS, provoquant un déni de
service causé par une explosion de complexité dans le traitement du regex (validé
via [regex101.com](https://regex101.com/?regex=%5E%28%5C%2B%3F1%5B-.%5Cs%5D%3F%29%3F%28%5C%28%3F%5Cd%7B3%7D%5C%29%3F%5B-.%5Cs%5D%3F%29%3F%28%5Cd%2B%5B-.%5Cs%5D%3F%29*%5Cd%7B4%7D%24&testString=1111111111111111111111111x&flags=gm&flavor=java&delimiter=%22)).

**Remarque** :

Nous nous sommes rendus compte que la règle de détection pour ReDoS ne s'appliquait pas à Java. Nous avons donc
introduit une régle personnalisée basée
sur [la règle de détection ReDoS pour C++.](https://semgrep.dev/playground/r/0xdea.c.regex-dos.raptor-regex-dos?editorMode=advanced)

---

