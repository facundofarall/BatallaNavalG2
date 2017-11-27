################################################################################
#                                                                              #
#    Micro Series 6801 Assembler V2.00/DOS              27/Nov/17  00:19:34    #
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
       2003 BD2583                    jsr             initdis         ;inicialización de display 7 seg
       2006 BD257B                    jsr             init_kb         ;inicialización de teclado
       2009 CE421C                    ldx             #hitlist
       200C 3C                        pshx                            ;se envía inicio de arreglo de barcos a poner en 0
       200D 8626                      ldaa            #(HITSIZE*2)
       200F 36                        psha                            ;se envía el tamaño del arreglo a limpiar
       2010 BD2425                    jsr             clrzone
       2013 8601                      ldaa            #P2             ;se inicia atacando al jugador 2
       2015 B741F4                    staa            player          ;se inicia con el jugador 1 (atacando al 2)
                      
       2018           main            EQU             *
       2018 BD243D                    jsr             whoplays        ;se recibirá en AccA el jugador a atacar a continuación
       201B B741F4                    staa            player          ;se habilita la siguiente jugada
                      
       201E CE5000                    ldx             #p_board
       2021 3C                        pshx                            ;se envía el inicio de las matrices de tableros
       2022 B641F4                    ldaa            player
       2025 36                        psha                            ;se envía el jugador atacado
       2026 CE5F00                    ldx             #display
       2029 3C                        pshx                            ;se envía la matriz de display (destino a copiar)
       202A 8624                      ldaa            #ARRAY_L
       202C 36                        psha                            ;se envía el tamaño de la matriz tablero
       202D BD2319                    jsr             mirror          ;se envía el tablero del jugador atacado al display
                      
       2030 BD2452                    jsr             input           ;se llama a la subrutina de interfaz de usuario
       2033 B741F5                    staa            letra_p         ;se recibe en AccA la letra jugada (fila) y en AccB el número (columna)
       2036 F741F6                    stab            num_p           ;se guarda la jugada
                      
       2039 CE421C                    ldx             #hitlist
       203C 3C                        pshx                            ;se envía la posición del primer arreglo de barcos tocados
       203D B641F5                    ldaa            letra_p
       2040 36                        psha                            ;se envía la letra jugada
       2041 F641F6                    ldab            num_p
       2044 37                        pshb                            ;se envía el número jugado
       2045 B641F4                    ldaa            player
       2048 36                        psha                            ;se envía el jugador a atacar
       2049 BD23A4                    jsr             play_src        ;se recibirá el carry en 1 si la posición corresponde a un barco ya atacado
       204C 25CA                      bcs             main            ;si la posición ya fue jugada, se pierde el turno y pasa al siguiente
       204E                           %CLRST          5               ;se limpia el stack
       0005           CONT_M          SET     5
       204E                           IF CONT_M > 0
       204E 31                        ins
       204F                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       204F                           IF CONT_M > 0
       204F 31                        ins
       2050                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2050                           IF CONT_M > 0
       2050 31                        ins
       2051                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2051                           IF CONT_M > 0
       2051 31                        ins
       2052                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2052                           IF CONT_M > 0
       2052 31                        ins
       2053                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2053                           ENDIF
       2053                           ENDIF
       2053                           ENDIF
       2053                           ENDIF
       2053                           ENDIF
       2053                           ENDIF
                                      
       2053 B641F5                    ldaa            letra_p
       2056 36                        psha                            ;se envía le letra jugada
       2057 F641F6                    ldab            num_p
       205A 37                        pshb                            ;se envía el número jugado
       205B CE5000                    ldx             #p_board
       205E 3C                        pshx                            ;se envía la dirección de inicio de los arreglos de tablero
       205F F641F4                    ldab            player
       2062 37                        pshb                            ;se envía el jugador a atacar
       2063 BD2342                    jsr             fetchpos
       2066                           %CLRST          4               ;se limpia el stack
       0004           CONT_M          SET     4
       2066                           IF CONT_M > 0
       2066 31                        ins
       2067                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2067                           IF CONT_M > 0
       2067 31                        ins
       2068                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2068                           IF CONT_M > 0
       2068 31                        ins
       2069                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2069                           IF CONT_M > 0
       2069 31                        ins
       206A                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       206A                           ENDIF
       206A                           ENDIF
       206A                           ENDIF
       206A                           ENDIF
       206A                           ENDIF
       206A 32                        pula                            ;se toma del stack el dato del byte jugado
                      
       206B CE421C                    ldx             #hitlist
       206E 3C                        pshx                            ;se envía inicio del arreglo de barcos atacados
       206F 36                        psha                            ;se envía el byte a jugado
       2070 B641F5                    ldaa            letra_p
       2073 36                        psha                            ;se envía le letra jugada
       2074 F641F6                    ldab            num_p
       2077 37                        pshb                            ;se envía el número jugado
       2078 CE5000                    ldx             #p_board
       207B 3C                        pshx                            ;se envía la dirección de inicio de los arreglos de tablero
       207C F641F4                    ldab            player
       207F 37                        pshb                            ;se envía el jugador a atacar
       2080 BD20B1                    jsr             gameplay
       2083                           %CLRST          8               ;se limpia el stack
       0008           CONT_M          SET     8
       2083                           IF CONT_M > 0
       2083 31                        ins
       2084                           %CLRST  CONT_M-1
       0007           CONT_M          SET     CONT_M-1
       2084                           IF CONT_M > 0
       2084 31                        ins
       2085                           %CLRST  CONT_M-1
       0006           CONT_M          SET     CONT_M-1
       2085                           IF CONT_M > 0
       2085 31                        ins
       2086                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2086                           IF CONT_M > 0
       2086 31                        ins
       2087                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2087                           IF CONT_M > 0
       2087 31                        ins
       2088                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2088                           IF CONT_M > 0
       2088 31                        ins
       2089                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2089                           IF CONT_M > 0
       2089 31                        ins
       208A                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       208A                           IF CONT_M > 0
       208A 31                        ins
       208B                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       208B                           ENDIF
       208B                           ENDIF
       208B                           ENDIF
       208B                           ENDIF
       208B                           ENDIF
       208B                           ENDIF
       208B                           ENDIF
       208B                           ENDIF
       208B                           ENDIF
                      
       208B CE5000                    ldx             #p_board
       208E 3C                        pshx                            ;se envía el inicio de las matrices de tableros
       208F B641F4                    ldaa            player
       2092 36                        psha                            ;se envía el jugador atacado
       2093 CE5F00                    ldx             #display
       2096 3C                        pshx                            ;se envía la matriz de display (destino a copiar)
       2097 8624                      ldaa            #ARRAY_L
       2099 36                        psha                            ;se envía el tamaño de la matriz tablero
       209A BD2319                    jsr             mirror          ;se envía el tablero del jugador atacado al display
                                      
       209D CE421C                    ldx             #hitlist
       20A0 3C                        pshx                            ;se envía la posición del primer arreglo de barcos tocados
       20A1 B641F4                    ldaa            player
       20A4 36                        psha                            ;se envía el jugador a atacar
       20A5 BD23D7                    jsr             play_cnt        
       20A8 2503                      bcs             end_game        ;si se recibe el Carry en 1, el jugador de turno ganó
                      
       20AA 7E2018                    jmp             main
                      
       20AD           end_game        EQU             *
                      *               ldaa            player
                      *               psha
                      *               jsr             winner
       20AD 20FE                      bra             end_game
                      
       20AF           end_loop        EQU             *
       20AF 20FE                      bra             end_loop
                      
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
       20B1           gameplay        EQU             *
       20B1                           %BACKUP
       20B1 36                        psha
       20B2 37                        pshb
       20B3 3C                        pshx
       20B4 183C                      pshy
                                      
       20B6 30                        tsx                             ;se usa el IX como frame pointer
       20B7 A60D                      ldaa            13,x            ;se carga en AccA el byte a analizar
       20B9 84C0                      anda            #11000000b      ;se enmascan los bits 0 a 5, para analizar los más significativos
                      
       20BB           gp_water        EQU             *
       20BB 2620                      bne             gp_sub          ;se estudia si era agua
       20BD                           %WRITEPOS       #WATER          ;se cambia el color en el tablero
       20BD C608                      ldab            #WATER
       20BF 37                        pshb                            ;se envía el byte a escribir
       20C0 E60C                      ldab            12,x
       20C2 37                        pshb                            ;se envía letra (fila)
       20C3 E60B                      ldab            11,x
       20C5 37                        pshb                            ;se envía num (columna)
       20C6 1AEE09                    ldy             9,x
       20C9 183C                      pshy                            ;se envía dirección de inicio de matriz
       20CB E608                      ldab            8,x
       20CD 37                        pshb                            ;se envía jugador atacado
       20CE BD2373                    jsr             writepos
       20D1                           %CLRST          6
       0006           CONT_M          SET     6
       20D1                           IF CONT_M > 0
       20D1 31                        ins
       20D2                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       20D2                           IF CONT_M > 0
       20D2 31                        ins
       20D3                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       20D3                           IF CONT_M > 0
       20D3 31                        ins
       20D4                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       20D4                           IF CONT_M > 0
       20D4 31                        ins
       20D5                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       20D5                           IF CONT_M > 0
       20D5 31                        ins
       20D6                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       20D6                           IF CONT_M > 0
       20D6 31                        ins
       20D7                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       20D7                           ENDIF
       20D7                           ENDIF
       20D7                           ENDIF
       20D7                           ENDIF
       20D7                           ENDIF
       20D7                           ENDIF
       20D7                           ENDIF
       20D7                           %RESTORE
       20D7 1838                      puly
       20D9 38                        pulx
       20DA 33                        pulb
       20DB 32                        pula
       20DC 39                        rts
                      
       20DD           gp_sub          EQU             *
       20DD 8140                      cmpa            #01000000b      ;se estudia si era un submarino
       20DF 2636                      bne             gp_shi_r
       20E1                           %WRITEPOS       #SUNK_SUB       ;se cambia el color en el tablero
       20E1 C650                      ldab            #SUNK_SUB
       20E3 37                        pshb                            ;se envía el byte a escribir
       20E4 E60C                      ldab            12,x
       20E6 37                        pshb                            ;se envía letra (fila)
       20E7 E60B                      ldab            11,x
       20E9 37                        pshb                            ;se envía num (columna)
       20EA 1AEE09                    ldy             9,x
       20ED 183C                      pshy                            ;se envía dirección de inicio de matriz
       20EF E608                      ldab            8,x
       20F1 37                        pshb                            ;se envía jugador atacado
       20F2 BD2373                    jsr             writepos
       20F5                           %CLRST          6
       0006           CONT_M          SET     6
       20F5                           IF CONT_M > 0
       20F5 31                        ins
       20F6                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       20F6                           IF CONT_M > 0
       20F6 31                        ins
       20F7                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       20F7                           IF CONT_M > 0
       20F7 31                        ins
       20F8                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       20F8                           IF CONT_M > 0
       20F8 31                        ins
       20F9                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       20F9                           IF CONT_M > 0
       20F9 31                        ins
       20FA                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       20FA                           IF CONT_M > 0
       20FA 31                        ins
       20FB                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       20FB                           ENDIF
       20FB                           ENDIF
       20FB                           ENDIF
       20FB                           ENDIF
       20FB                           ENDIF
       20FB                           ENDIF
       20FB                           ENDIF
       20FB                           %PLAY_WRT                       ;se agraga la posición al array de barcos atacados
       20FB E60C                      ldab            12,x
       20FD 37                        pshb                            ;se envía letra (fila)
       20FE E60B                      ldab            11,x
       2100 37                        pshb                            ;se envía num (columna)
       2101 1AEE0E                    ldy             14,x
       2104 183C                      pshy                            ;se envía inicio de array
       2106 E608                      ldab            8,x
       2108 37                        pshb                            ;se envía jugador atacado
       2109 BD23FB                    jsr             play_wrt
       210C                           %CLRST          5
       0005           CONT_M          SET     5
       210C                           IF CONT_M > 0
       210C 31                        ins
       210D                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       210D                           IF CONT_M > 0
       210D 31                        ins
       210E                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       210E                           IF CONT_M > 0
       210E 31                        ins
       210F                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       210F                           IF CONT_M > 0
       210F 31                        ins
       2110                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2110                           IF CONT_M > 0
       2110 31                        ins
       2111                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2111                           ENDIF
       2111                           ENDIF
       2111                           ENDIF
       2111                           ENDIF
       2111                           ENDIF
       2111                           ENDIF
       2111                           %RESTORE
       2111 1838                      puly
       2113 38                        pulx
       2114 33                        pulb
       2115 32                        pula
       2116 39                        rts
                      
       2117           gp_shi_r        EQU             *               ;si no era ninguno de los anteriores, es un carguero
       2117 E60B                      ldab            11,x
       2119 C106                      cmpb            #MAX_C
       211B 276D                      beq             gp_shi_l        ;si está en el borde derecho, se continúa
       211D E60C                      ldab            12,x
       211F 37                        pshb                            ;se envía la letra (fila)
       2120 E60B                      ldab            11,x
       2122 5C                        incb                            ;se avanza a la derecha
       2123 37                        pshb                            ;se envía el num (columna)
       2124 1AEE0E                    ldy             14,x
       2127 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       2129 E608                      ldab            8,x
       212B 37                        pshb                            ;se envía el jugador atacado
       212C BD23A4                    jsr             play_src        ;se busca la posición de la derecha
       212F                           %CLRST          5
       0005           CONT_M          SET     5
       212F                           IF CONT_M > 0
       212F 31                        ins
       2130                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2130                           IF CONT_M > 0
       2130 31                        ins
       2131                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2131                           IF CONT_M > 0
       2131 31                        ins
       2132                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2132                           IF CONT_M > 0
       2132 31                        ins
       2133                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2133                           IF CONT_M > 0
       2133 31                        ins
       2134                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2134                           ENDIF
       2134                           ENDIF
       2134                           ENDIF
       2134                           ENDIF
       2134                           ENDIF
       2134                           ENDIF
       2134 2454                      bcc             gp_shi_l        ;si no había barco atacado en la derecha, se busca la izquierda
       2136                           %WRITEPOS       #SUNK_SHI       
       2136 C690                      ldab            #SUNK_SHI
       2138 37                        pshb                            ;se envía el byte a escribir
       2139 E60C                      ldab            12,x
       213B 37                        pshb                            ;se envía letra (fila)
       213C E60B                      ldab            11,x
       213E 37                        pshb                            ;se envía num (columna)
       213F 1AEE09                    ldy             9,x
       2142 183C                      pshy                            ;se envía dirección de inicio de matriz
       2144 E608                      ldab            8,x
       2146 37                        pshb                            ;se envía jugador atacado
       2147 BD2373                    jsr             writepos
       214A                           %CLRST          6
       0006           CONT_M          SET     6
       214A                           IF CONT_M > 0
       214A 31                        ins
       214B                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       214B                           IF CONT_M > 0
       214B 31                        ins
       214C                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       214C                           IF CONT_M > 0
       214C 31                        ins
       214D                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       214D                           IF CONT_M > 0
       214D 31                        ins
       214E                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       214E                           IF CONT_M > 0
       214E 31                        ins
       214F                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       214F                           IF CONT_M > 0
       214F 31                        ins
       2150                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2150                           ENDIF
       2150                           ENDIF
       2150                           ENDIF
       2150                           ENDIF
       2150                           ENDIF
       2150                           ENDIF
       2150                           ENDIF
       2150 6C0B                      inc             11,x
       2152                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       2152 C690                      ldab            #SUNK_SHI
       2154 37                        pshb                            ;se envía el byte a escribir
       2155 E60C                      ldab            12,x
       2157 37                        pshb                            ;se envía letra (fila)
       2158 E60B                      ldab            11,x
       215A 37                        pshb                            ;se envía num (columna)
       215B 1AEE09                    ldy             9,x
       215E 183C                      pshy                            ;se envía dirección de inicio de matriz
       2160 E608                      ldab            8,x
       2162 37                        pshb                            ;se envía jugador atacado
       2163 BD2373                    jsr             writepos
       2166                           %CLRST          6
       0006           CONT_M          SET     6
       2166                           IF CONT_M > 0
       2166 31                        ins
       2167                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2167                           IF CONT_M > 0
       2167 31                        ins
       2168                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2168                           IF CONT_M > 0
       2168 31                        ins
       2169                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2169                           IF CONT_M > 0
       2169 31                        ins
       216A                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       216A                           IF CONT_M > 0
       216A 31                        ins
       216B                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       216B                           IF CONT_M > 0
       216B 31                        ins
       216C                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       216C                           ENDIF
       216C                           ENDIF
       216C                           ENDIF
       216C                           ENDIF
       216C                           ENDIF
       216C                           ENDIF
       216C                           ENDIF
       216C 6A0B                      dec             11,x            ;se recupera el dato
       216E                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       216E E60C                      ldab            12,x
       2170 37                        pshb                            ;se envía letra (fila)
       2171 E60B                      ldab            11,x
       2173 37                        pshb                            ;se envía num (columna)
       2174 1AEE0E                    ldy             14,x
       2177 183C                      pshy                            ;se envía inicio de array
       2179 E608                      ldab            8,x
       217B 37                        pshb                            ;se envía jugador atacado
       217C BD23FB                    jsr             play_wrt
       217F                           %CLRST          5
       0005           CONT_M          SET     5
       217F                           IF CONT_M > 0
       217F 31                        ins
       2180                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2180                           IF CONT_M > 0
       2180 31                        ins
       2181                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2181                           IF CONT_M > 0
       2181 31                        ins
       2182                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2182                           IF CONT_M > 0
       2182 31                        ins
       2183                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2183                           IF CONT_M > 0
       2183 31                        ins
       2184                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2184                           ENDIF
       2184                           ENDIF
       2184                           ENDIF
       2184                           ENDIF
       2184                           ENDIF
       2184                           ENDIF
       2184                           %RESTORE
       2184 1838                      puly
       2186 38                        pulx
       2187 33                        pulb
       2188 32                        pula
       2189 39                        rts
                      
       218A           gp_shi_l        EQU             *
       218A E60B                      ldab            11,x
       218C C101                      cmpb            #MIN_C
       218E 276D                      beq             gp_shi_u        ;si está en el borde izquierdo, se continúa
       2190 E60C                      ldab            12,x
       2192 37                        pshb                            ;se envía la letra (fila)
       2193 E60B                      ldab            11,x
       2195 5A                        decb                            ;se retrocede a la izquierda
       2196 37                        pshb                            ;se envía el número (columna)
       2197 1AEE0E                    ldy             14,x
       219A 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       219C E608                      ldab            8,x
       219E 37                        pshb                            ;se envía el jugador atacado
       219F BD23A4                    jsr             play_src        ;se busca la posición de la izquierda
       21A2                           %CLRST          5
       0005           CONT_M          SET     5
       21A2                           IF CONT_M > 0
       21A2 31                        ins
       21A3                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       21A3                           IF CONT_M > 0
       21A3 31                        ins
       21A4                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       21A4                           IF CONT_M > 0
       21A4 31                        ins
       21A5                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       21A5                           IF CONT_M > 0
       21A5 31                        ins
       21A6                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       21A6                           IF CONT_M > 0
       21A6 31                        ins
       21A7                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       21A7                           ENDIF
       21A7                           ENDIF
       21A7                           ENDIF
       21A7                           ENDIF
       21A7                           ENDIF
       21A7                           ENDIF
       21A7 2454                      bcc             gp_shi_u        ;si no había barco atacado a la izquierda, se busca arriba
       21A9                           %WRITEPOS       #SUNK_SHI       
       21A9 C690                      ldab            #SUNK_SHI
       21AB 37                        pshb                            ;se envía el byte a escribir
       21AC E60C                      ldab            12,x
       21AE 37                        pshb                            ;se envía letra (fila)
       21AF E60B                      ldab            11,x
       21B1 37                        pshb                            ;se envía num (columna)
       21B2 1AEE09                    ldy             9,x
       21B5 183C                      pshy                            ;se envía dirección de inicio de matriz
       21B7 E608                      ldab            8,x
       21B9 37                        pshb                            ;se envía jugador atacado
       21BA BD2373                    jsr             writepos
       21BD                           %CLRST          6
       0006           CONT_M          SET     6
       21BD                           IF CONT_M > 0
       21BD 31                        ins
       21BE                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       21BE                           IF CONT_M > 0
       21BE 31                        ins
       21BF                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       21BF                           IF CONT_M > 0
       21BF 31                        ins
       21C0                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       21C0                           IF CONT_M > 0
       21C0 31                        ins
       21C1                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       21C1                           IF CONT_M > 0
       21C1 31                        ins
       21C2                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       21C2                           IF CONT_M > 0
       21C2 31                        ins
       21C3                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       21C3                           ENDIF
       21C3                           ENDIF
       21C3                           ENDIF
       21C3                           ENDIF
       21C3                           ENDIF
       21C3                           ENDIF
       21C3                           ENDIF
       21C3 6A0B                      dec             11,x
       21C5                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       21C5 C690                      ldab            #SUNK_SHI
       21C7 37                        pshb                            ;se envía el byte a escribir
       21C8 E60C                      ldab            12,x
       21CA 37                        pshb                            ;se envía letra (fila)
       21CB E60B                      ldab            11,x
       21CD 37                        pshb                            ;se envía num (columna)
       21CE 1AEE09                    ldy             9,x
       21D1 183C                      pshy                            ;se envía dirección de inicio de matriz
       21D3 E608                      ldab            8,x
       21D5 37                        pshb                            ;se envía jugador atacado
       21D6 BD2373                    jsr             writepos
       21D9                           %CLRST          6
       0006           CONT_M          SET     6
       21D9                           IF CONT_M > 0
       21D9 31                        ins
       21DA                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       21DA                           IF CONT_M > 0
       21DA 31                        ins
       21DB                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       21DB                           IF CONT_M > 0
       21DB 31                        ins
       21DC                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       21DC                           IF CONT_M > 0
       21DC 31                        ins
       21DD                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       21DD                           IF CONT_M > 0
       21DD 31                        ins
       21DE                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       21DE                           IF CONT_M > 0
       21DE 31                        ins
       21DF                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       21DF                           ENDIF
       21DF                           ENDIF
       21DF                           ENDIF
       21DF                           ENDIF
       21DF                           ENDIF
       21DF                           ENDIF
       21DF                           ENDIF
       21DF 6C0B                      inc             11,x            ;se recupera el dato
       21E1                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       21E1 E60C                      ldab            12,x
       21E3 37                        pshb                            ;se envía letra (fila)
       21E4 E60B                      ldab            11,x
       21E6 37                        pshb                            ;se envía num (columna)
       21E7 1AEE0E                    ldy             14,x
       21EA 183C                      pshy                            ;se envía inicio de array
       21EC E608                      ldab            8,x
       21EE 37                        pshb                            ;se envía jugador atacado
       21EF BD23FB                    jsr             play_wrt
       21F2                           %CLRST          5
       0005           CONT_M          SET     5
       21F2                           IF CONT_M > 0
       21F2 31                        ins
       21F3                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       21F3                           IF CONT_M > 0
       21F3 31                        ins
       21F4                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       21F4                           IF CONT_M > 0
       21F4 31                        ins
       21F5                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       21F5                           IF CONT_M > 0
       21F5 31                        ins
       21F6                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       21F6                           IF CONT_M > 0
       21F6 31                        ins
       21F7                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       21F7                           ENDIF
       21F7                           ENDIF
       21F7                           ENDIF
       21F7                           ENDIF
       21F7                           ENDIF
       21F7                           ENDIF
       21F7                           %RESTORE
       21F7 1838                      puly
       21F9 38                        pulx
       21FA 33                        pulb
       21FB 32                        pula
       21FC 39                        rts
                      
       21FD           gp_shi_u        EQU             *
       21FD E60C                      ldab            12,x
       21FF C10A                      cmpb            #MIN_F
       2201 276D                      beq             gp_shi_d        ;si está en el borde superior, se continúa
       2203 E60C                      ldab            12,x
       2205 5A                        decb                            ;se va a la posición de arriba
       2206 37                        pshb                            ;se envía la letra (fila)
       2207 E60B                      ldab            11,x
       2209 37                        pshb                            ;se envía el num (columna)
       220A 1AEE0E                    ldy             14,x
       220D 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       220F E608                      ldab            8,x
       2211 37                        pshb                            ;se envía el jugador atacado
       2212 BD23A4                    jsr             play_src        ;se busca la posición de arriba
       2215                           %CLRST          5
       0005           CONT_M          SET     5
       2215                           IF CONT_M > 0
       2215 31                        ins
       2216                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2216                           IF CONT_M > 0
       2216 31                        ins
       2217                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2217                           IF CONT_M > 0
       2217 31                        ins
       2218                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2218                           IF CONT_M > 0
       2218 31                        ins
       2219                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2219                           IF CONT_M > 0
       2219 31                        ins
       221A                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       221A                           ENDIF
       221A                           ENDIF
       221A                           ENDIF
       221A                           ENDIF
       221A                           ENDIF
       221A                           ENDIF
       221A 2454                      bcc             gp_shi_d        ;si no había barco atacado arriba, se busca abajo
       221C                           %WRITEPOS       #SUNK_SHI       
       221C C690                      ldab            #SUNK_SHI
       221E 37                        pshb                            ;se envía el byte a escribir
       221F E60C                      ldab            12,x
       2221 37                        pshb                            ;se envía letra (fila)
       2222 E60B                      ldab            11,x
       2224 37                        pshb                            ;se envía num (columna)
       2225 1AEE09                    ldy             9,x
       2228 183C                      pshy                            ;se envía dirección de inicio de matriz
       222A E608                      ldab            8,x
       222C 37                        pshb                            ;se envía jugador atacado
       222D BD2373                    jsr             writepos
       2230                           %CLRST          6
       0006           CONT_M          SET     6
       2230                           IF CONT_M > 0
       2230 31                        ins
       2231                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2231                           IF CONT_M > 0
       2231 31                        ins
       2232                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2232                           IF CONT_M > 0
       2232 31                        ins
       2233                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2233                           IF CONT_M > 0
       2233 31                        ins
       2234                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2234                           IF CONT_M > 0
       2234 31                        ins
       2235                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2235                           IF CONT_M > 0
       2235 31                        ins
       2236                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2236                           ENDIF
       2236                           ENDIF
       2236                           ENDIF
       2236                           ENDIF
       2236                           ENDIF
       2236                           ENDIF
       2236                           ENDIF
       2236 6A0C                      dec             12,x
       2238                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       2238 C690                      ldab            #SUNK_SHI
       223A 37                        pshb                            ;se envía el byte a escribir
       223B E60C                      ldab            12,x
       223D 37                        pshb                            ;se envía letra (fila)
       223E E60B                      ldab            11,x
       2240 37                        pshb                            ;se envía num (columna)
       2241 1AEE09                    ldy             9,x
       2244 183C                      pshy                            ;se envía dirección de inicio de matriz
       2246 E608                      ldab            8,x
       2248 37                        pshb                            ;se envía jugador atacado
       2249 BD2373                    jsr             writepos
       224C                           %CLRST          6
       0006           CONT_M          SET     6
       224C                           IF CONT_M > 0
       224C 31                        ins
       224D                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       224D                           IF CONT_M > 0
       224D 31                        ins
       224E                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       224E                           IF CONT_M > 0
       224E 31                        ins
       224F                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       224F                           IF CONT_M > 0
       224F 31                        ins
       2250                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2250                           IF CONT_M > 0
       2250 31                        ins
       2251                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2251                           IF CONT_M > 0
       2251 31                        ins
       2252                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2252                           ENDIF
       2252                           ENDIF
       2252                           ENDIF
       2252                           ENDIF
       2252                           ENDIF
       2252                           ENDIF
       2252                           ENDIF
       2252 6C0C                      inc             12,x            ;se recupera el dato
       2254                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       2254 E60C                      ldab            12,x
       2256 37                        pshb                            ;se envía letra (fila)
       2257 E60B                      ldab            11,x
       2259 37                        pshb                            ;se envía num (columna)
       225A 1AEE0E                    ldy             14,x
       225D 183C                      pshy                            ;se envía inicio de array
       225F E608                      ldab            8,x
       2261 37                        pshb                            ;se envía jugador atacado
       2262 BD23FB                    jsr             play_wrt
       2265                           %CLRST          5
       0005           CONT_M          SET     5
       2265                           IF CONT_M > 0
       2265 31                        ins
       2266                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2266                           IF CONT_M > 0
       2266 31                        ins
       2267                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2267                           IF CONT_M > 0
       2267 31                        ins
       2268                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2268                           IF CONT_M > 0
       2268 31                        ins
       2269                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2269                           IF CONT_M > 0
       2269 31                        ins
       226A                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       226A                           ENDIF
       226A                           ENDIF
       226A                           ENDIF
       226A                           ENDIF
       226A                           ENDIF
       226A                           ENDIF
       226A                           %RESTORE
       226A 1838                      puly
       226C 38                        pulx
       226D 33                        pulb
       226E 32                        pula
       226F 39                        rts
                      
       2270           gp_shi_d        EQU             *
       2270 E60C                      ldab            12,x
       2272 C10F                      cmpb            #MAX_F
       2274 276D                      beq             gp_shi_h        ;si está en el borde inferior, se continúa
       2276 E60C                      ldab            12,x
       2278 5C                        incb                            ;se va a la posición de abajo
       2279 37                        pshb                            ;se envía la letra (fila)
       227A E60B                      ldab            11,x
       227C 37                        pshb                            ;se envía el num (columna)
       227D 1AEE0E                    ldy             14,x
       2280 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       2282 E608                      ldab            8,x
       2284 37                        pshb                            ;se envía el jugador atacado
       2285 BD23A4                    jsr             play_src        ;se busca la posición de abajo
       2288                           %CLRST          5
       0005           CONT_M          SET     5
       2288                           IF CONT_M > 0
       2288 31                        ins
       2289                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2289                           IF CONT_M > 0
       2289 31                        ins
       228A                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       228A                           IF CONT_M > 0
       228A 31                        ins
       228B                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       228B                           IF CONT_M > 0
       228B 31                        ins
       228C                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       228C                           IF CONT_M > 0
       228C 31                        ins
       228D                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       228D                           ENDIF
       228D                           ENDIF
       228D                           ENDIF
       228D                           ENDIF
       228D                           ENDIF
       228D                           ENDIF
       228D 2454                      bcc             gp_shi_h        ;si no había barco atacado abajo, el carguero está tocado
       228F                           %WRITEPOS       #SUNK_SHI       
       228F C690                      ldab            #SUNK_SHI
       2291 37                        pshb                            ;se envía el byte a escribir
       2292 E60C                      ldab            12,x
       2294 37                        pshb                            ;se envía letra (fila)
       2295 E60B                      ldab            11,x
       2297 37                        pshb                            ;se envía num (columna)
       2298 1AEE09                    ldy             9,x
       229B 183C                      pshy                            ;se envía dirección de inicio de matriz
       229D E608                      ldab            8,x
       229F 37                        pshb                            ;se envía jugador atacado
       22A0 BD2373                    jsr             writepos
       22A3                           %CLRST          6
       0006           CONT_M          SET     6
       22A3                           IF CONT_M > 0
       22A3 31                        ins
       22A4                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       22A4                           IF CONT_M > 0
       22A4 31                        ins
       22A5                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       22A5                           IF CONT_M > 0
       22A5 31                        ins
       22A6                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       22A6                           IF CONT_M > 0
       22A6 31                        ins
       22A7                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       22A7                           IF CONT_M > 0
       22A7 31                        ins
       22A8                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       22A8                           IF CONT_M > 0
       22A8 31                        ins
       22A9                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       22A9                           ENDIF
       22A9                           ENDIF
       22A9                           ENDIF
       22A9                           ENDIF
       22A9                           ENDIF
       22A9                           ENDIF
       22A9                           ENDIF
       22A9 6C0C                      inc             12,x
       22AB                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       22AB C690                      ldab            #SUNK_SHI
       22AD 37                        pshb                            ;se envía el byte a escribir
       22AE E60C                      ldab            12,x
       22B0 37                        pshb                            ;se envía letra (fila)
       22B1 E60B                      ldab            11,x
       22B3 37                        pshb                            ;se envía num (columna)
       22B4 1AEE09                    ldy             9,x
       22B7 183C                      pshy                            ;se envía dirección de inicio de matriz
       22B9 E608                      ldab            8,x
       22BB 37                        pshb                            ;se envía jugador atacado
       22BC BD2373                    jsr             writepos
       22BF                           %CLRST          6
       0006           CONT_M          SET     6
       22BF                           IF CONT_M > 0
       22BF 31                        ins
       22C0                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       22C0                           IF CONT_M > 0
       22C0 31                        ins
       22C1                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       22C1                           IF CONT_M > 0
       22C1 31                        ins
       22C2                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       22C2                           IF CONT_M > 0
       22C2 31                        ins
       22C3                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       22C3                           IF CONT_M > 0
       22C3 31                        ins
       22C4                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       22C4                           IF CONT_M > 0
       22C4 31                        ins
       22C5                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       22C5                           ENDIF
       22C5                           ENDIF
       22C5                           ENDIF
       22C5                           ENDIF
       22C5                           ENDIF
       22C5                           ENDIF
       22C5                           ENDIF
       22C5 6A0C                      dec             12,x            ;se recupera el dato
       22C7                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       22C7 E60C                      ldab            12,x
       22C9 37                        pshb                            ;se envía letra (fila)
       22CA E60B                      ldab            11,x
       22CC 37                        pshb                            ;se envía num (columna)
       22CD 1AEE0E                    ldy             14,x
       22D0 183C                      pshy                            ;se envía inicio de array
       22D2 E608                      ldab            8,x
       22D4 37                        pshb                            ;se envía jugador atacado
       22D5 BD23FB                    jsr             play_wrt
       22D8                           %CLRST          5
       0005           CONT_M          SET     5
       22D8                           IF CONT_M > 0
       22D8 31                        ins
       22D9                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       22D9                           IF CONT_M > 0
       22D9 31                        ins
       22DA                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       22DA                           IF CONT_M > 0
       22DA 31                        ins
       22DB                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       22DB                           IF CONT_M > 0
       22DB 31                        ins
       22DC                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       22DC                           IF CONT_M > 0
       22DC 31                        ins
       22DD                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       22DD                           ENDIF
       22DD                           ENDIF
       22DD                           ENDIF
       22DD                           ENDIF
       22DD                           ENDIF
       22DD                           ENDIF
       22DD                           %RESTORE
       22DD 1838                      puly
       22DF 38                        pulx
       22E0 33                        pulb
       22E1 32                        pula
       22E2 39                        rts
                      
       22E3           gp_shi_h        EQU             *
       22E3                           %WRITEPOS       #HIT            ;se marca al carguero como tocado
       22E3 C6B0                      ldab            #HIT
       22E5 37                        pshb                            ;se envía el byte a escribir
       22E6 E60C                      ldab            12,x
       22E8 37                        pshb                            ;se envía letra (fila)
       22E9 E60B                      ldab            11,x
       22EB 37                        pshb                            ;se envía num (columna)
       22EC 1AEE09                    ldy             9,x
       22EF 183C                      pshy                            ;se envía dirección de inicio de matriz
       22F1 E608                      ldab            8,x
       22F3 37                        pshb                            ;se envía jugador atacado
       22F4 BD2373                    jsr             writepos
       22F7                           %CLRST          6
       0006           CONT_M          SET     6
       22F7                           IF CONT_M > 0
       22F7 31                        ins
       22F8                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       22F8                           IF CONT_M > 0
       22F8 31                        ins
       22F9                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       22F9                           IF CONT_M > 0
       22F9 31                        ins
       22FA                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       22FA                           IF CONT_M > 0
       22FA 31                        ins
       22FB                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       22FB                           IF CONT_M > 0
       22FB 31                        ins
       22FC                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       22FC                           IF CONT_M > 0
       22FC 31                        ins
       22FD                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       22FD                           ENDIF
       22FD                           ENDIF
       22FD                           ENDIF
       22FD                           ENDIF
       22FD                           ENDIF
       22FD                           ENDIF
       22FD                           ENDIF
       22FD                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       22FD E60C                      ldab            12,x
       22FF 37                        pshb                            ;se envía letra (fila)
       2300 E60B                      ldab            11,x
       2302 37                        pshb                            ;se envía num (columna)
       2303 1AEE0E                    ldy             14,x
       2306 183C                      pshy                            ;se envía inicio de array
       2308 E608                      ldab            8,x
       230A 37                        pshb                            ;se envía jugador atacado
       230B BD23FB                    jsr             play_wrt
       230E                           %CLRST          5
       0005           CONT_M          SET     5
       230E                           IF CONT_M > 0
       230E 31                        ins
       230F                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       230F                           IF CONT_M > 0
       230F 31                        ins
       2310                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2310                           IF CONT_M > 0
       2310 31                        ins
       2311                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2311                           IF CONT_M > 0
       2311 31                        ins
       2312                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2312                           IF CONT_M > 0
       2312 31                        ins
       2313                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2313                           ENDIF
       2313                           ENDIF
       2313                           ENDIF
       2313                           ENDIF
       2313                           ENDIF
       2313                           ENDIF
       2313                           %RESTORE
       2313 1838                      puly
       2315 38                        pulx
       2316 33                        pulb
       2317 32                        pula
       2318 39                        rts
                      
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
                      
       2319           mirror          EQU             *       
                              
       2319                           %BACKUP
       2319 36                        psha
       231A 37                        pshb
       231B 3C                        pshx
       231C 183C                      pshy
       231E 30                        tsx                             ;creo frame pointer
       231F A60B                      ldaa            11,x            ;cargo número de jugador
       2321 E608                      ldab            8,x             ;cargo longitud de la copia (del array a copiar)
       2323 3D                        mul                             ;obtengo offset de jugador
       2324 E30C                      addd            12,x            ;sumo offset de jugador
       2326 ED0C                      std             12,x            ;guardo la dirección con offset del array fuente
                      
       2328 A60C                      ldaa            12,x            ;cargo longitud de copia en contador
       232A 2710                      beq             mi_end          ;si es cero, vuelvo desde subrutina
       232C 1AEE09                    ldy             9,x             ;cargo índice a destino de la copia.
       232F EE0C                      ldx             12,x            ;cargo índice a array origen de copia.
                      
       2331           mi_loop         EQU             *
                      
       2331 E600                      ldab            0,x             ;cargo byte a copiar
       2333 18E700                    stab            0,y             ;copio byte en destino
       2336 08                        inx                             ;siguiente byte
       2337 1808                      iny                             ;siguiente byte
       2339 4A                        deca                            ;decremento contador
       233A 26F5                      bne             mi_loop
                      
       233C           mi_end          EQU             *
                      
       233C                           %RESTORE
       233C 1838                      puly
       233E 38                        pulx
       233F 33                        pulb
       2340 32                        pula
                              
       2341 39                        rts     
                                      
                                                      
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
                      
       2342           fetchpos        EQU             *
                      
       2342                           %BACKUP
       2342 36                        psha
       2343 37                        pshb
       2344 3C                        pshx
       2345 183C                      pshy
       2347 30                        tsx                             ;creo frame pointer
       2348 A608                      ldaa            8,x             ;cargo numero de jugador
       234A C624                      ldab            #ARRAY_L        ;cargo longitud del arreglo (MAX 255 bytes)
       234C 3D                        mul                             ;obtengo offset de jugador
       234D E309                      addd            9,x             ;sumo offset de jugador a top of array
       234F ED09                      std             9,x             ;obtengo direccion al arreglo del jugador
                      
       2351 A60C                      ldaa            12,x            ;cargo coordenada letra (FILA)
       2353 C60A                      ldab            #NORM           ;cargo valor de normalización de filas
       2355 10                        sba                             ;obtengo numero de fila
       2356 C606                      ldab            #COL            ;cargo longitud de fila
       2358 3D                        mul                             ;obtengo offset de fila
       2359 E309                      addd            9,x             ;sumo offset de fila a la dirección del arreglo del jugador
       235B ED09                      std             9,x             ;obtengo dirección a la fila que se desea
                      
       235D E60B                      ldab            11,x            ;cargo coordenada (offset) de columna
       235F 5A                        decb                            ;normalizo offset de columna
       2360 4F                        clra                            ;borro AccA para tener el valor de AccB en AccD
       2361 E309                      addd            9,x             ;sumo offset de columna a la dirección temporal
       2363 ED09                      std             9,x             ;guardo en stack la dirección del byte a consultar
                      
       2365 1AEE09                    ldy             9,x             ;cargo en IY la dirección del byte que se desea obtener
       2368 18A600                    ldaa            0,y             ;cargo el byte a devolver en AccA
       236B A70C                      staa            12,x            ;guardo el byte a devolver en stack
       236D                           %RESTORE
       236D 1838                      puly
       236F 38                        pulx
       2370 33                        pulb
       2371 32                        pula
                                      
       2372 39                        rts
                                      
                                      
                      
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
                      
       2373           writepos        EQU             *
                      
       2373                           %BACKUP
       2373 36                        psha
       2374 37                        pshb
       2375 3C                        pshx
       2376 183C                      pshy
       2378 30                        tsx                             ;creo frame pointer
       2379 A608                      ldaa            8,x             ;cargo numero de jugador
       237B C624                      ldab            #ARRAY_L        ;cargo longitud del arreglo (MAX 255 bytes)
       237D 3D                        mul                             ;obtengo offset de jugador
       237E E309                      addd            9,x             ;sumo offset de jugador a top of array
       2380 ED09                      std             9,x             ;obtengo direccion al arreglo del jugador
                      
       2382 A60C                      ldaa            12,x            ;cargo coordenada letra (FILA)
       2384 C60A                      ldab            #NORM           ;cargo valor de normalización de filas
       2386 10                        sba                             ;obtengo numero de fila
       2387 C606                      ldab            #COL            ;cargo longitud de fila
       2389 3D                        mul                             ;obtengo offset de fila
       238A E309                      addd            9,x             ;sumo offset de fila a la dirección del arreglo del jugador
       238C ED09                      std             9,x             ;obtengo dirección a la fila que se desea
                      
       238E E60B                      ldab            11,x            ;cargo coordenada (offset) de columna
       2390 5A                        decb                            ;normalizo offset de columna
       2391 4F                        clra                            ;borro AccA para tener el valor de AccB en AccD
       2392 E309                      addd            9,x             ;sumo offset de columna a la dirección temporal
       2394 ED09                      std             9,x             ;guardo en stack la dirección del byte a consultar
                      
       2396 1AEE09                    ldy             9,x             ;cargo en IY la dirección donde se desea escribir
       2399 A60D                      ldaa            13,x            ;cargo en AccA el byte que se quiere escribir en el arreglo
       239B 18A700                    staa            0,y             ;escribo el byte sobre el arreglo
       239E                           %RESTORE
       239E 1838                      puly
       23A0 38                        pulx
       23A1 33                        pulb
       23A2 32                        pula
                                      
       23A3 39                        rts
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
                      
       23A4           play_src        EQU             *               
                                              
       23A4                           %BACKUP
       23A4 36                        psha
       23A5 37                        pshb
       23A6 3C                        pshx
       23A7 183C                      pshy
       23A9 30                        tsx                             ;creo frame pointer
       23AA A608                      ldaa            8,x             ;cargo número de array (jugador)
       23AC C613                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       23AE 3D                        mul                             ;obtengo offset de jugador
       23AF E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       23B1 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
                      
       23B3           ps_loop         EQU             *
                      
       23B3 18A600                    ldaa            0,y             ;cargo byte1 a comparar
       23B6 2718                      beq             ps_ntfnd        ;si encontré el terminador, el valor requerido no está en el array
                      
       23B8 A10C                      cmpa            12,x            ;comparo con el byte1 a buscar
       23BA 2706                      beq             ps_next         ;si coinciden, busco el byte 2
       23BC 1808                      iny                             ;me posiciono sobre siguiente byte1 a comparar
       23BE 1808                      iny
       23C0 20F1                      bra             ps_loop         ;vuelvo a comparar
                      
       23C2           ps_next         EQU             *
                      
       23C2 18E601                    ldab            1,y             ;cargo byte a continuación del anterior
       23C5 1808                      iny                             ;me posiciono sobre siguiente byte1 a comparar
       23C7 1808                      iny
       23C9 E10B                      cmpb            11,x            ;comparo con byte2
       23CB 26E6                      bne             ps_loop         ;si no son iguales, sigo buscando
       23CD 0D                        sec                             ;set del carry para informar que se encontró el valor
       23CE 2001                      bra             ps_end
                                              
       23D0           ps_ntfnd        EQU             *
                      
       23D0 0C                        clc                             ;carry en 0 para informar que no se encontró el valor
                      
       23D1           ps_end          EQU             *
                                      
       23D1                           %RESTORE
       23D1 1838                      puly
       23D3 38                        pulx
       23D4 33                        pulb
       23D5 32                        pula
                      
       23D6 39                        rts
                      
                      
                      
                                              
                      
                                              
                      
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
                      
       23D7           play_cnt        EQU             *
                      
       23D7                           %BACKUP
       23D7 36                        psha
       23D8 37                        pshb
       23D9 3C                        pshx
       23DA 183C                      pshy
       23DC 30                        tsx                             ;creo frame pointer
       23DD A608                      ldaa            8,x             ;cargo número de array (jugador)
       23DF C613                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       23E1 3D                        mul                             ;obtengo offset de jugador
       23E2 E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       23E4 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
       23E6 4F                        clra                            ;inicializo contador
                                              
       23E7           pc_loop         EQU             *               
                                              
       23E7 18E600                    ldab            0,y             ;cargo byte a comparar
       23EA 2707                      beq             pc_end          ;si es terminador, terminé de buscar
       23EC 1808                      iny                             ;si no es terminador, me posiciono sobre siguiente dato de 
       23EE 1808                      iny                             ;2 bytes
       23F0 4C                        inca                            ;incremento contador
       23F1 20F4                      bra             pc_loop         ;vuelvo a comparar
                                              
       23F3           pc_end          EQU             *       
                                              
       23F3 A70A                      staa            10,x            ;devuelvo valor por stack
       23F5                           %RESTORE
       23F5 1838                      puly
       23F7 38                        pulx
       23F8 33                        pulb
       23F9 32                        pula
                                      
       23FA 39                        rts
                      
                      
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
                      
       23FB           play_wrt        EQU             *
                      
       23FB                           %BACKUP
       23FB 36                        psha
       23FC 37                        pshb
       23FD 3C                        pshx
       23FE 183C                      pshy
       2400 30                        tsx                             ;creo frame pointer
       2401 A608                      ldaa            8,x             ;cargo número de array (jugador)
       2403 C613                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       2405 3D                        mul                             ;obtengo offset de jugador
       2406 E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       2408 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
                      
                      
       240A           pw_loop         EQU             *
                      
       240A 18A600                    ldaa            0,y             ;cargo byte del array
       240D 2704                      beq             pw_end          ;si es cero, entonces escribo los datos
       240F 1808                      iny                             ;si no es cero, me posiciono sobre la siguiente posición
       2411 20F7                      bra             pw_loop         ;vuelvo a buscar ceros
                      
       2413           pw_end          EQU             *
                      
       2413 A60C                      ldaa            12,x            ;cargo primer byte a escribir
       2415 18A700                    staa            0,y             ;guardo primer byte en memoria
       2418 1808                      iny                             ;me posiciono en el siguiente byte
       241A A60B                      ldaa            11,x            ;cargo segundo byte a escribir
       241C 18A700                    staa            0,y             ;guardo segundo byte en memoria
       241F                           %RESTORE
       241F 1838                      puly
       2421 38                        pulx
       2422 33                        pulb
       2423 32                        pula
                      
       2424 39                        rts
                      
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
       2425           clrzone         EQU     *
                      
       2425                           %BACKUP                         ;backup de registros en stack
       2425 36                        psha
       2426 37                        pshb
       2427 3C                        pshx
       2428 183C                      pshy
                      
       242A 30                        tsx
       242B A608                      ldaa    8,x                     ;cargo contador con longitud de zona a borrar
       242D EE09                      ldx     9,x                     ;cargo puntero zona de memoria a borrar
       242F D600                      ldab    $00                     ;cargo el cero
       2431           clr_loop        EQU     *
       2431 E700                      stab    0,x                     ;guardo caracter en posicion del array
       2433 08                        inx                             ;siguiente posición     
       2434 4A                        deca                            ;decremento contador
       2435 26FA                      bne     clr_loop                ;si no es cero, escribo en la posición siguiente.
                      
       2437                           %RESTORE                        :restauro backup en registros
       2437 1838                      puly
       2439 38                        pulx
       243A 33                        pulb
       243B 32                        pula
                      
       243C 39                        rts
                      
                      *---------------------------------------
                      * WhoPlays: subrutina para ver que 
                      *           jugador juega
                      *---------------------------------------
                      
       243D           whoplays        EQU             *
                      
       243D BD24FA    wait_ply        jsr             read_kb
       2440 4D                        tsta
       2441 2BFA                      bmi             wait_ply
       2443 8103                      cmpa            #B_P1
       2445 2708                      beq             play_p1
       2447 8107                      cmpa            #B_P2
       2449 26F2                      bne             wait_ply
       244B 8600                      ldaa            #P1
       244D 2002                      bra             ply_end
                      
       244F 8601      play_p1         ldaa            #P2
       2451 39        ply_end         rts             
                      
                      *---------------------------------------
                      * Input: subrutina que devuelve
                      *        entrada para la batalla naval
                      * AccA: Fila de la posicion jugada
                      * AccB: Columna de la posicion jugada
                      *---------------------------------------
                      
       2452           input           EQU             *
                      
                      * back up de registros
       2452 3C                        pshx
       2453 183C                      pshy
                      
                      * creo variables locales
       2455 36                        psha
       2456 36                        psha
       2457 36                        psha
                      
                      * inicializo framepointer
       2458 1830                      tsy
                      
                      * prendo el led
       245A                           %LED_INV
       245A B61000                    ldaa            PORTA
       245D 8840                      eora            #SHF_LED
       245F B71000                    staa            PORTA
                      
                      * inicializo el contador
       2462 8603      inp_rest        ldaa            #INPUTS
       2464 18A700                    staa            CONT,Y
                                      
                      * limpio el display
       2467 CE41F9                    ldx             #string
       246A 8604                      ldaa            #4
       246C C620                      ldab            #ESP
       246E BD25C4                    jsr             clrarray
                      
       2471 BD25B3                    jsr             clr_dis
                      
       2474 86FF                      ldaa            #$FF
       2476 B741F8                    staa            buffer
                      
                      * lectura de teclado
       2479 BD24FA    wait_kb         jsr             read_kb
       247C 4D                        tsta
       247D 2BFA                      bmi             wait_kb
       247F B141F8                    cmpa            buffer
       2482 27F5                      beq             wait_kb
       2484 B741F8                    staa            buffer
                                      
       2487 810F                      cmpa            #B_reset
       2489 27D7                      beq             inp_rest
                      
       248B 186A00                    dec             CONT,Y
       248E 2727                      beq             chk_fire
       2490 36                        psha
       2491 CE41FE                    ldx             #kb_value
       2494 16                        tab
       2495 3A                        abx
       2496 A600                      ldaa            0,X             
       2498 183C                      pshy
       249A 18E600                    ldab            CONT,Y
       249D 183A                      aby
       249F 18A700                    staa            0,Y
       24A2 1838                      puly
                      
       24A4 33                        pulb
       24A5 BD25A3                    jsr             to_ascii
       24A8 17                        tba
                                      
       24A9 CE41F9                    ldx             #string
       24AC BD2593                    jsr             sh_value
                      
       24AF CE41F9                    ldx             #string
       24B2 BD24E6                    jsr             show
                      
       24B5 20C2                      bra             wait_kb
                      
       24B7 810B      chk_fire        cmpa            #B_fire
       24B9 26A7                      bne             inp_rest
       24BB 18A602                    ldaa            FILA_L,Y
       24BE 810A                      cmpa            #MIN_FIL
       24C0 25A0                      blo             inp_rest
       24C2 810F                      cmpa            #MAX_FIL
       24C4 229C                      bhi             inp_rest
       24C6 18A601                    ldaa            COL_L,Y
       24C9 8101                      cmpa            #MIN_COL
       24CB 2595                      blo             inp_rest
       24CD 8106                      cmpa            #MAX_COL
       24CF 2291                      bhi             inp_rest
                      
                      * apago el led
       24D1                           %LED_INV
       24D1 B61000                    ldaa            PORTA
       24D4 8840                      eora            #SHF_LED
       24D6 B71000                    staa            PORTA
                      
                      * libero variables locales
       24D9 32                        pula
       24DA 32                        pula
       24DB 32                        pula
                      
       24DC 18A602    return          ldaa            FILA_L,Y
       24DF 18E601                    ldab            COL_L,Y         
                      
                      * restore de registros
       24E2 1838                      puly
       24E4 38                        pulx
                      
       24E5 39                        rts
                      
                      *------------------------------------
                      * Show: Por IX direccion de string de
                      *       4 caracteres para imprimir
                      *------------------------------------
                      
       24E6           show            EQU             *
                      
                      * back up de registros
       24E6 3C                        pshx
       24E7 37                        pshb
       24E8 36                        psha
                      
                      * inicializo contador de posiciones
       24E9 C604                      ldab            #LENGTH
                                      
                      * loop para enviar a cada posicion
       24EB 5A        sh_loop         decb
       24EC 2B08                      bmi             sh_end          ; si no pase por todas las posiciones
       24EE A600                      ldaa            0,X             ; tomo del string el caracter
       24F0 BD25AC                    jsr             outchar         ; y lo imprimo en la posicion actual
       24F3 08                        inx
       24F4 20F5                      bra             sh_loop
                      
                      * restore de registros                          
       24F6 32        sh_end          pula
       24F7 33                        pulb
       24F8 38                        pulx
       24F9 39                        rts
                      
                      *----------------------------------------------
                      * read_kb: AccA: tecla presionada
                      *          o -1
                      *----------------------------------------------
                      
       24FA           read_kb         EQU             *
                      
                      * back up de registros
       24FA 3C                        pshx
       24FB 37                        pshb
                      
                      * creo espacio para variables locales
       24FC                           %MALLOC         1               ; variable mascara de filas
       24FC                           IF 1 > 0
       24FC 30                        tsx
       24FD 8F                        xgdx
       24FE 830001                    subd    #1
       2501 8F                        xgdx
       2502 35                        txs
       2503                           ENDIF
       2503                           %MALLOC         1               ; variable mascara de columnas
       2503                           IF 1 > 0
       2503 30                        tsx
       2504 8F                        xgdx
       2505 830001                    subd    #1
       2508 8F                        xgdx
       2509 35                        txs
       250A                           ENDIF
       250A                           %MALLOC         1               ; variable contador de columnas
       250A                           IF 1 > 0
       250A 30                        tsx
       250B 8F                        xgdx
       250C 830001                    subd    #1
       250F 8F                        xgdx
       2510 35                        txs
       2511                           ENDIF
       2511                           %MALLOC         1               ; variable contador de tecla
       2511                           IF 1 > 0
       2511 30                        tsx
       2512 8F                        xgdx
       2513 830001                    subd    #1
       2516 8F                        xgdx
       2517 35                        txs
       2518                           ENDIF
                      
                      * inicializacion de variables
       2518 30                        tsx                             ; framepointer para acceder a locales
       2519 C604                      ldab            #LENGTH         ; contador de fila
       251B 860E                      ldaa            #FIL0           ; mascara para la fila cero
       251D A703                      staa            MASK_FIL,X      
       251F 8610                      ldaa            #COL0           ; mascara para la columna cero
       2521 A702                      staa            MASK_COL,X
       2523 4F                        clra                            ; contador de columna
       2524 A701                      staa            COL_CONT,X
       2526 A700                      staa            KB_PRESS,X      ; variable con numero de tecla
                      
                      * loop para barrer y mirar teclado matricial
       2528 86FF      kb_loop         ldaa            #NONE
       252A 5A                        decb                            
       252B 2B2C                      bmi             kb_end          ; me fijo si recorrio todas las filas
       252D A603                      ldaa            MASK_FIL,X      ; enmascaro la siguiente fila
       252F B71003                    staa            ROWS
       2532 0D                        sec                             ; carry para rotacion de byte
       2533 49                        rola                            
       2534 A703                      staa            MASK_FIL,X      ; cambio variable mascara para la proxima fila
                      
       2536 8604                      ldaa            #LENGTH
       2538 A701                      staa            COL_CONT,X
       253A 8610                      ldaa            #COL0           ; inicializo variable para barrer
       253C A702                      staa            MASK_COL,X      ; la columna
                      
       253E 6A01      kb_col          dec             COL_CONT,X      ; me fijo si barrio todas las columnas
       2540 2BE6                      bmi             kb_loop
       2542 B61005                    ldaa            COLS            ; leo el puerto de columnas
       2545 A402                      anda            MASK_COL,X      ; enmascaro columna
       2547 2707                      beq             kb_hit          ; si es nulo apretaron un boton
       2549 48                        lsla                            ; cambio mascara para proxima columna
       254A A702                      staa            MASK_COL,X
       254C 6C00                      inc             KB_PRESS,X      ; incremento numero de tecla
       254E 20EE                      bra             kb_col
                      
       2550 A600      kb_hit          ldaa            KB_PRESS,X      ; apretaron una tecla, lo comparo
       2552 B141F7                    cmpa            kb_buff         ; con la que fue apretada antes
       2555 2602                      bne             kb_end          ; aviso por led
                      
       2557 86FF      kb_none         ldaa            #NONE           ; en caso de que no apreten nada
                      
       2559 B741F7    kb_end          staa            kb_buff         ; actualizo variable
                                              
                      * elimino variables locales
       255C                           %RELMEM         1
       255C                           IF 1 > 0
       255C 30                        tsx
       255D 8F                        xgdx
       255E C30001                    addd    #1
       2561 8F                        xgdx
       2562 35                        txs
       2563                           ENDIF           
       2563                           %RELMEM         1
       2563                           IF 1 > 0
       2563 30                        tsx
       2564 8F                        xgdx
       2565 C30001                    addd    #1
       2568 8F                        xgdx
       2569 35                        txs
       256A                           ENDIF           
       256A                           %RELMEM         1
       256A                           IF 1 > 0
       256A 30                        tsx
       256B 8F                        xgdx
       256C C30001                    addd    #1
       256F 8F                        xgdx
       2570 35                        txs
       2571                           ENDIF           
       2571                           %RELMEM         1
       2571                           IF 1 > 0
       2571 30                        tsx
       2572 8F                        xgdx
       2573 C30001                    addd    #1
       2576 8F                        xgdx
       2577 35                        txs
       2578                           ENDIF           
                      
                      * restore de registros
       2578 33                        pulb
       2579 38                        pulx
                                      
       257A 39                        rts
                      
                      *----------------------------------------------
                      * init_kb: subrutina para inicializar teclado
                      *----------------------------------------------
                      
       257B           init_kb         EQU             *
                      
                      * back up de registros
       257B 36                        psha
                      
                      * inicializacion de teclado             
       257C 86FF                      ldaa            #NONE
       257E B741F7                    staa            kb_buff
                      
                      * restore de registros
       2581 32                        pula
       2582 39                        rts
                      
                      *--------------------------------
                      * Inicializa display (y teclado)
                      *--------------------------------
                      
       2583           initdis         EQU             *
                              
       2583 36                        psha
                      
       2584 860F                      ldaa            #$0F       ;set for input/output
       2586 B71007                    staa            DDRC       ;C output
       2589 B61002                    ldaa            PIOC       ;get current value
       258C 84FE                      anda            #$FE       ;reset bit 0 to 0
       258E B71002                    staa            PIOC       ;send it out
                              
       2591 32                        pula
       2592 39                        rts
                      
                      *--------------------------------------------------------------
                      * sh_value: toma un string, desplaza los caracteres a izquierda
                      *           y agrega un nuevo caracter
                      *           IX: direccion del string
                      *           AccA: nuevo valor
                      *--------------------------------------------------------------
                      
       2593           sh_value        EQU             *
                      
                      * back up de registros
       2593 3C                        pshx
       2594 37                        pshb
                      
                      * loop para desplazar caracteres
       2595 E601      v_loop          ldab            1,X             ;tomo elemento siguiente en string
       2597 2705                      beq             v_end
       2599 E700                      stab            0,X             ;y lo guardo en posicion actual
       259B 08                        inx
       259C 20F7                      bra             v_loop          
                      
       259E A700      v_end           staa            0,X             ;ingresa nuevo caracter en cadena desplazada
                      
                      * restore de registros
       25A0 33                        pulb
       25A1 38                        pulx
       25A2 39                        rts
                      
                      *----------------------------------------------
                      * toAscii: toma numero de boton del teclado
                      *          y devuelve su correspondiente en
                      *          ascii
                      *          toma  AccB: valor del boton
                      *          devuelve AccB: ascii
                      *----------------------------------------------
                      
       25A3           to_ascii        EQU             *
                      
                      * back up de registros
       25A3 3C                        pshx
                      
                      * inicializacion de variables
       25A4 CE420D                    ldx             #kb_ascii       ; apunto a string con caracteres asignados por tecla
       25A7 3A                        abx                             ; me muevo en string con indice y recupero segun tecla presionada
       25A8 E600                      ldab            0,X             
                      
                      * restore de registros
       25AA 38                        pulx
       25AB 39                        rts
                      
                      *--------------------------------
                      *Outchar: ACCA: Carácter a enviar
                      *           ACCB: Posición (0-3)
                      *--------------------------------
       25AC           outchar         EQU             *
                              
       25AC F71005                    stab            PORTCL          ;set the position 
       25AF B71004                    staa            PORTB           ;set the character
                              
       25B2 39                        rts
                      
                      *-------------------------
                      * clr_dis: Limpia display
                      *-------------------------
                      
       25B3           clr_dis         EQU             *
                      
                      * hago back up de registros
       25B3 37                        pshb
       25B4 36                        psha
                      
                      * inicializo el contador de posiciones
       25B5 C604                      ldab            #LENGTH
                      
                      * loop para limpiar cada posicion
       25B7 5A        clrloop         decb
       25B8 2B07                      bmi             clr_end         ; si no pase por todas las posiciones
       25BA 8620                      ldaa            #ESP            ; cargo en esta posicion espacio blanco
       25BC BD25AC                    jsr             outchar         ; y lo envio
       25BF 20F6                      bra             clrloop         
                      
                      * restore de registros
       25C1 32        clr_end         pula
       25C2 33                        pulb
       25C3 39                        rts
                      
                      *-------------------------------------------------------
                      * clrarray: subrutina para limpiar un array dado
                      * parametros: IX direccion del array
                      *             AccA cantidad de elementos del array
                      *             AccB caracter a poner
                      *-------------------------------------------------------
                      
       25C4           clrarray        EQU             *
                                      
       25C4 4A        cra_loop        deca                            ;contador con cantidad de elementos a limpiar
       25C5 2705                      beq             cra_end
       25C7 E700                      stab            0,X             ;reemplazar elemento por caracter a poner
       25C9 08                        inx
       25CA 20F8                      bra             cra_loop
                                      
       25CC E700      cra_end         stab            0,X
       25CE 39                        rts
                      
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
     ytes:   1630         # main_bn #
     RC:     D345         ###########
 