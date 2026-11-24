# Fluxo do programa

## O que é e quando usar

Em condições normais, o CLP executa o programa na ordem em que ele foi escrito (HANSSEN, 2015). As instruções de controle de fluxo permitem **executar seções da lógica apenas quando certas condições são atendidas**, o que dá mais flexibilidade e eficiência à varredura. Partes do programa que não estão em uso podem ser puladas, e as saídas de áreas específicas podem ficar nos estados desejados (PETRUZELLA, 2014). Essas instruções também servem para habilitar ou desabilitar um bloco de lógica e para mover a execução de um lugar para outro (PETRUZELLA, 2014).

As principais são (PETRUZELLA, 2014):

- **JMP** e **LBL:** salto para um rótulo, para a frente ou para trás.
- **JSR**, **SBR** e **RET:** chamada de sub-rotina, cabeçalho da sub-rotina e retorno.
- **MCR:** relé mestre de controle, que desliga as saídas não retentivas de uma zona.
- **TND:** fim temporário do programa.
- **SUS:** suspensão, usada para depuração e verificação de defeitos.

Saltar para a frente sobre um trecho também economiza tempo de varredura, porque o trecho deixa de ser executado até ser necessário (ROCKWELL AUTOMATION, 1998; ROCKWELL AUTOMATION, [s. d.]p).

## Três ferramentas, três efeitos

| Ferramenta | O que faz | Saídas na região | Temporizadores e contadores |
|------------|-----------|------------------|-----------------------------|
| JMP e LBL | Pula um trecho. | Todas ficam no **último estado** (PETRUZELLA, 2014). | Param de ser atualizados (PETRUZELLA, 2014). |
| MCR | Trata o trecho como falso. | As **não retentivas** desligam, e as retentivas ficam no último estado (PETRUZELLA, 2014). | A operação cessa quando a zona é desabilitada, e um TOF é ativado (PETRUZELLA, 2014). |
| Sub-rotina | Executa outra rotina e retorna (ROCKWELL AUTOMATION, 1998). | Definidas pela lógica da sub-rotina. | A rotina executa a cada vez que é varrida (ROCKWELL AUTOMATION, [s. d.]o). |

Petruzella (2014) resume a diferença principal: a MCR desliga as saídas não retentivas e mantém as retentivas, e o JMP deixa todas as saídas em seu último estado.

## JMP e LBL

O JMP é uma instrução de saída. Quando ele está ativo, o CLP não executa as instruções entre o JMP e o rótulo correspondente (PETRUZELLA, 2014). O LBL é o destino do salto, não contribui para a continuidade lógica e é sempre considerado verdadeiro (PETRUZELLA, 2014).

- **PLC-5:** JMP e LBL são usados em pares. O salto pode ser para a frente ou para trás, e o rótulo é um número de 000 a 255 (ROCKWELL AUTOMATION, 1998; BOLTON, 2015).
- **Logix:** JMP e LBL devem estar na **mesma rotina**. O nome do rótulo é único dentro da rotina, tem até 40 caracteres e aceita letras, números e sublinhado (ROCKWELL AUTOMATION, [s. d.]p).
- **IEC 61131-3:** o salto é mostrado como uma linha de sinal booleano terminada em seta dupla (`--->>LABEL`). O destino é um rótulo de rede dentro da mesma unidade de organização de programa (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003). Cada rótulo só pode ser usado uma vez dentro da mesma unidade (HANSSEN, 2015).
- **Mitsubishi e Siemens:** a Mitsubishi usa `CJP` e `EJP`, e a Siemens tem `JMP`, executado se a entrada é 1, e `JMPN`, executado se é 0 (BOLTON, 2015).

### Salto para trás e vigilância

É possível saltar para trás, mas isso não deve ser feito muitas vezes. A varredura não pode ficar muito tempo em um laço, porque o *watchdog* desliga o processador se o tempo máximo de uma varredura for excedido (PETRUZELLA, 2014). O manual do Logix faz o mesmo alerta (ROCKWELL AUTOMATION, [s. d.]p).

### Exemplo 1: trecho manual saltado no modo automático

Em modo automático, o trecho de comandos manuais é saltado. O exemplo é original.

