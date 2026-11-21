# Contatos e bobinas

## O que é e quando usar

O Ladder (LD) representa a lógica como um circuito de relés. Na norma IEC 61131-3, cada rede (degrau) fica entre um trilho de energia à esquerda e outro à direita, e o trilho esquerdo é considerado sempre ligado (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003). O "fluxo de energia" é apenas simbólico: cada elo entre elementos está ligado (1) ou desligado (0), e o elo horizontal repassa à direita o estado que recebe da esquerda (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003).

Dois elementos formam a base de qualquer programa:

- **Contato:** testa o estado de uma variável booleana, como uma entrada. Conduz ou não conduz, conforme o sinal (CODESYS GROUP, [s. d.]b).
- **Bobina:** grava o resultado da lógica à esquerda em uma variável booleana, como uma saída. Funciona de forma parecida com um operador de atribuição (CODESYS GROUP, [s. d.]b).

Contatos em série formam um **E** (AND), e contatos em ramos paralelos formam um **OU** (OR). A lógica é lida da esquerda para a direita, e as redes são processadas de cima para baixo (CODESYS GROUP, [s. d.]b), o que se conecta ao [ciclo de varredura](../01-fundamentos/ciclo-de-varredura.md).

### Contato "normalmente aberto" não é o mesmo que botão "normalmente aberto"

Os nomes NA e NF descrevem o **teste do bit**, não o dispositivo ligado à entrada. Petruzella (2014) recomenda separar a ação do dispositivo de campo da ação do CLP: sinal presente na entrada gera bit 1, e ausência de sinal gera bit 0. Em um de seus exemplos, a instrução XIC do botão de parada é verdadeira porque o botão de parada, fisicamente normalmente fechado, está fechado e o bit vale 1 (PETRUZELLA, 2014).

Na prática, o símbolo depende de **como o bit se comporta**, e não de como o botão é chamado.

## Elementos básicos da IEC 61131-3

| Elemento | Símbolo | Comportamento |
|----------|---------|---------------|
| Contato normalmente aberto (NA) | <code>--&#124; &#124;--</code> | Copia o estado do elo da esquerda para o da direita se a variável estiver ligada. Caso contrário, o elo da direita fica desligado. |
| Contato normalmente fechado (NF) | <code>--&#124;/&#124;--</code> | Copia o estado da esquerda para a direita se a variável estiver desligada. |
| Contato de borda de subida | <code>--&#124;P&#124;--</code> | O elo da direita fica ligado, de uma avaliação do elemento até a seguinte, quando se detecta transição de desligado para ligado da variável com o elo da esquerda ligado. Nos demais momentos fica desligado. |
| Contato de borda de descida | <code>--&#124;N&#124;--</code> | Igual ao anterior, para a transição de ligado para desligado. |
| Bobina | `--( )--` | Copia o estado da esquerda para a variável associada e para o elo da direita. |
| Bobina negada | `--(/)--` | Copia o estado da esquerda para o elo da direita e grava na variável o estado inverso. |
| Bobina Set (travamento) | `--(S)--` | Liga a variável quando o elo da esquerda está ligado, e ela permanece ligada até um Reset. |
| Bobina Reset (destravamento) | `--(R)--` | Desliga a variável quando o elo da esquerda está ligado, e ela permanece desligada até um Set. |
| Bobinas de borda | `--(P)--` e `--(N)--` | A variável fica ligada de uma avaliação à seguinte quando se detecta transição do elo da esquerda, de desligado para ligado (P) ou de ligado para desligado (N). O estado da esquerda sempre é copiado para a direita. |

Fonte da tabela: IEC 61131-3, 2ª edição (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003). Hanssen (2015) apresenta os mesmos elementos e informa que as bobinas Set e Reset correspondem aos blocos SR e RS, enquanto os contatos P e N correspondem aos blocos R_TRIG e F_TRIG.

## Exemplo 1: acionamento direto

Uma bomba liga quando o sensor de nível alto está ativo. O exemplo é original, no estilo do primeiro programa de Hanssen (2015).

```text
 |   sensor_nivel                          bomba    |
 +-----------] [------------------------------( )--+
```

Equivalente em ST: `bomba := sensor_nivel;`. A bobina se comporta como uma atribuição (CODESYS GROUP, [s. d.]b).

## Exemplo 2: partida e parada com selo

Um motor parte com um botão de pulso e só para quando se aciona o botão de parada. Para o motor continuar ligado depois de soltar a partida, o circuito usa um contato da própria saída em paralelo com o botão, chamado **selo** (BOLTON, 2015; PETRUZELLA, 2014).

