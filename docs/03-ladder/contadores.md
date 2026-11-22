# Contadores (CTU, CTD e CTUD)

## O que é e quando usar

Um contador conta ocorrências de um sinal de entrada, como itens que passam por uma esteira, voltas de um eixo ou pessoas que cruzam uma porta (BOLTON, 2015). Ele se parece com um temporizador, com a diferença de que não conta pulsos de um relógio interno: depende de pulsos externos ou de um programa-fonte (PETRUZELLA, 2014).

A IEC 61131-3 define três blocos: **CTU** (contador crescente), **CTD** (decrescente) e **CTUD** (crescente e decrescente) (HANSSEN, 2015; INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003).

## Como funcionam

Todos os contadores de CLP contam na **borda de subida** do sinal de entrada (PETRUZELLA, 2014; HANSSEN, 2015). A norma define o funcionamento por corpos de função em ST (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003):

| Bloco | Entradas | Saídas | Regra |
|-------|----------|--------|-------|
| CTU | `CU`, `R`, `PV` | `Q`, `CV` | `R` zera `CV`. Se não houver reset, cada borda de `CU` soma 1 a `CV`. `Q` liga quando `CV >= PV`. |
| CTD | `CD`, `LD`, `PV` | `Q`, `CV` | `LD` carrega `CV` com `PV`. Se não houver carga, cada borda de `CD` subtrai 1 de `CV`. `Q` liga quando `CV <= 0`. |
| CTUD | `CU`, `CD`, `R`, `LD`, `PV` | `QU`, `QD`, `CV` | `R` tem prioridade sobre `LD`, e `LD` sobre a contagem. Se `CU` e `CD` estiverem ativas juntas, `CV` não muda. `QU` liga quando `CV >= PV`, e `QD` quando `CV <= 0`. |

Os limites `PVmin` e `PVmax` **dependem da implementação** (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003).

Contadores crescentes sempre são reiniciados para zero. Os decrescentes podem ser reiniciados para zero ou para outro valor pré-ajustado (PETRUZELLA, 2014).

## Exemplo 1: CTU, contar peças e acionar um alarme

Um alarme liga quando 30 itens passam por uma fotocélula, e um botão reinicia a contagem. O exemplo é original, no estilo do exemplo de contagem de itens de Hanssen (2015).

```pascal
PROGRAM ContaPecas
VAR
    fotocelula AT %IX1.0 : BOOL;
    reinicio   AT %IX1.1 : BOOL;
    alarme     AT %QX2.1 : BOOL;
    contador   : CTU;
END_VAR

contador(CU := fotocelula, RESET := reinicio, PV := 30);
alarme := contador.Q;
```

```text
 |  fotocelula   +------------+                          |
 +-----] [--------| CU       Q |---------------( )-------+
 |  reinicio     |            |                alarme    |
 +-----] [--------| RESET      |                          |
 |           30 --| PV     CV  |                          |
 |               |    CTU     |                          |
 |               +------------+                          |
```

Na norma, a entrada de reinício chama-se `R`. Na biblioteca padrão do CODESYS descrita pela ABB, chama-se `RESET`, e `PV` e `CV` são do tipo `WORD`, enquanto a norma usa `INT` (ABB, 2025c; INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003). Confira os nomes e tipos na biblioteca da sua plataforma.

## Exemplo 2: CTD, magazine que esvazia

Um magazine começa com 10 peças. Cada retirada diminui a contagem, e o aviso de vazio liga quando o contador chega a zero. A recarga carrega o valor de volta.

```pascal
magazine(CD := sensor_retirada, LD := recarga, PV := 10);
vazio := magazine.Q;
```

Segundo a norma, `Q` liga quando `CV <= 0` (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003).

## Exemplo 3: CTUD, vagas em um estacionamento

Bolton (2015) descreve contar carros que entram e saem de um estacionamento e acender um aviso quando o número de vagas se esgota. Esta é uma adaptação para 50 vagas.

