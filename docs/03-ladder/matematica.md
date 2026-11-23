# Matemática

## O que é e quando usar

As instruções de matemática dão ao CLP parte dos recursos de um computador: executar operações aritméticas com valores guardados na memória (PETRUZELLA, 2014). Usos típicos citados na literatura:

- Subtrair a contagem de peças produzidas de uma cota e mostrar quantas faltam em um display (PETRUZELLA, 2014).
- Ajustar a leitura de um sensor, ou obter um valor pela diferença entre dois sensores (BOLTON, 2015).
- Alterar os valores de preset de temporizadores e contadores (BOLTON, 2015).
- Multiplicar uma entrada antes de somá-la a outra ou subtraí-la (BOLTON, 2015).

As quatro operações básicas são adição, subtração, multiplicação e divisão. Alguns CLPs fazem apenas adição e subtração, e outros incluem funções como a exponencial (BOLTON, 2015).

Em Ladder, as instruções de matemática são **instruções de saída**: executam quando o degrau está verdadeiro, diferentemente da comparação, que é de entrada (PETRUZELLA, 2014).

## Funções aritméticas da IEC 61131-3

| Função | Operador | O que faz |
|--------|----------|-----------|
| `ADD` | `+` | Soma, com número qualquer de entradas. |
| `SUB` | `-` | `IN1 - IN2`. |
| `MUL` | `*` | Multiplica, com número qualquer de entradas. |
| `DIV` | `/` | `IN1 / IN2`. |
| `MOD` | | Resto da divisão. |
| `EXPT` | `**` | Potenciação. |
| `MOVE` | `:=` | Atribuição, com uma só entrada. |

Fonte da tabela: HANSSEN (2015). As entradas são números ou variáveis dos tipos `ANY_NUM` ou `ANY_BIT`, e `ADD` e `SUB` também valem para o tipo `TIME` (HANSSEN, 2015). O módulo `MOD` devolve o resto: `14 MOD 4` é 2, porque 14 dividido por 4 dá 3 com resto 2 (HANSSEN, 2015).

A norma também define funções numéricas: `ABS`, `SQRT`, `LN`, `LOG`, `EXP`, `SIN`, `COS`, `TAN`, `ASIN`, `ACOS` e `ATAN` (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003). Funções de seleção como `SEL`, `MUX`, `MAX`, `MIN` e `LIMIT` ficam em uma família própria (HANSSEN, 2015).

### EN e ENO em Ladder

No Ladder, o trilho de energia é sempre verdadeiro, então as funções precisam de uma entrada de habilitação `EN` para serem ativadas, e têm uma saída `ENO` (HANSSEN, 2015). Pelas regras da norma (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003):

1. Se `EN` for falso quando a função é chamada, o corpo da função não executa e `ENO` é zerado.
2. Caso contrário, `ENO` é ligado e a função executa.
3. Se ocorrer um erro durante a execução de uma função padrão, `ENO` é zerado, ou o fabricante define outro tratamento.
4. Com `ENO` falso, os valores de todas as saídas da função são **dependentes da implementação**.

```text
 |   partida     +----------+                               |
 +-----] [-------| EN   ENO |-------------------------      |
 |   a ----------| IN1  OUT |---- c                         |
 |   b ----------| IN2      |                               |
 |               |   ADD    |                               |
 |               +----------+                               |
```

## Cuidados

### Estouro (overflow)

Um resultado guardado em uma variável `INT` precisa ficar dentro dos limites do tipo. Se ultrapassar, ocorre estouro, e o valor gravado pode ser qualquer um, o que Hanssen (2015) considera indesejável. O programador deve prever o estouro, limitar o resultado ao máximo ou mínimo, ou escolher outro tipo de dado (HANSSEN, 2015).

Segundo a norma, é **erro** quando o resultado de uma operação numérica excede a faixa do tipo de dado ou quando se tenta dividir por zero (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003). No PLC-5, os inteiros de 16 bits vão de −32.768 a +32.767 (ROCKWELL AUTOMATION, 1998), e há flags de status aritmético no arquivo de status do processador: `S:0/0` (carry), `S:0/1` (overflow), `S:0/2` (zero) e `S:0/3` (sinal) (ROCKWELL AUTOMATION, 1998).

### Divisão de inteiros

Quando a divisão de dois inteiros não é exata, o resultado é **truncado**: o resto é descartado, mesmo que a variável de destino seja `REAL` (HANSSEN, 2015). Truncar não é arredondar.

