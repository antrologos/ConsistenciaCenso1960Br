Amostra de 1,27%: origem dos dados e a necessidade de consist�ncia
------------------------------------------------------------------

Como dito anteriormente, o pr�prio IBGE n�o disponibiliza ou comercializa qualquer vers�o do Censo Demogr�fico de 1960. Aquela possu�da pelo Centro de Estudos da Metr�pole da Amostra de 1,27% � fruto de uma doa��o do acervo pessoal de dados dos pesquisadores Carlos Ant�nio Costa Ribeiro Filho e Adalberto Cardoso, ambos ligados ao Instituto de Estudos Sociais e Pol�ticos (IESP) da Universidade do Estado do Rio de Janeiro (UERJ), que por sua vez obtiveram os dados tamb�m como doa��o, feita pelos pesquisadores americano Edward Telles (UC-Santa Barbara) e Charles Wood (Univ. Florida).

Ainda que todos esses pesquisadores envolvidos sejam de compet�ncia extrema e publicamente atestada, o fato de que tais dados foram veiculados por meio de transfer�ncias interpessoais e n�o oficiais j� atentaria para a necessidade da realiza��o de chegagens e confer�ncias, contra relat�rios e tabula��es oficiais -- na expectativa de averiguar eventuais discrep�ncias e altera��es. Soma-se a isso o fato de que os m�todos de grava��o e meios de estocagem dos dados, � �poca de sua extra��o original (1965), eram muito menos confi�veis, podendo gerar erros e corrup��o de informa��o. Por essas duas raz�es foi preciso proceder uma cuidadosa an�lise e consist�ncia dos dados da Amostra de 1960, anterior a qualquer uso ou harmoniza��o.

### Consist�ncia, transpar�ncia e replicabilidade: RMarkdown e Github

N�o foi poss�vel, contudo, lan�ar m�o de procedimentos exclusivamente automatizados em todas as etapas. Diversos momentos exigiram cuidadosa leitura manual de linhas do arquivo de dados em formato fixo, para que fosse poss�vel identificar a natureza dos erros e procedimentos cab�veis para eventuais corre��es. Descrevemos detalhadamente neste documento todos passos, apresentando, no pr�prio corpo do texto as linhas de c�digo utilizadas na linguagem R. O prop�sito fundamental � maximizar a transpar�ncia e a replicabilidade, uma vez que ser�o aplicadas modifica��es e corre��es sobre dados oficiais, cuja import�ncia n�o � apenas hist�rica. Apesar das tecnicalidades envolvidas nessa estrat�gia de apresenta��o, zelaremos pela manuten��o da simplicidade da exposi��o.

