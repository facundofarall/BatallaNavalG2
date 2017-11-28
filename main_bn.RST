################################################################################
#                                                                              #
#    Micro Series 6801 Assembler V2.00/DOS              28/Nov/17  10:28:44    #
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
                      * limpia display bitmap
       0000                           MACRO   %CLS
                                      
                                      ldx             #display
                                      ldaa            #ARRAY_L
                                      ldab            #$03
                      cls\@           stab            0,X
                                      inx
                                      deca
                                      bne             cls\<
                      
       0000                           ENDMAC
                      
                      * espera
       0000                           MACRO   %EXPECT
                                      
                      expe\@          jsr             read_kb
                                      tsta
                                      bmi             expe\.
                                      cmpa            #\0
                                      bne             expe\<
                      
       0000                           ENDMAC
                      
                      * soltar
       0000                           MACRO   %KEEP
                                      
                      keep\@          jsr             read_kb
                                      cmpa            #\0
                                      beq             keep\<
                      
       0000                           ENDMAC
                      * filter array auxiliar
       0000                           MACRO   %FIX_ARRY
                      
                                      ldx             SA_MAT,Y
                                      ldaa            #ARRAY_L
                      loop\@          ldab            0,X
                                      andb            #11000000b
                                      orab            #00000011b
                                      stab            0,X
                                      inx
                                      deca
                                      bne             loop\<
                      
       0000                           ENDMAC
                      
                      * clear array auxiliar
       0000                           MACRO   %CLR_AUX
                                      
                                      ldx             #arry_aux
                                      ldaa            #N_AUX*SA_SIZE
                                      ldab            #0
                                      jsr             clrarray
                      
       0000                           ENDMAC
                      
                      * para ir actualizando el display
       0000                           MACRO   %REFRESH
                      
                                      pshy
                                      ldx             SA_MAT,Y
                                      pshx
                                      clra
                                      psha
                                      ldy             #display
                                      pshy
                                      ldaa            #ARRAY_L
                                      psha
                                      jsr             mirror
                                      pula
                                      puly
                                      pula
                                      pulx
                                      puly    
                      
       0000                           ENDMAC
                      
                      * saltos que pasan el limite
       0000                           MACRO   %GOTO
                      
                                      jmp     \0
                      
       0000                           ENDMAC
                      
                      * agrega posicion al arreglo de posiciones prohibidas
                      * toma en A la fila y en B la columna
       0000                           MACRO   %PROH_POS
                                      
                                      tsta
                                      bne             oka\@
                                      ldaa            #$FF
                      oka\<           tstb
                                      bne             okb\@
                                      ldab            #$FF
                      okb\<           psha
                                      pshb
                                      ldx             #arry_pr
                                      pshx
                                      clra
                                      psha
                                      jsr             play_wrt
                                      pula
                                      pulx
                                      pulb
                                      pula
                      
       0000                           ENDMAC
                      
                      * agrega las posiciones de la periferia de una posicion al arreglo
                      * de prohibidas, tomando en A la fila y en B la columna de dicha posicion
                      
       0000                           MACRO   %PROH_PER
                                              
                                      inca
                                      %PROH_POS
                              
                                      deca
                                      incb
                                      %PROH_POS
                      
                                      deca
                                      decb
                                      %PROH_POS
                      
                                      inca
                                      decb
                                      %PROH_POS
                      
       0000                           ENDMAC
                      
                      * escribe una posicion en la matriz del tablero y toma
                      * por A como fila y B como columna de dicha posicion
                      * el parametro \0 indica que tipo de barco es el grabado
                      
       0000                           MACRO   %SET_POS
                      
                                      ldaa            #\0
                                      psha
                                      ldaa            SA_FIL,Y
                                      psha
                                      ldab            SA_COL,Y
                                      pshb
                                      ldx             SA_MAT,Y
                                      pshx
                                      clra
                                      psha
                                      jsr             writepos
                                      pula
                                      pulx
                                      pulb
                                      pula
                                      pula
                      
       0000                           ENDMAC
                      
                      * me fijo que la posicion este o no este en el arreglo de posiciones
                      * prohibidas, la A se toma como la fila y la B como la columna
                      
       0000                           MACRO   %IS_PROH
                      
                                      psha
                                      pshb
                                      ldx             #arry_pr
                                      pshx
                                      clra
                                      psha
                                      jsr             play_src
                                      pula
                                      pulx
                                      pulb
                                      pula
                      
       0000                           ENDMAC
                      
                      * agrego una posicion al arreglo auxiliar para cargueros
                      * y se toma que en A esta la fila y en B la columna
                      
       0000                           MACRO   %AUX_POS
                      
                                      psha
                                      pshb
                                      ldx             #arry_aux
                                      pshx
                                      clra
                                      psha
                                      jsr             play_wrt
                                      pula
                                      pulx
                                      pulb
                                      pula
                      
       0000                           ENDMAC
                      
                      * establezco el arreglo auxiliar con las posiciones
                      * perifericas del carguero, toma en A fila y B columna
                      * de la parte uno del carguero
                      
       0000                           MACRO   %SET_AUX
                                      
                                      inca
                                      %AUX_POS
                                      
                                      deca
                                      incb
                                      %AUX_POS
                                      
                                      deca
                                      decb
                                      %AUX_POS
                      
                                      inca
                                      decb
                                      %AUX_POS                
                      
       0000                           ENDMAC
                      
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
       2000 8E4257                    lds             #STACKP
                      
                      * INICIALIZAR DISPLAY Y TECLADO
       2003 BD298B                    jsr             initdis         ;inicialización de display 7 seg
       2006 BD2983                    jsr             init_kb         ;inicialización de teclado
                      
                      * LIMPIAR ARREGLO DE BARCOS ATACADOS
       2009 CE4284                    ldx             #hitlist
       200C 3C                        pshx                            ;se envía inicio de arreglo de barcos a poner en 0
       200D 8636                      ldaa            #(HITSIZE*2)
       200F 36                        psha                            ;se envía el tamaño del arreglo a limpiar
       2010 BD2821                    jsr             clrzone
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
       2016 BD26BE                    jsr             def_ships
       2019 B6425B                    ldaa            cant_sub        ;se carga cantidad de submarinos
       201C F6425C                    ldab            cant_shi        ;se carga cantidad de cargueros
       201F 58                        lslb                            ;se multiplica a la cantidad de cargueros por dos, así se obtiene las posiciones atacables
       2020 1B                        aba
       2021 B7425D                    staa            tot_pos         ;se carga la variable con el total de posiciones atacables
                      
                      * DEFINIR BARCOS P1
       2024 B6425C                    ldaa            cant_shi
       2027 36                        psha                            ;se envía cantidad de cargueros
       2028 B6425B                    ldaa            cant_sub
       202B 36                        psha                            ;se envía cantidad de submarinos
       202C CE5000                    ldx             #p_board
       202F 3C                        pshx                            ;se envía el inicio del tablero de P1
       2030 BD2106                    jsr             setarmy
       2033                           %CLRST          4
       0004           CONT_M          SET     4
       2033                           IF CONT_M > 0
       2033 31                        ins
       2034                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2034                           IF CONT_M > 0
       2034 31                        ins
       2035                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2035                           IF CONT_M > 0
       2035 31                        ins
       2036                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2036                           IF CONT_M > 0
       2036 31                        ins
       2037                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2037                           ENDIF
       2037                           ENDIF
       2037                           ENDIF
       2037                           ENDIF
       2037                           ENDIF
                      
                      * DEFINIR BARCOS P2
       2037 B6425C                    ldaa            cant_shi
       203A 36                        psha                            ;se envía cantidad de cargueros
       203B B6425B                    ldaa            cant_sub
       203E 36                        psha                            ;se envía cantidad de submarinos
       203F CE5000                    ldx             #p_board        ;se toma la dirección al tablero de P1
       2042 C624                      ldab            #ARRAY_L
       2044 3A                        abx                             ;se desfasa al tablero de P2
       2045 3C                        pshx                            ;se envía el inicio del tablero de P2
       2046 BD2106                    jsr             setarmy
       2049                           %CLRST          4
       0004           CONT_M          SET     4
       2049                           IF CONT_M > 0
       2049 31                        ins
       204A                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       204A                           IF CONT_M > 0
       204A 31                        ins
       204B                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       204B                           IF CONT_M > 0
       204B 31                        ins
       204C                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       204C                           IF CONT_M > 0
       204C 31                        ins
       204D                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       204D                           ENDIF
       204D                           ENDIF
       204D                           ENDIF
       204D                           ENDIF
       204D                           ENDIF
                      
       204D           main            EQU             *
                      * ELEGIR JUGADOR
       204D BD2839                    jsr             whoplays        ;se recibirá en AccA el jugador a atacar a continuación
       2050 B74258                    staa            player          ;se habilita la siguiente jugada
                      
                      * ENVIAR TABLERO DE JUGADOR A ATACAR A DISPLAY
       2053 CE5000                    ldx             #p_board
       2056 3C                        pshx                            ;se envía el inicio de las matrices de tableros
       2057 B64258                    ldaa            player
       205A 36                        psha                            ;se envía el jugador atacado
       205B CE5F00                    ldx             #display
       205E 3C                        pshx                            ;se envía la matriz de display (destino a copiar)
       205F 8624                      ldaa            #ARRAY_L
       2061 36                        psha                            ;se envía el tamaño de la matriz tablero
       2062 BD2715                    jsr             mirror          ;se envía el tablero del jugador atacado al display
       2065                           %CLRST          6
       0006           CONT_M          SET     6
       2065                           IF CONT_M > 0
       2065 31                        ins
       2066                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2066                           IF CONT_M > 0
       2066 31                        ins
       2067                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2067                           IF CONT_M > 0
       2067 31                        ins
       2068                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2068                           IF CONT_M > 0
       2068 31                        ins
       2069                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2069                           IF CONT_M > 0
       2069 31                        ins
       206A                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       206A                           IF CONT_M > 0
       206A 31                        ins
       206B                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       206B                           ENDIF
       206B                           ENDIF
       206B                           ENDIF
       206B                           ENDIF
       206B                           ENDIF
       206B                           ENDIF
       206B                           ENDIF
                      
                      * INGRESAR POSICIÓN A ATACAR
       206B BD284E                    jsr             input           ;se llama a la subrutina de interfaz de usuario
       206E B74259                    staa            letra_p         ;se recibe en AccA la letra jugada (fila) y en AccB el número (columna)
       2071 F7425A                    stab            num_p           ;se guarda la jugada
                      
                      * VERIFICAR QUE NO SEA POSICIÓN DE BARCO YA ATACADA
       2074 B64259                    ldaa            letra_p
       2077 36                        psha                            ;se envía la letra jugada
       2078 F6425A                    ldab            num_p
       207B 37                        pshb                            ;se envía el número jugado
       207C CE4284                    ldx             #hitlist
       207F 3C                        pshx                            ;se envía la posición del primer arreglo de barcos tocados
       2080 B64258                    ldaa            player
       2083 36                        psha                            ;se envía el jugador a atacar
       2084 BD27A0                    jsr             play_src        ;se recibirá el carry en 1 si la posición corresponde a un barco ya atacado
       2087                           %CLRST          5               ;se limpia el stack
       0005           CONT_M          SET     5
       2087                           IF CONT_M > 0
       2087 31                        ins
       2088                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2088                           IF CONT_M > 0
       2088 31                        ins
       2089                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2089                           IF CONT_M > 0
       2089 31                        ins
       208A                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       208A                           IF CONT_M > 0
       208A 31                        ins
       208B                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       208B                           IF CONT_M > 0
       208B 31                        ins
       208C                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       208C                           ENDIF
       208C                           ENDIF
       208C                           ENDIF
       208C                           ENDIF
       208C                           ENDIF
       208C                           ENDIF
       208C 25BF                      bcs             main            ;si la posición ya fue jugada, se pierde el turno y pasa al siguiente
                      
                      * MANDAR QUE JUGADOR ESTÁ JUGANDO AL DISPLAY
       208E B64258                    ldaa            player
       2091 36                        psha                            ;se envía el jugador que está siendo atacado
       2092 BD29F5                    jsr             turn
       2095                           %CLRST          1
       0001           CONT_M          SET     1
       2095                           IF CONT_M > 0
       2095 31                        ins
       2096                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2096                           ENDIF
       2096                           ENDIF
                      
                      * BUSCAR EL BYTE ATACADO EN EL TABLERO DEL JUGADOR ATACADO
       2096 B64259                    ldaa            letra_p
       2099 36                        psha                            ;se envía le letra jugada
       209A F6425A                    ldab            num_p
       209D 37                        pshb                            ;se envía el número jugado
       209E CE5000                    ldx             #p_board
       20A1 3C                        pshx                            ;se envía la dirección de inicio de los arreglos de tablero
       20A2 F64258                    ldab            player
       20A5 37                        pshb                            ;se envía el jugador a atacar
       20A6 BD273E                    jsr             fetchpos
       20A9                           %CLRST          4               ;se limpia el stack
       0004           CONT_M          SET     4
       20A9                           IF CONT_M > 0
       20A9 31                        ins
       20AA                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       20AA                           IF CONT_M > 0
       20AA 31                        ins
       20AB                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       20AB                           IF CONT_M > 0
       20AB 31                        ins
       20AC                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       20AC                           IF CONT_M > 0
       20AC 31                        ins
       20AD                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       20AD                           ENDIF
       20AD                           ENDIF
       20AD                           ENDIF
       20AD                           ENDIF
       20AD                           ENDIF
       20AD 32                        pula                            ;se toma del stack el dato del byte jugado
                      
                      * ATACAR
       20AE CE4284                    ldx             #hitlist
       20B1 3C                        pshx                            ;se envía inicio del arreglo de barcos atacados
       20B2 36                        psha                            ;se envía el byte a jugado
       20B3 B64259                    ldaa            letra_p
       20B6 36                        psha                            ;se envía le letra jugada
       20B7 F6425A                    ldab            num_p
       20BA 37                        pshb                            ;se envía el número jugado
       20BB CE5000                    ldx             #p_board
       20BE 3C                        pshx                            ;se envía la dirección de inicio de los arreglos de tablero
       20BF F64258                    ldab            player
       20C2 37                        pshb                            ;se envía el jugador a atacar
       20C3 BD2456                    jsr             gameplay
       20C6                           %CLRST          8               ;se limpia el stack
       0008           CONT_M          SET     8
       20C6                           IF CONT_M > 0
       20C6 31                        ins
       20C7                           %CLRST  CONT_M-1
       0007           CONT_M          SET     CONT_M-1
       20C7                           IF CONT_M > 0
       20C7 31                        ins
       20C8                           %CLRST  CONT_M-1
       0006           CONT_M          SET     CONT_M-1
       20C8                           IF CONT_M > 0
       20C8 31                        ins
       20C9                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       20C9                           IF CONT_M > 0
       20C9 31                        ins
       20CA                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       20CA                           IF CONT_M > 0
       20CA 31                        ins
       20CB                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       20CB                           IF CONT_M > 0
       20CB 31                        ins
       20CC                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       20CC                           IF CONT_M > 0
       20CC 31                        ins
       20CD                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       20CD                           IF CONT_M > 0
       20CD 31                        ins
       20CE                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       20CE                           ENDIF
       20CE                           ENDIF
       20CE                           ENDIF
       20CE                           ENDIF
       20CE                           ENDIF
       20CE                           ENDIF
       20CE                           ENDIF
       20CE                           ENDIF
       20CE                           ENDIF
                      
                      * ACTUALIZAR DISPLAY CON LA POSICIÓN YA ATACADA
       20CE CE5000                    ldx             #p_board
       20D1 3C                        pshx                            ;se envía el inicio de las matrices de tableros
       20D2 B64258                    ldaa            player
       20D5 36                        psha                            ;se envía el jugador atacado
       20D6 CE5F00                    ldx             #display
       20D9 3C                        pshx                            ;se envía la matriz de display (destino a copiar)
       20DA 8624                      ldaa            #ARRAY_L
       20DC 36                        psha                            ;se envía el tamaño de la matriz tablero
       20DD BD2715                    jsr             mirror          ;se envía el tablero del jugador atacado al display
       20E0                           %CLRST          6
       0006           CONT_M          SET     6
       20E0                           IF CONT_M > 0
       20E0 31                        ins
       20E1                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       20E1                           IF CONT_M > 0
       20E1 31                        ins
       20E2                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       20E2                           IF CONT_M > 0
       20E2 31                        ins
       20E3                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       20E3                           IF CONT_M > 0
       20E3 31                        ins
       20E4                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       20E4                           IF CONT_M > 0
       20E4 31                        ins
       20E5                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       20E5                           IF CONT_M > 0
       20E5 31                        ins
       20E6                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       20E6                           ENDIF
       20E6                           ENDIF
       20E6                           ENDIF
       20E6                           ENDIF
       20E6                           ENDIF
       20E6                           ENDIF
       20E6                           ENDIF
                      
                      * CONTAR BARCOS ATACADOS A VER SI EL JUGADOR GANÓ               
       20E6 CE4284                    ldx             #hitlist
       20E9 3C                        pshx                            ;se envía la posición del primer arreglo de barcos tocados
       20EA B64258                    ldaa            player
       20ED 36                        psha                            ;se envía el jugador a atacar
       20EE BD27D3                    jsr             play_cnt
       20F1                           %CLRST          2
       0002           CONT_M          SET     2
       20F1                           IF CONT_M > 0
       20F1 31                        ins
       20F2                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       20F2                           IF CONT_M > 0
       20F2 31                        ins
       20F3                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       20F3                           ENDIF
       20F3                           ENDIF
       20F3                           ENDIF
       20F3 32                        pula                            ;se recibe la cantidad de bytes contados
       20F4 B1425D                    cmpa            tot_pos         
       20F7 2703                      beq             end_game        ;si se llegó al máximo de barcos, se termina el juego
                      
       20F9 7E204D                    jmp             main
                      
                      * MANDAR MENSAJE A DISPLAY DE QUIÉN GANÓ
       20FC           end_game        EQU             *
       20FC B64258                    ldaa            player
       20FF 36                        psha
       2100 BD29D7                    jsr             winner
       2103                           %CLRST          1
       0001           CONT_M          SET     1
       2103                           IF CONT_M > 0
       2103 31                        ins
       2104                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2104                           ENDIF
       2104                           ENDIF
                      
       2104           end_loop        EQU             *
       2104 20FE                      bra             end_loop
                      
                      **************
                      * SUBRUTINES *
                      **************
                      
                      *--------------------------------------------
                      * setarmy: subrutina para definir los barcos
                      *          de cada usuario
                      *
                      * parametros: Push cantidad de cargueros
                      *             Push cantidad de submarinos
                      *             Push direccion matriz tablero
                      *--------------------------------------------
                      
       0003           CH_TYPE         EQU             3
                      
       0002           SA_SIZE         EQU             2
       0064           N_PR            EQU             100
       0004           N_AUX           EQU             4
                      
       0000           SA_FIL          EQU             0
       0001           SA_COL          EQU             1
       0002           SA_CARG         EQU             2
       0003           SA_CONT         EQU             3
       0004           SA_TOT          EQU             4
       000D           SA_MAT          EQU             13
       000F           SA_CSUB         EQU             15
       0010           SA_CSHI         EQU             16
                      
       0040           BY_SUB          EQU             01000000b
       0080           BY_SHI          EQU             10000000b
                      
       2106           setarmy         EQU             *
                      
                      * back up de registros
       2106 36                        psha
       2107 37                        pshb
       2108 3C                        pshx
       2109 183C                      pshy
                      
                      * variables locales
       210B 36                        psha
       210C 36                        psha
       210D 36                        psha
       210E 36                        psha
       210F 36                        psha
                      
                      * inializo el framepointer
       2110 1830                      tsy
                      
                      * inicializo los contadores
       2112 186F03                    clr             SA_CONT,Y
       2115 186F02                    clr             SA_CARG,Y
       2118 18A60F                    ldaa            SA_CSUB,Y
       211B 18E610                    ldab            SA_CSHI,Y
       211E 1B                        aba
       211F 18A704                    staa            SA_TOT,Y
                      
                      * inicializo arreglos
       2122 CE42D8                    ldx             #arry_pr
       2125 86C8                      ldaa            #N_PR*SA_SIZE
       2127 C600                      ldab            #0
       2129 BD29CC                    jsr             clrarray
                      
       212C                           %CLR_AUX
                                      
       212C CE43A0                    ldx             #arry_aux
       212F 8608                      ldaa            #N_AUX*SA_SIZE
       2131 C600                      ldab            #0
       2133 BD29CC                    jsr             clrarray
                      
                      
                      * termine de entrar los barcos?
       2136 18A603    sa_check        ldaa            SA_CONT,Y
       2139 18A104                    cmpa            SA_TOT,Y
       213C 2603                      bne             sa_show
       213E                           %GOTO           sa_end  
                      
       213E 7E242D                    jmp     sa_end
                      
                      
                      * display mensaje para indicar barco
       2141 18A10F    sa_show         cmpa            SA_CSUB,Y
       2144 240C                      bhs             show_shi
       2146 CE43A8    show_sub        ldx             #msg_sub
       2149 8605                      ldaa            #5
       214B 18E603                    ldab            SA_CONT,Y
       214E 3D                        mul
       214F 3A                        abx
       2150 200F                      bra             show_jmp
       2152 CE43C1    show_shi        ldx             #msg_shi
       2155 18A603                    ldaa            SA_CONT,Y
       2158 18E60F                    ldab            SA_CSUB,Y
       215B 10                        sba
       215C 16                        tab
       215D 8605                      ldaa            #5
       215F 3D                        mul
       2160 3A                        abx
       2161 BD28EE    show_jmp        jsr             show
       2164                           %EXPECT         B_reset
                                      
       2164 BD2902    expe0000          jsr             read_kb
       2167 4D                        tsta
       2168 2BFA                      bmi             expe0000
       216A 810F                      cmpa            #B_reset
       216C 26F6                      bne             expe0000
                      
                      
                      * espera ingreso de coordenada
       216E BD284E    sa_again        jsr             input
                      
       2171 18A700                    staa            SA_FIL,Y
       2174 18E701                    stab            SA_COL,Y
                      
                      * verifico que no sea una posicion prohibida
       2177                           %IS_PROH
                      
       2177 36                        psha
       2178 37                        pshb
       2179 CE42D8                    ldx             #arry_pr
       217C 3C                        pshx
       217D 4F                        clra
       217E 36                        psha
       217F BD27A0                    jsr             play_src
       2182 32                        pula
       2183 38                        pulx
       2184 33                        pulb
       2185 32                        pula
                      
       2186 25E6                      bcs             sa_again
                      
                      * me fijo que tipo de barco se ingresa
       2188 36                        psha
       2189 18A603                    ldaa            SA_CONT,Y
       218C 18A10F                    cmpa            SA_CSUB,Y
       218F 32                        pula
       2190 2503                      blo             sa_sub
       2192                           %GOTO           sa_shi
                      
       2192 7E2253                    jmp     sa_shi
                      
                      
                      * escribo en matriz el submarino
       2195           sa_sub          %PROH_POS
                                      
       2195 4D                        tsta
       2196 2602                      bne             oka0001
       2198 86FF                      ldaa            #$FF
       219A 5D        oka0001           tstb
       219B 2602                      bne             okb0002
       219D C6FF                      ldab            #$FF
       219F 36        okb0002           psha
       21A0 37                        pshb
       21A1 CE42D8                    ldx             #arry_pr
       21A4 3C                        pshx
       21A5 4F                        clra
       21A6 36                        psha
       21A7 BD27F7                    jsr             play_wrt
       21AA 32                        pula
       21AB 38                        pulx
       21AC 33                        pulb
       21AD 32                        pula
                      
       21AE                           %PROH_PER
                                              
       21AE 4C                        inca
       21AF                           %PROH_POS
                                      
       21AF 4D                        tsta
       21B0 2602                      bne             oka0003
       21B2 86FF                      ldaa            #$FF
       21B4 5D        oka0003           tstb
       21B5 2602                      bne             okb0004
       21B7 C6FF                      ldab            #$FF
       21B9 36        okb0004           psha
       21BA 37                        pshb
       21BB CE42D8                    ldx             #arry_pr
       21BE 3C                        pshx
       21BF 4F                        clra
       21C0 36                        psha
       21C1 BD27F7                    jsr             play_wrt
       21C4 32                        pula
       21C5 38                        pulx
       21C6 33                        pulb
       21C7 32                        pula
                      
                              
       21C8 4A                        deca
       21C9 5C                        incb
       21CA                           %PROH_POS
                                      
       21CA 4D                        tsta
       21CB 2602                      bne             oka0005
       21CD 86FF                      ldaa            #$FF
       21CF 5D        oka0005           tstb
       21D0 2602                      bne             okb0006
       21D2 C6FF                      ldab            #$FF
       21D4 36        okb0006           psha
       21D5 37                        pshb
       21D6 CE42D8                    ldx             #arry_pr
       21D9 3C                        pshx
       21DA 4F                        clra
       21DB 36                        psha
       21DC BD27F7                    jsr             play_wrt
       21DF 32                        pula
       21E0 38                        pulx
       21E1 33                        pulb
       21E2 32                        pula
                      
                      
       21E3 4A                        deca
       21E4 5A                        decb
       21E5                           %PROH_POS
                                      
       21E5 4D                        tsta
       21E6 2602                      bne             oka0007
       21E8 86FF                      ldaa            #$FF
       21EA 5D        oka0007           tstb
       21EB 2602                      bne             okb0008
       21ED C6FF                      ldab            #$FF
       21EF 36        okb0008           psha
       21F0 37                        pshb
       21F1 CE42D8                    ldx             #arry_pr
       21F4 3C                        pshx
       21F5 4F                        clra
       21F6 36                        psha
       21F7 BD27F7                    jsr             play_wrt
       21FA 32                        pula
       21FB 38                        pulx
       21FC 33                        pulb
       21FD 32                        pula
                      
                      
       21FE 4C                        inca
       21FF 5A                        decb
       2200                           %PROH_POS
                                      
       2200 4D                        tsta
       2201 2602                      bne             oka0009
       2203 86FF                      ldaa            #$FF
       2205 5D        oka0009           tstb
       2206 2602                      bne             okb0010
       2208 C6FF                      ldab            #$FF
       220A 36        okb0010           psha
       220B 37                        pshb
       220C CE42D8                    ldx             #arry_pr
       220F 3C                        pshx
       2210 4F                        clra
       2211 36                        psha
       2212 BD27F7                    jsr             play_wrt
       2215 32                        pula
       2216 38                        pulx
       2217 33                        pulb
       2218 32                        pula
                      
                      
       2219                           %SET_POS        BY_SUB
                      
       2219 8640                      ldaa            #BY_SUB
       221B 36                        psha
       221C 18A600                    ldaa            SA_FIL,Y
       221F 36                        psha
       2220 18E601                    ldab            SA_COL,Y
       2223 37                        pshb
       2224 CDEE0D                    ldx             SA_MAT,Y
       2227 3C                        pshx
       2228 4F                        clra
       2229 36                        psha
       222A BD276F                    jsr             writepos
       222D 32                        pula
       222E 38                        pulx
       222F 33                        pulb
       2230 32                        pula
       2231 32                        pula
                      
       2232 186C03                    inc             SA_CONT,Y
       2235                           %REFRESH
                      
       2235 183C                      pshy
       2237 CDEE0D                    ldx             SA_MAT,Y
       223A 3C                        pshx
       223B 4F                        clra
       223C 36                        psha
       223D 18CE5F00                  ldy             #display
       2241 183C                      pshy
       2243 8624                      ldaa            #ARRAY_L
       2245 36                        psha
       2246 BD2715                    jsr             mirror
       2249 32                        pula
       224A 1838                      puly
       224C 32                        pula
       224D 38                        pulx
       224E 1838                      puly    
                      
       2250                           %GOTO           sa_check
                      
       2250 7E2136                    jmp     sa_check
                      
                      
       2253 186D02    sa_shi          tst             SA_CARG,Y
       2256 2703                      beq             sa_part1
       2258                           %GOTO           sa_part2
                      
       2258 7E230D                    jmp     sa_part2
                      
                      
       225B           sa_part1        %SET_AUX
                                      
       225B 4C                        inca
       225C                           %AUX_POS
                      
       225C 36                        psha
       225D 37                        pshb
       225E CE43A0                    ldx             #arry_aux
       2261 3C                        pshx
       2262 4F                        clra
       2263 36                        psha
       2264 BD27F7                    jsr             play_wrt
       2267 32                        pula
       2268 38                        pulx
       2269 33                        pulb
       226A 32                        pula
                      
                                      
       226B 4A                        deca
       226C 5C                        incb
       226D                           %AUX_POS
                      
       226D 36                        psha
       226E 37                        pshb
       226F CE43A0                    ldx             #arry_aux
       2272 3C                        pshx
       2273 4F                        clra
       2274 36                        psha
       2275 BD27F7                    jsr             play_wrt
       2278 32                        pula
       2279 38                        pulx
       227A 33                        pulb
       227B 32                        pula
                      
                                      
       227C 4A                        deca
       227D 5A                        decb
       227E                           %AUX_POS
                      
       227E 36                        psha
       227F 37                        pshb
       2280 CE43A0                    ldx             #arry_aux
       2283 3C                        pshx
       2284 4F                        clra
       2285 36                        psha
       2286 BD27F7                    jsr             play_wrt
       2289 32                        pula
       228A 38                        pulx
       228B 33                        pulb
       228C 32                        pula
                      
                      
       228D 4C                        inca
       228E 5A                        decb
       228F                           %AUX_POS                
                      
       228F 36                        psha
       2290 37                        pshb
       2291 CE43A0                    ldx             #arry_aux
       2294 3C                        pshx
       2295 4F                        clra
       2296 36                        psha
       2297 BD27F7                    jsr             play_wrt
       229A 32                        pula
       229B 38                        pulx
       229C 33                        pulb
       229D 32                        pula
                      
                      
                                      
       229E 183C                      pshy
       22A0 18CE43A0                  ldy             #arry_aux
       22A4 C604                      ldab            #N_AUX
       22A6 5A        sa_loop         decb
       22A7 2B1F                      bmi             sa_notok
       22A9 37                        pshb
       22AA 18A600                    ldaa            0,Y
       22AD 18E601                    ldab            1,Y
       22B0                           %IS_PROH
                      
       22B0 36                        psha
       22B1 37                        pshb
       22B2 CE42D8                    ldx             #arry_pr
       22B5 3C                        pshx
       22B6 4F                        clra
       22B7 36                        psha
       22B8 BD27A0                    jsr             play_src
       22BB 32                        pula
       22BC 38                        pulx
       22BD 33                        pulb
       22BE 32                        pula
                      
       22BF 33                        pulb
       22C0 240B                      bcc             sa_ok
       22C2 1808                      iny
       22C4 1808                      iny
       22C6 20DE                      bra             sa_loop
                      
       22C8 1838      sa_notok        puly
       22CA                           %GOTO           sa_again
                      
       22CA 7E216E                    jmp     sa_again
                      
                      
       22CD 1838      sa_ok           puly
       22CF 18A600                    ldaa            SA_FIL,Y
       22D2 18E601                    ldab            SA_COL,Y
       22D5                           %PROH_POS
                                      
       22D5 4D                        tsta
       22D6 2602                      bne             oka0011
       22D8 86FF                      ldaa            #$FF
       22DA 5D        oka0011           tstb
       22DB 2602                      bne             okb0012
       22DD C6FF                      ldab            #$FF
       22DF 36        okb0012           psha
       22E0 37                        pshb
       22E1 CE42D8                    ldx             #arry_pr
       22E4 3C                        pshx
       22E5 4F                        clra
       22E6 36                        psha
       22E7 BD27F7                    jsr             play_wrt
       22EA 32                        pula
       22EB 38                        pulx
       22EC 33                        pulb
       22ED 32                        pula
                      
       22EE 186C02                    inc             SA_CARG,Y
       22F1                           %SET_POS        BY_SHI
                      
       22F1 8680                      ldaa            #BY_SHI
       22F3 36                        psha
       22F4 18A600                    ldaa            SA_FIL,Y
       22F7 36                        psha
       22F8 18E601                    ldab            SA_COL,Y
       22FB 37                        pshb
       22FC CDEE0D                    ldx             SA_MAT,Y
       22FF 3C                        pshx
       2300 4F                        clra
       2301 36                        psha
       2302 BD276F                    jsr             writepos
       2305 32                        pula
       2306 38                        pulx
       2307 33                        pulb
       2308 32                        pula
       2309 32                        pula
                      
       230A                           %GOTO           sa_again
                      
       230A 7E216E                    jmp     sa_again
                      
       230D CE43A0    sa_part2        ldx             #arry_aux
       2310 8604                      ldaa            #N_AUX
       2312 4A        sa_verf         deca
       2313 2A03                      bpl             sa_chk          
       2315                           %GOTO           sa_again
                      
       2315 7E216E                    jmp     sa_again
                      
       2318 E600      sa_chk          ldab            0,X
       231A 18E100                    cmpb            SA_FIL,Y
       231D 2609                      bne             sa_not
       231F E601                      ldab            1,X
       2321 18E101                    cmpb            SA_COL,Y
       2324 2602                      bne             sa_not
       2326 2004                      bra             sa_good
       2328 08        sa_not          inx
       2329 08                        inx
       232A 20E6                      bra             sa_verf
                      
       232C 183C      sa_good         pshy
       232E 18CE43A0                  ldy             #arry_aux
       2332 C604                      ldab            #4
       2334 5A        sa_loop2        decb
       2335 2B23                      bmi             sa_done
       2337 37                        pshb
       2338 18A600                    ldaa            0,Y
       233B 18E601                    ldab            1,Y
       233E                           %PROH_POS
                                      
       233E 4D                        tsta
       233F 2602                      bne             oka0013
       2341 86FF                      ldaa            #$FF
       2343 5D        oka0013           tstb
       2344 2602                      bne             okb0014
       2346 C6FF                      ldab            #$FF
       2348 36        okb0014           psha
       2349 37                        pshb
       234A CE42D8                    ldx             #arry_pr
       234D 3C                        pshx
       234E 4F                        clra
       234F 36                        psha
       2350 BD27F7                    jsr             play_wrt
       2353 32                        pula
       2354 38                        pulx
       2355 33                        pulb
       2356 32                        pula
                      
       2357 33                        pulb
       2358 20DA                      bra             sa_loop2
                      
       235A 1838      sa_done         puly
       235C 18A600                    ldaa            SA_FIL,Y
       235F 18E601                    ldab            SA_COL,Y
       2362                           %PROH_POS
                                      
       2362 4D                        tsta
       2363 2602                      bne             oka0015
       2365 86FF                      ldaa            #$FF
       2367 5D        oka0015           tstb
       2368 2602                      bne             okb0016
       236A C6FF                      ldab            #$FF
       236C 36        okb0016           psha
       236D 37                        pshb
       236E CE42D8                    ldx             #arry_pr
       2371 3C                        pshx
       2372 4F                        clra
       2373 36                        psha
       2374 BD27F7                    jsr             play_wrt
       2377 32                        pula
       2378 38                        pulx
       2379 33                        pulb
       237A 32                        pula
                      
       237B                           %PROH_PER
                                              
       237B 4C                        inca
       237C                           %PROH_POS
                                      
       237C 4D                        tsta
       237D 2602                      bne             oka0017
       237F 86FF                      ldaa            #$FF
       2381 5D        oka0017           tstb
       2382 2602                      bne             okb0018
       2384 C6FF                      ldab            #$FF
       2386 36        okb0018           psha
       2387 37                        pshb
       2388 CE42D8                    ldx             #arry_pr
       238B 3C                        pshx
       238C 4F                        clra
       238D 36                        psha
       238E BD27F7                    jsr             play_wrt
       2391 32                        pula
       2392 38                        pulx
       2393 33                        pulb
       2394 32                        pula
                      
                              
       2395 4A                        deca
       2396 5C                        incb
       2397                           %PROH_POS
                                      
       2397 4D                        tsta
       2398 2602                      bne             oka0019
       239A 86FF                      ldaa            #$FF
       239C 5D        oka0019           tstb
       239D 2602                      bne             okb0020
       239F C6FF                      ldab            #$FF
       23A1 36        okb0020           psha
       23A2 37                        pshb
       23A3 CE42D8                    ldx             #arry_pr
       23A6 3C                        pshx
       23A7 4F                        clra
       23A8 36                        psha
       23A9 BD27F7                    jsr             play_wrt
       23AC 32                        pula
       23AD 38                        pulx
       23AE 33                        pulb
       23AF 32                        pula
                      
                      
       23B0 4A                        deca
       23B1 5A                        decb
       23B2                           %PROH_POS
                                      
       23B2 4D                        tsta
       23B3 2602                      bne             oka0021
       23B5 86FF                      ldaa            #$FF
       23B7 5D        oka0021           tstb
       23B8 2602                      bne             okb0022
       23BA C6FF                      ldab            #$FF
       23BC 36        okb0022           psha
       23BD 37                        pshb
       23BE CE42D8                    ldx             #arry_pr
       23C1 3C                        pshx
       23C2 4F                        clra
       23C3 36                        psha
       23C4 BD27F7                    jsr             play_wrt
       23C7 32                        pula
       23C8 38                        pulx
       23C9 33                        pulb
       23CA 32                        pula
                      
                      
       23CB 4C                        inca
       23CC 5A                        decb
       23CD                           %PROH_POS
                                      
       23CD 4D                        tsta
       23CE 2602                      bne             oka0023
       23D0 86FF                      ldaa            #$FF
       23D2 5D        oka0023           tstb
       23D3 2602                      bne             okb0024
       23D5 C6FF                      ldab            #$FF
       23D7 36        okb0024           psha
       23D8 37                        pshb
       23D9 CE42D8                    ldx             #arry_pr
       23DC 3C                        pshx
       23DD 4F                        clra
       23DE 36                        psha
       23DF BD27F7                    jsr             play_wrt
       23E2 32                        pula
       23E3 38                        pulx
       23E4 33                        pulb
       23E5 32                        pula
                      
                      
       23E6                           %SET_POS        BY_SHI
                      
       23E6 8680                      ldaa            #BY_SHI
       23E8 36                        psha
       23E9 18A600                    ldaa            SA_FIL,Y
       23EC 36                        psha
       23ED 18E601                    ldab            SA_COL,Y
       23F0 37                        pshb
       23F1 CDEE0D                    ldx             SA_MAT,Y
       23F4 3C                        pshx
       23F5 4F                        clra
       23F6 36                        psha
       23F7 BD276F                    jsr             writepos
       23FA 32                        pula
       23FB 38                        pulx
       23FC 33                        pulb
       23FD 32                        pula
       23FE 32                        pula
                      
       23FF 186C03                    inc             SA_CONT,Y
       2402 186F02                    clr             SA_CARG,Y
       2405                           %CLR_AUX
                                      
       2405 CE43A0                    ldx             #arry_aux
       2408 8608                      ldaa            #N_AUX*SA_SIZE
       240A C600                      ldab            #0
       240C BD29CC                    jsr             clrarray
                      
       240F                           %REFRESH
                      
       240F 183C                      pshy
       2411 CDEE0D                    ldx             SA_MAT,Y
       2414 3C                        pshx
       2415 4F                        clra
       2416 36                        psha
       2417 18CE5F00                  ldy             #display
       241B 183C                      pshy
       241D 8624                      ldaa            #ARRAY_L
       241F 36                        psha
       2420 BD2715                    jsr             mirror
       2423 32                        pula
       2424 1838                      puly
       2426 32                        pula
       2427 38                        pulx
       2428 1838                      puly    
                      
       242A                           %GOTO           sa_check
                      
       242A 7E2136                    jmp     sa_check
                      
                      
                      * libero variables temporales
       242D           sa_end          %FIX_ARRY
                      
       242D CDEE0D                    ldx             SA_MAT,Y
       2430 8624                      ldaa            #ARRAY_L
       2432 E600      loop0025          ldab            0,X
       2434 C4C0                      andb            #11000000b
       2436 CA03                      orab            #00000011b
       2438 E700                      stab            0,X
       243A 08                        inx
       243B 4A                        deca
       243C 26F4                      bne             loop0025
                      
       243E                           %CLS
                                      
       243E CE5F00                    ldx             #display
       2441 8624                      ldaa            #ARRAY_L
       2443 C603                      ldab            #$03
       2445 E700      cls0026           stab            0,X
       2447 08                        inx
       2448 4A                        deca
       2449 26FA                      bne             cls0026
                      
       244B 32                        pula
       244C 32                        pula
       244D 32                        pula
       244E 32                        pula
       244F 32                        pula
                      
                      * restore de registros
       2450 1838                      puly
       2452 38                        pulx
       2453 33                        pulb
       2454 32                        pula
                      
       2455 39                        rts
                      
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
       2456           gameplay        EQU             *
       2456                           %BACKUP
       2456 36                        psha
       2457 37                        pshb
       2458 3C                        pshx
       2459 183C                      pshy
                                      
       245B 30                        tsx                             ;se usa el IX como frame pointer
       245C A60D                      ldaa            13,x            ;se carga en AccA el byte a analizar
       245E 84C0                      anda            #11000000b      ;se enmascan los bits 0 a 5, para analizar los más significativos
                      
       2460           gp_water        EQU             *
       2460 2620                      bne             gp_sub          ;se estudia si era agua
       2462                           %WRITEPOS       #WATER          ;se cambia el color en el tablero
       2462 C608                      ldab            #WATER
       2464 37                        pshb                            ;se envía el byte a escribir
       2465 E60C                      ldab            12,x
       2467 37                        pshb                            ;se envía letra (fila)
       2468 E60B                      ldab            11,x
       246A 37                        pshb                            ;se envía num (columna)
       246B 1AEE09                    ldy             9,x
       246E 183C                      pshy                            ;se envía dirección de inicio de matriz
       2470 E608                      ldab            8,x
       2472 37                        pshb                            ;se envía jugador atacado
       2473 BD276F                    jsr             writepos
       2476                           %CLRST          6
       0006           CONT_M          SET     6
       2476                           IF CONT_M > 0
       2476 31                        ins
       2477                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2477                           IF CONT_M > 0
       2477 31                        ins
       2478                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2478                           IF CONT_M > 0
       2478 31                        ins
       2479                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2479                           IF CONT_M > 0
       2479 31                        ins
       247A                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       247A                           IF CONT_M > 0
       247A 31                        ins
       247B                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       247B                           IF CONT_M > 0
       247B 31                        ins
       247C                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       247C                           ENDIF
       247C                           ENDIF
       247C                           ENDIF
       247C                           ENDIF
       247C                           ENDIF
       247C                           ENDIF
       247C                           ENDIF
       247C                           %RESTORE
       247C 1838                      puly
       247E 38                        pulx
       247F 33                        pulb
       2480 32                        pula
       2481 39                        rts
                      
       2482           gp_sub          EQU             *
       2482 8140                      cmpa            #01000000b      ;se estudia si era un submarino
       2484 2636                      bne             gp_shi_r
       2486                           %WRITEPOS       #SUNK_SUB       ;se cambia el color en el tablero
       2486 C650                      ldab            #SUNK_SUB
       2488 37                        pshb                            ;se envía el byte a escribir
       2489 E60C                      ldab            12,x
       248B 37                        pshb                            ;se envía letra (fila)
       248C E60B                      ldab            11,x
       248E 37                        pshb                            ;se envía num (columna)
       248F 1AEE09                    ldy             9,x
       2492 183C                      pshy                            ;se envía dirección de inicio de matriz
       2494 E608                      ldab            8,x
       2496 37                        pshb                            ;se envía jugador atacado
       2497 BD276F                    jsr             writepos
       249A                           %CLRST          6
       0006           CONT_M          SET     6
       249A                           IF CONT_M > 0
       249A 31                        ins
       249B                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       249B                           IF CONT_M > 0
       249B 31                        ins
       249C                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       249C                           IF CONT_M > 0
       249C 31                        ins
       249D                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       249D                           IF CONT_M > 0
       249D 31                        ins
       249E                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       249E                           IF CONT_M > 0
       249E 31                        ins
       249F                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       249F                           IF CONT_M > 0
       249F 31                        ins
       24A0                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       24A0                           ENDIF
       24A0                           ENDIF
       24A0                           ENDIF
       24A0                           ENDIF
       24A0                           ENDIF
       24A0                           ENDIF
       24A0                           ENDIF
       24A0                           %PLAY_WRT                       ;se agraga la posición al array de barcos atacados
       24A0 E60C                      ldab            12,x
       24A2 37                        pshb                            ;se envía letra (fila)
       24A3 E60B                      ldab            11,x
       24A5 37                        pshb                            ;se envía num (columna)
       24A6 1AEE0E                    ldy             14,x
       24A9 183C                      pshy                            ;se envía inicio de array
       24AB E608                      ldab            8,x
       24AD 37                        pshb                            ;se envía jugador atacado
       24AE BD27F7                    jsr             play_wrt
       24B1                           %CLRST          5
       0005           CONT_M          SET     5
       24B1                           IF CONT_M > 0
       24B1 31                        ins
       24B2                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       24B2                           IF CONT_M > 0
       24B2 31                        ins
       24B3                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       24B3                           IF CONT_M > 0
       24B3 31                        ins
       24B4                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       24B4                           IF CONT_M > 0
       24B4 31                        ins
       24B5                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       24B5                           IF CONT_M > 0
       24B5 31                        ins
       24B6                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       24B6                           ENDIF
       24B6                           ENDIF
       24B6                           ENDIF
       24B6                           ENDIF
       24B6                           ENDIF
       24B6                           ENDIF
       24B6                           %RESTORE
       24B6 1838                      puly
       24B8 38                        pulx
       24B9 33                        pulb
       24BA 32                        pula
       24BB 39                        rts
                      
       24BC           gp_shi_r        EQU             *               ;si no era ninguno de los anteriores, es un carguero
       24BC E60B                      ldab            11,x
       24BE C106                      cmpb            #MAX_C
       24C0 276D                      beq             gp_shi_l        ;si está en el borde derecho, se continúa
       24C2 E60C                      ldab            12,x
       24C4 37                        pshb                            ;se envía la letra (fila)
       24C5 E60B                      ldab            11,x
       24C7 5C                        incb                            ;se avanza a la derecha
       24C8 37                        pshb                            ;se envía el num (columna)
       24C9 1AEE0E                    ldy             14,x
       24CC 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       24CE E608                      ldab            8,x
       24D0 37                        pshb                            ;se envía el jugador atacado
       24D1 BD27A0                    jsr             play_src        ;se busca la posición de la derecha
       24D4                           %CLRST          5
       0005           CONT_M          SET     5
       24D4                           IF CONT_M > 0
       24D4 31                        ins
       24D5                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       24D5                           IF CONT_M > 0
       24D5 31                        ins
       24D6                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       24D6                           IF CONT_M > 0
       24D6 31                        ins
       24D7                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       24D7                           IF CONT_M > 0
       24D7 31                        ins
       24D8                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       24D8                           IF CONT_M > 0
       24D8 31                        ins
       24D9                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       24D9                           ENDIF
       24D9                           ENDIF
       24D9                           ENDIF
       24D9                           ENDIF
       24D9                           ENDIF
       24D9                           ENDIF
       24D9 2454                      bcc             gp_shi_l        ;si no había barco atacado en la derecha, se busca la izquierda
       24DB                           %WRITEPOS       #SUNK_SHI       
       24DB C690                      ldab            #SUNK_SHI
       24DD 37                        pshb                            ;se envía el byte a escribir
       24DE E60C                      ldab            12,x
       24E0 37                        pshb                            ;se envía letra (fila)
       24E1 E60B                      ldab            11,x
       24E3 37                        pshb                            ;se envía num (columna)
       24E4 1AEE09                    ldy             9,x
       24E7 183C                      pshy                            ;se envía dirección de inicio de matriz
       24E9 E608                      ldab            8,x
       24EB 37                        pshb                            ;se envía jugador atacado
       24EC BD276F                    jsr             writepos
       24EF                           %CLRST          6
       0006           CONT_M          SET     6
       24EF                           IF CONT_M > 0
       24EF 31                        ins
       24F0                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       24F0                           IF CONT_M > 0
       24F0 31                        ins
       24F1                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       24F1                           IF CONT_M > 0
       24F1 31                        ins
       24F2                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       24F2                           IF CONT_M > 0
       24F2 31                        ins
       24F3                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       24F3                           IF CONT_M > 0
       24F3 31                        ins
       24F4                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       24F4                           IF CONT_M > 0
       24F4 31                        ins
       24F5                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       24F5                           ENDIF
       24F5                           ENDIF
       24F5                           ENDIF
       24F5                           ENDIF
       24F5                           ENDIF
       24F5                           ENDIF
       24F5                           ENDIF
       24F5 6C0B                      inc             11,x
       24F7                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       24F7 C690                      ldab            #SUNK_SHI
       24F9 37                        pshb                            ;se envía el byte a escribir
       24FA E60C                      ldab            12,x
       24FC 37                        pshb                            ;se envía letra (fila)
       24FD E60B                      ldab            11,x
       24FF 37                        pshb                            ;se envía num (columna)
       2500 1AEE09                    ldy             9,x
       2503 183C                      pshy                            ;se envía dirección de inicio de matriz
       2505 E608                      ldab            8,x
       2507 37                        pshb                            ;se envía jugador atacado
       2508 BD276F                    jsr             writepos
       250B                           %CLRST          6
       0006           CONT_M          SET     6
       250B                           IF CONT_M > 0
       250B 31                        ins
       250C                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       250C                           IF CONT_M > 0
       250C 31                        ins
       250D                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       250D                           IF CONT_M > 0
       250D 31                        ins
       250E                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       250E                           IF CONT_M > 0
       250E 31                        ins
       250F                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       250F                           IF CONT_M > 0
       250F 31                        ins
       2510                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2510                           IF CONT_M > 0
       2510 31                        ins
       2511                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2511                           ENDIF
       2511                           ENDIF
       2511                           ENDIF
       2511                           ENDIF
       2511                           ENDIF
       2511                           ENDIF
       2511                           ENDIF
       2511 6A0B                      dec             11,x            ;se recupera el dato
       2513                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       2513 E60C                      ldab            12,x
       2515 37                        pshb                            ;se envía letra (fila)
       2516 E60B                      ldab            11,x
       2518 37                        pshb                            ;se envía num (columna)
       2519 1AEE0E                    ldy             14,x
       251C 183C                      pshy                            ;se envía inicio de array
       251E E608                      ldab            8,x
       2520 37                        pshb                            ;se envía jugador atacado
       2521 BD27F7                    jsr             play_wrt
       2524                           %CLRST          5
       0005           CONT_M          SET     5
       2524                           IF CONT_M > 0
       2524 31                        ins
       2525                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2525                           IF CONT_M > 0
       2525 31                        ins
       2526                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2526                           IF CONT_M > 0
       2526 31                        ins
       2527                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2527                           IF CONT_M > 0
       2527 31                        ins
       2528                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2528                           IF CONT_M > 0
       2528 31                        ins
       2529                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2529                           ENDIF
       2529                           ENDIF
       2529                           ENDIF
       2529                           ENDIF
       2529                           ENDIF
       2529                           ENDIF
       2529                           %RESTORE
       2529 1838                      puly
       252B 38                        pulx
       252C 33                        pulb
       252D 32                        pula
       252E 39                        rts
                      
       252F           gp_shi_l        EQU             *
       252F E60B                      ldab            11,x
       2531 C101                      cmpb            #MIN_C
       2533 276D                      beq             gp_shi_u        ;si está en el borde izquierdo, se continúa
       2535 E60C                      ldab            12,x
       2537 37                        pshb                            ;se envía la letra (fila)
       2538 E60B                      ldab            11,x
       253A 5A                        decb                            ;se retrocede a la izquierda
       253B 37                        pshb                            ;se envía el número (columna)
       253C 1AEE0E                    ldy             14,x
       253F 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       2541 E608                      ldab            8,x
       2543 37                        pshb                            ;se envía el jugador atacado
       2544 BD27A0                    jsr             play_src        ;se busca la posición de la izquierda
       2547                           %CLRST          5
       0005           CONT_M          SET     5
       2547                           IF CONT_M > 0
       2547 31                        ins
       2548                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2548                           IF CONT_M > 0
       2548 31                        ins
       2549                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2549                           IF CONT_M > 0
       2549 31                        ins
       254A                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       254A                           IF CONT_M > 0
       254A 31                        ins
       254B                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       254B                           IF CONT_M > 0
       254B 31                        ins
       254C                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       254C                           ENDIF
       254C                           ENDIF
       254C                           ENDIF
       254C                           ENDIF
       254C                           ENDIF
       254C                           ENDIF
       254C 2454                      bcc             gp_shi_u        ;si no había barco atacado a la izquierda, se busca arriba
       254E                           %WRITEPOS       #SUNK_SHI       
       254E C690                      ldab            #SUNK_SHI
       2550 37                        pshb                            ;se envía el byte a escribir
       2551 E60C                      ldab            12,x
       2553 37                        pshb                            ;se envía letra (fila)
       2554 E60B                      ldab            11,x
       2556 37                        pshb                            ;se envía num (columna)
       2557 1AEE09                    ldy             9,x
       255A 183C                      pshy                            ;se envía dirección de inicio de matriz
       255C E608                      ldab            8,x
       255E 37                        pshb                            ;se envía jugador atacado
       255F BD276F                    jsr             writepos
       2562                           %CLRST          6
       0006           CONT_M          SET     6
       2562                           IF CONT_M > 0
       2562 31                        ins
       2563                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2563                           IF CONT_M > 0
       2563 31                        ins
       2564                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2564                           IF CONT_M > 0
       2564 31                        ins
       2565                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2565                           IF CONT_M > 0
       2565 31                        ins
       2566                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2566                           IF CONT_M > 0
       2566 31                        ins
       2567                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2567                           IF CONT_M > 0
       2567 31                        ins
       2568                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2568                           ENDIF
       2568                           ENDIF
       2568                           ENDIF
       2568                           ENDIF
       2568                           ENDIF
       2568                           ENDIF
       2568                           ENDIF
       2568 6A0B                      dec             11,x
       256A                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       256A C690                      ldab            #SUNK_SHI
       256C 37                        pshb                            ;se envía el byte a escribir
       256D E60C                      ldab            12,x
       256F 37                        pshb                            ;se envía letra (fila)
       2570 E60B                      ldab            11,x
       2572 37                        pshb                            ;se envía num (columna)
       2573 1AEE09                    ldy             9,x
       2576 183C                      pshy                            ;se envía dirección de inicio de matriz
       2578 E608                      ldab            8,x
       257A 37                        pshb                            ;se envía jugador atacado
       257B BD276F                    jsr             writepos
       257E                           %CLRST          6
       0006           CONT_M          SET     6
       257E                           IF CONT_M > 0
       257E 31                        ins
       257F                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       257F                           IF CONT_M > 0
       257F 31                        ins
       2580                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2580                           IF CONT_M > 0
       2580 31                        ins
       2581                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2581                           IF CONT_M > 0
       2581 31                        ins
       2582                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2582                           IF CONT_M > 0
       2582 31                        ins
       2583                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2583                           IF CONT_M > 0
       2583 31                        ins
       2584                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2584                           ENDIF
       2584                           ENDIF
       2584                           ENDIF
       2584                           ENDIF
       2584                           ENDIF
       2584                           ENDIF
       2584                           ENDIF
       2584 6C0B                      inc             11,x            ;se recupera el dato
       2586                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       2586 E60C                      ldab            12,x
       2588 37                        pshb                            ;se envía letra (fila)
       2589 E60B                      ldab            11,x
       258B 37                        pshb                            ;se envía num (columna)
       258C 1AEE0E                    ldy             14,x
       258F 183C                      pshy                            ;se envía inicio de array
       2591 E608                      ldab            8,x
       2593 37                        pshb                            ;se envía jugador atacado
       2594 BD27F7                    jsr             play_wrt
       2597                           %CLRST          5
       0005           CONT_M          SET     5
       2597                           IF CONT_M > 0
       2597 31                        ins
       2598                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2598                           IF CONT_M > 0
       2598 31                        ins
       2599                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2599                           IF CONT_M > 0
       2599 31                        ins
       259A                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       259A                           IF CONT_M > 0
       259A 31                        ins
       259B                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       259B                           IF CONT_M > 0
       259B 31                        ins
       259C                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       259C                           ENDIF
       259C                           ENDIF
       259C                           ENDIF
       259C                           ENDIF
       259C                           ENDIF
       259C                           ENDIF
       259C                           %RESTORE
       259C 1838                      puly
       259E 38                        pulx
       259F 33                        pulb
       25A0 32                        pula
       25A1 39                        rts
                      
       25A2           gp_shi_u        EQU             *
       25A2 E60C                      ldab            12,x
       25A4 C10A                      cmpb            #MIN_F
       25A6 276D                      beq             gp_shi_d        ;si está en el borde superior, se continúa
       25A8 E60C                      ldab            12,x
       25AA 5A                        decb                            ;se va a la posición de arriba
       25AB 37                        pshb                            ;se envía la letra (fila)
       25AC E60B                      ldab            11,x
       25AE 37                        pshb                            ;se envía el num (columna)
       25AF 1AEE0E                    ldy             14,x
       25B2 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       25B4 E608                      ldab            8,x
       25B6 37                        pshb                            ;se envía el jugador atacado
       25B7 BD27A0                    jsr             play_src        ;se busca la posición de arriba
       25BA                           %CLRST          5
       0005           CONT_M          SET     5
       25BA                           IF CONT_M > 0
       25BA 31                        ins
       25BB                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       25BB                           IF CONT_M > 0
       25BB 31                        ins
       25BC                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       25BC                           IF CONT_M > 0
       25BC 31                        ins
       25BD                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       25BD                           IF CONT_M > 0
       25BD 31                        ins
       25BE                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       25BE                           IF CONT_M > 0
       25BE 31                        ins
       25BF                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       25BF                           ENDIF
       25BF                           ENDIF
       25BF                           ENDIF
       25BF                           ENDIF
       25BF                           ENDIF
       25BF                           ENDIF
       25BF 2454                      bcc             gp_shi_d        ;si no había barco atacado arriba, se busca abajo
       25C1                           %WRITEPOS       #SUNK_SHI       
       25C1 C690                      ldab            #SUNK_SHI
       25C3 37                        pshb                            ;se envía el byte a escribir
       25C4 E60C                      ldab            12,x
       25C6 37                        pshb                            ;se envía letra (fila)
       25C7 E60B                      ldab            11,x
       25C9 37                        pshb                            ;se envía num (columna)
       25CA 1AEE09                    ldy             9,x
       25CD 183C                      pshy                            ;se envía dirección de inicio de matriz
       25CF E608                      ldab            8,x
       25D1 37                        pshb                            ;se envía jugador atacado
       25D2 BD276F                    jsr             writepos
       25D5                           %CLRST          6
       0006           CONT_M          SET     6
       25D5                           IF CONT_M > 0
       25D5 31                        ins
       25D6                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       25D6                           IF CONT_M > 0
       25D6 31                        ins
       25D7                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       25D7                           IF CONT_M > 0
       25D7 31                        ins
       25D8                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       25D8                           IF CONT_M > 0
       25D8 31                        ins
       25D9                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       25D9                           IF CONT_M > 0
       25D9 31                        ins
       25DA                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       25DA                           IF CONT_M > 0
       25DA 31                        ins
       25DB                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       25DB                           ENDIF
       25DB                           ENDIF
       25DB                           ENDIF
       25DB                           ENDIF
       25DB                           ENDIF
       25DB                           ENDIF
       25DB                           ENDIF
       25DB 6A0C                      dec             12,x
       25DD                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       25DD C690                      ldab            #SUNK_SHI
       25DF 37                        pshb                            ;se envía el byte a escribir
       25E0 E60C                      ldab            12,x
       25E2 37                        pshb                            ;se envía letra (fila)
       25E3 E60B                      ldab            11,x
       25E5 37                        pshb                            ;se envía num (columna)
       25E6 1AEE09                    ldy             9,x
       25E9 183C                      pshy                            ;se envía dirección de inicio de matriz
       25EB E608                      ldab            8,x
       25ED 37                        pshb                            ;se envía jugador atacado
       25EE BD276F                    jsr             writepos
       25F1                           %CLRST          6
       0006           CONT_M          SET     6
       25F1                           IF CONT_M > 0
       25F1 31                        ins
       25F2                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       25F2                           IF CONT_M > 0
       25F2 31                        ins
       25F3                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       25F3                           IF CONT_M > 0
       25F3 31                        ins
       25F4                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       25F4                           IF CONT_M > 0
       25F4 31                        ins
       25F5                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       25F5                           IF CONT_M > 0
       25F5 31                        ins
       25F6                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       25F6                           IF CONT_M > 0
       25F6 31                        ins
       25F7                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       25F7                           ENDIF
       25F7                           ENDIF
       25F7                           ENDIF
       25F7                           ENDIF
       25F7                           ENDIF
       25F7                           ENDIF
       25F7                           ENDIF
       25F7 6C0C                      inc             12,x            ;se recupera el dato
       25F9                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       25F9 E60C                      ldab            12,x
       25FB 37                        pshb                            ;se envía letra (fila)
       25FC E60B                      ldab            11,x
       25FE 37                        pshb                            ;se envía num (columna)
       25FF 1AEE0E                    ldy             14,x
       2602 183C                      pshy                            ;se envía inicio de array
       2604 E608                      ldab            8,x
       2606 37                        pshb                            ;se envía jugador atacado
       2607 BD27F7                    jsr             play_wrt
       260A                           %CLRST          5
       0005           CONT_M          SET     5
       260A                           IF CONT_M > 0
       260A 31                        ins
       260B                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       260B                           IF CONT_M > 0
       260B 31                        ins
       260C                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       260C                           IF CONT_M > 0
       260C 31                        ins
       260D                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       260D                           IF CONT_M > 0
       260D 31                        ins
       260E                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       260E                           IF CONT_M > 0
       260E 31                        ins
       260F                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       260F                           ENDIF
       260F                           ENDIF
       260F                           ENDIF
       260F                           ENDIF
       260F                           ENDIF
       260F                           ENDIF
       260F                           %RESTORE
       260F 1838                      puly
       2611 38                        pulx
       2612 33                        pulb
       2613 32                        pula
       2614 39                        rts
                      
       2615           gp_shi_d        EQU             *
       2615 E60C                      ldab            12,x
       2617 C10F                      cmpb            #MAX_F
       2619 276D                      beq             gp_shi_h        ;si está en el borde inferior, se continúa
       261B E60C                      ldab            12,x
       261D 5C                        incb                            ;se va a la posición de abajo
       261E 37                        pshb                            ;se envía la letra (fila)
       261F E60B                      ldab            11,x
       2621 37                        pshb                            ;se envía el num (columna)
       2622 1AEE0E                    ldy             14,x
       2625 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       2627 E608                      ldab            8,x
       2629 37                        pshb                            ;se envía el jugador atacado
       262A BD27A0                    jsr             play_src        ;se busca la posición de abajo
       262D                           %CLRST          5
       0005           CONT_M          SET     5
       262D                           IF CONT_M > 0
       262D 31                        ins
       262E                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       262E                           IF CONT_M > 0
       262E 31                        ins
       262F                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       262F                           IF CONT_M > 0
       262F 31                        ins
       2630                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2630                           IF CONT_M > 0
       2630 31                        ins
       2631                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2631                           IF CONT_M > 0
       2631 31                        ins
       2632                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2632                           ENDIF
       2632                           ENDIF
       2632                           ENDIF
       2632                           ENDIF
       2632                           ENDIF
       2632                           ENDIF
       2632 2454                      bcc             gp_shi_h        ;si no había barco atacado abajo, el carguero está tocado
       2634                           %WRITEPOS       #SUNK_SHI       
       2634 C690                      ldab            #SUNK_SHI
       2636 37                        pshb                            ;se envía el byte a escribir
       2637 E60C                      ldab            12,x
       2639 37                        pshb                            ;se envía letra (fila)
       263A E60B                      ldab            11,x
       263C 37                        pshb                            ;se envía num (columna)
       263D 1AEE09                    ldy             9,x
       2640 183C                      pshy                            ;se envía dirección de inicio de matriz
       2642 E608                      ldab            8,x
       2644 37                        pshb                            ;se envía jugador atacado
       2645 BD276F                    jsr             writepos
       2648                           %CLRST          6
       0006           CONT_M          SET     6
       2648                           IF CONT_M > 0
       2648 31                        ins
       2649                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2649                           IF CONT_M > 0
       2649 31                        ins
       264A                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       264A                           IF CONT_M > 0
       264A 31                        ins
       264B                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       264B                           IF CONT_M > 0
       264B 31                        ins
       264C                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       264C                           IF CONT_M > 0
       264C 31                        ins
       264D                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       264D                           IF CONT_M > 0
       264D 31                        ins
       264E                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       264E                           ENDIF
       264E                           ENDIF
       264E                           ENDIF
       264E                           ENDIF
       264E                           ENDIF
       264E                           ENDIF
       264E                           ENDIF
       264E 6C0C                      inc             12,x
       2650                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       2650 C690                      ldab            #SUNK_SHI
       2652 37                        pshb                            ;se envía el byte a escribir
       2653 E60C                      ldab            12,x
       2655 37                        pshb                            ;se envía letra (fila)
       2656 E60B                      ldab            11,x
       2658 37                        pshb                            ;se envía num (columna)
       2659 1AEE09                    ldy             9,x
       265C 183C                      pshy                            ;se envía dirección de inicio de matriz
       265E E608                      ldab            8,x
       2660 37                        pshb                            ;se envía jugador atacado
       2661 BD276F                    jsr             writepos
       2664                           %CLRST          6
       0006           CONT_M          SET     6
       2664                           IF CONT_M > 0
       2664 31                        ins
       2665                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       2665                           IF CONT_M > 0
       2665 31                        ins
       2666                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       2666                           IF CONT_M > 0
       2666 31                        ins
       2667                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       2667                           IF CONT_M > 0
       2667 31                        ins
       2668                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2668                           IF CONT_M > 0
       2668 31                        ins
       2669                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2669                           IF CONT_M > 0
       2669 31                        ins
       266A                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       266A                           ENDIF
       266A                           ENDIF
       266A                           ENDIF
       266A                           ENDIF
       266A                           ENDIF
       266A                           ENDIF
       266A                           ENDIF
       266A 6A0C                      dec             12,x            ;se recupera el dato
       266C                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       266C E60C                      ldab            12,x
       266E 37                        pshb                            ;se envía letra (fila)
       266F E60B                      ldab            11,x
       2671 37                        pshb                            ;se envía num (columna)
       2672 1AEE0E                    ldy             14,x
       2675 183C                      pshy                            ;se envía inicio de array
       2677 E608                      ldab            8,x
       2679 37                        pshb                            ;se envía jugador atacado
       267A BD27F7                    jsr             play_wrt
       267D                           %CLRST          5
       0005           CONT_M          SET     5
       267D                           IF CONT_M > 0
       267D 31                        ins
       267E                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       267E                           IF CONT_M > 0
       267E 31                        ins
       267F                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       267F                           IF CONT_M > 0
       267F 31                        ins
       2680                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       2680                           IF CONT_M > 0
       2680 31                        ins
       2681                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       2681                           IF CONT_M > 0
       2681 31                        ins
       2682                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       2682                           ENDIF
       2682                           ENDIF
       2682                           ENDIF
       2682                           ENDIF
       2682                           ENDIF
       2682                           ENDIF
       2682                           %RESTORE
       2682 1838                      puly
       2684 38                        pulx
       2685 33                        pulb
       2686 32                        pula
       2687 39                        rts
                      
       2688           gp_shi_h        EQU             *
       2688                           %WRITEPOS       #HIT            ;se marca al carguero como tocado
       2688 C6B0                      ldab            #HIT
       268A 37                        pshb                            ;se envía el byte a escribir
       268B E60C                      ldab            12,x
       268D 37                        pshb                            ;se envía letra (fila)
       268E E60B                      ldab            11,x
       2690 37                        pshb                            ;se envía num (columna)
       2691 1AEE09                    ldy             9,x
       2694 183C                      pshy                            ;se envía dirección de inicio de matriz
       2696 E608                      ldab            8,x
       2698 37                        pshb                            ;se envía jugador atacado
       2699 BD276F                    jsr             writepos
       269C                           %CLRST          6
       0006           CONT_M          SET     6
       269C                           IF CONT_M > 0
       269C 31                        ins
       269D                           %CLRST  CONT_M-1
       0005           CONT_M          SET     CONT_M-1
       269D                           IF CONT_M > 0
       269D 31                        ins
       269E                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       269E                           IF CONT_M > 0
       269E 31                        ins
       269F                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       269F                           IF CONT_M > 0
       269F 31                        ins
       26A0                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       26A0                           IF CONT_M > 0
       26A0 31                        ins
       26A1                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       26A1                           IF CONT_M > 0
       26A1 31                        ins
       26A2                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       26A2                           ENDIF
       26A2                           ENDIF
       26A2                           ENDIF
       26A2                           ENDIF
       26A2                           ENDIF
       26A2                           ENDIF
       26A2                           ENDIF
       26A2                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       26A2 E60C                      ldab            12,x
       26A4 37                        pshb                            ;se envía letra (fila)
       26A5 E60B                      ldab            11,x
       26A7 37                        pshb                            ;se envía num (columna)
       26A8 1AEE0E                    ldy             14,x
       26AB 183C                      pshy                            ;se envía inicio de array
       26AD E608                      ldab            8,x
       26AF 37                        pshb                            ;se envía jugador atacado
       26B0 BD27F7                    jsr             play_wrt
       26B3                           %CLRST          5
       0005           CONT_M          SET     5
       26B3                           IF CONT_M > 0
       26B3 31                        ins
       26B4                           %CLRST  CONT_M-1
       0004           CONT_M          SET     CONT_M-1
       26B4                           IF CONT_M > 0
       26B4 31                        ins
       26B5                           %CLRST  CONT_M-1
       0003           CONT_M          SET     CONT_M-1
       26B5                           IF CONT_M > 0
       26B5 31                        ins
       26B6                           %CLRST  CONT_M-1
       0002           CONT_M          SET     CONT_M-1
       26B6                           IF CONT_M > 0
       26B6 31                        ins
       26B7                           %CLRST  CONT_M-1
       0001           CONT_M          SET     CONT_M-1
       26B7                           IF CONT_M > 0
       26B7 31                        ins
       26B8                           %CLRST  CONT_M-1
       0000           CONT_M          SET     CONT_M-1
                                      IF CONT_M > 0
                                      ins
                                      %CLRST  CONT_M-1
       26B8                           ENDIF
       26B8                           ENDIF
       26B8                           ENDIF
       26B8                           ENDIF
       26B8                           ENDIF
       26B8                           ENDIF
       26B8                           %RESTORE
       26B8 1838                      puly
       26BA 38                        pulx
       26BB 33                        pulb
       26BC 32                        pula
       26BD 39                        rts
                      
                      *==========================================================================
                      * subrutina def_ships
                      * Función: Determina los valores de cargueros y submarinos a utilizar.
                      * Recibe: Nada
                      * Devuelve: Cantidad de cargueros y submarinos en memorias correspondientes.
                      * Requiere:
                      *==========================================================================
       26BE           def_ships       EQU             *
       26BE                           %BACKUP
       26BE 36                        psha
       26BF 37                        pshb
       26C0 3C                        pshx
       26C1 183C                      pshy
                      
       26C3           wait_nsu        EQU             *
       26C3 CE42CE                    ldx             #subm
       26C6 BD28EE                    jsr             show
                      
       26C9 BD2902                    jsr             read_kb
       26CC 4D                        tsta
       26CD 2BF4                      bmi             wait_nsu
       26CF 36                        psha
       26D0                           %EXPECT         B_fire
                                      
       26D0 BD2902    expe0027          jsr             read_kb
       26D3 4D                        tsta
       26D4 2BFA                      bmi             expe0027
       26D6 810B                      cmpa            #B_fire
       26D8 26F6                      bne             expe0027
                      
       26DA 32                        pula
                      
       26DB CE4265                    ldx             #kb_value
       26DE 16                        tab
       26DF 3A                        abx
       26E0 A600                      ldaa            0,x             ;se obtiene el valor ingresado ya codificado
                      
       26E2 8105                      cmpa            #SUB            ;se controla que no sea menor al máximo de submarinos
       26E4 22DD                      bhi             wait_nsu
       26E6 B7425B                    staa            cant_sub        ;se guarda la cantidad de submarinos
                      
       26E9           wait_nsh        EQU             *
       26E9 CE42D3                    ldx             #carg
       26EC BD28EE                    jsr             show
                      
       26EF BD2902                    jsr             read_kb
       26F2 4D                        tsta
       26F3 2BF4                      bmi             wait_nsh
       26F5 36                        psha
       26F6                           %EXPECT         B_fire
                                      
       26F6 BD2902    expe0028          jsr             read_kb
       26F9 4D                        tsta
       26FA 2BFA                      bmi             expe0028
       26FC 810B                      cmpa            #B_fire
       26FE 26F6                      bne             expe0028
                      
       2700 32                        pula
                      
       2701 CE4265                    ldx             #kb_value
       2704 16                        tab
       2705 3A                        abx
       2706 A600                      ldaa            0,x             ;se obtiene el valor ingresado ya codificado
                      
       2708 8104                      cmpa            #SHI            ;se controla que no sea menor al máximo de submarinos
       270A 22DD                      bhi             wait_nsh
       270C B7425C                    staa            cant_shi        ;se guarda la cantidad de submarinos
                      
       270F                           %RESTORE
       270F 1838                      puly
       2711 38                        pulx
       2712 33                        pulb
       2713 32                        pula
       2714 39                        rts
                      
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
                      
       2715           mirror          EQU             *       
                              
       2715                           %BACKUP
       2715 36                        psha
       2716 37                        pshb
       2717 3C                        pshx
       2718 183C                      pshy
       271A 30                        tsx                             ;creo frame pointer
       271B A60B                      ldaa            11,x            ;cargo número de jugador
       271D E608                      ldab            8,x             ;cargo longitud de la copia (del array a copiar)
       271F 3D                        mul                             ;obtengo offset de jugador
       2720 E30C                      addd            12,x            ;sumo offset de jugador
       2722 ED0C                      std             12,x            ;guardo la dirección con offset del array fuente
                      
       2724 A608                      ldaa            8,x             ;cargo longitud de copia en contador
       2726 2710                      beq             mi_end          ;si es cero, vuelvo desde subrutina
       2728 1AEE09                    ldy             9,x             ;cargo índice a destino de la copia.
       272B EE0C                      ldx             12,x            ;cargo índice a array origen de copia.
                      
       272D           mi_loop         EQU             *
                      
       272D E600                      ldab            0,x             ;cargo byte a copiar
       272F 18E700                    stab            0,y             ;copio byte en destino
       2732 08                        inx                             ;siguiente byte
       2733 1808                      iny                             ;siguiente byte
       2735 4A                        deca                            ;decremento contador
       2736 26F5                      bne             mi_loop
                      
       2738           mi_end          EQU             *
                      
       2738                           %RESTORE
       2738 1838                      puly
       273A 38                        pulx
       273B 33                        pulb
       273C 32                        pula
                              
       273D 39                        rts     
                                      
                                                      
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
                      
       273E           fetchpos        EQU             *
                      
       273E                           %BACKUP
       273E 36                        psha
       273F 37                        pshb
       2740 3C                        pshx
       2741 183C                      pshy
       2743 30                        tsx                             ;creo frame pointer
       2744 A608                      ldaa            8,x             ;cargo numero de jugador
       2746 C624                      ldab            #ARRAY_L        ;cargo longitud del arreglo (MAX 255 bytes)
       2748 3D                        mul                             ;obtengo offset de jugador
       2749 E309                      addd            9,x             ;sumo offset de jugador a top of array
       274B ED09                      std             9,x             ;obtengo direccion al arreglo del jugador
                      
       274D A60C                      ldaa            12,x            ;cargo coordenada letra (FILA)
       274F C60A                      ldab            #NORM           ;cargo valor de normalización de filas
       2751 10                        sba                             ;obtengo numero de fila
       2752 C606                      ldab            #COL            ;cargo longitud de fila
       2754 3D                        mul                             ;obtengo offset de fila
       2755 E309                      addd            9,x             ;sumo offset de fila a la dirección del arreglo del jugador
       2757 ED09                      std             9,x             ;obtengo dirección a la fila que se desea
                      
       2759 E60B                      ldab            11,x            ;cargo coordenada (offset) de columna
       275B 5A                        decb                            ;normalizo offset de columna
       275C 4F                        clra                            ;borro AccA para tener el valor de AccB en AccD
       275D E309                      addd            9,x             ;sumo offset de columna a la dirección temporal
       275F ED09                      std             9,x             ;guardo en stack la dirección del byte a consultar
                      
       2761 1AEE09                    ldy             9,x             ;cargo en IY la dirección del byte que se desea obtener
       2764 18A600                    ldaa            0,y             ;cargo el byte a devolver en AccA
       2767 A70C                      staa            12,x            ;guardo el byte a devolver en stack
       2769                           %RESTORE
       2769 1838                      puly
       276B 38                        pulx
       276C 33                        pulb
       276D 32                        pula
                                      
       276E 39                        rts
                                      
                                      
                      
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
                      
       276F           writepos        EQU             *
                      
       276F                           %BACKUP
       276F 36                        psha
       2770 37                        pshb
       2771 3C                        pshx
       2772 183C                      pshy
       2774 30                        tsx                             ;creo frame pointer
       2775 A608                      ldaa            8,x             ;cargo numero de jugador
       2777 C624                      ldab            #ARRAY_L        ;cargo longitud del arreglo (MAX 255 bytes)
       2779 3D                        mul                             ;obtengo offset de jugador
       277A E309                      addd            9,x             ;sumo offset de jugador a top of array
       277C ED09                      std             9,x             ;obtengo direccion al arreglo del jugador
                      
       277E A60C                      ldaa            12,x            ;cargo coordenada letra (FILA)
       2780 C60A                      ldab            #NORM           ;cargo valor de normalización de filas
       2782 10                        sba                             ;obtengo numero de fila
       2783 C606                      ldab            #COL            ;cargo longitud de fila
       2785 3D                        mul                             ;obtengo offset de fila
       2786 E309                      addd            9,x             ;sumo offset de fila a la dirección del arreglo del jugador
       2788 ED09                      std             9,x             ;obtengo dirección a la fila que se desea
                      
       278A E60B                      ldab            11,x            ;cargo coordenada (offset) de columna
       278C 5A                        decb                            ;normalizo offset de columna
       278D 4F                        clra                            ;borro AccA para tener el valor de AccB en AccD
       278E E309                      addd            9,x             ;sumo offset de columna a la dirección temporal
       2790 ED09                      std             9,x             ;guardo en stack la dirección del byte a consultar
                      
       2792 1AEE09                    ldy             9,x             ;cargo en IY la dirección donde se desea escribir
       2795 A60D                      ldaa            13,x            ;cargo en AccA el byte que se quiere escribir en el arreglo
       2797 18A700                    staa            0,y             ;escribo el byte sobre el arreglo
       279A                           %RESTORE
       279A 1838                      puly
       279C 38                        pulx
       279D 33                        pulb
       279E 32                        pula
                                      
       279F 39                        rts
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
                      
       27A0           play_src        EQU             *               
                                              
       27A0                           %BACKUP
       27A0 36                        psha
       27A1 37                        pshb
       27A2 3C                        pshx
       27A3 183C                      pshy
       27A5 30                        tsx                             ;creo frame pointer
       27A6 A608                      ldaa            8,x             ;cargo número de array (jugador)
       27A8 C61B                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       27AA 3D                        mul                             ;obtengo offset de jugador
       27AB E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       27AD 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
                      
       27AF           ps_loop         EQU             *
                      
       27AF 18A600                    ldaa            0,y             ;cargo byte1 a comparar
       27B2 2718                      beq             ps_ntfnd        ;si encontré el terminador, el valor requerido no está en el array
                      
       27B4 A10C                      cmpa            12,x            ;comparo con el byte1 a buscar
       27B6 2706                      beq             ps_next         ;si coinciden, busco el byte 2
       27B8 1808                      iny                             ;me posiciono sobre siguiente byte1 a comparar
       27BA 1808                      iny
       27BC 20F1                      bra             ps_loop         ;vuelvo a comparar
                      
       27BE           ps_next         EQU             *
                      
       27BE 18E601                    ldab            1,y             ;cargo byte a continuación del anterior
       27C1 1808                      iny                             ;me posiciono sobre siguiente byte1 a comparar
       27C3 1808                      iny
       27C5 E10B                      cmpb            11,x            ;comparo con byte2
       27C7 26E6                      bne             ps_loop         ;si no son iguales, sigo buscando
       27C9 0D                        sec                             ;set del carry para informar que se encontró el valor
       27CA 2001                      bra             ps_end
                                              
       27CC           ps_ntfnd        EQU             *
                      
       27CC 0C                        clc                             ;carry en 0 para informar que no se encontró el valor
                      
       27CD           ps_end          EQU             *
                                      
       27CD                           %RESTORE
       27CD 1838                      puly
       27CF 38                        pulx
       27D0 33                        pulb
       27D1 32                        pula
                      
       27D2 39                        rts
                      
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
                      
       27D3           play_cnt        EQU             *
                      
       27D3                           %BACKUP
       27D3 36                        psha
       27D4 37                        pshb
       27D5 3C                        pshx
       27D6 183C                      pshy
       27D8 30                        tsx                             ;creo frame pointer
       27D9 A608                      ldaa            8,x             ;cargo número de array (jugador)
       27DB C61B                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       27DD 3D                        mul                             ;obtengo offset de jugador
       27DE E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       27E0 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
       27E2 4F                        clra                            ;inicializo contador
                                              
       27E3           pc_loop         EQU             *               
                                              
       27E3 18E600                    ldab            0,y             ;cargo byte a comparar
       27E6 2707                      beq             pc_end          ;si es terminador, terminé de buscar
       27E8 1808                      iny                             ;si no es terminador, me posiciono sobre siguiente dato de 
       27EA 1808                      iny                             ;2 bytes
       27EC 4C                        inca                            ;incremento contador
       27ED 20F4                      bra             pc_loop         ;vuelvo a comparar
                                              
       27EF           pc_end          EQU             *       
                                              
       27EF A70A                      staa            10,x            ;devuelvo valor por stack
       27F1                           %RESTORE
       27F1 1838                      puly
       27F3 38                        pulx
       27F4 33                        pulb
       27F5 32                        pula
                                      
       27F6 39                        rts
                      
                      
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
                      
       27F7           play_wrt        EQU             *
                      
       27F7                           %BACKUP
       27F7 36                        psha
       27F8 37                        pshb
       27F9 3C                        pshx
       27FA 183C                      pshy
       27FC 30                        tsx                             ;creo frame pointer
       27FD A608                      ldaa            8,x             ;cargo número de array (jugador)
       27FF C61B                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       2801 3D                        mul                             ;obtengo offset de jugador
       2802 E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       2804 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
                      
                      
       2806           pw_loop         EQU             *
                      
       2806 18A600                    ldaa            0,y             ;cargo byte del array
       2809 2704                      beq             pw_end          ;si es cero, entonces escribo los datos
       280B 1808                      iny                             ;si no es cero, me posiciono sobre la siguiente posición
       280D 20F7                      bra             pw_loop         ;vuelvo a buscar ceros
                      
       280F           pw_end          EQU             *
                      
       280F A60C                      ldaa            12,x            ;cargo primer byte a escribir
       2811 18A700                    staa            0,y             ;guardo primer byte en memoria
       2814 1808                      iny                             ;me posiciono en el siguiente byte
       2816 A60B                      ldaa            11,x            ;cargo segundo byte a escribir
       2818 18A700                    staa            0,y             ;guardo segundo byte en memoria
       281B                           %RESTORE
       281B 1838                      puly
       281D 38                        pulx
       281E 33                        pulb
       281F 32                        pula
                      
       2820 39                        rts
                      
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
       2821           clrzone         EQU     *
                      
       2821                           %BACKUP                         ;backup de registros en stack
       2821 36                        psha
       2822 37                        pshb
       2823 3C                        pshx
       2824 183C                      pshy
                      
       2826 30                        tsx
       2827 A608                      ldaa    8,x                     ;cargo contador con longitud de zona a borrar
       2829 EE09                      ldx     9,x                     ;cargo puntero zona de memoria a borrar
       282B D600                      ldab    $00                     ;cargo el cero
       282D           clr_loop        EQU     *
       282D E700                      stab    0,x                     ;guardo caracter en posicion del array
       282F 08                        inx                             ;siguiente posición     
       2830 4A                        deca                            ;decremento contador
       2831 26FA                      bne     clr_loop                ;si no es cero, escribo en la posición siguiente.
                      
       2833                           %RESTORE                        :restauro backup en registros
       2833 1838                      puly
       2835 38                        pulx
       2836 33                        pulb
       2837 32                        pula
                      
       2838 39                        rts
                      
                      *---------------------------------------
                      * WhoPlays: subrutina para ver que 
                      *           jugador juega
                      *---------------------------------------
                      
       2839           whoplays        EQU             *
                      
       2839 BD2902    wait_ply        jsr             read_kb
       283C 4D                        tsta
       283D 2BFA                      bmi             wait_ply
       283F 8103                      cmpa            #B_P1
       2841 2708                      beq             play_p1
       2843 8107                      cmpa            #B_P2
       2845 26F2                      bne             wait_ply
       2847 8600                      ldaa            #P1
       2849 2002                      bra             ply_end
                      
       284B 8601      play_p1         ldaa            #P2
       284D 39        ply_end         rts             
                      
                      *---------------------------------------
                      * Input: subrutina que devuelve
                      *        entrada para la batalla naval
                      * AccA: Fila de la posicion jugada
                      * AccB: Columna de la posicion jugada
                      *---------------------------------------
                      
       284E           input           EQU             *
                      
                      * back up de registros
       284E 3C                        pshx
       284F 183C                      pshy
                      
                      * creo variables locales
       2851 36                        psha
       2852 36                        psha
       2853 36                        psha
                      
                      * inicializo framepointer
       2854 1830                      tsy
                      
                      * prendo el led
       2856                           %LED_INV
       2856 B61000                    ldaa            PORTA
       2859 8840                      eora            #SHF_LED
       285B B71000                    staa            PORTA
                      
                      * inicializo el contador
       285E 8603      inp_rest        ldaa            #INPUTS
       2860 18A700                    staa            CONT,Y
                                      
                      * limpio el display
       2863 CE4260                    ldx             #string
       2866 8604                      ldaa            #4
       2868 C620                      ldab            #ESP
       286A BD29CC                    jsr             clrarray
                      
       286D BD29BB                    jsr             clr_dis
                      
       2870 86FF                      ldaa            #$FF
       2872 B7425F                    staa            buffer
                      
                      * lectura de teclado
       2875 BD2902    wait_kb         jsr             read_kb
       2878 4D                        tsta
       2879 2BFA                      bmi             wait_kb
       287B B1425F                    cmpa            buffer
       287E 27F5                      beq             wait_kb
       2880 B7425F                    staa            buffer
                                      
       2883 810F                      cmpa            #B_reset
       2885 27D7                      beq             inp_rest
       2887 8103                      cmpa            #B_P1
       2889 27D3                      beq             inp_rest
       288B 8107                      cmpa            #B_P2
       288D 27CF                      beq             inp_rest
                      
       288F 186A00                    dec             CONT,Y
       2892 272B                      beq             chk_fire
       2894 810B                      cmpa            #B_fire
       2896 27C6                      beq             inp_rest
       2898 36                        psha
       2899 CE4265                    ldx             #kb_value
       289C 16                        tab
       289D 3A                        abx
       289E A600                      ldaa            0,X             
       28A0 183C                      pshy
       28A2 18E600                    ldab            CONT,Y
       28A5 183A                      aby
       28A7 18A700                    staa            0,Y
       28AA 1838                      puly
                      
       28AC 33                        pulb
       28AD BD29AB                    jsr             to_ascii
       28B0 17                        tba
                                      
       28B1 CE4260                    ldx             #string
       28B4 BD299B                    jsr             sh_value
                      
       28B7 CE4260                    ldx             #string
       28BA BD28EE                    jsr             show
                      
       28BD 20B6                      bra             wait_kb
                      
       28BF 810B      chk_fire        cmpa            #B_fire
       28C1 269B                      bne             inp_rest
       28C3 18A602                    ldaa            FILA_L,Y
       28C6 810A                      cmpa            #MIN_FIL
       28C8 2594                      blo             inp_rest
       28CA 810F                      cmpa            #MAX_FIL
       28CC 2290                      bhi             inp_rest
       28CE 18A601                    ldaa            COL_L,Y
       28D1 8101                      cmpa            #MIN_COL
       28D3 2589                      blo             inp_rest
       28D5 8106                      cmpa            #MAX_COL
       28D7 2285                      bhi             inp_rest
                      
                      * apago el led
       28D9                           %LED_INV
       28D9 B61000                    ldaa            PORTA
       28DC 8840                      eora            #SHF_LED
       28DE B71000                    staa            PORTA
                      
                      * libero variables locales
       28E1 32                        pula
       28E2 32                        pula
       28E3 32                        pula
                      
       28E4 18A602    return          ldaa            FILA_L,Y
       28E7 18E601                    ldab            COL_L,Y         
                      
                      * restore de registros
       28EA 1838                      puly
       28EC 38                        pulx
                      
       28ED 39                        rts
                      
                      *------------------------------------
                      * Show: Por IX direccion de string de
                      *       4 caracteres para imprimir
                      *------------------------------------
                      
       28EE           show            EQU             *
                      
                      * back up de registros
       28EE 3C                        pshx
       28EF 37                        pshb
       28F0 36                        psha
                      
                      * inicializo contador de posiciones
       28F1 C604                      ldab            #LENGTH
                                      
                      * loop para enviar a cada posicion
       28F3 5A        sh_loop         decb
       28F4 2B08                      bmi             sh_end          ; si no pase por todas las posiciones
       28F6 A600                      ldaa            0,X             ; tomo del string el caracter
       28F8 BD29B4                    jsr             outchar         ; y lo imprimo en la posicion actual
       28FB 08                        inx
       28FC 20F5                      bra             sh_loop
                      
                      * restore de registros                          
       28FE 32        sh_end          pula
       28FF 33                        pulb
       2900 38                        pulx
       2901 39                        rts
                      
                      *----------------------------------------------
                      * read_kb: AccA: tecla presionada
                      *          o -1
                      *----------------------------------------------
                      
       2902           read_kb         EQU             *
                      
                      * back up de registros
       2902 3C                        pshx
       2903 37                        pshb
                      
                      * creo espacio para variables locales
       2904                           %MALLOC         1               ; variable mascara de filas
       2904                           IF 1 > 0
       2904 30                        tsx
       2905 8F                        xgdx
       2906 830001                    subd    #1
       2909 8F                        xgdx
       290A 35                        txs
       290B                           ENDIF
       290B                           %MALLOC         1               ; variable mascara de columnas
       290B                           IF 1 > 0
       290B 30                        tsx
       290C 8F                        xgdx
       290D 830001                    subd    #1
       2910 8F                        xgdx
       2911 35                        txs
       2912                           ENDIF
       2912                           %MALLOC         1               ; variable contador de columnas
       2912                           IF 1 > 0
       2912 30                        tsx
       2913 8F                        xgdx
       2914 830001                    subd    #1
       2917 8F                        xgdx
       2918 35                        txs
       2919                           ENDIF
       2919                           %MALLOC         1               ; variable contador de tecla
       2919                           IF 1 > 0
       2919 30                        tsx
       291A 8F                        xgdx
       291B 830001                    subd    #1
       291E 8F                        xgdx
       291F 35                        txs
       2920                           ENDIF
                      
                      * inicializacion de variables
       2920 30                        tsx                             ; framepointer para acceder a locales
       2921 C604                      ldab            #LENGTH         ; contador de fila
       2923 860E                      ldaa            #FIL0           ; mascara para la fila cero
       2925 A703                      staa            MASK_FIL,X      
       2927 8610                      ldaa            #COL0           ; mascara para la columna cero
       2929 A702                      staa            MASK_COL,X
       292B 4F                        clra                            ; contador de columna
       292C A701                      staa            COL_CONT,X
       292E A700                      staa            KB_PRESS,X      ; variable con numero de tecla
                      
                      * loop para barrer y mirar teclado matricial
       2930 86FF      kb_loop         ldaa            #NONE
       2932 5A                        decb                            
       2933 2B2C                      bmi             kb_end          ; me fijo si recorrio todas las filas
       2935 A603                      ldaa            MASK_FIL,X      ; enmascaro la siguiente fila
       2937 B71003                    staa            ROWS
       293A 0D                        sec                             ; carry para rotacion de byte
       293B 49                        rola                            
       293C A703                      staa            MASK_FIL,X      ; cambio variable mascara para la proxima fila
                      
       293E 8604                      ldaa            #LENGTH
       2940 A701                      staa            COL_CONT,X
       2942 8610                      ldaa            #COL0           ; inicializo variable para barrer
       2944 A702                      staa            MASK_COL,X      ; la columna
                      
       2946 6A01      kb_col          dec             COL_CONT,X      ; me fijo si barrio todas las columnas
       2948 2BE6                      bmi             kb_loop
       294A B61005                    ldaa            COLS            ; leo el puerto de columnas
       294D A402                      anda            MASK_COL,X      ; enmascaro columna
       294F 2707                      beq             kb_hit          ; si es nulo apretaron un boton
       2951 48                        lsla                            ; cambio mascara para proxima columna
       2952 A702                      staa            MASK_COL,X
       2954 6C00                      inc             KB_PRESS,X      ; incremento numero de tecla
       2956 20EE                      bra             kb_col
                      
       2958 A600      kb_hit          ldaa            KB_PRESS,X      ; apretaron una tecla, lo comparo
       295A B1425E                    cmpa            kb_buff         ; con la que fue apretada antes
       295D 2602                      bne             kb_end          ; aviso por led
                      
       295F 86FF      kb_none         ldaa            #NONE           ; en caso de que no apreten nada
                      
       2961 B7425E    kb_end          staa            kb_buff         ; actualizo variable
                                              
                      * elimino variables locales
       2964                           %RELMEM         1
       2964                           IF 1 > 0
       2964 30                        tsx
       2965 8F                        xgdx
       2966 C30001                    addd    #1
       2969 8F                        xgdx
       296A 35                        txs
       296B                           ENDIF           
       296B                           %RELMEM         1
       296B                           IF 1 > 0
       296B 30                        tsx
       296C 8F                        xgdx
       296D C30001                    addd    #1
       2970 8F                        xgdx
       2971 35                        txs
       2972                           ENDIF           
       2972                           %RELMEM         1
       2972                           IF 1 > 0
       2972 30                        tsx
       2973 8F                        xgdx
       2974 C30001                    addd    #1
       2977 8F                        xgdx
       2978 35                        txs
       2979                           ENDIF           
       2979                           %RELMEM         1
       2979                           IF 1 > 0
       2979 30                        tsx
       297A 8F                        xgdx
       297B C30001                    addd    #1
       297E 8F                        xgdx
       297F 35                        txs
       2980                           ENDIF           
                      
                      * restore de registros
       2980 33                        pulb
       2981 38                        pulx
                                      
       2982 39                        rts
                      
                      *----------------------------------------------
                      * init_kb: subrutina para inicializar teclado
                      *----------------------------------------------
                      
       2983           init_kb         EQU             *
                      
                      * back up de registros
       2983 36                        psha
                      
                      * inicializacion de teclado             
       2984 86FF                      ldaa            #NONE
       2986 B7425E                    staa            kb_buff
                      
                      * restore de registros
       2989 32                        pula
       298A 39                        rts
                      
                      *--------------------------------
                      * Inicializa display (y teclado)
                      *--------------------------------
                      
       298B           initdis         EQU             *
                              
       298B 36                        psha
                      
       298C 860F                      ldaa            #$0F       ;set for input/output
       298E B71007                    staa            DDRC       ;C output
       2991 B61002                    ldaa            PIOC       ;get current value
       2994 84FE                      anda            #$FE       ;reset bit 0 to 0
       2996 B71002                    staa            PIOC       ;send it out
                              
       2999 32                        pula
       299A 39                        rts
                      
                      *--------------------------------------------------------------
                      * sh_value: toma un string, desplaza los caracteres a izquierda
                      *           y agrega un nuevo caracter
                      *           IX: direccion del string
                      *           AccA: nuevo valor
                      *--------------------------------------------------------------
                      
       299B           sh_value        EQU             *
                      
                      * back up de registros
       299B 3C                        pshx
       299C 37                        pshb
                      
                      * loop para desplazar caracteres
       299D E601      v_loop          ldab            1,X             ;tomo elemento siguiente en string
       299F 2705                      beq             v_end
       29A1 E700                      stab            0,X             ;y lo guardo en posicion actual
       29A3 08                        inx
       29A4 20F7                      bra             v_loop          
                      
       29A6 A700      v_end           staa            0,X             ;ingresa nuevo caracter en cadena desplazada
                      
                      * restore de registros
       29A8 33                        pulb
       29A9 38                        pulx
       29AA 39                        rts
                      
                      *----------------------------------------------
                      * toAscii: toma numero de boton del teclado
                      *          y devuelve su correspondiente en
                      *          ascii
                      *          toma  AccB: valor del boton
                      *          devuelve AccB: ascii
                      *----------------------------------------------
                      
       29AB           to_ascii        EQU             *
                      
                      * back up de registros
       29AB 3C                        pshx
                      
                      * inicializacion de variables
       29AC CE4275                    ldx             #kb_ascii       ; apunto a string con caracteres asignados por tecla
       29AF 3A                        abx                             ; me muevo en string con indice y recupero segun tecla presionada
       29B0 E600                      ldab            0,X             
                      
                      * restore de registros
       29B2 38                        pulx
       29B3 39                        rts
                      
                      *--------------------------------
                      *Outchar: ACCA: Carácter a enviar
                      *           ACCB: Posición (0-3)
                      *--------------------------------
       29B4           outchar         EQU             *
                              
       29B4 F71005                    stab            PORTCL          ;set the position 
       29B7 B71004                    staa            PORTB           ;set the character
                              
       29BA 39                        rts
                      
                      *-------------------------
                      * clr_dis: Limpia display
                      *-------------------------
                      
       29BB           clr_dis         EQU             *
                      
                      * hago back up de registros
       29BB 37                        pshb
       29BC 36                        psha
                      
                      * inicializo el contador de posiciones
       29BD C604                      ldab            #LENGTH
                      
                      * loop para limpiar cada posicion
       29BF 5A        clrloop         decb
       29C0 2B07                      bmi             clr_end         ; si no pase por todas las posiciones
       29C2 8620                      ldaa            #ESP            ; cargo en esta posicion espacio blanco
       29C4 BD29B4                    jsr             outchar         ; y lo envio
       29C7 20F6                      bra             clrloop         
                      
                      * restore de registros
       29C9 32        clr_end         pula
       29CA 33                        pulb
       29CB 39                        rts
                      
                      *-------------------------------------------------------
                      * clrarray: subrutina para limpiar un array dado
                      * parametros: IX direccion del array
                      *             AccA cantidad de elementos del array
                      *             AccB caracter a poner
                      *-------------------------------------------------------
                      
       29CC           clrarray        EQU             *
                                      
       29CC 4A        cra_loop        deca                            ;contador con cantidad de elementos a limpiar
       29CD 2705                      beq             cra_end
       29CF E700                      stab            0,X             ;reemplazar elemento por caracter a poner
       29D1 08                        inx
       29D2 20F8                      bra             cra_loop
                                      
       29D4 E700      cra_end         stab            0,X
       29D6 39                        rts
                      
                      *=============================================
                      *Subrutina winner: Recibe por stack un 1 si
                      *gano el player 1 y un 0 si gano el player 2.
                      *=============================================
                      
                                      
       29D7           winner          EQU             *
       29D7                           %BACKUP
       29D7 36                        psha
       29D8 37                        pshb
       29D9 3C                        pshx
       29DA 183C                      pshy
       29DC 30                        tsx
       29DD A608                      ldaa            8,x     ;recupero parametro enviado por stack
       29DF 2708                      beq             win2
                      
       29E1           win1            EQU             *
       29E1 CE42BA                    ldx             #string1
       29E4 BD28EE                    jsr             show
       29E7 2006                      bra             end
                      
       29E9           win2            EQU             *
       29E9 CE42BF                    ldx             #string2
       29EC BD28EE                    jsr             show
                              
       29EF           end             EQU             *
       29EF                           %RESTORE
       29EF 1838                      puly
       29F1 38                        pulx
       29F2 33                        pulb
       29F3 32                        pula
       29F4 39                        rts
                      
                      *=============================================
                      *Subrutina turn: Recibe por stack un 1 si
                      *juega el player 1 y un 0 si juega el player 2.
                      *=============================================
                      
       29F5           turn            EQU             *
       29F5                           %BACKUP
       29F5 36                        psha
       29F6 37                        pshb
       29F7 3C                        pshx
       29F8 183C                      pshy
       29FA 30                        tsx
       29FB A608                      ldaa            8,x
       29FD 2708                      beq             p2_t
                      
       29FF           p1_t            EQU             *
       29FF CE42C4                    ldx             #play1
       2A02 BD28EE                    jsr             show
       2A05 20E8                      bra             end
                              
       2A07           p2_t            EQU             *
       2A07 CE42C9                    ldx             #play2
       2A0A BD28EE                    jsr             show
                      
       2A0D           end_t           EQU             *
       2A0D                           %RESTORE        
       2A0D 1838                      puly
       2A0F 38                        pulx
       2A10 33                        pulb
       2A11 32                        pula
       2A12 39                        rts
                      
                      *************
                      * VARIABLES *
                      *************
                      ***STACK***
       4000                           ORG             $4000
       4000           stack           RMB             600
       4257           STACKP          EQU             *-1
                      
                      ***MAIN VARIABLES***
       4258           player          RMB             1               ;variable de jugador
       4259           letra_p         RMB             1
       425A           num_p           RMB             1
       425B           cant_sub        RMB             1
       425C           cant_shi        RMB             1
       425D           tot_pos         RMB             1
                      
                      ***KEYBOARD VARIABLES***
       425E           kb_buff         RMB             1
       425F           buffer          RMB             1
                      
       4260 20202020  string          FCC             '    '
       4264 00                        FCB             0
                      
       4265 0A0B0CFF  kb_value        FCB             10,11,12,255,13,14,15,255,1,2,3,255,4,5,6,255
       4269 0D0E0FFF
       426D 010203FF
       4271 040506FF
                      
       4275 41424320  kb_ascii        FCC             'ABC DEF 123 456'
       4279 44454620
       427D 31323320
       4281 343536
                      
                      ***ARREGLO DE BARCOS TOCADOS***
       0005           SUB             EQU             5
       0004           SHI             EQU             4
       0002           EL_SIZE         EQU             2
       000D           MAX_ATT         EQU             SUB*1+SHI*2
       001B           HITSIZE         EQU             EL_SIZE*MAX_ATT+1
       4284           hitlist         RMB             HITSIZE*2
                      
                      ***DISPLAY MESSAGES VARIABLES***
       42BA 57494E31  string1         FCC     'WIN1'
       42BE 00                        FCB     00
       42BF 57494E32  string2         FCC     'WIN2'
       42C3 00                        FCB     00
                      
       42C4 2D50312D  play1           FCC     '-P1-'
       42C8 00                        FCB     0
       42C9 2D50322D  play2           FCC     '-P2-'
       42CD 00                        FCB     0
                      
       42CE 5355424D  subm            FCC     'SUBM'
       42D2 00                        FCB     0
       42D3 43415247  carg            FCC     'CARG'
       42D7 00                        FCB     0
                      
       42D8           arry_pr         RMB             N_PR*SA_SIZE
       43A0           arry_aux        RMB             N_AUX*SA_SIZE
                      
       43A8 53554231  msg_sub         FCC             'SUB1'
       43AC 00                        FCB             0
       43AD 53554232                  FCC             'SUB2'
       43B1 00                        FCB             0
       43B2 53554233                  FCC             'SUB3'
       43B6 00                        FCB             0
       43B7 53554234                  FCC             'SUB4'
       43BB 00                        FCB             0
       43BC 53554235                  FCC             'SUB5'
       43C0 00                        FCB             0
                      
       43C1 43415231  msg_shi         FCC             'CAR1'
       43C5 00                        FCB             0       
       43C6 43415232                  FCC             'CAR2'
       43CA 00                        FCB             0       
       43CB 43415233                  FCC             'CAR3'
       43CF 00                        FCB             0
       43D0 43415234                  FCC             'CAR4'
       43D4 00                        FCB             0
                      
                      ***MATRICES DE TABLERO DE CADA JUGADOR***
       5000                           ORG             $5000
       5000 03030303  p_board         FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       5004 0303
       5006 03030303                  FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       500A 0303
       500C 03030303                  FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       5010 0303
       5012 03030303                  FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       5016 0303
       5018 03030303                  FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       501C 0303
       501E 03030303                  FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       5022 0303
                      
       5024 03030303                  FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       5028 0303
       502A 03030303                  FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       502E 0303
       5030 03030303                  FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       5034 0303
       5036 03030303                  FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       503A 0303
       503C 03030303                  FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
       5040 0303
       5042 03030303                  FCB             WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP,WAT_NP
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
     ytes:   2798         # main_bn #
     RC:     F051         ###########
 