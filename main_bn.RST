################################################################################
#                                                                              #
#    Micro Series 6801 Assembler V2.00/DOS              27/Nov/17  14:53:31    #
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
                      
       2003 BD25A8                    jsr             initdis         ;inicialización de display 7 seg
       2006 BD25A0                    jsr             init_kb         ;inicialización de teclado
                      
       2009 CE421C                    ldx             #hitlist
       200C 3C                        pshx                            ;se envía inicio de arreglo de barcos a poner en 0
       200D 8626                      ldaa            #(HITSIZE*2)
       200F 36                        psha                            ;se envía el tamaño del arreglo a limpiar
       2010 BD2442                    jsr             clrzone
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
                      
       2016           main            EQU             *
                      * ELEGIR JUGADOR
       2016 BD245A                    jsr             whoplays        ;se recibirá en AccA el jugador a atacar a continuación
       2019 B741F4                    staa            player          ;se habilita la siguiente jugada
                      
                      * ENVIAR TABLERO DE JUGADOR A ATACAR A DISPLAY
       201C CE5000                    ldx             #p_board
       201F 3C                        pshx                            ;se envía el inicio de las matrices de tableros
       2020 B641F4                    ldaa            player
       2023 36                        psha                            ;se envía el jugador atacado
       2024 CE5F00                    ldx             #display
       2027 3C                        pshx                            ;se envía la matriz de display (destino a copiar)
       2028 8624                      ldaa            #ARRAY_L
       202A 36                        psha                            ;se envía el tamaño de la matriz tablero
       202B BD2336                    jsr             mirror          ;se envía el tablero del jugador atacado al display
       202E                           %CLRST          6
       0006           CONT_M          SET     6
       202E                           IF CONT_M > 0
       202E 31                        ins
       202F                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       202F                           IF CONT_M > 0
       202F 31                        ins
       2030                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2030                           IF CONT_M > 0
       2030 31                        ins
       2031                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2031                           IF CONT_M > 0
       2031 31                        ins
       2032                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2032                           IF CONT_M > 0
       2032 31                        ins
       2033                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2033                           IF CONT_M > 0
       2033 31                        ins
       2034                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2034                           ENDIF
       2034                           ENDIF
       2034                           ENDIF
       2034                           ENDIF
       2034                           ENDIF
       2034                           ENDIF
       2034                           ENDIF
                      
                      * INGRESAR POSICIÓN A ATACAR
       2034 BD246F                    jsr             input           ;se llama a la subrutina de interfaz de usuario
       2037 B741F5                    staa            letra_p         ;se recibe en AccA la letra jugada (fila) y en AccB el número (columna)
       203A F741F6                    stab            num_p           ;se guarda la jugada
                      
                      * VERIFICAR QUE NO SEA POSICIÓN DE BARCO YA ATACADA
       203D B641F5                    ldaa            letra_p
       2040 36                        psha                            ;se envía la letra jugada
       2041 F641F6                    ldab            num_p
       2044 37                        pshb                            ;se envía el número jugado
       2045 CE421C                    ldx             #hitlist
       2048 3C                        pshx                            ;se envía la posición del primer arreglo de barcos tocados
       2049 B641F4                    ldaa            player
       204C 36                        psha                            ;se envía el jugador a atacar
       204D BD23C1                    jsr             play_src        ;se recibirá el carry en 1 si la posición corresponde a un barco ya atacado
       2050                           %CLRST          5               ;se limpia el stack
       0005           CONT_M          SET     5
       2050                           IF CONT_M > 0
       2050 31                        ins
       2051                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2051                           IF CONT_M > 0
       2051 31                        ins
       2052                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2052                           IF CONT_M > 0
       2052 31                        ins
       2053                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2053                           IF CONT_M > 0
       2053 31                        ins
       2054                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2054                           IF CONT_M > 0
       2054 31                        ins
       2055                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2055                           ENDIF
       2055                           ENDIF
       2055                           ENDIF
       2055                           ENDIF
       2055                           ENDIF
       2055                           ENDIF
       2055 25BF                      bcs             main            ;si la posición ya fue jugada, se pierde el turno y pasa al siguiente
                      
                      * MANDAR QUE JUGADOR ESTÁ JUGANDO AL DISPLAY
       2057 B641F4                    ldaa            player
       205A 36                        psha                            ;se envía el jugador que está siendo atacado
       205B BD2612                    jsr             turn
       205E                           %CLRST          1
       0001           CONT_M          SET     1
       205E                           IF CONT_M > 0
       205E 31                        ins
       205F                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       205F                           ENDIF
       205F                           ENDIF
                      
                      * BUSCAR EL BYTE ATACADO EN EL TABLERO DEL JUGADOR ATACADO
       205F B641F5                    ldaa            letra_p
       2062 36                        psha                            ;se envía le letra jugada
       2063 F641F6                    ldab            num_p
       2066 37                        pshb                            ;se envía el número jugado
       2067 CE5000                    ldx             #p_board
       206A 3C                        pshx                            ;se envía la dirección de inicio de los arreglos de tablero
       206B F641F4                    ldab            player
       206E 37                        pshb                            ;se envía el jugador a atacar
       206F BD235F                    jsr             fetchpos
       2072                           %CLRST          4               ;se limpia el stack
       0004           CONT_M          SET     4
       2072                           IF CONT_M > 0
       2072 31                        ins
       2073                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2073                           IF CONT_M > 0
       2073 31                        ins
       2074                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2074                           IF CONT_M > 0
       2074 31                        ins
       2075                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2075                           IF CONT_M > 0
       2075 31                        ins
       2076                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2076                           ENDIF
       2076                           ENDIF
       2076                           ENDIF
       2076                           ENDIF
       2076                           ENDIF
       2076 32                        pula                            ;se toma del stack el dato del byte jugado
                      
                      * ATACAR
       2077 CE421C                    ldx             #hitlist
       207A 3C                        pshx                            ;se envía inicio del arreglo de barcos atacados
       207B 36                        psha                            ;se envía el byte a jugado
       207C B641F5                    ldaa            letra_p
       207F 36                        psha                            ;se envía le letra jugada
       2080 F641F6                    ldab            num_p
       2083 37                        pshb                            ;se envía el número jugado
       2084 CE5000                    ldx             #p_board
       2087 3C                        pshx                            ;se envía la dirección de inicio de los arreglos de tablero
       2088 F641F4                    ldab            player
       208B 37                        pshb                            ;se envía el jugador a atacar
       208C BD20CE                    jsr             gameplay
       208F                           %CLRST          8               ;se limpia el stack
       0008           CONT_M          SET     8
       208F                           IF CONT_M > 0
       208F 31                        ins
       2090                           %CLRST  CONT_M-1
       0007           CONT_M          SET     CONT_M-1
       2090                           IF CONT_M > 0
       2090 31                        ins
       2091                           %CLRST  CONT_M-1
       0006           CONT_M          SET     CONT_M-1
       2091                           IF CONT_M > 0
       2091 31                        ins
       2092                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2092                           IF CONT_M > 0
       2092 31                        ins
       2093                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2093                           IF CONT_M > 0
       2093 31                        ins
       2094                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2094                           IF CONT_M > 0
       2094 31                        ins
       2095                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2095                           IF CONT_M > 0
       2095 31                        ins
       2096                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2096                           IF CONT_M > 0
       2096 31                        ins
       2097                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2097                           ENDIF
       2097                           ENDIF
       2097                           ENDIF
       2097                           ENDIF
       2097                           ENDIF
       2097                           ENDIF
       2097                           ENDIF
       2097                           ENDIF
       2097                           ENDIF
                      
                      * ACTUALIZAR DISPLAY CON LA POSICIÓN YA ATACADA
       2097 CE5000                    ldx             #p_board
       209A 3C                        pshx                            ;se envía el inicio de las matrices de tableros
       209B B641F4                    ldaa            player
       209E 36                        psha                            ;se envía el jugador atacado
       209F CE5F00                    ldx             #display
       20A2 3C                        pshx                            ;se envía la matriz de display (destino a copiar)
       20A3 8624                      ldaa            #ARRAY_L
       20A5 36                        psha                            ;se envía el tamaño de la matriz tablero
       20A6 BD2336                    jsr             mirror          ;se envía el tablero del jugador atacado al display
       20A9                           %CLRST          6
       0006           CONT_M          SET     6
       20A9                           IF CONT_M > 0
       20A9 31                        ins
       20AA                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       20AA                           IF CONT_M > 0
       20AA 31                        ins
       20AB                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       20AB                           IF CONT_M > 0
       20AB 31                        ins
       20AC                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       20AC                           IF CONT_M > 0
       20AC 31                        ins
       20AD                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       20AD                           IF CONT_M > 0
       20AD 31                        ins
       20AE                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       20AE                           IF CONT_M > 0
       20AE 31                        ins
       20AF                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       20AF                           ENDIF
       20AF                           ENDIF
       20AF                           ENDIF
       20AF                           ENDIF
       20AF                           ENDIF
       20AF                           ENDIF
       20AF                           ENDIF
                      
                      * CONTAR BARCOS ATACADOS A VER SI EL JUGADOR GANÓ               
       20AF CE421C                    ldx             #hitlist
       20B2 3C                        pshx                            ;se envía la posición del primer arreglo de barcos tocados
       20B3 B641F4                    ldaa            player
       20B6 36                        psha                            ;se envía el jugador a atacar
       20B7 BD23F4                    jsr             play_cnt
       20BA                           %CLRST          2
       0002           CONT_M          SET     2
       20BA                           IF CONT_M > 0
       20BA 31                        ins
       20BB                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       20BB                           IF CONT_M > 0
       20BB 31                        ins
       20BC                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       20BC                           ENDIF
       20BC                           ENDIF
       20BC                           ENDIF
       20BC 32                        pula                            ;se recibe la cantidad de bytes contados
       20BD 8109                      cmpa            #9              
       20BF 2703                      beq             end_game        ;si se llegó al máximo de barcos, se termina el juego
                      
       20C1 7E2016                    jmp             main
                      
                      * MANDAR MENSAJE A DISPLAY DE QUIÉN GANÓ
       20C4           end_game        EQU             *
       20C4 B641F4                    ldaa            player
       20C7 36                        psha
       20C8 BD25F4                    jsr             winner
       20CB                           %CLRST          1
       0001           CONT_M          SET     1
       20CB                           IF CONT_M > 0
       20CB 31                        ins
       20CC                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       20CC                           ENDIF
       20CC                           ENDIF
                      
       20CC           end_loop        EQU             *
       20CC 20FE                      bra             end_loop
                      
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
       20CE           gameplay        EQU             *
       20CE                           %BACKUP
       20CE 36                        psha
       20CF 37                        pshb
       20D0 3C                        pshx
       20D1 183C                      pshy
                                      
       20D3 30                        tsx                             ;se usa el IX como frame pointer
       20D4 A60D                      ldaa            13,x            ;se carga en AccA el byte a analizar
       20D6 84C0                      anda            #11000000b      ;se enmascan los bits 0 a 5, para analizar los más significativos
                      
       20D8           gp_water        EQU             *
       20D8 2620                      bne             gp_sub          ;se estudia si era agua
       20DA                           %WRITEPOS       #WATER          ;se cambia el color en el tablero
       20DA C608                      ldab            #WATER
       20DC 37                        pshb                            ;se envía el byte a escribir
       20DD E60C                      ldab            12,x
       20DF 37                        pshb                            ;se envía letra (fila)
       20E0 E60B                      ldab            11,x
       20E2 37                        pshb                            ;se envía num (columna)
       20E3 1AEE09                    ldy             9,x
       20E6 183C                      pshy                            ;se envía dirección de inicio de matriz
       20E8 E608                      ldab            8,x
       20EA 37                        pshb                            ;se envía jugador atacado
       20EB BD2390                    jsr             writepos
       20EE                           %CLRST          6
       0006           CONT_M          SET     6
       20EE                           IF CONT_M > 0
       20EE 31                        ins
       20EF                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       20EF                           IF CONT_M > 0
       20EF 31                        ins
       20F0                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       20F0                           IF CONT_M > 0
       20F0 31                        ins
       20F1                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       20F1                           IF CONT_M > 0
       20F1 31                        ins
       20F2                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       20F2                           IF CONT_M > 0
       20F2 31                        ins
       20F3                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       20F3                           IF CONT_M > 0
       20F3 31                        ins
       20F4                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       20F4                           ENDIF
       20F4                           ENDIF
       20F4                           ENDIF
       20F4                           ENDIF
       20F4                           ENDIF
       20F4                           ENDIF
       20F4                           ENDIF
       20F4                           %RESTORE
       20F4 1838                      puly
       20F6 38                        pulx
       20F7 33                        pulb
       20F8 32                        pula
       20F9 39                        rts
                      
       20FA           gp_sub          EQU             *
       20FA 8140                      cmpa            #01000000b      ;se estudia si era un submarino
       20FC 2636                      bne             gp_shi_r
       20FE                           %WRITEPOS       #SUNK_SUB       ;se cambia el color en el tablero
       20FE C650                      ldab            #SUNK_SUB
       2100 37                        pshb                            ;se envía el byte a escribir
       2101 E60C                      ldab            12,x
       2103 37                        pshb                            ;se envía letra (fila)
       2104 E60B                      ldab            11,x
       2106 37                        pshb                            ;se envía num (columna)
       2107 1AEE09                    ldy             9,x
       210A 183C                      pshy                            ;se envía dirección de inicio de matriz
       210C E608                      ldab            8,x
       210E 37                        pshb                            ;se envía jugador atacado
       210F BD2390                    jsr             writepos
       2112                           %CLRST          6
       0006           CONT_M          SET     6
       2112                           IF CONT_M > 0
       2112 31                        ins
       2113                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2113                           IF CONT_M > 0
       2113 31                        ins
       2114                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2114                           IF CONT_M > 0
       2114 31                        ins
       2115                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2115                           IF CONT_M > 0
       2115 31                        ins
       2116                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2116                           IF CONT_M > 0
       2116 31                        ins
       2117                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2117                           IF CONT_M > 0
       2117 31                        ins
       2118                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2118                           ENDIF
       2118                           ENDIF
       2118                           ENDIF
       2118                           ENDIF
       2118                           ENDIF
       2118                           ENDIF
       2118                           ENDIF
       2118                           %PLAY_WRT                       ;se agraga la posición al array de barcos atacados
       2118 E60C                      ldab            12,x
       211A 37                        pshb                            ;se envía letra (fila)
       211B E60B                      ldab            11,x
       211D 37                        pshb                            ;se envía num (columna)
       211E 1AEE0E                    ldy             14,x
       2121 183C                      pshy                            ;se envía inicio de array
       2123 E608                      ldab            8,x
       2125 37                        pshb                            ;se envía jugador atacado
       2126 BD2418                    jsr             play_wrt
       2129                           %CLRST          5
       0005           CONT_M          SET     5
       2129                           IF CONT_M > 0
       2129 31                        ins
       212A                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       212A                           IF CONT_M > 0
       212A 31                        ins
       212B                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       212B                           IF CONT_M > 0
       212B 31                        ins
       212C                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       212C                           IF CONT_M > 0
       212C 31                        ins
       212D                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       212D                           IF CONT_M > 0
       212D 31                        ins
       212E                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       212E                           ENDIF
       212E                           ENDIF
       212E                           ENDIF
       212E                           ENDIF
       212E                           ENDIF
       212E                           ENDIF
       212E                           %RESTORE
       212E 1838                      puly
       2130 38                        pulx
       2131 33                        pulb
       2132 32                        pula
       2133 39                        rts
                      
       2134           gp_shi_r        EQU             *               ;si no era ninguno de los anteriores, es un carguero
       2134 E60B                      ldab            11,x
       2136 C106                      cmpb            #MAX_C
       2138 276D                      beq             gp_shi_l        ;si está en el borde derecho, se continúa
       213A E60C                      ldab            12,x
       213C 37                        pshb                            ;se envía la letra (fila)
       213D E60B                      ldab            11,x
       213F 5C                        incb                            ;se avanza a la derecha
       2140 37                        pshb                            ;se envía el num (columna)
       2141 1AEE0E                    ldy             14,x
       2144 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       2146 E608                      ldab            8,x
       2148 37                        pshb                            ;se envía el jugador atacado
       2149 BD23C1                    jsr             play_src        ;se busca la posición de la derecha
       214C                           %CLRST          5
       0005           CONT_M          SET     5
       214C                           IF CONT_M > 0
       214C 31                        ins
       214D                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       214D                           IF CONT_M > 0
       214D 31                        ins
       214E                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       214E                           IF CONT_M > 0
       214E 31                        ins
       214F                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       214F                           IF CONT_M > 0
       214F 31                        ins
       2150                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2150                           IF CONT_M > 0
       2150 31                        ins
       2151                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2151                           ENDIF
       2151                           ENDIF
       2151                           ENDIF
       2151                           ENDIF
       2151                           ENDIF
       2151                           ENDIF
       2151 2454                      bcc             gp_shi_l        ;si no había barco atacado en la derecha, se busca la izquierda
       2153                           %WRITEPOS       #SUNK_SHI       
       2153 C690                      ldab            #SUNK_SHI
       2155 37                        pshb                            ;se envía el byte a escribir
       2156 E60C                      ldab            12,x
       2158 37                        pshb                            ;se envía letra (fila)
       2159 E60B                      ldab            11,x
       215B 37                        pshb                            ;se envía num (columna)
       215C 1AEE09                    ldy             9,x
       215F 183C                      pshy                            ;se envía dirección de inicio de matriz
       2161 E608                      ldab            8,x
       2163 37                        pshb                            ;se envía jugador atacado
       2164 BD2390                    jsr             writepos
       2167                           %CLRST          6
       0006           CONT_M          SET     6
       2167                           IF CONT_M > 0
       2167 31                        ins
       2168                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2168                           IF CONT_M > 0
       2168 31                        ins
       2169                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2169                           IF CONT_M > 0
       2169 31                        ins
       216A                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       216A                           IF CONT_M > 0
       216A 31                        ins
       216B                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       216B                           IF CONT_M > 0
       216B 31                        ins
       216C                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       216C                           IF CONT_M > 0
       216C 31                        ins
       216D                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       216D                           ENDIF
       216D                           ENDIF
       216D                           ENDIF
       216D                           ENDIF
       216D                           ENDIF
       216D                           ENDIF
       216D                           ENDIF
       216D 6C0B                      inc             11,x
       216F                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       216F C690                      ldab            #SUNK_SHI
       2171 37                        pshb                            ;se envía el byte a escribir
       2172 E60C                      ldab            12,x
       2174 37                        pshb                            ;se envía letra (fila)
       2175 E60B                      ldab            11,x
       2177 37                        pshb                            ;se envía num (columna)
       2178 1AEE09                    ldy             9,x
       217B 183C                      pshy                            ;se envía dirección de inicio de matriz
       217D E608                      ldab            8,x
       217F 37                        pshb                            ;se envía jugador atacado
       2180 BD2390                    jsr             writepos
       2183                           %CLRST          6
       0006           CONT_M          SET     6
       2183                           IF CONT_M > 0
       2183 31                        ins
       2184                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2184                           IF CONT_M > 0
       2184 31                        ins
       2185                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2185                           IF CONT_M > 0
       2185 31                        ins
       2186                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2186                           IF CONT_M > 0
       2186 31                        ins
       2187                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2187                           IF CONT_M > 0
       2187 31                        ins
       2188                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2188                           IF CONT_M > 0
       2188 31                        ins
       2189                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2189                           ENDIF
       2189                           ENDIF
       2189                           ENDIF
       2189                           ENDIF
       2189                           ENDIF
       2189                           ENDIF
       2189                           ENDIF
       2189 6A0B                      dec             11,x            ;se recupera el dato
       218B                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       218B E60C                      ldab            12,x
       218D 37                        pshb                            ;se envía letra (fila)
       218E E60B                      ldab            11,x
       2190 37                        pshb                            ;se envía num (columna)
       2191 1AEE0E                    ldy             14,x
       2194 183C                      pshy                            ;se envía inicio de array
       2196 E608                      ldab            8,x
       2198 37                        pshb                            ;se envía jugador atacado
       2199 BD2418                    jsr             play_wrt
       219C                           %CLRST          5
       0005           CONT_M          SET     5
       219C                           IF CONT_M > 0
       219C 31                        ins
       219D                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       219D                           IF CONT_M > 0
       219D 31                        ins
       219E                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       219E                           IF CONT_M > 0
       219E 31                        ins
       219F                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       219F                           IF CONT_M > 0
       219F 31                        ins
       21A0                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       21A0                           IF CONT_M > 0
       21A0 31                        ins
       21A1                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       21A1                           ENDIF
       21A1                           ENDIF
       21A1                           ENDIF
       21A1                           ENDIF
       21A1                           ENDIF
       21A1                           ENDIF
       21A1                           %RESTORE
       21A1 1838                      puly
       21A3 38                        pulx
       21A4 33                        pulb
       21A5 32                        pula
       21A6 39                        rts
                      
       21A7           gp_shi_l        EQU             *
       21A7 E60B                      ldab            11,x
       21A9 C101                      cmpb            #MIN_C
       21AB 276D                      beq             gp_shi_u        ;si está en el borde izquierdo, se continúa
       21AD E60C                      ldab            12,x
       21AF 37                        pshb                            ;se envía la letra (fila)
       21B0 E60B                      ldab            11,x
       21B2 5A                        decb                            ;se retrocede a la izquierda
       21B3 37                        pshb                            ;se envía el número (columna)
       21B4 1AEE0E                    ldy             14,x
       21B7 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       21B9 E608                      ldab            8,x
       21BB 37                        pshb                            ;se envía el jugador atacado
       21BC BD23C1                    jsr             play_src        ;se busca la posición de la izquierda
       21BF                           %CLRST          5
       0005           CONT_M          SET     5
       21BF                           IF CONT_M > 0
       21BF 31                        ins
       21C0                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       21C0                           IF CONT_M > 0
       21C0 31                        ins
       21C1                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       21C1                           IF CONT_M > 0
       21C1 31                        ins
       21C2                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       21C2                           IF CONT_M > 0
       21C2 31                        ins
       21C3                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       21C3                           IF CONT_M > 0
       21C3 31                        ins
       21C4                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       21C4                           ENDIF
       21C4                           ENDIF
       21C4                           ENDIF
       21C4                           ENDIF
       21C4                           ENDIF
       21C4                           ENDIF
       21C4 2454                      bcc             gp_shi_u        ;si no había barco atacado a la izquierda, se busca arriba
       21C6                           %WRITEPOS       #SUNK_SHI       
       21C6 C690                      ldab            #SUNK_SHI
       21C8 37                        pshb                            ;se envía el byte a escribir
       21C9 E60C                      ldab            12,x
       21CB 37                        pshb                            ;se envía letra (fila)
       21CC E60B                      ldab            11,x
       21CE 37                        pshb                            ;se envía num (columna)
       21CF 1AEE09                    ldy             9,x
       21D2 183C                      pshy                            ;se envía dirección de inicio de matriz
       21D4 E608                      ldab            8,x
       21D6 37                        pshb                            ;se envía jugador atacado
       21D7 BD2390                    jsr             writepos
       21DA                           %CLRST          6
       0006           CONT_M          SET     6
       21DA                           IF CONT_M > 0
       21DA 31                        ins
       21DB                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       21DB                           IF CONT_M > 0
       21DB 31                        ins
       21DC                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       21DC                           IF CONT_M > 0
       21DC 31                        ins
       21DD                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       21DD                           IF CONT_M > 0
       21DD 31                        ins
       21DE                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       21DE                           IF CONT_M > 0
       21DE 31                        ins
       21DF                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       21DF                           IF CONT_M > 0
       21DF 31                        ins
       21E0                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       21E0                           ENDIF
       21E0                           ENDIF
       21E0                           ENDIF
       21E0                           ENDIF
       21E0                           ENDIF
       21E0                           ENDIF
       21E0                           ENDIF
       21E0 6A0B                      dec             11,x
       21E2                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       21E2 C690                      ldab            #SUNK_SHI
       21E4 37                        pshb                            ;se envía el byte a escribir
       21E5 E60C                      ldab            12,x
       21E7 37                        pshb                            ;se envía letra (fila)
       21E8 E60B                      ldab            11,x
       21EA 37                        pshb                            ;se envía num (columna)
       21EB 1AEE09                    ldy             9,x
       21EE 183C                      pshy                            ;se envía dirección de inicio de matriz
       21F0 E608                      ldab            8,x
       21F2 37                        pshb                            ;se envía jugador atacado
       21F3 BD2390                    jsr             writepos
       21F6                           %CLRST          6
       0006           CONT_M          SET     6
       21F6                           IF CONT_M > 0
       21F6 31                        ins
       21F7                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       21F7                           IF CONT_M > 0
       21F7 31                        ins
       21F8                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       21F8                           IF CONT_M > 0
       21F8 31                        ins
       21F9                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       21F9                           IF CONT_M > 0
       21F9 31                        ins
       21FA                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       21FA                           IF CONT_M > 0
       21FA 31                        ins
       21FB                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       21FB                           IF CONT_M > 0
       21FB 31                        ins
       21FC                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       21FC                           ENDIF
       21FC                           ENDIF
       21FC                           ENDIF
       21FC                           ENDIF
       21FC                           ENDIF
       21FC                           ENDIF
       21FC                           ENDIF
       21FC 6C0B                      inc             11,x            ;se recupera el dato
       21FE                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       21FE E60C                      ldab            12,x
       2200 37                        pshb                            ;se envía letra (fila)
       2201 E60B                      ldab            11,x
       2203 37                        pshb                            ;se envía num (columna)
       2204 1AEE0E                    ldy             14,x
       2207 183C                      pshy                            ;se envía inicio de array
       2209 E608                      ldab            8,x
       220B 37                        pshb                            ;se envía jugador atacado
       220C BD2418                    jsr             play_wrt
       220F                           %CLRST          5
       0005           CONT_M          SET     5
       220F                           IF CONT_M > 0
       220F 31                        ins
       2210                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2210                           IF CONT_M > 0
       2210 31                        ins
       2211                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2211                           IF CONT_M > 0
       2211 31                        ins
       2212                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2212                           IF CONT_M > 0
       2212 31                        ins
       2213                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2213                           IF CONT_M > 0
       2213 31                        ins
       2214                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2214                           ENDIF
       2214                           ENDIF
       2214                           ENDIF
       2214                           ENDIF
       2214                           ENDIF
       2214                           ENDIF
       2214                           %RESTORE
       2214 1838                      puly
       2216 38                        pulx
       2217 33                        pulb
       2218 32                        pula
       2219 39                        rts
                      
       221A           gp_shi_u        EQU             *
       221A E60C                      ldab            12,x
       221C C10A                      cmpb            #MIN_F
       221E 276D                      beq             gp_shi_d        ;si está en el borde superior, se continúa
       2220 E60C                      ldab            12,x
       2222 5A                        decb                            ;se va a la posición de arriba
       2223 37                        pshb                            ;se envía la letra (fila)
       2224 E60B                      ldab            11,x
       2226 37                        pshb                            ;se envía el num (columna)
       2227 1AEE0E                    ldy             14,x
       222A 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       222C E608                      ldab            8,x
       222E 37                        pshb                            ;se envía el jugador atacado
       222F BD23C1                    jsr             play_src        ;se busca la posición de arriba
       2232                           %CLRST          5
       0005           CONT_M          SET     5
       2232                           IF CONT_M > 0
       2232 31                        ins
       2233                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2233                           IF CONT_M > 0
       2233 31                        ins
       2234                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2234                           IF CONT_M > 0
       2234 31                        ins
       2235                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2235                           IF CONT_M > 0
       2235 31                        ins
       2236                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2236                           IF CONT_M > 0
       2236 31                        ins
       2237                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2237                           ENDIF
       2237                           ENDIF
       2237                           ENDIF
       2237                           ENDIF
       2237                           ENDIF
       2237                           ENDIF
       2237 2454                      bcc             gp_shi_d        ;si no había barco atacado arriba, se busca abajo
       2239                           %WRITEPOS       #SUNK_SHI       
       2239 C690                      ldab            #SUNK_SHI
       223B 37                        pshb                            ;se envía el byte a escribir
       223C E60C                      ldab            12,x
       223E 37                        pshb                            ;se envía letra (fila)
       223F E60B                      ldab            11,x
       2241 37                        pshb                            ;se envía num (columna)
       2242 1AEE09                    ldy             9,x
       2245 183C                      pshy                            ;se envía dirección de inicio de matriz
       2247 E608                      ldab            8,x
       2249 37                        pshb                            ;se envía jugador atacado
       224A BD2390                    jsr             writepos
       224D                           %CLRST          6
       0006           CONT_M          SET     6
       224D                           IF CONT_M > 0
       224D 31                        ins
       224E                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       224E                           IF CONT_M > 0
       224E 31                        ins
       224F                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       224F                           IF CONT_M > 0
       224F 31                        ins
       2250                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2250                           IF CONT_M > 0
       2250 31                        ins
       2251                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2251                           IF CONT_M > 0
       2251 31                        ins
       2252                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2252                           IF CONT_M > 0
       2252 31                        ins
       2253                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2253                           ENDIF
       2253                           ENDIF
       2253                           ENDIF
       2253                           ENDIF
       2253                           ENDIF
       2253                           ENDIF
       2253                           ENDIF
       2253 6A0C                      dec             12,x
       2255                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       2255 C690                      ldab            #SUNK_SHI
       2257 37                        pshb                            ;se envía el byte a escribir
       2258 E60C                      ldab            12,x
       225A 37                        pshb                            ;se envía letra (fila)
       225B E60B                      ldab            11,x
       225D 37                        pshb                            ;se envía num (columna)
       225E 1AEE09                    ldy             9,x
       2261 183C                      pshy                            ;se envía dirección de inicio de matriz
       2263 E608                      ldab            8,x
       2265 37                        pshb                            ;se envía jugador atacado
       2266 BD2390                    jsr             writepos
       2269                           %CLRST          6
       0006           CONT_M          SET     6
       2269                           IF CONT_M > 0
       2269 31                        ins
       226A                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       226A                           IF CONT_M > 0
       226A 31                        ins
       226B                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       226B                           IF CONT_M > 0
       226B 31                        ins
       226C                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       226C                           IF CONT_M > 0
       226C 31                        ins
       226D                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       226D                           IF CONT_M > 0
       226D 31                        ins
       226E                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       226E                           IF CONT_M > 0
       226E 31                        ins
       226F                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       226F                           ENDIF
       226F                           ENDIF
       226F                           ENDIF
       226F                           ENDIF
       226F                           ENDIF
       226F                           ENDIF
       226F                           ENDIF
       226F 6C0C                      inc             12,x            ;se recupera el dato
       2271                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       2271 E60C                      ldab            12,x
       2273 37                        pshb                            ;se envía letra (fila)
       2274 E60B                      ldab            11,x
       2276 37                        pshb                            ;se envía num (columna)
       2277 1AEE0E                    ldy             14,x
       227A 183C                      pshy                            ;se envía inicio de array
       227C E608                      ldab            8,x
       227E 37                        pshb                            ;se envía jugador atacado
       227F BD2418                    jsr             play_wrt
       2282                           %CLRST          5
       0005           CONT_M          SET     5
       2282                           IF CONT_M > 0
       2282 31                        ins
       2283                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2283                           IF CONT_M > 0
       2283 31                        ins
       2284                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2284                           IF CONT_M > 0
       2284 31                        ins
       2285                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2285                           IF CONT_M > 0
       2285 31                        ins
       2286                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2286                           IF CONT_M > 0
       2286 31                        ins
       2287                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2287                           ENDIF
       2287                           ENDIF
       2287                           ENDIF
       2287                           ENDIF
       2287                           ENDIF
       2287                           ENDIF
       2287                           %RESTORE
       2287 1838                      puly
       2289 38                        pulx
       228A 33                        pulb
       228B 32                        pula
       228C 39                        rts
                      
       228D           gp_shi_d        EQU             *
       228D E60C                      ldab            12,x
       228F C10F                      cmpb            #MAX_F
       2291 276D                      beq             gp_shi_h        ;si está en el borde inferior, se continúa
       2293 E60C                      ldab            12,x
       2295 5C                        incb                            ;se va a la posición de abajo
       2296 37                        pshb                            ;se envía la letra (fila)
       2297 E60B                      ldab            11,x
       2299 37                        pshb                            ;se envía el num (columna)
       229A 1AEE0E                    ldy             14,x
       229D 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       229F E608                      ldab            8,x
       22A1 37                        pshb                            ;se envía el jugador atacado
       22A2 BD23C1                    jsr             play_src        ;se busca la posición de abajo
       22A5                           %CLRST          5
       0005           CONT_M          SET     5
       22A5                           IF CONT_M > 0
       22A5 31                        ins
       22A6                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       22A6                           IF CONT_M > 0
       22A6 31                        ins
       22A7                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       22A7                           IF CONT_M > 0
       22A7 31                        ins
       22A8                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       22A8                           IF CONT_M > 0
       22A8 31                        ins
       22A9                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       22A9                           IF CONT_M > 0
       22A9 31                        ins
       22AA                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       22AA                           ENDIF
       22AA                           ENDIF
       22AA                           ENDIF
       22AA                           ENDIF
       22AA                           ENDIF
       22AA                           ENDIF
       22AA 2454                      bcc             gp_shi_h        ;si no había barco atacado abajo, el carguero está tocado
       22AC                           %WRITEPOS       #SUNK_SHI       
       22AC C690                      ldab            #SUNK_SHI
       22AE 37                        pshb                            ;se envía el byte a escribir
       22AF E60C                      ldab            12,x
       22B1 37                        pshb                            ;se envía letra (fila)
       22B2 E60B                      ldab            11,x
       22B4 37                        pshb                            ;se envía num (columna)
       22B5 1AEE09                    ldy             9,x
       22B8 183C                      pshy                            ;se envía dirección de inicio de matriz
       22BA E608                      ldab            8,x
       22BC 37                        pshb                            ;se envía jugador atacado
       22BD BD2390                    jsr             writepos
       22C0                           %CLRST          6
       0006           CONT_M          SET     6
       22C0                           IF CONT_M > 0
       22C0 31                        ins
       22C1                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       22C1                           IF CONT_M > 0
       22C1 31                        ins
       22C2                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       22C2                           IF CONT_M > 0
       22C2 31                        ins
       22C3                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       22C3                           IF CONT_M > 0
       22C3 31                        ins
       22C4                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       22C4                           IF CONT_M > 0
       22C4 31                        ins
       22C5                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       22C5                           IF CONT_M > 0
       22C5 31                        ins
       22C6                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       22C6                           ENDIF
       22C6                           ENDIF
       22C6                           ENDIF
       22C6                           ENDIF
       22C6                           ENDIF
       22C6                           ENDIF
       22C6                           ENDIF
       22C6 6C0C                      inc             12,x
       22C8                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       22C8 C690                      ldab            #SUNK_SHI
       22CA 37                        pshb                            ;se envía el byte a escribir
       22CB E60C                      ldab            12,x
       22CD 37                        pshb                            ;se envía letra (fila)
       22CE E60B                      ldab            11,x
       22D0 37                        pshb                            ;se envía num (columna)
       22D1 1AEE09                    ldy             9,x
       22D4 183C                      pshy                            ;se envía dirección de inicio de matriz
       22D6 E608                      ldab            8,x
       22D8 37                        pshb                            ;se envía jugador atacado
       22D9 BD2390                    jsr             writepos
       22DC                           %CLRST          6
       0006           CONT_M          SET     6
       22DC                           IF CONT_M > 0
       22DC 31                        ins
       22DD                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       22DD                           IF CONT_M > 0
       22DD 31                        ins
       22DE                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       22DE                           IF CONT_M > 0
       22DE 31                        ins
       22DF                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       22DF                           IF CONT_M > 0
       22DF 31                        ins
       22E0                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       22E0                           IF CONT_M > 0
       22E0 31                        ins
       22E1                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       22E1                           IF CONT_M > 0
       22E1 31                        ins
       22E2                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       22E2                           ENDIF
       22E2                           ENDIF
       22E2                           ENDIF
       22E2                           ENDIF
       22E2                           ENDIF
       22E2                           ENDIF
       22E2                           ENDIF
       22E2 6A0C                      dec             12,x            ;se recupera el dato
       22E4                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       22E4 E60C                      ldab            12,x
       22E6 37                        pshb                            ;se envía letra (fila)
       22E7 E60B                      ldab            11,x
       22E9 37                        pshb                            ;se envía num (columna)
       22EA 1AEE0E                    ldy             14,x
       22ED 183C                      pshy                            ;se envía inicio de array
       22EF E608                      ldab            8,x
       22F1 37                        pshb                            ;se envía jugador atacado
       22F2 BD2418                    jsr             play_wrt
       22F5                           %CLRST          5
       0005           CONT_M          SET     5
       22F5                           IF CONT_M > 0
       22F5 31                        ins
       22F6                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       22F6                           IF CONT_M > 0
       22F6 31                        ins
       22F7                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       22F7                           IF CONT_M > 0
       22F7 31                        ins
       22F8                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       22F8                           IF CONT_M > 0
       22F8 31                        ins
       22F9                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       22F9                           IF CONT_M > 0
       22F9 31                        ins
       22FA                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       22FA                           ENDIF
       22FA                           ENDIF
       22FA                           ENDIF
       22FA                           ENDIF
       22FA                           ENDIF
       22FA                           ENDIF
       22FA                           %RESTORE
       22FA 1838                      puly
       22FC 38                        pulx
       22FD 33                        pulb
       22FE 32                        pula
       22FF 39                        rts
                      
       2300           gp_shi_h        EQU             *
       2300                           %WRITEPOS       #HIT            ;se marca al carguero como tocado
       2300 C6B0                      ldab            #HIT
       2302 37                        pshb                            ;se envía el byte a escribir
       2303 E60C                      ldab            12,x
       2305 37                        pshb                            ;se envía letra (fila)
       2306 E60B                      ldab            11,x
       2308 37                        pshb                            ;se envía num (columna)
       2309 1AEE09                    ldy             9,x
       230C 183C                      pshy                            ;se envía dirección de inicio de matriz
       230E E608                      ldab            8,x
       2310 37                        pshb                            ;se envía jugador atacado
       2311 BD2390                    jsr             writepos
       2314                           %CLRST          6
       0006           CONT_M          SET     6
       2314                           IF CONT_M > 0
       2314 31                        ins
       2315                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2315                           IF CONT_M > 0
       2315 31                        ins
       2316                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2316                           IF CONT_M > 0
       2316 31                        ins
       2317                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2317                           IF CONT_M > 0
       2317 31                        ins
       2318                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2318                           IF CONT_M > 0
       2318 31                        ins
       2319                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2319                           IF CONT_M > 0
       2319 31                        ins
       231A                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       231A                           ENDIF
       231A                           ENDIF
       231A                           ENDIF
       231A                           ENDIF
       231A                           ENDIF
       231A                           ENDIF
       231A                           ENDIF
       231A                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       231A E60C                      ldab            12,x
       231C 37                        pshb                            ;se envía letra (fila)
       231D E60B                      ldab            11,x
       231F 37                        pshb                            ;se envía num (columna)
       2320 1AEE0E                    ldy             14,x
       2323 183C                      pshy                            ;se envía inicio de array
       2325 E608                      ldab            8,x
       2327 37                        pshb                            ;se envía jugador atacado
       2328 BD2418                    jsr             play_wrt
       232B                           %CLRST          5
       0005           CONT_M          SET     5
       232B                           IF CONT_M > 0
       232B 31                        ins
       232C                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       232C                           IF CONT_M > 0
       232C 31                        ins
       232D                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       232D                           IF CONT_M > 0
       232D 31                        ins
       232E                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       232E                           IF CONT_M > 0
       232E 31                        ins
       232F                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       232F                           IF CONT_M > 0
       232F 31                        ins
       2330                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2330                           ENDIF
       2330                           ENDIF
       2330                           ENDIF
       2330                           ENDIF
       2330                           ENDIF
       2330                           ENDIF
       2330                           %RESTORE
       2330 1838                      puly
       2332 38                        pulx
       2333 33                        pulb
       2334 32                        pula
       2335 39                        rts
                      
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
                      
       2336           mirror          EQU             *       
                              
       2336                           %BACKUP
       2336 36                        psha
       2337 37                        pshb
       2338 3C                        pshx
       2339 183C                      pshy
       233B 30                        tsx                             ;creo frame pointer
       233C A60B                      ldaa            11,x            ;cargo número de jugador
       233E E608                      ldab            8,x             ;cargo longitud de la copia (del array a copiar)
       2340 3D                        mul                             ;obtengo offset de jugador
       2341 E30C                      addd            12,x            ;sumo offset de jugador
       2343 ED0C                      std             12,x            ;guardo la dirección con offset del array fuente
                      
       2345 A60C                      ldaa            12,x            ;cargo longitud de copia en contador
       2347 2710                      beq             mi_end          ;si es cero, vuelvo desde subrutina
       2349 1AEE09                    ldy             9,x             ;cargo índice a destino de la copia.
       234C EE0C                      ldx             12,x            ;cargo índice a array origen de copia.
                      
       234E           mi_loop         EQU             *
                      
       234E E600                      ldab            0,x             ;cargo byte a copiar
       2350 18E700                    stab            0,y             ;copio byte en destino
       2353 08                        inx                             ;siguiente byte
       2354 1808                      iny                             ;siguiente byte
       2356 4A                        deca                            ;decremento contador
       2357 26F5                      bne             mi_loop
                      
       2359           mi_end          EQU             *
                      
       2359                           %RESTORE
       2359 1838                      puly
       235B 38                        pulx
       235C 33                        pulb
       235D 32                        pula
                              
       235E 39                        rts     
                                      
                                                      
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
                      
       235F           fetchpos        EQU             *
                      
       235F                           %BACKUP
       235F 36                        psha
       2360 37                        pshb
       2361 3C                        pshx
       2362 183C                      pshy
       2364 30                        tsx                             ;creo frame pointer
       2365 A608                      ldaa            8,x             ;cargo numero de jugador
       2367 C624                      ldab            #ARRAY_L        ;cargo longitud del arreglo (MAX 255 bytes)
       2369 3D                        mul                             ;obtengo offset de jugador
       236A E309                      addd            9,x             ;sumo offset de jugador a top of array
       236C ED09                      std             9,x             ;obtengo direccion al arreglo del jugador
                      
       236E A60C                      ldaa            12,x            ;cargo coordenada letra (FILA)
       2370 C60A                      ldab            #NORM           ;cargo valor de normalización de filas
       2372 10                        sba                             ;obtengo numero de fila
       2373 C606                      ldab            #COL            ;cargo longitud de fila
       2375 3D                        mul                             ;obtengo offset de fila
       2376 E309                      addd            9,x             ;sumo offset de fila a la dirección del arreglo del jugador
       2378 ED09                      std             9,x             ;obtengo dirección a la fila que se desea
                      
       237A E60B                      ldab            11,x            ;cargo coordenada (offset) de columna
       237C 5A                        decb                            ;normalizo offset de columna
       237D 4F                        clra                            ;borro AccA para tener el valor de AccB en AccD
       237E E309                      addd            9,x             ;sumo offset de columna a la dirección temporal
       2380 ED09                      std             9,x             ;guardo en stack la dirección del byte a consultar
                      
       2382 1AEE09                    ldy             9,x             ;cargo en IY la dirección del byte que se desea obtener
       2385 18A600                    ldaa            0,y             ;cargo el byte a devolver en AccA
       2388 A70C                      staa            12,x            ;guardo el byte a devolver en stack
       238A                           %RESTORE
       238A 1838                      puly
       238C 38                        pulx
       238D 33                        pulb
       238E 32                        pula
                                      
       238F 39                        rts
                                      
                                      
                      
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
                      
       2390           writepos        EQU             *
                      
       2390                           %BACKUP
       2390 36                        psha
       2391 37                        pshb
       2392 3C                        pshx
       2393 183C                      pshy
       2395 30                        tsx                             ;creo frame pointer
       2396 A608                      ldaa            8,x             ;cargo numero de jugador
       2398 C624                      ldab            #ARRAY_L        ;cargo longitud del arreglo (MAX 255 bytes)
       239A 3D                        mul                             ;obtengo offset de jugador
       239B E309                      addd            9,x             ;sumo offset de jugador a top of array
       239D ED09                      std             9,x             ;obtengo direccion al arreglo del jugador
                      
       239F A60C                      ldaa            12,x            ;cargo coordenada letra (FILA)
       23A1 C60A                      ldab            #NORM           ;cargo valor de normalización de filas
       23A3 10                        sba                             ;obtengo numero de fila
       23A4 C606                      ldab            #COL            ;cargo longitud de fila
       23A6 3D                        mul                             ;obtengo offset de fila
       23A7 E309                      addd            9,x             ;sumo offset de fila a la dirección del arreglo del jugador
       23A9 ED09                      std             9,x             ;obtengo dirección a la fila que se desea
                      
       23AB E60B                      ldab            11,x            ;cargo coordenada (offset) de columna
       23AD 5A                        decb                            ;normalizo offset de columna
       23AE 4F                        clra                            ;borro AccA para tener el valor de AccB en AccD
       23AF E309                      addd            9,x             ;sumo offset de columna a la dirección temporal
       23B1 ED09                      std             9,x             ;guardo en stack la dirección del byte a consultar
                      
       23B3 1AEE09                    ldy             9,x             ;cargo en IY la dirección donde se desea escribir
       23B6 A60D                      ldaa            13,x            ;cargo en AccA el byte que se quiere escribir en el arreglo
       23B8 18A700                    staa            0,y             ;escribo el byte sobre el arreglo
       23BB                           %RESTORE
       23BB 1838                      puly
       23BD 38                        pulx
       23BE 33                        pulb
       23BF 32                        pula
                                      
       23C0 39                        rts
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
                      
       23C1           play_src        EQU             *               
                                              
       23C1                           %BACKUP
       23C1 36                        psha
       23C2 37                        pshb
       23C3 3C                        pshx
       23C4 183C                      pshy
       23C6 30                        tsx                             ;creo frame pointer
       23C7 A608                      ldaa            8,x             ;cargo número de array (jugador)
       23C9 C613                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       23CB 3D                        mul                             ;obtengo offset de jugador
       23CC E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       23CE 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
                      
       23D0           ps_loop         EQU             *
                      
       23D0 18A600                    ldaa            0,y             ;cargo byte1 a comparar
       23D3 2718                      beq             ps_ntfnd        ;si encontré el terminador, el valor requerido no está en el array
                      
       23D5 A10C                      cmpa            12,x            ;comparo con el byte1 a buscar
       23D7 2706                      beq             ps_next         ;si coinciden, busco el byte 2
       23D9 1808                      iny                             ;me posiciono sobre siguiente byte1 a comparar
       23DB 1808                      iny
       23DD 20F1                      bra             ps_loop         ;vuelvo a comparar
                      
       23DF           ps_next         EQU             *
                      
       23DF 18E601                    ldab            1,y             ;cargo byte a continuación del anterior
       23E2 1808                      iny                             ;me posiciono sobre siguiente byte1 a comparar
       23E4 1808                      iny
       23E6 E10B                      cmpb            11,x            ;comparo con byte2
       23E8 26E6                      bne             ps_loop         ;si no son iguales, sigo buscando
       23EA 0D                        sec                             ;set del carry para informar que se encontró el valor
       23EB 2001                      bra             ps_end
                                              
       23ED           ps_ntfnd        EQU             *
                      
       23ED 0C                        clc                             ;carry en 0 para informar que no se encontró el valor
                      
       23EE           ps_end          EQU             *
                                      
       23EE                           %RESTORE
       23EE 1838                      puly
       23F0 38                        pulx
       23F1 33                        pulb
       23F2 32                        pula
                      
       23F3 39                        rts
                      
                      
                      
                                              
                      
                                              
                      
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
                      
       23F4           play_cnt        EQU             *
                      
       23F4                           %BACKUP
       23F4 36                        psha
       23F5 37                        pshb
       23F6 3C                        pshx
       23F7 183C                      pshy
       23F9 30                        tsx                             ;creo frame pointer
       23FA A608                      ldaa            8,x             ;cargo número de array (jugador)
       23FC C613                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       23FE 3D                        mul                             ;obtengo offset de jugador
       23FF E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       2401 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
       2403 4F                        clra                            ;inicializo contador
                                              
       2404           pc_loop         EQU             *               
                                              
       2404 18E600                    ldab            0,y             ;cargo byte a comparar
       2407 2707                      beq             pc_end          ;si es terminador, terminé de buscar
       2409 1808                      iny                             ;si no es terminador, me posiciono sobre siguiente dato de 
       240B 1808                      iny                             ;2 bytes
       240D 4C                        inca                            ;incremento contador
       240E 20F4                      bra             pc_loop         ;vuelvo a comparar
                                              
       2410           pc_end          EQU             *       
                                              
       2410 A70A                      staa            10,x            ;devuelvo valor por stack
       2412                           %RESTORE
       2412 1838                      puly
       2414 38                        pulx
       2415 33                        pulb
       2416 32                        pula
                                      
       2417 39                        rts
                      
                      
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
                      
       2418           play_wrt        EQU             *
                      
       2418                           %BACKUP
       2418 36                        psha
       2419 37                        pshb
       241A 3C                        pshx
       241B 183C                      pshy
       241D 30                        tsx                             ;creo frame pointer
       241E A608                      ldaa            8,x             ;cargo número de array (jugador)
       2420 C613                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       2422 3D                        mul                             ;obtengo offset de jugador
       2423 E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       2425 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
                      
                      
       2427           pw_loop         EQU             *
                      
       2427 18A600                    ldaa            0,y             ;cargo byte del array
       242A 2704                      beq             pw_end          ;si es cero, entonces escribo los datos
       242C 1808                      iny                             ;si no es cero, me posiciono sobre la siguiente posición
       242E 20F7                      bra             pw_loop         ;vuelvo a buscar ceros
                      
       2430           pw_end          EQU             *
                      
       2430 A60C                      ldaa            12,x            ;cargo primer byte a escribir
       2432 18A700                    staa            0,y             ;guardo primer byte en memoria
       2435 1808                      iny                             ;me posiciono en el siguiente byte
       2437 A60B                      ldaa            11,x            ;cargo segundo byte a escribir
       2439 18A700                    staa            0,y             ;guardo segundo byte en memoria
       243C                           %RESTORE
       243C 1838                      puly
       243E 38                        pulx
       243F 33                        pulb
       2440 32                        pula
                      
       2441 39                        rts
                      
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
       2442           clrzone         EQU     *
                      
       2442                           %BACKUP                         ;backup de registros en stack
       2442 36                        psha
       2443 37                        pshb
       2444 3C                        pshx
       2445 183C                      pshy
                      
       2447 30                        tsx
       2448 A608                      ldaa    8,x                     ;cargo contador con longitud de zona a borrar
       244A EE09                      ldx     9,x                     ;cargo puntero zona de memoria a borrar
       244C D600                      ldab    $00                     ;cargo el cero
       244E           clr_loop        EQU     *
       244E E700                      stab    0,x                     ;guardo caracter en posicion del array
       2450 08                        inx                             ;siguiente posición     
       2451 4A                        deca                            ;decremento contador
       2452 26FA                      bne     clr_loop                ;si no es cero, escribo en la posición siguiente.
                      
       2454                           %RESTORE                        :restauro backup en registros
       2454 1838                      puly
       2456 38                        pulx
       2457 33                        pulb
       2458 32                        pula
                      
       2459 39                        rts
                      
                      *---------------------------------------
                      * WhoPlays: subrutina para ver que 
                      *           jugador juega
                      *---------------------------------------
                      
       245A           whoplays        EQU             *
                      
       245A BD251F    wait_ply        jsr             read_kb
       245D 4D                        tsta
       245E 2BFA                      bmi             wait_ply
       2460 8103                      cmpa            #B_P1
       2462 2708                      beq             play_p1
       2464 8107                      cmpa            #B_P2
       2466 26F2                      bne             wait_ply
       2468 8600                      ldaa            #P1
       246A 2002                      bra             ply_end
                      
       246C 8601      play_p1         ldaa            #P2
       246E 39        ply_end         rts             
                      
                      *---------------------------------------
                      * Input: subrutina que devuelve
                      *        entrada para la batalla naval
                      * AccA: Fila de la posicion jugada
                      * AccB: Columna de la posicion jugada
                      *---------------------------------------
                      
       246F           input           EQU             *
                      
                      * back up de registros
       246F 3C                        pshx
       2470 183C                      pshy
                      
                      * creo variables locales
       2472 36                        psha
       2473 36                        psha
       2474 36                        psha
                      
                      * inicializo framepointer
       2475 1830                      tsy
                      
                      * prendo el led
       2477                           %LED_INV
       2477 B61000                    ldaa            PORTA
       247A 8840                      eora            #SHF_LED
       247C B71000                    staa            PORTA
                      
                      * inicializo el contador
       247F 8603      inp_rest        ldaa            #INPUTS
       2481 18A700                    staa            CONT,Y
                                      
                      * limpio el display
       2484 CE41F9                    ldx             #string
       2487 8604                      ldaa            #4
       2489 C620                      ldab            #ESP
       248B BD25E9                    jsr             clrarray
                      
       248E BD25D8                    jsr             clr_dis
                      
       2491 86FF                      ldaa            #$FF
       2493 B741F8                    staa            buffer
                      
                      * lectura de teclado
       2496 BD251F    wait_kb         jsr             read_kb
       2499 4D                        tsta
       249A 2BFA                      bmi             wait_kb
       249C B141F8                    cmpa            buffer
       249F 27F5                      beq             wait_kb
       24A1 B741F8                    staa            buffer
                                      
       24A4 810F                      cmpa            #B_reset
       24A6 27D7                      beq             inp_rest
       24A8 8103                      cmpa            #B_P1
       24AA 27D3                      beq             inp_rest
       24AC 8107                      cmpa            #B_P2
       24AE 27CF                      beq             inp_rest
                      
       24B0 186A00                    dec             CONT,Y
       24B3 2727                      beq             chk_fire
       24B5 36                        psha
       24B6 CE41FE                    ldx             #kb_value
       24B9 16                        tab
       24BA 3A                        abx
       24BB A600                      ldaa            0,X             
       24BD 183C                      pshy
       24BF 18E600                    ldab            CONT,Y
       24C2 183A                      aby
       24C4 18A700                    staa            0,Y
       24C7 1838                      puly
                      
       24C9 33                        pulb
       24CA BD25C8                    jsr             to_ascii
       24CD 17                        tba
                                      
       24CE CE41F9                    ldx             #string
       24D1 BD25B8                    jsr             sh_value
                      
       24D4 CE41F9                    ldx             #string
       24D7 BD250B                    jsr             show
                      
       24DA 20BA                      bra             wait_kb
                      
       24DC 810B      chk_fire        cmpa            #B_fire
       24DE 269F                      bne             inp_rest
       24E0 18A602                    ldaa            FILA_L,Y
       24E3 810A                      cmpa            #MIN_FIL
       24E5 2598                      blo             inp_rest
       24E7 810F                      cmpa            #MAX_FIL
       24E9 2294                      bhi             inp_rest
       24EB 18A601                    ldaa            COL_L,Y
       24EE 8101                      cmpa            #MIN_COL
       24F0 258D                      blo             inp_rest
       24F2 8106                      cmpa            #MAX_COL
       24F4 2289                      bhi             inp_rest
                      
                      * apago el led
       24F6                           %LED_INV
       24F6 B61000                    ldaa            PORTA
       24F9 8840                      eora            #SHF_LED
       24FB B71000                    staa            PORTA
                      
                      * libero variables locales
       24FE 32                        pula
       24FF 32                        pula
       2500 32                        pula
                      
       2501 18A602    return          ldaa            FILA_L,Y
       2504 18E601                    ldab            COL_L,Y         
                      
                      * restore de registros
       2507 1838                      puly
       2509 38                        pulx
                      
       250A 39                        rts
                      
                      *------------------------------------
                      * Show: Por IX direccion de string de
                      *       4 caracteres para imprimir
                      *------------------------------------
                      
       250B           show            EQU             *
                      
                      * back up de registros
       250B 3C                        pshx
       250C 37                        pshb
       250D 36                        psha
                      
                      * inicializo contador de posiciones
       250E C604                      ldab            #LENGTH
                                      
                      * loop para enviar a cada posicion
       2510 5A        sh_loop         decb
       2511 2B08                      bmi             sh_end          ; si no pase por todas las posiciones
       2513 A600                      ldaa            0,X             ; tomo del string el caracter
       2515 BD25D1                    jsr             outchar         ; y lo imprimo en la posicion actual
       2518 08                        inx
       2519 20F5                      bra             sh_loop
                      
                      * restore de registros                          
       251B 32        sh_end          pula
       251C 33                        pulb
       251D 38                        pulx
       251E 39                        rts
                      
                      *----------------------------------------------
                      * read_kb: AccA: tecla presionada
                      *          o -1
                      *----------------------------------------------
                      
       251F           read_kb         EQU             *
                      
                      * back up de registros
       251F 3C                        pshx
       2520 37                        pshb
                      
                      * creo espacio para variables locales
       2521                           %MALLOC         1               ; variable mascara de filas
       2521                           IF 1 > 0
       2521 30                        tsx
       2522 8F                        xgdx
       2523 830001                    subd    #1
       2526 8F                        xgdx
       2527 35                        txs
       2528                           ENDIF
       2528                           %MALLOC         1               ; variable mascara de columnas
       2528                           IF 1 > 0
       2528 30                        tsx
       2529 8F                        xgdx
       252A 830001                    subd    #1
       252D 8F                        xgdx
       252E 35                        txs
       252F                           ENDIF
       252F                           %MALLOC         1               ; variable contador de columnas
       252F                           IF 1 > 0
       252F 30                        tsx
       2530 8F                        xgdx
       2531 830001                    subd    #1
       2534 8F                        xgdx
       2535 35                        txs
       2536                           ENDIF
       2536                           %MALLOC         1               ; variable contador de tecla
       2536                           IF 1 > 0
       2536 30                        tsx
       2537 8F                        xgdx
       2538 830001                    subd    #1
       253B 8F                        xgdx
       253C 35                        txs
       253D                           ENDIF
                      
                      * inicializacion de variables
       253D 30                        tsx                             ; framepointer para acceder a locales
       253E C604                      ldab            #LENGTH         ; contador de fila
       2540 860E                      ldaa            #FIL0           ; mascara para la fila cero
       2542 A703                      staa            MASK_FIL,X      
       2544 8610                      ldaa            #COL0           ; mascara para la columna cero
       2546 A702                      staa            MASK_COL,X
       2548 4F                        clra                            ; contador de columna
       2549 A701                      staa            COL_CONT,X
       254B A700                      staa            KB_PRESS,X      ; variable con numero de tecla
                      
                      * loop para barrer y mirar teclado matricial
       254D 86FF      kb_loop         ldaa            #NONE
       254F 5A                        decb                            
       2550 2B2C                      bmi             kb_end          ; me fijo si recorrio todas las filas
       2552 A603                      ldaa            MASK_FIL,X      ; enmascaro la siguiente fila
       2554 B71003                    staa            ROWS
       2557 0D                        sec                             ; carry para rotacion de byte
       2558 49                        rola                            
       2559 A703                      staa            MASK_FIL,X      ; cambio variable mascara para la proxima fila
                      
       255B 8604                      ldaa            #LENGTH
       255D A701                      staa            COL_CONT,X
       255F 8610                      ldaa            #COL0           ; inicializo variable para barrer
       2561 A702                      staa            MASK_COL,X      ; la columna
                      
       2563 6A01      kb_col          dec             COL_CONT,X      ; me fijo si barrio todas las columnas
       2565 2BE6                      bmi             kb_loop
       2567 B61005                    ldaa            COLS            ; leo el puerto de columnas
       256A A402                      anda            MASK_COL,X      ; enmascaro columna
       256C 2707                      beq             kb_hit          ; si es nulo apretaron un boton
       256E 48                        lsla                            ; cambio mascara para proxima columna
       256F A702                      staa            MASK_COL,X
       2571 6C00                      inc             KB_PRESS,X      ; incremento numero de tecla
       2573 20EE                      bra             kb_col
                      
       2575 A600      kb_hit          ldaa            KB_PRESS,X      ; apretaron una tecla, lo comparo
       2577 B141F7                    cmpa            kb_buff         ; con la que fue apretada antes
       257A 2602                      bne             kb_end          ; aviso por led
                      
       257C 86FF      kb_none         ldaa            #NONE           ; en caso de que no apreten nada
                      
       257E B741F7    kb_end          staa            kb_buff         ; actualizo variable
                                              
                      * elimino variables locales
       2581                           %RELMEM         1
       2581                           IF 1 > 0
       2581 30                        tsx
       2582 8F                        xgdx
       2583 C30001                    addd    #1
       2586 8F                        xgdx
       2587 35                        txs
       2588                           ENDIF           
       2588                           %RELMEM         1
       2588                           IF 1 > 0
       2588 30                        tsx
       2589 8F                        xgdx
       258A C30001                    addd    #1
       258D 8F                        xgdx
       258E 35                        txs
       258F                           ENDIF           
       258F                           %RELMEM         1
       258F                           IF 1 > 0
       258F 30                        tsx
       2590 8F                        xgdx
       2591 C30001                    addd    #1
       2594 8F                        xgdx
       2595 35                        txs
       2596                           ENDIF           
       2596                           %RELMEM         1
       2596                           IF 1 > 0
       2596 30                        tsx
       2597 8F                        xgdx
       2598 C30001                    addd    #1
       259B 8F                        xgdx
       259C 35                        txs
       259D                           ENDIF           
                      
                      * restore de registros
       259D 33                        pulb
       259E 38                        pulx
                                      
       259F 39                        rts
                      
                      *----------------------------------------------
                      * init_kb: subrutina para inicializar teclado
                      *----------------------------------------------
                      
       25A0           init_kb         EQU             *
                      
                      * back up de registros
       25A0 36                        psha
                      
                      * inicializacion de teclado             
       25A1 86FF                      ldaa            #NONE
       25A3 B741F7                    staa            kb_buff
                      
                      * restore de registros
       25A6 32                        pula
       25A7 39                        rts
                      
                      *--------------------------------
                      * Inicializa display (y teclado)
                      *--------------------------------
                      
       25A8           initdis         EQU             *
                              
       25A8 36                        psha
                      
       25A9 860F                      ldaa            #$0F       ;set for input/output
       25AB B71007                    staa            DDRC       ;C output
       25AE B61002                    ldaa            PIOC       ;get current value
       25B1 84FE                      anda            #$FE       ;reset bit 0 to 0
       25B3 B71002                    staa            PIOC       ;send it out
                              
       25B6 32                        pula
       25B7 39                        rts
                      
                      *--------------------------------------------------------------
                      * sh_value: toma un string, desplaza los caracteres a izquierda
                      *           y agrega un nuevo caracter
                      *           IX: direccion del string
                      *           AccA: nuevo valor
                      *--------------------------------------------------------------
                      
       25B8           sh_value        EQU             *
                      
                      * back up de registros
       25B8 3C                        pshx
       25B9 37                        pshb
                      
                      * loop para desplazar caracteres
       25BA E601      v_loop          ldab            1,X             ;tomo elemento siguiente en string
       25BC 2705                      beq             v_end
       25BE E700                      stab            0,X             ;y lo guardo en posicion actual
       25C0 08                        inx
       25C1 20F7                      bra             v_loop          
                      
       25C3 A700      v_end           staa            0,X             ;ingresa nuevo caracter en cadena desplazada
                      
                      * restore de registros
       25C5 33                        pulb
       25C6 38                        pulx
       25C7 39                        rts
                      
                      *----------------------------------------------
                      * toAscii: toma numero de boton del teclado
                      *          y devuelve su correspondiente en
                      *          ascii
                      *          toma  AccB: valor del boton
                      *          devuelve AccB: ascii
                      *----------------------------------------------
                      
       25C8           to_ascii        EQU             *
                      
                      * back up de registros
       25C8 3C                        pshx
                      
                      * inicializacion de variables
       25C9 CE420D                    ldx             #kb_ascii       ; apunto a string con caracteres asignados por tecla
       25CC 3A                        abx                             ; me muevo en string con indice y recupero segun tecla presionada
       25CD E600                      ldab            0,X             
                      
                      * restore de registros
       25CF 38                        pulx
       25D0 39                        rts
                      
                      *--------------------------------
                      *Outchar: ACCA: Carácter a enviar
                      *           ACCB: Posición (0-3)
                      *--------------------------------
       25D1           outchar         EQU             *
                              
       25D1 F71005                    stab            PORTCL          ;set the position 
       25D4 B71004                    staa            PORTB           ;set the character
                              
       25D7 39                        rts
                      
                      *-------------------------
                      * clr_dis: Limpia display
                      *-------------------------
                      
       25D8           clr_dis         EQU             *
                      
                      * hago back up de registros
       25D8 37                        pshb
       25D9 36                        psha
                      
                      * inicializo el contador de posiciones
       25DA C604                      ldab            #LENGTH
                      
                      * loop para limpiar cada posicion
       25DC 5A        clrloop         decb
       25DD 2B07                      bmi             clr_end         ; si no pase por todas las posiciones
       25DF 8620                      ldaa            #ESP            ; cargo en esta posicion espacio blanco
       25E1 BD25D1                    jsr             outchar         ; y lo envio
       25E4 20F6                      bra             clrloop         
                      
                      * restore de registros
       25E6 32        clr_end         pula
       25E7 33                        pulb
       25E8 39                        rts
                      
                      *-------------------------------------------------------
                      * clrarray: subrutina para limpiar un array dado
                      * parametros: IX direccion del array
                      *             AccA cantidad de elementos del array
                      *             AccB caracter a poner
                      *-------------------------------------------------------
                      
       25E9           clrarray        EQU             *
                                      
       25E9 4A        cra_loop        deca                            ;contador con cantidad de elementos a limpiar
       25EA 2705                      beq             cra_end
       25EC E700                      stab            0,X             ;reemplazar elemento por caracter a poner
       25EE 08                        inx
       25EF 20F8                      bra             cra_loop
                                      
       25F1 E700      cra_end         stab            0,X
       25F3 39                        rts
                      
                      *=============================================
                      *Subrutina winner: Recibe por stack un 1 si
                      *gano el player 1 y un 0 si gano el player 2.
                      *=============================================
                      
                                      
       25F4           winner          EQU             *
       25F4                           %BACKUP
       25F4 36                        psha
       25F5 37                        pshb
       25F6 3C                        pshx
       25F7 183C                      pshy
       25F9 30                        tsx
       25FA A608                      ldaa            8,x     ;recupero parametro enviado por stack
       25FC 2708                      beq             win2
                      
       25FE           win1            EQU             *
       25FE CE4242                    ldx             #string1
       2601 BD250B                    jsr             show
       2604 2006                      bra             end
                      
       2606           win2            EQU             *
       2606 CE4247                    ldx             #string2
       2609 BD250B                    jsr             show
                              
       260C           end             EQU             *
       260C                           %RESTORE
       260C 1838                      puly
       260E 38                        pulx
       260F 33                        pulb
       2610 32                        pula
       2611 39                        rts
                      
                      *=============================================
                      *Subrutina turn: Recibe por stack un 1 si
                      *juega el player 1 y un 0 si juega el player 2.
                      *=============================================
                      
       2612           turn            EQU             *
       2612                           %BACKUP
       2612 36                        psha
       2613 37                        pshb
       2614 3C                        pshx
       2615 183C                      pshy
       2617 30                        tsx
       2618 A608                      ldaa            8,x
       261A 2708                      beq             p2_t
                      
       261C           p1_t            EQU             *
       261C CE424C                    ldx             #play1
       261F BD250B                    jsr             show
       2622 20E8                      bra             end
                              
       2624           p2_t            EQU             *
       2624 CE4250                    ldx             #play2
       2627 BD250B                    jsr             show
                      
       262A           end_t           EQU             *
       262A                           %RESTORE        
       262A 1838                      puly
       262C 38                        pulx
       262D 33                        pulb
       262E 32                        pula
       262F 39                        rts
                      
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
                      
                      ***KEYBOARD VARIABLES***
       41F7           kb_buff         RMB             1
       41F8           buffer          RMB             1
                      
       41F9 20202020  string          FCC             '    '
       41FD 00                        FCB             0
                      
       41FE 0A0B0C00  kb_value        FCB             10,11,12,0,13,14,15,0,1,2,3,0,4,5,6
       4202 0D0E0F00
       4206 01020300
       420A 040506
                      
       420D 41424320  kb_ascii        FCC             'ABC DEF 123 456'
       4211 44454620
       4215 31323320
       4219 343536
                      
                      ***ARREGLO DE BARCOS TOCADOS***
       0003           SUB             EQU             3
       0003           CAR             EQU             3
       0002           EL_SIZE         EQU             2
       0009           CANTIDAD        EQU             SUB*1+CAR*2
       0013           HITSIZE         EQU             EL_SIZE*CANTIDAD+1
       421C           hitlist         RMB             HITSIZE*2
                      
                      ***DISPLAY MESSAGES VARIABLES***
       4242 57494E31  string1         FCC     'WIN1'
       4246 00                        FCB     00
       4247 57494E32  string2         FCC     'WIN2'
       424B 00                        FCB     00
                      
       424C 5031      play1           FCC     'P1'
       424E 0000                      FCB     0,0
       4250 5032      play2           FCC     'P2'
       4252 0000                      FCB     0,0
                      
                      ***MATRICES DE TABLERO DE CADA JUGADOR***
       5000                           ORG             $5000
       5000 03030303  p_board         FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       5004 0303
       5006 03038303                  FCB             WAT_NP,WAT_NP,SHI_NP,WAT_NP,SHI_NP,WAT_NP
       500A 8303
       500C 03038303                  FCB             WAT_NP,WAT_NP,SHI_NP,WAT_NP,SHI_NP,WAT_NP
       5010 8303
       5012 03830303                  FCB             WAT_NP,SHI_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       5016 0303
       5018 03830343                  FCB             WAT_NP,SHI_NP,WAT_NP,SUB_NP,WAT_NP,SUB_NP
       501C 0343
       501E 03030303                  FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,SUB_NP,WAT_NP
       5022 4303
                      
       5024 03038303                  FCB             WAT_NP,WAT_NP,SHI_NP,WAT_NP,WAT_NP,WAT_NP
       5028 0303
       502A 03038303                  FCB             WAT_NP,WAT_NP,SHI_NP,WAT_NP,WAT_NP,WAT_NP
       502E 0303
       5030 03430303                  FCB             WAT_NP,SUB_NP,WAT_NP,WAT_NP,SHI_NP,WAT_NP
       5034 8303
       5036 03030303                  FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,SHI_NP,WAT_NP
       503A 8303
       503C 03034303                  FCB             WAT_NP,WAT_NP,SUB_NP,WAT_NP,WAT_NP,WAT_NP
       5040 0303
       5042 83830303                  FCB             SHI_NP,SHI_NP,WAT_NP,WAT_NP,WAT_NP,SUB_NP
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
     ytes:   1745         # main_bn #
     RC:     D0CA         ###########
 