Este presente documento foi escrito na liguagem [RMarkdown](https://rmarkdown.rstudio.com/), um tipo simplificado de linguagem de marca��o, semelhante ao HTML, XML ou LaTeX, mas completamente integrada � plataforma R, permitindo a inser��o de blocos de c�digos intercalados entre partes do corpo do texto. Isto traz a vantagem de centralizar todos os procedimentos de an�lise num �nico documentos que cont�m, ao mesmo tempo, scripts que carregam dados, executam transforma��es e geram os finais. Todos os arquivos adicionais utilizados aqui (sempre por meio de chamadas em linhas de c�digo) encontram-se dispon�veis on-line num reposit�rio do GitHub: \[<https://github.com/antrologos/ConsistenciaCenso1960Br>\].

Iniciando os passos da an�lise, carregaremos aqui os pacotes necess�rios para todas as an�lises que ser�o realizadas posteriormente:

``` r
library(tidyverse)
library(readxl)
library(descr)
library(data.table)
```

Carregaremos tamb�m um conjunto de fun��es criadas especificamente para auxiliar nas tarefas avalia��o e implementa��o dos procedimentos de consist�ncia realizados aqui:

``` r
source("https://raw.githubusercontent.com/antrologos/ConsistenciaCenso1960Br/master/Code/Utils.R")
```

Arquivos originais e formato dos dados
--------------------------------------

A pasta dos arquivos doados do Censo de 1960 consistia de quatro itens:

1.  [HHOLDA.txt](https://github.com/antrologos/ConsistenciaCenso1960Br/blob/master/Original%20Files/HHOLDA.txt)
2.  [dicionario\_60\_last.doc](https://github.com/antrologos/ConsistenciaCenso1960Br/blob/master/Original%20Files/dicionario_60_last.doc)
3.  [CENSO1960 - MONTA ARQUIVO.SPS](https://github.com/antrologos/ConsistenciaCenso1960Br/blob/master/Original%20Files/CENSO1960%20-%20MONTA%20ARQUIVO.SPS)
4.  [GeraBookSAS CD60\_1%.txt](https://github.com/antrologos/ConsistenciaCenso1960Br/blob/master/Original%20Files/GeraBookSAS%20CD60_1%25.txt)

O primeiro deles, � o pr�prio conjunto dos dados. Trata-se, como adiantado anteriormente, de um arquivo de formato fixo, em que as posi��es absolutados dos caracteres nas colunas do texto, sem qualquer separador ou marca adicional, determinam o conte�do das vari�veis. O conte�do das dez primeiras linhas � o seguinte:

``` r
readLines("Original Files/HHOLDA.txt", n = 10) %>% writeLines()
```

    ## 0000110100014051111478950580206002                    \0000001
    ## 0000110100014051212716057149191415200400000076-       \0000001
    ## 0000110100014051312311657159191508210000000035-       \0000001
    ## 0000110100014052111482940580206001                    \0000002
    ## 000011010001405221271245704919041732000000008318728145\0000002
    ## 0000110100014053111478949579208004                    \0000003
    ## 000011010001405321171566567019041821073015150387326146\0000003
    ## 00001101000140533129103651392000-                     \0000003
    ## 0000110100014053312814965159190415200730151534-       \0000003
    ## 0000110100014053312912365049190407322000000035-       \0000003

Como podemos observar, aparentemente as linhas do arquivo de dados possuem o mesmo comprimento: 62 caracteres. A raz�o para isso � a presen�a de uma estrutura de d�gitos bastante padronizada, que se inicia no caractere 55, iniciando por uma barra invertida -- e.g.: `\0000001`. Trata-se de uma numera��o das fam�lias que habitam os domic�lios. Assim, todas as linhas marcadas com o mesmo sulfixo desse tipo pertencem � mesma fam�lia. As linhas n�o representam apenas os indiv�duos entrevistados, contudo. O primeiro registro de uma fam�lia representa traz consigo as caracter�sticas do domic�lio onde residem. Deste modo, dizemos que os dados dados acima apresentam uma estrutura hier�rquica, pelo fato de que suas linhas cont�m tanto a unidade prim�ria dos microdados (os indiv�duos) como tamb�m as estruturas de agrega��o nas quais est�o aninhados (fam�lias).

Al�m do arquivo de dados, a pasta continha tamb�m a) o dicion�rio de c�digos informando o layout e as coordenadas para a interpreta��o do arquivo de texto em formato fixo; b) uma sintaxe de abertura em SAS; c) uma sintaxe de abertura em SPSS. An�lises preliminares, contudo, indicaram que o dicion�rio de c�digos se dirigia a um layout muito diferente daquele utilizado nas duas sintaxes de abertura, tanto em SAS, como em SPSS. Divergiam, assim, nas coordenadas dos caracteres para in�cio de fim da leitura de praticamente todas as vari�veis. O documento do dicion�rio de c�digos, no item "Origem dos dados" informa que se trata da:

> AMOSTRA DE 25% DO CENSO DEMOGR�FICO DE 1960, EXTRA�DO DE ARQUIVO DE DADOS GRAVADO EM FITA MAGN�TICA, O QUAL INCLUI DADOS PARA OS SEGUINTES ESTADOS: RN, AL, BA, CE, PB, PE, SE, FN, GO, MT, DF, SA,MG, RJ, PR,SP E RS.

Com isso, inferimos que o layout desenhado para a amostra de 25% (com apenas algumas UFs) pelos t�cnicos do IBGE n�o � o mesmo daquele para a de 1,27% (com todas as UFs), contida no arquivo de dados.

Outra fonte de confus�o diz respeito ao t�tulo e conte�do do script em formato SAS, que informam que se trataria de uma amostra de 1%. No entanto, isso n�o parece ter fundamento. A amostra de 1,27% descrita no Volume 2 dos Resultados Preliminares do Censo Demogr�fico de 1960 (IBGE, 1965) teria sido gerada por sorteio aleat�rio e equiprobabilistico. Se isso � verdade, ent�o as propor��es de quaisquer categorias de vari�veis no banco de dados seriam semelhantes �s (convergiriam em probabilidade para as) populacionais; sem qualquer necessidade de corre��es por pesos amostrais. E, de fato, se criamos pesos amostrais id�nticos para todos os casos (com valor igual � 1/0,0127 - um sobre a fra��o amostral) para servir de mero fator de expans�o ) obtemos resultados e totais muito semelhantes aos n�meros das tabula��es oficiais. N�o h� raz�es para crer que o arquivo de dados diga respeito a outra vers�o, que n�o aquela originalmente extra�da em 1965.

Como layout, deste modo, seguiremos a estrutura descrita nos arquivo em SPSS e SAS, ao inv�s daquela apresentada no dicion�rio. No entanto, � preciso pontuar, os valores e r�tulos das categorias s�o os mesmos em todas essas tr�s fontes, a despeito das divergencias nas localiza��es dos caracteres.

A aus�ncia de um dicion�rio dos dados n�o � um problema de todo ignor�vel: as duas sintaxes de abertura n�o fazem a leitura de todos os caracteres do arquivo de dados e, na aus�ncia de uma documenta��o oficial e completa, n�o h� como avaliar de modo un�voco a natureza das informa��es guardadas naquelas posi��es. Mais especificamente, os caracteres localizados entre as posi��es (colunas) de 7 a 16 do arquivo de texto est�o sendo ignorados. Diversos testes e cruzamentos foram realizados na tentativa de identificar esses conte�dos, por�m sem sucesso.

Al�m disso, as seguintes variaveis estao citadas no dicionario de codigos (feito para a amostra de 25%), mas nao estao sendo abertas pelas sintaxes constru�das para o arquivo de dados (e nao parecem estar contidas nos caracteres de 7 a 16):

-   V100 - TOTAL DE PESSOAS
-   V119 - TOTAL FAM�LIAS
-   V120 - TOTAL MORADORES
-   V121 - PESO DOMIC�LIO
-   V122 - FILLER

A principio, � possivel calcular V100, V119 e V120 a posteriori. Com respeito � V121, se os dados forem mesmo uma amostra autoponderada, como supusemos acima, os pesos dos domic�lios poder�o ser calculadas tamb�m como iguais a 1/0,0127 para todos os registros. O conteudo da v122 n�o � descrito em nenhum outro lugar. Mas, pela sua denomina��o, parece ser tratar apenas de um caractere de preenchimento ou algum tipo de espa�o vazio, separando se��es dos dados, sem outro conte�do substantivo.

H� tamb�m variaveis de identificacao listadas no inicio do dicionario, mas n�o que s�o referidas nas sintaxes:

-   1 - PASTA
-   2 - BOLETIM
-   3 - IDENTIFICA��O
-   4 - DIGITO VERIFICADOR

Possivelmente, a elas se refere o conte�do dos caracteres de 7 a 16: identificadores do domic�lio, local de moradia e caracter�sticas que poderiam at� auxiliar na compreens�o do plano amostral. Mas n�o h� m�todo determ�stico e confi�vel para separar o teor dessas colunas, nem informa��es externas para valid�-los. Deste modo os caracteres de 7 a 16 permanecer�o ignorados.

Por fim, h� duas vari�veis bastante importantes listadas apenas nas sintaxes -- e n�o no dicion�rio: s�o elas as vari�veis sobre unidade da federacao (UF) e sobre o tipo de registro (RECD). A primeira identifica o lugar de resid�ncia do indiv�duo, onde a entrevista foi realizada, a segunda, discrimina a natureza da informa��o contida nas linhas: se fam�lias ou pessoas.

------------------------------------------------------------------------

**IMPORTANTE**:

O arquivo de dados de domicilios parece dizer respeito, na verdade, a dados de **FAMILIAS**. Os ind�cios s�o os seguintes:

1.  A variavel V101 identifica familias dentro do domicilio particular. Se v101 = 4, trata-se da segunda familia dentro de um domicilio particular.
2.  Registros identificados como 2a ou 3a familia (v101 == 4 ou v101 == 5) nao cont�m informa��es para as vari�veis v102 a 113. Presume-se que apenas a primeira familia traz consigo as informa��es estruturais sobre o domic�lio (n�mero de c�modos, acesso � energia el�trica etc.).

Sob esta suposi��o, apenas os registros identificados como v101 == 1, 2 ou 3 s�o de fato domicilios. Os procedimentos que aplicaremos deste ponto em diante partir�o desta suposi��o.

------------------------------------------------------------------------

Abertura dos dados e identifica��o de registros de fam�lias e pessoas
---------------------------------------------------------------------

Inicialmente, o arquivo de dados foi aberto como um grande arquivo de texto (formato character/string), sem separa��o de colunas.

``` r
# Objeto que guarda as os dados originais (em formato string/character)
c60_string <- readLines("Original Files/HHOLDA.txt")

# Remove os h�fens existentes no final das linhas
c60_string <- gsub(x = c60_string, pattern = "-", " ")

# Vetor que guarda o n�mero de linhas existentes (ser� usado para criar IDs de pessoas, fam�lias e domic�lios)
n_linhas   <- length(c60_string) 
```

Todos os registros apresentaram o mesmo tamanho:

``` r
freq(nchar(c60_string), plot = F)
```

    ## nchar(c60_string) 
    ##       Frequ�ncia Percentual
    ## 62       1074328        100
    ## Total    1074328        100

Foi preciso ent�o identificar e separar os registros de fam�lias e pessoas. A princ�pio, de acordo com o layout descrito nas sintaxes de abertura, essa informa��o estaria contida no caractere da posi��o 17 no arquivo de dados. A frequencia das categorias dessa vari�vel revela a seguinte distribui��o:

``` r
teste_char17_dom_pess = substr(c60_string, start = 17, stop = 17)
freq(teste_char17_dom_pess, plot = F)
```

    ## teste_char17_dom_pess 
    ##       Frequ�ncia Percentual
    ## 1         174467      16.24
    ## 2         174472      16.24
    ## 3         725389      67.52
    ## Total    1074328     100.00

De acordo com informa��es contidas nas sintaxes, o valor 1 identificaria os registros de domic�lios -- por conseguinte, os valores 2 e 3, diriam respeito �s pessoas. N�o h� na documenta��o, contudo, r�tulo para o conte�do desses dois valores. Por�m, posteriormente, cruzando essa informa��o com a vari�vel sobre rela��o com o chefe da fam�lia, � poss�vel inferir os seguintes significados das alternativas:

1.  Registro de familia
2.  Registro da pessoa na posi��o de chefe da familia
3.  Registro de outros moradores

A princ�pio, deveria haver um n�mero id�ntico de registros de fam�lias de chefes. H�, no entanto, uma pequena diferen�a, de 5 casos, como aponta a tabela acima - uma primeira inconsist�ncia detectada aqui.

A partir das sintaxes de abertura, um arquivo de layout foi elaborado para pessoas e fam�lias, para facilitar a leitura das posi��es das vari�veis, al�m de guardar outras informa��es relevantes sobre as vari�veis. � a esse arquivo que faremos refer�ncia a partir daqui, quando nos referirmos ao layout dos dados:

``` r
input_file <- "Auxiliary Files/Census1960_input_Sample_1.27.xlsx"
```

### Registros de fam�lias

Utilizando da informa��o acima, criamos ent�o um objeto separado para os dados de fam�lias, selecionando apenas as linhas que, no caractere 17, apresentaram o valor um:

``` r
# String apenas com registros de fam�lias
c60_string_hh <-   c60_string[which(teste_char17_dom_pess == 1)]
num_linha_dom <- (1:n_linhas)[which(teste_char17_dom_pess == 1)]
```

Abrimos tamb�m o arquivo de layout refere �s familias:

``` r
input_hh   <- read_xlsx(input_file, sheet = "Family_open")
input_hh
```

    ## # A tibble: 23 x 5
    ##    Var            Start   End Valid_values      obs                       
    ##    <chr>          <dbl> <dbl> <chr>             <chr>                     
    ##  1 UF                 1     2 sheet             <NA>                      
    ##  2 V116               3     6 <NA>              <NA>                      
    ##  3 place_holder_~     7     8 <NA>              pode ser o distrito do mu~
    ##  4 place_holder_~     9    16 <NA>              pode ser um ID de Fam�lia~
    ##  5 REC_TYPE          17    17 1;2;3             <NA>                      
    ##  6 V118              18    18 1;3;5             <NA>                      
    ##  7 V101              19    19 1;2;3;4;5;9       <NA>                      
    ##  8 V102              20    20 ;4;5;6;7          <NA>                      
    ##  9 V103              21    21 ;7;8;9;0          <NA>                      
    ## 10 V104              22    22 ;0;1;2;3;4;5;6;7~ <NA>                      
    ## # ... with 13 more rows

A partir da an�lise e leitura do layout, observamos:

-   O caractere 32 n�o est� sendo lido ou nem parece fazer parte de qualquer vari�vel
-   Os caracteres de 35 a 54 (intervalo de 20 posi��es), igualmente, n�o s�o lidos

No primeiro caso, na posi��o 32, encontramos 1.257 espa�os vazios, 173.209 valores zero, 1 (um) valor 2, como mostra a tabela abaixo.

``` r
teste_char32_dom = substr(c60_string_hh, start = 32, stop = 32)
freq(teste_char32_dom, plot = F)
```

    ## teste_char32_dom 
    ##       Frequ�ncia  Percentual
    ##             1257   0.7204801
    ## 0         173209  99.2789467
    ## 2              1   0.0005732
    ## Total     174467 100.0000000

Esses resultados parecem indicar que o conte�do esperado para essa c�lula seria o valor zero. Contudo, possivelmente por inconsistencia dos registros, outros conte�dos figuram. No caso do intervalo entre os caracteres 35 e 54, parece ocorrer um menor n�mero de problemas -- que, por�m, n�o s�o desprovidos de import�ncia, como mostra a tabela a seguir:

``` r
teste_char_35_54_dom = substr(c60_string_hh, start = 35, stop = 54)
freq(teste_char_35_54_dom, plot = F)
```

    ## teste_char_35_54_dom 
    ##                      Frequ�ncia  Percentual
    ##                          174465  99.9988537
    ## 02                            1   0.0005732
    ## 81 07 601 71 11 4789          1   0.0005732
    ## Total                    174467 100.0000000

Veremos adiante que esses dois casos problem�ticos s�o registros corrompidos -- provavelmente devido a erros de grava��o, t�picos dos antigos m�todos de grava��o serial (por meio de fitas magn�ticas). Possivelmente se devem a algum tipo de descarrilhamento dos cabe�otes de leitura/grava��o. Haver� muitos outros problemas da mesma natureza.

Aplicamos ent�o a fun��o especificamente constru�da para aplicar o arquivo de layout sobre os dados, separando as colunas e testando se os valores das observacoes em cada variavel est�o de fato listadas dentro do escopo de possibilidades definidas no dicionario:

``` r
censo1960_hh <- aplicaTesta_layout(dados_string = c60_string_hh,
                                   input        = input_hh) %>%
        mutate(
                # adicionamos uma coluna que indica a linha onde o registro se encontra no arquivo de dados
                num_linha_dom = num_linha_dom
                ) %>% 
        select(num_linha_dom, everything())
```

O resultado � um banco de dados no qual todas as colunas ainda possuem o formato texto (string/character), guardando todas as informa��es originais (sem transformar campos num�ricos em integer ou doubles, por exemplo, o que removeria os zeros � esquerda). Al�m disso, uma coluna adicional de verifica��o foi criada ao lado de cada vari�vel, indicando `TRUE` quando os valores observados s�o v�lidos (listados no dicion�rio) e `FALSE`, caso contr�rio.

``` r
censo1960_hh
```

    ## # A tibble: 174,467 x 43
    ##    num_linha_dom UF    test_UF V116  test_V116 place_holder_07~
    ##            <int> <chr> <lgl>   <chr> <lgl>     <chr>           
    ##  1             1 00    TRUE    0011  TRUE      01              
    ##  2             4 00    TRUE    0011  TRUE      01              
    ##  3             6 00    TRUE    0011  TRUE      01              
    ##  4            27 00    TRUE    0011  TRUE      01              
    ##  5            32 00    TRUE    0011  TRUE      01              
    ##  6            34 00    TRUE    0011  TRUE      01              
    ##  7            40 00    TRUE    0011  TRUE      01              
    ##  8            44 00    TRUE    0011  TRUE      01              
    ##  9            55 00    TRUE    0011  TRUE      01              
    ## 10            67 00    TRUE    0011  TRUE      01              
    ## # ... with 174,457 more rows, and 37 more variables:
    ## #   place_holder_09_16 <chr>, REC_TYPE <chr>, test_REC_TYPE <lgl>,
    ## #   V118 <chr>, test_V118 <lgl>, V101 <chr>, test_V101 <lgl>, V102 <chr>,
    ## #   test_V102 <lgl>, V103 <chr>, test_V103 <lgl>, V104 <chr>,
    ## #   test_V104 <lgl>, V105 <chr>, test_V105 <lgl>, V106 <chr>,
    ## #   test_V106 <lgl>, V107 <chr>, test_V107 <lgl>, V108 <chr>,
    ## #   test_V108 <lgl>, V109 <chr>, test_V109 <lgl>, V110 <chr>,
    ## #   test_V110 <lgl>, V111 <chr>, test_V111 <lgl>, V112 <chr>,
    ## #   test_V112 <lgl>, place_holder_32_32 <chr>, V113 <chr>,
    ## #   test_V113 <lgl>, place_holder_35_54 <chr>, barra <chr>,
    ## #   test_barra <lgl>, ID <chr>, test_ID <lgl>

O segundo passo � identificar as linhas que cont�m ao menos um erro. Para isso, criamos uma coluna adicional que aponta os registros em que h� pelo menos um valor `FALSE` nas vari�veis de teste:

``` r
censo1960_hh$contains_invalid <- censo1960_hh %>% 
        select(starts_with("test_")) %>%
        mutate_all(function(x) as.numeric(!(x))) %>%
        replace(is.na(.), 0) %>%
        rowSums(.)
```

Devemos tamb�m nos assegurar de o ID das fam�lias (valor que se inicia com `\0000001` e assim por diante) de fato � �nico para cada registro. Trata-se de uma informacao que parece ter sido adicionada ao banco posteriormente, sempre a partir do caractere 55de cada linha. O seguintes aspectos devem ser observados: a) como se trata de um valor, esse campo deve poder ser transformada em numerico sem gerar qualquer "caso perdido" (*missing*); b) n�o pode haver valores repetidos (afinal se trata de um ID).

Como se pode observar, nenhum valor missing � produzido pela convers�o do campo em num�rico:

``` r
id_numeric <- as.numeric(censo1960_hh$ID)
paste("Quantidade de valores missing:", sum(is.na(id_numeric)))
```

    ## [1] "Quantidade de valores missing: 0"

Al�m disso, tamb�m n�o se encontra valores repetidos:

``` r
paste("Quantidade de valores repetidos:", sum(duplicated(id_numeric)))
```

    ## [1] "Quantidade de valores repetidos: 0"

Os dois crit�rios s�o satisfeitos. Examinemos ent�o os casos com pelo menos um registro problem�tico, segundo o crit�rio aventado acima. Os testes preliminares revelam que s�o 12 casos problem�ticos, em que se se verificou pelo menos um valor inv�lido, n�o listado no dicion�rio de c�digos. Gravaremos ent�o num objeto o n�mero da linha desses registros identificados:

``` r
# Problema 1: presenca de caracteres ou valores n�o listados como validos pelo dicionario de codigos:
hh_linhas_problemas_1 <- censo1960_hh %>% 
        filter(contains_invalid > 0) %>%
        .$num_linha_dom
```