| Divisão | Valor exato | Resultado inteiro |
|---------|-------------|-------------------|
| 13 / 10 | 1,3 | 1 |
| 99 / 10 | 9,9 | 9 |
| 17 / 5 | 3,4 | 3 |
| 534 / 200 | 2,67 | 2 |
| 1 / 2 | 0,5 | 0 |

Fonte: HANSSEN (2015). Para evitar o truncamento, ao menos um dos números precisa ser convertido para ponto flutuante antes da divisão, com funções como `INT_TO_REAL` (HANSSEN, 2015).

## Exemplo 1: cota de produção

Um contador acumula as peças produzidas, e o programa calcula quantas faltam para a meta. O exemplo é original, inspirado no uso descrito por Petruzella (2014).

```pascal
restantes := meta_diaria - pecas_produzidas;
meta_atingida := restantes <= 0;
```

Em Ladder, uma instrução `SUB` guarda o resultado em `restantes`, e uma comparação `LEQ` aciona `meta_atingida`. Os tipos das variáveis precisam ser compatíveis entre si (ver [contadores](contadores.md), onde o tipo de `CV` depende da biblioteca).

## Exemplo 2: soma de dois contadores

Petruzella (2014) apresenta um programa que usa a instrução ADD para somar os valores acumulados de dois contadores crescentes e liga um sinaleiro quando a soma é **maior ou igual a 350**. Aqui, em ST:

```pascal
total := contador_a + contador_b;
sinaleiro := total >= 350;
```

## Exemplo 3: média de duas leituras

Com inteiros, `(a + b) / 2` é truncado. Para manter a parte fracionária, converte-se antes:

```pascal
media_inteira := (leitura1 + leitura2) / 2;
media_real    := (INT_TO_REAL(leitura1) + INT_TO_REAL(leitura2)) / 2.0;
```

Com `leitura1 = 3` e `leitura2 = 4`, `media_inteira` vale 3 e `media_real` vale 3,5 (HANSSEN, 2015).

## Equivalências entre plataformas

| Função | IEC 61131-3 | Rockwell PLC-5 | Rockwell Logix | WEG WLP |
|--------|-------------|----------------|----------------|---------|
| Soma | `ADD` | `ADD` | `ADD` | `MATH` com `+` |
| Subtração | `SUB` | `SUB` | Não consultado | `MATH` com `-` |
| Multiplicação | `MUL` | `MUL` | Não consultado | `MATH` com `*` |
| Divisão | `DIV` | `DIV` | Não consultado | `MATH` com `/` |
| Resto | `MOD` | Não encontrado entre as instruções listadas | Não consultado | Não encontrado |
| Expressão | Expressão em ST | `CPT` | Não consultado | Não encontrado |
| Raiz quadrada | `SQRT` | `SQR` | Não consultado | `FUNC` |

Fontes: IEC (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003; HANSSEN, 2015); PLC-5 (ROCKWELL AUTOMATION, 1998); Logix (ROCKWELL AUTOMATION, [s. d.]j); WLP (WEG, 2016). Observações:

- **PLC-5:** o capítulo de compute do manual reúne `CPT`, `ADD`, `SUB`, `MUL`, `DIV`, `NEG`, `CLR`, `SQR`, além de funções trigonométricas, logarítmicas e outras (ROCKWELL AUTOMATION, 1998). O `CPT` avalia uma expressão e guarda o resultado no destino. Seu tempo de execução é maior que o de uma instrução simples (PETRUZELLA, 2014).
- **Bolton:** no Allen-Bradley, as instruções são `ADD`, `SUB`, `DIV`, `MUL` e `SQR`. No exemplo da `ADD`, a fonte A em `N7:1` é somada à fonte B em `N7:3` e o resultado vai para `N7:5` (BOLTON, 2015).
- **WLP:** o bloco `MATH` tem `EN` e `ENO`, e `ENO` é uma cópia de `EN`. No formato ponto flutuante faz `+`, `-`, `*`, `/` e potência. No formato inteiro, também opera com palavras (`or`, `and`, `xor`, `nor`, `nand`, `xnor`, deslocamentos) e fornece saídas de overflow e sinal (WEG, 2016). O bloco `FUNC` executa raiz quadrada, seno, cosseno, tangente, exponencial, logaritmos, truncamento, parte fracionária e arredondamento (WEG, 2016).
- **Siemens e Schneider:** a completar.

## Atualizações e comparação com versões mais recentes

Consulta em 21 nov. 2026.

