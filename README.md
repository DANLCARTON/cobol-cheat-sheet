# COBOL ou quoient ?????

## Petite cheat sheet là :)

### Définition d'un fichier en entrée, sortie ou entrée-sortie

```cobol
* dans la INPUT-OUTPUT SECTION
     SELECT {{nom interne}} ASSIGN TO {{nom externe (ddname)}}
     FILE-STATUS IS {{nom interne préfixé par WS-FS-}}.

* dans la FILE SECTION
     FD {{nom interne}}
         RECORDING MODE IS {{type d'enregistrement}}.
     01 {{nom du buffer préfixé par FS-}} PIC {{format picture}}.

* dans la WORKING-STORAGE SECTION
    77 {{nom interne préfixé par WS-FS-}} PIC XX.
```

### Définition des variables

```cobol
* dans la WORKING-STORAGE SECTION

    01 WS-GROUPE.
        05 WS-SOUS-VARIABLE PIC {{format picture}}.
        05 WS-SOUS-VARIABLE-2.
            10 WS-SOUS-SOUS-VARIABLE PIC {{format picture}}.

    01 WS-GROUPE-AVEC-FILLER.
        05 FILLER PIC X(20) VALUE 'VALEUR FIXE        '.
        05 FILLER PIC X(20) VALUE ALL '*'.
```

### Définition et appel d'un paragraphe

```cobol
* Dans la PROCEDURE DIVISION

NNNN-NOM-PARAGRAPHE-DEB.
    {{Instructions, lignes de code à faire dans le paragraphe}}.
NNNN-NOM-PARAGRAPHE-FIN.
    {{EXIT. ou STOP RUN. dans le cas de 0000 ou 9999-ERREUR}}

NNNN-PARAGRAPHE-2-DEB.
    PERFORM NNNN-NOM-PARAGRAPHE-DEB
    THRU    NNNN-NOM-PARAGRAPHE-FIN.
NNNN-PARAGRAPHE-2-FIN.
    EXIT.
```

### Top 3 des différents types de PERFORM

```cobol
* perform simple
    PERFORM PARAGRAPHE-DEB
    THRU    PARAGRAPHE-FIN.

* perform until (Itérative)
    PERFORM PARAGRAPHE-DEB
    THRU    PARAGRAPHE-FIN
    UNTIL   {{condition qui arrête la boucle quand elle est vraie}}.

* perform varying
    PERFORM PARAGRAPHE-DEB
    THRU    PARAGRAPHE-FIN
    VARYING VARIABLE
    FROM    {{valeur initiale de VARIABLE}}
    BY      {{combien ajouter à VARIABLE à chaque itération}}
    UNTIL   {{condition qui arrête la boucle quand elle est vraie impliquant VARIABLE}}.
```

### Instructions conditionnelles

```cobol
* if (alternative incomplète)
    IF {{condition à tester}} THEN
        {{Instructions, lignes de code à faire quand la condition est vraie}}
    END-IF.

* if else (alternative simple)
    IF {{condition à tester}} THEN
        {{Instructions, lignes de code à faire quand la condition est vraie}}
    ELSE
        {{Instructions, lignes de code à faire quand la condition est fausse}}
    END-IF.

* evaluate (alternative multiple)
    EVALUATE VARIABLE
        WHEN {{valeur de VARIABLE à tester}}
            {{Instructions, lignes de code à faire quand la condition est vraie}}
        WHEN {{valeur de VARIABLE à tester}}
            {{Instructions, lignes de code à faire quand la condition est vraie}}
        WHEN {{valeur de VARIABLE à tester}}
            {{Instructions, lignes de code à faire quand la condition est vraie}}
        WHEN OTHER
            {{Instructions, lignes de code à faire quand aucune des conditions précédentes n`est vraie}}
    END-EVALUATE.

