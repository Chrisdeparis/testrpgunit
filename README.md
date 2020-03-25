# Test unitaire avec RPGUnit

- Dans ce dépot je vais exposer la méthode de réalisation des tests unitaires avec RPGUnit.

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

## Envoyer les modifs en intégration

- La commande est SAVRSTOBJ OBJ(T_NOMCLI) LIB(ADHTU) RMTLOCNAME(RECETTE) OBJTYPE(*SRVPGM)
> RMTLOCNAME indique la machine de destination
> LIB la Bibliothèque de Tests
