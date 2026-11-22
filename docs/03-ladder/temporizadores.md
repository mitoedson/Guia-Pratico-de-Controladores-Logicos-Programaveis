# Temporizadores (TON, TOF e TP)

## O que é e quando usar

Um temporizador permite mudar o estado de um endereço booleano **depois de um tempo desejado**, contado a partir de uma condição (HANSSEN, 2015). É o "relé de tempo" do CLP, só que em software.

A IEC 61131-3 define três tipos padrão: **TON** (retardo na energização), **TOF** (retardo na desenergização) e **TP** (pulso) (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003; BOLTON, 2015). Use TON para atrasar uma ação, TOF para manter uma saída ligada por um tempo depois que a condição some, e TP para gerar um pulso de duração fixa.

## Como funcionam

Os três blocos têm a mesma interface (HANSSEN, 2015):

- **IN:** entrada booleana que dispara o temporizador. TON e TP partem na borda de subida, e TOF parte na borda de descida.
- **PT:** tempo programado, do tipo `TIME`, como `T#2m30s`.
- **Q:** saída booleana. Seu estado depende do tipo de temporizador.
- **ET:** tempo decorrido, do tipo `TIME`. Quando ET chega a PT, Q muda de estado.

| Tipo | O que acontece com Q | Observação |
|------|----------------------|------------|
| TON | Q liga PT depois que IN ficou verdadeira. IN precisa permanecer verdadeira por pelo menos PT. | Se IN cair antes de PT, Q não liga (HANSSEN, 2015). |
| TOF | Q liga imediatamente quando IN fica verdadeira e desliga PT depois que IN caiu. | Se IN voltar a ser verdadeira antes de PT, ET é zerado (HANSSEN, 2015). |
| TP | Q liga quando IN sobe e permanece ligada por PT, mesmo que IN caia nesse intervalo. | ET só zera quando IN e Q estão em nível baixo (HANSSEN, 2015). |

Os diagramas de tempo da norma mostram os três comportamentos (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003). A norma também diz que o efeito de mudar PT durante a contagem, por exemplo zerá-lo para parar um TP, **depende da implementação** (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003).

Na biblioteca padrão do CODESYS descrita pela ABB, o TON começa a contar na borda de subida de IN e reinicia a contagem na borda de descida, com Q falsa quando IN é falsa. O TP mantém ET constante depois de atingir PT (ABB, 2025a; ABB, 2025b).

### Um bloco por temporizador

Cada temporizador precisa da **própria instância** do bloco de função, declarada com a mesma sintaxe de uma variável, mas indicando o tipo do bloco (HANSSEN, 2015). O guideline da Siemens recomenda usar os temporizadores e contadores IEC como **multi-instâncias**, em vez de memórias dependentes de hardware (SIEMENS, 2018).

## Exemplo 1: TON, atraso na partida

Uma bomba parte 5 s depois do comando. O exemplo é original, no estilo de Hanssen (2015).

```pascal
PROGRAM Atraso
VAR
    comando      AT %IX0.0 : BOOL;
    atraso_bomba : TON;
    bomba        AT %QX0.0 : BOOL;
END_VAR

atraso_bomba(IN := comando, PT := T#5S);
bomba := atraso_bomba.Q;
```

Em Ladder, o bloco TON recebe `comando` em IN e `T#5S` em PT, e `Q` aciona a bobina `bomba`:

```text
 |   comando     +------------+                           |
 +----] [--------| IN       Q |------------------( )-----+
 |               |    TON     |                  bomba    |
 |    T#5S ------| PT      ET |                           |
 |               +------------+                           |
```

Se `comando` ficar ligado por menos de 5 s, `bomba` nunca liga (HANSSEN, 2015).

## Exemplo 2: TOF, exaustor que continua ligado

O exaustor segue ligado por 10 s depois que a máquina desliga.

```pascal
atraso_exaustor(IN := maquina_ligada, PT := T#10S);
exaustor := atraso_exaustor.Q;
```

`exaustor` liga junto com `maquina_ligada` e só desliga 10 s depois da queda, a menos que a máquina volte a ligar antes (HANSSEN, 2015).

## Exemplo 3: TP, pulso de duração fixa

A buzina soa por 500 ms a cada acionamento do botão.

```pascal
pulso_buzina(IN := botao_alarme, PT := T#500MS);
buzina := pulso_buzina.Q;
```

A saída dura PT mesmo que o botão seja solto antes (HANSSEN, 2015).

## Exemplo 4: pisca-pisca com dois TON