```pascal
vagas(CU := entrada_carro, CD := saida_carro, R := reinicio, LD := FALSE, PV := 50);
lotado := vagas.QU;
```

Com 50 carros dentro, `QU` liga e acende o aviso de lotado. Quando um carro sai, `CV` cai para 49 e `QU` desliga (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003).

## Contadores em cascata

Para contar eventos além do máximo de um contador, liga-se a saída do primeiro à entrada do segundo. Com dois contadores de preset 500 em cascata, a saída final liga depois de 500 × 500 = 250.000 pulsos na entrada (PETRUZELLA, 2014).

## Equivalências entre plataformas

| Função | IEC 61131-3 / CODESYS | Rockwell (PLC-5 e Logix) | WEG WLP |
|--------|-----------------------|--------------------------|---------|
| Contar para cima | CTU | CTU | CTU |
| Contar para baixo | CTD | CTD | Não encontrado entre os blocos listados |
| Subida e descida | CTUD | CTU e CTD com o mesmo endereço | Não encontrado entre os blocos listados |
| Reset | Entrada `R`, ou `RESET` na biblioteca da ABB | Instrução RES separada | Entrada `R` |
| Saída de concluído | `Q` quando `CV >= PV` no CTU | Bit `.DN` | `Q` quando atinge o valor |
| Tipo e limites | `INT` na norma e `WORD` na biblioteca da ABB | PLC-5: 16 bits, de −32.768 a +32.767. Logix: `DINT` | Constante de PV até 30.000 |

Fontes: IEC (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003) e ABB (2025c). Rockwell (ROCKWELL AUTOMATION, 1998; ROCKWELL AUTOMATION, [s. d.]i). WEG (WEG, 2016). Observações:

- **PLC-5:** o CTU conta de −32.768 a +32.767 a cada transição de falso para verdadeiro do degrau, com bits `.CU`, `.DN` e `.OV` (estouro, quando o valor passa de +32.767 e volta a −32.768). O CTD tem o bit `.UN` (ROCKWELL AUTOMATION, 1998).
- **Diferença no `.DN` do CTD:** no PLC-5, o `.DN` do CTD fica ligado quando o acumulado é **maior ou igual** ao preset (ROCKWELL AUTOMATION, 1998). Na norma, o `Q` do CTD liga quando `CV <= 0` (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003). A lógica é **diferente**, mesmo com nomes parecidos.
- **RES:** zera o valor acumulado e os bits de status do contador. Em contador com o degrau habilitado, os bits `.CU` ou `.CD` ficam zerados enquanto o RES estiver ativo (ROCKWELL AUTOMATION, 1998).
- **WEG WLP:** o CTU tem entradas `CU` e `R` e saída `Q`. `Q` liga ao atingir o preset e só desliga quando `R` vai a 1. Enquanto `R` está em 1, o contador fica zerado e não conta (WEG, 2016). O WLP tem ainda blocos de contagem de pulsos de encoder (CTENC) (WEG, 2016).
- **Siemens e Schneider:** a completar.

## Atualizações e comparação com versões mais recentes

Consulta em 21 nov. 2026.

| Fonte citada | Versão mais recente encontrada | Mudança relevante ou observação |
|--------------|--------------------------------|---------------------------------|
| IEC 61131-3, 2ª ed. (2003) | 3ª ed. (2013) e 4ª ed. (2025) (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2013; INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2025). | O texto das edições novas é pago. Não consegui conferir se mudou o comportamento de CTU, CTD e CTUD. |
| Rockwell, manual do PLC-5 (1998) | Ajuda do Studio 5000 Logix Designer, versão 38.00 (ROCKWELL AUTOMATION, [s. d.]i). | O contador continua contando as transições de falso para verdadeiro do degrau. `.PRE` e `.ACC` agora são `DINT` (32 bits), e não mais 16 bits (ROCKWELL AUTOMATION, [s. d.]i). |
| CODESYS: manual 2.3 (2007) e Hanssen (2015) | Biblioteca padrão 2.9.0, na ajuda da ABB (ABB, 2025c). | `PV` e `CV` do CTU são `WORD`, e o reset se chama `RESET` (ABB, 2025c). |
| WEG, WLP V10.00 (06/2016) | Não encontrei manual do WLP posterior. | Nenhuma mudança verificada (WEG, 2016). |
| Petruzella, 4ª ed. (2014) | Não verifiquei se existe edição posterior. | |

