################################################################################
#                                                                              #
#    Micro Series 6801 Assembler V2.00/DOS              27/Nov/17  01:50:53    #
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
       2003 BD2597                    jsr             initdis         ;inicialización de display 7 seg
       2006 BD258F                    jsr             init_kb         ;inicialización de teclado
       2009 CE421C                    ldx             #hitlist
       200C 3C                        pshx                            ;se envía inicio de arreglo de barcos a poner en 0
       200D 8626                      ldaa            #(HITSIZE*2)
       200F 36                        psha                            ;se envía el tamaño del arreglo a limpiar
       2010 BD2439                    jsr             clrzone
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
       2016 8601                      ldaa            #P2             ;se inicia atacando al jugador 2
       2018 B741F4                    staa            player          ;se inicia con el jugador 1 (atacando al 2)
                      
       201B           main            EQU             *
       201B BD2451                    jsr             whoplays        ;se recibirá en AccA el jugador a atacar a continuación
       201E B741F4                    staa            player          ;se habilita la siguiente jugada
                      
       2021 CE5000                    ldx             #p_board
       2024 3C                        pshx                            ;se envía el inicio de las matrices de tableros
       2025 B641F4                    ldaa            player
       2028 36                        psha                            ;se envía el jugador atacado
       2029 CE5F00                    ldx             #display
       202C 3C                        pshx                            ;se envía la matriz de display (destino a copiar)
       202D 8624                      ldaa            #ARRAY_L
       202F 36                        psha                            ;se envía el tamaño de la matriz tablero
       2030 BD232D                    jsr             mirror          ;se envía el tablero del jugador atacado al display
       2033                           %CLRST          6
       0006           CONT_M          SET     6
       2033                           IF CONT_M > 0
       2033 31                        ins
       2034                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2034                           IF CONT_M > 0
       2034 31                        ins
       2035                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2035                           IF CONT_M > 0
       2035 31                        ins
       2036                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2036                           IF CONT_M > 0
       2036 31                        ins
       2037                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2037                           IF CONT_M > 0
       2037 31                        ins
       2038                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2038                           IF CONT_M > 0
       2038 31                        ins
       2039                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2039                           ENDIF
       2039                           ENDIF
       2039                           ENDIF
       2039                           ENDIF
       2039                           ENDIF
       2039                           ENDIF
       2039                           ENDIF
                      
       2039 BD2466                    jsr             input           ;se llama a la subrutina de interfaz de usuario
       203C B741F5                    staa            letra_p         ;se recibe en AccA la letra jugada (fila) y en AccB el número (columna)
       203F F741F6                    stab            num_p           ;se guarda la jugada
                      
       2042 B641F5                    ldaa            letra_p
       2045 36                        psha                            ;se envía la letra jugada
       2046 F641F6                    ldab            num_p
       2049 37                        pshb                            ;se envía el número jugado
       204A CE421C                    ldx             #hitlist
       204D 3C                        pshx                            ;se envía la posición del primer arreglo de barcos tocados
       204E B641F4                    ldaa            player
       2051 36                        psha                            ;se envía el jugador a atacar
       2052 BD23B8                    jsr             play_src        ;se recibirá el carry en 1 si la posición corresponde a un barco ya atacado
       2055                           %CLRST          5               ;se limpia el stack
       0005           CONT_M          SET     5
       2055                           IF CONT_M > 0
       2055 31                        ins
       2056                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2056                           IF CONT_M > 0
       2056 31                        ins
       2057                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2057                           IF CONT_M > 0
       2057 31                        ins
       2058                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2058                           IF CONT_M > 0
       2058 31                        ins
       2059                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2059                           IF CONT_M > 0
       2059 31                        ins
       205A                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       205A                           ENDIF
       205A                           ENDIF
       205A                           ENDIF
       205A                           ENDIF
       205A                           ENDIF
       205A                           ENDIF
       205A 25BF                      bcs             main            ;si la posición ya fue jugada, se pierde el turno y pasa al siguiente
                                      
       205C B641F5                    ldaa            letra_p
       205F 36                        psha                            ;se envía le letra jugada
       2060 F641F6                    ldab            num_p
       2063 37                        pshb                            ;se envía el número jugado
       2064 CE5000                    ldx             #p_board
       2067 3C                        pshx                            ;se envía la dirección de inicio de los arreglos de tablero
       2068 F641F4                    ldab            player
       206B 37                        pshb                            ;se envía el jugador a atacar
       206C BD2356                    jsr             fetchpos
       206F                           %CLRST          4               ;se limpia el stack
       0004           CONT_M          SET     4
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
       2073 32                        pula                            ;se toma del stack el dato del byte jugado
                      
       2074 CE421C                    ldx             #hitlist
       2077 3C                        pshx                            ;se envía inicio del arreglo de barcos atacados
       2078 36                        psha                            ;se envía el byte a jugado
       2079 B641F5                    ldaa            letra_p
       207C 36                        psha                            ;se envía le letra jugada
       207D F641F6                    ldab            num_p
       2080 37                        pshb                            ;se envía el número jugado
       2081 CE5000                    ldx             #p_board
       2084 3C                        pshx                            ;se envía la dirección de inicio de los arreglos de tablero
       2085 F641F4                    ldab            player
       2088 37                        pshb                            ;se envía el jugador a atacar
       2089 BD20C5                    jsr             gameplay
       208C                           %CLRST          8               ;se limpia el stack
       0008           CONT_M          SET     8
       208C                           IF CONT_M > 0
       208C 31                        ins
       208D                           %CLRST  CONT_M-1
       0007           CONT_M          SET     CONT_M-1
       208D                           IF CONT_M > 0
       208D 31                        ins
       208E                           %CLRST  CONT_M-1
       0006           CONT_M          SET     CONT_M-1
       208E                           IF CONT_M > 0
       208E 31                        ins
       208F                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       208F                           IF CONT_M > 0
       208F 31                        ins
       2090                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2090                           IF CONT_M > 0
       2090 31                        ins
       2091                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2091                           IF CONT_M > 0
       2091 31                        ins
       2092                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2092                           IF CONT_M > 0
       2092 31                        ins
       2093                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2093                           IF CONT_M > 0
       2093 31                        ins
       2094                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2094                           ENDIF
       2094                           ENDIF
       2094                           ENDIF
       2094                           ENDIF
       2094                           ENDIF
       2094                           ENDIF
       2094                           ENDIF
       2094                           ENDIF
       2094                           ENDIF
                      
       2094 CE5000                    ldx             #p_board
       2097 3C                        pshx                            ;se envía el inicio de las matrices de tableros
       2098 B641F4                    ldaa            player
       209B 36                        psha                            ;se envía el jugador atacado
       209C CE5F00                    ldx             #display
       209F 3C                        pshx                            ;se envía la matriz de display (destino a copiar)
       20A0 8624                      ldaa            #ARRAY_L
       20A2 36                        psha                            ;se envía el tamaño de la matriz tablero
       20A3 BD232D                    jsr             mirror          ;se envía el tablero del jugador atacado al display
       20A6                           %CLRST          6
       0006           CONT_M          SET     6
       20A6                           IF CONT_M > 0
       20A6 31                        ins
       20A7                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       20A7                           IF CONT_M > 0
       20A7 31                        ins
       20A8                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       20A8                           IF CONT_M > 0
       20A8 31                        ins
       20A9                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       20A9                           IF CONT_M > 0
       20A9 31                        ins
       20AA                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       20AA                           IF CONT_M > 0
       20AA 31                        ins
       20AB                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       20AB                           IF CONT_M > 0
       20AB 31                        ins
       20AC                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       20AC                           ENDIF
       20AC                           ENDIF
       20AC                           ENDIF
       20AC                           ENDIF
       20AC                           ENDIF
       20AC                           ENDIF
       20AC                           ENDIF
                                      
       20AC CE421C                    ldx             #hitlist
       20AF 3C                        pshx                            ;se envía la posición del primer arreglo de barcos tocados
       20B0 B641F4                    ldaa            player
       20B3 36                        psha                            ;se envía el jugador a atacar
       20B4 BD23EB                    jsr             play_cnt
       20B7                           %CLRST          2
       0002           CONT_M          SET     2
       20B7                           IF CONT_M > 0
       20B7 31                        ins
       20B8                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       20B8                           IF CONT_M > 0
       20B8 31                        ins
       20B9                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       20B9                           ENDIF
       20B9                           ENDIF
       20B9                           ENDIF
       20B9 32                        pula                            ;se recibe la cantidad de bytes contados
       20BA 8109                      cmpa            #9              
       20BC 2703                      beq             end_game        ;si se llegó al máximo de barcos, se termina el juego
                      
       20BE 7E201B                    jmp             main
                      
       20C1           end_game        EQU             *
                      *               ldaa            player
                      *               psha
                      *               jsr             winner
       20C1 20FE                      bra             end_game
                      
       20C3           end_loop        EQU             *
       20C3 20FE                      bra             end_loop
                      
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
       20C5           gameplay        EQU             *
       20C5                           %BACKUP
       20C5 36                        psha
       20C6 37                        pshb
       20C7 3C                        pshx
       20C8 183C                      pshy
                                      
       20CA 30                        tsx                             ;se usa el IX como frame pointer
       20CB A60D                      ldaa            13,x            ;se carga en AccA el byte a analizar
       20CD 84C0                      anda            #11000000b      ;se enmascan los bits 0 a 5, para analizar los más significativos
                      
       20CF           gp_water        EQU             *
       20CF 2620                      bne             gp_sub          ;se estudia si era agua
       20D1                           %WRITEPOS       #WATER          ;se cambia el color en el tablero
       20D1 C608                      ldab            #WATER
       20D3 37                        pshb                            ;se envía el byte a escribir
       20D4 E60C                      ldab            12,x
       20D6 37                        pshb                            ;se envía letra (fila)
       20D7 E60B                      ldab            11,x
       20D9 37                        pshb                            ;se envía num (columna)
       20DA 1AEE09                    ldy             9,x
       20DD 183C                      pshy                            ;se envía dirección de inicio de matriz
       20DF E608                      ldab            8,x
       20E1 37                        pshb                            ;se envía jugador atacado
       20E2 BD2387                    jsr             writepos
       20E5                           %CLRST          6
       0006           CONT_M          SET     6
       20E5                           IF CONT_M > 0
       20E5 31                        ins
       20E6                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       20E6                           IF CONT_M > 0
       20E6 31                        ins
       20E7                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       20E7                           IF CONT_M > 0
       20E7 31                        ins
       20E8                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       20E8                           IF CONT_M > 0
       20E8 31                        ins
       20E9                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       20E9                           IF CONT_M > 0
       20E9 31                        ins
       20EA                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       20EA                           IF CONT_M > 0
       20EA 31                        ins
       20EB                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       20EB                           ENDIF
       20EB                           ENDIF
       20EB                           ENDIF
       20EB                           ENDIF
       20EB                           ENDIF
       20EB                           ENDIF
       20EB                           ENDIF
       20EB                           %RESTORE
       20EB 1838                      puly
       20ED 38                        pulx
       20EE 33                        pulb
       20EF 32                        pula
       20F0 39                        rts
                      
       20F1           gp_sub          EQU             *
       20F1 8140                      cmpa            #01000000b      ;se estudia si era un submarino
       20F3 2636                      bne             gp_shi_r
       20F5                           %WRITEPOS       #SUNK_SUB       ;se cambia el color en el tablero
       20F5 C650                      ldab            #SUNK_SUB
       20F7 37                        pshb                            ;se envía el byte a escribir
       20F8 E60C                      ldab            12,x
       20FA 37                        pshb                            ;se envía letra (fila)
       20FB E60B                      ldab            11,x
       20FD 37                        pshb                            ;se envía num (columna)
       20FE 1AEE09                    ldy             9,x
       2101 183C                      pshy                            ;se envía dirección de inicio de matriz
       2103 E608                      ldab            8,x
       2105 37                        pshb                            ;se envía jugador atacado
       2106 BD2387                    jsr             writepos
       2109                           %CLRST          6
       0006           CONT_M          SET     6
       2109                           IF CONT_M > 0
       2109 31                        ins
       210A                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       210A                           IF CONT_M > 0
       210A 31                        ins
       210B                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       210B                           IF CONT_M > 0
       210B 31                        ins
       210C                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       210C                           IF CONT_M > 0
       210C 31                        ins
       210D                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       210D                           IF CONT_M > 0
       210D 31                        ins
       210E                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       210E                           IF CONT_M > 0
       210E 31                        ins
       210F                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       210F                           ENDIF
       210F                           ENDIF
       210F                           ENDIF
       210F                           ENDIF
       210F                           ENDIF
       210F                           ENDIF
       210F                           ENDIF
       210F                           %PLAY_WRT                       ;se agraga la posición al array de barcos atacados
       210F E60C                      ldab            12,x
       2111 37                        pshb                            ;se envía letra (fila)
       2112 E60B                      ldab            11,x
       2114 37                        pshb                            ;se envía num (columna)
       2115 1AEE0E                    ldy             14,x
       2118 183C                      pshy                            ;se envía inicio de array
       211A E608                      ldab            8,x
       211C 37                        pshb                            ;se envía jugador atacado
       211D BD240F                    jsr             play_wrt
       2120                           %CLRST          5
       0005           CONT_M          SET     5
       2120                           IF CONT_M > 0
       2120 31                        ins
       2121                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2121                           IF CONT_M > 0
       2121 31                        ins
       2122                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2122                           IF CONT_M > 0
       2122 31                        ins
       2123                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2123                           IF CONT_M > 0
       2123 31                        ins
       2124                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2124                           IF CONT_M > 0
       2124 31                        ins
       2125                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2125                           ENDIF
       2125                           ENDIF
       2125                           ENDIF
       2125                           ENDIF
       2125                           ENDIF
       2125                           ENDIF
       2125                           %RESTORE
       2125 1838                      puly
       2127 38                        pulx
       2128 33                        pulb
       2129 32                        pula
       212A 39                        rts
                      
       212B           gp_shi_r        EQU             *               ;si no era ninguno de los anteriores, es un carguero
       212B E60B                      ldab            11,x
       212D C106                      cmpb            #MAX_C
       212F 276D                      beq             gp_shi_l        ;si está en el borde derecho, se continúa
       2131 E60C                      ldab            12,x
       2133 37                        pshb                            ;se envía la letra (fila)
       2134 E60B                      ldab            11,x
       2136 5C                        incb                            ;se avanza a la derecha
       2137 37                        pshb                            ;se envía el num (columna)
       2138 1AEE0E                    ldy             14,x
       213B 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       213D E608                      ldab            8,x
       213F 37                        pshb                            ;se envía el jugador atacado
       2140 BD23B8                    jsr             play_src        ;se busca la posición de la derecha
       2143                           %CLRST          5
       0005           CONT_M          SET     5
       2143                           IF CONT_M > 0
       2143 31                        ins
       2144                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2144                           IF CONT_M > 0
       2144 31                        ins
       2145                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2145                           IF CONT_M > 0
       2145 31                        ins
       2146                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2146                           IF CONT_M > 0
       2146 31                        ins
       2147                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2147                           IF CONT_M > 0
       2147 31                        ins
       2148                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2148                           ENDIF
       2148                           ENDIF
       2148                           ENDIF
       2148                           ENDIF
       2148                           ENDIF
       2148                           ENDIF
       2148 2454                      bcc             gp_shi_l        ;si no había barco atacado en la derecha, se busca la izquierda
       214A                           %WRITEPOS       #SUNK_SHI       
       214A C690                      ldab            #SUNK_SHI
       214C 37                        pshb                            ;se envía el byte a escribir
       214D E60C                      ldab            12,x
       214F 37                        pshb                            ;se envía letra (fila)
       2150 E60B                      ldab            11,x
       2152 37                        pshb                            ;se envía num (columna)
       2153 1AEE09                    ldy             9,x
       2156 183C                      pshy                            ;se envía dirección de inicio de matriz
       2158 E608                      ldab            8,x
       215A 37                        pshb                            ;se envía jugador atacado
       215B BD2387                    jsr             writepos
       215E                           %CLRST          6
       0006           CONT_M          SET     6
       215E                           IF CONT_M > 0
       215E 31                        ins
       215F                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       215F                           IF CONT_M > 0
       215F 31                        ins
       2160                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2160                           IF CONT_M > 0
       2160 31                        ins
       2161                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2161                           IF CONT_M > 0
       2161 31                        ins
       2162                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2162                           IF CONT_M > 0
       2162 31                        ins
       2163                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2163                           IF CONT_M > 0
       2163 31                        ins
       2164                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2164                           ENDIF
       2164                           ENDIF
       2164                           ENDIF
       2164                           ENDIF
       2164                           ENDIF
       2164                           ENDIF
       2164                           ENDIF
       2164 6C0B                      inc             11,x
       2166                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       2166 C690                      ldab            #SUNK_SHI
       2168 37                        pshb                            ;se envía el byte a escribir
       2169 E60C                      ldab            12,x
       216B 37                        pshb                            ;se envía letra (fila)
       216C E60B                      ldab            11,x
       216E 37                        pshb                            ;se envía num (columna)
       216F 1AEE09                    ldy             9,x
       2172 183C                      pshy                            ;se envía dirección de inicio de matriz
       2174 E608                      ldab            8,x
       2176 37                        pshb                            ;se envía jugador atacado
       2177 BD2387                    jsr             writepos
       217A                           %CLRST          6
       0006           CONT_M          SET     6
       217A                           IF CONT_M > 0
       217A 31                        ins
       217B                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       217B                           IF CONT_M > 0
       217B 31                        ins
       217C                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       217C                           IF CONT_M > 0
       217C 31                        ins
       217D                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       217D                           IF CONT_M > 0
       217D 31                        ins
       217E                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       217E                           IF CONT_M > 0
       217E 31                        ins
       217F                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       217F                           IF CONT_M > 0
       217F 31                        ins
       2180                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2180                           ENDIF
       2180                           ENDIF
       2180                           ENDIF
       2180                           ENDIF
       2180                           ENDIF
       2180                           ENDIF
       2180                           ENDIF
       2180 6A0B                      dec             11,x            ;se recupera el dato
       2182                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       2182 E60C                      ldab            12,x
       2184 37                        pshb                            ;se envía letra (fila)
       2185 E60B                      ldab            11,x
       2187 37                        pshb                            ;se envía num (columna)
       2188 1AEE0E                    ldy             14,x
       218B 183C                      pshy                            ;se envía inicio de array
       218D E608                      ldab            8,x
       218F 37                        pshb                            ;se envía jugador atacado
       2190 BD240F                    jsr             play_wrt
       2193                           %CLRST          5
       0005           CONT_M          SET     5
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
       2198                           %RESTORE
       2198 1838                      puly
       219A 38                        pulx
       219B 33                        pulb
       219C 32                        pula
       219D 39                        rts
                      
       219E           gp_shi_l        EQU             *
       219E E60B                      ldab            11,x
       21A0 C101                      cmpb            #MIN_C
       21A2 276D                      beq             gp_shi_u        ;si está en el borde izquierdo, se continúa
       21A4 E60C                      ldab            12,x
       21A6 37                        pshb                            ;se envía la letra (fila)
       21A7 E60B                      ldab            11,x
       21A9 5A                        decb                            ;se retrocede a la izquierda
       21AA 37                        pshb                            ;se envía el número (columna)
       21AB 1AEE0E                    ldy             14,x
       21AE 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       21B0 E608                      ldab            8,x
       21B2 37                        pshb                            ;se envía el jugador atacado
       21B3 BD23B8                    jsr             play_src        ;se busca la posición de la izquierda
       21B6                           %CLRST          5
       0005           CONT_M          SET     5
       21B6                           IF CONT_M > 0
       21B6 31                        ins
       21B7                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       21B7                           IF CONT_M > 0
       21B7 31                        ins
       21B8                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       21B8                           IF CONT_M > 0
       21B8 31                        ins
       21B9                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       21B9                           IF CONT_M > 0
       21B9 31                        ins
       21BA                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       21BA                           IF CONT_M > 0
       21BA 31                        ins
       21BB                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       21BB                           ENDIF
       21BB                           ENDIF
       21BB                           ENDIF
       21BB                           ENDIF
       21BB                           ENDIF
       21BB                           ENDIF
       21BB 2454                      bcc             gp_shi_u        ;si no había barco atacado a la izquierda, se busca arriba
       21BD                           %WRITEPOS       #SUNK_SHI       
       21BD C690                      ldab            #SUNK_SHI
       21BF 37                        pshb                            ;se envía el byte a escribir
       21C0 E60C                      ldab            12,x
       21C2 37                        pshb                            ;se envía letra (fila)
       21C3 E60B                      ldab            11,x
       21C5 37                        pshb                            ;se envía num (columna)
       21C6 1AEE09                    ldy             9,x
       21C9 183C                      pshy                            ;se envía dirección de inicio de matriz
       21CB E608                      ldab            8,x
       21CD 37                        pshb                            ;se envía jugador atacado
       21CE BD2387                    jsr             writepos
       21D1                           %CLRST          6
       0006           CONT_M          SET     6
       21D1                           IF CONT_M > 0
       21D1 31                        ins
       21D2                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       21D2                           IF CONT_M > 0
       21D2 31                        ins
       21D3                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       21D3                           IF CONT_M > 0
       21D3 31                        ins
       21D4                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       21D4                           IF CONT_M > 0
       21D4 31                        ins
       21D5                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       21D5                           IF CONT_M > 0
       21D5 31                        ins
       21D6                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       21D6                           IF CONT_M > 0
       21D6 31                        ins
       21D7                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       21D7                           ENDIF
       21D7                           ENDIF
       21D7                           ENDIF
       21D7                           ENDIF
       21D7                           ENDIF
       21D7                           ENDIF
       21D7                           ENDIF
       21D7 6A0B                      dec             11,x
       21D9                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       21D9 C690                      ldab            #SUNK_SHI
       21DB 37                        pshb                            ;se envía el byte a escribir
       21DC E60C                      ldab            12,x
       21DE 37                        pshb                            ;se envía letra (fila)
       21DF E60B                      ldab            11,x
       21E1 37                        pshb                            ;se envía num (columna)
       21E2 1AEE09                    ldy             9,x
       21E5 183C                      pshy                            ;se envía dirección de inicio de matriz
       21E7 E608                      ldab            8,x
       21E9 37                        pshb                            ;se envía jugador atacado
       21EA BD2387                    jsr             writepos
       21ED                           %CLRST          6
       0006           CONT_M          SET     6
       21ED                           IF CONT_M > 0
       21ED 31                        ins
       21EE                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       21EE                           IF CONT_M > 0
       21EE 31                        ins
       21EF                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       21EF                           IF CONT_M > 0
       21EF 31                        ins
       21F0                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       21F0                           IF CONT_M > 0
       21F0 31                        ins
       21F1                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       21F1                           IF CONT_M > 0
       21F1 31                        ins
       21F2                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       21F2                           IF CONT_M > 0
       21F2 31                        ins
       21F3                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       21F3                           ENDIF
       21F3                           ENDIF
       21F3                           ENDIF
       21F3                           ENDIF
       21F3                           ENDIF
       21F3                           ENDIF
       21F3                           ENDIF
       21F3 6C0B                      inc             11,x            ;se recupera el dato
       21F5                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       21F5 E60C                      ldab            12,x
       21F7 37                        pshb                            ;se envía letra (fila)
       21F8 E60B                      ldab            11,x
       21FA 37                        pshb                            ;se envía num (columna)
       21FB 1AEE0E                    ldy             14,x
       21FE 183C                      pshy                            ;se envía inicio de array
       2200 E608                      ldab            8,x
       2202 37                        pshb                            ;se envía jugador atacado
       2203 BD240F                    jsr             play_wrt
       2206                           %CLRST          5
       0005           CONT_M          SET     5
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
       220B                           %RESTORE
       220B 1838                      puly
       220D 38                        pulx
       220E 33                        pulb
       220F 32                        pula
       2210 39                        rts
                      
       2211           gp_shi_u        EQU             *
       2211 E60C                      ldab            12,x
       2213 C10A                      cmpb            #MIN_F
       2215 276D                      beq             gp_shi_d        ;si está en el borde superior, se continúa
       2217 E60C                      ldab            12,x
       2219 5A                        decb                            ;se va a la posición de arriba
       221A 37                        pshb                            ;se envía la letra (fila)
       221B E60B                      ldab            11,x
       221D 37                        pshb                            ;se envía el num (columna)
       221E 1AEE0E                    ldy             14,x
       2221 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       2223 E608                      ldab            8,x
       2225 37                        pshb                            ;se envía el jugador atacado
       2226 BD23B8                    jsr             play_src        ;se busca la posición de arriba
       2229                           %CLRST          5
       0005           CONT_M          SET     5
       2229                           IF CONT_M > 0
       2229 31                        ins
       222A                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       222A                           IF CONT_M > 0
       222A 31                        ins
       222B                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       222B                           IF CONT_M > 0
       222B 31                        ins
       222C                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       222C                           IF CONT_M > 0
       222C 31                        ins
       222D                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       222D                           IF CONT_M > 0
       222D 31                        ins
       222E                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       222E                           ENDIF
       222E                           ENDIF
       222E                           ENDIF
       222E                           ENDIF
       222E                           ENDIF
       222E                           ENDIF
       222E 2454                      bcc             gp_shi_d        ;si no había barco atacado arriba, se busca abajo
       2230                           %WRITEPOS       #SUNK_SHI       
       2230 C690                      ldab            #SUNK_SHI
       2232 37                        pshb                            ;se envía el byte a escribir
       2233 E60C                      ldab            12,x
       2235 37                        pshb                            ;se envía letra (fila)
       2236 E60B                      ldab            11,x
       2238 37                        pshb                            ;se envía num (columna)
       2239 1AEE09                    ldy             9,x
       223C 183C                      pshy                            ;se envía dirección de inicio de matriz
       223E E608                      ldab            8,x
       2240 37                        pshb                            ;se envía jugador atacado
       2241 BD2387                    jsr             writepos
       2244                           %CLRST          6
       0006           CONT_M          SET     6
       2244                           IF CONT_M > 0
       2244 31                        ins
       2245                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2245                           IF CONT_M > 0
       2245 31                        ins
       2246                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2246                           IF CONT_M > 0
       2246 31                        ins
       2247                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2247                           IF CONT_M > 0
       2247 31                        ins
       2248                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2248                           IF CONT_M > 0
       2248 31                        ins
       2249                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2249                           IF CONT_M > 0
       2249 31                        ins
       224A                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       224A                           ENDIF
       224A                           ENDIF
       224A                           ENDIF
       224A                           ENDIF
       224A                           ENDIF
       224A                           ENDIF
       224A                           ENDIF
       224A 6A0C                      dec             12,x
       224C                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       224C C690                      ldab            #SUNK_SHI
       224E 37                        pshb                            ;se envía el byte a escribir
       224F E60C                      ldab            12,x
       2251 37                        pshb                            ;se envía letra (fila)
       2252 E60B                      ldab            11,x
       2254 37                        pshb                            ;se envía num (columna)
       2255 1AEE09                    ldy             9,x
       2258 183C                      pshy                            ;se envía dirección de inicio de matriz
       225A E608                      ldab            8,x
       225C 37                        pshb                            ;se envía jugador atacado
       225D BD2387                    jsr             writepos
       2260                           %CLRST          6
       0006           CONT_M          SET     6
       2260                           IF CONT_M > 0
       2260 31                        ins
       2261                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2261                           IF CONT_M > 0
       2261 31                        ins
       2262                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2262                           IF CONT_M > 0
       2262 31                        ins
       2263                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2263                           IF CONT_M > 0
       2263 31                        ins
       2264                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2264                           IF CONT_M > 0
       2264 31                        ins
       2265                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2265                           IF CONT_M > 0
       2265 31                        ins
       2266                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2266                           ENDIF
       2266                           ENDIF
       2266                           ENDIF
       2266                           ENDIF
       2266                           ENDIF
       2266                           ENDIF
       2266                           ENDIF
       2266 6C0C                      inc             12,x            ;se recupera el dato
       2268                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       2268 E60C                      ldab            12,x
       226A 37                        pshb                            ;se envía letra (fila)
       226B E60B                      ldab            11,x
       226D 37                        pshb                            ;se envía num (columna)
       226E 1AEE0E                    ldy             14,x
       2271 183C                      pshy                            ;se envía inicio de array
       2273 E608                      ldab            8,x
       2275 37                        pshb                            ;se envía jugador atacado
       2276 BD240F                    jsr             play_wrt
       2279                           %CLRST          5
       0005           CONT_M          SET     5
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
       227E                           %RESTORE
       227E 1838                      puly
       2280 38                        pulx
       2281 33                        pulb
       2282 32                        pula
       2283 39                        rts
                      
       2284           gp_shi_d        EQU             *
       2284 E60C                      ldab            12,x
       2286 C10F                      cmpb            #MAX_F
       2288 276D                      beq             gp_shi_h        ;si está en el borde inferior, se continúa
       228A E60C                      ldab            12,x
       228C 5C                        incb                            ;se va a la posición de abajo
       228D 37                        pshb                            ;se envía la letra (fila)
       228E E60B                      ldab            11,x
       2290 37                        pshb                            ;se envía el num (columna)
       2291 1AEE0E                    ldy             14,x
       2294 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       2296 E608                      ldab            8,x
       2298 37                        pshb                            ;se envía el jugador atacado
       2299 BD23B8                    jsr             play_src        ;se busca la posición de abajo
       229C                           %CLRST          5
       0005           CONT_M          SET     5
       229C                           IF CONT_M > 0
       229C 31                        ins
       229D                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       229D                           IF CONT_M > 0
       229D 31                        ins
       229E                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       229E                           IF CONT_M > 0
       229E 31                        ins
       229F                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       229F                           IF CONT_M > 0
       229F 31                        ins
       22A0                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       22A0                           IF CONT_M > 0
       22A0 31                        ins
       22A1                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       22A1                           ENDIF
       22A1                           ENDIF
       22A1                           ENDIF
       22A1                           ENDIF
       22A1                           ENDIF
       22A1                           ENDIF
       22A1 2454                      bcc             gp_shi_h        ;si no había barco atacado abajo, el carguero está tocado
       22A3                           %WRITEPOS       #SUNK_SHI       
       22A3 C690                      ldab            #SUNK_SHI
       22A5 37                        pshb                            ;se envía el byte a escribir
       22A6 E60C                      ldab            12,x
       22A8 37                        pshb                            ;se envía letra (fila)
       22A9 E60B                      ldab            11,x
       22AB 37                        pshb                            ;se envía num (columna)
       22AC 1AEE09                    ldy             9,x
       22AF 183C                      pshy                            ;se envía dirección de inicio de matriz
       22B1 E608                      ldab            8,x
       22B3 37                        pshb                            ;se envía jugador atacado
       22B4 BD2387                    jsr             writepos
       22B7                           %CLRST          6
       0006           CONT_M          SET     6
       22B7                           IF CONT_M > 0
       22B7 31                        ins
       22B8                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       22B8                           IF CONT_M > 0
       22B8 31                        ins
       22B9                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       22B9                           IF CONT_M > 0
       22B9 31                        ins
       22BA                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       22BA                           IF CONT_M > 0
       22BA 31                        ins
       22BB                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       22BB                           IF CONT_M > 0
       22BB 31                        ins
       22BC                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       22BC                           IF CONT_M > 0
       22BC 31                        ins
       22BD                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       22BD                           ENDIF
       22BD                           ENDIF
       22BD                           ENDIF
       22BD                           ENDIF
       22BD                           ENDIF
       22BD                           ENDIF
       22BD                           ENDIF
       22BD 6C0C                      inc             12,x
       22BF                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       22BF C690                      ldab            #SUNK_SHI
       22C1 37                        pshb                            ;se envía el byte a escribir
       22C2 E60C                      ldab            12,x
       22C4 37                        pshb                            ;se envía letra (fila)
       22C5 E60B                      ldab            11,x
       22C7 37                        pshb                            ;se envía num (columna)
       22C8 1AEE09                    ldy             9,x
       22CB 183C                      pshy                            ;se envía dirección de inicio de matriz
       22CD E608                      ldab            8,x
       22CF 37                        pshb                            ;se envía jugador atacado
       22D0 BD2387                    jsr             writepos
       22D3                           %CLRST          6
       0006           CONT_M          SET     6
       22D3                           IF CONT_M > 0
       22D3 31                        ins
       22D4                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       22D4                           IF CONT_M > 0
       22D4 31                        ins
       22D5                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       22D5                           IF CONT_M > 0
       22D5 31                        ins
       22D6                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       22D6                           IF CONT_M > 0
       22D6 31                        ins
       22D7                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       22D7                           IF CONT_M > 0
       22D7 31                        ins
       22D8                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       22D8                           IF CONT_M > 0
       22D8 31                        ins
       22D9                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       22D9                           ENDIF
       22D9                           ENDIF
       22D9                           ENDIF
       22D9                           ENDIF
       22D9                           ENDIF
       22D9                           ENDIF
       22D9                           ENDIF
       22D9 6A0C                      dec             12,x            ;se recupera el dato
       22DB                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       22DB E60C                      ldab            12,x
       22DD 37                        pshb                            ;se envía letra (fila)
       22DE E60B                      ldab            11,x
       22E0 37                        pshb                            ;se envía num (columna)
       22E1 1AEE0E                    ldy             14,x
       22E4 183C                      pshy                            ;se envía inicio de array
       22E6 E608                      ldab            8,x
       22E8 37                        pshb                            ;se envía jugador atacado
       22E9 BD240F                    jsr             play_wrt
       22EC                           %CLRST          5
       0005           CONT_M          SET     5
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
       22F1                           %RESTORE
       22F1 1838                      puly
       22F3 38                        pulx
       22F4 33                        pulb
       22F5 32                        pula
       22F6 39                        rts
                      
       22F7           gp_shi_h        EQU             *
       22F7                           %WRITEPOS       #HIT            ;se marca al carguero como tocado
       22F7 C6B0                      ldab            #HIT
       22F9 37                        pshb                            ;se envía el byte a escribir
       22FA E60C                      ldab            12,x
       22FC 37                        pshb                            ;se envía letra (fila)
       22FD E60B                      ldab            11,x
       22FF 37                        pshb                            ;se envía num (columna)
       2300 1AEE09                    ldy             9,x
       2303 183C                      pshy                            ;se envía dirección de inicio de matriz
       2305 E608                      ldab            8,x
       2307 37                        pshb                            ;se envía jugador atacado
       2308 BD2387                    jsr             writepos
       230B                           %CLRST          6
       0006           CONT_M          SET     6
       230B                           IF CONT_M > 0
       230B 31                        ins
       230C                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       230C                           IF CONT_M > 0
       230C 31                        ins
       230D                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       230D                           IF CONT_M > 0
       230D 31                        ins
       230E                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       230E                           IF CONT_M > 0
       230E 31                        ins
       230F                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       230F                           IF CONT_M > 0
       230F 31                        ins
       2310                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2310                           IF CONT_M > 0
       2310 31                        ins
       2311                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2311                           ENDIF
       2311                           ENDIF
       2311                           ENDIF
       2311                           ENDIF
       2311                           ENDIF
       2311                           ENDIF
       2311                           ENDIF
       2311                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       2311 E60C                      ldab            12,x
       2313 37                        pshb                            ;se envía letra (fila)
       2314 E60B                      ldab            11,x
       2316 37                        pshb                            ;se envía num (columna)
       2317 1AEE0E                    ldy             14,x
       231A 183C                      pshy                            ;se envía inicio de array
       231C E608                      ldab            8,x
       231E 37                        pshb                            ;se envía jugador atacado
       231F BD240F                    jsr             play_wrt
       2322                           %CLRST          5
       0005           CONT_M          SET     5
       2322                           IF CONT_M > 0
       2322 31                        ins
       2323                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2323                           IF CONT_M > 0
       2323 31                        ins
       2324                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2324                           IF CONT_M > 0
       2324 31                        ins
       2325                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2325                           IF CONT_M > 0
       2325 31                        ins
       2326                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2326                           IF CONT_M > 0
       2326 31                        ins
       2327                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2327                           ENDIF
       2327                           ENDIF
       2327                           ENDIF
       2327                           ENDIF
       2327                           ENDIF
       2327                           ENDIF
       2327                           %RESTORE
       2327 1838                      puly
       2329 38                        pulx
       232A 33                        pulb
       232B 32                        pula
       232C 39                        rts
                      
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
                      
       232D           mirror          EQU             *       
                              
       232D                           %BACKUP
       232D 36                        psha
       232E 37                        pshb
       232F 3C                        pshx
       2330 183C                      pshy
       2332 30                        tsx                             ;creo frame pointer
       2333 A60B                      ldaa            11,x            ;cargo número de jugador
       2335 E608                      ldab            8,x             ;cargo longitud de la copia (del array a copiar)
       2337 3D                        mul                             ;obtengo offset de jugador
       2338 E30C                      addd            12,x            ;sumo offset de jugador
       233A ED0C                      std             12,x            ;guardo la dirección con offset del array fuente
                      
       233C A60C                      ldaa            12,x            ;cargo longitud de copia en contador
       233E 2710                      beq             mi_end          ;si es cero, vuelvo desde subrutina
       2340 1AEE09                    ldy             9,x             ;cargo índice a destino de la copia.
       2343 EE0C                      ldx             12,x            ;cargo índice a array origen de copia.
                      
       2345           mi_loop         EQU             *
                      
       2345 E600                      ldab            0,x             ;cargo byte a copiar
       2347 18E700                    stab            0,y             ;copio byte en destino
       234A 08                        inx                             ;siguiente byte
       234B 1808                      iny                             ;siguiente byte
       234D 4A                        deca                            ;decremento contador
       234E 26F5                      bne             mi_loop
                      
       2350           mi_end          EQU             *
                      
       2350                           %RESTORE
       2350 1838                      puly
       2352 38                        pulx
       2353 33                        pulb
       2354 32                        pula
                              
       2355 39                        rts     
                                      
                                                      
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
                      
       2356           fetchpos        EQU             *
                      
       2356                           %BACKUP
       2356 36                        psha
       2357 37                        pshb
       2358 3C                        pshx
       2359 183C                      pshy
       235B 30                        tsx                             ;creo frame pointer
       235C A608                      ldaa            8,x             ;cargo numero de jugador
       235E C624                      ldab            #ARRAY_L        ;cargo longitud del arreglo (MAX 255 bytes)
       2360 3D                        mul                             ;obtengo offset de jugador
       2361 E309                      addd            9,x             ;sumo offset de jugador a top of array
       2363 ED09                      std             9,x             ;obtengo direccion al arreglo del jugador
                      
       2365 A60C                      ldaa            12,x            ;cargo coordenada letra (FILA)
       2367 C60A                      ldab            #NORM           ;cargo valor de normalización de filas
       2369 10                        sba                             ;obtengo numero de fila
       236A C606                      ldab            #COL            ;cargo longitud de fila
       236C 3D                        mul                             ;obtengo offset de fila
       236D E309                      addd            9,x             ;sumo offset de fila a la dirección del arreglo del jugador
       236F ED09                      std             9,x             ;obtengo dirección a la fila que se desea
                      
       2371 E60B                      ldab            11,x            ;cargo coordenada (offset) de columna
       2373 5A                        decb                            ;normalizo offset de columna
       2374 4F                        clra                            ;borro AccA para tener el valor de AccB en AccD
       2375 E309                      addd            9,x             ;sumo offset de columna a la dirección temporal
       2377 ED09                      std             9,x             ;guardo en stack la dirección del byte a consultar
                      
       2379 1AEE09                    ldy             9,x             ;cargo en IY la dirección del byte que se desea obtener
       237C 18A600                    ldaa            0,y             ;cargo el byte a devolver en AccA
       237F A70C                      staa            12,x            ;guardo el byte a devolver en stack
       2381                           %RESTORE
       2381 1838                      puly
       2383 38                        pulx
       2384 33                        pulb
       2385 32                        pula
                                      
       2386 39                        rts
                                      
                                      
                      
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
                      
       2387           writepos        EQU             *
                      
       2387                           %BACKUP
       2387 36                        psha
       2388 37                        pshb
       2389 3C                        pshx
       238A 183C                      pshy
       238C 30                        tsx                             ;creo frame pointer
       238D A608                      ldaa            8,x             ;cargo numero de jugador
       238F C624                      ldab            #ARRAY_L        ;cargo longitud del arreglo (MAX 255 bytes)
       2391 3D                        mul                             ;obtengo offset de jugador
       2392 E309                      addd            9,x             ;sumo offset de jugador a top of array
       2394 ED09                      std             9,x             ;obtengo direccion al arreglo del jugador
                      
       2396 A60C                      ldaa            12,x            ;cargo coordenada letra (FILA)
       2398 C60A                      ldab            #NORM           ;cargo valor de normalización de filas
       239A 10                        sba                             ;obtengo numero de fila
       239B C606                      ldab            #COL            ;cargo longitud de fila
       239D 3D                        mul                             ;obtengo offset de fila
       239E E309                      addd            9,x             ;sumo offset de fila a la dirección del arreglo del jugador
       23A0 ED09                      std             9,x             ;obtengo dirección a la fila que se desea
                      
       23A2 E60B                      ldab            11,x            ;cargo coordenada (offset) de columna
       23A4 5A                        decb                            ;normalizo offset de columna
       23A5 4F                        clra                            ;borro AccA para tener el valor de AccB en AccD
       23A6 E309                      addd            9,x             ;sumo offset de columna a la dirección temporal
       23A8 ED09                      std             9,x             ;guardo en stack la dirección del byte a consultar
                      
       23AA 1AEE09                    ldy             9,x             ;cargo en IY la dirección donde se desea escribir
       23AD A60D                      ldaa            13,x            ;cargo en AccA el byte que se quiere escribir en el arreglo
       23AF 18A700                    staa            0,y             ;escribo el byte sobre el arreglo
       23B2                           %RESTORE
       23B2 1838                      puly
       23B4 38                        pulx
       23B5 33                        pulb
       23B6 32                        pula
                                      
       23B7 39                        rts
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
                      
       23B8           play_src        EQU             *               
                                              
       23B8                           %BACKUP
       23B8 36                        psha
       23B9 37                        pshb
       23BA 3C                        pshx
       23BB 183C                      pshy
       23BD 30                        tsx                             ;creo frame pointer
       23BE A608                      ldaa            8,x             ;cargo número de array (jugador)
       23C0 C613                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       23C2 3D                        mul                             ;obtengo offset de jugador
       23C3 E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       23C5 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
                      
       23C7           ps_loop         EQU             *
                      
       23C7 18A600                    ldaa            0,y             ;cargo byte1 a comparar
       23CA 2718                      beq             ps_ntfnd        ;si encontré el terminador, el valor requerido no está en el array
                      
       23CC A10C                      cmpa            12,x            ;comparo con el byte1 a buscar
       23CE 2706                      beq             ps_next         ;si coinciden, busco el byte 2
       23D0 1808                      iny                             ;me posiciono sobre siguiente byte1 a comparar
       23D2 1808                      iny
       23D4 20F1                      bra             ps_loop         ;vuelvo a comparar
                      
       23D6           ps_next         EQU             *
                      
       23D6 18E601                    ldab            1,y             ;cargo byte a continuación del anterior
       23D9 1808                      iny                             ;me posiciono sobre siguiente byte1 a comparar
       23DB 1808                      iny
       23DD E10B                      cmpb            11,x            ;comparo con byte2
       23DF 26E6                      bne             ps_loop         ;si no son iguales, sigo buscando
       23E1 0D                        sec                             ;set del carry para informar que se encontró el valor
       23E2 2001                      bra             ps_end
                                              
       23E4           ps_ntfnd        EQU             *
                      
       23E4 0C                        clc                             ;carry en 0 para informar que no se encontró el valor
                      
       23E5           ps_end          EQU             *
                                      
       23E5                           %RESTORE
       23E5 1838                      puly
       23E7 38                        pulx
       23E8 33                        pulb
       23E9 32                        pula
                      
       23EA 39                        rts
                      
                      
                      
                                              
                      
                                              
                      
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
                      
       23EB           play_cnt        EQU             *
                      
       23EB                           %BACKUP
       23EB 36                        psha
       23EC 37                        pshb
       23ED 3C                        pshx
       23EE 183C                      pshy
       23F0 30                        tsx                             ;creo frame pointer
       23F1 A608                      ldaa            8,x             ;cargo número de array (jugador)
       23F3 C613                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       23F5 3D                        mul                             ;obtengo offset de jugador
       23F6 E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       23F8 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
       23FA 4F                        clra                            ;inicializo contador
                                              
       23FB           pc_loop         EQU             *               
                                              
       23FB 18E600                    ldab            0,y             ;cargo byte a comparar
       23FE 2707                      beq             pc_end          ;si es terminador, terminé de buscar
       2400 1808                      iny                             ;si no es terminador, me posiciono sobre siguiente dato de 
       2402 1808                      iny                             ;2 bytes
       2404 4C                        inca                            ;incremento contador
       2405 20F4                      bra             pc_loop         ;vuelvo a comparar
                                              
       2407           pc_end          EQU             *       
                                              
       2407 A70A                      staa            10,x            ;devuelvo valor por stack
       2409                           %RESTORE
       2409 1838                      puly
       240B 38                        pulx
       240C 33                        pulb
       240D 32                        pula
                                      
       240E 39                        rts
                      
                      
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
                      
       240F           play_wrt        EQU             *
                      
       240F                           %BACKUP
       240F 36                        psha
       2410 37                        pshb
       2411 3C                        pshx
       2412 183C                      pshy
       2414 30                        tsx                             ;creo frame pointer
       2415 A608                      ldaa            8,x             ;cargo número de array (jugador)
       2417 C613                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       2419 3D                        mul                             ;obtengo offset de jugador
       241A E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       241C 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
                      
                      
       241E           pw_loop         EQU             *
                      
       241E 18A600                    ldaa            0,y             ;cargo byte del array
       2421 2704                      beq             pw_end          ;si es cero, entonces escribo los datos
       2423 1808                      iny                             ;si no es cero, me posiciono sobre la siguiente posición
       2425 20F7                      bra             pw_loop         ;vuelvo a buscar ceros
                      
       2427           pw_end          EQU             *
                      
       2427 A60C                      ldaa            12,x            ;cargo primer byte a escribir
       2429 18A700                    staa            0,y             ;guardo primer byte en memoria
       242C 1808                      iny                             ;me posiciono en el siguiente byte
       242E A60B                      ldaa            11,x            ;cargo segundo byte a escribir
       2430 18A700                    staa            0,y             ;guardo segundo byte en memoria
       2433                           %RESTORE
       2433 1838                      puly
       2435 38                        pulx
       2436 33                        pulb
       2437 32                        pula
                      
       2438 39                        rts
                      
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
       2439           clrzone         EQU     *
                      
       2439                           %BACKUP                         ;backup de registros en stack
       2439 36                        psha
       243A 37                        pshb
       243B 3C                        pshx
       243C 183C                      pshy
                      
       243E 30                        tsx
       243F A608                      ldaa    8,x                     ;cargo contador con longitud de zona a borrar
       2441 EE09                      ldx     9,x                     ;cargo puntero zona de memoria a borrar
       2443 D600                      ldab    $00                     ;cargo el cero
       2445           clr_loop        EQU     *
       2445 E700                      stab    0,x                     ;guardo caracter en posicion del array
       2447 08                        inx                             ;siguiente posición     
       2448 4A                        deca                            ;decremento contador
       2449 26FA                      bne     clr_loop                ;si no es cero, escribo en la posición siguiente.
                      
       244B                           %RESTORE                        :restauro backup en registros
       244B 1838                      puly
       244D 38                        pulx
       244E 33                        pulb
       244F 32                        pula
                      
       2450 39                        rts
                      
                      *---------------------------------------
                      * WhoPlays: subrutina para ver que 
                      *           jugador juega
                      *---------------------------------------
                      
       2451           whoplays        EQU             *
                      
       2451 BD250E    wait_ply        jsr             read_kb
       2454 4D                        tsta
       2455 2BFA                      bmi             wait_ply
       2457 8103                      cmpa            #B_P1
       2459 2708                      beq             play_p1
       245B 8107                      cmpa            #B_P2
       245D 26F2                      bne             wait_ply
       245F 8600                      ldaa            #P1
       2461 2002                      bra             ply_end
                      
       2463 8601      play_p1         ldaa            #P2
       2465 39        ply_end         rts             
                      
                      *---------------------------------------
                      * Input: subrutina que devuelve
                      *        entrada para la batalla naval
                      * AccA: Fila de la posicion jugada
                      * AccB: Columna de la posicion jugada
                      *---------------------------------------
                      
       2466           input           EQU             *
                      
                      * back up de registros
       2466 3C                        pshx
       2467 183C                      pshy
                      
                      * creo variables locales
       2469 36                        psha
       246A 36                        psha
       246B 36                        psha
                      
                      * inicializo framepointer
       246C 1830                      tsy
                      
                      * prendo el led
       246E                           %LED_INV
       246E B61000                    ldaa            PORTA
       2471 8840                      eora            #SHF_LED
       2473 B71000                    staa            PORTA
                      
                      * inicializo el contador
       2476 8603      inp_rest        ldaa            #INPUTS
       2478 18A700                    staa            CONT,Y
                                      
                      * limpio el display
       247B CE41F9                    ldx             #string
       247E 8604                      ldaa            #4
       2480 C620                      ldab            #ESP
       2482 BD25D8                    jsr             clrarray
                      
       2485 BD25C7                    jsr             clr_dis
                      
       2488 86FF                      ldaa            #$FF
       248A B741F8                    staa            buffer
                      
                      * lectura de teclado
       248D BD250E    wait_kb         jsr             read_kb
       2490 4D                        tsta
       2491 2BFA                      bmi             wait_kb
       2493 B141F8                    cmpa            buffer
       2496 27F5                      beq             wait_kb
       2498 B741F8                    staa            buffer
                                      
       249B 810F                      cmpa            #B_reset
       249D 27D7                      beq             inp_rest
                      
       249F 186A00                    dec             CONT,Y
       24A2 2727                      beq             chk_fire
       24A4 36                        psha
       24A5 CE41FE                    ldx             #kb_value
       24A8 16                        tab
       24A9 3A                        abx
       24AA A600                      ldaa            0,X             
       24AC 183C                      pshy
       24AE 18E600                    ldab            CONT,Y
       24B1 183A                      aby
       24B3 18A700                    staa            0,Y
       24B6 1838                      puly
                      
       24B8 33                        pulb
       24B9 BD25B7                    jsr             to_ascii
       24BC 17                        tba
                                      
       24BD CE41F9                    ldx             #string
       24C0 BD25A7                    jsr             sh_value
                      
       24C3 CE41F9                    ldx             #string
       24C6 BD24FA                    jsr             show
                      
       24C9 20C2                      bra             wait_kb
                      
       24CB 810B      chk_fire        cmpa            #B_fire
       24CD 26A7                      bne             inp_rest
       24CF 18A602                    ldaa            FILA_L,Y
       24D2 810A                      cmpa            #MIN_FIL
       24D4 25A0                      blo             inp_rest
       24D6 810F                      cmpa            #MAX_FIL
       24D8 229C                      bhi             inp_rest
       24DA 18A601                    ldaa            COL_L,Y
       24DD 8101                      cmpa            #MIN_COL
       24DF 2595                      blo             inp_rest
       24E1 8106                      cmpa            #MAX_COL
       24E3 2291                      bhi             inp_rest
                      
                      * apago el led
       24E5                           %LED_INV
       24E5 B61000                    ldaa            PORTA
       24E8 8840                      eora            #SHF_LED
       24EA B71000                    staa            PORTA
                      
                      * libero variables locales
       24ED 32                        pula
       24EE 32                        pula
       24EF 32                        pula
                      
       24F0 18A602    return          ldaa            FILA_L,Y
       24F3 18E601                    ldab            COL_L,Y         
                      
                      * restore de registros
       24F6 1838                      puly
       24F8 38                        pulx
                      
       24F9 39                        rts
                      
                      *------------------------------------
                      * Show: Por IX direccion de string de
                      *       4 caracteres para imprimir
                      *------------------------------------
                      
       24FA           show            EQU             *
                      
                      * back up de registros
       24FA 3C                        pshx
       24FB 37                        pshb
       24FC 36                        psha
                      
                      * inicializo contador de posiciones
       24FD C604                      ldab            #LENGTH
                                      
                      * loop para enviar a cada posicion
       24FF 5A        sh_loop         decb
       2500 2B08                      bmi             sh_end          ; si no pase por todas las posiciones
       2502 A600                      ldaa            0,X             ; tomo del string el caracter
       2504 BD25C0                    jsr             outchar         ; y lo imprimo en la posicion actual
       2507 08                        inx
       2508 20F5                      bra             sh_loop
                      
                      * restore de registros                          
       250A 32        sh_end          pula
       250B 33                        pulb
       250C 38                        pulx
       250D 39                        rts
                      
                      *----------------------------------------------
                      * read_kb: AccA: tecla presionada
                      *          o -1
                      *----------------------------------------------
                      
       250E           read_kb         EQU             *
                      
                      * back up de registros
       250E 3C                        pshx
       250F 37                        pshb
                      
                      * creo espacio para variables locales
       2510                           %MALLOC         1               ; variable mascara de filas
       2510                           IF 1 > 0
       2510 30                        tsx
       2511 8F                        xgdx
       2512 830001                    subd    #1
       2515 8F                        xgdx
       2516 35                        txs
       2517                           ENDIF
       2517                           %MALLOC         1               ; variable mascara de columnas
       2517                           IF 1 > 0
       2517 30                        tsx
       2518 8F                        xgdx
       2519 830001                    subd    #1
       251C 8F                        xgdx
       251D 35                        txs
       251E                           ENDIF
       251E                           %MALLOC         1               ; variable contador de columnas
       251E                           IF 1 > 0
       251E 30                        tsx
       251F 8F                        xgdx
       2520 830001                    subd    #1
       2523 8F                        xgdx
       2524 35                        txs
       2525                           ENDIF
       2525                           %MALLOC         1               ; variable contador de tecla
       2525                           IF 1 > 0
       2525 30                        tsx
       2526 8F                        xgdx
       2527 830001                    subd    #1
       252A 8F                        xgdx
       252B 35                        txs
       252C                           ENDIF
                      
                      * inicializacion de variables
       252C 30                        tsx                             ; framepointer para acceder a locales
       252D C604                      ldab            #LENGTH         ; contador de fila
       252F 860E                      ldaa            #FIL0           ; mascara para la fila cero
       2531 A703                      staa            MASK_FIL,X      
       2533 8610                      ldaa            #COL0           ; mascara para la columna cero
       2535 A702                      staa            MASK_COL,X
       2537 4F                        clra                            ; contador de columna
       2538 A701                      staa            COL_CONT,X
       253A A700                      staa            KB_PRESS,X      ; variable con numero de tecla
                      
                      * loop para barrer y mirar teclado matricial
       253C 86FF      kb_loop         ldaa            #NONE
       253E 5A                        decb                            
       253F 2B2C                      bmi             kb_end          ; me fijo si recorrio todas las filas
       2541 A603                      ldaa            MASK_FIL,X      ; enmascaro la siguiente fila
       2543 B71003                    staa            ROWS
       2546 0D                        sec                             ; carry para rotacion de byte
       2547 49                        rola                            
       2548 A703                      staa            MASK_FIL,X      ; cambio variable mascara para la proxima fila
                      
       254A 8604                      ldaa            #LENGTH
       254C A701                      staa            COL_CONT,X
       254E 8610                      ldaa            #COL0           ; inicializo variable para barrer
       2550 A702                      staa            MASK_COL,X      ; la columna
                      
       2552 6A01      kb_col          dec             COL_CONT,X      ; me fijo si barrio todas las columnas
       2554 2BE6                      bmi             kb_loop
       2556 B61005                    ldaa            COLS            ; leo el puerto de columnas
       2559 A402                      anda            MASK_COL,X      ; enmascaro columna
       255B 2707                      beq             kb_hit          ; si es nulo apretaron un boton
       255D 48                        lsla                            ; cambio mascara para proxima columna
       255E A702                      staa            MASK_COL,X
       2560 6C00                      inc             KB_PRESS,X      ; incremento numero de tecla
       2562 20EE                      bra             kb_col
                      
       2564 A600      kb_hit          ldaa            KB_PRESS,X      ; apretaron una tecla, lo comparo
       2566 B141F7                    cmpa            kb_buff         ; con la que fue apretada antes
       2569 2602                      bne             kb_end          ; aviso por led
                      
       256B 86FF      kb_none         ldaa            #NONE           ; en caso de que no apreten nada
                      
       256D B741F7    kb_end          staa            kb_buff         ; actualizo variable
                                              
                      * elimino variables locales
       2570                           %RELMEM         1
       2570                           IF 1 > 0
       2570 30                        tsx
       2571 8F                        xgdx
       2572 C30001                    addd    #1
       2575 8F                        xgdx
       2576 35                        txs
       2577                           ENDIF           
       2577                           %RELMEM         1
       2577                           IF 1 > 0
       2577 30                        tsx
       2578 8F                        xgdx
       2579 C30001                    addd    #1
       257C 8F                        xgdx
       257D 35                        txs
       257E                           ENDIF           
       257E                           %RELMEM         1
       257E                           IF 1 > 0
       257E 30                        tsx
       257F 8F                        xgdx
       2580 C30001                    addd    #1
       2583 8F                        xgdx
       2584 35                        txs
       2585                           ENDIF           
       2585                           %RELMEM         1
       2585                           IF 1 > 0
       2585 30                        tsx
       2586 8F                        xgdx
       2587 C30001                    addd    #1
       258A 8F                        xgdx
       258B 35                        txs
       258C                           ENDIF           
                      
                      * restore de registros
       258C 33                        pulb
       258D 38                        pulx
                                      
       258E 39                        rts
                      
                      *----------------------------------------------
                      * init_kb: subrutina para inicializar teclado
                      *----------------------------------------------
                      
       258F           init_kb         EQU             *
                      
                      * back up de registros
       258F 36                        psha
                      
                      * inicializacion de teclado             
       2590 86FF                      ldaa            #NONE
       2592 B741F7                    staa            kb_buff
                      
                      * restore de registros
       2595 32                        pula
       2596 39                        rts
                      
                      *--------------------------------
                      * Inicializa display (y teclado)
                      *--------------------------------
                      
       2597           initdis         EQU             *
                              
       2597 36                        psha
                      
       2598 860F                      ldaa            #$0F       ;set for input/output
       259A B71007                    staa            DDRC       ;C output
       259D B61002                    ldaa            PIOC       ;get current value
       25A0 84FE                      anda            #$FE       ;reset bit 0 to 0
       25A2 B71002                    staa            PIOC       ;send it out
                              
       25A5 32                        pula
       25A6 39                        rts
                      
                      *--------------------------------------------------------------
                      * sh_value: toma un string, desplaza los caracteres a izquierda
                      *           y agrega un nuevo caracter
                      *           IX: direccion del string
                      *           AccA: nuevo valor
                      *--------------------------------------------------------------
                      
       25A7           sh_value        EQU             *
                      
                      * back up de registros
       25A7 3C                        pshx
       25A8 37                        pshb
                      
                      * loop para desplazar caracteres
       25A9 E601      v_loop          ldab            1,X             ;tomo elemento siguiente en string
       25AB 2705                      beq             v_end
       25AD E700                      stab            0,X             ;y lo guardo en posicion actual
       25AF 08                        inx
       25B0 20F7                      bra             v_loop          
                      
       25B2 A700      v_end           staa            0,X             ;ingresa nuevo caracter en cadena desplazada
                      
                      * restore de registros
       25B4 33                        pulb
       25B5 38                        pulx
       25B6 39                        rts
                      
                      *----------------------------------------------
                      * toAscii: toma numero de boton del teclado
                      *          y devuelve su correspondiente en
                      *          ascii
                      *          toma  AccB: valor del boton
                      *          devuelve AccB: ascii
                      *----------------------------------------------
                      
       25B7           to_ascii        EQU             *
                      
                      * back up de registros
       25B7 3C                        pshx
                      
                      * inicializacion de variables
       25B8 CE420D                    ldx             #kb_ascii       ; apunto a string con caracteres asignados por tecla
       25BB 3A                        abx                             ; me muevo en string con indice y recupero segun tecla presionada
       25BC E600                      ldab            0,X             
                      
                      * restore de registros
       25BE 38                        pulx
       25BF 39                        rts
                      
                      *--------------------------------
                      *Outchar: ACCA: Carácter a enviar
                      *           ACCB: Posición (0-3)
                      *--------------------------------
       25C0           outchar         EQU             *
                              
       25C0 F71005                    stab            PORTCL          ;set the position 
       25C3 B71004                    staa            PORTB           ;set the character
                              
       25C6 39                        rts
                      
                      *-------------------------
                      * clr_dis: Limpia display
                      *-------------------------
                      
       25C7           clr_dis         EQU             *
                      
                      * hago back up de registros
       25C7 37                        pshb
       25C8 36                        psha
                      
                      * inicializo el contador de posiciones
       25C9 C604                      ldab            #LENGTH
                      
                      * loop para limpiar cada posicion
       25CB 5A        clrloop         decb
       25CC 2B07                      bmi             clr_end         ; si no pase por todas las posiciones
       25CE 8620                      ldaa            #ESP            ; cargo en esta posicion espacio blanco
       25D0 BD25C0                    jsr             outchar         ; y lo envio
       25D3 20F6                      bra             clrloop         
                      
                      * restore de registros
       25D5 32        clr_end         pula
       25D6 33                        pulb
       25D7 39                        rts
                      
                      *-------------------------------------------------------
                      * clrarray: subrutina para limpiar un array dado
                      * parametros: IX direccion del array
                      *             AccA cantidad de elementos del array
                      *             AccB caracter a poner
                      *-------------------------------------------------------
                      
       25D8           clrarray        EQU             *
                                      
       25D8 4A        cra_loop        deca                            ;contador con cantidad de elementos a limpiar
       25D9 2705                      beq             cra_end
       25DB E700                      stab            0,X             ;reemplazar elemento por caracter a poner
       25DD 08                        inx
       25DE 20F8                      bra             cra_loop
                                      
       25E0 E700      cra_end         stab            0,X
       25E2 39                        rts
                      
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
     ytes:   1650         # main_bn #
     RC:     A2C8         ###########
 