# Registradores de deslocamento e sequenciadores

## O que é e quando usar

**Registradores de deslocamento** guardam o estado de um evento para que se possa agir sobre ele mais tarde, e deslocam estados ou valores por arquivos de dados. Aplicações comuns: rastrear peças em uma linha de montagem, controlar máquinas ou operações de processo, controlar inventário e fazer diagnóstico (PETRUZELLA, 2014).

**Sequenciadores** evoluíram das chaves mecânicas de tambor. Eles repetem um padrão fixo de saídas ligadas e desligadas, como a chave de tambor, mas com flexibilidade muito maior, e servem para máquinas que têm uma sequência de passos repetitivos (PETRUZELLA, 2014).

Hanssen (2015) descreve as funções de deslocamento de bits da norma como funções clássicas, usadas do controle de motores de passo à monitoração de estados, e que a maioria das pessoas associa a registradores de deslocamento.

## Registrador de deslocamento

### Conceito

Um registrador é um grupo de relés internos, normalmente de 8, 16 ou 32, cada um guardando um estado 0 ou 1 (BOLTON, 2015). Um registrador de deslocamento precisa de **três entradas**: uma para carregar o dado na primeira posição, uma para comandar o deslocamento de uma posição e uma para zerar o registrador. Quando chega o comando de deslocamento, o dado de entrada entra na primeira posição, os bits andam uma posição e o último bit transborda e se perde (BOLTON, 2015).

Em um registrador de 8 bits no estado `1 0 1 1 0 0 1 0`, a chegada do sinal 0 e do comando de deslocamento leva ao estado `0 1 0 1 1 0 0 1`, e o último bit, 0, é perdido (BOLTON, 2015).

Um registrador de deslocamento de bit opera de forma **síncrona**, ou serial: para cada bit que entra, um sai, e o dado precisa percorrer toda a extensão, uma posição por pulso, antes de poder ser avaliado na saída (PETRUZELLA, 2014).

### BSL e BSR

No SLC 500, o **BSL** desloca os bits do endereço baixo para o alto, e o **BSR** do alto para o baixo. Alguns CLPs têm também um registrador circular, que repete um padrão em sequência (PETRUZELLA, 2014). A instrução executa quando o degrau passa de falso para verdadeiro (PETRUZELLA, 2014).

Parâmetros no PLC-5 (ROCKWELL AUTOMATION, 1998):

- **Arquivo:** a matriz de bits, que deve começar no limite de uma palavra de 16 bits. Os bits restantes da última palavra ficam invalidados pela instrução e não podem ser usados em outro ponto do programa.
- **Controle:** estrutura de 48 bits (três palavras) com os bits de status, o tamanho da matriz e o ponteiro de bit.
- **Endereço do bit:** o bit de origem, inserido na primeira posição (BSL) ou na última (BSR).
- **Comprimento:** quantidade de bits da matriz.
- **Bits de status:** `.EN` (habilitado), `.DN` (a matriz deslocou uma posição), `.ER` (erro, como um comprimento negativo) e `.UL` (bit descarregado, a saída da instrução).

No Logix, o BSL usa uma matriz `DINT`, uma estrutura de controle, o bit de origem e o comprimento. É uma instrução transicional e descarrega o bit mais alto no `.UL`. Os bits restantes do último elemento ficam inválidos (ROCKWELL AUTOMATION, [s. d.]q).

### Funções SHL, SHR, ROL e ROR da IEC 61131-3

| Função | Resultado |
|--------|-----------|
| `SHL` | `IN` deslocado `N` bits para a esquerda, com zeros à direita. |
| `SHR` | `IN` deslocado `N` bits para a direita, com zeros à esquerda. |
| `ROL` | `IN` rotacionado `N` bits para a esquerda, de forma circular. |
| `ROR` | `IN` rotacionado `N` bits para a direita, de forma circular. |

Fonte: INTERNATIONAL ELECTROTECHNICAL COMMISSION (2003). As entradas são cadeias de bits (`ANY_BIT`), e a função se aplica a `BYTE`, `WORD`, `DWORD` e `LWORD`. Se `N` for maior que o número de bits de `IN`, o resultado de `SHL` e `SHR` é uma cadeia vazia (HANSSEN, 2015).

Exemplo com o `BYTE` `2#10110100` (B4 em hexadecimal) e `N = 2`, aplicando as definições acima:

| Função | Resultado em binário | Hexadecimal |
|--------|----------------------|-------------|
| `SHL` | `2#11010000` | D0 |
| `SHR` | `2#00101101` | 2D |
| `ROL` | `2#11010010` | D2 |
| `ROR` | `2#00101101` | 2D |

O ponto de partida do exemplo vem de Hanssen (2015), e os resultados são calculados a partir das definições da norma. Hanssen (2015) descreve a função `SHR` com uma frase que diz "esquerda" por engano. Vale a definição da norma.

### Exemplo 1: fila de peças com ejetor

Cada peça que passa pelo sensor desloca a fila uma posição e carrega 1 se a câmera detectou defeito. O ejetor fica na posição 8. O exemplo é original, em ST.

```pascal
borda_peca(CLK := sensor_peca);          (* uma vez por peça *)

IF borda_peca.Q THEN
    fila := SHL(fila, 1);                (* desloca uma posição *)
    IF defeito THEN
        fila := fila OR 16#0001;         (* carrega o novo bit *)
    END_IF
END_IF

ejetor := (fila AND 16#0100) <> 16#0000; (* bit 8 *)
```

A variável `fila` é uma `WORD` e `borda_peca` é uma instância de `R_TRIG`. Sem a borda, `SHL` seria executada a cada varredura, e a fila andaria sem relação com as peças. Isso é dedução do fato de que as funções executam sempre que são chamadas com `EN` verdadeiro (HANSSEN, 2015). No BSL, essa proteção já está no fato de a instrução ser transicional (ROCKWELL AUTOMATION, [s. d.]q).

### FIFO e LIFO

Em vez de deslocar bits, as instruções FIFO deslocam **palavras inteiras** por um arquivo, e são usadas, por exemplo, no acompanhamento de peças com número de peça ou código de montagem. A FFL carrega e a FFU descarrega, sempre em par, na mesma ordem de entrada (primeira a entrar, primeira a sair) (PETRUZELLA, 2014). Diferentemente do registrador de bits, a FIFO é assíncrona: a carga e a descarga operam de modo independente (PETRUZELLA, 2014). O PLC-5 tem também LFL e LFU, para descarregar em ordem inversa, e esse par só existe nos processadores Enhanced PLC-5 (ROCKWELL AUTOMATION, 1998).

## Sequenciadores

### De chave de tambor a instrução

Na chave de tambor, pinos em posições específicas acionam blocos de contatos normalmente abertos conforme o tambor gira (PETRUZELLA, 2014). A instrução **SQO** (saída do sequenciador) substitui o tambor. Ela lê, um de cada vez, os elementos de um arquivo de dados, aplica uma **máscara** e transfere o elemento mascarado para uma saída designada (PETRUZELLA, 2014).

- **Arquivo:** cada palavra é um passo. Cada bit da palavra define se uma saída fica ligada ou desligada naquele passo.
- **Máscara:** um 1 deixa o dado passar, e um 0 bloqueia. As saídas bloqueadas não são controladas pelo sequenciador e podem ser usadas por outras partes do programa (PETRUZELLA, 2014).
- **Posição:** o passo atual. A posição avança em 1 a cada transição de falso para verdadeiro do degrau (PETRUZELLA, 2014).
- **Comprimento:** número de passos, a partir da posição 1. A posição 0 é a de partida, e a instrução volta à posição 1 ao terminar o ciclo (ROCKWELL AUTOMATION, 1998). O arquivo ocupa o comprimento mais 1 palavra (ROCKWELL AUTOMATION, 1998).
- **Bits de status:** `.EN`, `.DN`, que liga depois de processar a última palavra do arquivo, e `.ER` (ROCKWELL AUTOMATION, 1998).

Petruzella (2014) observa que as instruções de sequenciadores geralmente são **retentivas** e podem ter limite para o número de saídas externas e de passos por instrução.

### Por tempo ou por evento

- **Acionado por tempo:** um temporizador avança o sequenciador. O exemplo clássico é o semáforo, em que uma SQO controla as lâmpadas e outra SQO fornece os tempos de cada passo, com valores pré-ajustados de 25, 5, 25 e 5 segundos (PETRUZELLA, 2014).
- **Acionado por evento:** o avanço depende de um evento, ou seja, de uma entrada. Cada ocorrência faz a SQO avançar para o próximo passo (PETRUZELLA, 2014).

### SQI, SQC e SQL