## Erros comuns

- Esperar contagem por nível. Os contadores da norma contam na **borda de subida** (HANSSEN, 2015). Em algumas bibliotecas isso não vale: a ajuda do Kollmorgen KAS 3.05 informa que o CTUD aumenta ou diminui a cada chamada em que a entrada está verdadeira, e manda usar `R_TRIG` ou `F_TRIG` para contar pulsos (KOLLMORGEN, [s. d.]).
- Esquecer o reset. O contador crescente precisa ser reiniciado para zero, e o PLC-5 usa uma instrução RES com o mesmo endereço do contador (PETRUZELLA, 2014; ROCKWELL AUTOMATION, 1998).
- Ignorar o estouro. No PLC-5, o contador dá a volta ao passar de +32.767 e liga o bit `.OV` (ROCKWELL AUTOMATION, 1998).
- Colocar um contador crítico em zona MCR ou em trecho de programa saltado. O manual do PLC-5 manda evitar isso, para impedir resultados inválidos que possam danificar equipamentos ou ferir pessoas (ROCKWELL AUTOMATION, 1998).
- Confundir o `.DN` do CTD do PLC-5 com o `Q` do CTD da norma (ROCKWELL AUTOMATION, 1998; INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003).
- Tentar contar sinais mais rápidos que o ciclo de varredura. Para esses casos, o WLP oferece blocos de contagem de pulsos de encoder, e a literatura cita módulos especiais (WEG, 2016; BOLTON, 2015). Ver [ciclo de varredura](../01-fundamentos/ciclo-de-varredura.md).
- Declarar contadores retentivos sem multi-instância. O guideline da Siemens recomenda programar contadores e temporizadores como multi-instância, inclusive os retentivos (SIEMENS, 2018).
- Esperar retenção em queda de energia sem conferir a plataforma. O CTU do SLC 500, descrito por Petruzella (2014), mantém o acumulado em um ciclo de desligar e ligar. Em outras plataformas isso pode ser diferente.

## Exercícios

1. Um CTU tem `PV := 4`, começa em zero e recebe 6 bordas de subida em `CU`. Quando `Q` liga, e quanto vale `CV` no final?

    ??? note "Resposta"
        `Q` liga na 4ª borda. Sem reset, `CV` chega a 6 e `Q` continua ligada, conforme o corpo da função na norma (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003).

2. Um CTD recebe `LD` com `PV := 3`. Depois ocorrem 3 bordas de subida em `CD`. Qual é o valor de `Q`?

    ??? note "Resposta"
        `LD` carrega `CV = 3`. Depois das 3 bordas, `CV = 0` e `Q` liga (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003).

3. Um CTUD com `PV := 50` recebe 50 entradas, depois 3 saídas e depois 2 entradas. Quanto valem `CV`, `QU` e `QD` no final?

    ??? note "Resposta"
        `CV` = 50 − 3 + 2 = 49. `QU` fica desligada, porque 49 é menor que 50, e `QD` também, porque `CV` não é zero (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003).

4. Quantos pulsos são necessários para ligar a saída de dois contadores em cascata, ambos com preset 500?

    ??? note "Resposta"
        500 × 500 = 250.000 pulsos (PETRUZELLA, 2014).

5. No CODESYS, implemente os Exemplos 1 a 3. Gere as bordas de subida pelo próprio contador e teste o que acontece com o sensor mantido ligado.

!!! note "Status de validação"
    Testado em: não testado. Os exemplos aguardam validação no CODESYS. Os diagramas em texto são originais.

Fontes: ver [Referências](../99-referencias/index.md).