Um segundo tipo de problema diz respeito � presen�a de outros tipos de caracteres em quaisquer posi��es do registro (inclusive aquelas n�o lidas, como os caracteres de 7 a 16, 32 e de 35 a 54). Fazemos ent�o uma busca por express�es regulares e registramos os n�meros das linhas dos casos problem�ticos:

``` r
# Problema 2: presenca de outros tipos de caracteres invalidos
hh_linhas_problemas_2 = grep(pattern = "[[:alpha:]]|[[:cntrl:]]|[[:punct:]]" , 
                               x       = gsub(pattern     = "[\\]", 
                                              replacement = "", 
                                              x           = c60_string_hh))
hh_linhas_problemas_2 = censo1960_hh[hh_linhas_problemas_2,"num_linha_dom"] %>% unlist()
```

S�o exemplos desse tipo de ocorr�ncia:

``` r
writeLines(c60_string[hh_linhas_problemas_2[1:7]])
```

    ## 212166X722366154151478384680205003                    \0037520
    ## 54554131553021421114849'2570202001                    \0104316
    ## 54554232553422051314821'0570204002                    \0104558
    ## 54554232553422591314849'2570206003                    \0104612
    ## 60623477616981071114099@2570204002                    \0124092
    ## 60623477616981091114869'2579105004                    \0124094
    ## 71724Z0771038216151598279680202001                    \0145606

O terceiro tipo de problema est� ligado � exist�ncia de espa�os em branco adicionais inseridos entre valores v�lidos, levando o registro a experimentar uma esp�cie de "descarrilhamento" da grava��o. Em boa parte dos casos dessa natureza, basta remover os espa�os brancos e ent�o o padr�o dos dados passa a fazer sentido: encontramos valores v�lidos para as vari�veis e padr�es de resposta consistentes. A identifica��o desses espa�os vazios � feita atrav�s do c�digo abaixo:

``` r
# Problema 3: presenca de espacos (at� 3) entre caracteres
hh_linhas_problemas_3 = grep(pattern = "[[:digit:]][[:blank:]]{1,3}[[:digit:]]",
                             x       = c60_string_hh)
hh_linhas_problemas_3 = censo1960_hh[hh_linhas_problemas_3,"num_linha_dom"] %>% unlist()
```

S�o exemplos desse tipo de ocorr�ncia:

``` r
writeLines(c60_string[hh_linhas_problemas_3[1:10]])
```

    ## 313131013143809415157838 680207002                    \0047256
    ## 5151680151242001111483 42580204001                    \0088188
    ## 52531101534000171314 81525 0205002                    \0096545
    ## 545441025438200711148294257920 001                    \0099422
    ## 5454410354382129111485 42579104001                    \0099536
    ## 60647201647541301114   42   20 001                    \0132129
    ## 60663601655660031 11 478941579207002                   0136455
    ## 74750901746262331515782896 0202001                    \0153837
    ## 182 1000000008371 323326\81 82620181 07 601 71 11 478940155539
    ## 818265098206021715148726967 210004                    \0161029

Observe que a �ltima linha do exemplo acima � justamente um dos registros identificados como problem�ticos anteriormente, por apresentar valores num�ricos no intervalo entre os caracteres 35 e 54, que deveria estar vazio.

Combinamos agora, num s� objeto, as linhas que contemplam os tr�s tipos de problemas definidos acima:

``` r
hh_linhas_problemas = c(hh_linhas_problemas_1, hh_linhas_problemas_2, hh_linhas_problemas_3) %>%
        unique() %>%
        sort()
```

Trata-se, assim de 29 registros problem�ticos, que dever�o receber aten��o focada -- manualmente, atrav�s de leitura e interpreta��o, o que permitir� elencar qual o tipo de solu��o (caso exista alguma) pode ser aplicada em cada caso. Salvamos, deste modo, um arquivo contendo as informa��es completas e os testes sobre cada vari�vel para todos aqueles registros identificados como problem�ticos.

``` r
write.csv2(x = censo1960_hh %>% 
                   mutate(string = c60_string_hh) %>%
                   filter(num_linha_dom %in% hh_linhas_problemas) %>%
                   select(num_linha_dom, string, starts_with("test_"), contains_invalid),
           file = "Auxiliary Files/linhas_problematicas_domicilios.csv",
           row.names = F)
```

Ap�s tal avalia��o detida, foi adicionada uma coluna de diagn�stico, indicando, para cada um dos registros problem�ticos, a a gravidade e a naturezado problema encontrado.

``` r
checks_hh   <- read_xlsx("Auxiliary Files/check_line_by_line.xlsx", sheet = "households")
freq(checks_hh$diagnostico, plot = F)
```

    ## checks_hh$diagnostico 
    ##                                                        Frequ�ncia
    ## corrupted_registry                                              1
    ## del_space_between=2; insert_space_end=1; insert_bar= 1          1
    ## ok                                                             27
    ## Total                                                          29
    ##                                                        Percentual
    ## corrupted_registry                                          3.448
    ## del_space_between=2; insert_space_end=1; insert_bar= 1      3.448
    ## ok                                                         93.103
    ## Total                                                     100.000

Os registros marcados como `ok` s�o aqueles que trazem problemas em apenas uma vari�vel, devido � presen�a de caracteres ou valores inv�lidos -- no entanto, o restante dos valores do mesmo registro n�o aprensentam problemas. O registro marcado como `del_space_between=2; insert_space_end=1; insert_bar= 1` � um caso um pouco mais grave: apresenta, em duas posi��es, espa�os adicionais separando valores v�lidos. Al�m disso, n�o cont�m a barra investida que usualmente antecede a informa��o sobre o ID da fam�lia. Mas trata-se de um problema com solu��o: remover os dois espa�os adicionais entre valores, inserindo, em seguida a barra (com um espa�o a antecedendo) resolve o problema. Trata-se, deste modo, de um registro corrompido, mas recuper�vel. H� certamente o suposto de que nenhum outro valor foi alterado. Mais adiante executaremos esses procedimentos de corre��o. Outros testes de consist�ncia, contudo, devem ser realizados. � preciso avaliar a coer�ncia entre respostas de diferentes quest�es.

-   Para domicilios coletivos (V101 == 3), todas as variaveis entre V102 e V113 devem estar em branco -- s�o quest�es que n�o se aplicam a esses registros. Todos as frequencias abaixo mostram que esse crit�rio foi satisfeito:

``` r
censo1960_hh %>%
        
        # neste passo nao analisaremos os registros problematicos
        filter(!(num_linha_dom %in% hh_linhas_problemas)) %>% 
        filter(V101 == 3) %>%
        select(starts_with("V")) %>%
        select(V102:V113) %>%
        map(., function(x) freq(x, plot=F))
```

    ## $V102
    ## x 
    ##       Frequ�ncia Percentual
    ##              835        100
    ## Total        835        100
    ## 
    ## $V103
    ## x 
    ##       Frequ�ncia Percentual
    ##              835        100
    ## Total        835        100
    ## 
    ## $V104
    ## x 
    ##       Frequ�ncia Percentual
    ##              835        100
    ## Total        835        100
    ## 
    ## $V105
    ## x 
    ##       Frequ�ncia Percentual
    ##              835        100
    ## Total        835        100
    ## 
    ## $V106
    ## x 
    ##       Frequ�ncia Percentual
    ##              835        100
    ## Total        835        100
    ## 
    ## $V107
    ## x 
    ##       Frequ�ncia Percentual
    ##              835        100
    ## Total        835        100
    ## 
    ## $V108
    ## x 
    ##       Frequ�ncia Percentual
    ##              835        100
    ## Total        835        100
    ## 
    ## $V109
    ## x 
    ##       Frequ�ncia Percentual
    ##              835        100
    ## Total        835        100
    ## 
    ## $V110
    ## x 
    ##       Frequ�ncia Percentual
    ##              835        100
    ## Total        835        100
    ## 
    ## $V111
    ## x 
    ##       Frequ�ncia Percentual
    ##              835        100
    ## Total        835        100
    ## 
    ## $V112
    ## x 
    ##       Frequ�ncia Percentual
    ##              835        100
    ## Total        835        100
    ## 
    ## $V113
    ## x 
    ##       Frequ�ncia Percentual
    ##              835        100
    ## Total        835        100

-   O cruzamento entre a especie de domicilio (v101: particular unico, particular com mais de uma familia ou coletivo) e o tipo de domic�lio (v102: duravel, rustico, improvisado etc) deve apresentar um padr�o espec�fico: \* Se v101 = particular unico (1) ou 1a familia (2), v102 pode assumir quaisquer valores entre 4 e 7 \* Se v101 = coletivo (3), v102 deve ser deixada em branco \* Se v101 = 2a ou 3a familia ou boletim individual (4 ou 5), v102 deve ser deixada em branco

O cruzamento abaixo mostra que todas essas condi��es s�o satisfeitas.

``` r
with(censo1960_hh %>% filter(!(num_linha_dom %in% hh_linhas_problemas)), {
        table(V101, V102)
})
```

    ##     V102
    ## V101             4      5      6      7
    ##    1      0 125114  47710     49      9
    ##    2      0    291     57      0      0
    ##    3    835      0      0      0      0
    ##    4    345      0      0      0      0
    ##    5     28      0      0      0      0

-   Para domicilios particulares improvisados (V102 == 6), as variaveis V103 a V113 devem permanecer em branco. Como podemos observar, essa condi��o tamb�m � satisfeita:

``` r
censo1960_hh %>%
        filter(!(num_linha_dom %in% hh_linhas_problemas)) %>%
        filter(V102 == 6) %>%
        select(starts_with("V")) %>%
        select(V103:V113) %>%
        map(., function(x) freq(x, plot=F))
```

    ## $V103
    ## x 
    ##       Frequ�ncia Percentual
    ##               49        100
    ## Total         49        100
    ## 
    ## $V104
    ## x 
    ##       Frequ�ncia Percentual
    ##               49        100
    ## Total         49        100
    ## 
    ## $V105
    ## x 
    ##       Frequ�ncia Percentual
    ##               49        100
    ## Total         49        100
    ## 
    ## $V106
    ## x 
    ##       Frequ�ncia Percentual
    ##               49        100
    ## Total         49        100
    ## 
    ## $V107
    ## x 
    ##       Frequ�ncia Percentual
    ##               49        100
    ## Total         49        100
    ## 
    ## $V108
    ## x 
    ##       Frequ�ncia Percentual
    ##               49        100
    ## Total         49        100
    ## 
    ## $V109
    ## x 
    ##       Frequ�ncia Percentual
    ##               49        100
    ## Total         49        100
    ## 
    ## $V110
    ## x 
    ##       Frequ�ncia Percentual
    ##               49        100
    ## Total         49        100
    ## 
    ## $V111
    ## x 
    ##       Frequ�ncia Percentual
    ##               49        100
    ## Total         49        100
    ## 
    ## $V112
    ## x 
    ##       Frequ�ncia Percentual
    ##               49        100
    ## Total         49        100
    ## 
    ## $V113
    ## x 
    ##       Frequ�ncia Percentual
    ##               49        100
    ## Total         49        100

------------------------------------------------------------------------

Observamos, deste modo, que com excessao dos 29 registros problematicos identificados, o arquivo de dados de domicilios parece consistente.

#### Corrigindo os registros de domicilios

Marcamos ent�o os casos, de acordo com as avalia��es feitas anteriormente:

1.  Problema corrigido: O arquivo de dados original (txt) apresentava caracteres deslocados.
2.  Problema n�o corrigido, mas ignor�vel: uma ou algumas vari�veis apresentavam valores inv�lidos (n�o listados no dicion�rio)
3.  Registro n�o problem�tico
4.  Registro completamente corrompido.

A rotina abaixo executa os procedimentos necess�rios para marcar os casos segundo esses r�tulos:

``` r
source("https://raw.githubusercontent.com/antrologos/ConsistenciaCenso1960Br/master/Code/Family_diagnostics_procedure.R")
```

Abriremos novamente o arquivo de dados em formato fixo aplicando o layout, e, desta vez, adicionando-lhe o resultados dos diagn�sticos:

