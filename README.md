# Test unitaire avec RPGUnit

Voici la méthode de réalisation des tests unitaires avec RPGUnit.

- Tout d'abord la réalisation du CL qui comprend les modules, car le test est un programme de service qui
utilise des modules.

- Nous verrons également que les tests sont à réaliser en Intégration.

## La compilation du CL

- je reprends l'exemple de mon premier CL de test réalisé pour un programme de conversion des celsius en Fahrenheit.

```
    PGM
    /* déclaration des variables */
    DCL        VAR(&LIBTUADH) TYPE(*CHAR) LEN(10) VALUE('ADHTU')

    /* suppression des modules */
    DLTMOD     MODULE(&LIBTUADH/SUcel2fahr)
    MONMSG     CPF2105

    DLTMOD     MODULE(&LIBTUADH/cel2fahrTU)
    MONMSG     CPF2105

    DLTMOD     MODULE(&LIBTUADH/TDcel2fahr)
    MONMSG     CPF2105

    /* création des modules */
    CRTSQLRPGI OBJ(&LIBTUADH/SUcel2fahr) SRCFILE(&LIBTUADH/QRPGSRC) OBJTYPE(*MODULE) +
                DBGVIEW(*SOURCE)
    CRTSQLRPGI OBJ(&LIBTUADH/cel2fahrTU) SRCFILE(&LIBTUADH/QRPGSRC) OBJTYPE(*MODULE) +
                DBGVIEW(*SOURCE)
    CRTSQLRPGI OBJ(&LIBTUADH/TDcel2fahr) SRCFILE(&LIBTUADH/QRPGSRC) OBJTYPE(*MODULE) +
                DBGVIEW(*SOURCE)

    /* création programme de service TU */
    CRTSRVPGM  SRVPGM(&LIBTUADH/T_cel2fahr) MODULE(&LIBTUADH/SUcel2fahr +
                &LIBTUADH/cel2fahrTU &LIBTUADH/TDcel2fahr) EXPORT(*ALL) TEXT('TU - +
                module cel2fahr') BNDDIR(RPGUNIT/RPGUNIT QC2LE LINK) ACTGRP(*CALLER)
    ENDPGM 
```
> Chaque module doit être préparé suivant les règles de RPGUnit SUnomdumodule, nomdumoduleTU et TDnomdumodule.

Le setup dans mon cas : SUcel2fahr
```
**free
ctl-opt nomain;

/copy RPGUNIT/RPGUNIT1,TESTCASE

  dcl-s wCommande   char(512);
  dcl-s wrc          int(10);

// définition des prototypes
dcl-pr setUp;
end-pr;

dcl-pr execcmd    int(10) extproc('system');
  cmdstring     pointer value   options(*string);
end-pr;

//---------------------------------------------------------------//
//  Procédure de mise en place des tests, appelée en entrée
//  elle permet d'initialiser l'environnement (bdd) pour avoir
//  des jeux de test toujours identiques et propres

dcl-proc setUp  export;

  dcl-pi *n;

  end-pi;

end-proc;

//--------------------------------------------------------------------
```

## Envoyer les modifs en intégration

- La commande est SAVRSTOBJ OBJ(T_cel2fahr) LIB(ADHTU) RMTLOCNAME(RECETTE) OBJTYPE(*SRVPGM)
> RMTLOCNAME indique la machine de destination
> LIB la Bibliothèque de Tests

## Tester en intégration

- La commande est RUCALLTST T_cel2fahr
> On peut voir l'affichage du resultat en 5250.

