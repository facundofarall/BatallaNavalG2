################################################################################
#                                                                              #
#    Micro Series 6801 Assembler V2.00/DOS              27/Nov/17  23:20:31    #
#                                                                              #
#       Source   =   main_bn.msa                                               #
#       List     =   main_bn.lst                                               #
#       Object   =   main_bn.r07                                               #
#       Options  =                                                             #
#                                                                              #
#                                               (c) Copyright IAR Systems 1990 #
################################################################################


       0000                           p68h11
                      $macros.msa
                      **********************************
                      * definicion de macro utilizados *
                      **********************************
                      * invocar a la subrutina writepos
       0000                   MACRO   %WRITEPOS
                                      ldab            \0
                                      pshb                            ;se envía el byte a escribir
                                      ldab            12,x
                                      pshb                            ;se envía letra (fila)
                                      ldab            11,x
                                      pshb                            ;se envía num (columna)
                                      ldy             9,x
                                      pshy                            ;se envía dirección de inicio de matriz
                                      ldab            8,x
                                      pshb                            ;se envía jugador atacado
                                      jsr             writepos
                                      %CLRST          6
       0000                   ENDMAC
                      
                      * invocar a la subrutina writepos
       0000                   MACRO   %PLAY_WRT
                                      ldab            12,x
                                      pshb                            ;se envía letra (fila)
                                      ldab            11,x
                                      pshb                            ;se envía num (columna)
                                      ldy             14,x
                                      pshy                            ;se envía inicio de array
                                      ldab            8,x
                                      pshb                            ;se envía jugador atacado
                                      jsr             play_wrt
                                      %CLRST          5
       0000                   ENDMAC
                      
                      * realizar back up de AccA, AccB, IX e IY
       0000                   MACRO   %BACKUP
                                      psha
                                      pshb
                                      pshx
                                      pshy
       0000                   ENDMAC
                      
                      * realizar restore de AccA, AccB, IX e IY
       0000                   MACRO   %RESTORE
                                      puly
                                      pulx
                                      pulb
                                      pula
       0000                   ENDMAC
                      
                      * limpiar el stack de datos enviados
       0000                   MACRO   %CLRST
                      CONT_M          SET     \0
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
                                      ENDIF
       0000                   ENDMAC
                      * activar un led en funcion de una variable
       0000                   MACRO   %LED_INV
                                      ldaa            PORTA
                                      eora            #SHF_LED
                                      staa            PORTA
       0000                   ENDMAC
                      
                      * declarar o reservar variables locales en subrutinas
       0000                   MACRO   %MALLOC
                                      IF \0 > 0
                                      tsx
                                      xgdx
                                      subd    #\0
                                      xgdx
                                      txs
                                      ENDIF
       0000                   ENDMAC
                      
                      * liberar espacio de variables en subrutinas
       0000                   MACRO   %RELMEM
                                      IF \0 > 0
                                      tsx
                                      xgdx
                                      addd    #\0
                                      xgdx
                                      txs
                                      ENDIF           
       0000                   ENDMAC
                      
                      *************
                      * CONSTANTS *
                      *************
       0000           P1              EQU             0
       0001           P2              EQU             1
       0002           PN              EQU             2               ;cantidad de jugadores
                      
       0006           MAX_C           EQU             6
       0001           MIN_C           EQU             1
       000F           MAX_F           EQU             15
       000A           MIN_F           EQU             10              
                      
       0003           WAT_NP          EQU             00000011b
       0043           SUB_NP          EQU             01000011b
       0083           SHI_NP          EQU             10000011b
       0008           WATER           EQU             00001000b       ;verde
       0050           SUNK_SUB        EQU             01010000b       ;bordeaux para submarinos
       00B0           HIT             EQU             10110000b       ;naranja
       0090           SUNK_SHI        EQU             10010000b       ;bordeaux para cargueros
                      
       0006           FILA            EQU             6
       0006           COL             EQU             6
       0024           ARRAY_L         EQU             FILA*COL
       000A           NORM            EQU             10
                      
       000A           MIN_FIL         EQU             10
       000F           MAX_FIL         EQU             15
       0001           MIN_COL         EQU             1
       0006           MAX_COL         EQU             6       
                      
       0000           CONT            EQU             0
       0002           FILA_L          EQU             2
       0001           COL_L           EQU             1
                      
       0003           INPUTS          EQU             3
                      
       FFFF           NONE            EQU             -1
       0003           MASK_FIL        EQU             3
       0002           MASK_COL        EQU             2
       0001           COL_CONT        EQU             1
       0000           KB_PRESS        EQU             0
                      
                      ******* Ports ***************************
                      
       1007           DDRC            EQU             $1007           ;data direction register for C
       1002           PIOC            EQU             $1002           ;for strobe B assertion change
       1005           PORTCL          EQU             $1005           ;port C latched
       1004           PORTB           EQU             $1004           ;port B
       1003           PORTC           EQU             $1003           ;rows
       1000           PORTA           EQU             $1000
                      
       0040           SHF_LED         EQU             $40
                      
                      ******* Cols and Row Ports **************
                      
       1003           ROWS            EQU             PORTC           ;outputs
       1005           COLS            EQU             PORTCL          ;inputs
                      
                      ***** Col Mask **************************
       0010           COL0            EQU             00010000b
                               
                      ***** Row Masks *************************
                      
       000E           FIL0            EQU             00001110b       ;mascaras para manejo de filas
       000D           FIL1            EQU             00001101b
       000B           FIL2            EQU             00001011b
       0007           FIL3            EQU             00000111b
                      
       0004           LENGTH          EQU             4               ; largo del display
                      
                      ****** Keyboard Mask ********************
       0008           Button_1        EQU             8
       0009           Button_2        EQU             9
       000A           Button_3        EQU             10
       000C           Button_4        EQU             12
       000D           Button_5        EQU             13
       000E           Button_6        EQU             14              
       0000           Button_A        EQU             0       
       0001           Button_B        EQU             1
       0002           Button_C        EQU             2
       0004           Button_D        EQU             4
       0005           Button_E        EQU             5
       0006           Button_F        EQU             6
       000B           B_fire          EQU             11
       0003           B_P1            EQU             3
       0007           B_P2            EQU             7
       000F           B_reset         EQU             15
                      
       0000           EOT             EQU             0
       0020           ESP             EQU             $20     
                      
                      *========================================================
                      * El siguiente es un programa de un juego de batalla
                      * naval para dos jugadores en el emulador Wookie FX.
                      *========================================================
                      
                      ****************
                      * MAIN PROGRAM *
                      ****************
                      
       2000                           ORG             $2000
       2000           init            EQU             *
       2000 8E41F3                    lds             #STACKP
                      
                      * INICIALIZAR DISPLAY Y TECLADO
       2003 BD2606                    jsr             initdis         ;inicialización de display 7 seg
       2006 BD25FE                    jsr             init_kb         ;inicialización de teclado
                      
                      * LIMPIAR ARREGLO DE BARCOS ATACADOS
       2009 CE4220                    ldx             #hitlist
       200C 3C                        pshx                            ;se envía inicio de arreglo de barcos a poner en 0
       200D 8636                      ldaa            #(HITSIZE*2)
       200F 36                        psha                            ;se envía el tamaño del arreglo a limpiar
       2010 BD24A0                    jsr             clrzone
       2013                           %CLRST          3
       0003           CONT_M          SET     3
       2013                           IF CONT_M > 0
       2013 31                        ins
       2014                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2014                           IF CONT_M > 0
       2014 31                        ins
       2015                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2015                           IF CONT_M > 0
       2015 31                        ins
       2016                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2016                           ENDIF
       2016                           ENDIF
       2016                           ENDIF
       2016                           ENDIF
                      
                      * DEFINIR CANTIDAD DE BARCOS A UTILIZAR
       2016 BD2345                    jsr             def_ships
       2019 B641F7                    ldaa            cant_sub        ;se carga cantidad de submarinos
       201C F641F8                    ldab            cant_shi        ;se carga cantidad de cargueros
       201F 58                        lslb                            ;se multiplica a la cantidad de cargueros por dos, así se obtiene las posiciones atacables
       2020 1B                        aba
       2021 B741F9                    staa            tot_pos         ;se carga la variable con el total de posiciones atacables
                                                      
                      
       2024           main            EQU             *
                      * ELEGIR JUGADOR
       2024 BD24B8                    jsr             whoplays        ;se recibirá en AccA el jugador a atacar a continuación
       2027 B741F4                    staa            player          ;se habilita la siguiente jugada
                      
                      * ENVIAR TABLERO DE JUGADOR A ATACAR A DISPLAY
       202A CE5000                    ldx             #p_board
       202D 3C                        pshx                            ;se envía el inicio de las matrices de tableros
       202E B641F4                    ldaa            player
       2031 36                        psha                            ;se envía el jugador atacado
       2032 CE5F00                    ldx             #display
       2035 3C                        pshx                            ;se envía la matriz de display (destino a copiar)
       2036 8624                      ldaa            #ARRAY_L
       2038 36                        psha                            ;se envía el tamaño de la matriz tablero
       2039 BD2394                    jsr             mirror          ;se envía el tablero del jugador atacado al display
       203C                           %CLRST          6
       0006           CONT_M          SET     6
       203C                           IF CONT_M > 0
       203C 31                        ins
       203D                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       203D                           IF CONT_M > 0
       203D 31                        ins
       203E                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       203E                           IF CONT_M > 0
       203E 31                        ins
       203F                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       203F                           IF CONT_M > 0
       203F 31                        ins
       2040                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2040                           IF CONT_M > 0
       2040 31                        ins
       2041                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2041                           IF CONT_M > 0
       2041 31                        ins
       2042                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2042                           ENDIF
       2042                           ENDIF
       2042                           ENDIF
       2042                           ENDIF
       2042                           ENDIF
       2042                           ENDIF
       2042                           ENDIF
                      
                      * INGRESAR POSICIÓN A ATACAR
       2042 BD24CD                    jsr             input           ;se llama a la subrutina de interfaz de usuario
       2045 B741F5                    staa            letra_p         ;se recibe en AccA la letra jugada (fila) y en AccB el número (columna)
       2048 F741F6                    stab            num_p           ;se guarda la jugada
                      
                      * VERIFICAR QUE NO SEA POSICIÓN DE BARCO YA ATACADA
       204B B641F5                    ldaa            letra_p
       204E 36                        psha                            ;se envía la letra jugada
       204F F641F6                    ldab            num_p
       2052 37                        pshb                            ;se envía el número jugado
       2053 CE4220                    ldx             #hitlist
       2056 3C                        pshx                            ;se envía la posición del primer arreglo de barcos tocados
       2057 B641F4                    ldaa            player
       205A 36                        psha                            ;se envía el jugador a atacar
       205B BD241F                    jsr             play_src        ;se recibirá el carry en 1 si la posición corresponde a un barco ya atacado
       205E                           %CLRST          5               ;se limpia el stack
       0005           CONT_M          SET     5
       205E                           IF CONT_M > 0
       205E 31                        ins
       205F                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       205F                           IF CONT_M > 0
       205F 31                        ins
       2060                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2060                           IF CONT_M > 0
       2060 31                        ins
       2061                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2061                           IF CONT_M > 0
       2061 31                        ins
       2062                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2062                           IF CONT_M > 0
       2062 31                        ins
       2063                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2063                           ENDIF
       2063                           ENDIF
       2063                           ENDIF
       2063                           ENDIF
       2063                           ENDIF
       2063                           ENDIF
       2063 25BF                      bcs             main            ;si la posición ya fue jugada, se pierde el turno y pasa al siguiente
                      
                      * MANDAR QUE JUGADOR ESTÁ JUGANDO AL DISPLAY
       2065 B641F4                    ldaa            player
       2068 36                        psha                            ;se envía el jugador que está siendo atacado
       2069 BD2670                    jsr             turn
       206C                           %CLRST          1
       0001           CONT_M          SET     1
       206C                           IF CONT_M > 0
       206C 31                        ins
       206D                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       206D                           ENDIF
       206D                           ENDIF
                      
                      * BUSCAR EL BYTE ATACADO EN EL TABLERO DEL JUGADOR ATACADO
       206D B641F5                    ldaa            letra_p
       2070 36                        psha                            ;se envía le letra jugada
       2071 F641F6                    ldab            num_p
       2074 37                        pshb                            ;se envía el número jugado
       2075 CE5000                    ldx             #p_board
       2078 3C                        pshx                            ;se envía la dirección de inicio de los arreglos de tablero
       2079 F641F4                    ldab            player
       207C 37                        pshb                            ;se envía el jugador a atacar
       207D BD23BD                    jsr             fetchpos
       2080                           %CLRST          4               ;se limpia el stack
       0004           CONT_M          SET     4
       2080                           IF CONT_M > 0
       2080 31                        ins
       2081                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2081                           IF CONT_M > 0
       2081 31                        ins
       2082                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2082                           IF CONT_M > 0
       2082 31                        ins
       2083                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2083                           IF CONT_M > 0
       2083 31                        ins
       2084                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2084                           ENDIF
       2084                           ENDIF
       2084                           ENDIF
       2084                           ENDIF
       2084                           ENDIF
       2084 32                        pula                            ;se toma del stack el dato del byte jugado
                      
                      * ATACAR
       2085 CE4220                    ldx             #hitlist
       2088 3C                        pshx                            ;se envía inicio del arreglo de barcos atacados
       2089 36                        psha                            ;se envía el byte a jugado
       208A B641F5                    ldaa            letra_p
       208D 36                        psha                            ;se envía le letra jugada
       208E F641F6                    ldab            num_p
       2091 37                        pshb                            ;se envía el número jugado
       2092 CE5000                    ldx             #p_board
       2095 3C                        pshx                            ;se envía la dirección de inicio de los arreglos de tablero
       2096 F641F4                    ldab            player
       2099 37                        pshb                            ;se envía el jugador a atacar
       209A BD20DD                    jsr             gameplay
       209D                           %CLRST          8               ;se limpia el stack
       0008           CONT_M          SET     8
       209D                           IF CONT_M > 0
       209D 31                        ins
       209E                           %CLRST  CONT_M-1
       0007           CONT_M          SET     CONT_M-1
       209E                           IF CONT_M > 0
       209E 31                        ins
       209F                           %CLRST  CONT_M-1
       0006           CONT_M          SET     CONT_M-1
       209F                           IF CONT_M > 0
       209F 31                        ins
       20A0                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       20A0                           IF CONT_M > 0
       20A0 31                        ins
       20A1                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       20A1                           IF CONT_M > 0
       20A1 31                        ins
       20A2                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       20A2                           IF CONT_M > 0
       20A2 31                        ins
       20A3                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       20A3                           IF CONT_M > 0
       20A3 31                        ins
       20A4                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       20A4                           IF CONT_M > 0
       20A4 31                        ins
       20A5                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       20A5                           ENDIF
       20A5                           ENDIF
       20A5                           ENDIF
       20A5                           ENDIF
       20A5                           ENDIF
       20A5                           ENDIF
       20A5                           ENDIF
       20A5                           ENDIF
       20A5                           ENDIF
                      
                      * ACTUALIZAR DISPLAY CON LA POSICIÓN YA ATACADA
       20A5 CE5000                    ldx             #p_board
       20A8 3C                        pshx                            ;se envía el inicio de las matrices de tableros
       20A9 B641F4                    ldaa            player
       20AC 36                        psha                            ;se envía el jugador atacado
       20AD CE5F00                    ldx             #display
       20B0 3C                        pshx                            ;se envía la matriz de display (destino a copiar)
       20B1 8624                      ldaa            #ARRAY_L
       20B3 36                        psha                            ;se envía el tamaño de la matriz tablero
       20B4 BD2394                    jsr             mirror          ;se envía el tablero del jugador atacado al display
       20B7                           %CLRST          6
       0006           CONT_M          SET     6
       20B7                           IF CONT_M > 0
       20B7 31                        ins
       20B8                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       20B8                           IF CONT_M > 0
       20B8 31                        ins
       20B9                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       20B9                           IF CONT_M > 0
       20B9 31                        ins
       20BA                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       20BA                           IF CONT_M > 0
       20BA 31                        ins
       20BB                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       20BB                           IF CONT_M > 0
       20BB 31                        ins
       20BC                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       20BC                           IF CONT_M > 0
       20BC 31                        ins
       20BD                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       20BD                           ENDIF
       20BD                           ENDIF
       20BD                           ENDIF
       20BD                           ENDIF
       20BD                           ENDIF
       20BD                           ENDIF
       20BD                           ENDIF
                      
                      * CONTAR BARCOS ATACADOS A VER SI EL JUGADOR GANÓ               
       20BD CE4220                    ldx             #hitlist
       20C0 3C                        pshx                            ;se envía la posición del primer arreglo de barcos tocados
       20C1 B641F4                    ldaa            player
       20C4 36                        psha                            ;se envía el jugador a atacar
       20C5 BD2452                    jsr             play_cnt
       20C8                           %CLRST          2
       0002           CONT_M          SET     2
       20C8                           IF CONT_M > 0
       20C8 31                        ins
       20C9                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       20C9                           IF CONT_M > 0
       20C9 31                        ins
       20CA                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       20CA                           ENDIF
       20CA                           ENDIF
       20CA                           ENDIF
       20CA 32                        pula                            ;se recibe la cantidad de bytes contados
       20CB B141F9                    cmpa            tot_pos         
       20CE 2703                      beq             end_game        ;si se llegó al máximo de barcos, se termina el juego
                      
       20D0 7E2024                    jmp             main
                      
                      * MANDAR MENSAJE A DISPLAY DE QUIÉN GANÓ
       20D3           end_game        EQU             *
       20D3 B641F4                    ldaa            player
       20D6 36                        psha
       20D7 BD2652                    jsr             winner
       20DA                           %CLRST          1
       0001           CONT_M          SET     1
       20DA                           IF CONT_M > 0
       20DA 31                        ins
       20DB                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       20DB                           ENDIF
       20DB                           ENDIF
                      
       20DB           end_loop        EQU             *
       20DB 20FE                      bra             end_loop
                      
                      **************
                      * SUBRUTINES *
                      **************
                      *=======================================================================
                      * subrutina gameplay
                      * Función: Determina si la posición jugada en el juego de batalla naval
                      *          para Wookie FX fue "agua", "submarion" o "carguero", y toma 
                      *          la decisión de qué hacer al respecto.
                      * Recibe: - Dirección arreglo auxiliar de barcos
                      *         - Byte a jugar.
                      *         - FILA del array (En HEX, de $0A (fil.0) en adelante)
                      *         - COLUMNA del array (en HEX)
                      *         - Dirección de comienzo del arreglo (2 bytes)
                      *         - Número de arreglo (de 0 en adelante) (número de jugador)
                      *       TODO POR STACK.
                      * Devuelve: parámetro jugado en posición correspondiente.
                      * Requiere: 
                      *=======================================================================
       20DD           gameplay        EQU             *
       20DD                           %BACKUP
       20DD 36                        psha
       20DE 37                        pshb
       20DF 3C                        pshx
       20E0 183C                      pshy
                                      
       20E2 30                        tsx                             ;se usa el IX como frame pointer
       20E3 A60D                      ldaa            13,x            ;se carga en AccA el byte a analizar
       20E5 84C0                      anda            #11000000b      ;se enmascan los bits 0 a 5, para analizar los más significativos
                      
       20E7           gp_water        EQU             *
       20E7 2620                      bne             gp_sub          ;se estudia si era agua
       20E9                           %WRITEPOS       #WATER          ;se cambia el color en el tablero
       20E9 C608                      ldab            #WATER
       20EB 37                        pshb                            ;se envía el byte a escribir
       20EC E60C                      ldab            12,x
       20EE 37                        pshb                            ;se envía letra (fila)
       20EF E60B                      ldab            11,x
       20F1 37                        pshb                            ;se envía num (columna)
       20F2 1AEE09                    ldy             9,x
       20F5 183C                      pshy                            ;se envía dirección de inicio de matriz
       20F7 E608                      ldab            8,x
       20F9 37                        pshb                            ;se envía jugador atacado
       20FA BD23EE                    jsr             writepos
       20FD                           %CLRST          6
       0006           CONT_M          SET     6
       20FD                           IF CONT_M > 0
       20FD 31                        ins
       20FE                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       20FE                           IF CONT_M > 0
       20FE 31                        ins
       20FF                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       20FF                           IF CONT_M > 0
       20FF 31                        ins
       2100                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2100                           IF CONT_M > 0
       2100 31                        ins
       2101                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2101                           IF CONT_M > 0
       2101 31                        ins
       2102                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2102                           IF CONT_M > 0
       2102 31                        ins
       2103                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2103                           ENDIF
       2103                           ENDIF
       2103                           ENDIF
       2103                           ENDIF
       2103                           ENDIF
       2103                           ENDIF
       2103                           ENDIF
       2103                           %RESTORE
       2103 1838                      puly
       2105 38                        pulx
       2106 33                        pulb
       2107 32                        pula
       2108 39                        rts
                      
       2109           gp_sub          EQU             *
       2109 8140                      cmpa            #01000000b      ;se estudia si era un submarino
       210B 2636                      bne             gp_shi_r
       210D                           %WRITEPOS       #SUNK_SUB       ;se cambia el color en el tablero
       210D C650                      ldab            #SUNK_SUB
       210F 37                        pshb                            ;se envía el byte a escribir
       2110 E60C                      ldab            12,x
       2112 37                        pshb                            ;se envía letra (fila)
       2113 E60B                      ldab            11,x
       2115 37                        pshb                            ;se envía num (columna)
       2116 1AEE09                    ldy             9,x
       2119 183C                      pshy                            ;se envía dirección de inicio de matriz
       211B E608                      ldab            8,x
       211D 37                        pshb                            ;se envía jugador atacado
       211E BD23EE                    jsr             writepos
       2121                           %CLRST          6
       0006           CONT_M          SET     6
       2121                           IF CONT_M > 0
       2121 31                        ins
       2122                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2122                           IF CONT_M > 0
       2122 31                        ins
       2123                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2123                           IF CONT_M > 0
       2123 31                        ins
       2124                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2124                           IF CONT_M > 0
       2124 31                        ins
       2125                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2125                           IF CONT_M > 0
       2125 31                        ins
       2126                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2126                           IF CONT_M > 0
       2126 31                        ins
       2127                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2127                           ENDIF
       2127                           ENDIF
       2127                           ENDIF
       2127                           ENDIF
       2127                           ENDIF
       2127                           ENDIF
       2127                           ENDIF
       2127                           %PLAY_WRT                       ;se agraga la posición al array de barcos atacados
       2127 E60C                      ldab            12,x
       2129 37                        pshb                            ;se envía letra (fila)
       212A E60B                      ldab            11,x
       212C 37                        pshb                            ;se envía num (columna)
       212D 1AEE0E                    ldy             14,x
       2130 183C                      pshy                            ;se envía inicio de array
       2132 E608                      ldab            8,x
       2134 37                        pshb                            ;se envía jugador atacado
       2135 BD2476                    jsr             play_wrt
       2138                           %CLRST          5
       0005           CONT_M          SET     5
       2138                           IF CONT_M > 0
       2138 31                        ins
       2139                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2139                           IF CONT_M > 0
       2139 31                        ins
       213A                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       213A                           IF CONT_M > 0
       213A 31                        ins
       213B                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       213B                           IF CONT_M > 0
       213B 31                        ins
       213C                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       213C                           IF CONT_M > 0
       213C 31                        ins
       213D                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       213D                           ENDIF
       213D                           ENDIF
       213D                           ENDIF
       213D                           ENDIF
       213D                           ENDIF
       213D                           ENDIF
       213D                           %RESTORE
       213D 1838                      puly
       213F 38                        pulx
       2140 33                        pulb
       2141 32                        pula
       2142 39                        rts
                      
       2143           gp_shi_r        EQU             *               ;si no era ninguno de los anteriores, es un carguero
       2143 E60B                      ldab            11,x
       2145 C106                      cmpb            #MAX_C
       2147 276D                      beq             gp_shi_l        ;si está en el borde derecho, se continúa
       2149 E60C                      ldab            12,x
       214B 37                        pshb                            ;se envía la letra (fila)
       214C E60B                      ldab            11,x
       214E 5C                        incb                            ;se avanza a la derecha
       214F 37                        pshb                            ;se envía el num (columna)
       2150 1AEE0E                    ldy             14,x
       2153 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       2155 E608                      ldab            8,x
       2157 37                        pshb                            ;se envía el jugador atacado
       2158 BD241F                    jsr             play_src        ;se busca la posición de la derecha
       215B                           %CLRST          5
       0005           CONT_M          SET     5
       215B                           IF CONT_M > 0
       215B 31                        ins
       215C                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       215C                           IF CONT_M > 0
       215C 31                        ins
       215D                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       215D                           IF CONT_M > 0
       215D 31                        ins
       215E                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       215E                           IF CONT_M > 0
       215E 31                        ins
       215F                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       215F                           IF CONT_M > 0
       215F 31                        ins
       2160                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2160                           ENDIF
       2160                           ENDIF
       2160                           ENDIF
       2160                           ENDIF
       2160                           ENDIF
       2160                           ENDIF
       2160 2454                      bcc             gp_shi_l        ;si no había barco atacado en la derecha, se busca la izquierda
       2162                           %WRITEPOS       #SUNK_SHI       
       2162 C690                      ldab            #SUNK_SHI
       2164 37                        pshb                            ;se envía el byte a escribir
       2165 E60C                      ldab            12,x
       2167 37                        pshb                            ;se envía letra (fila)
       2168 E60B                      ldab            11,x
       216A 37                        pshb                            ;se envía num (columna)
       216B 1AEE09                    ldy             9,x
       216E 183C                      pshy                            ;se envía dirección de inicio de matriz
       2170 E608                      ldab            8,x
       2172 37                        pshb                            ;se envía jugador atacado
       2173 BD23EE                    jsr             writepos
       2176                           %CLRST          6
       0006           CONT_M          SET     6
       2176                           IF CONT_M > 0
       2176 31                        ins
       2177                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2177                           IF CONT_M > 0
       2177 31                        ins
       2178                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2178                           IF CONT_M > 0
       2178 31                        ins
       2179                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2179                           IF CONT_M > 0
       2179 31                        ins
       217A                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       217A                           IF CONT_M > 0
       217A 31                        ins
       217B                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       217B                           IF CONT_M > 0
       217B 31                        ins
       217C                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       217C                           ENDIF
       217C                           ENDIF
       217C                           ENDIF
       217C                           ENDIF
       217C                           ENDIF
       217C                           ENDIF
       217C                           ENDIF
       217C 6C0B                      inc             11,x
       217E                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       217E C690                      ldab            #SUNK_SHI
       2180 37                        pshb                            ;se envía el byte a escribir
       2181 E60C                      ldab            12,x
       2183 37                        pshb                            ;se envía letra (fila)
       2184 E60B                      ldab            11,x
       2186 37                        pshb                            ;se envía num (columna)
       2187 1AEE09                    ldy             9,x
       218A 183C                      pshy                            ;se envía dirección de inicio de matriz
       218C E608                      ldab            8,x
       218E 37                        pshb                            ;se envía jugador atacado
       218F BD23EE                    jsr             writepos
       2192                           %CLRST          6
       0006           CONT_M          SET     6
       2192                           IF CONT_M > 0
       2192 31                        ins
       2193                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2193                           IF CONT_M > 0
       2193 31                        ins
       2194                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2194                           IF CONT_M > 0
       2194 31                        ins
       2195                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2195                           IF CONT_M > 0
       2195 31                        ins
       2196                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2196                           IF CONT_M > 0
       2196 31                        ins
       2197                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2197                           IF CONT_M > 0
       2197 31                        ins
       2198                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2198                           ENDIF
       2198                           ENDIF
       2198                           ENDIF
       2198                           ENDIF
       2198                           ENDIF
       2198                           ENDIF
       2198                           ENDIF
       2198 6A0B                      dec             11,x            ;se recupera el dato
       219A                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       219A E60C                      ldab            12,x
       219C 37                        pshb                            ;se envía letra (fila)
       219D E60B                      ldab            11,x
       219F 37                        pshb                            ;se envía num (columna)
       21A0 1AEE0E                    ldy             14,x
       21A3 183C                      pshy                            ;se envía inicio de array
       21A5 E608                      ldab            8,x
       21A7 37                        pshb                            ;se envía jugador atacado
       21A8 BD2476                    jsr             play_wrt
       21AB                           %CLRST          5
       0005           CONT_M          SET     5
       21AB                           IF CONT_M > 0
       21AB 31                        ins
       21AC                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       21AC                           IF CONT_M > 0
       21AC 31                        ins
       21AD                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       21AD                           IF CONT_M > 0
       21AD 31                        ins
       21AE                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       21AE                           IF CONT_M > 0
       21AE 31                        ins
       21AF                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       21AF                           IF CONT_M > 0
       21AF 31                        ins
       21B0                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       21B0                           ENDIF
       21B0                           ENDIF
       21B0                           ENDIF
       21B0                           ENDIF
       21B0                           ENDIF
       21B0                           ENDIF
       21B0                           %RESTORE
       21B0 1838                      puly
       21B2 38                        pulx
       21B3 33                        pulb
       21B4 32                        pula
       21B5 39                        rts
                      
       21B6           gp_shi_l        EQU             *
       21B6 E60B                      ldab            11,x
       21B8 C101                      cmpb            #MIN_C
       21BA 276D                      beq             gp_shi_u        ;si está en el borde izquierdo, se continúa
       21BC E60C                      ldab            12,x
       21BE 37                        pshb                            ;se envía la letra (fila)
       21BF E60B                      ldab            11,x
       21C1 5A                        decb                            ;se retrocede a la izquierda
       21C2 37                        pshb                            ;se envía el número (columna)
       21C3 1AEE0E                    ldy             14,x
       21C6 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       21C8 E608                      ldab            8,x
       21CA 37                        pshb                            ;se envía el jugador atacado
       21CB BD241F                    jsr             play_src        ;se busca la posición de la izquierda
       21CE                           %CLRST          5
       0005           CONT_M          SET     5
       21CE                           IF CONT_M > 0
       21CE 31                        ins
       21CF                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       21CF                           IF CONT_M > 0
       21CF 31                        ins
       21D0                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       21D0                           IF CONT_M > 0
       21D0 31                        ins
       21D1                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       21D1                           IF CONT_M > 0
       21D1 31                        ins
       21D2                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       21D2                           IF CONT_M > 0
       21D2 31                        ins
       21D3                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       21D3                           ENDIF
       21D3                           ENDIF
       21D3                           ENDIF
       21D3                           ENDIF
       21D3                           ENDIF
       21D3                           ENDIF
       21D3 2454                      bcc             gp_shi_u        ;si no había barco atacado a la izquierda, se busca arriba
       21D5                           %WRITEPOS       #SUNK_SHI       
       21D5 C690                      ldab            #SUNK_SHI
       21D7 37                        pshb                            ;se envía el byte a escribir
       21D8 E60C                      ldab            12,x
       21DA 37                        pshb                            ;se envía letra (fila)
       21DB E60B                      ldab            11,x
       21DD 37                        pshb                            ;se envía num (columna)
       21DE 1AEE09                    ldy             9,x
       21E1 183C                      pshy                            ;se envía dirección de inicio de matriz
       21E3 E608                      ldab            8,x
       21E5 37                        pshb                            ;se envía jugador atacado
       21E6 BD23EE                    jsr             writepos
       21E9                           %CLRST          6
       0006           CONT_M          SET     6
       21E9                           IF CONT_M > 0
       21E9 31                        ins
       21EA                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       21EA                           IF CONT_M > 0
       21EA 31                        ins
       21EB                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       21EB                           IF CONT_M > 0
       21EB 31                        ins
       21EC                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       21EC                           IF CONT_M > 0
       21EC 31                        ins
       21ED                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       21ED                           IF CONT_M > 0
       21ED 31                        ins
       21EE                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       21EE                           IF CONT_M > 0
       21EE 31                        ins
       21EF                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       21EF                           ENDIF
       21EF                           ENDIF
       21EF                           ENDIF
       21EF                           ENDIF
       21EF                           ENDIF
       21EF                           ENDIF
       21EF                           ENDIF
       21EF 6A0B                      dec             11,x
       21F1                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       21F1 C690                      ldab            #SUNK_SHI
       21F3 37                        pshb                            ;se envía el byte a escribir
       21F4 E60C                      ldab            12,x
       21F6 37                        pshb                            ;se envía letra (fila)
       21F7 E60B                      ldab            11,x
       21F9 37                        pshb                            ;se envía num (columna)
       21FA 1AEE09                    ldy             9,x
       21FD 183C                      pshy                            ;se envía dirección de inicio de matriz
       21FF E608                      ldab            8,x
       2201 37                        pshb                            ;se envía jugador atacado
       2202 BD23EE                    jsr             writepos
       2205                           %CLRST          6
       0006           CONT_M          SET     6
       2205                           IF CONT_M > 0
       2205 31                        ins
       2206                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2206                           IF CONT_M > 0
       2206 31                        ins
       2207                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2207                           IF CONT_M > 0
       2207 31                        ins
       2208                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2208                           IF CONT_M > 0
       2208 31                        ins
       2209                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2209                           IF CONT_M > 0
       2209 31                        ins
       220A                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       220A                           IF CONT_M > 0
       220A 31                        ins
       220B                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       220B                           ENDIF
       220B                           ENDIF
       220B                           ENDIF
       220B                           ENDIF
       220B                           ENDIF
       220B                           ENDIF
       220B                           ENDIF
       220B 6C0B                      inc             11,x            ;se recupera el dato
       220D                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       220D E60C                      ldab            12,x
       220F 37                        pshb                            ;se envía letra (fila)
       2210 E60B                      ldab            11,x
       2212 37                        pshb                            ;se envía num (columna)
       2213 1AEE0E                    ldy             14,x
       2216 183C                      pshy                            ;se envía inicio de array
       2218 E608                      ldab            8,x
       221A 37                        pshb                            ;se envía jugador atacado
       221B BD2476                    jsr             play_wrt
       221E                           %CLRST          5
       0005           CONT_M          SET     5
       221E                           IF CONT_M > 0
       221E 31                        ins
       221F                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       221F                           IF CONT_M > 0
       221F 31                        ins
       2220                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2220                           IF CONT_M > 0
       2220 31                        ins
       2221                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2221                           IF CONT_M > 0
       2221 31                        ins
       2222                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2222                           IF CONT_M > 0
       2222 31                        ins
       2223                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2223                           ENDIF
       2223                           ENDIF
       2223                           ENDIF
       2223                           ENDIF
       2223                           ENDIF
       2223                           ENDIF
       2223                           %RESTORE
       2223 1838                      puly
       2225 38                        pulx
       2226 33                        pulb
       2227 32                        pula
       2228 39                        rts
                      
       2229           gp_shi_u        EQU             *
       2229 E60C                      ldab            12,x
       222B C10A                      cmpb            #MIN_F
       222D 276D                      beq             gp_shi_d        ;si está en el borde superior, se continúa
       222F E60C                      ldab            12,x
       2231 5A                        decb                            ;se va a la posición de arriba
       2232 37                        pshb                            ;se envía la letra (fila)
       2233 E60B                      ldab            11,x
       2235 37                        pshb                            ;se envía el num (columna)
       2236 1AEE0E                    ldy             14,x
       2239 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       223B E608                      ldab            8,x
       223D 37                        pshb                            ;se envía el jugador atacado
       223E BD241F                    jsr             play_src        ;se busca la posición de arriba
       2241                           %CLRST          5
       0005           CONT_M          SET     5
       2241                           IF CONT_M > 0
       2241 31                        ins
       2242                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2242                           IF CONT_M > 0
       2242 31                        ins
       2243                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2243                           IF CONT_M > 0
       2243 31                        ins
       2244                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2244                           IF CONT_M > 0
       2244 31                        ins
       2245                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2245                           IF CONT_M > 0
       2245 31                        ins
       2246                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2246                           ENDIF
       2246                           ENDIF
       2246                           ENDIF
       2246                           ENDIF
       2246                           ENDIF
       2246                           ENDIF
       2246 2454                      bcc             gp_shi_d        ;si no había barco atacado arriba, se busca abajo
       2248                           %WRITEPOS       #SUNK_SHI       
       2248 C690                      ldab            #SUNK_SHI
       224A 37                        pshb                            ;se envía el byte a escribir
       224B E60C                      ldab            12,x
       224D 37                        pshb                            ;se envía letra (fila)
       224E E60B                      ldab            11,x
       2250 37                        pshb                            ;se envía num (columna)
       2251 1AEE09                    ldy             9,x
       2254 183C                      pshy                            ;se envía dirección de inicio de matriz
       2256 E608                      ldab            8,x
       2258 37                        pshb                            ;se envía jugador atacado
       2259 BD23EE                    jsr             writepos
       225C                           %CLRST          6
       0006           CONT_M          SET     6
       225C                           IF CONT_M > 0
       225C 31                        ins
       225D                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       225D                           IF CONT_M > 0
       225D 31                        ins
       225E                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       225E                           IF CONT_M > 0
       225E 31                        ins
       225F                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       225F                           IF CONT_M > 0
       225F 31                        ins
       2260                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2260                           IF CONT_M > 0
       2260 31                        ins
       2261                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2261                           IF CONT_M > 0
       2261 31                        ins
       2262                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2262                           ENDIF
       2262                           ENDIF
       2262                           ENDIF
       2262                           ENDIF
       2262                           ENDIF
       2262                           ENDIF
       2262                           ENDIF
       2262 6A0C                      dec             12,x
       2264                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       2264 C690                      ldab            #SUNK_SHI
       2266 37                        pshb                            ;se envía el byte a escribir
       2267 E60C                      ldab            12,x
       2269 37                        pshb                            ;se envía letra (fila)
       226A E60B                      ldab            11,x
       226C 37                        pshb                            ;se envía num (columna)
       226D 1AEE09                    ldy             9,x
       2270 183C                      pshy                            ;se envía dirección de inicio de matriz
       2272 E608                      ldab            8,x
       2274 37                        pshb                            ;se envía jugador atacado
       2275 BD23EE                    jsr             writepos
       2278                           %CLRST          6
       0006           CONT_M          SET     6
       2278                           IF CONT_M > 0
       2278 31                        ins
       2279                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2279                           IF CONT_M > 0
       2279 31                        ins
       227A                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       227A                           IF CONT_M > 0
       227A 31                        ins
       227B                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       227B                           IF CONT_M > 0
       227B 31                        ins
       227C                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       227C                           IF CONT_M > 0
       227C 31                        ins
       227D                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       227D                           IF CONT_M > 0
       227D 31                        ins
       227E                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       227E                           ENDIF
       227E                           ENDIF
       227E                           ENDIF
       227E                           ENDIF
       227E                           ENDIF
       227E                           ENDIF
       227E                           ENDIF
       227E 6C0C                      inc             12,x            ;se recupera el dato
       2280                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       2280 E60C                      ldab            12,x
       2282 37                        pshb                            ;se envía letra (fila)
       2283 E60B                      ldab            11,x
       2285 37                        pshb                            ;se envía num (columna)
       2286 1AEE0E                    ldy             14,x
       2289 183C                      pshy                            ;se envía inicio de array
       228B E608                      ldab            8,x
       228D 37                        pshb                            ;se envía jugador atacado
       228E BD2476                    jsr             play_wrt
       2291                           %CLRST          5
       0005           CONT_M          SET     5
       2291                           IF CONT_M > 0
       2291 31                        ins
       2292                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2292                           IF CONT_M > 0
       2292 31                        ins
       2293                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2293                           IF CONT_M > 0
       2293 31                        ins
       2294                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2294                           IF CONT_M > 0
       2294 31                        ins
       2295                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2295                           IF CONT_M > 0
       2295 31                        ins
       2296                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2296                           ENDIF
       2296                           ENDIF
       2296                           ENDIF
       2296                           ENDIF
       2296                           ENDIF
       2296                           ENDIF
       2296                           %RESTORE
       2296 1838                      puly
       2298 38                        pulx
       2299 33                        pulb
       229A 32                        pula
       229B 39                        rts
                      
       229C           gp_shi_d        EQU             *
       229C E60C                      ldab            12,x
       229E C10F                      cmpb            #MAX_F
       22A0 276D                      beq             gp_shi_h        ;si está en el borde inferior, se continúa
       22A2 E60C                      ldab            12,x
       22A4 5C                        incb                            ;se va a la posición de abajo
       22A5 37                        pshb                            ;se envía la letra (fila)
       22A6 E60B                      ldab            11,x
       22A8 37                        pshb                            ;se envía el num (columna)
       22A9 1AEE0E                    ldy             14,x
       22AC 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       22AE E608                      ldab            8,x
       22B0 37                        pshb                            ;se envía el jugador atacado
       22B1 BD241F                    jsr             play_src        ;se busca la posición de abajo
       22B4                           %CLRST          5
       0005           CONT_M          SET     5
       22B4                           IF CONT_M > 0
       22B4 31                        ins
       22B5                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       22B5                           IF CONT_M > 0
       22B5 31                        ins
       22B6                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       22B6                           IF CONT_M > 0
       22B6 31                        ins
       22B7                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       22B7                           IF CONT_M > 0
       22B7 31                        ins
       22B8                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       22B8                           IF CONT_M > 0
       22B8 31                        ins
       22B9                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       22B9                           ENDIF
       22B9                           ENDIF
       22B9                           ENDIF
       22B9                           ENDIF
       22B9                           ENDIF
       22B9                           ENDIF
       22B9 2454                      bcc             gp_shi_h        ;si no había barco atacado abajo, el carguero está tocado
       22BB                           %WRITEPOS       #SUNK_SHI       
       22BB C690                      ldab            #SUNK_SHI
       22BD 37                        pshb                            ;se envía el byte a escribir
       22BE E60C                      ldab            12,x
       22C0 37                        pshb                            ;se envía letra (fila)
       22C1 E60B                      ldab            11,x
       22C3 37                        pshb                            ;se envía num (columna)
       22C4 1AEE09                    ldy             9,x
       22C7 183C                      pshy                            ;se envía dirección de inicio de matriz
       22C9 E608                      ldab            8,x
       22CB 37                        pshb                            ;se envía jugador atacado
       22CC BD23EE                    jsr             writepos
       22CF                           %CLRST          6
       0006           CONT_M          SET     6
       22CF                           IF CONT_M > 0
       22CF 31                        ins
       22D0                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       22D0                           IF CONT_M > 0
       22D0 31                        ins
       22D1                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       22D1                           IF CONT_M > 0
       22D1 31                        ins
       22D2                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       22D2                           IF CONT_M > 0
       22D2 31                        ins
       22D3                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       22D3                           IF CONT_M > 0
       22D3 31                        ins
       22D4                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       22D4                           IF CONT_M > 0
       22D4 31                        ins
       22D5                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       22D5                           ENDIF
       22D5                           ENDIF
       22D5                           ENDIF
       22D5                           ENDIF
       22D5                           ENDIF
       22D5                           ENDIF
       22D5                           ENDIF
       22D5 6C0C                      inc             12,x
       22D7                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       22D7 C690                      ldab            #SUNK_SHI
       22D9 37                        pshb                            ;se envía el byte a escribir
       22DA E60C                      ldab            12,x
       22DC 37                        pshb                            ;se envía letra (fila)
       22DD E60B                      ldab            11,x
       22DF 37                        pshb                            ;se envía num (columna)
       22E0 1AEE09                    ldy             9,x
       22E3 183C                      pshy                            ;se envía dirección de inicio de matriz
       22E5 E608                      ldab            8,x
       22E7 37                        pshb                            ;se envía jugador atacado
       22E8 BD23EE                    jsr             writepos
       22EB                           %CLRST          6
       0006           CONT_M          SET     6
       22EB                           IF CONT_M > 0
       22EB 31                        ins
       22EC                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       22EC                           IF CONT_M > 0
       22EC 31                        ins
       22ED                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       22ED                           IF CONT_M > 0
       22ED 31                        ins
       22EE                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       22EE                           IF CONT_M > 0
       22EE 31                        ins
       22EF                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       22EF                           IF CONT_M > 0
       22EF 31                        ins
       22F0                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       22F0                           IF CONT_M > 0
       22F0 31                        ins
       22F1                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       22F1                           ENDIF
       22F1                           ENDIF
       22F1                           ENDIF
       22F1                           ENDIF
       22F1                           ENDIF
       22F1                           ENDIF
       22F1                           ENDIF
       22F1 6A0C                      dec             12,x            ;se recupera el dato
       22F3                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       22F3 E60C                      ldab            12,x
       22F5 37                        pshb                            ;se envía letra (fila)
       22F6 E60B                      ldab            11,x
       22F8 37                        pshb                            ;se envía num (columna)
       22F9 1AEE0E                    ldy             14,x
       22FC 183C                      pshy                            ;se envía inicio de array
       22FE E608                      ldab            8,x
       2300 37                        pshb                            ;se envía jugador atacado
       2301 BD2476                    jsr             play_wrt
       2304                           %CLRST          5
       0005           CONT_M          SET     5
       2304                           IF CONT_M > 0
       2304 31                        ins
       2305                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2305                           IF CONT_M > 0
       2305 31                        ins
       2306                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2306                           IF CONT_M > 0
       2306 31                        ins
       2307                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2307                           IF CONT_M > 0
       2307 31                        ins
       2308                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2308                           IF CONT_M > 0
       2308 31                        ins
       2309                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2309                           ENDIF
       2309                           ENDIF
       2309                           ENDIF
       2309                           ENDIF
       2309                           ENDIF
       2309                           ENDIF
       2309                           %RESTORE
       2309 1838                      puly
       230B 38                        pulx
       230C 33                        pulb
       230D 32                        pula
       230E 39                        rts
                      
       230F           gp_shi_h        EQU             *
       230F                           %WRITEPOS       #HIT            ;se marca al carguero como tocado
       230F C6B0                      ldab            #HIT
       2311 37                        pshb                            ;se envía el byte a escribir
       2312 E60C                      ldab            12,x
       2314 37                        pshb                            ;se envía letra (fila)
       2315 E60B                      ldab            11,x
       2317 37                        pshb                            ;se envía num (columna)
       2318 1AEE09                    ldy             9,x
       231B 183C                      pshy                            ;se envía dirección de inicio de matriz
       231D E608                      ldab            8,x
       231F 37                        pshb                            ;se envía jugador atacado
       2320 BD23EE                    jsr             writepos
       2323                           %CLRST          6
       0006           CONT_M          SET     6
       2323                           IF CONT_M > 0
       2323 31                        ins
       2324                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2324                           IF CONT_M > 0
       2324 31                        ins
       2325                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2325                           IF CONT_M > 0
       2325 31                        ins
       2326                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2326                           IF CONT_M > 0
       2326 31                        ins
       2327                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2327                           IF CONT_M > 0
       2327 31                        ins
       2328                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2328                           IF CONT_M > 0
       2328 31                        ins
       2329                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2329                           ENDIF
       2329                           ENDIF
       2329                           ENDIF
       2329                           ENDIF
       2329                           ENDIF
       2329                           ENDIF
       2329                           ENDIF
       2329                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       2329 E60C                      ldab            12,x
       232B 37                        pshb                            ;se envía letra (fila)
       232C E60B                      ldab            11,x
       232E 37                        pshb                            ;se envía num (columna)
       232F 1AEE0E                    ldy             14,x
       2332 183C                      pshy                            ;se envía inicio de array
       2334 E608                      ldab            8,x
       2336 37                        pshb                            ;se envía jugador atacado
       2337 BD2476                    jsr             play_wrt
       233A                           %CLRST          5
       0005           CONT_M          SET     5
       233A                           IF CONT_M > 0
       233A 31                        ins
       233B                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       233B                           IF CONT_M > 0
       233B 31                        ins
       233C                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       233C                           IF CONT_M > 0
       233C 31                        ins
       233D                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       233D                           IF CONT_M > 0
       233D 31                        ins
       233E                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       233E                           IF CONT_M > 0
       233E 31                        ins
       233F                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       233F                           ENDIF
       233F                           ENDIF
       233F                           ENDIF
       233F                           ENDIF
       233F                           ENDIF
       233F                           ENDIF
       233F                           %RESTORE
       233F 1838                      puly
       2341 38                        pulx
       2342 33                        pulb
       2343 32                        pula
       2344 39                        rts
                      
                      *==========================================================================
                      * subrutina def_ships
                      * Función: Determina los valores de cargueros y submarinos a utilizar.
                      * Recibe: Nada
                      * Devuelve: Cantidad de cargueros y submarinos en memorias correspondientes.
                      * Requiere:
                      *==========================================================================
       2345           def_ships       EQU             *
       2345                           %BACKUP
       2345 36                        psha
       2346 37                        pshb
       2347 3C                        pshx
       2348 183C                      pshy
                      
       234A           wait_nsu        EQU             *
       234A CE4268                    ldx             #subm
       234D BD2569                    jsr             show
       2350 BD257D                    jsr             read_kb
       2353 4D                        tsta
       2354 2BF4                      bmi             wait_nsu
       2356 B141FB                    cmpa            buffer
       2359 27EF                      beq             wait_nsu
       235B B741FB                    staa            buffer
                      
       235E CE4201                    ldx             #kb_value
       2361 16                        tab
       2362 3A                        abx
       2363 A600                      ldaa            0,x             ;se obtiene el valor ingresado ya codificado
                      
       2365 8105                      cmpa            #SUB            ;se controla que no sea menor al máximo de submarinos
       2367 22E1                      bhi             wait_nsu
       2369 B741F7                    staa            cant_sub        ;se guarda la cantidad de submarinos
                      
       236C           wait_nsh        EQU             *
       236C CE426C                    ldx             #carg
       236F BD2569                    jsr             show
       2372 BD257D                    jsr             read_kb
       2375 4D                        tsta
       2376 2BF4                      bmi             wait_nsh
       2378 B141FB                    cmpa            buffer
       237B 27EF                      beq             wait_nsh
       237D B741FB                    staa            buffer
                      
       2380 CE4201                    ldx             #kb_value
       2383 16                        tab
       2384 3A                        abx
       2385 A600                      ldaa            0,x             ;se obtiene el valor ingresado ya codificado
                      
       2387 8104                      cmpa            #SHI            ;se controla que no sea menor al máximo de submarinos
       2389 22E1                      bhi             wait_nsh
       238B B741F8                    staa            cant_shi        ;se guarda la cantidad de submarinos
                      
       238E                           %RESTORE
       238E 1838                      puly
       2390 38                        pulx
       2391 33                        pulb
       2392 32                        pula
       2393 39                        rts
                      
                      *====================================================================================
                      * Subrutina "MIRROR"
                      *
                      * Descripción:  Esta subrutina copia el contenido de las posiciones de un array fuente
                      *               a un espacio de memoria destino.
                      * Parámetros de entrada:
                      *               - Dirección de comienzo de array FUENTE
                      *               - Número de arreglo contenido en otro arreglo (número de jugador)(0 en adelante)
                      *               - Dirección de DESTINO de la copia 
                      *               - Número de posiciones a copiar (un byte, hasta 255 posiciones.)
                      *               - Número de arreglo (de 0 en adelante) (número de jugador)
                      *
                      *       Todos los parámetros se reciben por stack en el orden indicado. 
                      *       Se destruirá el dato de la dirección del array de origen.
                      *               
                      * Parámetros de salida: NINGUNO
                      *       
                      *
                      * Consideraciones:
                      *               - Luego de invocada, se debe normalizar el stack en 6 posiciones
                      *
                      *====================================================================================
                      
       2394           mirror          EQU             *       
                              
       2394                           %BACKUP
       2394 36                        psha
       2395 37                        pshb
       2396 3C                        pshx
       2397 183C                      pshy
       2399 30                        tsx                             ;creo frame pointer
       239A A60B                      ldaa            11,x            ;cargo número de jugador
       239C E608                      ldab            8,x             ;cargo longitud de la copia (del array a copiar)
       239E 3D                        mul                             ;obtengo offset de jugador
       239F E30C                      addd            12,x            ;sumo offset de jugador
       23A1 ED0C                      std             12,x            ;guardo la dirección con offset del array fuente
                      
       23A3 A60C                      ldaa            12,x            ;cargo longitud de copia en contador
       23A5 2710                      beq             mi_end          ;si es cero, vuelvo desde subrutina
       23A7 1AEE09                    ldy             9,x             ;cargo índice a destino de la copia.
       23AA EE0C                      ldx             12,x            ;cargo índice a array origen de copia.
                      
       23AC           mi_loop         EQU             *
                      
       23AC E600                      ldab            0,x             ;cargo byte a copiar
       23AE 18E700                    stab            0,y             ;copio byte en destino
       23B1 08                        inx                             ;siguiente byte
       23B2 1808                      iny                             ;siguiente byte
       23B4 4A                        deca                            ;decremento contador
       23B5 26F5                      bne             mi_loop
                      
       23B7           mi_end          EQU             *
                      
       23B7                           %RESTORE
       23B7 1838                      puly
       23B9 38                        pulx
       23BA 33                        pulb
       23BB 32                        pula
                              
       23BC 39                        rts     
                                      
                                                      
                      *====================================================================================
                      * Subrutina "FETCHPOS"
                      *
                      * Descripción:  La subrutina busca en un array de tres dimensiones una 
                      *               determinada posición, y devuelve el byte que se encuentra
                      *               en esa posición por stack. Subrutina limitada a longitud
                      *               de elementos del array de un byte.
                      *
                      * Parámetros de entrada:
                      *               - FILA del array (En HEX, de $0A (fil.0) en adelante)
                      *               - COLUMNA del array (en HEX)
                      *               - Dirección de comienzo del arreglo (2 bytes)
                      *               - Número de arreglo (de 0 en adelante) (número de jugador)
                      *
                      *       Todos los parámetros se reciben por stack en el orden indicado.
                      *       Se destruirá el dato del índice al arreglo.
                      *               
                      * Parámetros de salida:
                      *               - Byte encontrado en esa posición (se devuelve por stack)
                      *
                      *
                      * Consideraciones:
                      *               - Antes de recuperar el byte en stack se debe normalizar el mismo
                      *                 en 4 posiciones (4 bytes)
                      *               - Requiere definición de constante "ARRAY_L"
                      *
                      *====================================================================================
                      
       23BD           fetchpos        EQU             *
                      
       23BD                           %BACKUP
       23BD 36                        psha
       23BE 37                        pshb
       23BF 3C                        pshx
       23C0 183C                      pshy
       23C2 30                        tsx                             ;creo frame pointer
       23C3 A608                      ldaa            8,x             ;cargo numero de jugador
       23C5 C624                      ldab            #ARRAY_L        ;cargo longitud del arreglo (MAX 255 bytes)
       23C7 3D                        mul                             ;obtengo offset de jugador
       23C8 E309                      addd            9,x             ;sumo offset de jugador a top of array
       23CA ED09                      std             9,x             ;obtengo direccion al arreglo del jugador
                      
       23CC A60C                      ldaa            12,x            ;cargo coordenada letra (FILA)
       23CE C60A                      ldab            #NORM           ;cargo valor de normalización de filas
       23D0 10                        sba                             ;obtengo numero de fila
       23D1 C606                      ldab            #COL            ;cargo longitud de fila
       23D3 3D                        mul                             ;obtengo offset de fila
       23D4 E309                      addd            9,x             ;sumo offset de fila a la dirección del arreglo del jugador
       23D6 ED09                      std             9,x             ;obtengo dirección a la fila que se desea
                      
       23D8 E60B                      ldab            11,x            ;cargo coordenada (offset) de columna
       23DA 5A                        decb                            ;normalizo offset de columna
       23DB 4F                        clra                            ;borro AccA para tener el valor de AccB en AccD
       23DC E309                      addd            9,x             ;sumo offset de columna a la dirección temporal
       23DE ED09                      std             9,x             ;guardo en stack la dirección del byte a consultar
                      
       23E0 1AEE09                    ldy             9,x             ;cargo en IY la dirección del byte que se desea obtener
       23E3 18A600                    ldaa            0,y             ;cargo el byte a devolver en AccA
       23E6 A70C                      staa            12,x            ;guardo el byte a devolver en stack
       23E8                           %RESTORE
       23E8 1838                      puly
       23EA 38                        pulx
       23EB 33                        pulb
       23EC 32                        pula
                                      
       23ED 39                        rts
                                      
                                      
                      
                      *====================================================================================
                      * Subrutina "WRITEPOS"
                      *
                      * Descripción:  La subrutina busca en un array de tres dimensiones una 
                      *               determinada posición, y escribe en la misma un byte de
                      *               datos que será recibido como parámetro de entrada por stack.
                      *
                      * Parámetros de entrada:
                      *               - BYTE a escribir en la posición indicada
                      *               - FILA del array (En HEX, de $0A (fil.0) en adelante)
                      *               - COLUMNA del array (en HEX)
                      *               - Dirección de comienzo del arreglo (2 bytes)
                      *               - Número de arreglo (de 0 en adelante) (número de jugador)
                      *
                      *       Todos los parámetros se reciben por stack en el orden indicado. 
                      *       Se destruirá el dato del índice al arreglo.
                      *               
                      * Parámetros de salida: NINGUNO
                      *       
                      *
                      * Consideraciones:
                      *               - Luego de invocada, se debe normalizar el stack en 6 posiciones
                      *               - Requiere definición de constante "ARRAY_L"
                      *
                      *====================================================================================
                      
       23EE           writepos        EQU             *
                      
       23EE                           %BACKUP
       23EE 36                        psha
       23EF 37                        pshb
       23F0 3C                        pshx
       23F1 183C                      pshy
       23F3 30                        tsx                             ;creo frame pointer
       23F4 A608                      ldaa            8,x             ;cargo numero de jugador
       23F6 C624                      ldab            #ARRAY_L        ;cargo longitud del arreglo (MAX 255 bytes)
       23F8 3D                        mul                             ;obtengo offset de jugador
       23F9 E309                      addd            9,x             ;sumo offset de jugador a top of array
       23FB ED09                      std             9,x             ;obtengo direccion al arreglo del jugador
                      
       23FD A60C                      ldaa            12,x            ;cargo coordenada letra (FILA)
       23FF C60A                      ldab            #NORM           ;cargo valor de normalización de filas
       2401 10                        sba                             ;obtengo numero de fila
       2402 C606                      ldab            #COL            ;cargo longitud de fila
       2404 3D                        mul                             ;obtengo offset de fila
       2405 E309                      addd            9,x             ;sumo offset de fila a la dirección del arreglo del jugador
       2407 ED09                      std             9,x             ;obtengo dirección a la fila que se desea
                      
       2409 E60B                      ldab            11,x            ;cargo coordenada (offset) de columna
       240B 5A                        decb                            ;normalizo offset de columna
       240C 4F                        clra                            ;borro AccA para tener el valor de AccB en AccD
       240D E309                      addd            9,x             ;sumo offset de columna a la dirección temporal
       240F ED09                      std             9,x             ;guardo en stack la dirección del byte a consultar
                      
       2411 1AEE09                    ldy             9,x             ;cargo en IY la dirección donde se desea escribir
       2414 A60D                      ldaa            13,x            ;cargo en AccA el byte que se quiere escribir en el arreglo
       2416 18A700                    staa            0,y             ;escribo el byte sobre el arreglo
       2419                           %RESTORE
       2419 1838                      puly
       241B 38                        pulx
       241C 33                        pulb
       241D 32                        pula
                                      
       241E 39                        rts
                      *====================================================================================
                      * Subrutina "PLAY_SRC"
                      *
                      * Descripción:  Esta subrutina busca dos bytes consecutivos (posicion jugada) dentro de un arreglo.
                      *               El valor $00 podrá estar en el array solo como terminador
                      * Parámetros de entrada:
                      *               - Byte1 a buscar (en este caso, n° fila)
                      *               - Byte2 a buscar (en este caso, n° columna)
                      *               - Dirección de inicio del array
                      *               - Número de jugador (0 o mayor)
                      *
                      *       Todos los parámetros se reciben por stack en el orden indicado. 
                      *               
                      * Parámetros de salida:
                      *               - Carry en 1, si se encontró la posición.
                      *       
                      *
                      * Consideraciones:
                      *               - Luego de invocada, se debe normalizar el stack en 5 posiciones antes de
                      *                 recuperar el parámetro de salida.
                      *
                      *====================================================================================
                      
       241F           play_src        EQU             *               
                                              
       241F                           %BACKUP
       241F 36                        psha
       2420 37                        pshb
       2421 3C                        pshx
       2422 183C                      pshy
       2424 30                        tsx                             ;creo frame pointer
       2425 A608                      ldaa            8,x             ;cargo número de array (jugador)
       2427 C61B                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       2429 3D                        mul                             ;obtengo offset de jugador
       242A E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       242C 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
                      
       242E           ps_loop         EQU             *
                      
       242E 18A600                    ldaa            0,y             ;cargo byte1 a comparar
       2431 2718                      beq             ps_ntfnd        ;si encontré el terminador, el valor requerido no está en el array
                      
       2433 A10C                      cmpa            12,x            ;comparo con el byte1 a buscar
       2435 2706                      beq             ps_next         ;si coinciden, busco el byte 2
       2437 1808                      iny                             ;me posiciono sobre siguiente byte1 a comparar
       2439 1808                      iny
       243B 20F1                      bra             ps_loop         ;vuelvo a comparar
                      
       243D           ps_next         EQU             *
                      
       243D 18E601                    ldab            1,y             ;cargo byte a continuación del anterior
       2440 1808                      iny                             ;me posiciono sobre siguiente byte1 a comparar
       2442 1808                      iny
       2444 E10B                      cmpb            11,x            ;comparo con byte2
       2446 26E6                      bne             ps_loop         ;si no son iguales, sigo buscando
       2448 0D                        sec                             ;set del carry para informar que se encontró el valor
       2449 2001                      bra             ps_end
                                              
       244B           ps_ntfnd        EQU             *
                      
       244B 0C                        clc                             ;carry en 0 para informar que no se encontró el valor
                      
       244C           ps_end          EQU             *
                                      
       244C                           %RESTORE
       244C 1838                      puly
       244E 38                        pulx
       244F 33                        pulb
       2450 32                        pula
                      
       2451 39                        rts
                      
                      
                      
                                              
                      
                                              
                      
                      *====================================================================================
                      * Subrutina "PLAY_CNT"
                      *
                      * Descripción:  Esta subrutina cuenta la cantidad de datos de dos bytes que se encuentran en una
                      *               determinada zona de memoria configurada como array. Se supone que $00 no es un dato presente en dicho array, 
                      *               usándose solamente como terminador del mismo.
                      *
                      * Parámetros de entrada:
                      *               - Dirección de inicio del array
                      *               - Número de jugador (0 o mayor)
                      *
                      *       Todos los parámetros se reciben por stack en el orden indicado. 
                      *               
                      * Parámetros de salida:
                      *               - Cantidad de datos de dos bytes encontrados, por stack (máximo 255 datos).
                      *       
                      *
                      * Consideraciones:
                      *               - Luego de invocada, se debe normalizar el stack en 2 posiciones antes de
                      *                 recuperar el parámetro de salida.
                      *
                      *====================================================================================
                      
       2452           play_cnt        EQU             *
                      
       2452                           %BACKUP
       2452 36                        psha
       2453 37                        pshb
       2454 3C                        pshx
       2455 183C                      pshy
       2457 30                        tsx                             ;creo frame pointer
       2458 A608                      ldaa            8,x             ;cargo número de array (jugador)
       245A C61B                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       245C 3D                        mul                             ;obtengo offset de jugador
       245D E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       245F 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
       2461 4F                        clra                            ;inicializo contador
                                              
       2462           pc_loop         EQU             *               
                                              
       2462 18E600                    ldab            0,y             ;cargo byte a comparar
       2465 2707                      beq             pc_end          ;si es terminador, terminé de buscar
       2467 1808                      iny                             ;si no es terminador, me posiciono sobre siguiente dato de 
       2469 1808                      iny                             ;2 bytes
       246B 4C                        inca                            ;incremento contador
       246C 20F4                      bra             pc_loop         ;vuelvo a comparar
                                              
       246E           pc_end          EQU             *       
                                              
       246E A70A                      staa            10,x            ;devuelvo valor por stack
       2470                           %RESTORE
       2470 1838                      puly
       2472 38                        pulx
       2473 33                        pulb
       2474 32                        pula
                                      
       2475 39                        rts
                      
                      
                      *====================================================================================
                      * Subrutina "PLAY_WRT"
                      *
                      * Descripción:  Esta subrutina escribe dos bytes distintos (en este caso, la posición 
                      *               jugada, compuesta de una letra y un número) en la primera posición "libre"
                      *               de un array y en la siguiente. La subrutina escribirá los datos desde la primera posición con ceros
                      *               que encuentre en el array, por lo que el mismo NO podrá contener el dato "$00" en ninguno
                      *               de sus bytes, caso contrario la subrutina lo interpretará como vacío y sobreescribirá información. Se
                      *               supone que el array tendrá por lo menos, dos posiciones libres.
                      *
                      * Parámetros de entrada:
                      *               - Primer byte a escribir (en este caso, fila jugada)
                      *               - Byte a escribir a continuación del primero (en este caso, columna jugada)
                      *               - Dirección de inicio del array en el que se va a escribir
                      *               - Número de jugador (0 o mayor)
                      *
                      *       Todos los parámetros se reciben por stack en el orden indicado. 
                      *               
                      * Parámetros de salida: NINGUNO
                      *       
                      *
                      * Consideraciones:
                      *               - Luego de invocada, se debe normalizar el stack en 5 posiciones
                      *
                      *====================================================================================
                      
       2476           play_wrt        EQU             *
                      
       2476                           %BACKUP
       2476 36                        psha
       2477 37                        pshb
       2478 3C                        pshx
       2479 183C                      pshy
       247B 30                        tsx                             ;creo frame pointer
       247C A608                      ldaa            8,x             ;cargo número de array (jugador)
       247E C61B                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       2480 3D                        mul                             ;obtengo offset de jugador
       2481 E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       2483 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
                      
                      
       2485           pw_loop         EQU             *
                      
       2485 18A600                    ldaa            0,y             ;cargo byte del array
       2488 2704                      beq             pw_end          ;si es cero, entonces escribo los datos
       248A 1808                      iny                             ;si no es cero, me posiciono sobre la siguiente posición
       248C 20F7                      bra             pw_loop         ;vuelvo a buscar ceros
                      
       248E           pw_end          EQU             *
                      
       248E A60C                      ldaa            12,x            ;cargo primer byte a escribir
       2490 18A700                    staa            0,y             ;guardo primer byte en memoria
       2493 1808                      iny                             ;me posiciono en el siguiente byte
       2495 A60B                      ldaa            11,x            ;cargo segundo byte a escribir
       2497 18A700                    staa            0,y             ;guardo segundo byte en memoria
       249A                           %RESTORE
       249A 1838                      puly
       249C 38                        pulx
       249D 33                        pulb
       249E 32                        pula
                      
       249F 39                        rts
                      
                      ***************************************************************************************
                      *Subrutina "CLRZONE"
                      *Función: "borra" (guarda ceros) una determinada zona de memoria.
                      *         
                      *
                      * Parámetros de entrada: -Dirección de comienzo de sección a borrar (por stack)
                      *                        -Longitud de zona de memoria (por stack)
                      *                       
                      *
                      * Parámetros de salida: Ninguno
                      *
                      * consideraciones: 
                      *                  - Incluir macros "%BACKUP" y "%RESTORE".
                      *                                       
                      *************************************************************************************** 
       24A0           clrzone         EQU     *
                      
       24A0                           %BACKUP                         ;backup de registros en stack
       24A0 36                        psha
       24A1 37                        pshb
       24A2 3C                        pshx
       24A3 183C                      pshy
                      
       24A5 30                        tsx
       24A6 A608                      ldaa    8,x                     ;cargo contador con longitud de zona a borrar
       24A8 EE09                      ldx     9,x                     ;cargo puntero zona de memoria a borrar
       24AA D600                      ldab    $00                     ;cargo el cero
       24AC           clr_loop        EQU     *
       24AC E700                      stab    0,x                     ;guardo caracter en posicion del array
       24AE 08                        inx                             ;siguiente posición     
       24AF 4A                        deca                            ;decremento contador
       24B0 26FA                      bne     clr_loop                ;si no es cero, escribo en la posición siguiente.
                      
       24B2                           %RESTORE                        :restauro backup en registros
       24B2 1838                      puly
       24B4 38                        pulx
       24B5 33                        pulb
       24B6 32                        pula
                      
       24B7 39                        rts
                      
                      *---------------------------------------
                      * WhoPlays: subrutina para ver que 
                      *           jugador juega
                      *---------------------------------------
                      
       24B8           whoplays        EQU             *
                      
       24B8 BD257D    wait_ply        jsr             read_kb
       24BB 4D                        tsta
       24BC 2BFA                      bmi             wait_ply
       24BE 8103                      cmpa            #B_P1
       24C0 2708                      beq             play_p1
       24C2 8107                      cmpa            #B_P2
       24C4 26F2                      bne             wait_ply
       24C6 8600                      ldaa            #P1
       24C8 2002                      bra             ply_end
                      
       24CA 8601      play_p1         ldaa            #P2
       24CC 39        ply_end         rts             
                      
                      *---------------------------------------
                      * Input: subrutina que devuelve
                      *        entrada para la batalla naval
                      * AccA: Fila de la posicion jugada
                      * AccB: Columna de la posicion jugada
                      *---------------------------------------
                      
       24CD           input           EQU             *
                      
                      * back up de registros
       24CD 3C                        pshx
       24CE 183C                      pshy
                      
                      * creo variables locales
       24D0 36                        psha
       24D1 36                        psha
       24D2 36                        psha
                      
                      * inicializo framepointer
       24D3 1830                      tsy
                      
                      * prendo el led
       24D5                           %LED_INV
       24D5 B61000                    ldaa            PORTA
       24D8 8840                      eora            #SHF_LED
       24DA B71000                    staa            PORTA
                      
                      * inicializo el contador
       24DD 8603      inp_rest        ldaa            #INPUTS
       24DF 18A700                    staa            CONT,Y
                                      
                      * limpio el display
       24E2 CE41FC                    ldx             #string
       24E5 8604                      ldaa            #4
       24E7 C620                      ldab            #ESP
       24E9 BD2647                    jsr             clrarray
                      
       24EC BD2636                    jsr             clr_dis
                      
       24EF 86FF                      ldaa            #$FF
       24F1 B741FB                    staa            buffer
                      
                      * lectura de teclado
       24F4 BD257D    wait_kb         jsr             read_kb
       24F7 4D                        tsta
       24F8 2BFA                      bmi             wait_kb
       24FA B141FB                    cmpa            buffer
       24FD 27F5                      beq             wait_kb
       24FF B741FB                    staa            buffer
                                      
       2502 810F                      cmpa            #B_reset
       2504 27D7                      beq             inp_rest
       2506 8103                      cmpa            #B_P1
       2508 27D3                      beq             inp_rest
       250A 8107                      cmpa            #B_P2
       250C 27CF                      beq             inp_rest
                      
       250E 186A00                    dec             CONT,Y
       2511 2727                      beq             chk_fire
       2513 36                        psha
       2514 CE4201                    ldx             #kb_value
       2517 16                        tab
       2518 3A                        abx
       2519 A600                      ldaa            0,X             
       251B 183C                      pshy
       251D 18E600                    ldab            CONT,Y
       2520 183A                      aby
       2522 18A700                    staa            0,Y
       2525 1838                      puly
                      
       2527 33                        pulb
       2528 BD2626                    jsr             to_ascii
       252B 17                        tba
                                      
       252C CE41FC                    ldx             #string
       252F BD2616                    jsr             sh_value
                      
       2532 CE41FC                    ldx             #string
       2535 BD2569                    jsr             show
                      
       2538 20BA                      bra             wait_kb
                      
       253A 810B      chk_fire        cmpa            #B_fire
       253C 269F                      bne             inp_rest
       253E 18A602                    ldaa            FILA_L,Y
       2541 810A                      cmpa            #MIN_FIL
       2543 2598                      blo             inp_rest
       2545 810F                      cmpa            #MAX_FIL
       2547 2294                      bhi             inp_rest
       2549 18A601                    ldaa            COL_L,Y
       254C 8101                      cmpa            #MIN_COL
       254E 258D                      blo             inp_rest
       2550 8106                      cmpa            #MAX_COL
       2552 2289                      bhi             inp_rest
                      
                      * apago el led
       2554                           %LED_INV
       2554 B61000                    ldaa            PORTA
       2557 8840                      eora            #SHF_LED
       2559 B71000                    staa            PORTA
                      
                      * libero variables locales
       255C 32                        pula
       255D 32                        pula
       255E 32                        pula
                      
       255F 18A602    return          ldaa            FILA_L,Y
       2562 18E601                    ldab            COL_L,Y         
                      
                      * restore de registros
       2565 1838                      puly
       2567 38                        pulx
                      
       2568 39                        rts
                      
                      *------------------------------------
                      * Show: Por IX direccion de string de
                      *       4 caracteres para imprimir
                      *------------------------------------
                      
       2569           show            EQU             *
                      
                      * back up de registros
       2569 3C                        pshx
       256A 37                        pshb
       256B 36                        psha
                      
                      * inicializo contador de posiciones
       256C C604                      ldab            #LENGTH
                                      
                      * loop para enviar a cada posicion
       256E 5A        sh_loop         decb
       256F 2B08                      bmi             sh_end          ; si no pase por todas las posiciones
       2571 A600                      ldaa            0,X             ; tomo del string el caracter
       2573 BD262F                    jsr             outchar         ; y lo imprimo en la posicion actual
       2576 08                        inx
       2577 20F5                      bra             sh_loop
                      
                      * restore de registros                          
       2579 32        sh_end          pula
       257A 33                        pulb
       257B 38                        pulx
       257C 39                        rts
                      
                      *----------------------------------------------
                      * read_kb: AccA: tecla presionada
                      *          o -1
                      *----------------------------------------------
                      
       257D           read_kb         EQU             *
                      
                      * back up de registros
       257D 3C                        pshx
       257E 37                        pshb
                      
                      * creo espacio para variables locales
       257F                           %MALLOC         1               ; variable mascara de filas
       257F                           IF 1 > 0
       257F 30                        tsx
       2580 8F                        xgdx
       2581 830001                    subd    #1
       2584 8F                        xgdx
       2585 35                        txs
       2586                           ENDIF
       2586                           %MALLOC         1               ; variable mascara de columnas
       2586                           IF 1 > 0
       2586 30                        tsx
       2587 8F                        xgdx
       2588 830001                    subd    #1
       258B 8F                        xgdx
       258C 35                        txs
       258D                           ENDIF
       258D                           %MALLOC         1               ; variable contador de columnas
       258D                           IF 1 > 0
       258D 30                        tsx
       258E 8F                        xgdx
       258F 830001                    subd    #1
       2592 8F                        xgdx
       2593 35                        txs
       2594                           ENDIF
       2594                           %MALLOC         1               ; variable contador de tecla
       2594                           IF 1 > 0
       2594 30                        tsx
       2595 8F                        xgdx
       2596 830001                    subd    #1
       2599 8F                        xgdx
       259A 35                        txs
       259B                           ENDIF
                      
                      * inicializacion de variables
       259B 30                        tsx                             ; framepointer para acceder a locales
       259C C604                      ldab            #LENGTH         ; contador de fila
       259E 860E                      ldaa            #FIL0           ; mascara para la fila cero
       25A0 A703                      staa            MASK_FIL,X      
       25A2 8610                      ldaa            #COL0           ; mascara para la columna cero
       25A4 A702                      staa            MASK_COL,X
       25A6 4F                        clra                            ; contador de columna
       25A7 A701                      staa            COL_CONT,X
       25A9 A700                      staa            KB_PRESS,X      ; variable con numero de tecla
                      
                      * loop para barrer y mirar teclado matricial
       25AB 86FF      kb_loop         ldaa            #NONE
       25AD 5A                        decb                            
       25AE 2B2C                      bmi             kb_end          ; me fijo si recorrio todas las filas
       25B0 A603                      ldaa            MASK_FIL,X      ; enmascaro la siguiente fila
       25B2 B71003                    staa            ROWS
       25B5 0D                        sec                             ; carry para rotacion de byte
       25B6 49                        rola                            
       25B7 A703                      staa            MASK_FIL,X      ; cambio variable mascara para la proxima fila
                      
       25B9 8604                      ldaa            #LENGTH
       25BB A701                      staa            COL_CONT,X
       25BD 8610                      ldaa            #COL0           ; inicializo variable para barrer
       25BF A702                      staa            MASK_COL,X      ; la columna
                      
       25C1 6A01      kb_col          dec             COL_CONT,X      ; me fijo si barrio todas las columnas
       25C3 2BE6                      bmi             kb_loop
       25C5 B61005                    ldaa            COLS            ; leo el puerto de columnas
       25C8 A402                      anda            MASK_COL,X      ; enmascaro columna
       25CA 2707                      beq             kb_hit          ; si es nulo apretaron un boton
       25CC 48                        lsla                            ; cambio mascara para proxima columna
       25CD A702                      staa            MASK_COL,X
       25CF 6C00                      inc             KB_PRESS,X      ; incremento numero de tecla
       25D1 20EE                      bra             kb_col
                      
       25D3 A600      kb_hit          ldaa            KB_PRESS,X      ; apretaron una tecla, lo comparo
       25D5 B141FA                    cmpa            kb_buff         ; con la que fue apretada antes
       25D8 2602                      bne             kb_end          ; aviso por led
                      
       25DA 86FF      kb_none         ldaa            #NONE           ; en caso de que no apreten nada
                      
       25DC B741FA    kb_end          staa            kb_buff         ; actualizo variable
                                              
                      * elimino variables locales
       25DF                           %RELMEM         1
       25DF                           IF 1 > 0
       25DF 30                        tsx
       25E0 8F                        xgdx
       25E1 C30001                    addd    #1
       25E4 8F                        xgdx
       25E5 35                        txs
       25E6                           ENDIF           
       25E6                           %RELMEM         1
       25E6                           IF 1 > 0
       25E6 30                        tsx
       25E7 8F                        xgdx
       25E8 C30001                    addd    #1
       25EB 8F                        xgdx
       25EC 35                        txs
       25ED                           ENDIF           
       25ED                           %RELMEM         1
       25ED                           IF 1 > 0
       25ED 30                        tsx
       25EE 8F                        xgdx
       25EF C30001                    addd    #1
       25F2 8F                        xgdx
       25F3 35                        txs
       25F4                           ENDIF           
       25F4                           %RELMEM         1
       25F4                           IF 1 > 0
       25F4 30                        tsx
       25F5 8F                        xgdx
       25F6 C30001                    addd    #1
       25F9 8F                        xgdx
       25FA 35                        txs
       25FB                           ENDIF           
                      
                      * restore de registros
       25FB 33                        pulb
       25FC 38                        pulx
                                      
       25FD 39                        rts
                      
                      *----------------------------------------------
                      * init_kb: subrutina para inicializar teclado
                      *----------------------------------------------
                      
       25FE           init_kb         EQU             *
                      
                      * back up de registros
       25FE 36                        psha
                      
                      * inicializacion de teclado             
       25FF 86FF                      ldaa            #NONE
       2601 B741FA                    staa            kb_buff
                      
                      * restore de registros
       2604 32                        pula
       2605 39                        rts
                      
                      *--------------------------------
                      * Inicializa display (y teclado)
                      *--------------------------------
                      
       2606           initdis         EQU             *
                              
       2606 36                        psha
                      
       2607 860F                      ldaa            #$0F       ;set for input/output
       2609 B71007                    staa            DDRC       ;C output
       260C B61002                    ldaa            PIOC       ;get current value
       260F 84FE                      anda            #$FE       ;reset bit 0 to 0
       2611 B71002                    staa            PIOC       ;send it out
                              
       2614 32                        pula
       2615 39                        rts
                      
                      *--------------------------------------------------------------
                      * sh_value: toma un string, desplaza los caracteres a izquierda
                      *           y agrega un nuevo caracter
                      *           IX: direccion del string
                      *           AccA: nuevo valor
                      *--------------------------------------------------------------
                      
       2616           sh_value        EQU             *
                      
                      * back up de registros
       2616 3C                        pshx
       2617 37                        pshb
                      
                      * loop para desplazar caracteres
       2618 E601      v_loop          ldab            1,X             ;tomo elemento siguiente en string
       261A 2705                      beq             v_end
       261C E700                      stab            0,X             ;y lo guardo en posicion actual
       261E 08                        inx
       261F 20F7                      bra             v_loop          
                      
       2621 A700      v_end           staa            0,X             ;ingresa nuevo caracter en cadena desplazada
                      
                      * restore de registros
       2623 33                        pulb
       2624 38                        pulx
       2625 39                        rts
                      
                      *----------------------------------------------
                      * toAscii: toma numero de boton del teclado
                      *          y devuelve su correspondiente en
                      *          ascii
                      *          toma  AccB: valor del boton
                      *          devuelve AccB: ascii
                      *----------------------------------------------
                      
       2626           to_ascii        EQU             *
                      
                      * back up de registros
       2626 3C                        pshx
                      
                      * inicializacion de variables
       2627 CE4211                    ldx             #kb_ascii       ; apunto a string con caracteres asignados por tecla
       262A 3A                        abx                             ; me muevo en string con indice y recupero segun tecla presionada
       262B E600                      ldab            0,X             
                      
                      * restore de registros
       262D 38                        pulx
       262E 39                        rts
                      
                      *--------------------------------
                      *Outchar: ACCA: Carácter a enviar
                      *           ACCB: Posición (0-3)
                      *--------------------------------
       262F           outchar         EQU             *
                              
       262F F71005                    stab            PORTCL          ;set the position 
       2632 B71004                    staa            PORTB           ;set the character
                              
       2635 39                        rts
                      
                      *-------------------------
                      * clr_dis: Limpia display
                      *-------------------------
                      
       2636           clr_dis         EQU             *
                      
                      * hago back up de registros
       2636 37                        pshb
       2637 36                        psha
                      
                      * inicializo el contador de posiciones
       2638 C604                      ldab            #LENGTH
                      
                      * loop para limpiar cada posicion
       263A 5A        clrloop         decb
       263B 2B07                      bmi             clr_end         ; si no pase por todas las posiciones
       263D 8620                      ldaa            #ESP            ; cargo en esta posicion espacio blanco
       263F BD262F                    jsr             outchar         ; y lo envio
       2642 20F6                      bra             clrloop         
                      
                      * restore de registros
       2644 32        clr_end         pula
       2645 33                        pulb
       2646 39                        rts
                      
                      *-------------------------------------------------------
                      * clrarray: subrutina para limpiar un array dado
                      * parametros: IX direccion del array
                      *             AccA cantidad de elementos del array
                      *             AccB caracter a poner
                      *-------------------------------------------------------
                      
       2647           clrarray        EQU             *
                                      
       2647 4A        cra_loop        deca                            ;contador con cantidad de elementos a limpiar
       2648 2705                      beq             cra_end
       264A E700                      stab            0,X             ;reemplazar elemento por caracter a poner
       264C 08                        inx
       264D 20F8                      bra             cra_loop
                                      
       264F E700      cra_end         stab            0,X
       2651 39                        rts
                      
                      *=============================================
                      *Subrutina winner: Recibe por stack un 1 si
                      *gano el player 1 y un 0 si gano el player 2.
                      *=============================================
                      
                                      
       2652           winner          EQU             *
       2652                           %BACKUP
       2652 36                        psha
       2653 37                        pshb
       2654 3C                        pshx
       2655 183C                      pshy
       2657 30                        tsx
       2658 A608                      ldaa            8,x     ;recupero parametro enviado por stack
       265A 2708                      beq             win2
                      
       265C           win1            EQU             *
       265C CE4256                    ldx             #string1
       265F BD2569                    jsr             show
       2662 2006                      bra             end
                      
       2664           win2            EQU             *
       2664 CE425B                    ldx             #string2
       2667 BD2569                    jsr             show
                              
       266A           end             EQU             *
       266A                           %RESTORE
       266A 1838                      puly
       266C 38                        pulx
       266D 33                        pulb
       266E 32                        pula
       266F 39                        rts
                      
                      *=============================================
                      *Subrutina turn: Recibe por stack un 1 si
                      *juega el player 1 y un 0 si juega el player 2.
                      *=============================================
                      
       2670           turn            EQU             *
       2670                           %BACKUP
       2670 36                        psha
       2671 37                        pshb
       2672 3C                        pshx
       2673 183C                      pshy
       2675 30                        tsx
       2676 A608                      ldaa            8,x
       2678 2708                      beq             p2_t
                      
       267A           p1_t            EQU             *
       267A CE4260                    ldx             #play1
       267D BD2569                    jsr             show
       2680 20E8                      bra             end
                              
       2682           p2_t            EQU             *
       2682 CE4264                    ldx             #play2
       2685 BD2569                    jsr             show
                      
       2688           end_t           EQU             *
       2688                           %RESTORE        
       2688 1838                      puly
       268A 38                        pulx
       268B 33                        pulb
       268C 32                        pula
       268D 39                        rts
                      
                      *************
                      * VARIABLES *
                      *************
                      ***STACK***
       4000                           ORG             $4000
       4000           stack           RMB             500
       41F3           STACKP          EQU             *-1
                      
                      ***MAIN VARIABLES***
       41F4           player          RMB             1               ;variable de jugador
       41F5           letra_p         RMB             1
       41F6           num_p           RMB             1
       41F7           cant_sub        RMB             1
       41F8           cant_shi        RMB             1
       41F9           tot_pos         RMB             1
                      
                      ***KEYBOARD VARIABLES***
       41FA           kb_buff         RMB             1
       41FB           buffer          RMB             1
                      
       41FC 20202020  string          FCC             '    '
       4200 00                        FCB             0
                      
       4201 0A0B0CFF  kb_value        FCB             10,11,12,255,13,14,15,255,1,2,3,255,4,5,6,255
       4205 0D0E0FFF
       4209 010203FF
       420D 040506FF
                      
       4211 41424320  kb_ascii        FCC             'ABC DEF 123 456'
       4215 44454620
       4219 31323320
       421D 343536
                      
                      ***ARREGLO DE BARCOS TOCADOS***
       0005           SUB             EQU             5
       0004           SHI             EQU             4
       0002           EL_SIZE         EQU             2
       000D           MAX_ATT         EQU             SUB*1+SHI*2
       001B           HITSIZE         EQU             EL_SIZE*MAX_ATT+1
       4220           hitlist         RMB             HITSIZE*2
                      
                      ***DISPLAY MESSAGES VARIABLES***
       4256 57494E31  string1         FCC     'WIN1'
       425A 00                        FCB     00
       425B 57494E32  string2         FCC     'WIN2'
       425F 00                        FCB     00
                      
       4260 5031      play1           FCC     'P1'
       4262 0000                      FCB     0,0
       4264 5032      play2           FCC     'P2'
       4266 0000                      FCB     0,0
                      
       4268 5355424D  subm            FCC     'SUBM'
       426C 43415247  carg            FCC     'CARG'
                      
                      ***MATRICES DE TABLERO DE CADA JUGADOR***
       5000                           ORG             $5000
       5000 83830303  p_board         FCB             SHI_NP,SHI_NP,WAT_NP,WAT_NP,WAT_NP,SUB_NP
       5004 0343
       5006 03038303                  FCB             WAT_NP,WAT_NP,SHI_NP,WAT_NP,SHI_NP,WAT_NP
       500A 8303
       500C 03038303                  FCB             WAT_NP,WAT_NP,SHI_NP,WAT_NP,SHI_NP,WAT_NP
       5010 8303
       5012 03830303                  FCB             WAT_NP,SHI_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       5016 0303
       5018 03830343                  FCB             WAT_NP,SHI_NP,WAT_NP,SUB_NP,WAT_NP,SUB_NP
       501C 0343
       501E 03034303                  FCB             WAT_NP,WAT_NP,SUB_NP,WAT_NP,SUB_NP,WAT_NP
       5022 4303
                      
       5024 83038303                  FCB             SHI_NP,WAT_NP,SHI_NP,WAT_NP,WAT_NP,SUB_NP
       5028 0343
       502A 83038303                  FCB             SHI_NP,WAT_NP,SHI_NP,WAT_NP,WAT_NP,WAT_NP
       502E 0303
       5030 03430303                  FCB             WAT_NP,SUB_NP,WAT_NP,WAT_NP,SHI_NP,WAT_NP
       5034 8303
       5036 03030303                  FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,SHI_NP,WAT_NP
       503A 8303
       503C 03034303                  FCB             WAT_NP,WAT_NP,SUB_NP,WAT_NP,WAT_NP,WAT_NP
       5040 0303
       5042 83830343                  FCB             SHI_NP,SHI_NP,WAT_NP,SUB_NP,WAT_NP,SUB_NP
       5046 0343
                      
                      ***MATRIZ DE DISPLAY***
       5F00                           ORG             $5F00
       5F00 03030303  display         FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       5F04 0303
       5F06 03030303                  FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       5F0A 0303
       5F0C 03030303                  FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       5F10 0303
       5F12 03030303                  FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       5F16 0303
       5F18 03030303                  FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       5F1C 0303
       5F1E 03030303                  FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       5F22 0303
                      
                                      
       5F24                           END
     Errors:  None         ###########
     ytes:   1848         # main_bn #
     RC:     24AD         ###########
 