Bolton (2015) mostra um temporizador de ciclo liga e desliga feito com dois temporizadores de retardo. Esta é uma adaptação em ST, com 5 s ligado e 5 s desligado.

```pascal
t_desligado(IN := NOT t_ligado.Q, PT := T#5S);   // conta o tempo desligado
t_ligado(IN := t_desligado.Q, PT := T#5S);       // conta o tempo ligado
pisca := t_desligado.Q;
```

Funcionamento:

1. Com `t_ligado.Q` falso, `t_desligado` conta 5 s. A lâmpada está desligada.
2. Ao fim dos 5 s, `t_desligado.Q` liga a lâmpada e dispara `t_ligado`.
3. Passados mais 5 s, `t_ligado.Q` fica verdadeiro, `t_desligado` perde a entrada e desliga a lâmpada, e o ciclo recomeça.

!!! note "Dependência da ordem de execução"
    Este circuito usa o valor de `t_ligado.Q` calculado no ciclo anterior para alimentar `t_desligado`. O funcionamento depende da execução sequencial descrita no [ciclo de varredura](../01-fundamentos/ciclo-de-varredura.md).

## Temporizador retentivo (RTO)

O TON zera ao perder a entrada. O **temporizador retentivo** acumula o tempo enquanto habilitado e **mantém o acumulado** quando a habilitação cai. Ele só é zerado por uma instrução de reset (BOLTON, 2015).

No manual do PLC-5, o RTO preserva o valor acumulado mesmo quando o degrau fica falso, o processador muda para modo Program, falha ou perde energia. A contagem continua quando o processador volta ou o degrau volta a ser verdadeiro, e o bit de concluído (`.DN`) só é zerado pela instrução RES (ROCKWELL AUTOMATION, 1998).

## Equivalências entre plataformas

| Plataforma | Temporizadores | Unidade e estrutura | Observações |
|------------|----------------|---------------------|-------------|
| IEC 61131-3 / CODESYS | TON, TOF, TP | Entradas IN e PT, saídas Q e ET, tempo do tipo `TIME` (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003). | A ajuda da ABB documenta os blocos TON e TP da biblioteca padrão do CODESYS (ABB, 2025a; ABB, 2025b). |
| Rockwell PLC-5 | TON, TOF, RTO | Palavra de controle com bits EN, TT e DN, preset e acumulado de 16 bits, base de tempo de 1 s ou 0,01 s (ROCKWELL AUTOMATION, 1998). | Com base de 1 s, o acumulado vai até 32.767 intervalos, cerca de 9,1 horas (ROCKWELL AUTOMATION, 1998). |
| Rockwell Logix (Studio 5000) | TON, RTO e outros | Estrutura `TIMER` com `.PRE` e `.ACC` em milissegundos e bits `.EN`, `.TT` e `.DN` (ROCKWELL AUTOMATION, [s. d.]f). | A TON é não retentiva e pode ser pausada ao se ligar o bit `.DN` (ROCKWELL AUTOMATION, [s. d.]f). A base de tempo é sempre de 1 ms (ROCKWELL AUTOMATION, [s. d.]g). |
| WEG WLP | TON | Entrada IN, saída Q, argumentos PT e ET. O PT constante vai até 30.000 ms (WEG, 2016). | Com IN em 0, Q fica em 0 e o ET **permanece inalterado**. Com IN em 1, o ET é zerado e passa a contar (WEG, 2016). |
| Siemens S7-1200/1500 | TP, TON, TOF | Instâncias como multi-instância (SIEMENS, 2018). | Em programas de segurança, o guideline manda usar TP, TON e TOF o mínimo possível, porque cada um exige blocos adicionais (SIEMENS, 2018). |

Uma diferença importante: no WLP, o ET **não zera** quando IN cai, ao contrário do que a norma mostra para o TON (WEG, 2016; INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003).

Siemens e Schneider: detalhes dos blocos a completar.

## Atualizações e comparação com versões mais recentes

Consulta em 21 nov. 2026.