* ou
    EVALUATE {{TRUE ou FALSE}}
        WHEN {{condition à vérifier}}
            {{Instructions, lignes de code à faire quand la condition est TRUE ou FALSE en fonction de ce qui est précisé}}
        WHEN {{condition à vérifier}}
            {{Instructions, lignes de code à faire quand la condition est TRUE ou FALSE en fonction de ce qui est précisé}}
        WHEN {{condition à vérifier}}
            {{Instructions, lignes de code à faire quand la condition est TRUE ou FALSE en fonction de ce qui est précisé}}
        WHEN OTHER
            {{Instructions, lignes de code à faire quand aucune des conditions précédentes n`est vérfiée}}
    END-EVALUATE.
```

#### Opérateurs logiques

| comparaison | opérateur |
|---|---|
| A égal B ?  | `A = B` |
| A différent de B ? | `A NOT = B` |
| A inférieur à B ? | `A < B` |
| A inférieur ou égal à B ? | `A <= B` |
| A supérieur à B ? | `A > B` |
| A supérieur ou égal à B ? | `A >= B` |
| A n'est pas inférieur à B ? | `A NOT < B` $$\Leftrightarrow$$ `(A >= B)` |
| A n'est pas supérieur à B ? | `A NOT > B` $$\Leftrightarrow$$ `(A <= B)` |

Il est possible d'utiliser des parenthèses pour créer des tests plus complèxes. Notamment, `NOT (condition)`, permet d'inverser toute une condition. Par exemple :

`NOT (A = B AND C < D OR E >= F)` $$\Leftrightarrow$$ `A NOT = B OR C >= D AND E < F` ou `A NOT = B OR C NOT < D AND E NOT >= F`

### Manipulation des fichiers (paragraphes 6000)

#### Fichiers séquentiels

```cobol
* ouvrir un fichier
6NNN-OUVRIR-FICHIER-DEB.
    OPEN {{INPUT, OUTPUT ou I-O}} {{nom interne}}.
    IF {{variable du File Status}} NOT = '00' THEN
        DISPLAY 'problème d''ouverture du fichier'
        DISPLAY 'file status :' {{variable du File Status}}

        PERFORM 9999-ERREUR-PROGRAMME-DEB
        THRU    9999-ERREUR-PROGRAMME-FIN
    END-IF.
6NNN-OUVRIR-FICHIER-FIN.
    EXIT.

* lire un fichier
6NNN-LIRE-FICHIER-DEB.
    READ {{nom interne}} INTO {{variable de l'enregistrement}}.
    IF  {{variable du File Status}} NOT = '00'
    AND {{variable du File Status}} NOT = '10' THEN
        DISPLAY 'problème de lecture du fichier'
        DISPLAY 'file status :' {{variable du File Status}}

        PERFORM 9999-ERREUR-PROGRAMME-DEB
        THRU    9999-ERREUR-PROGRAMME-FIN
    END-IF.
6NNN-LIRE-FICHIER-FIN.
    EXIT.

* écrire dans le fichier
6NNN-ECRIRE-FICHIER-DEB.
    WRITE {{variable du buffer du fichier}} FROM {{variable ou littéral}}.

    WRITE {{variable du buffer du fichier}} FROM {{variable ou littéral}} AFTER {{nombre de lignes}}.
*   (écriture dans le fichier avec la variable ou le litteral ARPÈS avoir sauté le nombre de lignes indiqué)

    WRITE {{variable du buffer du fichier}} FROM {{variable ou littéral}} AFTER PAGE.
*   (écriture dans le fichier avec la variable ou le litteral ARPÈS avoir sauté une page)

    IF {{variable du File Status}} NOT = '00' THEN
        DISPLAY 'problème d''écriture du fichier'
        DISPLAY 'file status :' {{variable du File Status}}

        PERFORM 9999-ERREUR-PROGRAMME-DEB
        THRU    9999-ERREUR-PROGRAMME-FIN
    END-IF.
6NNN-ECRIRE-FICHIER-FIN.
    EXIT.

* fermer un fichier
6NNN-FERMER-FICHIER-DEB.
    CLOSE {{nom interne}}.
    IF {{variable du File Status}} NOT = '00' THEN
        DISPLAY 'problème de fermeture du fichier'
        DISPLAY 'file status :' {{variable du File Status}}

        PERFORM 9999-ERREUR-PROGRAMME-DEB
        THRU    9999-ERREUR-PROGRAMME-FIN
    END-IF.
6NNN-FERMER-FICHIER-FIN.
    EXIT.
```

### Opérations, transferts et calculs (paragraphes 7000)

```cobol
* additionner
    ADD {{une ou plusieurs valeurs ou variables séparées par des espaces}} 
        TO {{variable source}}. 
*   (le résultat est placé dans variable source)

    ADD {{une ou plusieurs valeurs ou variables séparées par des espaces}} 
        GIVING {{variable cible}}. 
*   (le résultat est placé dans variable cible) 

* soustraire
    SUBTRACT {{une ou plusieurs valeurs ou variables séparées par des espaces}} 
        FROM {{variable source}}.
*   (le résultat est placé dans variable soruce)

    SUBTRACT {{une ou plusieurs valeurs ou variables séparées par des espaces}} 
        FROM {{une ou plusieurs valeurs ou variables séparées par des espaces}} 
        GIVING {{variable cible}}. 
*   (le résultat est placé dans variable cible)

* multiplier
    MULTIPLY {{valeur ou variable}} 
        BY {{variable source}}. 
*   (le résultat est placé dans variable source) 

    MULTIPLY {{valeur ou variable}} 
        BY {{valeur ou variable}} 
        GIVING {{variable cible}}. 
*   (le résultat est placé dans variable cible) 

* diviser
    DIVIDE {{valeur ou variable}} 
        INTO {{variable source}}. 
*   (le résultat est place dans variable source)

    DIVIDE {{valeur ou variable}} 
        INTO {{valeur ou variable}} 
        GIVING {{variable cible}} 
        REMAINDER {{variable cible 2}}. 
*   (division euclidienne. le quotient est placé dans variable cible le reste est placé dans varaible cible 2)

    DIVIDE {{valeur ou variable}} 
        BY {{valeur ou variable}} 
        GIVING {{variable cible}}. 
*   (division exate. le résultat est placé dans variable cible)

* transfert
    MOVE {{valeur ou variable}} TO {{variable cible}}.

* calcul
    COMPUTE {{variable cible}} = {{expression arithmétique}}.
```

Il faut porter attention aux formats (PIC) des valeurs et variables utilisées dans les opérations pour éviter les pertes de données. 

COMPUTE devient plus intéressant à utiliser que les opérations classiques quand une variable doit subir plusieurs types d'opérations à la suite. La priorité des opérations est appliquée. Dans COMPUTE, les symboles mathématiques doivent être entourées d'espaces, les symboles suivants peuvent être utilisés : 

| Symbole | Opération |
|---|---|
| + | Addition |
| - | Soustraction |
| * | Multiplication |
| / | Division exacte (fraction) |
| ** | Puissance (2 ** 3 = 8) |

### Affichages (paragraphes 8000)

```cobol
* Afficher dans la SYSOUT
    DISPLAY {{valeurs, chaines de caractères ou variables séparées par les espaces}}.
```

### Appels de sous-programmes (paragraphes 9000)

n/c

### Erreurs programme (paragraphe 9999-ERREUR-PROGRAMME)

À priori celui-là est fourni à chaque fois mais je le met là quand même on sait jamais 😶

```cobol
9999-ERREUR-PROGRAMME-DEB.
    {{Instructions, lignes de code à faire en cas d'erreur du programme}}.
    MOVE 12 TO RETURN-CODE.
    
9999-ERREUR-PROGRAMME-FIN.
    STOP RUN.
```
