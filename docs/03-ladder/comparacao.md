# Comparação

## O que é e quando usar

Uma instrução de comparação confronta dois valores e responde se a relação é verdadeira. Serve, por exemplo, para iniciar uma ação quando a leitura digital de um sensor de temperatura fica abaixo de um valor ajustado e guardado em um registrador (BOLTON, 2015).

Em Ladder, a comparação é uma **instrução de entrada**: ela se comporta como um contato e deixa passar o fluxo quando a relação é verdadeira. As instruções de transferência e de matemática, ao contrário, são instruções de saída (PETRUZELLA, 2014).

## Os seis operadores

Os CLPs costumam comparar por menor que, igual a, menor ou igual a, maior que, maior ou igual a e diferente de, e cada fabricante usa abreviações próprias (BOLTON, 2015).

| Relação | Operador em ST | Nome na norma | Abreviações comuns |
|---------|----------------|---------------|--------------------|
| Igual a | `=` | EQ | EQU |
| Diferente de | `<>` | NE | NEQ |
| Menor que | `<` | LT | LES |
| Menor ou igual a | `<=` | LE | LEQ |
| Maior que | `>` | GT | GRT |
| Maior ou igual a | `>=` | GE | GEQ |

Fontes: HANSSEN (2015) e BOLTON (2015).

### Mais de dois operandos

Na norma, as funções de comparação aceitam vários operandos e comparam cada par vizinho. O `GT`, por exemplo, devolve `(IN1>IN2) & (IN2>IN3) & ...`, e o `NE` aceita só dois (HANSSEN, 2015). A **ordem dos operandos importa**: para a saída ficar verdadeira quando `Num1` é maior que `Num2`, `Num1` precisa estar na entrada de cima do bloco `GT` (HANSSEN, 2015).

## Formas de escrever

- **ST:** a comparação é uma expressão booleana, como `saida := P2 < P1;` (BOLTON, 2015). Hanssen (2015) mostra duas variantes equivalentes, com `IF` e com atribuição direta, e destaca a segunda como a mais compacta.
- **Bloco em Ladder ou FBD:** a representação é a mesma para todas as comparações e muda só o nome ou o símbolo no bloco. Em LD, o bloco precisa de uma entrada de habilitação, `EN` (HANSSEN, 2015).
- **Instrução no degrau (estilo Rockwell e Mitsubishi):** a instrução traz a origem A e a origem B, e Bolton (2015) mostra o formato dos três fabricantes para o "maior que".
- **Bloco COMP do WEG WLP:** tem `EN`, `ENO` e quatro argumentos (formato, operador, dado 1 e dado 2). Com `EN` em 1 e a comparação verdadeira, `ENO` vai a 1 (WEG, 2016).

## Exemplo 1: alarme de temperatura alta

O alarme liga quando a temperatura passa do limite. O exemplo é original, no estilo de Hanssen (2015).

```pascal
alarme_temperatura := temperatura > limite_alto;
```

```text
 |   GRT                                         alarme_temperatura |
 +---[ A: temperatura   B: limite_alto ]-----------------( )--------+
```

O resultado é recalculado a cada varredura (ver [ciclo de varredura](../01-fundamentos/ciclo-de-varredura.md)).

## Exemplo 2: aquecedor com duas faixas

O aquecedor liga abaixo de 18 e desliga acima de 22. Entre os dois valores, mantém o estado em que estava. O exemplo é original.

```pascal
IF temperatura < 18.0 THEN
    aquecedor := TRUE;
ELSIF temperatura > 22.0 THEN
    aquecedor := FALSE;
END_IF
```

Em Ladder, a mesma lógica usa duas comparações e as bobinas Set e Reset, descritas em [contatos e bobinas](contatos-e-bobinas.md).

## Exemplo 3: valor dentro de uma faixa

Para testar se uma leitura analógica está entre dois limites, a norma permite combinar duas comparações:

```pascal
leitura_ok := (leitura >= limite_baixo) AND (leitura <= limite_alto);
```

O PLC-5 tem uma instrução própria para isso, a **LIM** (limit test). Ela é falsa até detectar que o valor de teste está dentro dos limites, e fica falsa de novo quando o valor sai da faixa. O manual cita o uso para verificar se uma entrada analógica está dentro de limites (ROCKWELL AUTOMATION, 1998).

Segundo o manual, a LIM tem dois modos conforme os limites (ROCKWELL AUTOMATION, 1998):

- **Limite baixo ≤ limite alto:** a instrução é verdadeira quando o valor está entre os limites ou é igual a um deles.
- **Limite baixo ≥ limite alto:** a instrução é verdadeira quando o valor está fora dos limites ou é igual a um deles.

## Exemplo 4: comparar o acumulado de um temporizador

Petruzella (2014) mostra uma instrução EQU que compara o valor acumulado de um temporizador (`T4:0`) com um valor em `N7:40`: quando os dois são iguais, a saída é energizada. Bolton (2015) mostra um GRT que compara o acumulado do temporizador `T4:0` com a constante 400.

```text
 |   GRT                                              saida |
 +---[ A: T4:0.ACC   B: 400 ]---------------------------( )--+
```

Os conceitos de comparação já apareciam nos temporizadores e contadores, nos quais a saída liga quando o acumulado atinge o preset (PETRUZELLA, 2014). Ver [temporizadores](temporizadores.md) e [contadores](contadores.md).

## Tipos de dados

