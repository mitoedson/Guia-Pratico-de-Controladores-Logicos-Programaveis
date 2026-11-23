# Transferência e conversão de dados

## O que é e quando usar

A **transferência** copia o valor de um endereço de origem para um endereço de destino. A origem permanece inalterada (ROCKWELL AUTOMATION, 1998). A **conversão** muda a representação de um valor, por exemplo de inteiro para ponto flutuante ou de binário para BCD (HANSSEN, 2015; ROCKWELL AUTOMATION, 1998).

Usos típicos da transferência, segundo Bolton (2015):

- mover um valor pré-ajustado para o preset de um temporizador ou contador;
- guardar em um registrador o acumulado de um temporizador ou contador;
- mover o dado de uma entrada para um registrador;
- mover o conteúdo de um registrador para uma saída.

A conversão aparece com mais frequência em cálculos numéricos com tipos diferentes. Os valores de entradas analógicas costumam ficar em endereços do tipo `WORD`, e para usá-los em expressões que exigem precisão é preciso convertê-los para ponto flutuante (HANSSEN, 2015).

## Transferência

### MOVE e MOV

Na norma, `MOVE` é a função de atribuição, com uma só entrada (HANSSEN, 2015). Nas linguagens textuais, o equivalente é o operador `:=`. Cada fabricante representa o movimento de modo próprio: Bolton (2015) mostra o formato da Mitsubishi, da Allen-Bradley e da Siemens, em que a Siemens usa um bloco `MOVE` com `EN`, `ENO`, `IN` e `OUT`. Em geral, usa-se um degrau para cada operação de movimento (BOLTON, 2015).

No manual do PLC-5, o MOV é uma instrução de saída que copia a origem para o destino, e **move o dado a cada varredura enquanto o degrau permanece verdadeiro**. O destino é sobrescrito, e a origem não muda (ROCKWELL AUTOMATION, 1998). Quando o degrau fica falso, o destino mantém o valor, a menos que outra parte do programa o altere (PETRUZELLA, 2014).

### MVM, a movimentação mascarada

A instrução MVM copia só uma parte selecionada de uma palavra. A máscara determina quais bits da origem passam para o destino (PETRUZELLA, 2014):

- onde a máscara tem 1, o dado passa da origem para o destino;
- onde a máscara tem 0, o bit do destino permanece no último estado;
- a máscara deve ter o mesmo tamanho da origem e do destino.

### Arquivos: COP e FLL

A **COP** (copiar arquivo) copia um arquivo de origem para um de destino, com a extensão total em cada varredura em que a instrução é verdadeira. A **FLL** (preencher arquivo) copia o valor de uma única origem para todas as palavras do arquivo de destino, e é muito usada para zerar um arquivo (PETRUZELLA, 2014).

### TRANSFER no WEG WLP

O bloco `TRANSFER` tem `EN`, `ENO` e dois argumentos, `SRC` (origem) e `DST` (destino). `ENO` vai a 1 quando a transferência termina, o que em geral é imediato, exceto quando a origem ou o destino é um parâmetro do inversor (WEG, 2016). A origem pode ser, entre outras opções, constante, marcador de bit, de palavra ou de ponto flutuante, marcador de sistema, entrada ou saída digital e entrada analógica (WEG, 2016).

## Conversão

### Tipos de dados

A norma define os tipos elementares abaixo (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003):

| Tipo | Descrição | Bits |
|------|-----------|------|
| `BOOL` | Booleano | 1 |
| `SINT`, `INT`, `DINT`, `LINT` | Inteiros com sinal | 8, 16, 32, 64 |
| `USINT`, `UINT`, `UDINT`, `ULINT` | Inteiros sem sinal | 8, 16, 32, 64 |
| `REAL`, `LREAL` | Ponto flutuante | 32, 64 |
| `BYTE`, `WORD`, `DWORD`, `LWORD` | Cadeias de bits | 8, 16, 32, 64 |

Há ainda `TIME`, `DATE`, `TIME_OF_DAY`, `DATE_AND_TIME` e `STRING` (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003).

### Funções de conversão

