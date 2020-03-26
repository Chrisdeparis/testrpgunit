# Test unitaire avec RPGUnit

Voici la méthode de réalisation des tests unitaires avec RPGUnit.

- Tout d'abord la réalisation du CL qui comprend les modules, car le test est un programme de service qui
utilise des modules.

- Nous verrons également que les tests sont à réaliser en Intégration.

## La compilation du CL

- je reprends l'exemple de mon premier CL de test réalisé pour un programme de conversion des celsius en Fahrenheit.

```sqlrpgle
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
> Chaque module doit être préparé suivant les règles de RPGUnit SUnomdumodule, nomdumoduleTU et TDnomdumodule dans le fichier QRPGSRC.
> CALL **T_NOMCLI pour créer le module et programme de service**

## Le Setup
Le setup dans mon cas : **SUcel2fahr**
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
## Le Test
Les tests sont à faire au pas à pas. Dans mon cas **cel2fahrTU**.

```
**free
ctl-opt nomain;

/copy RPGUNIT/RPGUNIT1,TESTCASE
/copy h1frptechs/qcopsrc,s_spvDS

// définition des constantes
dcl-c APPLICATION     const('ADHTU');

// définition des variables
dcl-s wcurrentdate   date;
dcl-s wvanpalier     packed(18:11);
dcl-s rc             int(10);
dcl-s wCommande      char(512);

dcl-s wtempFahr    packed(5:0);
dcl-s wtempCelsius packed(5:2);
dcl-s wstatus      char(10);

// définition des prototypes
dcl-pr execcmd    int(10) extproc('system');
  cmdstring     pointer value   options(*string);
end-pr;

dcl-pr chargeDB2  ;
  nomTest  char(10) const;
  testCase char(10) const;
end-pr;

dcl-pr celsToFahr Extproc('CELSTOFAHR');
  in_tempCel    packed(5:2);
  ou_tempFahr packed(5:0);
  ou_status      char(10);
end-pr;

dcl-pr test_01_jetepasse10jerecois50etliquid;
end-pr;

dcl-pr test_02_jetepasse0jerecois32etfreezing;
end-pr;

//==============================================================
// CAS de TEST 1 : passant

dcl-proc test_01_jetepasse10jerecois50etliquid export;

  wtempFahr    = 0;
  wtempCelsius = 10;
  wstatus      = *blanks;

  // appel
  celsToFahr (wtempCelsius
             :wtempFahr
             :wstatus);

  assert (wtempFahr=50
         : ' Erreur temp dans celsToFahr '
         );

  assert (wstatus = 'liquid'
         : ' Erreur status dans celsToFahr '
         );

end-proc;

//==============================================================
// CAS de TEST 2 : passant

dcl-proc test_02_jetepasse0jerecois32etfreezing export;

  wtempFahr    = 0;
  wtempCelsius = 0;
  wstatus      = *blanks;

  // appel
  celsToFahr (wtempCelsius
             :wtempFahr
             :wstatus);

  assert (wtempFahr = 32
         : ' Erreur temp dans celsToFahr '
         );

  assert (wstatus = 'freezing'
         : ' Erreur status dans celsToFahr '
         );

end-proc;

//----------------------------------------------------------------
dcl-proc chargeDB2;

  dcl-pi *n;
    APPLICATION    char(10) const;
    testCase       char(10) const;
  end-pi;

  dcl-s wCommand    char(512);

  wrc=0;

  wCommand = 'RUNSQLSTM '
           + 'SRCSTMF'
           + '('''
           + '/Application/Adhesion/TU/chargeDB2/t_spvrcapp/'
           + %trim(testCase)
           + '.sql'
           + ''') '
           + 'COMMIT(*NC) '
           + 'MARGINS(112)';

   wrc = execCmd(wCommand);

   assert(wrc=0
      :'Une erreur est survenue lors de la creation des BDD');

end-proc;

```

## Le Tear Down
C'est la dernière partie des modules.
```
**free
ctl-opt nomain;

/copy RPGUNIT/RPGUNIT1,TESTCASE
/copy h1frptechs/QCOPSRC,s_errorDS

dcl-s wCommande    char(512);
dcl-s wrc          int(10);

dcl-pr execcmd    int(10) extproc('system');
  cmdstring     pointer value   options(*string);
end-pr;

dcl-pr tearDown end-pr;

//---------------------------------------------------------------//
//Procédure de nettoyage en fin de test (appelée en sortie)
dcl-proc tearDown export;

  dcl-pi *n;

  end-pi;

end-proc;
// Procédure fin: ENDJRNPF
//------------------------------------------------------------------
dcl-proc endjrnpf;

  wCommande = 'ENDJRNPF FILE(*ALL) JRN(t_fahrtoce/QSQJRN)';

  // execution commande
  wrc = execCmd(%trim(wcommande));

  assert (wrc=0
         :'Une erreur est survenue lors du endjrnpf, rc = '
          + %char(wrc)
         );
end-proc;

// Procédure fin: DLTJRNRCV
//------------------------------------------------------------------
dcl-proc dltjrnrcv;

  wCommande = 'DLTJRNRCV JRNRCV(t_cel2fahr/QSQJR*) DLTOPT(*IGNINQMSG)';

  // execution commande
  wrc = execCmd(%trim(wcommande));

  assert(wrc=0
      :'Une erreur est survenue lors du dltjrnrcv');

end-proc;

// Procédure fin: DLTJRN
//------------------------------------------------------------------
dcl-proc dltjrn;

  wCommande = 'DLTJRN JRN(t_cel2fahr/QSQJRN)';

  // execution commande
  wrc = execCmd(%trim(wcommande));

  assert(wrc=0
      :'Une erreur est survenue lors du dltjrn');

end-proc;

//------------------------------------------------
dcl-proc rmvlible export;

  dcl-pi *n;
    library    char(10) const;
  end-pi;

  wcommande = 'RMVLIBLE LIB('
              + %trim(library)
              + ')';

  // execution commande
  wrc = execCmd(%trim(wcommande));

  assert (wrc=0
         :'Erreur lors du rmvlible de ' + library
         );

end-proc; 
```


## Envoyer les modifs en intégration

- La commande est **SAVRSTOBJ OBJ(T_cel2fahr) LIB(ADHTU) RMTLOCNAME(RECETTE) OBJTYPE(*SRVPGM)**
> Je l'ai ajouté au CL comme ça toute modification est envoyée directe en recette.
> RMTLOCNAME indique la machine de destination
> LIB la Bibliothèque de Tests

## Tester en intégration

- La commande est **RUCALLTST T_cel2fahr**
> On peut voir l'affichage du resultat en 5250. Faire les Tests un par un.