```text
 |  modo_auto                                        JMP         |
 +-------] [------------------------------------( FIM_MANUAL )--+
 |  btn_avancar                                   avanca_manual  |
 +-------] [------------------------------------------( )-------+
 |  btn_recuar                                    recua_manual   |
 +-------] [------------------------------------------( )-------+
 |  FIM_MANUAL                                                   |
 +-- LBL ---------------------------------------------------------+
```

!!! warning "Saídas ficam no último estado"
    Se `avanca_manual` estiver ligada no instante em que `modo_auto` ativa o salto, ela **permanece ligada**, porque o salto deixa as saídas no último estado e não examina as entradas dos degraus saltados (PETRUZELLA, 2014). Desligue essas saídas antes de saltar, ou use outra técnica.

## MCR, o relé mestre de controle

A MCR é uma instrução de saída usada em **pares**, que define uma zona do programa. A zona é fechada por um degrau de início, com MCR condicional, e um degrau de fim, com MCR incondicional (PETRUZELLA, 2014; ROCKWELL AUTOMATION, 1998).

- **Condição verdadeira:** as saídas obedecem à lógica de cada degrau, como se a zona não existisse (PETRUZELLA, 2014).
- **Condição falsa:** todos os degraus não retentivos da zona são desenergizados, mesmo que a lógica de cada degrau seja verdadeira. Os dispositivos retentivos, como travas, temporizadores retentivos e contadores, permanecem nos últimos estados (PETRUZELLA, 2014).

Duas descrições do mesmo recurso merecem atenção. Petruzella (2014) diz que a varredura **ignora as entradas** da zona falsa. O manual do PLC-5 diz que os degraus da zona **continuam sendo varridos**, mas o tempo de varredura diminui porque as saídas não retentivas ficam falsas (ROCKWELL AUTOMATION, 1998).

No Logix, cada vez que a MCR é executada com condição falsa, o comportamento de override é **alternado**, e por isso normalmente se usam duas instruções. Há apenas um bit de override por programa, então tentar aninhar zonas cria várias zonas menores (ROCKWELL AUTOMATION, [s. d.]n).

### Exemplo 2: zona habilitada por uma receita

```text
 |  receita_A_ativa                                    MCR       |
 +-------] [------------------------------------------( MCR )---+
 |  ... degraus da receita A ...                                 |
 |                                                     MCR       |
 +--------------------------------------------------( MCR )------+
```

Bolton (2015) apresenta a MCR como alternativa a repetir o contato de um mesmo relé interno em cada degrau de uma seção.

!!! danger "A MCR não substitui o contator mestre"
    Petruzella (2014) afirma que, por serem programadas, essas instruções **não devem substituir um contator principal**, que fornece a parada de emergência da alimentação. O manual do Logix é igual: a instrução não substitui um relé mestre cabeado, que deve continuar existindo para o desligamento de emergência da alimentação de E/S (ROCKWELL AUTOMATION, [s. d.]n).

## Sub-rotinas

Uma sub-rotina é um programa reduzido, chamado pelo programa principal para executar uma função específica (PETRUZELLA, 2014). Isso evita programação duplicada quando uma parte do ciclo da máquina se repete, e cada sub-rotina pode ser editada e testada individualmente (PETRUZELLA, 2014).

### JSR, SBR e RET

- **PLC-5:** a JSR leva o processador a um arquivo de sub-rotina, que é varrido uma vez, e depois retorna ao ponto de partida. A SBR é o cabeçalho opcional que guarda os parâmetros recebidos (ROCKWELL AUTOMATION, 1998). Dados de ponto flutuante passados a um endereço inteiro perdem a parte fracionária (ROCKWELL AUTOMATION, 1998). O manual fala em aninhar até oito sub-rotinas (ROCKWELL AUTOMATION, 1998).
- **SLC 500:** o programa principal fica no arquivo 2, e as sub-rotinas ficam nos arquivos de 3 a 255 (PETRUZELLA, 2014).
- **Logix:** a JSR invoca outra rotina, a SBR recebe os parâmetros e a RET os devolve e encerra a varredura da sub-rotina. Cabem até 40 parâmetros de entrada, até 40 de retorno e até 25 sub-rotinas aninhadas. Uma JSR não pode chamar a rotina principal (ROCKWELL AUTOMATION, [s. d.]o).