| Fonte citada | Versão mais recente encontrada | Mudança relevante ou observação |
|--------------|--------------------------------|---------------------------------|
| IEC 61131-3, 2ª ed. (2003) | 3ª ed. (2013) e 4ª ed. (2025) (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2013; INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2025). | A PLCopen informa que a 3ª edição trouxe novos tipos de dados elementares (PLCOPEN, [s. d.]). Não consegui conferir se mudaram as regras das funções aritméticas, porque o texto é pago. |
| Rockwell, manual do PLC-5 (1998) | Ajuda do Studio 5000 Logix Designer, versão 38.00 (ROCKWELL AUTOMATION, [s. d.]j). | Os operandos do `ADD` deixaram de ser apenas inteiros de 16 bits: os controladores CompactLogix 5370 e ControlLogix 5570 aceitam `SINT`, `INT`, `DINT` e `REAL`, e os modelos 5380, 5580 e 5590 aceitam também `LINT`, tipos sem sinal, `LREAL` e tipos de tempo e data. Nos modelos novos o `ADD` afeta os flags de matemática de forma condicional, e nos 5370 e 5570 sempre (ROCKWELL AUTOMATION, [s. d.]j). |
| WEG, WLP V10.00 (06/2016) | Não encontrei manual do WLP posterior. | Nenhuma mudança verificada (WEG, 2016). |
| Hanssen (2015), Bolton (2015) e Petruzella (2014) | Bolton: 6ª edição é a mais recente segundo a Elsevier (ELSEVIER, 2015). | Não verifiquei edições posteriores de Hanssen e Petruzella. |

## Erros comuns

- Deixar o resultado estourar o tipo de dado sem tratar o caso (HANSSEN, 2015).
- Esperar arredondamento em uma divisão de inteiros. O resultado é truncado, mesmo guardado em `REAL` (HANSSEN, 2015).
- Dividir por zero. A norma trata como erro, com `ENO` zerado e saídas dependentes da implementação (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003).
- Usar uma instrução de saída, como `ADD`, em um degrau que fica verdadeiro por vários ciclos. No Logix, quando o degrau está verdadeiro o `ADD` calcula `Dest = Source A + Source B` (ROCKWELL AUTOMATION, [s. d.]j), e por dedução isso se repete a cada varredura. Se o destino for um dos operandos, o valor acumula a cada ciclo (ver [ciclo de varredura](../01-fundamentos/ciclo-de-varredura.md)).
- Misturar tipos numéricos sem conferir as regras de conversão da plataforma. A ajuda do Logix manda consultar as regras de conversão de dados quando se misturam tipos em uma instrução (ROCKWELL AUTOMATION, [s. d.]j).
- Ignorar os flags de status aritmético do PLC-5 (`S:0/0` a `S:0/3`), que o manual manda monitorar ao executar funções aritméticas (ROCKWELL AUTOMATION, 1998).

## Exercícios

1. Quanto vale `534 / 200` com inteiros? E `14 MOD 4`?

    ??? note "Resposta"
        `534 / 200` dá 2, porque o resultado é truncado (2,67 vira 2). `14 MOD 4` dá 2, o resto da divisão (HANSSEN, 2015).

2. `a` e `b` são `INT` com valores 30000 e 10000. O que acontece com `c := a + b;`?

    ??? note "Resposta"
        A soma, 40000, excede o máximo de +32.767 de um inteiro de 16 bits (ROCKWELL AUTOMATION, 1998), o que caracteriza estouro. O valor gravado pode ser qualquer um (HANSSEN, 2015), e a norma trata o caso como erro (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003).

3. Reescreva o Exemplo 1 de modo que `restantes` nunca fique negativo.

    ??? note "Resposta"
        Uma possibilidade, em ST: `IF restantes < 0 THEN restantes := 0; END_IF`. É uma sugestão original, e vale testar no CODESYS.

4. Uma instrução `ADD` com `EN` ligado e um destino igual a um dos operandos, em um degrau verdadeiro por 10 ciclos, soma quantas vezes?

    ??? note "Resposta"
        Por dedução, 10 vezes, uma por varredura, porque a instrução executa enquanto o degrau está verdadeiro (ROCKWELL AUTOMATION, [s. d.]j).

5. No CODESYS, implemente os Exemplos 1 a 3 e observe o efeito do tipo `INT` contra `REAL`.

!!! note "Status de validação"
    Testado em: não testado. Os exemplos aguardam validação no CODESYS. Os diagramas em texto são originais.

Fontes: ver [Referências](../99-referencias/index.md).