``` r
censo1960_hh <- aplicaTesta_layout(dados_string = c60_string_hh,
                                   input        = input_hh) %>%
        
        mutate(
                # adicionamos uma coluna que indica a linha onde o registro se encontra no arquivo de dados
                num_linha_dom = num_linha_dom,
                
                # Adicionamos uma coluna com os resultados dos procedimentos de diagn�stico
                cem_diagnosis_dom = diagnosis) %>% 
        select(num_linha_dom, everything())
```

Marcamos ent�o as linhas que cont�m ao menos um erro por meio de uma vari�vel espec�fica:

``` r
censo1960_hh$cem_vars_still_problematic_dom <- censo1960_hh %>% 
        select(starts_with("test_")) %>%  
        mutate_all(function(x) as.numeric(!(x))) %>%
        mutate_all(function(x) ifelse(is.na(x), 0, x)) %>% 
        rowSums(.)
```

Criamos ent�o uma outra vari�vel nova, que lista o nome das variaveis corrompidas em cada registro:

``` r
casos_problematicos <- censo1960_hh %>%
        filter(cem_vars_still_problematic_dom > 0) 

teste_problematicos <- casos_problematicos %>%
        select(starts_with("test_"), -test_ID, -test_barra)

casos_problematicos$cem_problematic_vars_list_dom = ""
vars <- names(teste_problematicos)
for(var in vars){
        problematic_cases <- which(!teste_problematicos[[var]])
        var_name = gsub(pattern = "test_", replacement = "" , x = var)
        casos_problematicos[problematic_cases, "cem_problematic_vars_list_dom"] = 
                paste(casos_problematicos[problematic_cases, ]$cem_problematic_vars_list_dom, var_name)
}

casos_problematicos$cem_problematic_vars_list_dom <- str_trim(casos_problematicos$cem_problematic_vars_list_dom)

censo1960_hh <- left_join(x = censo1960_hh,
                          y = casos_problematicos %>%
                                  select(num_linha_dom, cem_problematic_vars_list_dom),
                          by = "num_linha_dom")
```

Substituimos ent�o os valores invalidos por missing:

``` r
num_linha_dom_prob <- censo1960_hh %>%
        filter(cem_vars_still_problematic_dom >= 1) %>%
        .$num_linha_dom

for(i in num_linha_dom_prob){
        vars_prob <- censo1960_hh[censo1960_hh$num_linha_dom == i, "cem_problematic_vars_list_dom"] %>% 
                unlist() %>%
                strsplit(split = " ") %>%
                unlist() %>%
                str_trim()
        var = vars_prob[2]
        for(var in vars_prob){
                censo1960_hh[censo1960_hh$num_linha_dom == i,][[var]] <- NA        
        }
}
```

Por fim, compilando o banco de dados consistido das fam�lias, em seu formato preliminar. Removemos as colunas de teste, Transformamos as vari�veis originais em num�ricas e acoplamos as vari�veis sint�ticas de diagn�stico que criamos.

``` r
censo1960_hh_numeric <- censo1960_hh%>%
        select(-starts_with("test_"), - barra) %>%
        select(-cem_diagnosis_dom, -cem_problematic_vars_list_dom,
               -starts_with("place_")) %>%
        mutate_all(as.numeric)

censo1960_hh_character <- censo1960_hh %>%
        select(cem_diagnosis_dom, cem_problematic_vars_list_dom,
               starts_with("place_"))

censo1960_hh_preliminar <- cbind(censo1960_hh_numeric, censo1960_hh_character) %>% as_tibble()
```

### Registros de pessoas

A consist�ncia dos registros de pessoas seguir� a mesma ordem e estrutura daquela aplicada para os registros de fam�lias. Primeiramente, carregamos o arquivo de layout, criado a partir das sintaxes:

``` r
input_person  <- read_xlsx(input_file, sheet = "Person_open")
```

No arquivo de dados original, ainda em formato de texto com caracteres de posi��es fixas, selecionamos apenas os dados de pessoas (valores diferentes de 1 na coluna 17):

``` r
c60_string_pess <-   c60_string[which(teste_char17_dom_pess != 1)]
num_linha_pess  <- (1:n_linhas)[which(teste_char17_dom_pess != 1)]
```

Aplicamos ent�o o arquivo de layout para separar as colunas e executamos testes para averiguar se os valores encontrados no arquivo de dados est�o contemplados no dicion�rio de c�digos:

``` r
censo1960_pess <- aplicaTesta_layout(dados_string = c60_string_pess,
                                   input        = input_person) %>%
        mutate(num_linha_pess = num_linha_pess) %>% 
        select(num_linha_pess, everything())
```

Identificamos ent�o as linhas que cont�m ao menos um erro:

``` r
censo1960_pess$contains_invalid <- censo1960_pess %>% 
        select(starts_with("test_")) %>%                          
        mutate_all(function(x) as.numeric(!(x))) %>%
        mutate_all(function(x) ifelse(is.na(x), 0, x)) %>% 
        rowSums(.) 
```

Gravamos ent�o, num objeto � parte, os identificadores �nicos (n�meros das linhas) dos registros problematicos:

``` r
# Problema 1: presenca de caracteres nao listados como validos pelo dicionario de codigo
pess_linhas_problemas_1 <- censo1960_pess %>% 
        filter(contains_invalid > 0) %>%
        .$num_linha_pess
```

As seguintes linhas do arquivo de dados original (89 no total) apresentaram problemas desta natureza. Alguns exemplos desse desse tipo de ocorr�ncia s�o:

``` r
writeLines(c60_string[pess_linhas_problemas_1[1:10]])
```

    ## 02024301021102572117128571"912183110085404048352425156\0001403
    ## 17180603174940273329105532392000311                   \0024733
    ##    \212135072103211531 291 08542092000052              0029831
    ## 2121190121536143352812554209142031 100953000034        0030734
    ## 2121190121536245352911 857 209200014200000000034       0030823
    ## 212135072103211531 2813554209200016440648060503191281550032581
    ## 212138052189801725171255720R20003110095702025332321176\0035191
    ## 212144012199206635281605420 200031100625000034        \0035911
    ## 212166X72236615425171505520 20003110040010055332321207\0037520
    ## 31   M       1BA33HIA                                 \0046690

Observe que no �ltimo caso do exemplo acima, n�o h� apenas d�gitos, mas tamb�m caracteres do alfabeto, formando, aproximadamente, a palavra "BAHIA". Trate-se muito provavelmente de um erro de grava��o em consequ�ncia das mudan�as de m�dia, formato e esquemas de codifica��o. O dicion�rio de c�digos que acompanhava o conjunto de dados recebidos como doa��o pelo Centro de Estudos da Metr�pole menciona que ao menos um processo do tipo teria sido realizado para a amostra de 25%:

> O ARQUIVO ORIGINAL ESTAVA GRAVADO EM FORMATO EBCDIC (FORMATO PARA M�QINAS IBM) E ALGUMAS VARI�VEIS EM C�DIGO BIN�RIO. OS DADOS FORAM RECUPERADOS E GRAVADOS EM FORMATO ASCII. ESTE TRABALHO FOI REALIZADO PELO POPULATION RESEARCH CENTER DA UNIVERSIDADE DO TEXAS�AUSTIN, EM COMUM ACORDO COM O IBGE.

H� raz�es para crer que procedimentos de leitura, grava��o e convers�o do mesmo g�nero teriam ocorrido tamb�m para a amostra de 1,27%.

Passamos � identifica��o do segundo tipo de problema: a presen�a de caracteres inv�lidos:

``` r
# Problema 2: presenca de outros tipos de caracteres invalidos
pess_linhas_problemas_2 = grep(pattern = "[[:alpha:]]|[[:cntrl:]]|[[:punct:]]" , 
                               x       = gsub(pattern     = "[\\]", 
                                              replacement = "", 
                                              x           = c60_string_pess))
pess_linhas_problemas_2 = censo1960_pess[pess_linhas_problemas_2,"num_linha_pess"] %>% unlist()
```

No todo, foram encontrados 36 desse tipo. S�o exemplos desta ocorr�ncia:

``` r
writeLines(c60_string[pess_linhas_problemas_2[1:10]])
```

    ## 02024301021102572117128571"912183110085404048352425156\0001403
    ## 212138052189801725171255720R20003110095702025332321176\0035191
    ## 212166X72236615425171505520 20003110040010055332321207\0037520
    ## 31   M       1BA33HIA                                 \0046690
    ## 54   M       4GU35ANABARA                             \0097157
    ## 54553230552620473319011'41292000                      \0104113
    ## 5455413155302026211712'5423908231620095702029385125176\0104200
    ## 54554131553020862117144'708909081620064006069361328145\0104260
    ## 54554131553021433129102'41292000                      \0104317
    ## 54554131553021443119116'4129200008210000000035        \0104318

Repare que novamente nos deparamos com o registro que cont�m caracteres formando, de modo aproximado, a palavra "BAHIA" (UF da qual o registro de fato faria parte, de acordo com os dois d�gitos iniciais). Ou seja, os dois m�todos de identifica��o de erros capturam os problemas existentes nessa linha. Al�m disso, observamos que um caso bastante semelhante ocorre logo em seguida, no qual se forma, tamb�m de forma aproximada, a palavra "GUANABARA". Esses s�o dois registros completamente corrompidos, que, por precau��o, dever�o ser ignorados. **Casos dessa natureza ter�o suas vari�veis marcadas como missing e ser�o assinalados, em vari�vel apropriada, criada pelo Centro de Estudos da Metr�pole, como registros inutiliz�veis para a maioria das an�lises**.

Por fim, buscamos pela presen�a de caracteres adicionais, separando vari�veis e valores que deveriam ser adjacentes nos dados:

``` r
# Problema 3: presenca de espacos (at� 3) entre caracteres
pess_linhas_problemas_3 = grep(pattern = "[[:digit:]][[:blank:]]{1,3}[[:digit:]]",
                             x       = c60_string_pess)
pess_linhas_problemas_3 = censo1960_pess[pess_linhas_problemas_3,"num_linha_pess"] %>% unlist()
```

S�o exemplos desse tipo de ocorr�ncia:

``` r
writeLines(c60_string[pess_linhas_problemas_3[1:10]])
```

    ## 14164101152 00673519104570792000                      \0021501
    ##    \212135072103211531 291 08542092000052              0029831
    ## 2121190121536143352812554209142031 100953000034        0030734
    ## 2121190121536245352911 857 209200014200000000034       0030823
    ## 212135072103211531 2813554209200016440648060503191281550032581
    ## 212144012199206635281605420 200031100625000034        \0035911
    ## 212166X72236615425171505520 20003110040010055332321207\0037520
    ## 2122620722 34069352911354209200015200000000034        \0040955
    ## 31 1 80731608050352911557059200031100000000034        \0046690
    ## 4040940140880001311 1 954169200017210400130776        \0064744

Compilamos num objeto as linhas detectadas como problem�ticas segundo as tr�s defini��es acima e salvamos num arquivo separado os dados e diagn�sticos sobre os registros de pessoas, que ser�o avaliados manualmente:

``` r
pess_linhas_problemas = c(pess_linhas_problemas_1, pess_linhas_problemas_2, pess_linhas_problemas_3) %>%
        unique() %>%
        sort()

write.csv2(x = censo1960_pess %>% 
                   mutate(string = c60_string_pess) %>%
                   filter(num_linha_pess %in% pess_linhas_problemas) %>%
                   select(num_linha_pess, string, starts_with("test_"), contains_invalid),
           file = "Auxiliary Files/linhas_problematicas_pessoas.csv",
           row.names = F)
```

Trata-se, assim, de um banco de dados consistido dos registros de fam�lia. Resta ainda construir o banco de domic�lios e realizar procedimentos de consist�ncia, comparando-o com os dados de pessoas (a serem analisados na se��o seguinte).

#### Corrigindo os registros de pessoas

Uma vez realizado o diagn�stico manual, carregamos os seus resultados:

``` r
checks_pess <- read_xlsx("Auxiliary Files/check_line_by_line.xlsx", sheet = "persons")
```

E, tal como fizemos anteriormente, executamos uma breve rotina que assinala os casos problem�ticos:

``` r
source("https://raw.githubusercontent.com/antrologos/ConsistenciaCenso1960Br/master/Code/Persons_diagnostics_procedure.R")
```

