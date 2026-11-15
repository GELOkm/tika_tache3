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