```text
 |        partida           parada                motor    |
 +--+--------] [--------+-----]/[--------------------( )--+
    |                   |
    |         motor     |
    +--------] [--------+
```

Neste desenho, `parada` usa contato NF porque se supõe um botão de parada que **liga o bit quando é pressionado**. Se o botão de parada for fisicamente NF, o bit vale 1 em repouso e o contato correto é NA, como no exemplo de Petruzella (2014) citado acima.

Funcionamento, segundo Bolton (2015):

1. Ao acionar a partida, a saída liga.
2. Um segundo contato da saída fecha e forma um OU com o contato de partida.
3. Mesmo que a partida seja solta, o circuito mantém a saída ligada.
4. A única forma de liberar a saída é abrir o contato NF da parada.

### Mesma função com Set e Reset

```text
 |   partida                                    motor   |
 +------------] [-------------------------------(S)----+
 |   parada                                     motor   |
 +------------] [-------------------------------(R)----+
```

Hanssen (2015) mostra essa solução e avisa que a ordem dos degraus pode mudar o funcionamento. Pela execução sequencial e pela regra de que a última escrita vence (ver [ciclo de varredura](../01-fundamentos/ciclo-de-varredura.md)), o Reset colocado depois do Set faz a parada prevalecer quando os dois botões estão acionados. Isso é uma dedução, e vale conferir na plataforma usada. Em blocos, Hanssen (2015) informa que o bloco RS tem reset dominante e o SR tem set dominante.

!!! warning "Retenção na queda de energia"
    No manual do PLC-5, a instrução OTL (travamento) é retentiva: quando o CLP perde energia, muda para modo Program ou detecta falha grave, as saídas desligam, mas o estado das instruções retentivas fica na memória. Ao voltar para RUN, as saídas retentivas retornam imediatamente ao estado anterior, enquanto as não retentivas, como a OTE, são zeradas (ROCKWELL AUTOMATION, 1998). Por consequência, um motor travado por OTL volta a ligar sozinho quando o CLP retorna ao modo RUN.

## Bobina dupla

Hanssen (2015) descreve como erro típico de iniciantes escrever duas bobinas com o mesmo endereço. Exemplo do autor: duas chaves ligadas a %IX1.1 e %IX1.2 e uma saída %QX2.0, cada uma em um degrau próprio. Com apenas a chave 1 acionada, o primeiro degrau liga a saída na memória, mas o segundo a desliga logo em seguida. As saídas físicas só são atualizadas depois da execução completa, então a saída permanece desligada (HANSSEN, 2015).

A solução é reunir as condições em **uma só bobina**, com os contatos em paralelo (OU). Na ajuda do CODESYS LD FBD hospedada pela ABB, várias bobinas em uma mesma rede só podem ficar em paralelo (ABB, 2026).

## Detecção de borda

Um contato comum conduz **enquanto** o bit está ligado, e isso se repete a cada ciclo. Para agir **uma vez** por acionamento, usa-se um contato ou uma bobina de borda, que fica ativo por um único ciclo (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003; HANSSEN, 2015). Hanssen (2015) informa que os contatos P e N não existem no CODESYS 2.3.

## Equivalências entre plataformas

| Função | IEC 61131-3 / CODESYS | Rockwell (PLC-5 e Logix) | WEG WLP |
|--------|-----------------------|--------------------------|---------|
| Contato NA | Normally open contact | XIC | NO CONTACT |
| Contato NF | Normally closed contact | XIO | NC CONTACT |
| Bobina | Coil | OTE | COIL |
| Bobina negada | Negated coil | Não encontrada nas fontes consultadas | NEG COIL |
| Travar (Set) | SET coil | OTL | SET COIL |
| Destravar (Reset) | RESET coil | OTU | RESET COIL |
| Borda de subida | Positive transition-sensing contact ou coil | ONS e OSR | PTS COIL |
| Borda de descida | Negative transition-sensing contact ou coil | OSF | NTS COIL |

Fontes: IEC e Hanssen, nas seções anteriores; Rockwell (ROCKWELL AUTOMATION, 1998; ROCKWELL AUTOMATION, [s. d.]e); WEG (WEG, 2016). Observações:

- **Rockwell:** o PLC-5 descreve XIC e XIO como instruções que examinam um bit ligado ou desligado, e OTE como a que liga o bit se o degrau for verdadeiro e o desliga se for falso, semelhante a uma bobina de relé (ROCKWELL AUTOMATION, 1998). A página atual das instruções de bit lista as funções equivalentes, inclusive acionar um bit por um ciclo quando o degrau fica verdadeiro ou falso (ROCKWELL AUTOMATION, [s. d.]e).
- **WEG WLP:** entre os elementos listados no manual há apenas dois contatos (NO e NC). A detecção de borda é feita por bobinas (PTS e NTS), que ligam o bit por um ciclo de varredura (WEG, 2016). Existe ainda a **bobina imediata**, que grava a saída no momento da execução, e não no fim do ciclo (WEG, 2016).
- **Siemens e Schneider:** a completar.