Aplicamos ent�o o arquivo de layout sobre os registros de pessoas para gerar novamente um banco de dados com separa��o de colunas -- adicionando a vari�vel de diagn�stico, rec�m-criada.

``` r
censo1960_pess <- aplicaTesta_layout(dados_string = c60_string_pess,
                                     input        = input_person) %>%
        mutate(num_linha_pess = num_linha_pess,
               cem_diagnosis_pess = diagnosis) %>% 
        select(num_linha_pess, everything())
```

Marcamos as linhas que contem ao menos um erro:

``` r
censo1960_pess$cem_vars_still_problematic_pess <- censo1960_pess %>% 
        select(starts_with("test_")) %>%  
        mutate_all(function(x) as.numeric(!(x))) %>%
        mutate_all(function(x) ifelse(is.na(x), 0, x)) %>% 
        rowSums(.) 
```

E criamos a coluna adicional que identifica e nomeia todas as variaveis problematicas em cada registro:

``` r
casos_problematicos <- censo1960_pess %>%
        filter(cem_vars_still_problematic_pess > 0) 

teste_problematicos <- casos_problematicos %>%
        select(starts_with("test_"), -test_ID, -test_barra)

casos_problematicos$cem_problematic_vars_list_pess = ""
vars <- names(teste_problematicos)
for(var in vars){
        problematic_cases <- which(!teste_problematicos[[var]])
        
        var_name = gsub(pattern = "test_", replacement = "" , x = var)
        
        casos_problematicos[problematic_cases, "cem_problematic_vars_list_pess"] = 
                paste(casos_problematicos[problematic_cases, ]$cem_problematic_vars_list_pess, var_name)
}

casos_problematicos$cem_problematic_vars_list_pess <- str_trim(casos_problematicos$cem_problematic_vars_list_pess)

censo1960_pess <- left_join(x = censo1960_pess,
                          y = casos_problematicos %>%
                                  select(num_linha_pess, cem_problematic_vars_list_pess),
                          by = "num_linha_pess")
```

Ao fim, os casos corrompidos ter�o marca��es como as do exemplo abaixo:

``` r
censo1960_pess %>%
        filter(cem_problematic_vars_list_pess != "") %>%
        select(num_linha_pess, cem_problematic_vars_list_pess)
```

    ## # A tibble: 81 x 2
    ##    num_linha_pess cem_problematic_vars_list_pess                          
    ##             <int> <chr>                                                   
    ##  1           9817 V207                                                    
    ##  2         162537 V206                                                    
    ##  3         194609 UF V116 REC_TYPE V118 V202 V203 V204 AGE V205 V206 V207~
    ##  4         224345 V208                                                    
    ##  5         228629 V208                                                    
    ##  6         238423 V208                                                    
    ##  7         293555 UF V116 REC_TYPE V118 V202 V203 V204 AGE V205 V206 V207~
    ##  8         293556 V116                                                    
    ##  9         388894 UF V116 REC_TYPE V118 V202 V203 V204 AGE V205 V206 V207~
    ## 10         388895 UF V116 REC_TYPE V118 V202 V203 V204 AGE V205 V206 V207~
    ## # ... with 71 more rows

Substituiremos ent�o os valores invalidos por missing. Uma estrat�gia alternativa a esse passo seria lan�ar m�o de m�todos probabil�sticos de imputa��o de dados faltantes (como a Imputa��o M�ltipla, por exemplo). No entanto, decidimos por deixar a cargo dos futuros usu�rios a avalia��o da necessidade desse procedimento.

``` r
num_linha_pess_prob <- censo1960_pess %>%
        filter(cem_vars_still_problematic_pess >= 1) %>%
        .$num_linha_pess

for(i in num_linha_pess_prob){
        vars_prob <- censo1960_pess[censo1960_pess$num_linha_pess == i, "cem_problematic_vars_list_pess"] %>% 
                unlist() %>%
                strsplit(split = " ") %>%
                unlist()
        
        for(var in vars_prob){
                censo1960_pess[censo1960_pess$num_linha_pess == i,][[var]] <- NA        
        }        
}
```

Compilamos, por fim, uma vers�o "semi-final" do banco de pessoas, ap�s os procedimentos de consist�ncia. Eles dever�o ainda ser combinados com os dados de domic�lios e as informa��es que se repetem nos dois tipos de registro (como UF, Munic�pio e �rea de resid�ncia) dever�o ainda ser contrastadas.

``` r
censo1960_pess_numeric <- censo1960_pess%>%
        select(-starts_with("test_"), - barra) %>% 
        select(-cem_diagnosis_pess, -cem_problematic_vars_list_pess,
               -starts_with("place_")) %>%
        mutate_all(as.numeric)


censo1960_pess_character <- censo1960_pess %>%
        select(cem_diagnosis_pess, cem_problematic_vars_list_pess,
               starts_with("place_"))

censo1960_pess_semifinal <- cbind(censo1960_pess_numeric, censo1960_pess_character) %>% as_tibble()
```

Identificando domic�lios
------------------------

O primeiro passo para a cria��o de um banco de domic�lios a partir dos dados de fam�lias e pessoas � a elabora��o de um ID �nico para cada dom�cilio. Como vimos anterioremente, as vari�veis V101 e V102 s�o essenciais para esse passo, por identificarem os tipos de fam�lia e composi��o dos moradores.

Observamos que dois registros do banco de fam�lias, contudo, n�o possuem informa��o para a V101:

``` r
censo1960_hh_preliminar %>% 
        filter(is.na(V101 == 0)) %>%
        select(ID)
```

    ## # A tibble: 2 x 1
    ##       ID
    ##    <dbl>
    ## 1 155539
    ## 2 165232

O registro com ID 155539 ser� tratado adiante.

A fam�lia com ID 165232, por sua vez, tinha informa��o inv�lida na V101 (valor 0). Em passos anteriores, esse valor foi substitu�do por missing. Alteraremos esse campo para 1 (c�digo referente aos "Domic�lios Particulares �nicos"). Avaliando os casos que precedem e sucedem esse registro, ele n�o parece estar ligado a outros domic�lios (i.e. n�o parece ser uma fam�lia principal, secund�ria ou terci�ria). Al�m disso, a estrutura familiar de seus moradores n�o � indicativa de um domic�lio coletivo.

``` r
censo1960_hh_preliminar <- censo1960_hh_preliminar %>% 
        mutate(V101 = ifelse(ID == 165232, 1, V101))
```

Outra inconsist�ncia que devemos resolver neste momento refere-se � presen�a de um caso de missing para a variavel sobre tipo de registro (que diferenciava pessoas e domic�lios). Esse valor missing era anteriormente inexistente -- foi atribu�do em meio aos procedimentos de consist�ncia, pelo fato de que o registro ao qual se refere estava majoritariamente corrompido. Assumiremos que se trata mesmo de um domic�lio, atribuindo-lhe o valor para a vari�vel em quest�o:

``` r
censo1960_hh_preliminar$REC_TYPE = 1
```

Criamos ent�o variaveis separadas para os IDs de domicilio e de familia, assinalando ainda os tipos de fam�lia encontrados:

``` r
censo1960_hh_preliminar <- censo1960_hh_preliminar %>%
        arrange(ID, V101) %>%
        mutate(cem_IDdomicilio   = ifelse(V101 %in% c(1,2,3), ID, NA),
               cem_tipofamilia   = ifelse(V101 %in% c(1,2), 1, ifelse(V101 %in% c(4,5), 2, NA)),
               n_linha = 1:n())
```

A codifica��o da vari�vel "cem\_tipofamilia" ser� a seguinte:

-   **1** - Familia principal ou unica
-   **2** - Outras familias
-   **NA** - Nao se aplica (domicilios coletivos)

As linhas sem valores v�lidos para a vari�vel cem\_IDdomicilio devem, deste modo, dizer respeito a fam�lias conviventes. Por defini��o, esses registros devem reecber o mesmo ID de domic�lios das fam�lias principais com as quais co-habitam. Para isso, primeiramente, selecionamos esses casos:

``` r
linhas_familias <- censo1960_hh_preliminar %>%
        filter(is.na(cem_IDdomicilio)) %>% 
        .$n_linha
```

Ent�o, para as familias secundarias, copiamos a informa��o sobre ID de Domic�lio do caso anterior (que deve ser, necessariamente, uma fam�lia principal):

``` r
for(linha_familia in linhas_familias){
        caso_anterior <- censo1960_hh_preliminar[(linha_familia - 1) , "cem_tipofamilia"] == 1 
        dim(caso_anterior) <- NULL
        
        if( caso_anterior & !is.na(caso_anterior)){
                censo1960_hh_preliminar[linha_familia , "cem_IDdomicilio"] <- censo1960_hh_preliminar[(linha_familia-1) , "cem_IDdomicilio"]
        }
}
```

Repetimos o procedimento para as familias terciarias, copiando a informa��o do caso anterior -- mas agora, apenas se o caso anterior for o de uma fam�lia secund�ria:

``` r
linhas_familias = which(is.na(censo1960_hh_preliminar$cem_IDdomicilio))
for(linha_familia in linhas_familias){
        caso_anterior <- censo1960_hh_preliminar[(linha_familia - 1) , "V101"] == 4
        proprio_caso  <- censo1960_hh_preliminar[(linha_familia ) , "V101"] == 5
                
        dim(caso_anterior) <- NULL
        dim(proprio_caso) <- NULL
        
        if( caso_anterior & proprio_caso & !is.na(caso_anterior) & !is.na(proprio_caso)){
                censo1960_hh_preliminar[linha_familia , "cem_IDdomicilio"] <- censo1960_hh_preliminar[(linha_familia-1) , "cem_IDdomicilio"]
        }
}
```

Por fim, a antiga vari�vel ID converte-se agora em ID apenas das fam�lias. E j� podemos apagar a vari�vel auxiliar "n\_linha", criada para este passo:

``` r
censo1960_hh_preliminar <- censo1960_hh_preliminar %>%
        rename(cem_IDfamilia = ID)
censo1960_hh_preliminar$n_linha = NULL
```

Examinado agora o registro com ID de fam�lia igual a 155539, o que observamos � que ele parece constituir um domicilio autonomo, n�o faz parte do domicilio 155537, que o precede. Trata-se, na realidade, de um registro de domicilio com dados corrompidos. Mais adiante, quando constrastarmos suas informa��es com aquelas existentes nos registros das pessoas que nele habitam, detectaremos a exist�ncia de discrep�ncias. Ainda assim, devemos mant�-lo no banco de dados. Neste passo, � preciso separ�-lo do domic�lio anterior:

``` r
censo1960_hh_preliminar <- censo1960_hh_preliminar %>%
        mutate(cem_IDdomicilio = ifelse(cem_IDfamilia == 155539, 155539, cem_IDdomicilio),
               V101            = ifelse(cem_IDfamilia == 155539, 1, V101))
```

Se identificamos corretamente os domic�lios, n�o deve haver varia��o de suas caracter�sticas estruturais entre as fam�lias conviventes. Ou seja: fam�lias que moram juntas n�o devem apresentar informa��es discrepantes com respeito ao n�mero de c�modos, presen�a de energia el�trica, �gua etc. Testamos isso por meio do procedimento abaixo. A vari�ncia dessas caracter�sticas deve ser igual a zero:

``` r
test = censo1960_hh_preliminar %>%
        select(-num_linha_dom, -V101, -cem_IDfamilia, -cem_vars_still_problematic_dom,
               -cem_diagnosis_dom, -cem_problematic_vars_list_dom, -cem_tipofamilia) %>%
        select_if(function(x) !is.character(x)) %>%
        group_by(cem_IDdomicilio) %>%
        summarise_all(stats::var, na.rm= T)
        
domicilios_problema = NULL
for(i in 2:ncol(test)){
        domicilios_problema <- c(domicilios_problema, test[which(test[,i] > 0 & !is.na(test[,2])),"cem_IDdomicilio"] %>% unlist())
}

domicilios_problema <- domicilios_problema %>% unique() 
```

Nenhum problema foi encontrado: n�o h� domicilios que apresentem varia��o de caracteristicas entre familias:

``` r
censo1960_hh_preliminar %>%
        filter(cem_IDdomicilio %in% domicilios_problema) 
```