### Na IEC 61131-3

A norma não usa JSR. A reutilização é feita por **unidades de organização de programa** (POU): funções, blocos de função e programas, chamados pelo nome (HANSSEN, 2015). O retorno antecipado é feito pela construção `RETURN`, que em LD é um elemento ligado ao trilho esquerdo ou a uma condição (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003). O retorno é implícito no fim de uma função ou bloco de função (HANSSEN, 2015).

### Exemplo 3: um bloco de função reutilizado

Uma lógica de controle de bomba é escrita uma vez e usada para duas bombas. O exemplo é original, em ST.

```pascal
FUNCTION_BLOCK ControleBomba
VAR_INPUT
    comando : BOOL;
    falha   : BOOL;
END_VAR
VAR_OUTPUT
    ligada  : BOOL;
END_VAR

ligada := comando AND NOT falha;
END_FUNCTION_BLOCK
```

Na declaração do programa, cada bomba usa a sua instância, `bomba1 : ControleBomba;` e `bomba2 : ControleBomba;`. A chamada é:

```pascal
bomba1(comando := cmd_b1, falha := falha_b1);
bomba2(comando := cmd_b2, falha := falha_b2);
motor1 := bomba1.ligada;
motor2 := bomba2.ligada;
```

### USERFB no WEG WLP

No WLP, o bloco `USERFB` executa uma sub-rotina ladder criada pelo usuário. Os parâmetros são áreas de memória (`%PM`) compartilhadas entre o programa principal e a sub-rotina (WEG, 2016). Um `USERFB` **não pode ser inserido dentro de outro**, o que gera o erro C121 de compilação (WEG, 2016). Como todas as chamadas compartilham a mesma área de memória, o monitoramento mostra os valores da última chamada. Para depurar, o manual recomenda usar uma única chamada no programa principal (WEG, 2016).

## TND, fim temporário

No PLC-5, ao encontrar a TND, o processador zera o temporizador *watchdog*, faz uma atualização de E/S e recomeça a executar o programa pela primeira instrução do programa principal. A instrução é inserida ao depurar ou diagnosticar o programa (ROCKWELL AUTOMATION, 1998).

## Equivalências entre plataformas

| Função | IEC 61131-3 | Rockwell PLC-5 | Rockwell Logix | WEG WLP |
|--------|-------------|----------------|----------------|---------|
| Salto | `--->>LABEL` | `JMP` e `LBL`, rótulo numérico | `JMP` e `LBL`, rótulo com nome, só em LD | Não encontrado |
| Zona desabilitável | Não consultado | `MCR`, em pares | `MCR`, em pares, só em LD | Não encontrado |
| Sub-rotina | POU chamada pelo nome | `JSR`, `SBR`, `RET` | `JSR`, `SBR`, `RET` | `USERFB` |
| Retorno antecipado | `RETURN` | `RET` | `RET` | Não consultado |
| Fim temporário | Não consultado | `TND` | Não consultado | Não encontrado |

Fontes: IEC (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2003; HANSSEN, 2015); PLC-5 (ROCKWELL AUTOMATION, 1998); Logix (ROCKWELL AUTOMATION, [s. d.]n; ROCKWELL AUTOMATION, [s. d.]o; ROCKWELL AUTOMATION, [s. d.]p); WLP (WEG, 2016). **Siemens e Schneider:** a completar.

## Atualizações e comparação com versões mais recentes

Consulta em 21 nov. 2026.