## Atualizações e comparação com versões mais recentes

Consulta em 21 nov. 2026.

| Fonte citada | Versão mais recente encontrada | Mudança relevante ou observação |
|--------------|--------------------------------|---------------------------------|
| IEC 61131-3, 2ª ed. (2003) | 3ª ed. (2013) e 4ª ed. (2025) (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2013; INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2025). | O texto das edições novas é pago. Não consegui conferir se mudou a definição de contatos e bobinas. |
| Rockwell, manual do PLC-5 (1998) | Ajuda do Studio 5000 Logix Designer, versão 38.00 (ROCKWELL AUTOMATION, [s. d.]b; ROCKWELL AUTOMATION, [s. d.]c; ROCKWELL AUTOMATION, [s. d.]d). | O operando passou a ser uma **tag BOOL**, e não um endereço da tabela de imagem como `I:012/07`. A OTE passou a **limpar o bit no pré-scan e no pós-scan**. A OTL mantém o bit ligado até ele ser limpo, tipicamente por uma OTU. Não verifiquei o comportamento da retenção em queda de energia nos controladores atuais. Consulte o manual do controlador. |
| CODESYS 2.3 (3S-SMART SOFTWARE SOLUTIONS, 2007) e Hanssen (2015) | Ajuda do CODESYS 4 (CODESYS GROUP, [s. d.]b; CODESYS GROUP, [s. d.]c). | Hanssen informa que os contatos P e N não existem no CODESYS 2.3. A ajuda do CODESYS 4 descreve contato com detecção de borda de subida e de descida. Essa ajuda é do CODESYS 4. Confira os recursos do LD no CODESYS V3.5 em uso. |
| WEG, WLP V10.00 (06/2016) | Não encontrei manual do WLP posterior. | Nenhuma mudança verificada (WEG, 2016). |
| Bolton, 6ª ed. (2015) | A 6ª edição é a mais recente segundo a Elsevier (ELSEVIER, 2015). | Nenhuma mudança verificada. |
| Hanssen (2015) e Petruzella, 4ª ed. (2014) | Não verifiquei se existem edições posteriores. | |

## Erros comuns

- Escrever duas bobinas para a mesma variável (HANSSEN, 2015).
- Usar o nome do botão (NA ou NF) para escolher o contato, em vez de observar o valor do bit (PETRUZELLA, 2014).
- Usar Set sem prever um Reset. O PLC-5 descreve OTL e OTU como instruções normalmente usadas em par para o mesmo bit (ROCKWELL AUTOMATION, 1998).
- Esquecer que uma saída travada por instrução retentiva pode voltar ligada quando o CLP retorna ao modo RUN (ROCKWELL AUTOMATION, 1998).
- Usar um contato comum quando se quer agir uma única vez. Nesse caso, use detecção de borda.
- Esquecer que a mesma entrada pode precisar de uma instância separada de `R_TRIG` para cada uso, quando a borda é feita por bloco (HANSSEN, 2015).

## Exercícios

1. Desenhe o circuito de partida e parada com selo para um botão de parada fisicamente NF. Qual contato representa a parada?

    ??? note "Resposta"
        Contato NA. Com o botão NF em repouso, o bit vale 1, e o contato NA conduz (PETRUZELLA, 2014). Ao pressionar o botão, o bit cai para 0 e o contato abre.

2. Duas chaves, %IX1.1 e %IX1.2, devem ligar a saída %QX2.0 quando qualquer uma estiver acionada. Corrija o programa com duas bobinas.

    ??? note "Resposta"
        Uma única bobina em %QX2.0, com os contatos de %IX1.1 e %IX1.2 em paralelo (OU), como na solução de Hanssen (2015).

3. Reescreva o selo usando Set e Reset. O que acontece se partida e parada forem acionadas juntas, com o Reset depois do Set?

    ??? note "Resposta"
        A saída fica desligada, porque a última escrita do ciclo é o Reset (dedução a partir da execução sequencial).

4. No CODESYS, crie o selo em LD e depois a versão com Set e Reset. Compare o comportamento no modo de simulação.

!!! note "Status de validação"
    Testado em: não testado. Os exemplos aguardam validação no CODESYS. Os diagramas em texto são originais.

Fontes: ver [Referências](../99-referencias/index.md).
