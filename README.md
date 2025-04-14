# CBT159
Converted to GitHub via [cbt2git](https://github.com/wizardofzos/cbt2git)

This is still a work in progress. 
Due to amazing work by Alison Zhang and Jake Choi repos are no longer deleted.

```
//***FILE 159 IS FROM CBT AND CONTAINS A COPY OF THEIR UCBFIND      *   FILE 159
//*           ROUTINE FOR MVS/SP AND MVS/SP XA.  THIS SUBROUTINE    *   FILE 159
//*           HAS TO RUN AUTHORIZED.  THAT IS ACCOMPLISHED THROUGH  *   FILE 159
//*           A USER WRITTEN SVC.  SEE THE CODE FOR COMPLETE        *   FILE 159
//*           DOCUMENTATION.                                        *   FILE 159
//*                                                                 *   FILE 159
//*             UCBFIND IS A SUBROUTINE FOR USE IN SP3 OR X-A       *   FILE 159
//*       SYSTEM FOR UCB LOOK UP FUNCTIONS.  THE CHARTS BELOW       *   FILE 159
//*       DESCRIBE THE FUNCTIONS.                                   *   FILE 159
//*                                                                 *   FILE 159
//*    |-------------------------------------------------------|    *   FILE 159
//*    |FUNC|     INPUT DATA         |  RETURNED OUTPUT DATA   |    *   FILE 159
//*    |CODE|                        |                         |    *   FILE 159
//*    |----|------------------------|-------------------------|    *   FILE 159
//*    | 00 | GENERIC OR ESOTERIC    | ALL MATCHING UCBS       |    *   FILE 159
//*    |    | NAME                   |                         |    *   FILE 159
//*    | 01 | DEV CLASS+TYPE FOR     | ALL MATCHING UCBS       |    *   FILE 159
//*    |    | GENERIC DEVICE         |                         |    *   FILE 159
//*    | 01 | DEV CLASS+TYPE FOR     | CURRENTLY NOT SUPPORTED |    *   FILE 159
//*    |    | ESOTERIC DEVICE        |                         |    *   FILE 159
//*    | 02 | 1 COMPLETE VOL-SER     | CURRENTLY NOT SUPPORTED |    *   FILE 159
//*    | 03 | FIRST 3 CHARS OF       | CURRENTLY NOT SUPPORTED |    *   FILE 159
//*    |    | VOL-SER                |                         |    *   FILE 159
//*    |-------------------------------------------------------|    *   FILE 159
//*      00   DEVICE SEARCH VIA GENERIC NAME:                       *   FILE 159
//*       THE EDT IS SEARCHED FOR GENERIC NAME.  IF                 *   FILE 159
//*       MATCH FOUND, THE COMPLETE MASK (DEVICE CLASS AND          *   FILE 159
//*       DEVICE TYPE) IS RETRIEVED FROM THE EDT.  THE NAME         *   FILE 159
//*       IS GENERIC IF THE DEVICE TYPE NOT 00.  ALL UCBS OF        *   FILE 159
//*       'THAT DEVICE CLASS' ARE REQUESTED FROM THE SCAN           *   FILE 159
//*       SERVICE ROUTINE.  EACH RETRIEVED UCB IS COMPARED          *   FILE 159
//*       WITH THE HELD DEVICE TYPE.  UCB ADDRESSES OF ALL          *   FILE 159
//*       MATCHES ARE STORED IN THE CALLER'S STORAGE AREA WITH      *   FILE 159
//*       A COUNT OF THE NUMBER FOUND.                              *   FILE 159
//*                                                                 *   FILE 159
//*      00   DEVICE SEARCH VIA ESOTERIC NAME:                      *   FILE 159
//*       THE EDT IS SEARCHED FOR ESOTERIC NAME.  IF                *   FILE 159
//*       MATCH FOUND, THE COMPLETE MASK (DEVICE CLASS AND          *   FILE 159
//*       DEVICE TYPE) IS RETRIEVED FROM THE EDT.  THE NAME         *   FILE 159
//*       IS ESOTERIC IF THE DEVICE TYPE = 00.  THE SCAN            *   FILE 159
//*       SERVICE ROUTINE CAN NOT BE USED, IEFAB4UV IS              *   FILE 159
//*       UTILIZED (PROTECT KEY 1)                                  *   FILE 159
//*                                                                 *   FILE 159
//*      01   DEVICE SEARCH VIA DEVICE CLASS + TYPE                 *   FILE 159
//*           FOR GENERIC NAME:                                     *   FILE 159
//*       THE DEVICE IS GENERIC IF THE DEVICE TYPE NOT 00.          *   FILE 159
//*       ALL UCBS OF 'THAT DEVICE CLASS' ARE REQUESTED FROM        *   FILE 159
//*       THE SCAN SERVICE ROUTINE EACH RETRIEVED UCB IS            *   FILE 159
//*       COMPARED WITH THE HELD DEVICE TYPE.  ALL MATCHES ARE      *   FILE 159
//*       STORED IN THE CALLER'S STORAGE AREA WITH A COUNT.         *   FILE 159
//*                                                                 *   FILE 159
//*      01   DEVICE SEARCH VIA DEVICE CLASS + TYPE                 *   FILE 159
//*           FOR ESOTERIC NAME:                                    *   FILE 159
//*       SUPPORTED ONLY UNDER X-A:                                 *   FILE 159
//*       THE DEVICE IS ESOTERIC IF THE DEVICE TYPE = 00.           *   FILE 159
//*       IEFAB4UV IS INVOKED WITH THE UCBTYP AS INPUT              *   FILE 159
//*       REQUESTING A UNIT NAME AS OUTPUT (THIS FUNCTION           *   FILE 159
//*       ONLY WORKS UNDER X-A).  IEFAB4UV IS THEN REINVOKED        *   FILE 159
//*       WITH THE UNIT NAME AS INPUT.  OUTPUT CONSISTS OF THE      *   FILE 159
//*       UCBS AND A COUNT OF THEM.                                 *   FILE 159
//*                                                                 *   FILE 159
//*       IF ALL THE UCB ADDRESSES DO NOT FIT INTO THE OUTPUT       *   FILE 159
//*       AREA, THE CALLER MUST RE-INVOKE THE SUBROUTINE WITH       *   FILE 159
//*       THE SAME REQUEST, ''WITHOUT'' CLEARING OUT THE 100        *   FILE 159
//*       BYTE WORK AREA (INFO IN THAT AREA TELLS THE               *   FILE 159
//*       SUBROUTINE WHERE TO CONTINUE PROCESSING UCBS.)            *   FILE 159
//*         THIS CODE IS NOT COMPLETED                              *   FILE 159
//*                                                                 *   FILE 159
//*         CMD BUFFER (INPUT) POINTED TO BY CPPL:                  *   FILE 159
//*                                                                 *   FILE 159
//*       ____________________________________________              *   FILE 159
//*       |   XX    |XXXXXX|CCCCCCCC|    XXXXXXXX    |              *   FILE 159
//*       |________________|________|________________|              *   FILE 159
//*       |FUNCTION |(NOT  | NAME/  |  ADDRESS OF A  |              *   FILE 159
//*       |  CODE   | USED)| DEVICE | 2K OUTPUT AREA |              *   FILE 159
//*       |         |      |  TYPE  |                |              *   FILE 159
//*       |_________|______|________|________________|              *   FILE 159
//*                                                                 *   FILE 159
//*      F  - XX FUNCTION BITS -                                    *   FILE 159
//*       00: GENERIC/ESOTERIC NAME BEING PASSED FOR UCBS           *   FILE 159
//*       01: UNITTYP (3010200E) IS BEING PASSED FOR MATCHING       *   FILE 159
//*           UCBS                                                  *   FILE 159
//*                   (00012000) ESOTERIC DEVICES   WORKS           *   FILE 159
//*                              ONLY FOR X-A                       *   FILE 159
//*       THE FOLLOWING FUNCTIONS ARE NOT SUPPORTED:                *   FILE 159
//*       02: 1 COMPLETE VOL-SER IS BEING PASSED FOR MATCHING       *   FILE 159
//*           UCB                                                   *   FILE 159
//*       03: FIRST 3 CHARS OF VOL-SER BEING PASSED FOR             *   FILE 159
//*           MATCHING UCBS                                         *   FILE 159
//*         - XXXXXX NOT UTILIZED CURRENTLY                         *   FILE 159
//*      CL8-GENERIC/ESOTERIC/VOL-SER NAME                          *   FILE 159
//*         OR:                                                     *   FILE 159
//*         - XX DEVICE CLASS                                       *   FILE 159
//*         - XXXXXX NOT UTILIZED FOR FUNCTION=01                   *   FILE 159
//*      F  - AN ADDRESS OF A 2K STORAGE AREA WHICH THE CALLER      *   FILE 159
//*          IS RESPONSIBLE TO GET/FREEMAIN.  IT WILL CONTAIN       *   FILE 159
//*          ALL THE OUTPUT FROM THE SUBROUTINE.  THE               *   FILE 159
//*          BREAKDOWN OF ITS CONTENTS IS :                         *   FILE 159
//*          - 100 BYTE WORK AREA WHICH MUST BE INITIALIZED TO      *   FILE 159
//*              BINARY ZEROS 'ONLY' ON THE FIRST CALL TO           *   FILE 159
//*              THIS SUBROUTINE FOR A SPECIFIC FUNCTION.  FOR      *   FILE 159
//*              SUBSEQUENT ACCESSES 'OF SAME' FUNCTION, DO         *   FILE 159
//*              'NOT' TOUCH THE CONTENTS OF THIS WORK AREA.        *   FILE 159
//*          - 4 BYTES (1 FULLWORD) FOR RETURNED COUNT OF # OF      *   FILE 159
//*               UCBS BEING RETURNED. THIS SHOULD BE               *   FILE 159
//*               INITIALIZED TO ZEROS                              *   FILE 159
//*          - 1944 BYTES (486 FULLWORDS) FOR RETURNED UCB          *   FILE 159
//*              ADDRESSES.  THIS SHOULD BE INITIALIZED TO          *   FILE 159
//*              ZEROS.                                             *   FILE 159
//*                                                                 *   FILE 159
//*       RETURN CODE SETTINGS:                                     *   FILE 159
//*       R15 = 00 - ALL UCBS RETURNED                              *   FILE 159
//*       R15 = 04 - NOT ALL UCBS RETURNED, MUST RETURN FOR         *   FILE 159
//*                  THE REST                                       *   FILE 159
//*       R15 = 08 - NO UCBS FOUND                                  *   FILE 159
//*       R15 = 16 - FUNCTION NOT SUPPORTED                         *   FILE 159
//*       R15 = 20 - STORAGE NOT AVAIL TO IEFAB4UV FOR UCB          *   FILE 159
//*                  LIST                                           *   FILE 159
//*       R15 = 24 - DEVICE TYPE NOT DEFINED TO SYSTEM              *   FILE 159
//*       R15 = 28 - NOT ALL UCBS RETURNED, RECODE FOR MORE         *   FILE 159
//*                  THAT 486 UCBS                                  *   FILE 159
//*       R15 = 99 - PROBLEM - ABEND PROGRAM                        *   FILE 159
//*                                                                 *   FILE 159
```