| Fonte citada | Versão mais recente encontrada | Mudança relevante ou observação |
|--------------|--------------------------------|---------------------------------|
| IEC 61131-3, 2ª ed. (2003) | 3ª ed. (2013) e 4ª ed. (2025) (INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2013; INTERNATIONAL ELECTROTECHNICAL COMMISSION, 2025). | Não consegui conferir se mudaram as regras de salto e retorno, porque o texto é pago. |
| Rockwell, manual do PLC-5 (1998) | Ajuda do Studio 5000 Logix Designer, versão 38.00 (ROCKWELL AUTOMATION, [s. d.]n; ROCKWELL AUTOMATION, [s. d.]o; ROCKWELL AUTOMATION, [s. d.]p). | O rótulo deixou de ser numérico (000 a 255) e passou a ser um **nome**, com até 40 caracteres. O aninhamento de sub-rotinas foi de oito para **25**, e a JSR passou a aceitar até 40 parâmetros de cada lado. A JSR não pode chamar a rotina principal, e os tipos dos parâmetros de SBR e RET devem ser os mesmos da JSR. JMP, LBL e MCR **não existem em FBD nem em ST** (ROCKWELL AUTOMATION, [s. d.]n; ROCKWELL AUTOMATION, [s. d.]o; ROCKWELL AUTOMATION, [s. d.]p). |
| WEG, WLP V10.00 (06/2016) | Não encontrei manual do WLP posterior. | Nenhuma mudança verificada (WEG, 2016). |
| Bolton (2015), Hanssen (2015) e Petruzella (2014) | Bolton: 6ª edição é a mais recente segundo a Elsevier (ELSEVIER, 2015). | Não verifiquei edições posteriores de Hanssen e Petruzella. |

## Erros comuns

- Saltar sobre um trecho que contém saídas e esperar que elas desliguem. Elas ficam no último estado (PETRUZELLA, 2014).
- Programar temporizadores e contadores dentro de uma área saltada ou de uma zona MCR. Eles param de ser atualizados, e o PLC-5 manda reprogramar operações críticas **fora** da zona saltada (PETRUZELLA, 2014; ROCKWELL AUTOMATION, 1998). O Logix diz o mesmo: lógica saltada não é varrida, e operações críticas devem ficar fora da zona saltada e da zona MCR, porque na zona desabilitada o temporizador é zerado (ROCKWELL AUTOMATION, [s. d.]n; ROCKWELL AUTOMATION, [s. d.]p).
- Saltar para trás um número excessivo de vezes e estourar o *watchdog* (PETRUZELLA, 2014; ROCKWELL AUTOMATION, [s. d.]p).
- Saltar para dentro de uma zona MCR, ou sobrepor ou aninhar zonas MCR. O resultado pode ser operação imprevisível da máquina, com risco a equipamentos e pessoas (PETRUZELLA, 2014; ROCKWELL AUTOMATION, [s. d.]n).
- Colocar instruções retentivas dentro de uma zona MCR, porque elas permanecem nos últimos estados ativos (PETRUZELLA, 2014).
- Usar a MCR como parada de emergência (PETRUZELLA, 2014; ROCKWELL AUTOMATION, [s. d.]n).
- Repetir um rótulo na mesma rotina (HANSSEN, 2015; ROCKWELL AUTOMATION, [s. d.]p).
- Usar tipos de dados diferentes na JSR e na SBR ou RET correspondentes, no Logix. O manual avisa que o resultado pode ser inesperado (ROCKWELL AUTOMATION, [s. d.]o).

## Exercícios

1. Um JMP salta sobre três degraus que acionam um motor. O motor estava ligado quando o salto foi ativado. Qual é o estado do motor?

    ??? note "Resposta"
        Continua ligado, porque o salto deixa as saídas no último estado e não examina as entradas dos degraus saltados (PETRUZELLA, 2014).

2. Uma zona MCR fica falsa. Uma bobina normal e uma bobina de travamento (OTL) estão dentro da zona. O que acontece com cada uma?

    ??? note "Resposta"
        A bobina normal é desligada. A de travamento, retentiva, permanece no último estado (PETRUZELLA, 2014).

3. Quantas sub-rotinas aninhadas o Logix aceita? E quantos parâmetros de entrada por JSR?

    ??? note "Resposta"
        Até 25 sub-rotinas aninhadas e até 40 parâmetros de entrada (ROCKWELL AUTOMATION, [s. d.]o).

4. Estenda o Exemplo 3 para três bombas, sem duplicar a lógica do bloco de função.

    ??? note "Resposta"
        Declare uma terceira instância, `bomba3 : ControleBomba;`, e chame-a como as outras. É a ideia de reutilização do bloco de função, e vale testar no CODESYS.

5. No CODESYS, implemente o Exemplo 3 e observe o estado de `ligada` ao ativar `falha`.

!!! note "Status de validação"
    Testado em: não testado. Os exemplos aguardam validação no CODESYS. Os diagramas em texto são originais.

Fontes: ver [Referências](../99-referencias/index.md).