As funções de conversão têm a forma `*_TO_**`, em que `*` é o tipo da entrada e `**` o da saída, como `INT_TO_REAL` (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003). Em ST, a sintaxe é `Out := tipo1_TO_tipo2(In)`, e em FBD e LD a função é um bloco com uma entrada e uma saída (HANSSEN, 2015).

Os efeitos da conversão sobre a precisão e os tipos de erro que podem ocorrer são **dependentes da implementação** (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003). Em geral, **não se deve converter de um tipo maior para um menor**, como `WORD` para `BYTE` ou `DINT` para `INT`. No melhor caso há perda de informação, e no pior, um erro (HANSSEN, 2015).

### Ponto flutuante para inteiro: arredondar ou truncar

Segundo a norma, a conversão de `REAL` ou `LREAL` para inteiro **arredonda** pela convenção da IEC 60559: se os dois inteiros mais próximos estão à mesma distância, o resultado é o inteiro par mais próximo. A função `TRUNC` **trunca** em direção a zero (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003).

| Valor | `REAL_TO_INT` | `TRUNC` |
|-------|---------------|---------|
| 1,4 | 1 | 1 |
| 1,5 | 2 | 1 |
| 1,6 | 2 | 1 |
| 2,5 | 2 | 2 |
| −1,5 | −2 | −1 |
| −1,6 | −2 | −1 |
| −2,5 | −2 | −2 |

Fonte: INTERNATIONAL ELECTROTECHNICAL COMMISSION (2003). No CODESYS, o `TRUNC` converte para `DINT` e o `TRUNC_INT` converte para `INT`. No CODESYS 2.3, só existe o `TRUNC`, que corresponde ao `TRUNC_INT` da versão 3.x (HANSSEN, 2015).

!!! warning "O arredondamento do meio pode variar"
    Na ajuda do CODESYS publicada pela ABB, os valores com parte fracionária de 5 a 9 sobem e os de 1 a 4 descem, mas **o caso do meio depende do sistema-alvo ou da unidade de ponto flutuante**. Um valor como −1,5 pode ser convertido de modo diferente em controladores diferentes. Se o valor está fora da faixa do tipo de destino, o resultado é indefinido, e para escrever código independente do alvo é preciso tratar o estouro no aplicativo (ABB, 2025d).

### BCD

No formato BCD (decimal codificado em binário), cada dígito decimal é representado por 4 bits (BOLTON, 2015). O número 6923 fica 0110 1001 0010 0011, bem diferente do binário puro. Em 16 bits, o maior número em BCD é 9999 (HANSSEN, 2015).

O BCD é usado com dispositivos como mostradores digitais e chaves de dígito (*thumbwheel*), e muitos CLPs têm entradas e saídas programáveis para converter BCD em binário e o contrário (BOLTON, 2015).

Na norma, as funções `*_BCD_TO_**` e `**_TO_BCD_*` convertem entre cadeias de bits (`BYTE`, `WORD`, `DWORD`, `LWORD`) e inteiros sem sinal (`USINT`, `UINT`, `UDINT`, `ULINT`). Por exemplo, `USINT_TO_BCD_BYTE(25)` vale `2#0010_0101`, e `WORD_BCD_TO_UINT(2#0011_0110_1001)` vale 369 (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003).

No PLC-5, a instrução **TOD** converte inteiro em BCD. Se o inteiro for maior que 9999, o processador grava 9999 e liga o bit de overflow. Se for negativo, grava 0 e liga os bits de overflow e de zero (ROCKWELL AUTOMATION, 1998). A **FRD** converte BCD em inteiro, e o manual manda convertê-lo **antes** de manipulá-lo na lógica, porque o processador trata valores BCD como inteiros, e o valor BCD real pode se perder ou se distorcer (ROCKWELL AUTOMATION, 1998).

### INT2FL e FL2INT no WLP

O bloco `INT2FL` copia a palavra inteira para um marcador de ponto flutuante enquanto `EN` está em 1. O `FL2INT` faz o inverso e **satura** o resultado: se o valor for maior que 32767, a palavra inteira fica em 32767, e se for menor que −32768, fica em −32768. No exemplo do manual, a constante 3,14 vira 3 no parâmetro do usuário (WEG, 2016).

## Exemplo 1: escolher o tempo de um temporizador