Temos ent�o um banco de fam�lias, com marca��es dos IDs dos domic�lios aos quais pertencem.

Importando IDs de domic�lio para o banco de pessoas
---------------------------------------------------

No banco de fam�lias, devemos ent�o checar se o n�mero de linhas � identico ao n�mero de IDs:

``` r
length(censo1960_hh_preliminar$cem_IDfamilia)
## [1] 174467
length(censo1960_hh_preliminar$cem_IDfamilia %>% unique()) #ok
## [1] 174467
```

No banco de pessoas, checamos se o n�mero de IDs de familia � identico ao n�mero de linhas no banco de fam�lias:

``` r
censo1960_pess_semifinal <- censo1960_pess_semifinal %>%
        rename(cem_IDfamilia = ID)
length(censo1960_pess_semifinal$cem_IDfamilia %>% unique()) # ok
## [1] 174467
```

Por fim, verificamos se todos os valores listados como IDs de familia no arquivo de pessoas est�o de fato contemplados no arquivo de fam�lias. E n�o encontramos nenhuma discrep�ncia:

``` r
sum( !(censo1960_pess_semifinal$cem_IDfamilia %in% censo1960_hh_preliminar$cem_IDfamilia) ) # ok!
```

    ## [1] 0

Como o pareamento existe, podemos trazer a informa��o rec�m-criada sobre ID de domic�lio para os registros de indiv�duos:

``` r
censo1960_pess_semifinal <- left_join(x = censo1960_pess_semifinal,
                                  y = censo1960_hh_preliminar %>%
                                          select(cem_IDdomicilio,cem_IDfamilia, V101),
                                  by = "cem_IDfamilia")
```

Construindo um banco de domic�lios
----------------------------------

Para construir um banco de domic�lios daquele sobre fam�lias, devemos remover os registros de familias secundarias e terciarias, mantendo apenas as unicas, principais e os domicilios coletivos. Podemos tamb�m remover algumas vari�veis desnecess�rias:

``` r
censo1960_hh_semifinal = censo1960_hh_preliminar %>%
        filter(!(V101 %in% c(4,5))) %>%  
        select(-cem_tipofamilia, -cem_IDfamilia, -REC_TYPE) 
```

Problemas de grava��o em dois registros adjacentes voltam agora a ser relevantes. Trata-se das linhas 293555 e 293556 do arquivo original:

1.  `31   M       1BA33HIA                                 \0046690`
2.  `31 1 80731608050352911557059200031100000000034        \0046690`

Visualisando esses dois registros dentro de um contexto maior, observamos que est�o justamente na fronteira entre dois estados: Sergipe (UF=30) e Bahia (UF=31). Ent�o, no entanto, sendo atribuidos ao domicilio de Sergipe dos casos que os antecedem: de ID=0046690. H�, deste modo, uma constradi��o entre a informa��o do ID e a da UF.

``` r
writeLines(c60_string[293552:293560])
```

    ## 3030950130378219151478389680203001                    \0046690
    ## 303095013037821925271195427920003110010001015393325137\0046690
    ## 30309501303782193529105542792000311                   \0046690
    ## 31   M       1BA33HIA                                 \0046690
    ## 31 1 80731608050352911557059200031100000000034        \0046690
    ## 3131110131344001131578380680208004                    \0046691
    ## 313111013134400123171605705920003110062312096366423157\0046691
    ## 3131110131344001332815654059200031100623120934        \0046691
    ## 3131110131344001331912657059200015200000000031        \0046691

A palavra "BAHIA", presente no registro 293555, � forte evid�ncia de que se trataria, de fato, de um caso daquela UF. Devemos ent�o estabelecer uma regras de desambigua��o desses dois registros -- **isso afetar� o n�mero de casos �nicos de domic�lios e fam�lias**. Mas uma observa��o pode facilitar a decis�o. Os IDs de fam�lia (caracteres nas posi��es 56-62) parecem ter sido criados a posteriori, n�o sendo dados originais. Se isso for verdade, podem conter erros. A numeracao daqueles IDs parece seguir sempre uma mesma l�gica: seguindo a ordem dos casos no banco de dados, inicia-se um novo valor que o caracteres da posi��o 17 assume o valor 1 (que seria indicativo de fam�lia/domic�lio). Como, nesse caso, o registro da linha 293555 est� corrompido, n�o teria havido uma nova contagem do ID. Provavelmente, por esta raz�o, o numero do domicilio continuou o mesmo dos registros anteriores.

Para iniciar um novo ID temos ent�o duas sa�das:

1.  Assumir que o registro da linha 293555 � ele mesmo um domic�lio (o que poderia ser feito com `substr(c60_string[293555], 17, 17) <- "1")`.
2.  Criar uma nova linha no banco de dados, representando o domic�lio faltante. Com isso, os registros corrompidos seriam entendidos como "pessoas".

Ambas as solu��es envolvem modifica��o dr�stica dos dados originais -- o que n�o parece ser recomendado. No primeiro caso, al�m disso, o registro da linha 293556 seria compreendido como um indiv�duo morando sozinho. Se as demais informa��es da linha forem consideradas, isso introduziria outros problemas: trataria-se de uma pessoa na posi��o de "filho" (v203=9, caractere 20), com apenas 15 anos de idade(caracteres 22 e 23) -- o que introduziria uma nova inconsist�ncia. Essas duas solu��es parecem ser por demais radicais. **Por esta raz�o, decidimos n�o proceder nenhuma altera��o, mantendo a inconsist�ncia original**.

Imputa��o de valores faltantes
------------------------------

H� vari�veis que se repetem tanto nos registros de pessoas e domic�lios: UF, V116 (Munic�pio) e V118 (�rea de resid�ncia, se rural ou urbana). Esse fato permite que, nos casos de informa��o faltante ou corrompida, seja poss�vel copiar o dado do outro tipo de registro. Iniciamos ent�o pela importa��o dos dados de domic�lios para os registros de pessoas:

``` r
censo1960_pess_semifinal <- left_join(x = censo1960_pess_semifinal, 
                                  y = censo1960_hh_semifinal %>%
                                          select(-V101),  
                                  by = "cem_IDdomicilio") %>%
        rename(UF_pess   = UF.x,
               UF_dom    = UF.y,
               V116_pess = V116.x,
               V116_dom  = V116.y,
               V118_pess = V118.x,
               V118_dom  = V118.y)
```

Identificamos agora os casos em que h� missing apenas numa das fontes de informacao (ou pessoas ou domicilios):

``` r
censo1960_pess_semifinal <- censo1960_pess_semifinal %>%
        mutate(cem_UF_missingDom_notmissingPerson   = as.numeric(is.na(UF_dom)    & !is.na(UF_pess)  ),
               cem_UF_missingPerson_notmissingDom   = as.numeric(is.na(UF_pess)   & !is.na(UF_dom)   ),
               cem_V116_missingDom_notmissingPerson = as.numeric(is.na(V116_dom)  & !is.na(V116_pess)),
               cem_V116_missingPerson_notmissingDom = as.numeric(is.na(V116_pess) & !is.na(V116_dom) ),
               cem_V118_missingDom_notmissingPerson = as.numeric(is.na(V118_dom)  & !is.na(V118_pess)),
               cem_V118_missingPerson_notmissingDom = as.numeric(is.na(V118_pess) & !is.na(V118_dom))
               )
```

Iniciamos o procedimento de imputa��o pela c�pia de informa��es: quando h� missing na vari�vel de domic�lio, lhe atribu�mos o valor observado no registro de pessoa e vice-versa:

``` r
censo1960_pess_semifinal <- censo1960_pess_semifinal %>% 
        mutate(UF_dom  = ifelse(cem_UF_missingDom_notmissingPerson == 1, UF_pess, UF_dom),
               UF_pess = ifelse(cem_UF_missingPerson_notmissingDom == 1, UF_dom,  UF_pess),
               
               V116_dom  = ifelse(cem_V116_missingDom_notmissingPerson == 1, V116_pess, V116_dom),
               V116_pess = ifelse(cem_V116_missingPerson_notmissingDom == 1, V116_dom,  V116_pess),
               
               V118_dom  = ifelse(cem_V118_missingDom_notmissingPerson == 1, V118_pess, V118_dom),
               V118_pess = ifelse(cem_V118_missingPerson_notmissingDom == 1, V118_dom,  V118_pess)
               )
```

Deletando ent�o as variaveis temporarias utilizadas nesse procedimento:

``` r
censo1960_pess_semifinal$cem_UF_missingDom_notmissingPerson   = NULL
censo1960_pess_semifinal$cem_UF_missingPerson_notmissingDom   = NULL
censo1960_pess_semifinal$cem_V116_missingDom_notmissingPerson = NULL
censo1960_pess_semifinal$cem_V116_missingPerson_notmissingDom = NULL
censo1960_pess_semifinal$cem_V118_missingDom_notmissingPerson = NULL
censo1960_pess_semifinal$cem_V118_missingPerson_notmissingDom = NULL
```

Podemos ainda atribuir valores para pessoas com missing, tomando como doadores as informa��es de outroso indiv�duos residentes no mesmo domic�lio.

#### Identificando casos de missing na variavel de UF

``` r
censo1960_pess_semifinal = data.table(censo1960_pess_semifinal)
censo1960_pess_semifinal[ , num_NA_UF_dom  := sum(is.na(UF_dom)),  by = cem_IDfamilia]
censo1960_pess_semifinal[ , num_NA_UF_pess := sum(is.na(UF_pess)), by = cem_IDfamilia]
censo1960_pess_semifinal[ num_NA_UF_dom > 0  | num_NA_UF_pess > 0 , ]
```

    ##    num_linha_pess UF_pess V116_pess REC_TYPE V118_pess V202 V203 V204 AGE
    ## 1:         951431      NA        NA       NA        NA   NA   NA   NA  NA
    ## 2:         951432      NA        NA       NA        NA   NA   NA   NA  NA
    ## 3:         951433      NA        NA       NA        NA   NA   NA   NA  NA
    ## 4:         951434      81      8006        3         5    1    9    1   3
    ## 5:         951435      81      8006        3         5    2    9    1   6
    ##    V205 V206 V207 V208 V209 V299 V210 V211 V212 V213 V214 V215 V216 V217
    ## 1:   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA
    ## 2:   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA
    ## 3:   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA
    ## 4:    5    4   24    9    2    0    0   NA   NA   NA   NA   NA   NA   NA
    ## 5:    5    7   24    9    2    0    0    3    1    1   NA   NA   NA   NA
    ##    V218 V219 V220 V221 V223 V223B V224 cem_IDfamilia
    ## 1:   NA   NA   NA   NA   NA    NA   NA        155539
    ## 2:   NA   NA   NA   NA   NA    NA   NA        155539
    ## 3:   NA   NA   NA   NA   NA    NA   NA        155539
    ## 4:   NA   NA   NA   NA   NA    NA   NA        155539
    ## 5:   NA   NA   NA   NA   NA    NA   NA        155539
    ##    cem_vars_still_problematic_pess cem_diagnosis_pess
    ## 1:                              15                  4
    ## 2:                              15                  4
    ## 3:                              15                  4
    ## 4:                               0                  3
    ## 5:                               0                  3
    ##                                                 cem_problematic_vars_list_pess
    ## 1: UF V116 REC_TYPE V118 V202 V203 V204 AGE V205 V206 V207 V208 V209 V299 V210
    ## 2: UF V116 REC_TYPE V118 V202 V203 V204 AGE V205 V206 V207 V208 V209 V299 V210
    ## 3: UF V116 REC_TYPE V118 V202 V203 V204 AGE V205 V206 V207 V208 V209 V299 V210
    ## 4:                                                                        <NA>
    ## 5:                                                                        <NA>
    ##    place_holder_07_08.x place_holder_09_16.x cem_IDdomicilio V101
    ## 1:                                                    155539    1
    ## 2:                                                    155539    1
    ## 3:                                                    155539    1
    ## 4:                   05             82402101          155539    1
    ## 5:                   05             82402103          155539    1
    ##    num_linha_dom UF_dom V116_dom V118_dom V102 V103 V104 V105 V106 V107
    ## 1:        951430     NA       NA       NA   NA   NA   NA   NA   NA   NA
    ## 2:        951430     NA       NA       NA   NA   NA   NA   NA   NA   NA
    ## 3:        951430     NA       NA       NA   NA   NA   NA   NA   NA   NA
    ## 4:        951430     81     8006        5   NA   NA   NA   NA   NA   NA
    ## 5:        951430     81     8006        5   NA   NA   NA   NA   NA   NA
    ##    V108 V109 V110 V111 V112 V113 cem_vars_still_problematic_dom
    ## 1:   NA   NA   NA   NA   NA   NA                              5
    ## 2:   NA   NA   NA   NA   NA   NA                              5
    ## 3:   NA   NA   NA   NA   NA   NA                              5
    ## 4:   NA   NA   NA   NA   NA   NA                              5
    ## 5:   NA   NA   NA   NA   NA   NA                              5
    ##    cem_diagnosis_dom cem_problematic_vars_list_dom place_holder_07_08.y
    ## 1:                 4    UF V116 REC_TYPE V118 V101                     
    ## 2:                 4    UF V116 REC_TYPE V118 V101                     
    ## 3:                 4    UF V116 REC_TYPE V118 V101                     
    ## 4:                 4    UF V116 REC_TYPE V118 V101                     
    ## 5:                 4    UF V116 REC_TYPE V118 V101                     
    ##    place_holder_09_16.y place_holder_32_32   place_holder_35_54
    ## 1:                                                             
    ## 2:                                                             
    ## 3:                                                             
    ## 4:                                                             
    ## 5:                                                             
    ##    num_NA_UF_dom num_NA_UF_pess
    ## 1:             3              3
    ## 2:             3              3
    ## 3:             3              3
    ## 4:             3              3
    ## 5:             3              3

O domic�lio/fam�lia 155539 era um caso de registro corrompido para a variavel UF. Lan�amos m�o, ent�o de informa��es advindas de casos doadores:

``` r
censo1960_pess_semifinal[cem_IDfamilia == 155539, UF_pess := 81] 
censo1960_pess_semifinal[cem_IDfamilia == 155539, UF_dom  := 81] 
censo1960_pess_semifinal[cem_IDfamilia == 155539, V116_pess := 8006] 
censo1960_pess_semifinal[cem_IDfamilia == 155539, V116_dom  := 8006] 
censo1960_pess_semifinal[cem_IDfamilia == 155539, V118_pess := 5] 
censo1960_pess_semifinal[cem_IDfamilia == 155539, V118_dom  := 5] 
censo1960_pess_semifinal[cem_IDfamilia == 155539, V101 := 1] 
censo1960_pess_semifinal[cem_IDfamilia == 155539]
```

    ##    num_linha_pess UF_pess V116_pess REC_TYPE V118_pess V202 V203 V204 AGE
    ## 1:         951431      81      8006       NA         5   NA   NA   NA  NA
    ## 2:         951432      81      8006       NA         5   NA   NA   NA  NA
    ## 3:         951433      81      8006       NA         5   NA   NA   NA  NA
    ## 4:         951434      81      8006        3         5    1    9    1   3
    ## 5:         951435      81      8006        3         5    2    9    1   6
    ##    V205 V206 V207 V208 V209 V299 V210 V211 V212 V213 V214 V215 V216 V217
    ## 1:   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA
    ## 2:   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA
    ## 3:   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA   NA
    ## 4:    5    4   24    9    2    0    0   NA   NA   NA   NA   NA   NA   NA
    ## 5:    5    7   24    9    2    0    0    3    1    1   NA   NA   NA   NA
    ##    V218 V219 V220 V221 V223 V223B V224 cem_IDfamilia
    ## 1:   NA   NA   NA   NA   NA    NA   NA        155539
    ## 2:   NA   NA   NA   NA   NA    NA   NA        155539
    ## 3:   NA   NA   NA   NA   NA    NA   NA        155539
    ## 4:   NA   NA   NA   NA   NA    NA   NA        155539
    ## 5:   NA   NA   NA   NA   NA    NA   NA        155539
    ##    cem_vars_still_problematic_pess cem_diagnosis_pess
    ## 1:                              15                  4
    ## 2:                              15                  4
    ## 3:                              15                  4
    ## 4:                               0                  3
    ## 5:                               0                  3
    ##                                                 cem_problematic_vars_list_pess
    ## 1: UF V116 REC_TYPE V118 V202 V203 V204 AGE V205 V206 V207 V208 V209 V299 V210
    ## 2: UF V116 REC_TYPE V118 V202 V203 V204 AGE V205 V206 V207 V208 V209 V299 V210
    ## 3: UF V116 REC_TYPE V118 V202 V203 V204 AGE V205 V206 V207 V208 V209 V299 V210
    ## 4:                                                                        <NA>
    ## 5:                                                                        <NA>
    ##    place_holder_07_08.x place_holder_09_16.x cem_IDdomicilio V101
    ## 1:                                                    155539    1
    ## 2:                                                    155539    1
    ## 3:                                                    155539    1
    ## 4:                   05             82402101          155539    1
    ## 5:                   05             82402103          155539    1
    ##    num_linha_dom UF_dom V116_dom V118_dom V102 V103 V104 V105 V106 V107
    ## 1:        951430     81     8006        5   NA   NA   NA   NA   NA   NA
    ## 2:        951430     81     8006        5   NA   NA   NA   NA   NA   NA
    ## 3:        951430     81     8006        5   NA   NA   NA   NA   NA   NA
    ## 4:        951430     81     8006        5   NA   NA   NA   NA   NA   NA
    ## 5:        951430     81     8006        5   NA   NA   NA   NA   NA   NA
    ##    V108 V109 V110 V111 V112 V113 cem_vars_still_problematic_dom
    ## 1:   NA   NA   NA   NA   NA   NA                              5
    ## 2:   NA   NA   NA   NA   NA   NA                              5
    ## 3:   NA   NA   NA   NA   NA   NA                              5
    ## 4:   NA   NA   NA   NA   NA   NA                              5
    ## 5:   NA   NA   NA   NA   NA   NA                              5
    ##    cem_diagnosis_dom cem_problematic_vars_list_dom place_holder_07_08.y
    ## 1:                 4    UF V116 REC_TYPE V118 V101                     
    ## 2:                 4    UF V116 REC_TYPE V118 V101                     
    ## 3:                 4    UF V116 REC_TYPE V118 V101                     
    ## 4:                 4    UF V116 REC_TYPE V118 V101                     
    ## 5:                 4    UF V116 REC_TYPE V118 V101                     
    ##    place_holder_09_16.y place_holder_32_32   place_holder_35_54
    ## 1:                                                             
    ## 2:                                                             
    ## 3:                                                             
    ## 4:                                                             
    ## 5:                                                             
    ##    num_NA_UF_dom num_NA_UF_pess
    ## 1:             3              3
    ## 2:             3              3
    ## 3:             3              3
    ## 4:             3              3
    ## 5:             3              3

#### Identificando casos de missing na variavel v116 \*\*

O domic�lio/fam�lia 145606, com informa��o faltante na vari�vel v116 n�o poder� ser corrigido. Trata-se de uma residencia com um �nico morados. Logo, n�o � poss�vel encontrar doadores.

``` r
censo1960_pess_semifinal[ , num_NA_V116_dom  := sum(is.na(V116_dom)),  by = cem_IDfamilia]
censo1960_pess_semifinal[ , num_NA_V116_pess := sum(is.na(V116_pess)), by = cem_IDfamilia]
censo1960_pess_semifinal[ num_NA_V116_dom > 0  | num_NA_V116_pess > 0 ]
```

    ##    num_linha_pess UF_pess V116_pess REC_TYPE V118_pess V202 V203 V204 AGE
    ## 1:         887256      71        NA        2         5    1    7    1  21
    ##    V205 V206 V207 V208 V209 V299 V210 V211 V212 V213 V214 V215 V216 V217
    ## 1:    5    4   19    9    2    0    0    3    1    1    0    9   60    0
    ##    V218 V219 V220 V221 V223 V223B V224 cem_IDfamilia
    ## 1:    0    5    3  323    2   120    7        145606
    ##    cem_vars_still_problematic_pess cem_diagnosis_pess
    ## 1:                               1                  2
    ##    cem_problematic_vars_list_pess place_holder_07_08.x
    ## 1:                           V116                   07
    ##    place_holder_09_16.x cem_IDdomicilio V101 num_linha_dom UF_dom V116_dom
    ## 1:             71038216          145606    1        887255     71       NA
    ##    V118_dom V102 V103 V104 V105 V106 V107 V108 V109 V110 V111 V112 V113
    ## 1:        5    5    9    8    2    7    9    6    8    0    2    2    1
    ##    cem_vars_still_problematic_dom cem_diagnosis_dom
    ## 1:                              1                 2
    ##    cem_problematic_vars_list_dom place_holder_07_08.y place_holder_09_16.y
    ## 1:                          V116                   07             71038216
    ##    place_holder_32_32   place_holder_35_54 num_NA_UF_dom num_NA_UF_pess
    ## 1:                  0                                  0              0
    ##    num_NA_V116_dom num_NA_V116_pess
    ## 1:               1                1

\*\* Identificando casos de missing na variavel v118\*\*

N�o h� necessidade de registros doadores para a vari�vel V118:

``` r
censo1960_pess_semifinal[ , num_NA_V118_dom  := sum(is.na(V118_dom)),  by = cem_IDfamilia]
censo1960_pess_semifinal[ , num_NA_V118_pess := sum(is.na(V118_pess)), by = cem_IDfamilia]
censo1960_pess_semifinal[ num_NA_V118_dom > 0  | num_NA_V118_pess > 0 ]
```

    ## Empty data.table (0 rows) of 67 cols: num_linha_pess,UF_pess,V116_pess,REC_TYPE,V118_pess,V202...

Inconsist�ncias entre pessoas e domic�lios
------------------------------------------

H� inconsist�ncias incontorn�veis entre o restante dos dados de domic�lios e pessoas: informa��es discrepantes sobre UF de resid�ncia, munic�pio e local de moradia (rural/urbano). Assinalaremos esses registros com v�ri�veis adicionais, contru�das para esse fim:

``` r
# Transformado o objeto de volta em tibble
censo1960_pess_semifinal = as_tibble(censo1960_pess_semifinal) %>%
        select(-starts_with("num_NA_"))

censo1960_pess_semifinal$cem_dissonant_UF   = as.numeric(!(censo1960_pess_semifinal$UF_pess   == censo1960_pess_semifinal$UF_dom))
censo1960_pess_semifinal$cem_dissonant_V116 = as.numeric(!(censo1960_pess_semifinal$V116_pess == censo1960_pess_semifinal$V116_dom))
censo1960_pess_semifinal$cem_dissonant_V118 = as.numeric(!(censo1960_pess_semifinal$V118_pess == censo1960_pess_semifinal$V118_dom))
```

Casos que permanecem problem�ticos na vari�vel UF (3 registros):

``` r
censo1960_pess_semifinal %>%
        filter((cem_dissonant_UF %in% 1   | is.na(cem_dissonant_UF)  )) %>%
        nrow()
```

    ## [1] 3

Casos que permanecem problem�ticos na vari�vel v116 (44 registros):

``` r
censo1960_pess_semifinal %>%
        filter( (cem_dissonant_V116 == 1 | is.na(cem_dissonant_V116)) ) %>% 
        nrow() 
```

    ## [1] 44

Casos que permanecem problem�ticos na vari�vel v118 (280 registros):

``` r
censo1960_pess_semifinal %>%
        filter((cem_dissonant_V118 == 1 | is.na(cem_dissonant_V118))) %>%
        nrow() 
```

    ## [1] 280

Total de registros problem�ticos que permanecem no banco: 299.

``` r
censo1960_pess_semifinal %>%
        filter((cem_dissonant_UF == 1   | is.na(cem_dissonant_UF) ) |
               (cem_dissonant_V116 == 1 | is.na(cem_dissonant_V116))|
               (cem_dissonant_V118 == 1 | is.na(cem_dissonant_V118))) %>%
        nrow()
```

    ## [1] 299

Re-criando o banco de domic�lios (ap�s as imputa��es) a partir dos dados de pessoas
-----------------------------------------------------------------------------------

No banco de pessoas, criamos algumas vari�veis adicionais: um identificador de moradores (que exclui pessoas listadas como "n�o moradores presentes") e o peso dos indiv�duos (lan�ando m�o das informa��es sobre a fra��o amostral). Essas informa��es ser�o usadas para produzir vari�veis sobre domic�lios: n�mero de pessoas em cada domic�lio e o pr�prio peso dos domic�lios. Calculamos tamb�m o n�mero de fam�lias por domic�lio (lembrando que, por defini��o, o Censo de 1960 considera como domic�lios coletivos aqueles com 4 ou mais fam�lias conviventes).

``` r
censo1960_pess_semifinal <- censo1960_pess_semifinal %>%
        mutate(ind_morador = as.numeric(V203 %in% c(0:3, 7:9)),
               ind_morador = ifelse(is.na(ind_morador), 0, ind_morador),
               cem_wgt = 1/0.0127) %>%
        
        group_by(cem_IDdomicilio) %>%
        mutate(cem_num_moradores = n(),
               cem_num_familias  = length(unique(cem_IDfamilia)), 
               cem_num_familias  = ifelse(V101 == 3, NA, cem_num_familias), 
               cem_num_pessoas   = sum(ind_morador),              
               ind_morador = NULL) %>%
        ungroup()
```

Procedemos ent�o a agrega��o das informa��es das pessoas por IDs de domic�lios. Com essa opera��o, re-criamos os registros de domic�lios a partir da sumariza��o das informa��es individuais. Obviamente, apenas utilizamos as vari�veis cab�veis, que n�o apresentam varia��es entre moradores de uma mesma resid�ncia -- aquelas iniciadas por "V1", UF e aquelas criadas acima, nos procedimentos anteriores.

Como procedimento para agrega��o dos casos, calcularemos o valor m�ximo de cada vari�vel dentro do domic�lio. Trata-se de uma opera��o arbitr�ria: como todos os indiv�duos que co-habitam apresentam necessariamente valores id�nticos, poder�amos ter calculado a m�dia, o m�nimo, a mediana ou qualquer outra medida. No entanto, todas essas se aplicam apenas aos dados num�ricos. Aplicaremos um procedimento de sumariza��o para as variaveis de tipo character separadamente.

``` r
censo1960_hh_final <- censo1960_pess_semifinal %>%
        select(UF_dom,  starts_with("V1"),
               ends_with("_dom"), -ends_with("pess"), 
               cem_IDdomicilio,
               cem_num_pessoas, cem_num_moradores, cem_num_familias, cem_wgt,
               cem_dissonant_UF,cem_dissonant_V116, cem_dissonant_V118, 
               -cem_vars_still_problematic_dom, -num_linha_dom, -cem_problematic_vars_list_dom) %>%
        group_by(cem_IDdomicilio) %>%
        summarise_all(max_sem_na) %>%
        ungroup() %>%
        mutate_all(replace_infinite) %>%
        rename(UF   = UF_dom,
               V116 = V116_dom, 
               V118 = V118_dom)
```

No passo a seguir, sumarizamos apenas a vari�vel character faltante:

``` r
censo1960_hh_tmp <- censo1960_pess_semifinal %>% 
        select(cem_IDdomicilio, cem_problematic_vars_list_dom) %>%
        group_by(cem_IDdomicilio) %>%
        summarise(cem_problematic_vars_list_dom = first(cem_problematic_vars_list_dom))
```

A partir dos IDs de domic�lio, fundimos os dois bancos-sum�rio:

``` r
censo1960_hh_final <- censo1960_hh_final %>%
        left_join(x  = .,
                  y  = censo1960_hh_tmp,
                  by = "cem_IDdomicilio")
names(censo1960_hh_final) = tolower(names(censo1960_hh_final))
```

Por fim, re-ordenamos as vari�veis e eliminamos da lista da vari�veis problem�ticas a coluna "REC\_TYPE" (que j� n�o consta mais no banco de dados):

``` r
censo1960_hh_final <- censo1960_hh_final %>%
        select(uf, v116, v118, cem_iddomicilio, 
               v101, v102, cem_wgt,
               v103, v104, v105, v106, v107, v108,
               v109, v110, v111, v112, v113, 
               cem_num_pessoas, cem_num_moradores,
               cem_num_familias, cem_diagnosis_dom,
               cem_problematic_vars_list_dom,
               cem_dissonant_uf, cem_dissonant_v116,
               cem_dissonant_v118)

censo1960_hh_final <- censo1960_hh_final %>%
        mutate(cem_problematic_vars_list_dom = str_replace_all(string = cem_problematic_vars_list_dom,
                                                               pattern = "REC_TYPE",
                                                               replacement = "") %>%
                       str_replace_all(pattern = "  ",
                                       replacement = " ") %>% 
                       tolower()
        )
```

Obtemos ent�o a **vers�o final do banco de domic�lios** e a salvamos em disco:

``` r
data_address <- "Data - After Consistency/Censo.1960.brasil.domicilios.amostra.1.27porcento.csv"

fwrite(x = censo1960_hh_final,   
          file = data_address,
          na="",
          row.names = F,
          quote = TRUE)

zip::zip(zipfile = 'Data - After Consistency/Censo.1960.brasil.domicilios.amostra.1.27porcento.zip', 
    files   = data_address)

file.remove(data_address)
```

    ## [1] TRUE

A lista de vari�veis contidas no arquivo de dados, ao fim, � a seguinte:

``` r
names(censo1960_hh_final)
```

    ##  [1] "uf"                            "v116"                         
    ##  [3] "v118"                          "cem_iddomicilio"              
    ##  [5] "v101"                          "v102"                         
    ##  [7] "cem_wgt"                       "v103"                         
    ##  [9] "v104"                          "v105"                         
    ## [11] "v106"                          "v107"                         
    ## [13] "v108"                          "v109"                         
    ## [15] "v110"                          "v111"                         
    ## [17] "v112"                          "v113"                         
    ## [19] "cem_num_pessoas"               "cem_num_moradores"            
    ## [21] "cem_num_familias"              "cem_diagnosis_dom"            
    ## [23] "cem_problematic_vars_list_dom" "cem_dissonant_uf"             
    ## [25] "cem_dissonant_v116"            "cem_dissonant_v118"

O banco de pessoas
------------------

O banco de pessoas, neste ponto, apenas requer alguns ajustes finais: renomear vari�veis, excluir colunas auxiliares e alguma re-ordena��o. Iniciamos por fazer com que os nomes de todas as vari�veis sejam escritos apenas com letras min�sculas, como fizemos no banco de domic�lios:

``` r
names(censo1960_pess_semifinal) = tolower(names(censo1960_pess_semifinal))
```

Mantemos ent�o no banco de dados apenas as vari�veis finais, excluindo aquelas utilizadas nos passos intermedi�rios da consist�ncia. Renomeamos tamb�m a vari�vel idade, anteriormente denominada "age", para v204b. No dicion�rio de c�digos o nome v204 est� reservado para a vari�vel "Tipo de Idade" (se em meses ou em anos). O nome "age" era o que constava nas sintaxes recebidas juntamente com o arquivo de dados original.A decis�o de denomin�-la v204b tem como objetivo manter o padr�o de nomea��o usual do IBGE e seguir a mesma l�gica da vari�vel referente aos setores de atividade econ�mica, denominada v223b, que tamb�m n�o possuia c�digo pr�prio (no dicion�rio, o c�digo v223 � usado para descrever a vari�vel que capta a ocupa��o na �ltima semana, como o setor de atividade).

``` r
censo1960_pess_final <- censo1960_pess_semifinal %>%
        rename(cem_idindividuo = num_linha_pess,
               v204b = age) %>% 
        select(uf_pess, uf_dom, 
               v116_pess, v116_dom, 
               v118_pess, v118_dom,
               starts_with("v"),
               starts_with("cem_id"),
               starts_with("c"),
               everything(),
               -rec_type, 
               -starts_with("place"),
               -num_linha_dom,
               -cem_vars_still_problematic_dom, -cem_vars_still_problematic_pess) %>% 
        select(starts_with("u"), starts_with("v"), everything()) 
```

Reordenamos ent�o as vari�veis do banco de dados:

``` r
censo1960_pess_final <- censo1960_pess_final %>%
        select(uf_pess,uf_dom,v116_pess,v116_dom,v118_pess,v118_dom,
               cem_idindividuo,cem_idfamilia,cem_iddomicilio,cem_wgt,
               v202,v203,v204,v204b,v205,v206,v207,v208,v209,v299,
               v210,v211,v212,v213,v214,v215,v216,v217,v218,v219,
               v220,v221,v223,v223b,v224,v101,v102,v103,v104,v105,
               v106,v107,v108,v109,v110,v111,v112,v113,
               cem_num_pessoas,cem_num_moradores,cem_num_familias,
               cem_diagnosis_pess,cem_problematic_vars_list_pess,cem_diagnosis_dom,
               cem_problematic_vars_list_dom,cem_dissonant_uf,cem_dissonant_v116,
               cem_dissonant_v118)
```

Por fim, substitu�mos o nome "age" por "v204b", onde houver, na coluna que indica as vari�veis problem�ticas, dos registros com informa��es corrompidas.

``` r
censo1960_pess_final <- censo1960_pess_final %>% 
        mutate(cem_problematic_vars_list_pess = str_replace_all(string = cem_problematic_vars_list_pess,
                                                                pattern = "AGE",
                                                                replacement = "v204b") %>%
                       str_replace_all(pattern = "REC_TYPE",
                                       replacement = "") %>%
                       str_replace_all(pattern = "  ",
                                       replacement = " ") %>% 
                       tolower(),
               cem_problematic_vars_list_dom = str_replace_all(string = cem_problematic_vars_list_dom,
                                                               pattern = "REC_TYPE",
                                                               replacement = "") %>%
                       str_replace_all(pattern = "  ",
                                       replacement = " ") %>% 
                       tolower()
        )
```

Salvamos em disco o **arquivo final de pessoas do Censo de 1960**:

``` r
data_address <- "Data - After Consistency/Censo.1960.brasil.pessoas.amostra.1.27porcento.csv"

fwrite(x = censo1960_pess_final,
       file = data_address,
       na = "",
       row.names = F,
       quote = TRUE)

zip::zip(zipfile = 'Data - After Consistency/Censo.1960.brasil.pessoas.amostra.1.27porcento.zip', 
    files   = data_address)

file.remove(data_address)
```

    ## [1] TRUE

A lista de vari�veis contidas no arquivo de dados, ao fim, �:

``` r
names(censo1960_pess_final)
```

    ##  [1] "uf_pess"                        "uf_dom"                        
    ##  [3] "v116_pess"                      "v116_dom"                      
    ##  [5] "v118_pess"                      "v118_dom"                      
    ##  [7] "cem_idindividuo"                "cem_idfamilia"                 
    ##  [9] "cem_iddomicilio"                "cem_wgt"                       
    ## [11] "v202"                           "v203"                          
    ## [13] "v204"                           "v204b"                         
    ## [15] "v205"                           "v206"                          
    ## [17] "v207"                           "v208"                          
    ## [19] "v209"                           "v299"                          
    ## [21] "v210"                           "v211"                          
    ## [23] "v212"                           "v213"                          
    ## [25] "v214"                           "v215"                          
    ## [27] "v216"                           "v217"                          
    ## [29] "v218"                           "v219"                          
    ## [31] "v220"                           "v221"                          
    ## [33] "v223"                           "v223b"                         
    ## [35] "v224"                           "v101"                          
    ## [37] "v102"                           "v103"                          
    ## [39] "v104"                           "v105"                          
    ## [41] "v106"                           "v107"                          
    ## [43] "v108"                           "v109"                          
    ## [45] "v110"                           "v111"                          
    ## [47] "v112"                           "v113"                          
    ## [49] "cem_num_pessoas"                "cem_num_moradores"             
    ## [51] "cem_num_familias"               "cem_diagnosis_pess"            
    ## [53] "cem_problematic_vars_list_pess" "cem_diagnosis_dom"             
    ## [55] "cem_problematic_vars_list_dom"  "cem_dissonant_uf"              
    ## [57] "cem_dissonant_v116"             "cem_dissonant_v118"
