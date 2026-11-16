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


#### 3. Générateur de nombres aléatoires prédictible
Fichier modifié : PDFParser.java (module tika-parser-pdf-module)

Type de vulnérabilité : Générateur de nombres aléatoires prédictible (CWE-338)

**Justification de l'introduction de la vulnérabilité :**
Lors de l'analyse d'un document PDF, Tika doit fréquemment extraire des ressources imbriquées (comme des images ou des polices) et les stocker temporairement. Nous avons jugé qu'il serait réaliste d'introduire une méthode générant un identifiant aléatoire. Pour introduire une vulnérabilité, nous avons utilisé java.util.Random au lieu de SecureRandom.

**Mise en scénario** :

_Développeur_ : A besoin de générer rapidement des identifiants uniques pour isoler les fichiers extraits du PDF. Il choisit la classe standard Random ne connaissant pas le risque de sécurité.

_Attaquant_ : Calcule le seed du générateur faible pour prédire les prochains noms de fichiers temporaires qui seront générés. L'attaquant peut ensuite lire des données sensibles extraites d'autres PDF sur le serveur ou écraser ces fichiers avant que l'application ne les lise.

#### 4. Secret codé en dur (Vulnérabilité déjà exsistante)
Fichier analysé : PDFParser.java (module tika-parser-pdf-module)

Type de vulnérabilité : Utilisation d'un mot de passe codé en dur (CWE-798)

**Justification de l'introduction de cette vulnérabilité :**
Lors de nos tests de scan avec Semgrep et le paquet findsecbugs, l'outil a levé une alerte bloquante authentique (HARD_CODE_KEY-2) sur la ligne 176 de PDFParser.java pour l'initialisation String password = "";. Nous ne l'avons pas injectée nous -même, mais nous l'avons détectée.

**Mise en scénario** :

_Développeur_ : Laisse une variable de mot de passe initialisée à une chaîne vide hard-coded directement dans le code source pour faciliter le débogage local.

_Attaquant_ : Inspecte le code source disponible publiquement, repère cette variable hard-coded et tente de contourner la protection des documents en forçant l'application à retomber sur ce mot de passe vide par défaut.

#### 5. Injection de commandes système
Fichier modifié : OCR2XHTML.java (module tika-parser-ocr-module)

Type de vulnérabilité : Injection de commandes système (CWE-78)

**Justification de l'introduction de la vulnérabilité :**
Dans l'OCR on fait souvent appel à des programmes externes.
Une commande système peut être passée via les métadonnées Nous avons donc déterminé que l'ajout d'une fonctionnalité avec un script de prétraitement d'image serait extrêmement plausible dans cette classe. Nous avons introduit un appel direct à Runtime.getRuntime().exec() prenant en paramètre une chaîne de caractères non validée.

**Mise en scénario** :

_Développeur_ : Ajoute une méthode pour permettre aux utilisateurs avancés de fournir leur propre script de traitement d'image avant la passe d'OCR. Il passe directement le chemin du script au processus système sans échapper les caractères spéciaux ni valider les entrées.

_Attaquant_ : Fournit un chemin de script malveillant contenant des caractères d'évasion système ce qui lui donne la capacité d'exécuter des commandes arbitraires avec les privilèges de l'application Tika sur le serveur hôte.