Uma chave seletora escolhe entre dois tempos, e o valor escolhido é movido para o preset do temporizador. A ideia segue o exemplo de Petruzella (2014), com a chave de duas posições que seleciona um de dois valores pré-ajustados. Esta versão é original, em ST:

```pascal
IF chave_10s THEN
    tempo := T#10S;
ELSE
    tempo := T#20S;
END_IF

atraso(IN := partida, PT := tempo);
saida := atraso.Q;
```

Ver [temporizadores](temporizadores.md).

## Exemplo 2: entrada analógica para ponto flutuante

```pascal
leitura_real := WORD_TO_REAL(entrada_analogica);
```

O valor de uma entrada analógica chega em uma `WORD` e precisa virar `REAL` para entrar em cálculos que exigem precisão (HANSSEN, 2015). Confira o nome da função na biblioteca da sua plataforma. O escalonamento para unidades de engenharia é tema do capítulo de instrumentação.

## Exemplo 3: arredondar ou truncar

```pascal
a := REAL_TO_INT(7.5);        (* 8 *)
b := REAL_TO_INT(-23.6);      (* -24 *)
c := TRUNC_INT(-23.6);        (* -23 *)
```

Os resultados são os do exemplo de Hanssen (2015). Para `REAL_TO_INT(7.5)`, a regra do par mais próximo da norma também dá 8.

## Exemplo 4: contagem para um mostrador BCD

Para enviar a contagem a um mostrador de quatro dígitos BCD, o valor precisa ser convertido e não pode passar de 9999:

```pascal
IF pecas > 9999 THEN
    pecas := 9999;
END_IF
saida_display := UINT_TO_BCD_WORD(pecas);
```

O limite de 9999 vem do formato BCD em 16 bits (HANSSEN, 2015). Confira na sua plataforma se a função existe e com que nome.

## Equivalências entre plataformas

| Função | IEC 61131-3 | Rockwell PLC-5 | Rockwell Logix | WEG WLP |
|--------|-------------|----------------|----------------|---------|
| Copiar | `MOVE` ou `:=` | `MOV` | `MOVE`, antes `MOV` | `TRANSFER` |
| Copiar com máscara | Não consultado | `MVM` | Não consultado | Não encontrado |
| Copiar arquivo | Não consultado | `COP` | Não consultado | Não encontrado |
| Preencher arquivo | Não consultado | `FLL` | Não consultado | Não encontrado |
| Inteiro para real | `INT_TO_REAL` | Não consultado | Conversão automática ao misturar tipos | `INT2FL` |
| Real para inteiro | `REAL_TO_INT`, `TRUNC` | Pelo MOV, com bit de overflow | Conversão automática, com arredondamento | `FL2INT` |
| Inteiro para BCD | `*_TO_BCD_*` | `TOD` | Não consultado | Não encontrado |
| BCD para inteiro | `*_BCD_TO_*` | `FRD` | Não consultado | Não encontrado |

Fontes: IEC (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003); PLC-5 (ROCKWELL AUTOMATION, 1998; PETRUZELLA, 2014); Logix (ROCKWELL AUTOMATION, [s. d.]l; ROCKWELL AUTOMATION, [s. d.]m); WLP (WEG, 2016). **Siemens e Schneider:** a completar.

## Atualizações e comparação com versões mais recentes

Consulta em 21 nov. 2026.

