################################################################################
#                                                                              #
#    Micro Series 6801 Assembler V2.00/DOS              26/Nov/17  23:56:31    #
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
       2003 BD2571                    jsr             initdis         ;inicialización de display 7 seg
       2006 BD2569                    jsr             init_kb         ;inicialización de teclado
       2009 CE421C                    ldx             #hitlist
       200C 3C                        pshx                            ;se envía inicio de arreglo de barcos a poner en 0
       200D 8626                      ldaa            #(HITSIZE*2)
       200F 36                        psha                            ;se envía el tamaño del arreglo a limpiar
       2010 BD2413                    jsr             clrzone
       2013 8601                      ldaa            #P2             ;se inicia atacando al jugador 2
       2015 B741F4                    staa            player          ;se inicia con el jugador 1 (atacando al 2)
                      
       2018           main            EQU             *
       2018 BD2440                    jsr             input           ;se llama a la subrutina de interfaz de usuario
       201B B741F5                    staa            letra_p         ;se recibe en AccA la letra jugada (fila) y en AccB el número (columna)
       201E F741F6                    stab            num_p           ;se guarda la jugada
                      
       2021 CE421C                    ldx             #hitlist
       2024 3C                        pshx                            ;se envía la posición del primer arreglo de barcos tocados
       2025 B641F5                    ldaa            letra_p
       2028 36                        psha                            ;se envía la letra jugada
       2029 F641F6                    ldab            num_p
       202C 37                        pshb                            ;se envía el número jugado
       202D B641F4                    ldaa            player
       2030 36                        psha                            ;se envía el jugador a atacar
       2031 BD2392                    jsr             play_src        ;se recibirá el carry en 1 si la posición corresponde a un barco ya atacado
       2034 255C                      bcs             next_p          ;si la posición ya fue jugada, se pierde el turno y pasa al siguiente
       2036                           %CLRST          5               ;se limpia el stack
       0005           CONT_M          SET     5
       2036                           IF CONT_M > 0
       2036 31                        ins
       2037                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2037                           IF CONT_M > 0
       2037 31                        ins
       2038                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2038                           IF CONT_M > 0
       2038 31                        ins
       2039                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2039                           IF CONT_M > 0
       2039 31                        ins
       203A                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       203A                           IF CONT_M > 0
       203A 31                        ins
       203B                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       203B                           ENDIF
       203B                           ENDIF
       203B                           ENDIF
       203B                           ENDIF
       203B                           ENDIF
       203B                           ENDIF
                                      
       203B B641F5                    ldaa            letra_p
       203E 36                        psha                            ;se envía le letra jugada
       203F F641F6                    ldab            num_p
       2042 37                        pshb                            ;se envía el número jugado
       2043 CE5000                    ldx             #p_board
       2046 3C                        pshx                            ;se envía la dirección de inicio de los arreglos de tablero
       2047 F641F4                    ldab            player
       204A 37                        pshb                            ;se envía el jugador a atacar
       204B BD2330                    jsr             fetchpos
       204E                           %CLRST          4               ;se limpia el stack
       0004           CONT_M          SET     4
       204E                           IF CONT_M > 0
       204E 31                        ins
       204F                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       204F                           IF CONT_M > 0
       204F 31                        ins
       2050                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2050                           IF CONT_M > 0
       2050 31                        ins
       2051                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2051                           IF CONT_M > 0
       2051 31                        ins
       2052                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2052                           ENDIF
       2052                           ENDIF
       2052                           ENDIF
       2052                           ENDIF
       2052                           ENDIF
       2052 32                        pula                            ;se toma del stack el dato del byte jugado
                      
       2053 CE421C                    ldx             #hitlist
       2056 3C                        pshx                            ;se envía inicio del arreglo de barcos atacados
       2057 36                        psha                            ;se envía el byte a jugado
       2058 B641F5                    ldaa            letra_p
       205B 36                        psha                            ;se envía le letra jugada
       205C F641F6                    ldab            num_p
       205F 37                        pshb                            ;se envía el número jugado
       2060 CE5000                    ldx             #p_board
       2063 3C                        pshx                            ;se envía la dirección de inicio de los arreglos de tablero
       2064 F641F4                    ldab            player
       2067 37                        pshb                            ;se envía el jugador a atacar
       2068 BD209F                    jsr             gameplay
       206B                           %CLRST          8               ;se limpia el stack
       0008           CONT_M          SET     8
       206B                           IF CONT_M > 0
       206B 31                        ins
       206C                           %CLRST  CONT_M-1
       0007           CONT_M          SET     CONT_M-1
       206C                           IF CONT_M > 0
       206C 31                        ins
       206D                           %CLRST  CONT_M-1
       0006           CONT_M          SET     CONT_M-1
       206D                           IF CONT_M > 0
       206D 31                        ins
       206E                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       206E                           IF CONT_M > 0
       206E 31                        ins
       206F                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       206F                           IF CONT_M > 0
       206F 31                        ins
       2070                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2070                           IF CONT_M > 0
       2070 31                        ins
       2071                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2071                           IF CONT_M > 0
       2071 31                        ins
       2072                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2072                           IF CONT_M > 0
       2072 31                        ins
       2073                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2073                           ENDIF
       2073                           ENDIF
       2073                           ENDIF
       2073                           ENDIF
       2073                           ENDIF
       2073                           ENDIF
       2073                           ENDIF
       2073                           ENDIF
       2073                           ENDIF
                      
       2073 CE5000                    ldx             #p_board
       2076 3C                        pshx                            ;se envía el inicio de las matrices de tableros
       2077 B641F4                    ldaa            player
       207A 36                        psha                            ;se envía el jugador atacado
       207B CE5F00                    ldx             #display
       207E 3C                        pshx                            ;se envía la matriz de display (destino a copiar)
       207F 8624                      ldaa            #ARRAY_L
       2081 36                        psha                            ;se envía el tamaño de la matriz tablero
       2082 BD2307                    jsr             mirror          ;se envía el tablero del jugador atacado al display
                                      
       2085 CE421C                    ldx             #hitlist
       2088 3C                        pshx                            ;se envía la posición del primer arreglo de barcos tocados
       2089 B641F4                    ldaa            player
       208C 36                        psha                            ;se envía el jugador a atacar
       208D BD23C5                    jsr             play_cnt        
       2090 2509                      bcs             end_game        ;si se recibe el Carry en 1, el jugador de turno ganó
                      
       2092           next_p          EQU             *
       2092 BD242B                    jsr             whoplays        ;se recibirá en AccA el jugador a atacar a continuación
       2095 B741F4                    staa            player          ;se habilita la siguiente jugada
                      
       2098 7E2018                    jmp             main
                      
       209B           end_game        EQU             *
                      *               ldaa            player
                      *               psha
                      *               jsr             winner
       209B 20FE                      bra             end_game
                      
       209D           end_loop        EQU             *
       209D 20FE                      bra             end_loop
                      
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
       209F           gameplay        EQU             *
       209F                           %BACKUP
       209F 36                        psha
       20A0 37                        pshb
       20A1 3C                        pshx
       20A2 183C                      pshy
                                      
       20A4 30                        tsx                             ;se usa el IX como frame pointer
       20A5 A60D                      ldaa            13,x            ;se carga en AccA el byte a analizar
       20A7 84C0                      anda            #11000000b      ;se enmascan los bits 0 a 5, para analizar los más significativos
                      
       20A9           gp_water        EQU             *
       20A9 2620                      bne             gp_sub          ;se estudia si era agua
       20AB                           %WRITEPOS       #WATER          ;se cambia el color en el tablero
       20AB C608                      ldab            #WATER
       20AD 37                        pshb                            ;se envía el byte a escribir
       20AE E60C                      ldab            12,x
       20B0 37                        pshb                            ;se envía letra (fila)
       20B1 E60B                      ldab            11,x
       20B3 37                        pshb                            ;se envía num (columna)
       20B4 1AEE09                    ldy             9,x
       20B7 183C                      pshy                            ;se envía dirección de inicio de matriz
       20B9 E608                      ldab            8,x
       20BB 37                        pshb                            ;se envía jugador atacado
       20BC BD2361                    jsr             writepos
       20BF                           %CLRST          6
       0006           CONT_M          SET     6
       20BF                           IF CONT_M > 0
       20BF 31                        ins
       20C0                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       20C0                           IF CONT_M > 0
       20C0 31                        ins
       20C1                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       20C1                           IF CONT_M > 0
       20C1 31                        ins
       20C2                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       20C2                           IF CONT_M > 0
       20C2 31                        ins
       20C3                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       20C3                           IF CONT_M > 0
       20C3 31                        ins
       20C4                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       20C4                           IF CONT_M > 0
       20C4 31                        ins
       20C5                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       20C5                           ENDIF
       20C5                           ENDIF
       20C5                           ENDIF
       20C5                           ENDIF
       20C5                           ENDIF
       20C5                           ENDIF
       20C5                           ENDIF
       20C5                           %RESTORE
       20C5 1838                      puly
       20C7 38                        pulx
       20C8 33                        pulb
       20C9 32                        pula
       20CA 39                        rts
                      
       20CB           gp_sub          EQU             *
       20CB 8140                      cmpa            #01000000b      ;se estudia si era un submarino
       20CD 2636                      bne             gp_shi_r
       20CF                           %WRITEPOS       #SUNK_SUB       ;se cambia el color en el tablero
       20CF C650                      ldab            #SUNK_SUB
       20D1 37                        pshb                            ;se envía el byte a escribir
       20D2 E60C                      ldab            12,x
       20D4 37                        pshb                            ;se envía letra (fila)
       20D5 E60B                      ldab            11,x
       20D7 37                        pshb                            ;se envía num (columna)
       20D8 1AEE09                    ldy             9,x
       20DB 183C                      pshy                            ;se envía dirección de inicio de matriz
       20DD E608                      ldab            8,x
       20DF 37                        pshb                            ;se envía jugador atacado
       20E0 BD2361                    jsr             writepos
       20E3                           %CLRST          6
       0006           CONT_M          SET     6
       20E3                           IF CONT_M > 0
       20E3 31                        ins
       20E4                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       20E4                           IF CONT_M > 0
       20E4 31                        ins
       20E5                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       20E5                           IF CONT_M > 0
       20E5 31                        ins
       20E6                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       20E6                           IF CONT_M > 0
       20E6 31                        ins
       20E7                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       20E7                           IF CONT_M > 0
       20E7 31                        ins
       20E8                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       20E8                           IF CONT_M > 0
       20E8 31                        ins
       20E9                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       20E9                           ENDIF
       20E9                           ENDIF
       20E9                           ENDIF
       20E9                           ENDIF
       20E9                           ENDIF
       20E9                           ENDIF
       20E9                           ENDIF
       20E9                           %PLAY_WRT                       ;se agraga la posición al array de barcos atacados
       20E9 E60C                      ldab            12,x
       20EB 37                        pshb                            ;se envía letra (fila)
       20EC E60B                      ldab            11,x
       20EE 37                        pshb                            ;se envía num (columna)
       20EF 1AEE0E                    ldy             14,x
       20F2 183C                      pshy                            ;se envía inicio de array
       20F4 E608                      ldab            8,x
       20F6 37                        pshb                            ;se envía jugador atacado
       20F7 BD23E9                    jsr             play_wrt
       20FA                           %CLRST          5
       0005           CONT_M          SET     5
       20FA                           IF CONT_M > 0
       20FA 31                        ins
       20FB                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       20FB                           IF CONT_M > 0
       20FB 31                        ins
       20FC                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       20FC                           IF CONT_M > 0
       20FC 31                        ins
       20FD                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       20FD                           IF CONT_M > 0
       20FD 31                        ins
       20FE                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       20FE                           IF CONT_M > 0
       20FE 31                        ins
       20FF                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       20FF                           ENDIF
       20FF                           ENDIF
       20FF                           ENDIF
       20FF                           ENDIF
       20FF                           ENDIF
       20FF                           ENDIF
       20FF                           %RESTORE
       20FF 1838                      puly
       2101 38                        pulx
       2102 33                        pulb
       2103 32                        pula
       2104 39                        rts
                      
       2105           gp_shi_r        EQU             *               ;si no era ninguno de los anteriores, es un carguero
       2105 E60B                      ldab            11,x
       2107 C106                      cmpb            #MAX_C
       2109 276D                      beq             gp_shi_l        ;si está en el borde derecho, se continúa
       210B E60C                      ldab            12,x
       210D 37                        pshb                            ;se envía la letra (fila)
       210E E60B                      ldab            11,x
       2110 5C                        incb                            ;se avanza a la derecha
       2111 37                        pshb                            ;se envía el num (columna)
       2112 1AEE0E                    ldy             14,x
       2115 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       2117 E608                      ldab            8,x
       2119 37                        pshb                            ;se envía el jugador atacado
       211A BD2392                    jsr             play_src        ;se busca la posición de la derecha
       211D                           %CLRST          5
       0005           CONT_M          SET     5
       211D                           IF CONT_M > 0
       211D 31                        ins
       211E                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       211E                           IF CONT_M > 0
       211E 31                        ins
       211F                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       211F                           IF CONT_M > 0
       211F 31                        ins
       2120                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2120                           IF CONT_M > 0
       2120 31                        ins
       2121                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2121                           IF CONT_M > 0
       2121 31                        ins
       2122                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2122                           ENDIF
       2122                           ENDIF
       2122                           ENDIF
       2122                           ENDIF
       2122                           ENDIF
       2122                           ENDIF
       2122 2454                      bcc             gp_shi_l        ;si no había barco atacado en la derecha, se busca la izquierda
       2124                           %WRITEPOS       #SUNK_SHI       
       2124 C690                      ldab            #SUNK_SHI
       2126 37                        pshb                            ;se envía el byte a escribir
       2127 E60C                      ldab            12,x
       2129 37                        pshb                            ;se envía letra (fila)
       212A E60B                      ldab            11,x
       212C 37                        pshb                            ;se envía num (columna)
       212D 1AEE09                    ldy             9,x
       2130 183C                      pshy                            ;se envía dirección de inicio de matriz
       2132 E608                      ldab            8,x
       2134 37                        pshb                            ;se envía jugador atacado
       2135 BD2361                    jsr             writepos
       2138                           %CLRST          6
       0006           CONT_M          SET     6
       2138                           IF CONT_M > 0
       2138 31                        ins
       2139                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2139                           IF CONT_M > 0
       2139 31                        ins
       213A                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       213A                           IF CONT_M > 0
       213A 31                        ins
       213B                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       213B                           IF CONT_M > 0
       213B 31                        ins
       213C                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       213C                           IF CONT_M > 0
       213C 31                        ins
       213D                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       213D                           IF CONT_M > 0
       213D 31                        ins
       213E                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       213E                           ENDIF
       213E                           ENDIF
       213E                           ENDIF
       213E                           ENDIF
       213E                           ENDIF
       213E                           ENDIF
       213E                           ENDIF
       213E 6C0B                      inc             11,x
       2140                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       2140 C690                      ldab            #SUNK_SHI
       2142 37                        pshb                            ;se envía el byte a escribir
       2143 E60C                      ldab            12,x
       2145 37                        pshb                            ;se envía letra (fila)
       2146 E60B                      ldab            11,x
       2148 37                        pshb                            ;se envía num (columna)
       2149 1AEE09                    ldy             9,x
       214C 183C                      pshy                            ;se envía dirección de inicio de matriz
       214E E608                      ldab            8,x
       2150 37                        pshb                            ;se envía jugador atacado
       2151 BD2361                    jsr             writepos
       2154                           %CLRST          6
       0006           CONT_M          SET     6
       2154                           IF CONT_M > 0
       2154 31                        ins
       2155                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2155                           IF CONT_M > 0
       2155 31                        ins
       2156                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2156                           IF CONT_M > 0
       2156 31                        ins
       2157                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2157                           IF CONT_M > 0
       2157 31                        ins
       2158                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2158                           IF CONT_M > 0
       2158 31                        ins
       2159                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2159                           IF CONT_M > 0
       2159 31                        ins
       215A                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       215A                           ENDIF
       215A                           ENDIF
       215A                           ENDIF
       215A                           ENDIF
       215A                           ENDIF
       215A                           ENDIF
       215A                           ENDIF
       215A 6A0B                      dec             11,x            ;se recupera el dato
       215C                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       215C E60C                      ldab            12,x
       215E 37                        pshb                            ;se envía letra (fila)
       215F E60B                      ldab            11,x
       2161 37                        pshb                            ;se envía num (columna)
       2162 1AEE0E                    ldy             14,x
       2165 183C                      pshy                            ;se envía inicio de array
       2167 E608                      ldab            8,x
       2169 37                        pshb                            ;se envía jugador atacado
       216A BD23E9                    jsr             play_wrt
       216D                           %CLRST          5
       0005           CONT_M          SET     5
       216D                           IF CONT_M > 0
       216D 31                        ins
       216E                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       216E                           IF CONT_M > 0
       216E 31                        ins
       216F                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       216F                           IF CONT_M > 0
       216F 31                        ins
       2170                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2170                           IF CONT_M > 0
       2170 31                        ins
       2171                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2171                           IF CONT_M > 0
       2171 31                        ins
       2172                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2172                           ENDIF
       2172                           ENDIF
       2172                           ENDIF
       2172                           ENDIF
       2172                           ENDIF
       2172                           ENDIF
       2172                           %RESTORE
       2172 1838                      puly
       2174 38                        pulx
       2175 33                        pulb
       2176 32                        pula
       2177 39                        rts
                      
       2178           gp_shi_l        EQU             *
       2178 E60B                      ldab            11,x
       217A C101                      cmpb            #MIN_C
       217C 276D                      beq             gp_shi_u        ;si está en el borde izquierdo, se continúa
       217E E60C                      ldab            12,x
       2180 37                        pshb                            ;se envía la letra (fila)
       2181 E60B                      ldab            11,x
       2183 5A                        decb                            ;se retrocede a la izquierda
       2184 37                        pshb                            ;se envía el número (columna)
       2185 1AEE0E                    ldy             14,x
       2188 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       218A E608                      ldab            8,x
       218C 37                        pshb                            ;se envía el jugador atacado
       218D BD2392                    jsr             play_src        ;se busca la posición de la izquierda
       2190                           %CLRST          5
       0005           CONT_M          SET     5
       2190                           IF CONT_M > 0
       2190 31                        ins
       2191                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2191                           IF CONT_M > 0
       2191 31                        ins
       2192                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2192                           IF CONT_M > 0
       2192 31                        ins
       2193                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2193                           IF CONT_M > 0
       2193 31                        ins
       2194                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2194                           IF CONT_M > 0
       2194 31                        ins
       2195                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2195                           ENDIF
       2195                           ENDIF
       2195                           ENDIF
       2195                           ENDIF
       2195                           ENDIF
       2195                           ENDIF
       2195 2454                      bcc             gp_shi_u        ;si no había barco atacado a la izquierda, se busca arriba
       2197                           %WRITEPOS       #SUNK_SHI       
       2197 C690                      ldab            #SUNK_SHI
       2199 37                        pshb                            ;se envía el byte a escribir
       219A E60C                      ldab            12,x
       219C 37                        pshb                            ;se envía letra (fila)
       219D E60B                      ldab            11,x
       219F 37                        pshb                            ;se envía num (columna)
       21A0 1AEE09                    ldy             9,x
       21A3 183C                      pshy                            ;se envía dirección de inicio de matriz
       21A5 E608                      ldab            8,x
       21A7 37                        pshb                            ;se envía jugador atacado
       21A8 BD2361                    jsr             writepos
       21AB                           %CLRST          6
       0006           CONT_M          SET     6
       21AB                           IF CONT_M > 0
       21AB 31                        ins
       21AC                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       21AC                           IF CONT_M > 0
       21AC 31                        ins
       21AD                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       21AD                           IF CONT_M > 0
       21AD 31                        ins
       21AE                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       21AE                           IF CONT_M > 0
       21AE 31                        ins
       21AF                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       21AF                           IF CONT_M > 0
       21AF 31                        ins
       21B0                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       21B0                           IF CONT_M > 0
       21B0 31                        ins
       21B1                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       21B1                           ENDIF
       21B1                           ENDIF
       21B1                           ENDIF
       21B1                           ENDIF
       21B1                           ENDIF
       21B1                           ENDIF
       21B1                           ENDIF
       21B1 6A0B                      dec             11,x
       21B3                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       21B3 C690                      ldab            #SUNK_SHI
       21B5 37                        pshb                            ;se envía el byte a escribir
       21B6 E60C                      ldab            12,x
       21B8 37                        pshb                            ;se envía letra (fila)
       21B9 E60B                      ldab            11,x
       21BB 37                        pshb                            ;se envía num (columna)
       21BC 1AEE09                    ldy             9,x
       21BF 183C                      pshy                            ;se envía dirección de inicio de matriz
       21C1 E608                      ldab            8,x
       21C3 37                        pshb                            ;se envía jugador atacado
       21C4 BD2361                    jsr             writepos
       21C7                           %CLRST          6
       0006           CONT_M          SET     6
       21C7                           IF CONT_M > 0
       21C7 31                        ins
       21C8                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       21C8                           IF CONT_M > 0
       21C8 31                        ins
       21C9                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       21C9                           IF CONT_M > 0
       21C9 31                        ins
       21CA                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       21CA                           IF CONT_M > 0
       21CA 31                        ins
       21CB                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       21CB                           IF CONT_M > 0
       21CB 31                        ins
       21CC                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       21CC                           IF CONT_M > 0
       21CC 31                        ins
       21CD                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       21CD                           ENDIF
       21CD                           ENDIF
       21CD                           ENDIF
       21CD                           ENDIF
       21CD                           ENDIF
       21CD                           ENDIF
       21CD                           ENDIF
       21CD 6C0B                      inc             11,x            ;se recupera el dato
       21CF                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       21CF E60C                      ldab            12,x
       21D1 37                        pshb                            ;se envía letra (fila)
       21D2 E60B                      ldab            11,x
       21D4 37                        pshb                            ;se envía num (columna)
       21D5 1AEE0E                    ldy             14,x
       21D8 183C                      pshy                            ;se envía inicio de array
       21DA E608                      ldab            8,x
       21DC 37                        pshb                            ;se envía jugador atacado
       21DD BD23E9                    jsr             play_wrt
       21E0                           %CLRST          5
       0005           CONT_M          SET     5
       21E0                           IF CONT_M > 0
       21E0 31                        ins
       21E1                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       21E1                           IF CONT_M > 0
       21E1 31                        ins
       21E2                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       21E2                           IF CONT_M > 0
       21E2 31                        ins
       21E3                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       21E3                           IF CONT_M > 0
       21E3 31                        ins
       21E4                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       21E4                           IF CONT_M > 0
       21E4 31                        ins
       21E5                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       21E5                           ENDIF
       21E5                           ENDIF
       21E5                           ENDIF
       21E5                           ENDIF
       21E5                           ENDIF
       21E5                           ENDIF
       21E5                           %RESTORE
       21E5 1838                      puly
       21E7 38                        pulx
       21E8 33                        pulb
       21E9 32                        pula
       21EA 39                        rts
                      
       21EB           gp_shi_u        EQU             *
       21EB E60C                      ldab            12,x
       21ED C10A                      cmpb            #MIN_F
       21EF 276D                      beq             gp_shi_d        ;si está en el borde superior, se continúa
       21F1 E60C                      ldab            12,x
       21F3 5A                        decb                            ;se va a la posición de arriba
       21F4 37                        pshb                            ;se envía la letra (fila)
       21F5 E60B                      ldab            11,x
       21F7 37                        pshb                            ;se envía el num (columna)
       21F8 1AEE0E                    ldy             14,x
       21FB 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       21FD E608                      ldab            8,x
       21FF 37                        pshb                            ;se envía el jugador atacado
       2200 BD2392                    jsr             play_src        ;se busca la posición de arriba
       2203                           %CLRST          5
       0005           CONT_M          SET     5
       2203                           IF CONT_M > 0
       2203 31                        ins
       2204                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2204                           IF CONT_M > 0
       2204 31                        ins
       2205                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2205                           IF CONT_M > 0
       2205 31                        ins
       2206                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2206                           IF CONT_M > 0
       2206 31                        ins
       2207                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2207                           IF CONT_M > 0
       2207 31                        ins
       2208                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2208                           ENDIF
       2208                           ENDIF
       2208                           ENDIF
       2208                           ENDIF
       2208                           ENDIF
       2208                           ENDIF
       2208 2454                      bcc             gp_shi_d        ;si no había barco atacado arriba, se busca abajo
       220A                           %WRITEPOS       #SUNK_SHI       
       220A C690                      ldab            #SUNK_SHI
       220C 37                        pshb                            ;se envía el byte a escribir
       220D E60C                      ldab            12,x
       220F 37                        pshb                            ;se envía letra (fila)
       2210 E60B                      ldab            11,x
       2212 37                        pshb                            ;se envía num (columna)
       2213 1AEE09                    ldy             9,x
       2216 183C                      pshy                            ;se envía dirección de inicio de matriz
       2218 E608                      ldab            8,x
       221A 37                        pshb                            ;se envía jugador atacado
       221B BD2361                    jsr             writepos
       221E                           %CLRST          6
       0006           CONT_M          SET     6
       221E                           IF CONT_M > 0
       221E 31                        ins
       221F                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       221F                           IF CONT_M > 0
       221F 31                        ins
       2220                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2220                           IF CONT_M > 0
       2220 31                        ins
       2221                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2221                           IF CONT_M > 0
       2221 31                        ins
       2222                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2222                           IF CONT_M > 0
       2222 31                        ins
       2223                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2223                           IF CONT_M > 0
       2223 31                        ins
       2224                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2224                           ENDIF
       2224                           ENDIF
       2224                           ENDIF
       2224                           ENDIF
       2224                           ENDIF
       2224                           ENDIF
       2224                           ENDIF
       2224 6A0C                      dec             12,x
       2226                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       2226 C690                      ldab            #SUNK_SHI
       2228 37                        pshb                            ;se envía el byte a escribir
       2229 E60C                      ldab            12,x
       222B 37                        pshb                            ;se envía letra (fila)
       222C E60B                      ldab            11,x
       222E 37                        pshb                            ;se envía num (columna)
       222F 1AEE09                    ldy             9,x
       2232 183C                      pshy                            ;se envía dirección de inicio de matriz
       2234 E608                      ldab            8,x
       2236 37                        pshb                            ;se envía jugador atacado
       2237 BD2361                    jsr             writepos
       223A                           %CLRST          6
       0006           CONT_M          SET     6
       223A                           IF CONT_M > 0
       223A 31                        ins
       223B                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       223B                           IF CONT_M > 0
       223B 31                        ins
       223C                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       223C                           IF CONT_M > 0
       223C 31                        ins
       223D                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       223D                           IF CONT_M > 0
       223D 31                        ins
       223E                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       223E                           IF CONT_M > 0
       223E 31                        ins
       223F                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       223F                           IF CONT_M > 0
       223F 31                        ins
       2240                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2240                           ENDIF
       2240                           ENDIF
       2240                           ENDIF
       2240                           ENDIF
       2240                           ENDIF
       2240                           ENDIF
       2240                           ENDIF
       2240 6C0C                      inc             12,x            ;se recupera el dato
       2242                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       2242 E60C                      ldab            12,x
       2244 37                        pshb                            ;se envía letra (fila)
       2245 E60B                      ldab            11,x
       2247 37                        pshb                            ;se envía num (columna)
       2248 1AEE0E                    ldy             14,x
       224B 183C                      pshy                            ;se envía inicio de array
       224D E608                      ldab            8,x
       224F 37                        pshb                            ;se envía jugador atacado
       2250 BD23E9                    jsr             play_wrt
       2253                           %CLRST          5
       0005           CONT_M          SET     5
       2253                           IF CONT_M > 0
       2253 31                        ins
       2254                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2254                           IF CONT_M > 0
       2254 31                        ins
       2255                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2255                           IF CONT_M > 0
       2255 31                        ins
       2256                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2256                           IF CONT_M > 0
       2256 31                        ins
       2257                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2257                           IF CONT_M > 0
       2257 31                        ins
       2258                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2258                           ENDIF
       2258                           ENDIF
       2258                           ENDIF
       2258                           ENDIF
       2258                           ENDIF
       2258                           ENDIF
       2258                           %RESTORE
       2258 1838                      puly
       225A 38                        pulx
       225B 33                        pulb
       225C 32                        pula
       225D 39                        rts
                      
       225E           gp_shi_d        EQU             *
       225E E60C                      ldab            12,x
       2260 C10F                      cmpb            #MAX_F
       2262 276D                      beq             gp_shi_h        ;si está en el borde inferior, se continúa
       2264 E60C                      ldab            12,x
       2266 5C                        incb                            ;se va a la posición de abajo
       2267 37                        pshb                            ;se envía la letra (fila)
       2268 E60B                      ldab            11,x
       226A 37                        pshb                            ;se envía el num (columna)
       226B 1AEE0E                    ldy             14,x
       226E 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       2270 E608                      ldab            8,x
       2272 37                        pshb                            ;se envía el jugador atacado
       2273 BD2392                    jsr             play_src        ;se busca la posición de abajo
       2276                           %CLRST          5
       0005           CONT_M          SET     5
       2276                           IF CONT_M > 0
       2276 31                        ins
       2277                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2277                           IF CONT_M > 0
       2277 31                        ins
       2278                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2278                           IF CONT_M > 0
       2278 31                        ins
       2279                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2279                           IF CONT_M > 0
       2279 31                        ins
       227A                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       227A                           IF CONT_M > 0
       227A 31                        ins
       227B                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       227B                           ENDIF
       227B                           ENDIF
       227B                           ENDIF
       227B                           ENDIF
       227B                           ENDIF
       227B                           ENDIF
       227B 2454                      bcc             gp_shi_h        ;si no había barco atacado abajo, el carguero está tocado
       227D                           %WRITEPOS       #SUNK_SHI       
       227D C690                      ldab            #SUNK_SHI
       227F 37                        pshb                            ;se envía el byte a escribir
       2280 E60C                      ldab            12,x
       2282 37                        pshb                            ;se envía letra (fila)
       2283 E60B                      ldab            11,x
       2285 37                        pshb                            ;se envía num (columna)
       2286 1AEE09                    ldy             9,x
       2289 183C                      pshy                            ;se envía dirección de inicio de matriz
       228B E608                      ldab            8,x
       228D 37                        pshb                            ;se envía jugador atacado
       228E BD2361                    jsr             writepos
       2291                           %CLRST          6
       0006           CONT_M          SET     6
       2291                           IF CONT_M > 0
       2291 31                        ins
       2292                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2292                           IF CONT_M > 0
       2292 31                        ins
       2293                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2293                           IF CONT_M > 0
       2293 31                        ins
       2294                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2294                           IF CONT_M > 0
       2294 31                        ins
       2295                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2295                           IF CONT_M > 0
       2295 31                        ins
       2296                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2296                           IF CONT_M > 0
       2296 31                        ins
       2297                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2297                           ENDIF
       2297                           ENDIF
       2297                           ENDIF
       2297                           ENDIF
       2297                           ENDIF
       2297                           ENDIF
       2297                           ENDIF
       2297 6C0C                      inc             12,x
       2299                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       2299 C690                      ldab            #SUNK_SHI
       229B 37                        pshb                            ;se envía el byte a escribir
       229C E60C                      ldab            12,x
       229E 37                        pshb                            ;se envía letra (fila)
       229F E60B                      ldab            11,x
       22A1 37                        pshb                            ;se envía num (columna)
       22A2 1AEE09                    ldy             9,x
       22A5 183C                      pshy                            ;se envía dirección de inicio de matriz
       22A7 E608                      ldab            8,x
       22A9 37                        pshb                            ;se envía jugador atacado
       22AA BD2361                    jsr             writepos
       22AD                           %CLRST          6
       0006           CONT_M          SET     6
       22AD                           IF CONT_M > 0
       22AD 31                        ins
       22AE                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       22AE                           IF CONT_M > 0
       22AE 31                        ins
       22AF                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       22AF                           IF CONT_M > 0
       22AF 31                        ins
       22B0                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       22B0                           IF CONT_M > 0
       22B0 31                        ins
       22B1                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       22B1                           IF CONT_M > 0
       22B1 31                        ins
       22B2                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       22B2                           IF CONT_M > 0
       22B2 31                        ins
       22B3                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       22B3                           ENDIF
       22B3                           ENDIF
       22B3                           ENDIF
       22B3                           ENDIF
       22B3                           ENDIF
       22B3                           ENDIF
       22B3                           ENDIF
       22B3 6A0C                      dec             12,x            ;se recupera el dato
       22B5                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       22B5 E60C                      ldab            12,x
       22B7 37                        pshb                            ;se envía letra (fila)
       22B8 E60B                      ldab            11,x
       22BA 37                        pshb                            ;se envía num (columna)
       22BB 1AEE0E                    ldy             14,x
       22BE 183C                      pshy                            ;se envía inicio de array
       22C0 E608                      ldab            8,x
       22C2 37                        pshb                            ;se envía jugador atacado
       22C3 BD23E9                    jsr             play_wrt
       22C6                           %CLRST          5
       0005           CONT_M          SET     5
       22C6                           IF CONT_M > 0
       22C6 31                        ins
       22C7                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       22C7                           IF CONT_M > 0
       22C7 31                        ins
       22C8                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       22C8                           IF CONT_M > 0
       22C8 31                        ins
       22C9                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       22C9                           IF CONT_M > 0
       22C9 31                        ins
       22CA                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       22CA                           IF CONT_M > 0
       22CA 31                        ins
       22CB                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       22CB                           ENDIF
       22CB                           ENDIF
       22CB                           ENDIF
       22CB                           ENDIF
       22CB                           ENDIF
       22CB                           ENDIF
       22CB                           %RESTORE
       22CB 1838                      puly
       22CD 38                        pulx
       22CE 33                        pulb
       22CF 32                        pula
       22D0 39                        rts
                      
       22D1           gp_shi_h        EQU             *
       22D1                           %WRITEPOS       #HIT            ;se marca al carguero como tocado
       22D1 C6B0                      ldab            #HIT
       22D3 37                        pshb                            ;se envía el byte a escribir
       22D4 E60C                      ldab            12,x
       22D6 37                        pshb                            ;se envía letra (fila)
       22D7 E60B                      ldab            11,x
       22D9 37                        pshb                            ;se envía num (columna)
       22DA 1AEE09                    ldy             9,x
       22DD 183C                      pshy                            ;se envía dirección de inicio de matriz
       22DF E608                      ldab            8,x
       22E1 37                        pshb                            ;se envía jugador atacado
       22E2 BD2361                    jsr             writepos
       22E5                           %CLRST          6
       0006           CONT_M          SET     6
       22E5                           IF CONT_M > 0
       22E5 31                        ins
       22E6                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       22E6                           IF CONT_M > 0
       22E6 31                        ins
       22E7                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       22E7                           IF CONT_M > 0
       22E7 31                        ins
       22E8                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       22E8                           IF CONT_M > 0
       22E8 31                        ins
       22E9                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       22E9                           IF CONT_M > 0
       22E9 31                        ins
       22EA                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       22EA                           IF CONT_M > 0
       22EA 31                        ins
       22EB                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       22EB                           ENDIF
       22EB                           ENDIF
       22EB                           ENDIF
       22EB                           ENDIF
       22EB                           ENDIF
       22EB                           ENDIF
       22EB                           ENDIF
       22EB                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       22EB E60C                      ldab            12,x
       22ED 37                        pshb                            ;se envía letra (fila)
       22EE E60B                      ldab            11,x
       22F0 37                        pshb                            ;se envía num (columna)
       22F1 1AEE0E                    ldy             14,x
       22F4 183C                      pshy                            ;se envía inicio de array
       22F6 E608                      ldab            8,x
       22F8 37                        pshb                            ;se envía jugador atacado
       22F9 BD23E9                    jsr             play_wrt
       22FC                           %CLRST          5
       0005           CONT_M          SET     5
       22FC                           IF CONT_M > 0
       22FC 31                        ins
       22FD                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       22FD                           IF CONT_M > 0
       22FD 31                        ins
       22FE                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       22FE                           IF CONT_M > 0
       22FE 31                        ins
       22FF                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       22FF                           IF CONT_M > 0
       22FF 31                        ins
       2300                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2300                           IF CONT_M > 0
       2300 31                        ins
       2301                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2301                           ENDIF
       2301                           ENDIF
       2301                           ENDIF
       2301                           ENDIF
       2301                           ENDIF
       2301                           ENDIF
       2301                           %RESTORE
       2301 1838                      puly
       2303 38                        pulx
       2304 33                        pulb
       2305 32                        pula
       2306 39                        rts
                      
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
                      
       2307           mirror          EQU             *       
                              
       2307                           %BACKUP
       2307 36                        psha
       2308 37                        pshb
       2309 3C                        pshx
       230A 183C                      pshy
       230C 30                        tsx                             ;creo frame pointer
       230D A60B                      ldaa            11,x            ;cargo número de jugador
       230F E608                      ldab            8,x             ;cargo longitud de la copia (del array a copiar)
       2311 3D                        mul                             ;obtengo offset de jugador
       2312 E30C                      addd            12,x            ;sumo offset de jugador
       2314 ED0C                      std             12,x            ;guardo la dirección con offset del array fuente
                      
       2316 A60C                      ldaa            12,x            ;cargo longitud de copia en contador
       2318 2710                      beq             mi_end          ;si es cero, vuelvo desde subrutina
       231A 1AEE09                    ldy             9,x             ;cargo índice a destino de la copia.
       231D EE0C                      ldx             12,x            ;cargo índice a array origen de copia.
                      
       231F           mi_loop         EQU             *
                      
       231F E600                      ldab            0,x             ;cargo byte a copiar
       2321 18E700                    stab            0,y             ;copio byte en destino
       2324 08                        inx                             ;siguiente byte
       2325 1808                      iny                             ;siguiente byte
       2327 4A                        deca                            ;decremento contador
       2328 26F5                      bne             mi_loop
                      
       232A           mi_end          EQU             *
                      
       232A                           %RESTORE
       232A 1838                      puly
       232C 38                        pulx
       232D 33                        pulb
       232E 32                        pula
                              
       232F 39                        rts     
                                      
                                                      
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
                      
       2330           fetchpos        EQU             *
                      
       2330                           %BACKUP
       2330 36                        psha
       2331 37                        pshb
       2332 3C                        pshx
       2333 183C                      pshy
       2335 30                        tsx                             ;creo frame pointer
       2336 A608                      ldaa            8,x             ;cargo numero de jugador
       2338 C624                      ldab            #ARRAY_L        ;cargo longitud del arreglo (MAX 255 bytes)
       233A 3D                        mul                             ;obtengo offset de jugador
       233B E309                      addd            9,x             ;sumo offset de jugador a top of array
       233D ED09                      std             9,x             ;obtengo direccion al arreglo del jugador
                      
       233F A60C                      ldaa            12,x            ;cargo coordenada letra (FILA)
       2341 C60A                      ldab            #NORM           ;cargo valor de normalización de filas
       2343 10                        sba                             ;obtengo numero de fila
       2344 C606                      ldab            #COL            ;cargo longitud de fila
       2346 3D                        mul                             ;obtengo offset de fila
       2347 E309                      addd            9,x             ;sumo offset de fila a la dirección del arreglo del jugador
       2349 ED09                      std             9,x             ;obtengo dirección a la fila que se desea
                      
       234B E60B                      ldab            11,x            ;cargo coordenada (offset) de columna
       234D 5A                        decb                            ;normalizo offset de columna
       234E 4F                        clra                            ;borro AccA para tener el valor de AccB en AccD
       234F E309                      addd            9,x             ;sumo offset de columna a la dirección temporal
       2351 ED09                      std             9,x             ;guardo en stack la dirección del byte a consultar
                      
       2353 1AEE09                    ldy             9,x             ;cargo en IY la dirección del byte que se desea obtener
       2356 18A600                    ldaa            0,y             ;cargo el byte a devolver en AccA
       2359 A70C                      staa            12,x            ;guardo el byte a devolver en stack
       235B                           %RESTORE
       235B 1838                      puly
       235D 38                        pulx
       235E 33                        pulb
       235F 32                        pula
                                      
       2360 39                        rts
                                      
                                      
                      
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
                      
       2361           writepos        EQU             *
                      
       2361                           %BACKUP
       2361 36                        psha
       2362 37                        pshb
       2363 3C                        pshx
       2364 183C                      pshy
       2366 30                        tsx                             ;creo frame pointer
       2367 A608                      ldaa            8,x             ;cargo numero de jugador
       2369 C624                      ldab            #ARRAY_L        ;cargo longitud del arreglo (MAX 255 bytes)
       236B 3D                        mul                             ;obtengo offset de jugador
       236C E309                      addd            9,x             ;sumo offset de jugador a top of array
       236E ED09                      std             9,x             ;obtengo direccion al arreglo del jugador
                      
       2370 A60C                      ldaa            12,x            ;cargo coordenada letra (FILA)
       2372 C60A                      ldab            #NORM           ;cargo valor de normalización de filas
       2374 10                        sba                             ;obtengo numero de fila
       2375 C606                      ldab            #COL            ;cargo longitud de fila
       2377 3D                        mul                             ;obtengo offset de fila
       2378 E309                      addd            9,x             ;sumo offset de fila a la dirección del arreglo del jugador
       237A ED09                      std             9,x             ;obtengo dirección a la fila que se desea
                      
       237C E60B                      ldab            11,x            ;cargo coordenada (offset) de columna
       237E 5A                        decb                            ;normalizo offset de columna
       237F 4F                        clra                            ;borro AccA para tener el valor de AccB en AccD
       2380 E309                      addd            9,x             ;sumo offset de columna a la dirección temporal
       2382 ED09                      std             9,x             ;guardo en stack la dirección del byte a consultar
                      
       2384 1AEE09                    ldy             9,x             ;cargo en IY la dirección donde se desea escribir
       2387 A60D                      ldaa            13,x            ;cargo en AccA el byte que se quiere escribir en el arreglo
       2389 18A700                    staa            0,y             ;escribo el byte sobre el arreglo
       238C                           %RESTORE
       238C 1838                      puly
       238E 38                        pulx
       238F 33                        pulb
       2390 32                        pula
                                      
       2391 39                        rts
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
                      
       2392           play_src        EQU             *               
                                              
       2392                           %BACKUP
       2392 36                        psha
       2393 37                        pshb
       2394 3C                        pshx
       2395 183C                      pshy
       2397 30                        tsx                             ;creo frame pointer
       2398 A608                      ldaa            8,x             ;cargo número de array (jugador)
       239A C613                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       239C 3D                        mul                             ;obtengo offset de jugador
       239D E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       239F 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
                      
       23A1           ps_loop         EQU             *
                      
       23A1 18A600                    ldaa            0,y             ;cargo byte1 a comparar
       23A4 2718                      beq             ps_ntfnd        ;si encontré el terminador, el valor requerido no está en el array
                      
       23A6 A10C                      cmpa            12,x            ;comparo con el byte1 a buscar
       23A8 2706                      beq             ps_next         ;si coinciden, busco el byte 2
       23AA 1808                      iny                             ;me posiciono sobre siguiente byte1 a comparar
       23AC 1808                      iny
       23AE 20F1                      bra             ps_loop         ;vuelvo a comparar
                      
       23B0           ps_next         EQU             *
                      
       23B0 18E601                    ldab            1,y             ;cargo byte a continuación del anterior
       23B3 1808                      iny                             ;me posiciono sobre siguiente byte1 a comparar
       23B5 1808                      iny
       23B7 E10B                      cmpb            11,x            ;comparo con byte2
       23B9 26E6                      bne             ps_loop         ;si no son iguales, sigo buscando
       23BB 0D                        sec                             ;set del carry para informar que se encontró el valor
       23BC 2001                      bra             ps_end
                                              
       23BE           ps_ntfnd        EQU             *
                      
       23BE 0C                        clc                             ;carry en 0 para informar que no se encontró el valor
                      
       23BF           ps_end          EQU             *
                                      
       23BF                           %RESTORE
       23BF 1838                      puly
       23C1 38                        pulx
       23C2 33                        pulb
       23C3 32                        pula
                      
       23C4 39                        rts
                      
                      
                      
                                              
                      
                                              
                      
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
                      
       23C5           play_cnt        EQU             *
                      
       23C5                           %BACKUP
       23C5 36                        psha
       23C6 37                        pshb
       23C7 3C                        pshx
       23C8 183C                      pshy
       23CA 30                        tsx                             ;creo frame pointer
       23CB A608                      ldaa            8,x             ;cargo número de array (jugador)
       23CD C613                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       23CF 3D                        mul                             ;obtengo offset de jugador
       23D0 E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       23D2 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
       23D4 4F                        clra                            ;inicializo contador
                                              
       23D5           pc_loop         EQU             *               
                                              
       23D5 18E600                    ldab            0,y             ;cargo byte a comparar
       23D8 2707                      beq             pc_end          ;si es terminador, terminé de buscar
       23DA 1808                      iny                             ;si no es terminador, me posiciono sobre siguiente dato de 
       23DC 1808                      iny                             ;2 bytes
       23DE 4C                        inca                            ;incremento contador
       23DF 20F4                      bra             pc_loop         ;vuelvo a comparar
                                              
       23E1           pc_end          EQU             *       
                                              
       23E1 A70A                      staa            10,x            ;devuelvo valor por stack
       23E3                           %RESTORE
       23E3 1838                      puly
       23E5 38                        pulx
       23E6 33                        pulb
       23E7 32                        pula
                                      
       23E8 39                        rts
                      
                      
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
                      
       23E9           play_wrt        EQU             *
                      
       23E9                           %BACKUP
       23E9 36                        psha
       23EA 37                        pshb
       23EB 3C                        pshx
       23EC 183C                      pshy
       23EE 30                        tsx                             ;creo frame pointer
       23EF A608                      ldaa            8,x             ;cargo número de array (jugador)
       23F1 C613                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       23F3 3D                        mul                             ;obtengo offset de jugador
       23F4 E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       23F6 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
                      
                      
       23F8           pw_loop         EQU             *
                      
       23F8 18A600                    ldaa            0,y             ;cargo byte del array
       23FB 2704                      beq             pw_end          ;si es cero, entonces escribo los datos
       23FD 1808                      iny                             ;si no es cero, me posiciono sobre la siguiente posición
       23FF 20F7                      bra             pw_loop         ;vuelvo a buscar ceros
                      
       2401           pw_end          EQU             *
                      
       2401 A60C                      ldaa            12,x            ;cargo primer byte a escribir
       2403 18A700                    staa            0,y             ;guardo primer byte en memoria
       2406 1808                      iny                             ;me posiciono en el siguiente byte
       2408 A60B                      ldaa            11,x            ;cargo segundo byte a escribir
       240A 18A700                    staa            0,y             ;guardo segundo byte en memoria
       240D                           %RESTORE
       240D 1838                      puly
       240F 38                        pulx
       2410 33                        pulb
       2411 32                        pula
                      
       2412 39                        rts
                      
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
       2413           clrzone         EQU     *
                      
       2413                           %BACKUP                         ;backup de registros en stack
       2413 36                        psha
       2414 37                        pshb
       2415 3C                        pshx
       2416 183C                      pshy
                      
       2418 30                        tsx
       2419 A608                      ldaa    8,x                     ;cargo contador con longitud de zona a borrar
       241B EE09                      ldx     9,x                     ;cargo puntero zona de memoria a borrar
       241D D600                      ldab    $00                     ;cargo el cero
       241F           clr_loop        EQU     *
       241F E700                      stab    0,x                     ;guardo caracter en posicion del array
       2421 08                        inx                             ;siguiente posición     
       2422 4A                        deca                            ;decremento contador
       2423 26FA                      bne     clr_loop                ;si no es cero, escribo en la posición siguiente.
                      
       2425                           %RESTORE                        :restauro backup en registros
       2425 1838                      puly
       2427 38                        pulx
       2428 33                        pulb
       2429 32                        pula
                      
       242A 39                        rts
                      
                      *---------------------------------------
                      * WhoPlays: subrutina para ver que 
                      *           jugador juega
                      *---------------------------------------
                      
       242B           whoplays        EQU             *
                      
       242B BD24E8    wait_ply        jsr             read_kb
       242E 4D                        tsta
       242F 2BFA                      bmi             wait_ply
       2431 8103                      cmpa            #B_P1
       2433 2708                      beq             play_p1
       2435 8107                      cmpa            #B_P2
       2437 26F2                      bne             wait_ply
       2439 8601                      ldaa            #P2
       243B 2002                      bra             ply_end
                      
       243D 8600      play_p1         ldaa            #P1
       243F 39        ply_end         rts             
                      
                      *---------------------------------------
                      * Input: subrutina que devuelve
                      *        entrada para la batalla naval
                      * AccA: Fila de la posicion jugada
                      * AccB: Columna de la posicion jugada
                      *---------------------------------------
                      
       2440           input           EQU             *
                      
                      * back up de registros
       2440 3C                        pshx
       2441 183C                      pshy
                      
                      * creo variables locales
       2443 36                        psha
       2444 36                        psha
       2445 36                        psha
                      
                      * inicializo framepointer
       2446 1830                      tsy
                      
                      * prendo el led
       2448                           %LED_INV
       2448 B61000                    ldaa            PORTA
       244B 8840                      eora            #SHF_LED
       244D B71000                    staa            PORTA
                      
                      * inicializo el contador
       2450 8603      inp_rest        ldaa            #INPUTS
       2452 18A700                    staa            CONT,Y
                                      
                      * limpio el display
       2455 CE41F9                    ldx             #string
       2458 8604                      ldaa            #4
       245A C620                      ldab            #ESP
       245C BD25B2                    jsr             clrarray
                      
       245F BD25A1                    jsr             clr_dis
                      
       2462 86FF                      ldaa            #$FF
       2464 B741F8                    staa            buffer
                      
                      * lectura de teclado
       2467 BD24E8    wait_kb         jsr             read_kb
       246A 4D                        tsta
       246B 2BFA                      bmi             wait_kb
       246D B141F8                    cmpa            buffer
       2470 27F5                      beq             wait_kb
       2472 B741F8                    staa            buffer
                                      
       2475 810F                      cmpa            #B_reset
       2477 27D7                      beq             inp_rest
                      
       2479 186A00                    dec             CONT,Y
       247C 2727                      beq             chk_fire
       247E 36                        psha
       247F CE41FE                    ldx             #kb_value
       2482 16                        tab
       2483 3A                        abx
       2484 A600                      ldaa            0,X             
       2486 183C                      pshy
       2488 18E600                    ldab            CONT,Y
       248B 183A                      aby
       248D 18A700                    staa            0,Y
       2490 1838                      puly
                      
       2492 33                        pulb
       2493 BD2591                    jsr             to_ascii
       2496 17                        tba
                                      
       2497 CE41F9                    ldx             #string
       249A BD2581                    jsr             sh_value
                      
       249D CE41F9                    ldx             #string
       24A0 BD24D4                    jsr             show
                      
       24A3 20C2                      bra             wait_kb
                      
       24A5 810B      chk_fire        cmpa            #B_fire
       24A7 26A7                      bne             inp_rest
       24A9 18A602                    ldaa            FILA_L,Y
       24AC 810A                      cmpa            #MIN_FIL
       24AE 25A0                      blo             inp_rest
       24B0 810F                      cmpa            #MAX_FIL
       24B2 229C                      bhi             inp_rest
       24B4 18A601                    ldaa            COL_L,Y
       24B7 8101                      cmpa            #MIN_COL
       24B9 2595                      blo             inp_rest
       24BB 8106                      cmpa            #MAX_COL
       24BD 2291                      bhi             inp_rest
                      
                      * apago el led
       24BF                           %LED_INV
       24BF B61000                    ldaa            PORTA
       24C2 8840                      eora            #SHF_LED
       24C4 B71000                    staa            PORTA
                      
                      * libero variables locales
       24C7 32                        pula
       24C8 32                        pula
       24C9 32                        pula
                      
       24CA 18A602    return          ldaa            FILA_L,Y
       24CD 18E601                    ldab            COL_L,Y         
                      
                      * restore de registros
       24D0 1838                      puly
       24D2 38                        pulx
                      
       24D3 39                        rts
                      
                      *------------------------------------
                      * Show: Por IX direccion de string de
                      *       4 caracteres para imprimir
                      *------------------------------------
                      
       24D4           show            EQU             *
                      
                      * back up de registros
       24D4 3C                        pshx
       24D5 37                        pshb
       24D6 36                        psha
                      
                      * inicializo contador de posiciones
       24D7 C604                      ldab            #LENGTH
                                      
                      * loop para enviar a cada posicion
       24D9 5A        sh_loop         decb
       24DA 2B08                      bmi             sh_end          ; si no pase por todas las posiciones
       24DC A600                      ldaa            0,X             ; tomo del string el caracter
       24DE BD259A                    jsr             outchar         ; y lo imprimo en la posicion actual
       24E1 08                        inx
       24E2 20F5                      bra             sh_loop
                      
                      * restore de registros                          
       24E4 32        sh_end          pula
       24E5 33                        pulb
       24E6 38                        pulx
       24E7 39                        rts
                      
                      *----------------------------------------------
                      * read_kb: AccA: tecla presionada
                      *          o -1
                      *----------------------------------------------
                      
       24E8           read_kb         EQU             *
                      
                      * back up de registros
       24E8 3C                        pshx
       24E9 37                        pshb
                      
                      * creo espacio para variables locales
       24EA                           %MALLOC         1               ; variable mascara de filas
       24EA                           IF 1 > 0
       24EA 30                        tsx
       24EB 8F                        xgdx
       24EC 830001                    subd    #1
       24EF 8F                        xgdx
       24F0 35                        txs
       24F1                           ENDIF
       24F1                           %MALLOC         1               ; variable mascara de columnas
       24F1                           IF 1 > 0
       24F1 30                        tsx
       24F2 8F                        xgdx
       24F3 830001                    subd    #1
       24F6 8F                        xgdx
       24F7 35                        txs
       24F8                           ENDIF
       24F8                           %MALLOC         1               ; variable contador de columnas
       24F8                           IF 1 > 0
       24F8 30                        tsx
       24F9 8F                        xgdx
       24FA 830001                    subd    #1
       24FD 8F                        xgdx
       24FE 35                        txs
       24FF                           ENDIF
       24FF                           %MALLOC         1               ; variable contador de tecla
       24FF                           IF 1 > 0
       24FF 30                        tsx
       2500 8F                        xgdx
       2501 830001                    subd    #1
       2504 8F                        xgdx
       2505 35                        txs
       2506                           ENDIF
                      
                      * inicializacion de variables
       2506 30                        tsx                             ; framepointer para acceder a locales
       2507 C604                      ldab            #LENGTH         ; contador de fila
       2509 860E                      ldaa            #FIL0           ; mascara para la fila cero
       250B A703                      staa            MASK_FIL,X      
       250D 8610                      ldaa            #COL0           ; mascara para la columna cero
       250F A702                      staa            MASK_COL,X
       2511 4F                        clra                            ; contador de columna
       2512 A701                      staa            COL_CONT,X
       2514 A700                      staa            KB_PRESS,X      ; variable con numero de tecla
                      
                      * loop para barrer y mirar teclado matricial
       2516 86FF      kb_loop         ldaa            #NONE
       2518 5A                        decb                            
       2519 2B2C                      bmi             kb_end          ; me fijo si recorrio todas las filas
       251B A603                      ldaa            MASK_FIL,X      ; enmascaro la siguiente fila
       251D B71003                    staa            ROWS
       2520 0D                        sec                             ; carry para rotacion de byte
       2521 49                        rola                            
       2522 A703                      staa            MASK_FIL,X      ; cambio variable mascara para la proxima fila
                      
       2524 8604                      ldaa            #LENGTH
       2526 A701                      staa            COL_CONT,X
       2528 8610                      ldaa            #COL0           ; inicializo variable para barrer
       252A A702                      staa            MASK_COL,X      ; la columna
                      
       252C 6A01      kb_col          dec             COL_CONT,X      ; me fijo si barrio todas las columnas
       252E 2BE6                      bmi             kb_loop
       2530 B61005                    ldaa            COLS            ; leo el puerto de columnas
       2533 A402                      anda            MASK_COL,X      ; enmascaro columna
       2535 2707                      beq             kb_hit          ; si es nulo apretaron un boton
       2537 48                        lsla                            ; cambio mascara para proxima columna
       2538 A702                      staa            MASK_COL,X
       253A 6C00                      inc             KB_PRESS,X      ; incremento numero de tecla
       253C 20EE                      bra             kb_col
                      
       253E A600      kb_hit          ldaa            KB_PRESS,X      ; apretaron una tecla, lo comparo
       2540 B141F7                    cmpa            kb_buff         ; con la que fue apretada antes
       2543 2602                      bne             kb_end          ; aviso por led
                      
       2545 86FF      kb_none         ldaa            #NONE           ; en caso de que no apreten nada
                      
       2547 B741F7    kb_end          staa            kb_buff         ; actualizo variable
                                              
                      * elimino variables locales
       254A                           %RELMEM         1
       254A                           IF 1 > 0
       254A 30                        tsx
       254B 8F                        xgdx
       254C C30001                    addd    #1
       254F 8F                        xgdx
       2550 35                        txs
       2551                           ENDIF           
       2551                           %RELMEM         1
       2551                           IF 1 > 0
       2551 30                        tsx
       2552 8F                        xgdx
       2553 C30001                    addd    #1
       2556 8F                        xgdx
       2557 35                        txs
       2558                           ENDIF           
       2558                           %RELMEM         1
       2558                           IF 1 > 0
       2558 30                        tsx
       2559 8F                        xgdx
       255A C30001                    addd    #1
       255D 8F                        xgdx
       255E 35                        txs
       255F                           ENDIF           
       255F                           %RELMEM         1
       255F                           IF 1 > 0
       255F 30                        tsx
       2560 8F                        xgdx
       2561 C30001                    addd    #1
       2564 8F                        xgdx
       2565 35                        txs
       2566                           ENDIF           
                      
                      * restore de registros
       2566 33                        pulb
       2567 38                        pulx
                                      
       2568 39                        rts
                      
                      *----------------------------------------------
                      * init_kb: subrutina para inicializar teclado
                      *----------------------------------------------
                      
       2569           init_kb         EQU             *
                      
                      * back up de registros
       2569 36                        psha
                      
                      * inicializacion de teclado             
       256A 86FF                      ldaa            #NONE
       256C B741F7                    staa            kb_buff
                      
                      * restore de registros
       256F 32                        pula
       2570 39                        rts
                      
                      *--------------------------------
                      * Inicializa display (y teclado)
                      *--------------------------------
                      
       2571           initdis         EQU             *
                              
       2571 36                        psha
                      
       2572 860F                      ldaa            #$0F       ;set for input/output
       2574 B71007                    staa            DDRC       ;C output
       2577 B61002                    ldaa            PIOC       ;get current value
       257A 84FE                      anda            #$FE       ;reset bit 0 to 0
       257C B71002                    staa            PIOC       ;send it out
                              
       257F 32                        pula
       2580 39                        rts
                      
                      *--------------------------------------------------------------
                      * sh_value: toma un string, desplaza los caracteres a izquierda
                      *           y agrega un nuevo caracter
                      *           IX: direccion del string
                      *           AccA: nuevo valor
                      *--------------------------------------------------------------
                      
       2581           sh_value        EQU             *
                      
                      * back up de registros
       2581 3C                        pshx
       2582 37                        pshb
                      
                      * loop para desplazar caracteres
       2583 E601      v_loop          ldab            1,X             ;tomo elemento siguiente en string
       2585 2705                      beq             v_end
       2587 E700                      stab            0,X             ;y lo guardo en posicion actual
       2589 08                        inx
       258A 20F7                      bra             v_loop          
                      
       258C A700      v_end           staa            0,X             ;ingresa nuevo caracter en cadena desplazada
                      
                      * restore de registros
       258E 33                        pulb
       258F 38                        pulx
       2590 39                        rts
                      
                      *----------------------------------------------
                      * toAscii: toma numero de boton del teclado
                      *          y devuelve su correspondiente en
                      *          ascii
                      *          toma  AccB: valor del boton
                      *          devuelve AccB: ascii
                      *----------------------------------------------
                      
       2591           to_ascii        EQU             *
                      
                      * back up de registros
       2591 3C                        pshx
                      
                      * inicializacion de variables
       2592 CE420D                    ldx             #kb_ascii       ; apunto a string con caracteres asignados por tecla
       2595 3A                        abx                             ; me muevo en string con indice y recupero segun tecla presionada
       2596 E600                      ldab            0,X             
                      
                      * restore de registros
       2598 38                        pulx
       2599 39                        rts
                      
                      *--------------------------------
                      *Outchar: ACCA: Carácter a enviar
                      *           ACCB: Posición (0-3)
                      *--------------------------------
       259A           outchar         EQU             *
                              
       259A F71005                    stab            PORTCL          ;set the position 
       259D B71004                    staa            PORTB           ;set the character
                              
       25A0 39                        rts
                      
                      *-------------------------
                      * clr_dis: Limpia display
                      *-------------------------
                      
       25A1           clr_dis         EQU             *
                      
                      * hago back up de registros
       25A1 37                        pshb
       25A2 36                        psha
                      
                      * inicializo el contador de posiciones
       25A3 C604                      ldab            #LENGTH
                      
                      * loop para limpiar cada posicion
       25A5 5A        clrloop         decb
       25A6 2B07                      bmi             clr_end         ; si no pase por todas las posiciones
       25A8 8620                      ldaa            #ESP            ; cargo en esta posicion espacio blanco
       25AA BD259A                    jsr             outchar         ; y lo envio
       25AD 20F6                      bra             clrloop         
                      
                      * restore de registros
       25AF 32        clr_end         pula
       25B0 33                        pulb
       25B1 39                        rts
                      
                      *-------------------------------------------------------
                      * clrarray: subrutina para limpiar un array dado
                      * parametros: IX direccion del array
                      *             AccA cantidad de elementos del array
                      *             AccB caracter a poner
                      *-------------------------------------------------------
                      
       25B2           clrarray        EQU             *
                                      
       25B2 4A        cra_loop        deca                            ;contador con cantidad de elementos a limpiar
       25B3 2705                      beq             cra_end
       25B5 E700                      stab            0,X             ;reemplazar elemento por caracter a poner
       25B7 08                        inx
       25B8 20F8                      bra             cra_loop
                                      
       25BA E700      cra_end         stab            0,X
       25BC 39                        rts
                      
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
                      
                      ***MATRICES DE TABLERO DE CADA JUGADOR***
       5000                           ORG             $5000
       5000 83830383  p_board         FCB             SHI_NP,SHI_NP,WAT_NP,SHI_NP,WAT_NP,SUB_NP
       5004 0343
       5006 03030383                  FCB             WAT_NP,WAT_NP,WAT_NP,SHI_NP,WAT_NP,WAT_NP
       500A 0303
       500C 03030303                  FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       5010 0303
       5012 03030303                  FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,SHI_NP,SHI_NP
       5016 8383
       5018 43030303                  FCB             SUB_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       501C 0303
       501E 03034303                  FCB             WAT_NP,WAT_NP,SUB_NP,WAT_NP,WAT_NP,WAT_NP
       5022 0303
                      
       5024 03830383                  FCB             WAT_NP,SHI_NP,WAT_NP,SHI_NP,SHI_NP,WAT_NP
       5028 8303
       502A 03830303                  FCB             WAT_NP,SHI_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       502E 0303
       5030 03030383                  FCB             WAT_NP,WAT_NP,WAT_NP,SHI_NP,SHI_NP,WAT_NP
       5034 8303
       5036 03030303                  FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       503A 0303
       503C 03034303                  FCB             WAT_NP,WAT_NP,SUB_NP,WAT_NP,WAT_NP,WAT_NP
       5040 0303
       5042 03430343                  FCB             WAT_NP,SUB_NP,WAT_NP,SUB_NP,WAT_NP,WAT_NP
       5046 0303
                      
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
     ytes:   1612         # main_bn #
     RC:     1371         ###########
 