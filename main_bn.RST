################################################################################
#                                                                              #
#    Micro Series 6801 Assembler V2.00/DOS              28/Nov/17  14:33:12    #
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
       2003 BD29B1                    jsr             initdis         ;inicialización de display 7 seg
       2006 BD29A9                    jsr             init_kb         ;inicialización de teclado
                      
                      * LIMPIAR ARREGLO DE BARCOS ATACADOS
       2009 CE4284                    ldx             #hitlist
       200C 3C                        pshx                            ;se envía inicio de arreglo de barcos a poner en 0
       200D 8636                      ldaa            #(HITSIZE*2)
       200F 36                        psha                            ;se envía el tamaño del arreglo a limpiar
       2010 BD2847                    jsr             clrzone
       2013                           %CLRST          3
                      
                      * DEFINIR CANTIDAD DE BARCOS A UTILIZAR
       2016 BD26E4                    jsr             def_ships
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
       2030 BD210C                    jsr             setarmy
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
       2046 BD210C                    jsr             setarmy
       2049                           %CLRST          4
                      
                      * ENVIAR MENSAJE DE INICIO
       204D CE42D8                    ldx             #play
       2050 BD2914                    jsr             show
                      
       2053           main            EQU             *
                      * ELEGIR JUGADOR
       2053 BD285F                    jsr             whoplays        ;se recibirá en AccA el jugador a atacar a continuación
       2056 B74258                    staa            player          ;se habilita la siguiente jugada
                      
                      * ENVIAR TABLERO DE JUGADOR A ATACAR A DISPLAY
       2059 CE5000                    ldx             #p_board
       205C 3C                        pshx                            ;se envía el inicio de las matrices de tableros
       205D B64258                    ldaa            player
       2060 36                        psha                            ;se envía el jugador atacado
       2061 CE5F00                    ldx             #display
       2064 3C                        pshx                            ;se envía la matriz de display (destino a copiar)
       2065 8624                      ldaa            #ARRAY_L
       2067 36                        psha                            ;se envía el tamaño de la matriz tablero
       2068 BD273B                    jsr             mirror          ;se envía el tablero del jugador atacado al display
       206B                           %CLRST          6
                      
                      * INGRESAR POSICIÓN A ATACAR
       2071 BD2874                    jsr             input           ;se llama a la subrutina de interfaz de usuario
       2074 B74259                    staa            letra_p         ;se recibe en AccA la letra jugada (fila) y en AccB el número (columna)
       2077 F7425A                    stab            num_p           ;se guarda la jugada
                      
                      * VERIFICAR QUE NO SEA POSICIÓN DE BARCO YA ATACADA
       207A B64259                    ldaa            letra_p
       207D 36                        psha                            ;se envía la letra jugada
       207E F6425A                    ldab            num_p
       2081 37                        pshb                            ;se envía el número jugado
       2082 CE4284                    ldx             #hitlist
       2085 3C                        pshx                            ;se envía la posición del primer arreglo de barcos tocados
       2086 B64258                    ldaa            player
       2089 36                        psha                            ;se envía el jugador a atacar
       208A BD27C6                    jsr             play_src        ;se recibirá el carry en 1 si la posición corresponde a un barco ya atacado
       208D                           %CLRST          5               ;se limpia el stack
       2092 25BF                      bcs             main            ;si la posición ya fue jugada, se pierde el turno y pasa al siguiente
                      
                      * MANDAR QUE JUGADOR ESTÁ JUGANDO AL DISPLAY
       2094 B64258                    ldaa            player
       2097 36                        psha                            ;se envía el jugador que está siendo atacado
       2098 BD2A1B                    jsr             turn
       209B                           %CLRST          1
                      
                      * BUSCAR EL BYTE ATACADO EN EL TABLERO DEL JUGADOR ATACADO
       209C B64259                    ldaa            letra_p
       209F 36                        psha                            ;se envía le letra jugada
       20A0 F6425A                    ldab            num_p
       20A3 37                        pshb                            ;se envía el número jugado
       20A4 CE5000                    ldx             #p_board
       20A7 3C                        pshx                            ;se envía la dirección de inicio de los arreglos de tablero
       20A8 F64258                    ldab            player
       20AB 37                        pshb                            ;se envía el jugador a atacar
       20AC BD2764                    jsr             fetchpos
       20AF                           %CLRST          4               ;se limpia el stack
       20B3 32                        pula                            ;se toma del stack el dato del byte jugado
                      
                      * ATACAR
       20B4 CE4284                    ldx             #hitlist
       20B7 3C                        pshx                            ;se envía inicio del arreglo de barcos atacados
       20B8 36                        psha                            ;se envía el byte a jugado
       20B9 B64259                    ldaa            letra_p
       20BC 36                        psha                            ;se envía le letra jugada
       20BD F6425A                    ldab            num_p
       20C0 37                        pshb                            ;se envía el número jugado
       20C1 CE5000                    ldx             #p_board
       20C4 3C                        pshx                            ;se envía la dirección de inicio de los arreglos de tablero
       20C5 F64258                    ldab            player
       20C8 37                        pshb                            ;se envía el jugador a atacar
       20C9 BD247C                    jsr             gameplay
       20CC                           %CLRST          8               ;se limpia el stack
                      
                      * ACTUALIZAR DISPLAY CON LA POSICIÓN YA ATACADA
       20D4 CE5000                    ldx             #p_board
       20D7 3C                        pshx                            ;se envía el inicio de las matrices de tableros
       20D8 B64258                    ldaa            player
       20DB 36                        psha                            ;se envía el jugador atacado
       20DC CE5F00                    ldx             #display
       20DF 3C                        pshx                            ;se envía la matriz de display (destino a copiar)
       20E0 8624                      ldaa            #ARRAY_L
       20E2 36                        psha                            ;se envía el tamaño de la matriz tablero
       20E3 BD273B                    jsr             mirror          ;se envía el tablero del jugador atacado al display
       20E6                           %CLRST          6
                      
                      * CONTAR BARCOS ATACADOS A VER SI EL JUGADOR GANÓ               
       20EC CE4284                    ldx             #hitlist
       20EF 3C                        pshx                            ;se envía la posición del primer arreglo de barcos tocados
       20F0 B64258                    ldaa            player
       20F3 36                        psha                            ;se envía el jugador a atacar
       20F4 BD27F9                    jsr             play_cnt
       20F7                           %CLRST          2
       20F9 32                        pula                            ;se recibe la cantidad de bytes contados
       20FA B1425D                    cmpa            tot_pos         
       20FD 2703                      beq             end_game        ;si se llegó al máximo de barcos, se termina el juego
                      
       20FF 7E2053                    jmp             main
                      
                      * MANDAR MENSAJE A DISPLAY DE QUIÉN GANÓ
       2102           end_game        EQU             *
       2102 B64258                    ldaa            player
       2105 36                        psha
       2106 BD29FD                    jsr             winner
       2109                           %CLRST          1
                      
       210A           end_loop        EQU             *
       210A 20FE                      bra             end_loop
                      
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
                      
       210C           setarmy         EQU             *
                      
                      * back up de registros
       210C 36                        psha
       210D 37                        pshb
       210E 3C                        pshx
       210F 183C                      pshy
                      
                      * variables locales
       2111 36                        psha
       2112 36                        psha
       2113 36                        psha
       2114 36                        psha
       2115 36                        psha
                      
                      * inializo el framepointer
       2116 1830                      tsy
                      
                      * inicializo los contadores
       2118 186F03                    clr             SA_CONT,Y
       211B 186F02                    clr             SA_CARG,Y
       211E 18A60F                    ldaa            SA_CSUB,Y
       2121 18E610                    ldab            SA_CSHI,Y
       2124 1B                        aba
       2125 18A704                    staa            SA_TOT,Y
                      
                      * inicializo arreglos
       2128 CE42DD                    ldx             #arry_pr
       212B 86C8                      ldaa            #N_PR*SA_SIZE
       212D C600                      ldab            #0
       212F BD29F2                    jsr             clrarray
                      
       2132                           %CLR_AUX
                      
                      * termine de entrar los barcos?
       213C 18A603    sa_check        ldaa            SA_CONT,Y
       213F 18A104                    cmpa            SA_TOT,Y
       2142 2603                      bne             sa_show
       2144                           %GOTO           sa_end  
                      
                      * display mensaje para indicar barco
       2147 18A10F    sa_show         cmpa            SA_CSUB,Y
       214A 240C                      bhs             show_shi                ;me fijo que tipod e barco es para elegir
       214C CE43AD    show_sub        ldx             #msg_sub                ;uno u otro string
       214F 8605                      ldaa            #5
       2151 18E603                    ldab            SA_CONT,Y
       2154 3D                        mul
       2155 3A                        abx
       2156 200F                      bra             show_jmp
       2158 CE43C6    show_shi        ldx             #msg_shi
       215B 18A603                    ldaa            SA_CONT,Y
       215E 18E60F                    ldab            SA_CSUB,Y               ;cambio el corrimiento adentro de los strings
       2161 10                        sba
       2162 16                        tab
       2163 8605                      ldaa            #5
       2165 3D                        mul
       2166 3A                        abx
       2167 BD2914    show_jmp        jsr             show                    ;y luego imprimo el mensaje
       216A                           %EXPECT         B_reset
                      
                      * espera ingreso de coordenada
       2174 BD2874    sa_again        jsr             input                   ;espero una coordenada
                      
       2177 18A700                    staa            SA_FIL,Y                ;la guardo en memoria temporal
       217A 18E701                    stab            SA_COL,Y
                      
                      * verifico que no sea una posicion prohibida
       217D                           %IS_PROH                                ;me fijo si esta en la lista de posiciones prohibidas
       218C 25E6                      bcs             sa_again
                      
                      * me fijo que tipo de barco se ingresa
       218E 36                        psha
       218F 18A603                    ldaa            SA_CONT,Y
       2192 18A10F                    cmpa            SA_CSUB,Y
       2195 32                        pula
       2196 2503                      blo             sa_sub                  ;segun que tipo de barco manejo distinto
       2198                           %GOTO           sa_shi                  ;porque el carguero tiene dos partes
                      
                      * escribo en matriz el submarino
       219B           sa_sub          %PROH_POS                               ;si es submarino agrego a prohibida la posicion
       21B4                           %PROH_PER                               ;y tambien prohibo la periferia del submarino
       221F                           %SET_POS        BY_SUB                  ;grabo ese barco en la matriz
       2238 186C03                    inc             SA_CONT,Y
       223B                           %REFRESH                                ;y luego refresco la pantalla para que lo vea el jugador
       2256                           %GOTO           sa_check
                      
       2259 186D02    sa_shi          tst             SA_CARG,Y               ;si es carguero me tengo que fijar
       225C 2703                      beq             sa_part1                ;que parte del carguero es
       225E                           %GOTO           sa_part2
                      
       2261           sa_part1        %SET_AUX                                ;creo arreglo auxiliar con posibles partes 2
                                      
       22A4 183C                      pshy
       22A6 18CE43A5                  ldy             #arry_aux
       22AA C604                      ldab            #N_AUX
       22AC 5A        sa_loop         decb
       22AD 2B35                      bmi             sa_notok                ;me fijo que las auxiliares sean validas
                      
                      * comprobar que sea valida
       22AF 18A600                    ldaa            0,Y                     ;y tambien que esten en el tablero
       22B2 810A                      cmpa            #MIN_FIL
       22B4 2528                      blo             sa_nval
       22B6 810F                      cmpa            #MAX_FIL
       22B8 2224                      bhi             sa_nval
       22BA 18A601                    ldaa            1,Y
       22BD 8101                      cmpa            #MIN_COL
       22BF 251D                      blo             sa_nval
       22C1 8106                      cmpa            #MAX_COL
       22C3 2219                      bhi             sa_nval
                              
       22C5 37                        pshb
       22C6 18A600                    ldaa            0,Y
       22C9 18E601                    ldab            1,Y
       22CC                           %IS_PROH
       22DB 33                        pulb
       22DC 2415                      bcc             sa_ok
       22DE 1808      sa_nval         iny
       22E0 1808                      iny
       22E2 20C8                      bra             sa_loop
                      
       22E4 1838      sa_notok        puly
       22E6                           %CLR_AUX
       22F0                           %GOTO           sa_again
                      
       22F3 1838      sa_ok           puly
       22F5 18A600                    ldaa            SA_FIL,Y
       22F8 18E601                    ldab            SA_COL,Y
       22FB                           %PROH_POS                               ;prohibo la posicion de la parte uno
       2314 186C02                    inc             SA_CARG,Y
       2317                           %SET_POS        BY_SHI                  ;grabo la posicion de la parte uno
       2330                           %GOTO           sa_again
       2333 CE43A5    sa_part2        ldx             #arry_aux               ;me fijo que ingreso una parte dos permitida
       2336 8604                      ldaa            #N_AUX
       2338 4A        sa_verf         deca
       2339 2A03                      bpl             sa_chk          
       233B                           %GOTO           sa_again
       233E E600      sa_chk          ldab            0,X
       2340 18E100                    cmpb            SA_FIL,Y
       2343 2609                      bne             sa_not
       2345 E601                      ldab            1,X
       2347 18E101                    cmpb            SA_COL,Y
       234A 2602                      bne             sa_not
       234C 2004                      bra             sa_good
       234E 08        sa_not          inx
       234F 08                        inx
       2350 20E6                      bra             sa_verf
                      
       2352 183C      sa_good         pshy
       2354 18CE43A5                  ldy             #arry_aux
       2358 C604                      ldab            #4
       235A 5A        sa_loop2        decb
       235B 2B23                      bmi             sa_done
       235D 37                        pshb
       235E 18A600                    ldaa            0,Y
       2361 18E601                    ldab            1,Y
       2364                           %PROH_POS                               ;prohibo la periferia de la parte uno
       237D 33                        pulb
       237E 20DA                      bra             sa_loop2
                      
       2380 1838      sa_done         puly
       2382 18A600                    ldaa            SA_FIL,Y
       2385 18E601                    ldab            SA_COL,Y
       2388                           %PROH_POS                               ;prohibo la parte dos
       23A1                           %PROH_PER                               ;prohibo la periferia de la parte dos
       240C                           %SET_POS        BY_SHI
       2425 186C03                    inc             SA_CONT,Y
       2428 186F02                    clr             SA_CARG,Y
       242B                           %CLR_AUX
       2435                           %REFRESH                                ;refresco display para visualizar al jugador
       2450                           %GOTO           sa_check
                      
                      * libero variables temporales
       2453           sa_end          %FIX_ARRY                               ;correccion en matrices por visualizacion
       2464                           %CLS                                    ;preliminar de la flota del jugador
       2471 32                        pula
       2472 32                        pula
       2473 32                        pula
       2474 32                        pula
       2475 32                        pula
                      
                      * restore de registros
       2476 1838                      puly
       2478 38                        pulx
       2479 33                        pulb
       247A 32                        pula
                      
       247B 39                        rts
                      
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
       247C           gameplay        EQU             *
       247C                           %BACKUP
                                      
       2481 30                        tsx                             ;se usa el IX como frame pointer
       2482 A60D                      ldaa            13,x            ;se carga en AccA el byte a analizar
       2484 84C0                      anda            #11000000b      ;se enmascan los bits 0 a 5, para analizar los más significativos
                      
       2486           gp_water        EQU             *
       2486 2620                      bne             gp_sub          ;se estudia si era agua
       2488                           %WRITEPOS       #WATER          ;se cambia el color en el tablero
       24A2                           %RESTORE
       24A7 39                        rts
                      
       24A8           gp_sub          EQU             *
       24A8 8140                      cmpa            #01000000b      ;se estudia si era un submarino
       24AA 2636                      bne             gp_shi_r
       24AC                           %WRITEPOS       #SUNK_SUB       ;se cambia el color en el tablero
       24C6                           %PLAY_WRT                       ;se agraga la posición al array de barcos atacados
       24DC                           %RESTORE
       24E1 39                        rts
                      
       24E2           gp_shi_r        EQU             *               ;si no era ninguno de los anteriores, es un carguero
       24E2 E60B                      ldab            11,x
       24E4 C106                      cmpb            #MAX_C
       24E6 276D                      beq             gp_shi_l        ;si está en el borde derecho, se continúa
       24E8 E60C                      ldab            12,x
       24EA 37                        pshb                            ;se envía la letra (fila)
       24EB E60B                      ldab            11,x
       24ED 5C                        incb                            ;se avanza a la derecha
       24EE 37                        pshb                            ;se envía el num (columna)
       24EF 1AEE0E                    ldy             14,x
       24F2 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       24F4 E608                      ldab            8,x
       24F6 37                        pshb                            ;se envía el jugador atacado
       24F7 BD27C6                    jsr             play_src        ;se busca la posición de la derecha
       24FA                           %CLRST          5
       24FF 2454                      bcc             gp_shi_l        ;si no había barco atacado en la derecha, se busca la izquierda
       2501                           %WRITEPOS       #SUNK_SHI       
       251B 6C0B                      inc             11,x
       251D                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       2537 6A0B                      dec             11,x            ;se recupera el dato
       2539                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       254F                           %RESTORE
       2554 39                        rts
                      
       2555           gp_shi_l        EQU             *
       2555 E60B                      ldab            11,x
       2557 C101                      cmpb            #MIN_C
       2559 276D                      beq             gp_shi_u        ;si está en el borde izquierdo, se continúa
       255B E60C                      ldab            12,x
       255D 37                        pshb                            ;se envía la letra (fila)
       255E E60B                      ldab            11,x
       2560 5A                        decb                            ;se retrocede a la izquierda
       2561 37                        pshb                            ;se envía el número (columna)
       2562 1AEE0E                    ldy             14,x
       2565 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       2567 E608                      ldab            8,x
       2569 37                        pshb                            ;se envía el jugador atacado
       256A BD27C6                    jsr             play_src        ;se busca la posición de la izquierda
       256D                           %CLRST          5
       2572 2454                      bcc             gp_shi_u        ;si no había barco atacado a la izquierda, se busca arriba
       2574                           %WRITEPOS       #SUNK_SHI       
       258E 6A0B                      dec             11,x
       2590                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       25AA 6C0B                      inc             11,x            ;se recupera el dato
       25AC                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       25C2                           %RESTORE
       25C7 39                        rts
                      
       25C8           gp_shi_u        EQU             *
       25C8 E60C                      ldab            12,x
       25CA C10A                      cmpb            #MIN_F
       25CC 276D                      beq             gp_shi_d        ;si está en el borde superior, se continúa
       25CE E60C                      ldab            12,x
       25D0 5A                        decb                            ;se va a la posición de arriba
       25D1 37                        pshb                            ;se envía la letra (fila)
       25D2 E60B                      ldab            11,x
       25D4 37                        pshb                            ;se envía el num (columna)
       25D5 1AEE0E                    ldy             14,x
       25D8 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       25DA E608                      ldab            8,x
       25DC 37                        pshb                            ;se envía el jugador atacado
       25DD BD27C6                    jsr             play_src        ;se busca la posición de arriba
       25E0                           %CLRST          5
       25E5 2454                      bcc             gp_shi_d        ;si no había barco atacado arriba, se busca abajo
       25E7                           %WRITEPOS       #SUNK_SHI       
       2601 6A0C                      dec             12,x
       2603                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       261D 6C0C                      inc             12,x            ;se recupera el dato
       261F                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       2635                           %RESTORE
       263A 39                        rts
                      
       263B           gp_shi_d        EQU             *
       263B E60C                      ldab            12,x
       263D C10F                      cmpb            #MAX_F
       263F 276D                      beq             gp_shi_h        ;si está en el borde inferior, se continúa
       2641 E60C                      ldab            12,x
       2643 5C                        incb                            ;se va a la posición de abajo
       2644 37                        pshb                            ;se envía la letra (fila)
       2645 E60B                      ldab            11,x
       2647 37                        pshb                            ;se envía el num (columna)
       2648 1AEE0E                    ldy             14,x
       264B 183C                      pshy                            ;se envía el inicio del arreglo de barcos atacados
       264D E608                      ldab            8,x
       264F 37                        pshb                            ;se envía el jugador atacado
       2650 BD27C6                    jsr             play_src        ;se busca la posición de abajo
       2653                           %CLRST          5
       2658 2454                      bcc             gp_shi_h        ;si no había barco atacado abajo, el carguero está tocado
       265A                           %WRITEPOS       #SUNK_SHI       
       2674 6C0C                      inc             12,x
       2676                           %WRITEPOS       #SUNK_SHI       ;se "hunde" a TODO el carguero
       2690 6A0C                      dec             12,x            ;se recupera el dato
       2692                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       26A8                           %RESTORE
       26AD 39                        rts
                      
       26AE           gp_shi_h        EQU             *
       26AE                           %WRITEPOS       #HIT            ;se marca al carguero como tocado
       26C8                           %PLAY_WRT                       ;se agrega la posición al array de barcos atacados
       26DE                           %RESTORE
       26E3 39                        rts
                      
                      *==========================================================================
                      * subrutina def_ships
                      * Función: Determina los valores de cargueros y submarinos a utilizar.
                      * Recibe: Nada
                      * Devuelve: Cantidad de cargueros y submarinos en memorias correspondientes.
                      * Requiere:
                      *==========================================================================
       26E4           def_ships       EQU             *
       26E4                           %BACKUP
                      
       26E9           wait_nsu        EQU             *
       26E9 CE42CE                    ldx             #subm
       26EC BD2914                    jsr             show
                      
       26EF BD2928                    jsr             read_kb
       26F2 4D                        tsta
       26F3 2BF4                      bmi             wait_nsu
       26F5 36                        psha
       26F6                           %EXPECT         B_fire
       2700 32                        pula
                      
       2701 CE4265                    ldx             #kb_value
       2704 16                        tab
       2705 3A                        abx
       2706 A600                      ldaa            0,x             ;se obtiene el valor ingresado ya codificado
                      
       2708 8105                      cmpa            #SUB            ;se controla que no sea menor al máximo de submarinos
       270A 22DD                      bhi             wait_nsu
       270C B7425B                    staa            cant_sub        ;se guarda la cantidad de submarinos
                      
       270F           wait_nsh        EQU             *
       270F CE42D3                    ldx             #carg
       2712 BD2914                    jsr             show
                      
       2715 BD2928                    jsr             read_kb
       2718 4D                        tsta
       2719 2BF4                      bmi             wait_nsh
       271B 36                        psha
       271C                           %EXPECT         B_fire
       2726 32                        pula
                      
       2727 CE4265                    ldx             #kb_value
       272A 16                        tab
       272B 3A                        abx
       272C A600                      ldaa            0,x             ;se obtiene el valor ingresado ya codificado
                      
       272E 8104                      cmpa            #SHI            ;se controla que no sea menor al máximo de submarinos
       2730 22DD                      bhi             wait_nsh
       2732 B7425C                    staa            cant_shi        ;se guarda la cantidad de submarinos
                      
       2735                           %RESTORE
       273A 39                        rts
                      
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
                      
       273B           mirror          EQU             *       
                              
       273B                           %BACKUP
       2740 30                        tsx                             ;creo frame pointer
       2741 A60B                      ldaa            11,x            ;cargo número de jugador
       2743 E608                      ldab            8,x             ;cargo longitud de la copia (del array a copiar)
       2745 3D                        mul                             ;obtengo offset de jugador
       2746 E30C                      addd            12,x            ;sumo offset de jugador
       2748 ED0C                      std             12,x            ;guardo la dirección con offset del array fuente
                      
       274A A608                      ldaa            8,x             ;cargo longitud de copia en contador
       274C 2710                      beq             mi_end          ;si es cero, vuelvo desde subrutina
       274E 1AEE09                    ldy             9,x             ;cargo índice a destino de la copia.
       2751 EE0C                      ldx             12,x            ;cargo índice a array origen de copia.
                      
       2753           mi_loop         EQU             *
                      
       2753 E600                      ldab            0,x             ;cargo byte a copiar
       2755 18E700                    stab            0,y             ;copio byte en destino
       2758 08                        inx                             ;siguiente byte
       2759 1808                      iny                             ;siguiente byte
       275B 4A                        deca                            ;decremento contador
       275C 26F5                      bne             mi_loop
                      
       275E           mi_end          EQU             *
                      
       275E                           %RESTORE
                              
       2763 39                        rts     
                                      
                                                      
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
                      
       2764           fetchpos        EQU             *
                      
       2764                           %BACKUP
       2769 30                        tsx                             ;creo frame pointer
       276A A608                      ldaa            8,x             ;cargo numero de jugador
       276C C624                      ldab            #ARRAY_L        ;cargo longitud del arreglo (MAX 255 bytes)
       276E 3D                        mul                             ;obtengo offset de jugador
       276F E309                      addd            9,x             ;sumo offset de jugador a top of array
       2771 ED09                      std             9,x             ;obtengo direccion al arreglo del jugador
                      
       2773 A60C                      ldaa            12,x            ;cargo coordenada letra (FILA)
       2775 C60A                      ldab            #NORM           ;cargo valor de normalización de filas
       2777 10                        sba                             ;obtengo numero de fila
       2778 C606                      ldab            #COL            ;cargo longitud de fila
       277A 3D                        mul                             ;obtengo offset de fila
       277B E309                      addd            9,x             ;sumo offset de fila a la dirección del arreglo del jugador
       277D ED09                      std             9,x             ;obtengo dirección a la fila que se desea
                      
       277F E60B                      ldab            11,x            ;cargo coordenada (offset) de columna
       2781 5A                        decb                            ;normalizo offset de columna
       2782 4F                        clra                            ;borro AccA para tener el valor de AccB en AccD
       2783 E309                      addd            9,x             ;sumo offset de columna a la dirección temporal
       2785 ED09                      std             9,x             ;guardo en stack la dirección del byte a consultar
                      
       2787 1AEE09                    ldy             9,x             ;cargo en IY la dirección del byte que se desea obtener
       278A 18A600                    ldaa            0,y             ;cargo el byte a devolver en AccA
       278D A70C                      staa            12,x            ;guardo el byte a devolver en stack
       278F                           %RESTORE
                                      
       2794 39                        rts
                                      
                                      
                      
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
                      
       2795           writepos        EQU             *
                      
       2795                           %BACKUP
       279A 30                        tsx                             ;creo frame pointer
       279B A608                      ldaa            8,x             ;cargo numero de jugador
       279D C624                      ldab            #ARRAY_L        ;cargo longitud del arreglo (MAX 255 bytes)
       279F 3D                        mul                             ;obtengo offset de jugador
       27A0 E309                      addd            9,x             ;sumo offset de jugador a top of array
       27A2 ED09                      std             9,x             ;obtengo direccion al arreglo del jugador
                      
       27A4 A60C                      ldaa            12,x            ;cargo coordenada letra (FILA)
       27A6 C60A                      ldab            #NORM           ;cargo valor de normalización de filas
       27A8 10                        sba                             ;obtengo numero de fila
       27A9 C606                      ldab            #COL            ;cargo longitud de fila
       27AB 3D                        mul                             ;obtengo offset de fila
       27AC E309                      addd            9,x             ;sumo offset de fila a la dirección del arreglo del jugador
       27AE ED09                      std             9,x             ;obtengo dirección a la fila que se desea
                      
       27B0 E60B                      ldab            11,x            ;cargo coordenada (offset) de columna
       27B2 5A                        decb                            ;normalizo offset de columna
       27B3 4F                        clra                            ;borro AccA para tener el valor de AccB en AccD
       27B4 E309                      addd            9,x             ;sumo offset de columna a la dirección temporal
       27B6 ED09                      std             9,x             ;guardo en stack la dirección del byte a consultar
                      
       27B8 1AEE09                    ldy             9,x             ;cargo en IY la dirección donde se desea escribir
       27BB A60D                      ldaa            13,x            ;cargo en AccA el byte que se quiere escribir en el arreglo
       27BD 18A700                    staa            0,y             ;escribo el byte sobre el arreglo
       27C0                           %RESTORE
                                      
       27C5 39                        rts
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
                      
       27C6           play_src        EQU             *               
                                              
       27C6                           %BACKUP
       27CB 30                        tsx                             ;creo frame pointer
       27CC A608                      ldaa            8,x             ;cargo número de array (jugador)
       27CE C61B                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       27D0 3D                        mul                             ;obtengo offset de jugador
       27D1 E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       27D3 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
                      
       27D5           ps_loop         EQU             *
                      
       27D5 18A600                    ldaa            0,y             ;cargo byte1 a comparar
       27D8 2718                      beq             ps_ntfnd        ;si encontré el terminador, el valor requerido no está en el array
                      
       27DA A10C                      cmpa            12,x            ;comparo con el byte1 a buscar
       27DC 2706                      beq             ps_next         ;si coinciden, busco el byte 2
       27DE 1808                      iny                             ;me posiciono sobre siguiente byte1 a comparar
       27E0 1808                      iny
       27E2 20F1                      bra             ps_loop         ;vuelvo a comparar
                      
       27E4           ps_next         EQU             *
                      
       27E4 18E601                    ldab            1,y             ;cargo byte a continuación del anterior
       27E7 1808                      iny                             ;me posiciono sobre siguiente byte1 a comparar
       27E9 1808                      iny
       27EB E10B                      cmpb            11,x            ;comparo con byte2
       27ED 26E6                      bne             ps_loop         ;si no son iguales, sigo buscando
       27EF 0D                        sec                             ;set del carry para informar que se encontró el valor
       27F0 2001                      bra             ps_end
                                              
       27F2           ps_ntfnd        EQU             *
                      
       27F2 0C                        clc                             ;carry en 0 para informar que no se encontró el valor
                      
       27F3           ps_end          EQU             *
                                      
       27F3                           %RESTORE
                      
       27F8 39                        rts
                      
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
                      
       27F9           play_cnt        EQU             *
                      
       27F9                           %BACKUP
       27FE 30                        tsx                             ;creo frame pointer
       27FF A608                      ldaa            8,x             ;cargo número de array (jugador)
       2801 C61B                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       2803 3D                        mul                             ;obtengo offset de jugador
       2804 E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       2806 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
       2808 4F                        clra                            ;inicializo contador
                                              
       2809           pc_loop         EQU             *               
                                              
       2809 18E600                    ldab            0,y             ;cargo byte a comparar
       280C 2707                      beq             pc_end          ;si es terminador, terminé de buscar
       280E 1808                      iny                             ;si no es terminador, me posiciono sobre siguiente dato de 
       2810 1808                      iny                             ;2 bytes
       2812 4C                        inca                            ;incremento contador
       2813 20F4                      bra             pc_loop         ;vuelvo a comparar
                                              
       2815           pc_end          EQU             *       
                                              
       2815 A70A                      staa            10,x            ;devuelvo valor por stack
       2817                           %RESTORE
                                      
       281C 39                        rts
                      
                      
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
                      
       281D           play_wrt        EQU             *
                      
       281D                           %BACKUP
       2822 30                        tsx                             ;creo frame pointer
       2823 A608                      ldaa            8,x             ;cargo número de array (jugador)
       2825 C61B                      ldab            #HITSIZE        ;cargo tamaño de array de 1 jugador
       2827 3D                        mul                             ;obtengo offset de jugador
       2828 E309                      addd            9,x             ;sumo el offset de jugador al puntero al array
       282A 188F                      xgdy                            ;obtengo puntero al array del jugador deseado
                      
                      
       282C           pw_loop         EQU             *
                      
       282C 18A600                    ldaa            0,y             ;cargo byte del array
       282F 2704                      beq             pw_end          ;si es cero, entonces escribo los datos
       2831 1808                      iny                             ;si no es cero, me posiciono sobre la siguiente posición
       2833 20F7                      bra             pw_loop         ;vuelvo a buscar ceros
                      
       2835           pw_end          EQU             *
                      
       2835 A60C                      ldaa            12,x            ;cargo primer byte a escribir
       2837 18A700                    staa            0,y             ;guardo primer byte en memoria
       283A 1808                      iny                             ;me posiciono en el siguiente byte
       283C A60B                      ldaa            11,x            ;cargo segundo byte a escribir
       283E 18A700                    staa            0,y             ;guardo segundo byte en memoria
       2841                           %RESTORE
                      
       2846 39                        rts
                      
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
       2847           clrzone         EQU     *
                      
       2847                           %BACKUP                         ;backup de registros en stack
                      
       284C 30                        tsx
       284D A608                      ldaa    8,x                     ;cargo contador con longitud de zona a borrar
       284F EE09                      ldx     9,x                     ;cargo puntero zona de memoria a borrar
       2851 D600                      ldab    $00                     ;cargo el cero
       2853           clr_loop        EQU     *
       2853 E700                      stab    0,x                     ;guardo caracter en posicion del array
       2855 08                        inx                             ;siguiente posición     
       2856 4A                        deca                            ;decremento contador
       2857 26FA                      bne     clr_loop                ;si no es cero, escribo en la posición siguiente.
                      
       2859                           %RESTORE                        :restauro backup en registros
                      
       285E 39                        rts
                      
                      *---------------------------------------
                      * WhoPlays: subrutina para ver que 
                      *           jugador juega
                      *---------------------------------------
                      
       285F           whoplays        EQU             *
                      
       285F BD2928    wait_ply        jsr             read_kb
       2862 4D                        tsta
       2863 2BFA                      bmi             wait_ply
       2865 8103                      cmpa            #B_P1
       2867 2708                      beq             play_p1
       2869 8107                      cmpa            #B_P2
       286B 26F2                      bne             wait_ply
       286D 8600                      ldaa            #P1
       286F 2002                      bra             ply_end
                      
       2871 8601      play_p1         ldaa            #P2
       2873 39        ply_end         rts             
                      
                      *---------------------------------------
                      * Input: subrutina que devuelve
                      *        entrada para la batalla naval
                      * AccA: Fila de la posicion jugada
                      * AccB: Columna de la posicion jugada
                      *---------------------------------------
                      
       2874           input           EQU             *
                      
                      * back up de registros
       2874 3C                        pshx
       2875 183C                      pshy
                      
                      * creo variables locales
       2877 36                        psha
       2878 36                        psha
       2879 36                        psha
                      
                      * inicializo framepointer
       287A 1830                      tsy
                      
                      * prendo el led
       287C                           %LED_INV
                      
                      * inicializo el contador
       2884 8603      inp_rest        ldaa            #INPUTS
       2886 18A700                    staa            CONT,Y
                                      
                      * limpio el display
       2889 CE4260                    ldx             #string
       288C 8604                      ldaa            #4
       288E C620                      ldab            #ESP
       2890 BD29F2                    jsr             clrarray
                      
       2893 BD29E1                    jsr             clr_dis
                      
       2896 86FF                      ldaa            #$FF
       2898 B7425F                    staa            buffer
                      
                      * lectura de teclado
       289B BD2928    wait_kb         jsr             read_kb
       289E 4D                        tsta
       289F 2BFA                      bmi             wait_kb
       28A1 B1425F                    cmpa            buffer
       28A4 27F5                      beq             wait_kb
       28A6 B7425F                    staa            buffer
                                      
       28A9 810F                      cmpa            #B_reset
       28AB 27D7                      beq             inp_rest
       28AD 8103                      cmpa            #B_P1
       28AF 27D3                      beq             inp_rest
       28B1 8107                      cmpa            #B_P2
       28B3 27CF                      beq             inp_rest
                      
       28B5 186A00                    dec             CONT,Y
       28B8 272B                      beq             chk_fire
       28BA 810B                      cmpa            #B_fire
       28BC 27C6                      beq             inp_rest
       28BE 36                        psha
       28BF CE4265                    ldx             #kb_value
       28C2 16                        tab
       28C3 3A                        abx
       28C4 A600                      ldaa            0,X             
       28C6 183C                      pshy
       28C8 18E600                    ldab            CONT,Y
       28CB 183A                      aby
       28CD 18A700                    staa            0,Y
       28D0 1838                      puly
                      
       28D2 33                        pulb
       28D3 BD29D1                    jsr             to_ascii
       28D6 17                        tba
                                      
       28D7 CE4260                    ldx             #string
       28DA BD29C1                    jsr             sh_value
                      
       28DD CE4260                    ldx             #string
       28E0 BD2914                    jsr             show
                      
       28E3 20B6                      bra             wait_kb
                      
       28E5 810B      chk_fire        cmpa            #B_fire
       28E7 269B                      bne             inp_rest
       28E9 18A602                    ldaa            FILA_L,Y
       28EC 810A                      cmpa            #MIN_FIL
       28EE 2594                      blo             inp_rest
       28F0 810F                      cmpa            #MAX_FIL
       28F2 2290                      bhi             inp_rest
       28F4 18A601                    ldaa            COL_L,Y
       28F7 8101                      cmpa            #MIN_COL
       28F9 2589                      blo             inp_rest
       28FB 8106                      cmpa            #MAX_COL
       28FD 2285                      bhi             inp_rest
                      
                      * apago el led
       28FF                           %LED_INV
                      
                      * libero variables locales
       2907 32                        pula
       2908 32                        pula
       2909 32                        pula
                      
       290A 18A602    return          ldaa            FILA_L,Y
       290D 18E601                    ldab            COL_L,Y         
                      
                      * restore de registros
       2910 1838                      puly
       2912 38                        pulx
                      
       2913 39                        rts
                      
                      *------------------------------------
                      * Show: Por IX direccion de string de
                      *       4 caracteres para imprimir
                      *------------------------------------
                      
       2914           show            EQU             *
                      
                      * back up de registros
       2914 3C                        pshx
       2915 37                        pshb
       2916 36                        psha
                      
                      * inicializo contador de posiciones
       2917 C604                      ldab            #LENGTH
                                      
                      * loop para enviar a cada posicion
       2919 5A        sh_loop         decb
       291A 2B08                      bmi             sh_end          ; si no pase por todas las posiciones
       291C A600                      ldaa            0,X             ; tomo del string el caracter
       291E BD29DA                    jsr             outchar         ; y lo imprimo en la posicion actual
       2921 08                        inx
       2922 20F5                      bra             sh_loop
                      
                      * restore de registros                          
       2924 32        sh_end          pula
       2925 33                        pulb
       2926 38                        pulx
       2927 39                        rts
                      
                      *----------------------------------------------
                      * read_kb: AccA: tecla presionada
                      *          o -1
                      *----------------------------------------------
                      
       2928           read_kb         EQU             *
                      
                      * back up de registros
       2928 3C                        pshx
       2929 37                        pshb
                      
                      * creo espacio para variables locales
       292A                           %MALLOC         1               ; variable mascara de filas
       2931                           %MALLOC         1               ; variable mascara de columnas
       2938                           %MALLOC         1               ; variable contador de columnas
       293F                           %MALLOC         1               ; variable contador de tecla
                      
                      * inicializacion de variables
       2946 30                        tsx                             ; framepointer para acceder a locales
       2947 C604                      ldab            #LENGTH         ; contador de fila
       2949 860E                      ldaa            #FIL0           ; mascara para la fila cero
       294B A703                      staa            MASK_FIL,X      
       294D 8610                      ldaa            #COL0           ; mascara para la columna cero
       294F A702                      staa            MASK_COL,X
       2951 4F                        clra                            ; contador de columna
       2952 A701                      staa            COL_CONT,X
       2954 A700                      staa            KB_PRESS,X      ; variable con numero de tecla
                      
                      * loop para barrer y mirar teclado matricial
       2956 86FF      kb_loop         ldaa            #NONE
       2958 5A                        decb                            
       2959 2B2C                      bmi             kb_end          ; me fijo si recorrio todas las filas
       295B A603                      ldaa            MASK_FIL,X      ; enmascaro la siguiente fila
       295D B71003                    staa            ROWS
       2960 0D                        sec                             ; carry para rotacion de byte
       2961 49                        rola                            
       2962 A703                      staa            MASK_FIL,X      ; cambio variable mascara para la proxima fila
                      
       2964 8604                      ldaa            #LENGTH
       2966 A701                      staa            COL_CONT,X
       2968 8610                      ldaa            #COL0           ; inicializo variable para barrer
       296A A702                      staa            MASK_COL,X      ; la columna
                      
       296C 6A01      kb_col          dec             COL_CONT,X      ; me fijo si barrio todas las columnas
       296E 2BE6                      bmi             kb_loop
       2970 B61005                    ldaa            COLS            ; leo el puerto de columnas
       2973 A402                      anda            MASK_COL,X      ; enmascaro columna
       2975 2707                      beq             kb_hit          ; si es nulo apretaron un boton
       2977 48                        lsla                            ; cambio mascara para proxima columna
       2978 A702                      staa            MASK_COL,X
       297A 6C00                      inc             KB_PRESS,X      ; incremento numero de tecla
       297C 20EE                      bra             kb_col
                      
       297E A600      kb_hit          ldaa            KB_PRESS,X      ; apretaron una tecla, lo comparo
       2980 B1425E                    cmpa            kb_buff         ; con la que fue apretada antes
       2983 2602                      bne             kb_end          ; aviso por led
                      
       2985 86FF      kb_none         ldaa            #NONE           ; en caso de que no apreten nada
                      
       2987 B7425E    kb_end          staa            kb_buff         ; actualizo variable
                                              
                      * elimino variables locales
       298A                           %RELMEM         1
       2991                           %RELMEM         1
       2998                           %RELMEM         1
       299F                           %RELMEM         1
                      
                      * restore de registros
       29A6 33                        pulb
       29A7 38                        pulx
                                      
       29A8 39                        rts
                      
                      *----------------------------------------------
                      * init_kb: subrutina para inicializar teclado
                      *----------------------------------------------
                      
       29A9           init_kb         EQU             *
                      
                      * back up de registros
       29A9 36                        psha
                      
                      * inicializacion de teclado             
       29AA 86FF                      ldaa            #NONE
       29AC B7425E                    staa            kb_buff
                      
                      * restore de registros
       29AF 32                        pula
       29B0 39                        rts
                      
                      *--------------------------------
                      * Inicializa display (y teclado)
                      *--------------------------------
                      
       29B1           initdis         EQU             *
                              
       29B1 36                        psha
                      
       29B2 860F                      ldaa            #$0F       ;set for input/output
       29B4 B71007                    staa            DDRC       ;C output
       29B7 B61002                    ldaa            PIOC       ;get current value
       29BA 84FE                      anda            #$FE       ;reset bit 0 to 0
       29BC B71002                    staa            PIOC       ;send it out
                              
       29BF 32                        pula
       29C0 39                        rts
                      
                      *--------------------------------------------------------------
                      * sh_value: toma un string, desplaza los caracteres a izquierda
                      *           y agrega un nuevo caracter
                      *           IX: direccion del string
                      *           AccA: nuevo valor
                      *--------------------------------------------------------------
                      
       29C1           sh_value        EQU             *
                      
                      * back up de registros
       29C1 3C                        pshx
       29C2 37                        pshb
                      
                      * loop para desplazar caracteres
       29C3 E601      v_loop          ldab            1,X             ;tomo elemento siguiente en string
       29C5 2705                      beq             v_end
       29C7 E700                      stab            0,X             ;y lo guardo en posicion actual
       29C9 08                        inx
       29CA 20F7                      bra             v_loop          
                      
       29CC A700      v_end           staa            0,X             ;ingresa nuevo caracter en cadena desplazada
                      
                      * restore de registros
       29CE 33                        pulb
       29CF 38                        pulx
       29D0 39                        rts
                      
                      *----------------------------------------------
                      * toAscii: toma numero de boton del teclado
                      *          y devuelve su correspondiente en
                      *          ascii
                      *          toma  AccB: valor del boton
                      *          devuelve AccB: ascii
                      *----------------------------------------------
                      
       29D1           to_ascii        EQU             *
                      
                      * back up de registros
       29D1 3C                        pshx
                      
                      * inicializacion de variables
       29D2 CE4275                    ldx             #kb_ascii       ; apunto a string con caracteres asignados por tecla
       29D5 3A                        abx                             ; me muevo en string con indice y recupero segun tecla presionada
       29D6 E600                      ldab            0,X             
                      
                      * restore de registros
       29D8 38                        pulx
       29D9 39                        rts
                      
                      *--------------------------------
                      *Outchar: ACCA: Carácter a enviar
                      *           ACCB: Posición (0-3)
                      *--------------------------------
       29DA           outchar         EQU             *
                              
       29DA F71005                    stab            PORTCL          ;set the position 
       29DD B71004                    staa            PORTB           ;set the character
                              
       29E0 39                        rts
                      
                      *-------------------------
                      * clr_dis: Limpia display
                      *-------------------------
                      
       29E1           clr_dis         EQU             *
                      
                      * hago back up de registros
       29E1 37                        pshb
       29E2 36                        psha
                      
                      * inicializo el contador de posiciones
       29E3 C604                      ldab            #LENGTH
                      
                      * loop para limpiar cada posicion
       29E5 5A        clrloop         decb
       29E6 2B07                      bmi             clr_end         ; si no pase por todas las posiciones
       29E8 8620                      ldaa            #ESP            ; cargo en esta posicion espacio blanco
       29EA BD29DA                    jsr             outchar         ; y lo envio
       29ED 20F6                      bra             clrloop         
                      
                      * restore de registros
       29EF 32        clr_end         pula
       29F0 33                        pulb
       29F1 39                        rts
                      
                      *-------------------------------------------------------
                      * clrarray: subrutina para limpiar un array dado
                      * parametros: IX direccion del array
                      *             AccA cantidad de elementos del array
                      *             AccB caracter a poner
                      *-------------------------------------------------------
                      
       29F2           clrarray        EQU             *
                                      
       29F2 4A        cra_loop        deca                            ;contador con cantidad de elementos a limpiar
       29F3 2705                      beq             cra_end
       29F5 E700                      stab            0,X             ;reemplazar elemento por caracter a poner
       29F7 08                        inx
       29F8 20F8                      bra             cra_loop
                                      
       29FA E700      cra_end         stab            0,X
       29FC 39                        rts
                      
                      *=============================================
                      *Subrutina winner: Recibe por stack un 1 si
                      *gano el player 1 y un 0 si gano el player 2.
                      *=============================================
                      
                                      
       29FD           winner          EQU             *
       29FD                           %BACKUP
       2A02 30                        tsx
       2A03 A608                      ldaa            8,x     ;recupero parametro enviado por stack
       2A05 2708                      beq             win2
                      
       2A07           win1            EQU             *
       2A07 CE42BA                    ldx             #string1
       2A0A BD2914                    jsr             show
       2A0D 2006                      bra             end
                      
       2A0F           win2            EQU             *
       2A0F CE42BF                    ldx             #string2
       2A12 BD2914                    jsr             show
                              
       2A15           end             EQU             *
       2A15                           %RESTORE
       2A1A 39                        rts
                      
                      *=============================================
                      *Subrutina turn: Recibe por stack un 1 si
                      *juega el player 1 y un 0 si juega el player 2.
                      *=============================================
                      
       2A1B           turn            EQU             *
       2A1B                           %BACKUP
       2A20 30                        tsx
       2A21 A608                      ldaa            8,x
       2A23 2708                      beq             p2_t
                      
       2A25           p1_t            EQU             *
       2A25 CE42C4                    ldx             #play1
       2A28 BD2914                    jsr             show
       2A2B 20E8                      bra             end
                              
       2A2D           p2_t            EQU             *
       2A2D CE42C9                    ldx             #play2
       2A30 BD2914                    jsr             show
                      
       2A33           end_t           EQU             *
       2A33                           %RESTORE        
       2A38 39                        rts
                      
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
                      
       42C4 2D50322D  play1           FCC     '-P2-'
       42C8 00                        FCB     0
       42C9 2D50312D  play2           FCC     '-P1-'
       42CD 00                        FCB     0
                      
       42CE 5355424D  subm            FCC     'SUBM'
       42D2 00                        FCB     0
       42D3 43415247  carg            FCC     'CARG'
       42D7 00                        FCB     0
       42D8 504C4159  play            FCC     'PLAY'
       42DC 00                        FCB     0
                      
       42DD           arry_pr         RMB             N_PR*SA_SIZE
       43A5           arry_aux        RMB             N_AUX*SA_SIZE
                      
       43AD 53554231  msg_sub         FCC             'SUB1'
       43B1 00                        FCB             0
       43B2 53554232                  FCC             'SUB2'
       43B6 00                        FCB             0
       43B7 53554233                  FCC             'SUB3'
       43BB 00                        FCB             0
       43BC 53554234                  FCC             'SUB4'
       43C0 00                        FCB             0
       43C1 53554235                  FCC             'SUB5'
       43C5 00                        FCB             0
                      
       43C6 43415231  msg_shi         FCC             'CAR1'
       43CA 00                        FCB             0       
       43CB 43415232                  FCC             'CAR2'
       43CF 00                        FCB             0       
       43D0 43415233                  FCC             'CAR3'
       43D4 00                        FCB             0
       43D5 43415234                  FCC             'CAR4'
       43D9 00                        FCB             0
                      
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
     ytes:   2841         # main_bn #
     RC:     181C         ###########
 