| Fonte citada | Versão mais recente encontrada | Mudança relevante ou observação |
|--------------|--------------------------------|---------------------------------|
| IEC 61131-3, 2ª ed. (2003) | 3ª ed. (2013) e 4ª ed. (2025) (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2013; INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2025). | A PLCopen informa que o Anexo B da 3ª edição lista novos tipos de dados elementares, contatos de comparação no Ladder e a **obsolescência da Instruction List (IL)** (PLCOPEN, [s. d.]). Não consegui conferir se mudou o comportamento de TP, TON e TOF, porque o texto da norma é pago. |
| Rockwell, manual do PLC-5 (1998) | Ajuda do Studio 5000 Logix Designer, versão 38.00 (ROCKWELL AUTOMATION, [s. d.]f; ROCKWELL AUTOMATION, [s. d.]g). | O temporizador deixou de ter base de 1 s ou 0,01 s: a base é **sempre de 1 ms**, e `.PRE` e `.ACC` são `DINT`. A versão 38.00 traz também a estrutura `TIMER_T`, com `.ACC` e `.PRE` do tipo `TIME` e fidelidade de microssegundo (ROCKWELL AUTOMATION, [s. d.]f). O tempo acumulado é calculado pela diferença entre o horário atual e o da última varredura (ROCKWELL AUTOMATION, [s. d.]f). |
| CODESYS: manual 2.3 (2007) e Hanssen (2015) | Biblioteca padrão 2.9.0, na ajuda da ABB (ABB, 2025a; ABB, 2025b). | Os blocos TON e TP continuam documentados na biblioteca padrão 2.9.0, com IN, PT, Q e ET. Não comparei o comportamento com o do CODESYS 2.3. |
| WEG, WLP V10.00 (06/2016) | Não encontrei manual do WLP posterior. | Nenhuma mudança verificada (WEG, 2016). |
| Siemens, Programming Guideline V1.6 (12/2018) | Não encontrei versão posterior do guideline. | Confira na ajuda do TIA Portal em uso. |
| Petruzella, 4ª ed. (2014) | Não verifiquei se existe edição posterior. | |

## Erros comuns

- Compartilhar a mesma instância de temporizador em mais de um ponto do programa. Cada temporizador precisa da sua (HANSSEN, 2015).
- Esperar que um TON ligue com uma condição que dura menos que PT. Nesse caso Q nunca liga (HANSSEN, 2015).
- Alterar PT durante a contagem esperando um comportamento específico. A norma deixa isso a cargo da implementação (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003).
- Usar RTO sem uma instrução de reset. O bit de concluído só desliga com o RES (ROCKWELL AUTOMATION, 1998).
- Deixar um temporizador Logix em uma parte do código que pode deixar de ser executada, como sub-rotina ou trecho entre JMP e LBL. A ajuda do RTOR alerta que o temporizador precisa ser varrido pelo menos a cada 69 minutos enquanto roda, ou o valor acumulado fica incorreto (ROCKWELL AUTOMATION, [s. d.]h).
- Supor que o ET zera quando IN cai em qualquer plataforma. No WLP ele permanece inalterado (WEG, 2016).
- Esquecer a diferença de unidade: PLC-5 usa base de 1 s ou 0,01 s, e Logix usa milissegundos (ROCKWELL AUTOMATION, 1998; ROCKWELL AUTOMATION, [s. d.]g).

## Exercícios

1. Um TON tem `PT := T#3S`. `IN` fica verdadeira por 2 s, cai, e depois fica verdadeira por 4 s. Quando `Q` liga?

    ??? note "Resposta"
        No primeiro pulso, `Q` não liga, porque `IN` durou menos que PT. No segundo, `Q` liga 3 s depois da subida e permanece ligada até `IN` cair (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003; HANSSEN, 2015).

2. Um TOF tem `PT := T#5S`. `IN` fica verdadeira por 10 s e cai. Se `IN` voltar a ser verdadeira 2 s depois de cair, o que acontece com `Q`?

    ??? note "Resposta"
        `Q` permanece ligada, e o tempo decorrido é zerado quando `IN` volta a ser verdadeira (HANSSEN, 2015).

3. Um TP tem `PT := T#2S`. `IN` fica verdadeira por 0,5 s. Por quanto tempo `Q` fica ligada?

    ??? note "Resposta"
        Por 2 s, mesmo com `IN` caindo antes (HANSSEN, 2015).

4. No Exemplo 4, o que acontece se os dois `PT` forem diferentes, 2 s e 8 s?

    ??? note "Resposta"
        A lâmpada fica desligada pelo `PT` de `t_desligado` e ligada pelo `PT` de `t_ligado`, por dedução do funcionamento descrito. Teste no CODESYS.

5. No CODESYS, implemente os Exemplos 1 a 4 e observe `ET` de cada instância no modo de simulação.

!!! note "Status de validação"
    Testado em: não testado. Os exemplos aguardam validação no CODESYS. Os diagramas em texto são originais.

Fontes: ver [Referências](../99-referencias/index.md).