| Fonte citada | Versão mais recente encontrada | Mudança relevante ou observação |
|--------------|--------------------------------|---------------------------------|
| IEC 61131-3, 2ª ed. (2003) | 3ª ed. (2013) e 4ª ed. (2025) (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2013; INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2025). | A PLCopen informa que a 3ª edição trouxe novos tipos de dados elementares (PLCOPEN, [s. d.]). Não consegui conferir se mudaram as funções de conversão, porque o texto é pago. |
| Rockwell, manual do PLC-5 (1998) | Ajuda do Studio 5000 Logix Designer, versão 38.00 (ROCKWELL AUTOMATION, [s. d.]l; ROCKWELL AUTOMATION, [s. d.]m). | A partir da versão 36 do Logix Designer, o mnemônico passou de `MOV` para `MOVE`. A instrução não existe em FBD nem em ST, onde se usa `:=` (ROCKWELL AUTOMATION, [s. d.]l). Se o valor da origem está fora da faixa do tipo do destino e a detecção de overflow está ativa, ocorre uma falha menor. Cadeias de texto são truncadas ao tamanho do destino (ROCKWELL AUTOMATION, [s. d.]l). A conversão de `REAL` para inteiro arredonda, e o meio vai para o par mais próximo, como na norma (ROCKWELL AUTOMATION, [s. d.]m). |
| CODESYS 2.3 (2007) e Hanssen (2015) | Ajuda do CODESYS publicada pela ABB, atualizada em 20 mar. 2025 (ABB, 2025d). | O arredondamento do caso do meio depende do sistema-alvo, e valores fora da faixa dão resultado indefinido (ABB, 2025d). |
| WEG, WLP V10.00 (06/2016) | Não encontrei manual do WLP posterior. | Nenhuma mudança verificada (WEG, 2016). |
| Bolton (2015), Hanssen (2015) e Petruzella (2014) | Bolton: 6ª edição é a mais recente segundo a Elsevier (ELSEVIER, 2015). | Não verifiquei edições posteriores de Hanssen e Petruzella. |

## Erros comuns

- Converter de um tipo maior para um menor sem tratar a perda. No Logix, converter `DINT` 16#0001_0081 (65.665) para `INT` dá 129, e para `SINT` dá −127, porque o controlador guarda só os bits baixos que cabem (ROCKWELL AUTOMATION, [s. d.]m; HANSSEN, 2015).
- Confundir arredondar com truncar (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003; HANSSEN, 2015).
- Esperar o mesmo arredondamento do valor 0,5 em todas as plataformas (ABB, 2025d).
- Esquecer que o MOV do PLC-5 executa a cada varredura enquanto o degrau está verdadeiro (ROCKWELL AUTOMATION, 1998).
- Fazer contas com valores BCD sem convertê-los antes (ROCKWELL AUTOMATION, 1998).
- Passar de 9999 em um campo BCD de 16 bits. No PLC-5, o TOD satura em 9999 e liga o overflow (ROCKWELL AUTOMATION, 1998).
- Sobrescrever o mesmo destino em mais de um ponto do programa. A ajuda do Logix avisa que operação inesperada pode ocorrer se operandos de saída forem sobrescritos (ROCKWELL AUTOMATION, [s. d.]l).
- Misturar tipos de tempo no Logix. Um tipo de tempo relativo só se move de ou para outro relativo, e um absoluto só de ou para outro absoluto (ROCKWELL AUTOMATION, [s. d.]l).

## Exercícios

1. Quanto valem `REAL_TO_INT(2.5)`, `REAL_TO_INT(3.5)` e `TRUNC(-1.6)`, segundo a norma?

    ??? note "Resposta"
        2, 4 e −1. O meio vai para o inteiro par mais próximo, e o `TRUNC` corta em direção a zero (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003).

2. Escreva o número 5280 em BCD.

    ??? note "Resposta"
        5 = 0101, 2 = 0010, 8 = 1000 e 0 = 0000, então 0101 0010 1000 0000 (BOLTON, 2015).

3. Uma instrução MVM tem origem `1010 1010 1100 1010`, destino inicial zerado e máscara `1111 1111 0000 1111`. Qual é o destino depois da execução?

    ??? note "Resposta"
        Onde a máscara é 1, o bit vem da origem. Onde é 0, o destino mantém o valor anterior. O resultado é `1010 1010 0000 1010` (PETRUZELLA, 2014).

4. No Logix, um `DINT` vale 65.665. Quanto vale depois de convertido para `INT`?

    ??? note "Resposta"
        129, os 16 bits baixos (16#0081) (ROCKWELL AUTOMATION, [s. d.]m).

5. No CODESYS, implemente os Exemplos 1 a 4. Teste `REAL_TO_INT` com valores como 2,5 e −2,5 e compare com a tabela da norma.

!!! note "Status de validação"
    Testado em: não testado. Os exemplos aguardam validação no CODESYS. Os exemplos são originais, salvo quando indicado.

Fontes: ver [Referências](../99-referencias/index.md).