| Instrução | Tipo | Função |
|-----------|------|--------|
| SQI | Entrada | Compara os bits de uma origem com o arquivo. É verdadeira se todos os pares de bits forem iguais. |
| SQC | Saída | Compara a origem com a palavra do arquivo e liga o bit `.FD` quando todos os pares combinam. Incrementa a posição. |
| SQL | Saída | Carrega o arquivo do sequenciador com os estados lidos de um módulo de entrada, para registrar as condições de cada passo. |

Fonte: PETRUZELLA (2014). A SQI se emparelha com uma SQO de mesmo endereço de controle. A posição da SQI segue a da SQO, o que permite que uma saída específica ocorra quando uma entrada específica acontece naquele passo (PETRUZELLA, 2014).

### Exemplo 2: tabela de passos em ST

A norma, nas partes consultadas, não traz uma instrução equivalente ao SQO. A mesma ideia é montada com uma tabela e um índice de passo. Aqui, as três saídas de uma máquina avançam passo a passo a cada pulso em `avanca`. O exemplo é original.

```pascal
VAR
    tabela : ARRAY[0..3] OF BYTE :=
             [2#00000001, 2#00000011, 2#00000110, 2#00000100];
    passo  : INT := 0;
    saidas : BYTE;
END_VAR

borda_avanca(CLK := avanca);

IF reinicio THEN
    passo := 0;
ELSIF borda_avanca.Q THEN
    passo := (passo + 1) MOD 4;
END_IF

saidas := tabela[passo];
valvula_a := (saidas AND 2#00000001) <> 0;
valvula_b := (saidas AND 2#00000010) <> 0;
motor     := (saidas AND 2#00000100) <> 0;
```

`borda_avanca` é uma instância de `R_TRIG`. O resto da divisão faz o passo voltar a 0 depois do último, o que equivale ao retorno do SQO à posição inicial. Para sequências com condições de transição entre etapas, o caminho é o SFC, previsto no capítulo 5.

## Deslocamento no WEG WLP

O bloco `MATH` do WLP tem um operador de deslocamento em palavra dupla. O segundo dado é a quantidade de posições: se positivo, o deslocamento é para a esquerda, e se negativo, para a direita, com zeros nos bits deslocados. O operador `Ashift` funciona igual, mas **não desloca nem altera o sinal** do primeiro dado (WEG, 2016).

!!! warning "O bloco SHIFT do WLP não é deslocamento de bits"
    O bloco `SHIFT` do WLP pertence ao grupo de posicionamento e define o valor máximo de correção de pulsos por ciclo de varredura (WEG, 2016).

Não encontrei no manual do WLP blocos de FIFO, LIFO ou sequenciador.

## Equivalências entre plataformas

| Função | IEC 61131-3 | Rockwell PLC-5 | Rockwell Logix | WEG WLP |
|--------|-------------|----------------|----------------|---------|
| Deslocar uma palavra | `SHL`, `SHR` | Não consultado | Não consultado | `MATH` com `Shift` e `Ashift` |
| Rotacionar | `ROL`, `ROR` | Não consultado | Não consultado | Não encontrado |
| Registrador de bits | Montado com `SHL` e borda | `BSL`, `BSR` | `BSL` | Não encontrado |
| FIFO e LIFO | Não consultado | `FFL`, `FFU`, `LFL`, `LFU` | Não consultado | Não encontrado |
| Sequenciador de saída | Tabela e índice | `SQO` | `SQO` | Não encontrado |
| Comparação e carga do sequenciador | Não encontrado | `SQI`, `SQC`, `SQL` | Mesma estrutura de controle de `SQI` e `SQL` | Não encontrado |

Fontes: IEC (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003); PLC-5 (ROCKWELL AUTOMATION, 1998; PETRUZELLA, 2014); Logix (ROCKWELL AUTOMATION, [s. d.]q; ROCKWELL AUTOMATION, [s. d.]r); WLP (WEG, 2016). **Siemens e Schneider:** a completar.

## Atualizações e comparação com versões mais recentes

Consulta em 21 nov. 2026.