- **Norma:** os operandos são números ou variáveis da classe `ANY_ELEMENTARY`. Em cadeias de bits de comprimentos diferentes, a mais curta é preenchida com zeros à direita (HANSSEN, 2015).
- **PLC-5:** é possível comparar valores de tipos diferentes, como ponto flutuante e inteiro. Valores BCD e ASCII são tratados como inteiros (ROCKWELL AUTOMATION, 1998).
- **Logix:** a comparação aceita muitos tipos numéricos, de tempo e de data, além de cadeias de texto. Na comparação numérica, o resultado só é verdadeiro se nenhum dos operandos for `NaN` (ROCKWELL AUTOMATION, [s. d.]k).
- **WLP:** o formato do bloco COMP é sempre constante e pode ser inteiro ou ponto flutuante (WEG, 2016).

## Equivalências entre plataformas

| Função | IEC 61131-3 | Rockwell PLC-5 | Rockwell Logix | WEG WLP |
|--------|-------------|----------------|----------------|---------|
| Igual a | `EQ` | `EQU` | `EQ`, antes `EQU` | `COMP` com `=` |
| Demais relações | `NE`, `LT`, `LE`, `GT`, `GE` | `NEQ`, `LES`, `LEQ`, `GRT`, `GEQ` | Não consultado | `COMP` com `~=`, `<`, `<=`, `>`, `>=` |
| Teste de faixa | Duas comparações em `AND` | `LIM` | Não consultado | Não encontrado |
| Comparação mascarada | Não consultado | `MEQ` | Não consultado | Não encontrado |
| Comparação por expressão | Expressão em ST | `CMP` | Não consultado | Não encontrado |

Fontes: IEC e Hanssen (HANSSEN, 2015); PLC-5 (ROCKWELL AUTOMATION, 1998); Logix (ROCKWELL AUTOMATION, [s. d.]k); WLP (WEG, 2016). **Siemens e Schneider:** a completar.

## Atualizações e comparação com versões mais recentes

Consulta em 21 nov. 2026.

| Fonte citada | Versão mais recente encontrada | Mudança relevante ou observação |
|--------------|--------------------------------|---------------------------------|
| IEC 61131-3, 2ª ed. (2003) | 3ª ed. (2013) e 4ª ed. (2025) (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2013; INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2025). | A PLCopen informa que o Anexo B da 3ª edição inclui **contatos de comparação** no Ladder, tipados e sobrecarregados (PLCOPEN, [s. d.]). Não consegui ler o texto da norma, que é pago. |
| Rockwell, manual do PLC-5 (1998) | Ajuda do Studio 5000 Logix Designer, versão 38.00 (ROCKWELL AUTOMATION, [s. d.]k). | A partir da versão 36 do Logix Designer, o mnemônico mudou de `EQU` para `EQ` (ROCKWELL AUTOMATION, [s. d.]k). Os operandos agora aceitam muitos tipos, inclusive texto, e a comparação numérica exige operandos que não sejam `NaN`. A comparação **não afeta** os flags de matemática (ROCKWELL AUTOMATION, [s. d.]k). |
| WEG, WLP V10.00 (06/2016) | Não encontrei manual do WLP posterior. | Nenhuma mudança verificada (WEG, 2016). |
| Hanssen (2015), Bolton (2015) e Petruzella (2014) | Bolton: 6ª edição é a mais recente segundo a Elsevier (ELSEVIER, 2015). | Não verifiquei edições posteriores de Hanssen e Petruzella. |

## Erros comuns

- Inverter a ordem dos operandos. `A > B` não é o mesmo que `B > A` (HANSSEN, 2015).
- Comparar valores de tipos ou comprimentos diferentes sem conferir as regras da plataforma (HANSSEN, 2015; ROCKWELL AUTOMATION, 1998).
- Esperar que a comparação funcione com `NaN` no Logix. Nesse caso o resultado numérico não é verdadeiro (ROCKWELL AUTOMATION, [s. d.]k).
- Usar uma instrução de matemática ou de transferência onde se queria um teste. Elas são instruções de saída, enquanto a comparação é de entrada (PETRUZELLA, 2014).
- Escolher o operador errado no limite. Maior que (`GRT`) e maior ou igual a (`GEQ`) dão resultados diferentes quando os valores são iguais, como no exemplo de Petruzella com um sinal em `350` ou mais (PETRUZELLA, 2014).

## Exercícios

1. Um bloco `GE` recebe `a = 5`, `b = 3` e `c = 3`. Qual é a saída?

    ??? note "Resposta"
        `(5 >= 3) AND (3 >= 3)` é verdadeiro, então a saída é `TRUE` (HANSSEN, 2015).

2. No PLC-5, uma instrução LIM tem limite baixo 10, limite alto 20 e valor de teste 15. Qual é o estado? E se os limites forem invertidos, 20 e 10?

    ??? note "Resposta"
        Com 10 e 20, é verdadeira, porque 15 está entre os limites. Com 20 e 10, é falsa, porque 15 está entre os dois e não é igual a nenhum (ROCKWELL AUTOMATION, 1998).

3. Reescreva o Exemplo 2 em Ladder com bobinas Set e Reset. Qual dos dois degraus deve vir por último, se a temperatura puder estar em duas faixas ao mesmo tempo?

    ??? note "Resposta"
        As faixas do exemplo não se sobrepõem, então a ordem não importa. Se elas se sobrepusessem, valeria o Reset ou o Set colocado por último, pela regra de que a última escrita vence (ver [ciclo de varredura](../01-fundamentos/ciclo-de-varredura.md)). É uma dedução, e vale testar na plataforma.

4. No CODESYS, implemente os exemplos e varie a variável `temperatura` na simulação.

!!! note "Status de validação"
    Testado em: não testado. Os exemplos aguardam validação no CODESYS. Os diagramas em texto são originais.

Fontes: ver [Referências](../99-referencias/index.md).
