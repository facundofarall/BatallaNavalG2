################################################################################
#                                                                              #
#    Micro Series 6801 Assembler V2.00/DOS              27/Nov/17  12:18:57    #
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
       2003 BD25A5                    jsr             initdis         ;inicialización de display 7 seg
       2006 BD259D                    jsr             init_kb         ;inicialización de teclado
       2009 CE421C                    ldx             #hitlist
       200C 3C                        pshx                            ;se envía inicio de arreglo de barcos a poner en 0
       200D 8626                      ldaa            #(HITSIZE*2)
       200F 36                        psha                            ;se envía el tamaño del arreglo a limpiar
       2010 BD243F                    jsr             clrzone
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
       201B BD2457                    jsr             whoplays        ;se recibirá en AccA el jugador a atacar a continuación
       201E B741F4                    staa            player          ;se habilita la siguiente jugada
                      
       2021 CE5000                    ldx             #p_board
       2024 3C                        pshx                            ;se envía el inicio de las matrices de tableros
       2025 B641F4                    ldaa            player
       2028 36                        psha                            ;se envía el jugador atacado
       2029 CE5F00                    ldx             #display
       202C 3C                        pshx                            ;se envía la matriz de display (destino a copiar)
       202D 8624                      ldaa            #ARRAY_L
       202F 36                        psha                            ;se envía el tamaño de la matriz tablero
       2030 BD2333                    jsr             mirror          ;se envía el tablero del jugador atacado al display
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
                      
       2039 BD246C                    jsr             input           ;se llama a la subrutina de interfaz de usuario
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
       2052 BD23BE                    jsr             play_src        ;se recibirá el carry en 1 si la posición corresponde a un barco ya atacado
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
       206C BD235C                    jsr             fetchpos
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
       2089 BD20CB                    jsr             gameplay
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
       20A3 BD2333                    jsr             mirror          ;se envía el tablero del jugador atacado al display
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
       20B4 BD23F1                    jsr             play_cnt
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
       20C1 B641F4                    ldaa            player
       20C4 36                        psha
       20C5 BD25F1                    jsr             winner
       20C8                           %CLRST          1
       0001           CONT_M          SET     1
       20C8                           IF CONT_M > 0
       20C8 31                        ins
       20C9                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       20C9                           ENDIF
       20C9                           ENDIF
                      
       20C9           end_loop        EQU             *
       20C9 20FE                      bra             end_loop
                      
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
       20CB           gameplay        EQU             *
       20CB                           %BACKUP
       20CB 36                        psha
       20CC 37                        pshb
       20CD 3C                        pshx
       20CE 183C                      pshy
                                      
       20D0 30                        tsx                             ;se usa el IX como frame pointer
       20D1 A60D                      ldaa            13,x            ;se carga en AccA el byte a analizar
       20D3 84C0                      anda            #11000000b      ;se enmascan los bits 0 a 5, para analizar los más significativos
                      
       20D5           gp_water        EQU             *
       20D5 2620                      bne             gp_sub          ;se estudia si era agua
       20D7                           %WRITEPOS       #WATER          ;se cambia el color en el tablero
       20D7 C608                      ldab            #WATER
       20D9 37                        pshb                            ;se envía el byte a escribir
       20DA E60C                      ldab            12,x
       20DC 37                        pshb                            ;se envía letra (fila)
       20DD E60B                      ldab            11,x
       20DF 37                        pshb                            ;se envía num (columna)
       20E0 1AEE09                    ldy             9,x
       20E3 183C                      pshy                            ;se envía dirección de inicio de matriz
       20E5 E608                      ldab            8,x
       20E7 37                        pshb                            ;se envía jugador atacado
       20E8 BD238D                    jsr             writepos
       20EB                           %CLRST          6
       0006           CONT_M          SET     6
       20EB                           IF CONT_M > 0
       20EB 31                        ins
       20EC                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       20EC                           IF CONT_M > 0
       20EC 31                        ins
       20ED                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       20ED                           IF CONT_M > 0
       20ED 31                        ins
       20EE                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       20EE                           IF CONT_M > 0
       20EE 31                        ins
       20EF                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       20EF                           IF CONT_M > 0
       20EF 31                        ins
       20F0                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       20F0                           IF CONT_M > 0
       20F0 31                        ins
       20F1                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       20F1                           ENDIF
       20F1                           ENDIF
       20F1                           ENDIF
       20F1                           ENDIF
       20F1                           ENDIF
       20F1                           ENDIF
       20F1                           ENDIF
       20F1                           %RESTORE
       20F1 1838                      puly
       20F3 38                        pulx
       20F4 33                        pulb
       20F5 32                        pula
       20F6 39                        rts
                      
       20F7           gp_sub          EQU             *
       20F7 8140                      cmpa            #01000000b      ;se estudia si era un submarino
       20F9 2636                      bne             gp_shi_r
       20FB                           %WRITEPOS       #SUNK_SUB       ;se cambia el color en el tablero
       20FB C650                      ldab            #SUNK_SUB
       20FD 37                        pshb                            ;se envía el byte a escribir
       20FE E60C                      ldab            12,x
       2100 37                        pshb                            ;se envía letra (fila)
       2101 E60B                      ldab            11,x
       2103 37                        pshb                            ;se envía num (columna)
       2104 1AEE09                    ldy             9,x
       2107 183C                      pshy                            ;se envía dirección de inicio de matriz
       2109 E608                      ldab            8,x
       210B 37                        pshb                            ;se envía jugador atacado
       210C BD238D                    jsr             writepos
       210F                           %CLRST          6
       0006           CONT_M          SET     6
       210F                           IF CONT_M > 0
       210F 31                        ins
       2110                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2110                           IF CONT_M > 0
       2110 31                        ins
       2111                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2111                           IF CONT_M > 0
       2111 31                        ins
       2112                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2112                           IF CONT_M > 0
       2112 31                        ins
       2113                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2113                           IF CONT_M > 0
       2113 31                        ins
       2114                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2114                           IF CONT_M > 0
       2114 31                        ins
       2115                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2115                           ENDIF
       2115                           ENDIF
       2115                           ENDIF
       2115                           ENDIF
       2115                           ENDIF
       2115                           ENDIF
       2115                           ENDIF
       2115                           %PLAY_WRT                       ;se agraga la posición al array de barcos atacados
       2115 E60C                      ldab            12,x
       2117 37                        pshb                            ;se envía letra (fila)
       2118 E60B                      ldab            11,x
       211A 37                        pshb                            ;se envía num (columna)
       211B 1AEE0E                    ldy             14,x
       211E 183C                      pshy                            ;se envía inicio de array
       2120 E608                      ldab            8,x
       2122 37                        pshb                            ;se envía jugador atacado
       2123 BD2415                    jsr             play_wrt
       2126                           %CLRST          5
       0005           CONT_M          SET     5
       2126                           IF CONT_M > 0
       2126 31                        ins
       2127                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2127                           IF CONT_M > 0
       2127 31                        ins
       2128                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2128                           IF CONT_M > 0
       2128 31                        ins
       2129                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2129                           IF CONT_M > 0
       2129 31                        ins
       212A                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       212A                           IF CONT_M > 0
       212A 31                        ins
       212B                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       212B                           ENDIF
       212B                           ENDIF
       212B                           ENDIF
       212B                           ENDIF
       212B                           ENDIF
       212B                           ENDIF
       212B                           %RESTORE
       212B 1838                      puly
       212D 38                        pulx
       212E 33                        pulb
       212F 32                        pula
       2130 39                        rts
                      
       2131           gp_shi_r        EQU             *               ;si no era ninguno de los anteriores, es un carguero
       2131 E60B                      ldab            11,x
       2133 C106                      cmpb            #MAX_C
       2135 276D                      beq             gp_shi_l        ;si está en el borde derecho, se continúa
       2137 E60C                      ldab            12,x
       2139 37                        pshb                            ;se envía la letra (fila)
       213A E60B                      ldab            11,x
       213C 5C                        incb                            ;se avanza a la derecha
       213D 37                        pshb                            ;se envía el num (columna)
       213E 1AEE0E                    ldy             14,x
       2141 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       2143 E608                      ldab            8,x
       2145 37                        pshb                            ;se envía el jugador atacado
       2146 BD23BE                    jsr             play_src        ;se busca la posición de la derecha
       2149                           %CLRST          5
       0005           CONT_M          SET     5
       2149                           IF CONT_M > 0
       2149 31                        ins
       214A                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       214A                           IF CONT_M > 0
       214A 31                        ins
       214B                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       214B                           IF CONT_M > 0
       214B 31                        ins
       214C                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       214C                           IF CONT_M > 0
       214C 31                        ins
       214D                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       214D                           IF CONT_M > 0
       214D 31                        ins
       214E                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       214E                           ENDIF
       214E                           ENDIF
       214E                           ENDIF
       214E                           ENDIF
       214E                           ENDIF
       214E                           ENDIF
       214E 2454                      bcc             gp_shi_l        ;si no había barco atacado en la derecha, se busca la izquierda
       2150                           %WRITEPOS       #SUNK_SHI       
       2150 C690                      ldab            #SUNK_SHI
       2152 37                        pshb                            ;se envía el byte a escribir
       2153 E60C                      ldab            12,x
       2155 37                        pshb                            ;se envía letra (fila)
       2156 E60B                      ldab            11,x
       2158 37                        pshb                            ;se envía num (columna)
       2159 1AEE09                    ldy             9,x
       215C 183C                      pshy                            ;se envía dirección de inicio de matriz
       215E E608                      ldab            8,x
       2160 37                        pshb                            ;se envía jugador atacado
       2161 BD238D                    jsr             writepos
       2164                           %CLRST          6
       0006           CONT_M          SET     6
       2164                           IF CONT_M > 0
       2164 31                        ins
       2165                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2165                           IF CONT_M > 0
       2165 31                        ins
       2166                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2166                           IF CONT_M > 0
       2166 31                        ins
       2167                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2167                           IF CONT_M > 0
       2167 31                        ins
       2168                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2168                           IF CONT_M > 0
       2168 31                        ins
       2169                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2169                           IF CONT_M > 0
       2169 31                        ins
       216A                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       216A                           ENDIF
       216A                           ENDIF
       216A                           ENDIF
       216A                           ENDIF
       216A                           ENDIF
       216A                           ENDIF
       216A                           ENDIF
       216A 6C0B                      inc             11,x
       216C                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       216C C690                      ldab            #SUNK_SHI
       216E 37                        pshb                            ;se envía el byte a escribir
       216F E60C                      ldab            12,x
       2171 37                        pshb                            ;se envía letra (fila)
       2172 E60B                      ldab            11,x
       2174 37                        pshb                            ;se envía num (columna)
       2175 1AEE09                    ldy             9,x
       2178 183C                      pshy                            ;se envía dirección de inicio de matriz
       217A E608                      ldab            8,x
       217C 37                        pshb                            ;se envía jugador atacado
       217D BD238D                    jsr             writepos
       2180                           %CLRST          6
       0006           CONT_M          SET     6
       2180                           IF CONT_M > 0
       2180 31                        ins
       2181                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2181                           IF CONT_M > 0
       2181 31                        ins
       2182                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2182                           IF CONT_M > 0
       2182 31                        ins
       2183                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2183                           IF CONT_M > 0
       2183 31                        ins
       2184                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2184                           IF CONT_M > 0
       2184 31                        ins
       2185                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2185                           IF CONT_M > 0
       2185 31                        ins
       2186                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2186                           ENDIF
       2186                           ENDIF
       2186                           ENDIF
       2186                           ENDIF
       2186                           ENDIF
       2186                           ENDIF
       2186                           ENDIF
       2186 6A0B                      dec             11,x            ;se recupera el dato
       2188                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       2188 E60C                      ldab            12,x
       218A 37                        pshb                            ;se envía letra (fila)
       218B E60B                      ldab            11,x
       218D 37                        pshb                            ;se envía num (columna)
       218E 1AEE0E                    ldy             14,x
       2191 183C                      pshy                            ;se envía inicio de array
       2193 E608                      ldab            8,x
       2195 37                        pshb                            ;se envía jugador atacado
       2196 BD2415                    jsr             play_wrt
       2199                           %CLRST          5
       0005           CONT_M          SET     5
       2199                           IF CONT_M > 0
       2199 31                        ins
       219A                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       219A                           IF CONT_M > 0
       219A 31                        ins
       219B                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       219B                           IF CONT_M > 0
       219B 31                        ins
       219C                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       219C                           IF CONT_M > 0
       219C 31                        ins
       219D                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       219D                           IF CONT_M > 0
       219D 31                        ins
       219E                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       219E                           ENDIF
       219E                           ENDIF
       219E                           ENDIF
       219E                           ENDIF
       219E                           ENDIF
       219E                           ENDIF
       219E                           %RESTORE
       219E 1838                      puly
       21A0 38                        pulx
       21A1 33                        pulb
       21A2 32                        pula
       21A3 39                        rts
                      
       21A4           gp_shi_l        EQU             *
       21A4 E60B                      ldab            11,x
       21A6 C101                      cmpb            #MIN_C
       21A8 276D                      beq             gp_shi_u        ;si está en el borde izquierdo, se continúa
       21AA E60C                      ldab            12,x
       21AC 37                        pshb                            ;se envía la letra (fila)
       21AD E60B                      ldab            11,x
       21AF 5A                        decb                            ;se retrocede a la izquierda
       21B0 37                        pshb                            ;se envía el número (columna)
       21B1 1AEE0E                    ldy             14,x
       21B4 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       21B6 E608                      ldab            8,x
       21B8 37                        pshb                            ;se envía el jugador atacado
       21B9 BD23BE                    jsr             play_src        ;se busca la posición de la izquierda
       21BC                           %CLRST          5
       0005           CONT_M          SET     5
       21BC                           IF CONT_M > 0
       21BC 31                        ins
       21BD                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       21BD                           IF CONT_M > 0
       21BD 31                        ins
       21BE                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       21BE                           IF CONT_M > 0
       21BE 31                        ins
       21BF                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       21BF                           IF CONT_M > 0
       21BF 31                        ins
       21C0                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       21C0                           IF CONT_M > 0
       21C0 31                        ins
       21C1                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       21C1                           ENDIF
       21C1                           ENDIF
       21C1                           ENDIF
       21C1                           ENDIF
       21C1                           ENDIF
       21C1                           ENDIF
       21C1 2454                      bcc             gp_shi_u        ;si no había barco atacado a la izquierda, se busca arriba
       21C3                           %WRITEPOS       #SUNK_SHI       
       21C3 C690                      ldab            #SUNK_SHI
       21C5 37                        pshb                            ;se envía el byte a escribir
       21C6 E60C                      ldab            12,x
       21C8 37                        pshb                            ;se envía letra (fila)
       21C9 E60B                      ldab            11,x
       21CB 37                        pshb                            ;se envía num (columna)
       21CC 1AEE09                    ldy             9,x
       21CF 183C                      pshy                            ;se envía dirección de inicio de matriz
       21D1 E608                      ldab            8,x
       21D3 37                        pshb                            ;se envía jugador atacado
       21D4 BD238D                    jsr             writepos
       21D7                           %CLRST          6
       0006           CONT_M          SET     6
       21D7                           IF CONT_M > 0
       21D7 31                        ins
       21D8                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       21D8                           IF CONT_M > 0
       21D8 31                        ins
       21D9                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       21D9                           IF CONT_M > 0
       21D9 31                        ins
       21DA                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       21DA                           IF CONT_M > 0
       21DA 31                        ins
       21DB                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       21DB                           IF CONT_M > 0
       21DB 31                        ins
       21DC                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       21DC                           IF CONT_M > 0
       21DC 31                        ins
       21DD                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       21DD                           ENDIF
       21DD                           ENDIF
       21DD                           ENDIF
       21DD                           ENDIF
       21DD                           ENDIF
       21DD                           ENDIF
       21DD                           ENDIF
       21DD 6A0B                      dec             11,x
       21DF                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       21DF C690                      ldab            #SUNK_SHI
       21E1 37                        pshb                            ;se envía el byte a escribir
       21E2 E60C                      ldab            12,x
       21E4 37                        pshb                            ;se envía letra (fila)
       21E5 E60B                      ldab            11,x
       21E7 37                        pshb                            ;se envía num (columna)
       21E8 1AEE09                    ldy             9,x
       21EB 183C                      pshy                            ;se envía dirección de inicio de matriz
       21ED E608                      ldab            8,x
       21EF 37                        pshb                            ;se envía jugador atacado
       21F0 BD238D                    jsr             writepos
       21F3                           %CLRST          6
       0006           CONT_M          SET     6
       21F3                           IF CONT_M > 0
       21F3 31                        ins
       21F4                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       21F4                           IF CONT_M > 0
       21F4 31                        ins
       21F5                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       21F5                           IF CONT_M > 0
       21F5 31                        ins
       21F6                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       21F6                           IF CONT_M > 0
       21F6 31                        ins
       21F7                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       21F7                           IF CONT_M > 0
       21F7 31                        ins
       21F8                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       21F8                           IF CONT_M > 0
       21F8 31                        ins
       21F9                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       21F9                           ENDIF
       21F9                           ENDIF
       21F9                           ENDIF
       21F9                           ENDIF
       21F9                           ENDIF
       21F9                           ENDIF
       21F9                           ENDIF
       21F9 6C0B                      inc             11,x            ;se recupera el dato
       21FB                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       21FB E60C                      ldab            12,x
       21FD 37                        pshb                            ;se envía letra (fila)
       21FE E60B                      ldab            11,x
       2200 37                        pshb                            ;se envía num (columna)
       2201 1AEE0E                    ldy             14,x
       2204 183C                      pshy                            ;se envía inicio de array
       2206 E608                      ldab            8,x
       2208 37                        pshb                            ;se envía jugador atacado
       2209 BD2415                    jsr             play_wrt
       220C                           %CLRST          5
       0005           CONT_M          SET     5
       220C                           IF CONT_M > 0
       220C 31                        ins
       220D                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       220D                           IF CONT_M > 0
       220D 31                        ins
       220E                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       220E                           IF CONT_M > 0
       220E 31                        ins
       220F                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       220F                           IF CONT_M > 0
       220F 31                        ins
       2210                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2210                           IF CONT_M > 0
       2210 31                        ins
       2211                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2211                           ENDIF
       2211                           ENDIF
       2211                           ENDIF
       2211                           ENDIF
       2211                           ENDIF
       2211                           ENDIF
       2211                           %RESTORE
       2211 1838                      puly
       2213 38                        pulx
       2214 33                        pulb
       2215 32                        pula
       2216 39                        rts
                      
       2217           gp_shi_u        EQU             *
       2217 E60C                      ldab            12,x
       2219 C10A                      cmpb            #MIN_F
       221B 276D                      beq             gp_shi_d        ;si está en el borde superior, se continúa
       221D E60C                      ldab            12,x
       221F 5A                        decb                            ;se va a la posición de arriba
       2220 37                        pshb                            ;se envía la letra (fila)
       2221 E60B                      ldab            11,x
       2223 37                        pshb                            ;se envía el num (columna)
       2224 1AEE0E                    ldy             14,x
       2227 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       2229 E608                      ldab            8,x
       222B 37                        pshb                            ;se envía el jugador atacado
       222C BD23BE                    jsr             play_src        ;se busca la posición de arriba
       222F                           %CLRST          5
       0005           CONT_M          SET     5
       222F                           IF CONT_M > 0
       222F 31                        ins
       2230                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2230                           IF CONT_M > 0
       2230 31                        ins
       2231                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2231                           IF CONT_M > 0
       2231 31                        ins
       2232                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2232                           IF CONT_M > 0
       2232 31                        ins
       2233                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2233                           IF CONT_M > 0
       2233 31                        ins
       2234                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2234                           ENDIF
       2234                           ENDIF
       2234                           ENDIF
       2234                           ENDIF
       2234                           ENDIF
       2234                           ENDIF
       2234 2454                      bcc             gp_shi_d        ;si no había barco atacado arriba, se busca abajo
       2236                           %WRITEPOS       #SUNK_SHI       
       2236 C690                      ldab            #SUNK_SHI
       2238 37                        pshb                            ;se envía el byte a escribir
       2239 E60C                      ldab            12,x
       223B 37                        pshb                            ;se envía letra (fila)
       223C E60B                      ldab            11,x
       223E 37                        pshb                            ;se envía num (columna)
       223F 1AEE09                    ldy             9,x
       2242 183C                      pshy                            ;se envía dirección de inicio de matriz
       2244 E608                      ldab            8,x
       2246 37                        pshb                            ;se envía jugador atacado
       2247 BD238D                    jsr             writepos
       224A                           %CLRST          6
       0006           CONT_M          SET     6
       224A                           IF CONT_M > 0
       224A 31                        ins
       224B                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       224B                           IF CONT_M > 0
       224B 31                        ins
       224C                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       224C                           IF CONT_M > 0
       224C 31                        ins
       224D                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       224D                           IF CONT_M > 0
       224D 31                        ins
       224E                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       224E                           IF CONT_M > 0
       224E 31                        ins
       224F                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       224F                           IF CONT_M > 0
       224F 31                        ins
       2250                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2250                           ENDIF
       2250                           ENDIF
       2250                           ENDIF
       2250                           ENDIF
       2250                           ENDIF
       2250                           ENDIF
       2250                           ENDIF
       2250 6A0C                      dec             12,x
       2252                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       2252 C690                      ldab            #SUNK_SHI
       2254 37                        pshb                            ;se envía el byte a escribir
       2255 E60C                      ldab            12,x
       2257 37                        pshb                            ;se envía letra (fila)
       2258 E60B                      ldab            11,x
       225A 37                        pshb                            ;se envía num (columna)
       225B 1AEE09                    ldy             9,x
       225E 183C                      pshy                            ;se envía dirección de inicio de matriz
       2260 E608                      ldab            8,x
       2262 37                        pshb                            ;se envía jugador atacado
       2263 BD238D                    jsr             writepos
       2266                           %CLRST          6
       0006           CONT_M          SET     6
       2266                           IF CONT_M > 0
       2266 31                        ins
       2267                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2267                           IF CONT_M > 0
       2267 31                        ins
       2268                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2268                           IF CONT_M > 0
       2268 31                        ins
       2269                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2269                           IF CONT_M > 0
       2269 31                        ins
       226A                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       226A                           IF CONT_M > 0
       226A 31                        ins
       226B                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       226B                           IF CONT_M > 0
       226B 31                        ins
       226C                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       226C                           ENDIF
       226C                           ENDIF
       226C                           ENDIF
       226C                           ENDIF
       226C                           ENDIF
       226C                           ENDIF
       226C                           ENDIF
       226C 6C0C                      inc             12,x            ;se recupera el dato
       226E                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       226E E60C                      ldab            12,x
       2270 37                        pshb                            ;se envía letra (fila)
       2271 E60B                      ldab            11,x
       2273 37                        pshb                            ;se envía num (columna)
       2274 1AEE0E                    ldy             14,x
       2277 183C                      pshy                            ;se envía inicio de array
       2279 E608                      ldab            8,x
       227B 37                        pshb                            ;se envía jugador atacado
       227C BD2415                    jsr             play_wrt
       227F                           %CLRST          5
       0005           CONT_M          SET     5
       227F                           IF CONT_M > 0
       227F 31                        ins
       2280                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2280                           IF CONT_M > 0
       2280 31                        ins
       2281                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2281                           IF CONT_M > 0
       2281 31                        ins
       2282                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2282                           IF CONT_M > 0
       2282 31                        ins
       2283                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2283                           IF CONT_M > 0
       2283 31                        ins
       2284                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2284                           ENDIF
       2284                           ENDIF
       2284                           ENDIF
       2284                           ENDIF
       2284                           ENDIF
       2284                           ENDIF
       2284                           %RESTORE
       2284 1838                      puly
       2286 38                        pulx
       2287 33                        pulb
       2288 32                        pula
       2289 39                        rts
                      
       228A           gp_shi_d        EQU             *
       228A E60C                      ldab            12,x
       228C C10F                      cmpb            #MAX_F
       228E 276D                      beq             gp_shi_h        ;si está en el borde inferior, se continúa
       2290 E60C                      ldab            12,x
       2292 5C                        incb                            ;se va a la posición de abajo
       2293 37                        pshb                            ;se envía la letra (fila)
       2294 E60B                      ldab            11,x
       2296 37                        pshb                            ;se envía el num (columna)
       2297 1AEE0E                    ldy             14,x
       229A 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       229C E608                      ldab            8,x
       229E 37                        pshb                            ;se envía el jugador atacado
       229F BD23BE                    jsr             play_src        ;se busca la posición de abajo
       22A2                           %CLRST          5
       0005           CONT_M          SET     5
       22A2                           IF CONT_M > 0
       22A2 31                        ins
       22A3                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       22A3                           IF CONT_M > 0
       22A3 31                        ins
       22A4                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       22A4                           IF CONT_M > 0
       22A4 31                        ins
       22A5                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       22A5                           IF CONT_M > 0
       22A5 31                        ins
       22A6                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       22A6                           IF CONT_M > 0
       22A6 31                        ins
       22A7                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       22A7                           ENDIF
       22A7                           ENDIF
       22A7                           ENDIF
       22A7                           ENDIF
       22A7                           ENDIF
       22A7                           ENDIF
       22A7 2454                      bcc             gp_shi_h        ;si no había barco atacado abajo, el carguero está tocado
       22A9                           %WRITEPOS       #SUNK_SHI       
       22A9 C690                      ldab            #SUNK_SHI
       22AB 37                        pshb                            ;se envía el byte a escribir
       22AC E60C                      ldab            12,x
       22AE 37                        pshb                            ;se envía letra (fila)
       22AF E60B                      ldab            11,x
       22B1 37                        pshb                            ;se envía num (columna)
       22B2 1AEE09                    ldy             9,x
       22B5 183C                      pshy                            ;se envía dirección de inicio de matriz
       22B7 E608                      ldab            8,x
       22B9 37                        pshb                            ;se envía jugador atacado
       22BA BD238D                    jsr             writepos
       22BD                           %CLRST          6
       0006           CONT_M          SET     6
       22BD                           IF CONT_M > 0
       22BD 31                        ins
       22BE                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       22BE                           IF CONT_M > 0
       22BE 31                        ins
       22BF                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       22BF                           IF CONT_M > 0
       22BF 31                        ins
       22C0                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       22C0                           IF CONT_M > 0
       22C0 31                        ins
       22C1                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       22C1                           IF CONT_M > 0
       22C1 31                        ins
       22C2                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       22C2                           IF CONT_M > 0
       22C2 31                        ins
       22C3                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       22C3                           ENDIF
       22C3                           ENDIF
       22C3                           ENDIF
       22C3                           ENDIF
       22C3                           ENDIF
       22C3                           ENDIF
       22C3                           ENDIF
       22C3 6C0C                      inc             12,x
       22C5                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       22C5 C690                      ldab            #SUNK_SHI
       22C7 37                        pshb                            ;se envía el byte a escribir
       22C8 E60C                      ldab            12,x
       22CA 37                        pshb                            ;se envía letra (fila)
       22CB E60B                      ldab            11,x
       22CD 37                        pshb                            ;se envía num (columna)
       22CE 1AEE09                    ldy             9,x
       22D1 183C                      pshy                            ;se envía dirección de inicio de matriz
       22D3 E608                      ldab            8,x
       22D5 37                        pshb                            ;se envía jugador atacado
       22D6 BD238D                    jsr             writepos
       22D9                           %CLRST          6
       0006           CONT_M          SET     6
       22D9                           IF CONT_M > 0
       22D9 31                        ins
       22DA                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       22DA                           IF CONT_M > 0
       22DA 31                        ins
       22DB                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       22DB                           IF CONT_M > 0
       22DB 31                        ins
       22DC                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       22DC                           IF CONT_M > 0
       22DC 31                        ins
       22DD                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       22DD                           IF CONT_M > 0
       22DD 31                        ins
       22DE                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       22DE                           IF CONT_M > 0
       22DE 31                        ins
       22DF                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       22DF                           ENDIF
       22DF                           ENDIF
       22DF                           ENDIF
       22DF                           ENDIF
       22DF                           ENDIF
       22DF                           ENDIF
       22DF                           ENDIF
       22DF 6A0C                      dec             12,x            ;se recupera el dato
       22E1                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       22E1 E60C                      ldab            12,x
       22E3 37                        pshb                            ;se envía letra (fila)
       22E4 E60B                      ldab            11,x
       22E6 37                        pshb                            ;se envía num (columna)
       22E7 1AEE0E                    ldy             14,x
       22EA 183C                      pshy                            ;se envía inicio de array
       22EC E608                      ldab            8,x
       22EE 37                        pshb                            ;se envía jugador atacado
       22EF BD2415                    jsr             play_wrt
       22F2                           %CLRST          5
       0005           CONT_M          SET     5
       22F2                           IF CONT_M > 0
       22F2 31                        ins
       22F3                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       22F3                           IF CONT_M > 0
       22F3 31                        ins
       22F4                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       22F4                           IF CONT_M > 0
       22F4 31                        ins
       22F5                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       22F5                           IF CONT_M > 0
       22F5 31                        ins
       22F6                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       22F6                           IF CONT_M > 0
       22F6 31                        ins
       22F7                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       22F7                           ENDIF
       22F7                           ENDIF
       22F7                           ENDIF
       22F7                           ENDIF
       22F7                           ENDIF
       22F7                           ENDIF
       22F7                           %RESTORE
       22F7 1838                      puly
       22F9 38                        pulx
       22FA 33                        pulb
       22FB 32                        pula
       22FC 39                        rts
                      
       22FD           gp_shi_h        EQU             *
       22FD                           %WRITEPOS       #HIT            ;se marca al carguero como tocado
       22FD C6B0                      ldab            #HIT
       22FF 37                        pshb                            ;se envía el byte a escribir
       2300 E60C                      ldab            12,x
       2302 37                        pshb                            ;se envía letra (fila)
       2303 E60B                      ldab            11,x
       2305 37                        pshb                            ;se envía num (columna)
       2306 1AEE09                    ldy             9,x
       2309 183C                      pshy                            ;se envía dirección de inicio de matriz
       230B E608                      ldab            8,x
       230D 37                        pshb                            ;se envía jugador atacado
       230E BD238D                    jsr             writepos
       2311                           %CLRST          6
       0006           CONT_M          SET     6
       2311                           IF CONT_M > 0
       2311 31                        ins
       2312                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2312                           IF CONT_M > 0
       2312 31                        ins
       2313                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2313                           IF CONT_M > 0
       2313 31                        ins
       2314                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2314                           IF CONT_M > 0
       2314 31                        ins
       2315                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2315                           IF CONT_M > 0
       2315 31                        ins
       2316                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2316                           IF CONT_M > 0
       2316 31                        ins
       2317                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2317                           ENDIF
       2317                           ENDIF
       2317                           ENDIF
       2317                           ENDIF
       2317                           ENDIF
       2317                           ENDIF
       2317                           ENDIF
       2317                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       2317 E60C                      ldab            12,x
       2319 37                        pshb                            ;se envía letra (fila)
       231A E60B                      ldab            11,x
       231C 37                        pshb                            ;se envía num (columna)
       231D 1AEE0E                    ldy             14,x
       2320 183C                      pshy                            ;se envía inicio de array
       2322 E608                      ldab            8,x
       2324 37                        pshb                            ;se envía jugador atacado
       2325 BD2415                    jsr             play_wrt
       2328                           %CLRST          5
       0005           CONT_M          SET     5
       2328                           IF CONT_M > 0
       2328 31                        ins
       2329                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2329                           IF CONT_M > 0
       2329 31                        ins
       232A                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       232A                           IF CONT_M > 0
       232A 31                        ins
       232B                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       232B                           IF CONT_M > 0
       232B 31                        ins
       232C                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       232C                           IF CONT_M > 0
       232C 31                        ins
       232D                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       232D                           ENDIF
       232D                           ENDIF
       232D                           ENDIF
       232D                           ENDIF
       232D                           ENDIF
       232D                           ENDIF
       232D                           %RESTORE
       232D 1838                      puly
       232F 38                        pulx
       2330 33                        pulb
       2331 32                        pula
       2332 39                        rts
                      
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
                      
       2333           mirror          EQU             *       
                              
       2333                           %BACKUP
       2333 36                        psha
       2334 37                        pshb
       2335 3C                        pshx
       2336 183C                      pshy
       2338 30                        tsx                             ;creo frame pointer
       2339 A60B                      ldaa            11,x            ;cargo número de jugador
       233B E608                      ldab            8,x             ;cargo longitud de la copia (del array a copiar)
       233D 3D                        mul                             ;obtengo offset de jugador
       233E E30C                      addd            12,x            ;sumo offset de jugador
       2340 ED0C                      std             12,x            ;guardo la dirección con offset del array fuente
                      
       2342 A60C                      ldaa            12,x            ;cargo longitud de copia en contador
       2344 2710                      beq             mi_end          ;si es cero, vuelvo desde subrutina
       2346 1AEE09                    ldy             9,x             ;cargo índice a destino de la copia.
       2349 EE0C                      ldx             12,x            ;cargo índice a array origen de copia.
                      
       234B           mi_loop         EQU             *
                      
       234B E600                      ldab            0,x             ;cargo byte a copiar
       234D 18E700                    stab            0,y             ;copio byte en destino
       2350 08                        inx                             ;siguiente byte
       2351 1808                      iny                             ;siguiente byte
       2353 4A                        deca                            ;decremento contador
       2354 26F5                      bne             mi_loop
                      
       2356           mi_end          EQU             *
                      
       2356                           %RESTORE
       2356 1838                      puly
       2358 38                        pulx
       2359 33                        pulb
       235A 32                        pula
                              
       235B 39                        rts     
                                      
                                                      
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
                      
       235C           fetchpos        EQU             *
                      
       235C                           %BACKUP
       235C 36                        psha
       235D 37                        pshb
       235E 3C                        pshx
       235F 183C                      pshy
       2361 30                        tsx                             ;creo frame pointer
       2362 A608                      ldaa            8,x             ;cargo numero de jugador
       2364 C624                      ldab            #ARRAY_L        ;cargo longitud del arreglo (MAX 255 bytes)
       2366 3D                        mul                             ;obtengo offset de jugador
       2367 E309                      addd            9,x             ;sumo offset de jugador a top of array
       2369 ED09                      std             9,x             ;obtengo direccion al arreglo del jugador
                      
       236B A60C                      ldaa            12,x            ;cargo coordenada letra (FILA)
       236D C60A                      ldab            #NORM           ;cargo valor de normalización de filas
       236F 10                        sba                             ;obtengo numero de fila
       2370 C606                      ldab            #COL            ;cargo longitud de fila
       2372 3D                        mul                             ;obtengo offset de fila
       2373 E309                      addd            9,x             ;sumo offset de fila a la dirección del arreglo del jugador
       2375 ED09                      std             9,x             ;obtengo dirección a la fila que se desea
                      
       2377 E60B                      ldab            11,x            ;cargo coordenada (offset) de columna
       2379 5A                        decb                            ;normalizo offset de columna
       237A 4F                        clra                            ;borro AccA para tener el valor de AccB en AccD
       237B E309                      addd            9,x             ;sumo offset de columna a la dirección temporal
       237D ED09                      std             9,x             ;guardo en stack la dirección del byte a consultar
                      
       237F 1AEE09                    ldy             9,x             ;cargo en IY la dirección del byte que se desea obtener
       2382 18A600                    ldaa            0,y             ;cargo el byte a devolver en AccA
       2385 A70C                      staa            12,x            ;guardo el byte a devolver en stack
       2387                           %RESTORE
       2387 1838                      puly
       2389 38                        pulx
       238A 33                        pulb
       238B 32                        pula
                                      
       238C 39                        rts
                                      
                                      
                      
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
                      
       238D           writepos        EQU             *
                      
       238D                           %BACKUP
       238D 36                        psha
       238E 37                        pshb
       238F 3C                        pshx
       2390 183C                      pshy
       2392 30                        tsx                             ;creo frame pointer
       2393 A608                      ldaa            8,x             ;cargo numero de jugador
       2395 C624                      ldab            #ARRAY_L        ;cargo longitud del arreglo (MAX 255 bytes)
       2397 3D                        mul                             ;obtengo offset de jugador
       2398 E309                      addd            9,x             ;sumo offset de jugador a top of array
       239A ED09                      std             9,x             ;obtengo direccion al arreglo del jugador
                      
       239C A60C                      ldaa            12,x            ;cargo coordenada letra (FILA)
       239E C60A                      ldab            #NORM           ;cargo valor de normalización de filas
       23A0 10                        sba                             ;obtengo numero de fila
       23A1 C606                      ldab            #COL            ;cargo longitud de fila
       23A3 3D                        mul                             ;obtengo offset de fila
       23A4 E309                      addd            9,x             ;sumo offset de fila a la dirección del arreglo del jugador
       23A6 ED09                      std             9,x             ;obtengo dirección a la fila que se desea
                      
       23A8 E60B                      ldab            11,x            ;cargo coordenada (offset) de columna
       23AA 5A                        decb                            ;normalizo offset de columna
       23AB 4F                        clra                            ;borro AccA para tener el valor de AccB en AccD
       23AC E309                      addd            9,x             ;sumo offset de columna a la dirección temporal
       23AE ED09                      std             9,x             ;guardo en stack la dirección del byte a consultar
                      
       23B0 1AEE09                    ldy             9,x             ;cargo en IY la dirección donde se desea escribir
       23B3 A60D                      ldaa            13,x            ;cargo en AccA el byte que se quiere escribir en el arreglo
       23B5 18A700                    staa            0,y             ;escribo el byte sobre el arreglo
       23B8                           %RESTORE
       23B8 1838                      puly
       23BA 38                        pulx
       23BB 33                        pulb
       23BC 32                        pula
                                      
       23BD 39                        rts
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
                      
       23BE           play_src        EQU             *               
                                              
       23BE                           %BACKUP
       23BE 36                        psha
       23BF 37                        pshb
       23C0 3C                        pshx
       23C1 183C                      pshy
       23C3 30                        tsx                             ;creo frame pointer
       23C4 A608                      ldaa            8,x             ;cargo número de array (jugador)
       23C6 C613                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       23C8 3D                        mul                             ;obtengo offset de jugador
       23C9 E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       23CB 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
                      
       23CD           ps_loop         EQU             *
                      
       23CD 18A600                    ldaa            0,y             ;cargo byte1 a comparar
       23D0 2718                      beq             ps_ntfnd        ;si encontré el terminador, el valor requerido no está en el array
                      
       23D2 A10C                      cmpa            12,x            ;comparo con el byte1 a buscar
       23D4 2706                      beq             ps_next         ;si coinciden, busco el byte 2
       23D6 1808                      iny                             ;me posiciono sobre siguiente byte1 a comparar
       23D8 1808                      iny
       23DA 20F1                      bra             ps_loop         ;vuelvo a comparar
                      
       23DC           ps_next         EQU             *
                      
       23DC 18E601                    ldab            1,y             ;cargo byte a continuación del anterior
       23DF 1808                      iny                             ;me posiciono sobre siguiente byte1 a comparar
       23E1 1808                      iny
       23E3 E10B                      cmpb            11,x            ;comparo con byte2
       23E5 26E6                      bne             ps_loop         ;si no son iguales, sigo buscando
       23E7 0D                        sec                             ;set del carry para informar que se encontró el valor
       23E8 2001                      bra             ps_end
                                              
       23EA           ps_ntfnd        EQU             *
                      
       23EA 0C                        clc                             ;carry en 0 para informar que no se encontró el valor
                      
       23EB           ps_end          EQU             *
                                      
       23EB                           %RESTORE
       23EB 1838                      puly
       23ED 38                        pulx
       23EE 33                        pulb
       23EF 32                        pula
                      
       23F0 39                        rts
                      
                      
                      
                                              
                      
                                              
                      
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
                      
       23F1           play_cnt        EQU             *
                      
       23F1                           %BACKUP
       23F1 36                        psha
       23F2 37                        pshb
       23F3 3C                        pshx
       23F4 183C                      pshy
       23F6 30                        tsx                             ;creo frame pointer
       23F7 A608                      ldaa            8,x             ;cargo número de array (jugador)
       23F9 C613                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       23FB 3D                        mul                             ;obtengo offset de jugador
       23FC E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       23FE 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
       2400 4F                        clra                            ;inicializo contador
                                              
       2401           pc_loop         EQU             *               
                                              
       2401 18E600                    ldab            0,y             ;cargo byte a comparar
       2404 2707                      beq             pc_end          ;si es terminador, terminé de buscar
       2406 1808                      iny                             ;si no es terminador, me posiciono sobre siguiente dato de 
       2408 1808                      iny                             ;2 bytes
       240A 4C                        inca                            ;incremento contador
       240B 20F4                      bra             pc_loop         ;vuelvo a comparar
                                              
       240D           pc_end          EQU             *       
                                              
       240D A70A                      staa            10,x            ;devuelvo valor por stack
       240F                           %RESTORE
       240F 1838                      puly
       2411 38                        pulx
       2412 33                        pulb
       2413 32                        pula
                                      
       2414 39                        rts
                      
                      
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
                      
       2415           play_wrt        EQU             *
                      
       2415                           %BACKUP
       2415 36                        psha
       2416 37                        pshb
       2417 3C                        pshx
       2418 183C                      pshy
       241A 30                        tsx                             ;creo frame pointer
       241B A608                      ldaa            8,x             ;cargo número de array (jugador)
       241D C613                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       241F 3D                        mul                             ;obtengo offset de jugador
       2420 E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       2422 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
                      
                      
       2424           pw_loop         EQU             *
                      
       2424 18A600                    ldaa            0,y             ;cargo byte del array
       2427 2704                      beq             pw_end          ;si es cero, entonces escribo los datos
       2429 1808                      iny                             ;si no es cero, me posiciono sobre la siguiente posición
       242B 20F7                      bra             pw_loop         ;vuelvo a buscar ceros
                      
       242D           pw_end          EQU             *
                      
       242D A60C                      ldaa            12,x            ;cargo primer byte a escribir
       242F 18A700                    staa            0,y             ;guardo primer byte en memoria
       2432 1808                      iny                             ;me posiciono en el siguiente byte
       2434 A60B                      ldaa            11,x            ;cargo segundo byte a escribir
       2436 18A700                    staa            0,y             ;guardo segundo byte en memoria
       2439                           %RESTORE
       2439 1838                      puly
       243B 38                        pulx
       243C 33                        pulb
       243D 32                        pula
                      
       243E 39                        rts
                      
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
       243F           clrzone         EQU     *
                      
       243F                           %BACKUP                         ;backup de registros en stack
       243F 36                        psha
       2440 37                        pshb
       2441 3C                        pshx
       2442 183C                      pshy
                      
       2444 30                        tsx
       2445 A608                      ldaa    8,x                     ;cargo contador con longitud de zona a borrar
       2447 EE09                      ldx     9,x                     ;cargo puntero zona de memoria a borrar
       2449 D600                      ldab    $00                     ;cargo el cero
       244B           clr_loop        EQU     *
       244B E700                      stab    0,x                     ;guardo caracter en posicion del array
       244D 08                        inx                             ;siguiente posición     
       244E 4A                        deca                            ;decremento contador
       244F 26FA                      bne     clr_loop                ;si no es cero, escribo en la posición siguiente.
                      
       2451                           %RESTORE                        :restauro backup en registros
       2451 1838                      puly
       2453 38                        pulx
       2454 33                        pulb
       2455 32                        pula
                      
       2456 39                        rts
                      
                      *---------------------------------------
                      * WhoPlays: subrutina para ver que 
                      *           jugador juega
                      *---------------------------------------
                      
       2457           whoplays        EQU             *
                      
       2457 BD251C    wait_ply        jsr             read_kb
       245A 4D                        tsta
       245B 2BFA                      bmi             wait_ply
       245D 8103                      cmpa            #B_P1
       245F 2708                      beq             play_p1
       2461 8107                      cmpa            #B_P2
       2463 26F2                      bne             wait_ply
       2465 8600                      ldaa            #P1
       2467 2002                      bra             ply_end
                      
       2469 8601      play_p1         ldaa            #P2
       246B 39        ply_end         rts             
                      
                      *---------------------------------------
                      * Input: subrutina que devuelve
                      *        entrada para la batalla naval
                      * AccA: Fila de la posicion jugada
                      * AccB: Columna de la posicion jugada
                      *---------------------------------------
                      
       246C           input           EQU             *
                      
                      * back up de registros
       246C 3C                        pshx
       246D 183C                      pshy
                      
                      * creo variables locales
       246F 36                        psha
       2470 36                        psha
       2471 36                        psha
                      
                      * inicializo framepointer
       2472 1830                      tsy
                      
                      * prendo el led
       2474                           %LED_INV
       2474 B61000                    ldaa            PORTA
       2477 8840                      eora            #SHF_LED
       2479 B71000                    staa            PORTA
                      
                      * inicializo el contador
       247C 8603      inp_rest        ldaa            #INPUTS
       247E 18A700                    staa            CONT,Y
                                      
                      * limpio el display
       2481 CE41F9                    ldx             #string
       2484 8604                      ldaa            #4
       2486 C620                      ldab            #ESP
       2488 BD25E6                    jsr             clrarray
                      
       248B BD25D5                    jsr             clr_dis
                      
       248E 86FF                      ldaa            #$FF
       2490 B741F8                    staa            buffer
                      
                      * lectura de teclado
       2493 BD251C    wait_kb         jsr             read_kb
       2496 4D                        tsta
       2497 2BFA                      bmi             wait_kb
       2499 B141F8                    cmpa            buffer
       249C 27F5                      beq             wait_kb
       249E B741F8                    staa            buffer
                                      
       24A1 810F                      cmpa            #B_reset
       24A3 27D7                      beq             inp_rest
       24A5 8103                      cmpa            #B_P1
       24A7 27D3                      beq             inp_rest
       24A9 8107                      cmpa            #B_P2
       24AB 27CF                      beq             inp_rest
                      
       24AD 186A00                    dec             CONT,Y
       24B0 2727                      beq             chk_fire
       24B2 36                        psha
       24B3 CE41FE                    ldx             #kb_value
       24B6 16                        tab
       24B7 3A                        abx
       24B8 A600                      ldaa            0,X             
       24BA 183C                      pshy
       24BC 18E600                    ldab            CONT,Y
       24BF 183A                      aby
       24C1 18A700                    staa            0,Y
       24C4 1838                      puly
                      
       24C6 33                        pulb
       24C7 BD25C5                    jsr             to_ascii
       24CA 17                        tba
                                      
       24CB CE41F9                    ldx             #string
       24CE BD25B5                    jsr             sh_value
                      
       24D1 CE41F9                    ldx             #string
       24D4 BD2508                    jsr             show
                      
       24D7 20BA                      bra             wait_kb
                      
       24D9 810B      chk_fire        cmpa            #B_fire
       24DB 269F                      bne             inp_rest
       24DD 18A602                    ldaa            FILA_L,Y
       24E0 810A                      cmpa            #MIN_FIL
       24E2 2598                      blo             inp_rest
       24E4 810F                      cmpa            #MAX_FIL
       24E6 2294                      bhi             inp_rest
       24E8 18A601                    ldaa            COL_L,Y
       24EB 8101                      cmpa            #MIN_COL
       24ED 258D                      blo             inp_rest
       24EF 8106                      cmpa            #MAX_COL
       24F1 2289                      bhi             inp_rest
                      
                      * apago el led
       24F3                           %LED_INV
       24F3 B61000                    ldaa            PORTA
       24F6 8840                      eora            #SHF_LED
       24F8 B71000                    staa            PORTA
                      
                      * libero variables locales
       24FB 32                        pula
       24FC 32                        pula
       24FD 32                        pula
                      
       24FE 18A602    return          ldaa            FILA_L,Y
       2501 18E601                    ldab            COL_L,Y         
                      
                      * restore de registros
       2504 1838                      puly
       2506 38                        pulx
                      
       2507 39                        rts
                      
                      *------------------------------------
                      * Show: Por IX direccion de string de
                      *       4 caracteres para imprimir
                      *------------------------------------
                      
       2508           show            EQU             *
                      
                      * back up de registros
       2508 3C                        pshx
       2509 37                        pshb
       250A 36                        psha
                      
                      * inicializo contador de posiciones
       250B C604                      ldab            #LENGTH
                                      
                      * loop para enviar a cada posicion
       250D 5A        sh_loop         decb
       250E 2B08                      bmi             sh_end          ; si no pase por todas las posiciones
       2510 A600                      ldaa            0,X             ; tomo del string el caracter
       2512 BD25CE                    jsr             outchar         ; y lo imprimo en la posicion actual
       2515 08                        inx
       2516 20F5                      bra             sh_loop
                      
                      * restore de registros                          
       2518 32        sh_end          pula
       2519 33                        pulb
       251A 38                        pulx
       251B 39                        rts
                      
                      *----------------------------------------------
                      * read_kb: AccA: tecla presionada
                      *          o -1
                      *----------------------------------------------
                      
       251C           read_kb         EQU             *
                      
                      * back up de registros
       251C 3C                        pshx
       251D 37                        pshb
                      
                      * creo espacio para variables locales
       251E                           %MALLOC         1               ; variable mascara de filas
       251E                           IF 1 > 0
       251E 30                        tsx
       251F 8F                        xgdx
       2520 830001                    subd    #1
       2523 8F                        xgdx
       2524 35                        txs
       2525                           ENDIF
       2525                           %MALLOC         1               ; variable mascara de columnas
       2525                           IF 1 > 0
       2525 30                        tsx
       2526 8F                        xgdx
       2527 830001                    subd    #1
       252A 8F                        xgdx
       252B 35                        txs
       252C                           ENDIF
       252C                           %MALLOC         1               ; variable contador de columnas
       252C                           IF 1 > 0
       252C 30                        tsx
       252D 8F                        xgdx
       252E 830001                    subd    #1
       2531 8F                        xgdx
       2532 35                        txs
       2533                           ENDIF
       2533                           %MALLOC         1               ; variable contador de tecla
       2533                           IF 1 > 0
       2533 30                        tsx
       2534 8F                        xgdx
       2535 830001                    subd    #1
       2538 8F                        xgdx
       2539 35                        txs
       253A                           ENDIF
                      
                      * inicializacion de variables
       253A 30                        tsx                             ; framepointer para acceder a locales
       253B C604                      ldab            #LENGTH         ; contador de fila
       253D 860E                      ldaa            #FIL0           ; mascara para la fila cero
       253F A703                      staa            MASK_FIL,X      
       2541 8610                      ldaa            #COL0           ; mascara para la columna cero
       2543 A702                      staa            MASK_COL,X
       2545 4F                        clra                            ; contador de columna
       2546 A701                      staa            COL_CONT,X
       2548 A700                      staa            KB_PRESS,X      ; variable con numero de tecla
                      
                      * loop para barrer y mirar teclado matricial
       254A 86FF      kb_loop         ldaa            #NONE
       254C 5A                        decb                            
       254D 2B2C                      bmi             kb_end          ; me fijo si recorrio todas las filas
       254F A603                      ldaa            MASK_FIL,X      ; enmascaro la siguiente fila
       2551 B71003                    staa            ROWS
       2554 0D                        sec                             ; carry para rotacion de byte
       2555 49                        rola                            
       2556 A703                      staa            MASK_FIL,X      ; cambio variable mascara para la proxima fila
                      
       2558 8604                      ldaa            #LENGTH
       255A A701                      staa            COL_CONT,X
       255C 8610                      ldaa            #COL0           ; inicializo variable para barrer
       255E A702                      staa            MASK_COL,X      ; la columna
                      
       2560 6A01      kb_col          dec             COL_CONT,X      ; me fijo si barrio todas las columnas
       2562 2BE6                      bmi             kb_loop
       2564 B61005                    ldaa            COLS            ; leo el puerto de columnas
       2567 A402                      anda            MASK_COL,X      ; enmascaro columna
       2569 2707                      beq             kb_hit          ; si es nulo apretaron un boton
       256B 48                        lsla                            ; cambio mascara para proxima columna
       256C A702                      staa            MASK_COL,X
       256E 6C00                      inc             KB_PRESS,X      ; incremento numero de tecla
       2570 20EE                      bra             kb_col
                      
       2572 A600      kb_hit          ldaa            KB_PRESS,X      ; apretaron una tecla, lo comparo
       2574 B141F7                    cmpa            kb_buff         ; con la que fue apretada antes
       2577 2602                      bne             kb_end          ; aviso por led
                      
       2579 86FF      kb_none         ldaa            #NONE           ; en caso de que no apreten nada
                      
       257B B741F7    kb_end          staa            kb_buff         ; actualizo variable
                                              
                      * elimino variables locales
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
       258C                           %RELMEM         1
       258C                           IF 1 > 0
       258C 30                        tsx
       258D 8F                        xgdx
       258E C30001                    addd    #1
       2591 8F                        xgdx
       2592 35                        txs
       2593                           ENDIF           
       2593                           %RELMEM         1
       2593                           IF 1 > 0
       2593 30                        tsx
       2594 8F                        xgdx
       2595 C30001                    addd    #1
       2598 8F                        xgdx
       2599 35                        txs
       259A                           ENDIF           
                      
                      * restore de registros
       259A 33                        pulb
       259B 38                        pulx
                                      
       259C 39                        rts
                      
                      *----------------------------------------------
                      * init_kb: subrutina para inicializar teclado
                      *----------------------------------------------
                      
       259D           init_kb         EQU             *
                      
                      * back up de registros
       259D 36                        psha
                      
                      * inicializacion de teclado             
       259E 86FF                      ldaa            #NONE
       25A0 B741F7                    staa            kb_buff
                      
                      * restore de registros
       25A3 32                        pula
       25A4 39                        rts
                      
                      *--------------------------------
                      * Inicializa display (y teclado)
                      *--------------------------------
                      
       25A5           initdis         EQU             *
                              
       25A5 36                        psha
                      
       25A6 860F                      ldaa            #$0F       ;set for input/output
       25A8 B71007                    staa            DDRC       ;C output
       25AB B61002                    ldaa            PIOC       ;get current value
       25AE 84FE                      anda            #$FE       ;reset bit 0 to 0
       25B0 B71002                    staa            PIOC       ;send it out
                              
       25B3 32                        pula
       25B4 39                        rts
                      
                      *--------------------------------------------------------------
                      * sh_value: toma un string, desplaza los caracteres a izquierda
                      *           y agrega un nuevo caracter
                      *           IX: direccion del string
                      *           AccA: nuevo valor
                      *--------------------------------------------------------------
                      
       25B5           sh_value        EQU             *
                      
                      * back up de registros
       25B5 3C                        pshx
       25B6 37                        pshb
                      
                      * loop para desplazar caracteres
       25B7 E601      v_loop          ldab            1,X             ;tomo elemento siguiente en string
       25B9 2705                      beq             v_end
       25BB E700                      stab            0,X             ;y lo guardo en posicion actual
       25BD 08                        inx
       25BE 20F7                      bra             v_loop          
                      
       25C0 A700      v_end           staa            0,X             ;ingresa nuevo caracter en cadena desplazada
                      
                      * restore de registros
       25C2 33                        pulb
       25C3 38                        pulx
       25C4 39                        rts
                      
                      *----------------------------------------------
                      * toAscii: toma numero de boton del teclado
                      *          y devuelve su correspondiente en
                      *          ascii
                      *          toma  AccB: valor del boton
                      *          devuelve AccB: ascii
                      *----------------------------------------------
                      
       25C5           to_ascii        EQU             *
                      
                      * back up de registros
       25C5 3C                        pshx
                      
                      * inicializacion de variables
       25C6 CE420D                    ldx             #kb_ascii       ; apunto a string con caracteres asignados por tecla
       25C9 3A                        abx                             ; me muevo en string con indice y recupero segun tecla presionada
       25CA E600                      ldab            0,X             
                      
                      * restore de registros
       25CC 38                        pulx
       25CD 39                        rts
                      
                      *--------------------------------
                      *Outchar: ACCA: Carácter a enviar
                      *           ACCB: Posición (0-3)
                      *--------------------------------
       25CE           outchar         EQU             *
                              
       25CE F71005                    stab            PORTCL          ;set the position 
       25D1 B71004                    staa            PORTB           ;set the character
                              
       25D4 39                        rts
                      
                      *-------------------------
                      * clr_dis: Limpia display
                      *-------------------------
                      
       25D5           clr_dis         EQU             *
                      
                      * hago back up de registros
       25D5 37                        pshb
       25D6 36                        psha
                      
                      * inicializo el contador de posiciones
       25D7 C604                      ldab            #LENGTH
                      
                      * loop para limpiar cada posicion
       25D9 5A        clrloop         decb
       25DA 2B07                      bmi             clr_end         ; si no pase por todas las posiciones
       25DC 8620                      ldaa            #ESP            ; cargo en esta posicion espacio blanco
       25DE BD25CE                    jsr             outchar         ; y lo envio
       25E1 20F6                      bra             clrloop         
                      
                      * restore de registros
       25E3 32        clr_end         pula
       25E4 33                        pulb
       25E5 39                        rts
                      
                      *-------------------------------------------------------
                      * clrarray: subrutina para limpiar un array dado
                      * parametros: IX direccion del array
                      *             AccA cantidad de elementos del array
                      *             AccB caracter a poner
                      *-------------------------------------------------------
                      
       25E6           clrarray        EQU             *
                                      
       25E6 4A        cra_loop        deca                            ;contador con cantidad de elementos a limpiar
       25E7 2705                      beq             cra_end
       25E9 E700                      stab            0,X             ;reemplazar elemento por caracter a poner
       25EB 08                        inx
       25EC 20F8                      bra             cra_loop
                                      
       25EE E700      cra_end         stab            0,X
       25F0 39                        rts
                      
                      *=============================================
                      *Subrutina winner: Recibe por stack un 1 si
                      *gano el player 1 y un 0 si gano el player 2.
                      *=============================================
                      
                                      
       25F1           winner          equ     *
       25F1                           %BACKUP
       25F1 36                        psha
       25F2 37                        pshb
       25F3 3C                        pshx
       25F4 183C                      pshy
       25F6 30                        tsx
       25F7 A608                      ldaa    8,x     ;recupero parametro enviado por stack
       25F9 2708                      beq     win2
                      
       25FB           win1            equ     *
       25FB CE4242                    ldx     #string1
       25FE BD2508                    jsr     show
       2601 2006                      bra     end
                      
       2603           win2            equ     *
       2603 CE4247                    ldx     #string2
       2606 BD2508                    jsr     show
                              
       2609           end             equ     *
       2609                           %RESTORE
       2609 1838                      puly
       260B 38                        pulx
       260C 33                        pulb
       260D 32                        pula
       260E 39                        rts
                      
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
                      
                      ***END VARIABLES***
       4242 57494E31  string1         fcc     'WIN1'
       4246 00                        fcb     00
                      
       4247 57494E32  string2         fcc     'WIN2'
       424B 00                        fcb     00
                      
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
     ytes:   1704         # main_bn #
     RC:     7EF2         ###########
 