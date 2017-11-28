################################################################################
#                                                                              #
#    Micro Series 6801 Assembler V2.00/DOS              28/Nov/17  12:37:30    #
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
       0070           SUNK_SUB        EQU             01110000b       ;rojo para submarinos
       00B8           HIT             EQU             10111000b       ;amarillo
       00B0           SUNK_SHI        EQU             10110000b       ;rojo para cargueros
                      
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
       2003 BD29AB                    jsr             initdis         ;inicialización de display 7 seg
       2006 BD29A3                    jsr             init_kb         ;inicialización de teclado
                      
                      * LIMPIAR ARREGLO DE BARCOS ATACADOS
       2009 CE4284                    ldx             #hitlist
       200C 3C                        pshx                            ;se envía inicio de arreglo de barcos a poner en 0
       200D 863E                      ldaa            #(HITSIZE*2)
       200F 36                        psha                            ;se envía el tamaño del arreglo a limpiar
       2010 BD2841                    jsr             clrzone
       2013                           %CLRST          3
                      
                      * DEFINIR CANTIDAD DE BARCOS A UTILIZAR
       2016 BD26DE                    jsr             def_ships
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
                      
       204D           main            EQU             *
                      * ELEGIR JUGADOR
       204D BD2859                    jsr             whoplays        ;se recibirá en AccA el jugador a atacar a continuación
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
       2062 BD2735                    jsr             mirror          ;se envía el tablero del jugador atacado al display
       2065                           %CLRST          6
                      
                      * INGRESAR POSICIÓN A ATACAR
       206B BD286E                    jsr             input           ;se llama a la subrutina de interfaz de usuario
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
       2084 BD27C0                    jsr             play_src        ;se recibirá el carry en 1 si la posición corresponde a un barco ya atacado
       2087                           %CLRST          5               ;se limpia el stack
       208C 25BF                      bcs             main            ;si la posición ya fue jugada, se pierde el turno y pasa al siguiente
                      
                      * MANDAR QUE JUGADOR ESTÁ JUGANDO AL DISPLAY
       208E B64258                    ldaa            player
       2091 36                        psha                            ;se envía el jugador que está siendo atacado
       2092 BD2A15                    jsr             turn
       2095                           %CLRST          1
                      
                      * BUSCAR EL BYTE ATACADO EN EL TABLERO DEL JUGADOR ATACADO
       2096 B64259                    ldaa            letra_p
       2099 36                        psha                            ;se envía le letra jugada
       209A F6425A                    ldab            num_p
       209D 37                        pshb                            ;se envía el número jugado
       209E CE5000                    ldx             #p_board
       20A1 3C                        pshx                            ;se envía la dirección de inicio de los arreglos de tablero
       20A2 F64258                    ldab            player
       20A5 37                        pshb                            ;se envía el jugador a atacar
       20A6 BD275E                    jsr             fetchpos
       20A9                           %CLRST          4               ;se limpia el stack
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
       20C3 BD2476                    jsr             gameplay
       20C6                           %CLRST          8               ;se limpia el stack
                      
                      * ACTUALIZAR DISPLAY CON LA POSICIÓN YA ATACADA
       20CE CE5000                    ldx             #p_board
       20D1 3C                        pshx                            ;se envía el inicio de las matrices de tableros
       20D2 B64258                    ldaa            player
       20D5 36                        psha                            ;se envía el jugador atacado
       20D6 CE5F00                    ldx             #display
       20D9 3C                        pshx                            ;se envía la matriz de display (destino a copiar)
       20DA 8624                      ldaa            #ARRAY_L
       20DC 36                        psha                            ;se envía el tamaño de la matriz tablero
       20DD BD2735                    jsr             mirror          ;se envía el tablero del jugador atacado al display
       20E0                           %CLRST          6
                      
                      * CONTAR BARCOS ATACADOS A VER SI EL JUGADOR GANÓ               
       20E6 CE4284                    ldx             #hitlist
       20E9 3C                        pshx                            ;se envía la posición del primer arreglo de barcos tocados
       20EA B64258                    ldaa            player
       20ED 36                        psha                            ;se envía el jugador a atacar
       20EE BD27F3                    jsr             play_cnt
       20F1                           %CLRST          2
       20F3 32                        pula                            ;se recibe la cantidad de bytes contados
       20F4 B1425D                    cmpa            tot_pos         
       20F7 2703                      beq             end_game        ;si se llegó al máximo de barcos, se termina el juego
                      
       20F9 7E204D                    jmp             main
                      
                      * MANDAR MENSAJE A DISPLAY DE QUIÉN GANÓ
       20FC           end_game        EQU             *
       20FC B64258                    ldaa            player
       20FF 36                        psha
       2100 BD29F7                    jsr             winner
       2103                           %CLRST          1
                      
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
       2122 CE42E0                    ldx             #arry_pr
       2125 86C8                      ldaa            #N_PR*SA_SIZE
       2127 C600                      ldab            #0
       2129 BD29EC                    jsr             clrarray
                      
       212C                           %CLR_AUX
                      
                      * termine de entrar los barcos?
       2136 18A603    sa_check        ldaa            SA_CONT,Y
       2139 18A104                    cmpa            SA_TOT,Y
       213C 2603                      bne             sa_show
       213E                           %GOTO           sa_end  
                      
                      * display mensaje para indicar barco
       2141 18A10F    sa_show         cmpa            SA_CSUB,Y
       2144 240C                      bhs             show_shi                ;me fijo que tipod e barco es para elegir
       2146 CE43B0    show_sub        ldx             #msg_sub                ;uno u otro string
       2149 8605                      ldaa            #5
       214B 18E603                    ldab            SA_CONT,Y
       214E 3D                        mul
       214F 3A                        abx
       2150 200F                      bra             show_jmp
       2152 CE43C9    show_shi        ldx             #msg_shi
       2155 18A603                    ldaa            SA_CONT,Y
       2158 18E60F                    ldab            SA_CSUB,Y               ;cambio el corrimiento adentro de los strings
       215B 10                        sba
       215C 16                        tab
       215D 8605                      ldaa            #5
       215F 3D                        mul
       2160 3A                        abx
       2161 BD290E    show_jmp        jsr             show                    ;y luego imprimo el mensaje
       2164                           %EXPECT         B_reset
                      
                      * espera ingreso de coordenada
       216E BD286E    sa_again        jsr             input                   ;espero una coordenada
                      
       2171 18A700                    staa            SA_FIL,Y                ;la guardo en memoria temporal
       2174 18E701                    stab            SA_COL,Y
                      
                      * verifico que no sea una posicion prohibida
       2177                           %IS_PROH                                ;me fijo si esta en la lista de posiciones prohibidas
       2186 25E6                      bcs             sa_again
                      
                      * me fijo que tipo de barco se ingresa
       2188 36                        psha
       2189 18A603                    ldaa            SA_CONT,Y
       218C 18A10F                    cmpa            SA_CSUB,Y
       218F 32                        pula
       2190 2503                      blo             sa_sub                  ;segun que tipo de barco manejo distinto
       2192                           %GOTO           sa_shi                  ;porque el carguero tiene dos partes
                      
                      * escribo en matriz el submarino
       2195           sa_sub          %PROH_POS                               ;si es submarino agrego a prohibida la posicion
       21AE                           %PROH_PER                               ;y tambien prohibo la periferia del submarino
       2219                           %SET_POS        BY_SUB                  ;grabo ese barco en la matriz
       2232 186C03                    inc             SA_CONT,Y
       2235                           %REFRESH                                ;y luego refresco la pantalla para que lo vea el jugador
       2250                           %GOTO           sa_check
                      
       2253 186D02    sa_shi          tst             SA_CARG,Y               ;si es carguero me tengo que fijar
       2256 2703                      beq             sa_part1                ;que parte del carguero es
       2258                           %GOTO           sa_part2
                      
       225B           sa_part1        %SET_AUX                                ;creo arreglo auxiliar con posibles partes 2
                                      
       229E 183C                      pshy
       22A0 18CE43A8                  ldy             #arry_aux
       22A4 C604                      ldab            #N_AUX
       22A6 5A        sa_loop         decb
       22A7 2B35                      bmi             sa_notok                ;me fijo que las auxiliares sean validas
                      
                      * comprobar que sea valida
       22A9 18A600                    ldaa            0,Y                     ;y tambien que esten en el tablero
       22AC 810A                      cmpa            #MIN_FIL
       22AE 2528                      blo             sa_nval
       22B0 810F                      cmpa            #MAX_FIL
       22B2 2224                      bhi             sa_nval
       22B4 18A601                    ldaa            1,Y
       22B7 8101                      cmpa            #MIN_COL
       22B9 251D                      blo             sa_nval
       22BB 8106                      cmpa            #MAX_COL
       22BD 2219                      bhi             sa_nval
                              
       22BF 37                        pshb
       22C0 18A600                    ldaa            0,Y
       22C3 18E601                    ldab            1,Y
       22C6                           %IS_PROH
       22D5 33                        pulb
       22D6 2415                      bcc             sa_ok
       22D8 1808      sa_nval         iny
       22DA 1808                      iny
       22DC 20C8                      bra             sa_loop
                      
       22DE 1838      sa_notok        puly
       22E0                           %CLR_AUX
       22EA                           %GOTO           sa_again
                      
       22ED 1838      sa_ok           puly
       22EF 18A600                    ldaa            SA_FIL,Y
       22F2 18E601                    ldab            SA_COL,Y
       22F5                           %PROH_POS                               ;prohibo la posicion de la parte uno
       230E 186C02                    inc             SA_CARG,Y
       2311                           %SET_POS        BY_SHI                  ;grabo la posicion de la parte uno
       232A                           %GOTO           sa_again
       232D CE43A8    sa_part2        ldx             #arry_aux               ;me fijo que ingreso una parte dos permitida
       2330 8604                      ldaa            #N_AUX
       2332 4A        sa_verf         deca
       2333 2A03                      bpl             sa_chk          
       2335                           %GOTO           sa_again
       2338 E600      sa_chk          ldab            0,X
       233A 18E100                    cmpb            SA_FIL,Y
       233D 2609                      bne             sa_not
       233F E601                      ldab            1,X
       2341 18E101                    cmpb            SA_COL,Y
       2344 2602                      bne             sa_not
       2346 2004                      bra             sa_good
       2348 08        sa_not          inx
       2349 08                        inx
       234A 20E6                      bra             sa_verf
                      
       234C 183C      sa_good         pshy
       234E 18CE43A8                  ldy             #arry_aux
       2352 C604                      ldab            #4
       2354 5A        sa_loop2        decb
       2355 2B23                      bmi             sa_done
       2357 37                        pshb
       2358 18A600                    ldaa            0,Y
       235B 18E601                    ldab            1,Y
       235E                           %PROH_POS                               ;prohibo la periferia de la parte uno
       2377 33                        pulb
       2378 20DA                      bra             sa_loop2
                      
       237A 1838      sa_done         puly
       237C 18A600                    ldaa            SA_FIL,Y
       237F 18E601                    ldab            SA_COL,Y
       2382                           %PROH_POS                               ;prohibo la parte dos
       239B                           %PROH_PER                               ;prohibo la periferia de la parte dos
       2406                           %SET_POS        BY_SHI
       241F 186C03                    inc             SA_CONT,Y
       2422 186F02                    clr             SA_CARG,Y
       2425                           %CLR_AUX
       242F                           %REFRESH                                ;refresco display para visualizar al jugador
       244A                           %GOTO           sa_check
                      
                      * libero variables temporales
       244D           sa_end          %FIX_ARRY                               ;correccion en matrices por visualizacion
       245E                           %CLS                                    ;preliminar de la flota del jugador
       246B 32                        pula
       246C 32                        pula
       246D 32                        pula
       246E 32                        pula
       246F 32                        pula
                      
                      * restore de registros
       2470 1838                      puly
       2472 38                        pulx
       2473 33                        pulb
       2474 32                        pula
                      
       2475 39                        rts
                      
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
       2476           gameplay        EQU             *
       2476                           %BACKUP
                                      
       247B 30                        tsx                             ;se usa el IX como frame pointer
       247C A60D                      ldaa            13,x            ;se carga en AccA el byte a analizar
       247E 84C0                      anda            #11000000b      ;se enmascan los bits 0 a 5, para analizar los más significativos
                      
       2480           gp_water        EQU             *
       2480 2620                      bne             gp_sub          ;se estudia si era agua
       2482                           %WRITEPOS       #WATER          ;se cambia el color en el tablero
       249C                           %RESTORE
       24A1 39                        rts
                      
       24A2           gp_sub          EQU             *
       24A2 8140                      cmpa            #01000000b      ;se estudia si era un submarino
       24A4 2636                      bne             gp_shi_r
       24A6                           %WRITEPOS       #SUNK_SUB       ;se cambia el color en el tablero
       24C0                           %PLAY_WRT                       ;se agraga la posición al array de barcos atacados
       24D6                           %RESTORE
       24DB 39                        rts
                      
       24DC           gp_shi_r        EQU             *               ;si no era ninguno de los anteriores, es un carguero
       24DC E60B                      ldab            11,x
       24DE C106                      cmpb            #MAX_C
       24E0 276D                      beq             gp_shi_l        ;si está en el borde derecho, se continúa
       24E2 E60C                      ldab            12,x
       24E4 37                        pshb                            ;se envía la letra (fila)
       24E5 E60B                      ldab            11,x
       24E7 5C                        incb                            ;se avanza a la derecha
       24E8 37                        pshb                            ;se envía el num (columna)
       24E9 1AEE0E                    ldy             14,x
       24EC 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       24EE E608                      ldab            8,x
       24F0 37                        pshb                            ;se envía el jugador atacado
       24F1 BD27C0                    jsr             play_src        ;se busca la posición de la derecha
       24F4                           %CLRST          5
       24F9 2454                      bcc             gp_shi_l        ;si no había barco atacado en la derecha, se busca la izquierda
       24FB                           %WRITEPOS       #SUNK_SHI       
       2515 6C0B                      inc             11,x
       2517                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       2531 6A0B                      dec             11,x            ;se recupera el dato
       2533                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       2549                           %RESTORE
       254E 39                        rts
                      
       254F           gp_shi_l        EQU             *
       254F E60B                      ldab            11,x
       2551 C101                      cmpb            #MIN_C
       2553 276D                      beq             gp_shi_u        ;si está en el borde izquierdo, se continúa
       2555 E60C                      ldab            12,x
       2557 37                        pshb                            ;se envía la letra (fila)
       2558 E60B                      ldab            11,x
       255A 5A                        decb                            ;se retrocede a la izquierda
       255B 37                        pshb                            ;se envía el número (columna)
       255C 1AEE0E                    ldy             14,x
       255F 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       2561 E608                      ldab            8,x
       2563 37                        pshb                            ;se envía el jugador atacado
       2564 BD27C0                    jsr             play_src        ;se busca la posición de la izquierda
       2567                           %CLRST          5
       256C 2454                      bcc             gp_shi_u        ;si no había barco atacado a la izquierda, se busca arriba
       256E                           %WRITEPOS       #SUNK_SHI       
       2588 6A0B                      dec             11,x
       258A                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       25A4 6C0B                      inc             11,x            ;se recupera el dato
       25A6                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       25BC                           %RESTORE
       25C1 39                        rts
                      
       25C2           gp_shi_u        EQU             *
       25C2 E60C                      ldab            12,x
       25C4 C10A                      cmpb            #MIN_F
       25C6 276D                      beq             gp_shi_d        ;si está en el borde superior, se continúa
       25C8 E60C                      ldab            12,x
       25CA 5A                        decb                            ;se va a la posición de arriba
       25CB 37                        pshb                            ;se envía la letra (fila)
       25CC E60B                      ldab            11,x
       25CE 37                        pshb                            ;se envía el num (columna)
       25CF 1AEE0E                    ldy             14,x
       25D2 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       25D4 E608                      ldab            8,x
       25D6 37                        pshb                            ;se envía el jugador atacado
       25D7 BD27C0                    jsr             play_src        ;se busca la posición de arriba
       25DA                           %CLRST          5
       25DF 2454                      bcc             gp_shi_d        ;si no había barco atacado arriba, se busca abajo
       25E1                           %WRITEPOS       #SUNK_SHI       
       25FB 6A0C                      dec             12,x
       25FD                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       2617 6C0C                      inc             12,x            ;se recupera el dato
       2619                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       262F                           %RESTORE
       2634 39                        rts
                      
       2635           gp_shi_d        EQU             *
       2635 E60C                      ldab            12,x
       2637 C10F                      cmpb            #MAX_F
       2639 276D                      beq             gp_shi_h        ;si está en el borde inferior, se continúa
       263B E60C                      ldab            12,x
       263D 5C                        incb                            ;se va a la posición de abajo
       263E 37                        pshb                            ;se envía la letra (fila)
       263F E60B                      ldab            11,x
       2641 37                        pshb                            ;se envía el num (columna)
       2642 1AEE0E                    ldy             14,x
       2645 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       2647 E608                      ldab            8,x
       2649 37                        pshb                            ;se envía el jugador atacado
       264A BD27C0                    jsr             play_src        ;se busca la posición de abajo
       264D                           %CLRST          5
       2652 2454                      bcc             gp_shi_h        ;si no había barco atacado abajo, el carguero está tocado
       2654                           %WRITEPOS       #SUNK_SHI       
       266E 6C0C                      inc             12,x
       2670                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       268A 6A0C                      dec             12,x            ;se recupera el dato
       268C                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       26A2                           %RESTORE
       26A7 39                        rts
                      
       26A8           gp_shi_h        EQU             *
       26A8                           %WRITEPOS       #HIT            ;se marca al carguero como tocado
       26C2                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       26D8                           %RESTORE
       26DD 39                        rts
                      
                      *==========================================================================
                      * subrutina def_ships
                      * Función: Determina los valores de cargueros y submarinos a utilizar.
                      * Recibe: Nada
                      * Devuelve: Cantidad de cargueros y submarinos en memorias correspondientes.
                      * Requiere:
                      *==========================================================================
       26DE           def_ships       EQU             *
       26DE                           %BACKUP
                      
       26E3           wait_nsu        EQU             *
       26E3 CE42D6                    ldx             #subm
       26E6 BD290E                    jsr             show
                      
       26E9 BD2922                    jsr             read_kb
       26EC 4D                        tsta
       26ED 2BF4                      bmi             wait_nsu
       26EF 36                        psha
       26F0                           %EXPECT         B_fire
       26FA 32                        pula
                      
       26FB CE4265                    ldx             #kb_value
       26FE 16                        tab
       26FF 3A                        abx
       2700 A600                      ldaa            0,x             ;se obtiene el valor ingresado ya codificado
                      
       2702 8105                      cmpa            #SUB            ;se controla que no sea menor al máximo de submarinos
       2704 22DD                      bhi             wait_nsu
       2706 B7425B                    staa            cant_sub        ;se guarda la cantidad de submarinos
                      
       2709           wait_nsh        EQU             *
       2709 CE42DB                    ldx             #carg
       270C BD290E                    jsr             show
                      
       270F BD2922                    jsr             read_kb
       2712 4D                        tsta
       2713 2BF4                      bmi             wait_nsh
       2715 36                        psha
       2716                           %EXPECT         B_fire
       2720 32                        pula
                      
       2721 CE4265                    ldx             #kb_value
       2724 16                        tab
       2725 3A                        abx
       2726 A600                      ldaa            0,x             ;se obtiene el valor ingresado ya codificado
                      
       2728 8105                      cmpa            #SHI            ;se controla que no sea menor al máximo de submarinos
       272A 22DD                      bhi             wait_nsh
       272C B7425C                    staa            cant_shi        ;se guarda la cantidad de submarinos
                      
       272F                           %RESTORE
       2734 39                        rts
                      
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
                      
       2735           mirror          EQU             *       
                              
       2735                           %BACKUP
       273A 30                        tsx                             ;creo frame pointer
       273B A60B                      ldaa            11,x            ;cargo número de jugador
       273D E608                      ldab            8,x             ;cargo longitud de la copia (del array a copiar)
       273F 3D                        mul                             ;obtengo offset de jugador
       2740 E30C                      addd            12,x            ;sumo offset de jugador
       2742 ED0C                      std             12,x            ;guardo la dirección con offset del array fuente
                      
       2744 A608                      ldaa            8,x             ;cargo longitud de copia en contador
       2746 2710                      beq             mi_end          ;si es cero, vuelvo desde subrutina
       2748 1AEE09                    ldy             9,x             ;cargo índice a destino de la copia.
       274B EE0C                      ldx             12,x            ;cargo índice a array origen de copia.
                      
       274D           mi_loop         EQU             *
                      
       274D E600                      ldab            0,x             ;cargo byte a copiar
       274F 18E700                    stab            0,y             ;copio byte en destino
       2752 08                        inx                             ;siguiente byte
       2753 1808                      iny                             ;siguiente byte
       2755 4A                        deca                            ;decremento contador
       2756 26F5                      bne             mi_loop
                      
       2758           mi_end          EQU             *
                      
       2758                           %RESTORE
                              
       275D 39                        rts     
                                      
                                                      
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
                      
       275E           fetchpos        EQU             *
                      
       275E                           %BACKUP
       2763 30                        tsx                             ;creo frame pointer
       2764 A608                      ldaa            8,x             ;cargo numero de jugador
       2766 C624                      ldab            #ARRAY_L        ;cargo longitud del arreglo (MAX 255 bytes)
       2768 3D                        mul                             ;obtengo offset de jugador
       2769 E309                      addd            9,x             ;sumo offset de jugador a top of array
       276B ED09                      std             9,x             ;obtengo direccion al arreglo del jugador
                      
       276D A60C                      ldaa            12,x            ;cargo coordenada letra (FILA)
       276F C60A                      ldab            #NORM           ;cargo valor de normalización de filas
       2771 10                        sba                             ;obtengo numero de fila
       2772 C606                      ldab            #COL            ;cargo longitud de fila
       2774 3D                        mul                             ;obtengo offset de fila
       2775 E309                      addd            9,x             ;sumo offset de fila a la dirección del arreglo del jugador
       2777 ED09                      std             9,x             ;obtengo dirección a la fila que se desea
                      
       2779 E60B                      ldab            11,x            ;cargo coordenada (offset) de columna
       277B 5A                        decb                            ;normalizo offset de columna
       277C 4F                        clra                            ;borro AccA para tener el valor de AccB en AccD
       277D E309                      addd            9,x             ;sumo offset de columna a la dirección temporal
       277F ED09                      std             9,x             ;guardo en stack la dirección del byte a consultar
                      
       2781 1AEE09                    ldy             9,x             ;cargo en IY la dirección del byte que se desea obtener
       2784 18A600                    ldaa            0,y             ;cargo el byte a devolver en AccA
       2787 A70C                      staa            12,x            ;guardo el byte a devolver en stack
       2789                           %RESTORE
                                      
       278E 39                        rts
                                      
                                      
                      
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
                      
       278F           writepos        EQU             *
                      
       278F                           %BACKUP
       2794 30                        tsx                             ;creo frame pointer
       2795 A608                      ldaa            8,x             ;cargo numero de jugador
       2797 C624                      ldab            #ARRAY_L        ;cargo longitud del arreglo (MAX 255 bytes)
       2799 3D                        mul                             ;obtengo offset de jugador
       279A E309                      addd            9,x             ;sumo offset de jugador a top of array
       279C ED09                      std             9,x             ;obtengo direccion al arreglo del jugador
                      
       279E A60C                      ldaa            12,x            ;cargo coordenada letra (FILA)
       27A0 C60A                      ldab            #NORM           ;cargo valor de normalización de filas
       27A2 10                        sba                             ;obtengo numero de fila
       27A3 C606                      ldab            #COL            ;cargo longitud de fila
       27A5 3D                        mul                             ;obtengo offset de fila
       27A6 E309                      addd            9,x             ;sumo offset de fila a la dirección del arreglo del jugador
       27A8 ED09                      std             9,x             ;obtengo dirección a la fila que se desea
                      
       27AA E60B                      ldab            11,x            ;cargo coordenada (offset) de columna
       27AC 5A                        decb                            ;normalizo offset de columna
       27AD 4F                        clra                            ;borro AccA para tener el valor de AccB en AccD
       27AE E309                      addd            9,x             ;sumo offset de columna a la dirección temporal
       27B0 ED09                      std             9,x             ;guardo en stack la dirección del byte a consultar
                      
       27B2 1AEE09                    ldy             9,x             ;cargo en IY la dirección donde se desea escribir
       27B5 A60D                      ldaa            13,x            ;cargo en AccA el byte que se quiere escribir en el arreglo
       27B7 18A700                    staa            0,y             ;escribo el byte sobre el arreglo
       27BA                           %RESTORE
                                      
       27BF 39                        rts
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
                      
       27C0           play_src        EQU             *               
                                              
       27C0                           %BACKUP
       27C5 30                        tsx                             ;creo frame pointer
       27C6 A608                      ldaa            8,x             ;cargo número de array (jugador)
       27C8 C61F                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       27CA 3D                        mul                             ;obtengo offset de jugador
       27CB E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       27CD 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
                      
       27CF           ps_loop         EQU             *
                      
       27CF 18A600                    ldaa            0,y             ;cargo byte1 a comparar
       27D2 2718                      beq             ps_ntfnd        ;si encontré el terminador, el valor requerido no está en el array
                      
       27D4 A10C                      cmpa            12,x            ;comparo con el byte1 a buscar
       27D6 2706                      beq             ps_next         ;si coinciden, busco el byte 2
       27D8 1808                      iny                             ;me posiciono sobre siguiente byte1 a comparar
       27DA 1808                      iny
       27DC 20F1                      bra             ps_loop         ;vuelvo a comparar
                      
       27DE           ps_next         EQU             *
                      
       27DE 18E601                    ldab            1,y             ;cargo byte a continuación del anterior
       27E1 1808                      iny                             ;me posiciono sobre siguiente byte1 a comparar
       27E3 1808                      iny
       27E5 E10B                      cmpb            11,x            ;comparo con byte2
       27E7 26E6                      bne             ps_loop         ;si no son iguales, sigo buscando
       27E9 0D                        sec                             ;set del carry para informar que se encontró el valor
       27EA 2001                      bra             ps_end
                                              
       27EC           ps_ntfnd        EQU             *
                      
       27EC 0C                        clc                             ;carry en 0 para informar que no se encontró el valor
                      
       27ED           ps_end          EQU             *
                                      
       27ED                           %RESTORE
                      
       27F2 39                        rts
                      
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
                      
       27F3           play_cnt        EQU             *
                      
       27F3                           %BACKUP
       27F8 30                        tsx                             ;creo frame pointer
       27F9 A608                      ldaa            8,x             ;cargo número de array (jugador)
       27FB C61F                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       27FD 3D                        mul                             ;obtengo offset de jugador
       27FE E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       2800 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
       2802 4F                        clra                            ;inicializo contador
                                              
       2803           pc_loop         EQU             *               
                                              
       2803 18E600                    ldab            0,y             ;cargo byte a comparar
       2806 2707                      beq             pc_end          ;si es terminador, terminé de buscar
       2808 1808                      iny                             ;si no es terminador, me posiciono sobre siguiente dato de 
       280A 1808                      iny                             ;2 bytes
       280C 4C                        inca                            ;incremento contador
       280D 20F4                      bra             pc_loop         ;vuelvo a comparar
                                              
       280F           pc_end          EQU             *       
                                              
       280F A70A                      staa            10,x            ;devuelvo valor por stack
       2811                           %RESTORE
                                      
       2816 39                        rts
                      
                      
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
                      
       2817           play_wrt        EQU             *
                      
       2817                           %BACKUP
       281C 30                        tsx                             ;creo frame pointer
       281D A608                      ldaa            8,x             ;cargo número de array (jugador)
       281F C61F                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       2821 3D                        mul                             ;obtengo offset de jugador
       2822 E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       2824 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
                      
                      
       2826           pw_loop         EQU             *
                      
       2826 18A600                    ldaa            0,y             ;cargo byte del array
       2829 2704                      beq             pw_end          ;si es cero, entonces escribo los datos
       282B 1808                      iny                             ;si no es cero, me posiciono sobre la siguiente posición
       282D 20F7                      bra             pw_loop         ;vuelvo a buscar ceros
                      
       282F           pw_end          EQU             *
                      
       282F A60C                      ldaa            12,x            ;cargo primer byte a escribir
       2831 18A700                    staa            0,y             ;guardo primer byte en memoria
       2834 1808                      iny                             ;me posiciono en el siguiente byte
       2836 A60B                      ldaa            11,x            ;cargo segundo byte a escribir
       2838 18A700                    staa            0,y             ;guardo segundo byte en memoria
       283B                           %RESTORE
                      
       2840 39                        rts
                      
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
       2841           clrzone         EQU     *
                      
       2841                           %BACKUP                         ;backup de registros en stack
                      
       2846 30                        tsx
       2847 A608                      ldaa    8,x                     ;cargo contador con longitud de zona a borrar
       2849 EE09                      ldx     9,x                     ;cargo puntero zona de memoria a borrar
       284B D600                      ldab    $00                     ;cargo el cero
       284D           clr_loop        EQU     *
       284D E700                      stab    0,x                     ;guardo caracter en posicion del array
       284F 08                        inx                             ;siguiente posición     
       2850 4A                        deca                            ;decremento contador
       2851 26FA                      bne     clr_loop                ;si no es cero, escribo en la posición siguiente.
                      
       2853                           %RESTORE                        :restauro backup en registros
                      
       2858 39                        rts
                      
                      *---------------------------------------
                      * WhoPlays: subrutina para ver que 
                      *           jugador juega
                      *---------------------------------------
                      
       2859           whoplays        EQU             *
                      
       2859 BD2922    wait_ply        jsr             read_kb
       285C 4D                        tsta
       285D 2BFA                      bmi             wait_ply
       285F 8103                      cmpa            #B_P1
       2861 2708                      beq             play_p1
       2863 8107                      cmpa            #B_P2
       2865 26F2                      bne             wait_ply
       2867 8600                      ldaa            #P1
       2869 2002                      bra             ply_end
                      
       286B 8601      play_p1         ldaa            #P2
       286D 39        ply_end         rts             
                      
                      *---------------------------------------
                      * Input: subrutina que devuelve
                      *        entrada para la batalla naval
                      * AccA: Fila de la posicion jugada
                      * AccB: Columna de la posicion jugada
                      *---------------------------------------
                      
       286E           input           EQU             *
                      
                      * back up de registros
       286E 3C                        pshx
       286F 183C                      pshy
                      
                      * creo variables locales
       2871 36                        psha
       2872 36                        psha
       2873 36                        psha
                      
                      * inicializo framepointer
       2874 1830                      tsy
                      
                      * prendo el led
       2876                           %LED_INV
                      
                      * inicializo el contador
       287E 8603      inp_rest        ldaa            #INPUTS
       2880 18A700                    staa            CONT,Y
                                      
                      * limpio el display
       2883 CE4260                    ldx             #string
       2886 8604                      ldaa            #4
       2888 C620                      ldab            #ESP
       288A BD29EC                    jsr             clrarray
                      
       288D BD29DB                    jsr             clr_dis
                      
       2890 86FF                      ldaa            #$FF
       2892 B7425F                    staa            buffer
                      
                      * lectura de teclado
       2895 BD2922    wait_kb         jsr             read_kb
       2898 4D                        tsta
       2899 2BFA                      bmi             wait_kb
       289B B1425F                    cmpa            buffer
       289E 27F5                      beq             wait_kb
       28A0 B7425F                    staa            buffer
                                      
       28A3 810F                      cmpa            #B_reset
       28A5 27D7                      beq             inp_rest
       28A7 8103                      cmpa            #B_P1
       28A9 27D3                      beq             inp_rest
       28AB 8107                      cmpa            #B_P2
       28AD 27CF                      beq             inp_rest
                      
       28AF 186A00                    dec             CONT,Y
       28B2 272B                      beq             chk_fire
       28B4 810B                      cmpa            #B_fire
       28B6 27C6                      beq             inp_rest
       28B8 36                        psha
       28B9 CE4265                    ldx             #kb_value
       28BC 16                        tab
       28BD 3A                        abx
       28BE A600                      ldaa            0,X             
       28C0 183C                      pshy
       28C2 18E600                    ldab            CONT,Y
       28C5 183A                      aby
       28C7 18A700                    staa            0,Y
       28CA 1838                      puly
                      
       28CC 33                        pulb
       28CD BD29CB                    jsr             to_ascii
       28D0 17                        tba
                                      
       28D1 CE4260                    ldx             #string
       28D4 BD29BB                    jsr             sh_value
                      
       28D7 CE4260                    ldx             #string
       28DA BD290E                    jsr             show
                      
       28DD 20B6                      bra             wait_kb
                      
       28DF 810B      chk_fire        cmpa            #B_fire
       28E1 269B                      bne             inp_rest
       28E3 18A602                    ldaa            FILA_L,Y
       28E6 810A                      cmpa            #MIN_FIL
       28E8 2594                      blo             inp_rest
       28EA 810F                      cmpa            #MAX_FIL
       28EC 2290                      bhi             inp_rest
       28EE 18A601                    ldaa            COL_L,Y
       28F1 8101                      cmpa            #MIN_COL
       28F3 2589                      blo             inp_rest
       28F5 8106                      cmpa            #MAX_COL
       28F7 2285                      bhi             inp_rest
                      
                      * apago el led
       28F9                           %LED_INV
                      
                      * libero variables locales
       2901 32                        pula
       2902 32                        pula
       2903 32                        pula
                      
       2904 18A602    return          ldaa            FILA_L,Y
       2907 18E601                    ldab            COL_L,Y         
                      
                      * restore de registros
       290A 1838                      puly
       290C 38                        pulx
                      
       290D 39                        rts
                      
                      *------------------------------------
                      * Show: Por IX direccion de string de
                      *       4 caracteres para imprimir
                      *------------------------------------
                      
       290E           show            EQU             *
                      
                      * back up de registros
       290E 3C                        pshx
       290F 37                        pshb
       2910 36                        psha
                      
                      * inicializo contador de posiciones
       2911 C604                      ldab            #LENGTH
                                      
                      * loop para enviar a cada posicion
       2913 5A        sh_loop         decb
       2914 2B08                      bmi             sh_end          ; si no pase por todas las posiciones
       2916 A600                      ldaa            0,X             ; tomo del string el caracter
       2918 BD29D4                    jsr             outchar         ; y lo imprimo en la posicion actual
       291B 08                        inx
       291C 20F5                      bra             sh_loop
                      
                      * restore de registros                          
       291E 32        sh_end          pula
       291F 33                        pulb
       2920 38                        pulx
       2921 39                        rts
                      
                      *----------------------------------------------
                      * read_kb: AccA: tecla presionada
                      *          o -1
                      *----------------------------------------------
                      
       2922           read_kb         EQU             *
                      
                      * back up de registros
       2922 3C                        pshx
       2923 37                        pshb
                      
                      * creo espacio para variables locales
       2924                           %MALLOC         1               ; variable mascara de filas
       292B                           %MALLOC         1               ; variable mascara de columnas
       2932                           %MALLOC         1               ; variable contador de columnas
       2939                           %MALLOC         1               ; variable contador de tecla
                      
                      * inicializacion de variables
       2940 30                        tsx                             ; framepointer para acceder a locales
       2941 C604                      ldab            #LENGTH         ; contador de fila
       2943 860E                      ldaa            #FIL0           ; mascara para la fila cero
       2945 A703                      staa            MASK_FIL,X      
       2947 8610                      ldaa            #COL0           ; mascara para la columna cero
       2949 A702                      staa            MASK_COL,X
       294B 4F                        clra                            ; contador de columna
       294C A701                      staa            COL_CONT,X
       294E A700                      staa            KB_PRESS,X      ; variable con numero de tecla
                      
                      * loop para barrer y mirar teclado matricial
       2950 86FF      kb_loop         ldaa            #NONE
       2952 5A                        decb                            
       2953 2B2C                      bmi             kb_end          ; me fijo si recorrio todas las filas
       2955 A603                      ldaa            MASK_FIL,X      ; enmascaro la siguiente fila
       2957 B71003                    staa            ROWS
       295A 0D                        sec                             ; carry para rotacion de byte
       295B 49                        rola                            
       295C A703                      staa            MASK_FIL,X      ; cambio variable mascara para la proxima fila
                      
       295E 8604                      ldaa            #LENGTH
       2960 A701                      staa            COL_CONT,X
       2962 8610                      ldaa            #COL0           ; inicializo variable para barrer
       2964 A702                      staa            MASK_COL,X      ; la columna
                      
       2966 6A01      kb_col          dec             COL_CONT,X      ; me fijo si barrio todas las columnas
       2968 2BE6                      bmi             kb_loop
       296A B61005                    ldaa            COLS            ; leo el puerto de columnas
       296D A402                      anda            MASK_COL,X      ; enmascaro columna
       296F 2707                      beq             kb_hit          ; si es nulo apretaron un boton
       2971 48                        lsla                            ; cambio mascara para proxima columna
       2972 A702                      staa            MASK_COL,X
       2974 6C00                      inc             KB_PRESS,X      ; incremento numero de tecla
       2976 20EE                      bra             kb_col
                      
       2978 A600      kb_hit          ldaa            KB_PRESS,X      ; apretaron una tecla, lo comparo
       297A B1425E                    cmpa            kb_buff         ; con la que fue apretada antes
       297D 2602                      bne             kb_end          ; aviso por led
                      
       297F 86FF      kb_none         ldaa            #NONE           ; en caso de que no apreten nada
                      
       2981 B7425E    kb_end          staa            kb_buff         ; actualizo variable
                                              
                      * elimino variables locales
       2984                           %RELMEM         1
       298B                           %RELMEM         1
       2992                           %RELMEM         1
       2999                           %RELMEM         1
                      
                      * restore de registros
       29A0 33                        pulb
       29A1 38                        pulx
                                      
       29A2 39                        rts
                      
                      *----------------------------------------------
                      * init_kb: subrutina para inicializar teclado
                      *----------------------------------------------
                      
       29A3           init_kb         EQU             *
                      
                      * back up de registros
       29A3 36                        psha
                      
                      * inicializacion de teclado             
       29A4 86FF                      ldaa            #NONE
       29A6 B7425E                    staa            kb_buff
                      
                      * restore de registros
       29A9 32                        pula
       29AA 39                        rts
                      
                      *--------------------------------
                      * Inicializa display (y teclado)
                      *--------------------------------
                      
       29AB           initdis         EQU             *
                              
       29AB 36                        psha
                      
       29AC 860F                      ldaa            #$0F       ;set for input/output
       29AE B71007                    staa            DDRC       ;C output
       29B1 B61002                    ldaa            PIOC       ;get current value
       29B4 84FE                      anda            #$FE       ;reset bit 0 to 0
       29B6 B71002                    staa            PIOC       ;send it out
                              
       29B9 32                        pula
       29BA 39                        rts
                      
                      *--------------------------------------------------------------
                      * sh_value: toma un string, desplaza los caracteres a izquierda
                      *           y agrega un nuevo caracter
                      *           IX: direccion del string
                      *           AccA: nuevo valor
                      *--------------------------------------------------------------
                      
       29BB           sh_value        EQU             *
                      
                      * back up de registros
       29BB 3C                        pshx
       29BC 37                        pshb
                      
                      * loop para desplazar caracteres
       29BD E601      v_loop          ldab            1,X             ;tomo elemento siguiente en string
       29BF 2705                      beq             v_end
       29C1 E700                      stab            0,X             ;y lo guardo en posicion actual
       29C3 08                        inx
       29C4 20F7                      bra             v_loop          
                      
       29C6 A700      v_end           staa            0,X             ;ingresa nuevo caracter en cadena desplazada
                      
                      * restore de registros
       29C8 33                        pulb
       29C9 38                        pulx
       29CA 39                        rts
                      
                      *----------------------------------------------
                      * toAscii: toma numero de boton del teclado
                      *          y devuelve su correspondiente en
                      *          ascii
                      *          toma  AccB: valor del boton
                      *          devuelve AccB: ascii
                      *----------------------------------------------
                      
       29CB           to_ascii        EQU             *
                      
                      * back up de registros
       29CB 3C                        pshx
                      
                      * inicializacion de variables
       29CC CE4275                    ldx             #kb_ascii       ; apunto a string con caracteres asignados por tecla
       29CF 3A                        abx                             ; me muevo en string con indice y recupero segun tecla presionada
       29D0 E600                      ldab            0,X             
                      
                      * restore de registros
       29D2 38                        pulx
       29D3 39                        rts
                      
                      *--------------------------------
                      *Outchar: ACCA: Carácter a enviar
                      *           ACCB: Posición (0-3)
                      *--------------------------------
       29D4           outchar         EQU             *
                              
       29D4 F71005                    stab            PORTCL          ;set the position 
       29D7 B71004                    staa            PORTB           ;set the character
                              
       29DA 39                        rts
                      
                      *-------------------------
                      * clr_dis: Limpia display
                      *-------------------------
                      
       29DB           clr_dis         EQU             *
                      
                      * hago back up de registros
       29DB 37                        pshb
       29DC 36                        psha
                      
                      * inicializo el contador de posiciones
       29DD C604                      ldab            #LENGTH
                      
                      * loop para limpiar cada posicion
       29DF 5A        clrloop         decb
       29E0 2B07                      bmi             clr_end         ; si no pase por todas las posiciones
       29E2 8620                      ldaa            #ESP            ; cargo en esta posicion espacio blanco
       29E4 BD29D4                    jsr             outchar         ; y lo envio
       29E7 20F6                      bra             clrloop         
                      
                      * restore de registros
       29E9 32        clr_end         pula
       29EA 33                        pulb
       29EB 39                        rts
                      
                      *-------------------------------------------------------
                      * clrarray: subrutina para limpiar un array dado
                      * parametros: IX direccion del array
                      *             AccA cantidad de elementos del array
                      *             AccB caracter a poner
                      *-------------------------------------------------------
                      
       29EC           clrarray        EQU             *
                                      
       29EC 4A        cra_loop        deca                            ;contador con cantidad de elementos a limpiar
       29ED 2705                      beq             cra_end
       29EF E700                      stab            0,X             ;reemplazar elemento por caracter a poner
       29F1 08                        inx
       29F2 20F8                      bra             cra_loop
                                      
       29F4 E700      cra_end         stab            0,X
       29F6 39                        rts
                      
                      *=============================================
                      *Subrutina winner: Recibe por stack un 1 si
                      *gano el player 1 y un 0 si gano el player 2.
                      *=============================================
                      
                                      
       29F7           winner          EQU             *
       29F7                           %BACKUP
       29FC 30                        tsx
       29FD A608                      ldaa            8,x     ;recupero parametro enviado por stack
       29FF 2708                      beq             win2
                      
       2A01           win1            EQU             *
       2A01 CE42C2                    ldx             #string1
       2A04 BD290E                    jsr             show
       2A07 2006                      bra             end
                      
       2A09           win2            EQU             *
       2A09 CE42C7                    ldx             #string2
       2A0C BD290E                    jsr             show
                              
       2A0F           end             EQU             *
       2A0F                           %RESTORE
       2A14 39                        rts
                      
                      *=============================================
                      *Subrutina turn: Recibe por stack un 1 si
                      *juega el player 1 y un 0 si juega el player 2.
                      *=============================================
                      
       2A15           turn            EQU             *
       2A15                           %BACKUP
       2A1A 30                        tsx
       2A1B A608                      ldaa            8,x
       2A1D 2708                      beq             p2_t
                      
       2A1F           p1_t            EQU             *
       2A1F CE42CC                    ldx             #play1
       2A22 BD290E                    jsr             show
       2A25 20E8                      bra             end
                              
       2A27           p2_t            EQU             *
       2A27 CE42D1                    ldx             #play2
       2A2A BD290E                    jsr             show
                      
       2A2D           end_t           EQU             *
       2A2D                           %RESTORE        
       2A32 39                        rts
                      
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
       0005           SHI             EQU             5
       0002           EL_SIZE         EQU             2
       000F           MAX_ATT         EQU             SUB*1+SHI*2
       001F           HITSIZE         EQU             EL_SIZE*MAX_ATT+1
       4284           hitlist         RMB             HITSIZE*2
                      
                      ***DISPLAY MESSAGES VARIABLES***
       42C2 57494E31  string1         FCC     'WIN1'
       42C6 00                        FCB     00
       42C7 57494E32  string2         FCC     'WIN2'
       42CB 00                        FCB     00
                      
       42CC 2D50312D  play1           FCC     '-P1-'
       42D0 00                        FCB     0
       42D1 2D50322D  play2           FCC     '-P2-'
       42D5 00                        FCB     0
                      
       42D6 5355424D  subm            FCC     'SUBM'
       42DA 00                        FCB     0
       42DB 43415247  carg            FCC     'CARG'
       42DF 00                        FCB     0
                      
       42E0           arry_pr         RMB             N_PR*SA_SIZE
       43A8           arry_aux        RMB             N_AUX*SA_SIZE
                      
       43B0 53554231  msg_sub         FCC             'SUB1'
       43B4 00                        FCB             0
       43B5 53554232                  FCC             'SUB2'
       43B9 00                        FCB             0
       43BA 53554233                  FCC             'SUB3'
       43BE 00                        FCB             0
       43BF 53554234                  FCC             'SUB4'
       43C3 00                        FCB             0
       43C4 53554235                  FCC             'SUB5'
       43C8 00                        FCB             0
                      
       43C9 43415231  msg_shi         FCC             'CAR1'
       43CD 00                        FCB             0       
       43CE 43415232                  FCC             'CAR2'
       43D2 00                        FCB             0       
       43D3 43415233                  FCC             'CAR3'
       43D7 00                        FCB             0
       43D8 43415234                  FCC             'CAR4'
       43DC 00                        FCB             0
                      
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
     ytes:   2830         # main_bn #
     RC:     088D         ###########
 