| Fonte citada | Versão mais recente encontrada | Mudança relevante ou observação |
|--------------|--------------------------------|---------------------------------|
| IEC 61131-3, 2ª ed. (2003) | 3ª ed. (2013) e 4ª ed. (2025) (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2013; INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2025). | Não consegui conferir se mudaram as funções de deslocamento, porque o texto é pago. |
| Rockwell, manual do PLC-5 (1998) | Ajuda do Studio 5000 Logix Designer, versão 38.00 (ROCKWELL AUTOMATION, [s. d.]q; ROCKWELL AUTOMATION, [s. d.]r). | O BSL agora trabalha com matriz `DINT`, e a estrutura de controle tem `.LEN` e `.POS`. Na pré-varredura e com o degrau falso, o BSL limpa `.EN`, `.DN`, `.ER` e `.POS`. A ajuda manda testar e confirmar que a instrução não altera dados indesejados. No SQO, a matriz é `DINT`, a máscara aceita `SINT`, `INT` ou `DINT`, o `.DN` liga quando `.POS` é igual a `.LEN`, e a ajuda manda usar o mesmo controle do SQI e do SQL (ROCKWELL AUTOMATION, [s. d.]q; ROCKWELL AUTOMATION, [s. d.]r). |
| WEG, WLP V10.00 (06/2016) | Não encontrei manual do WLP posterior. | Nenhuma mudança verificada (WEG, 2016). |
| Bolton (2015), Hanssen (2015) e Petruzella (2014) | Bolton: 6ª edição é a mais recente segundo a Elsevier (ELSEVIER, 2015). | Não verifiquei edições posteriores de Hanssen e Petruzella. |

## Erros comuns

- Usar o mesmo endereço de controle em instruções diferentes. No PLC-5, o BSL, o SQO e o SQL exigem um endereço de controle próprio, e a exceção é o par SQO e SQI. O manual avisa que a duplicação pode causar operação imprevisível e danos a equipamentos ou pessoas (ROCKWELL AUTOMATION, 1998).
- Não reiniciar o sequenciador ao passar de Program para Run. O manual do PLC-5 manda zerar a posição do SQO com o bit de primeira varredura `S:1/15` (ROCKWELL AUTOMATION, 1998).
- Usar os bits restantes da última palavra de uma matriz de BSL. A instrução os invalida (ROCKWELL AUTOMATION, 1998; ROCKWELL AUTOMATION, [s. d.]q).
- Esperar deslocamento contínuo de uma instrução transicional. O BSL só desloca a cada transição de falso para verdadeiro do degrau (ROCKWELL AUTOMATION, [s. d.]q).
- Chamar `SHL` ou `SHR` a cada varredura sem uma borda, o que desloca o registrador em todo ciclo.
- Tratar a máscara ao contrário. O `1` deixa o dado passar, e o `0` bloqueia (PETRUZELLA, 2014).
- Esquecer que o SQC é uma instrução de saída, e não de entrada como a SQI (PETRUZELLA, 2014).
- Confundir o bloco `SHIFT` do WLP, de posicionamento, com deslocamento de bits (WEG, 2016).

## Exercícios

1. Aplique `SHL` e `ROL` com `N = 2` ao `BYTE` `2#10110100`.

    ??? note "Resposta"
        `SHL` dá `2#11010000` (D0), e `ROL` dá `2#11010010` (D2). No `ROL`, os dois bits mais altos, `10`, voltam pela direita (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003).

2. Um registrador de 8 bits está em `1 0 1 1 0 0 1 0`. Chega o sinal 0 e o comando de deslocamento. Qual é o novo estado?

    ??? note "Resposta"
        `0 1 0 1 1 0 0 1`, e o último bit, 0, transborda e se perde (BOLTON, 2015).

3. Um SQO tem comprimento 4 e já executou o passo 4, com `.DN` ligado. O que acontece na próxima transição de falso para verdadeiro do degrau?

    ??? note "Resposta"
        A posição volta ao passo 1 (PETRUZELLA, 2014; ROCKWELL AUTOMATION, 1998).

4. Um passo do sequenciador contém `1010 1010 1010 1010`, a máscara é `1111 0000 1111 0000` e a palavra de saída estava em `0101 0101 0101 0101`. Qual é a saída depois do passo?

    ??? note "Resposta"
        Onde a máscara é 1, o bit vem do passo. Onde é 0, a saída mantém o valor. O resultado é `1010 0101 1010 0101` (PETRUZELLA, 2014).

5. No CODESYS, implemente os Exemplos 1 e 2 e observe o que acontece se o `R_TRIG` for removido.

!!! note "Status de validação"
    Testado em: não testado. Os exemplos aguardam validação no CODESYS. Os exemplos em ST são originais.

Fontes: ver [Referências](../99-referencias/index.md).
