# Guia Prático de Controladores Lógicos Programáveis (CLP)

Guia técnico de estudo sobre CLPs e automação industrial, com foco prático: referência de instruções, exemplos testados, projetos e comparação entre fabricantes. Plataforma principal: **CODESYS** (IEC 61131-3), com validação posterior em outras plataformas.

Este repositório complementa o material teórico e acadêmico da disciplina de Automação de Sistemas Industriais, que traz a base conceitual. Aqui, o objetivo é a aplicação.

## Sumário

1. Fundamentos
2. Linguagens da IEC 61131-3
3. Ladder (LD)
4. Diagrama de Blocos Funcionais (FBD)
5. Lógica e sequenciamento
6. Acionamentos e movimento
7. Instrumentação e controle de processos
8. Comunicação e integração
9. IHM e SCADA
10. Boas práticas
11. Referências

Comparativo entre fabricantes, exemplos e projetos completos ficam em pastas próprias, descritas na seção seguinte.

## Estrutura do repositório

```
guia-clp/
├── README.md
├── docs/
│   ├── 01-fundamentos/
│   ├── 02-iec-61131-3/
│   ├── 03-ladder/
│   ├── 04-fbd/
│   ├── 05-logica-sequenciamento/
│   ├── 06-acionamentos-movimento/
│   ├── 07-instrumentacao-processo/
│   ├── 08-comunicacao/
│   ├── 09-ihm-scada/
│   ├── 10-boas-praticas/
│   └── 99-referencias/
├── fabricantes/
├── exemplos/
└── projetos/
```

## Padrão de citação

As páginas do guia citam as fontes no texto pelo sistema autor-data (NBR 10520), por exemplo (BOLTON, 2015) ou (PETRUZELLA, 2014). A lista completa está na seção **Referências**, abaixo, elaborada conforme a NBR 6023.

---

## REFERÊNCIAS

<!-- --8<-- [start:referencias] -->

3S-SMART SOFTWARE SOLUTIONS. **User manual for PLC programming with CoDeSys 2.3**. Document version 4.0, CoDeSys V.2.3.8.0. Kempten: 3S-Smart Software Solutions, 2007.

ABB. **Task configuration**: ajuda online de programação de CLP. [S. l.]: ABB, [s. d.]. Disponível em: https://help.plc.abb.com/_cds_f_task_configuration.html. Acesso em: 21 nov. 2026.

______. **LD Element: Coil**. *In*: ABB. **Automation Builder**: ajuda online: programação com CODESYS. [S. l.]: ABB, 4 nov. 2026. Disponível em: https://en.help.plc.abb.com/_cds_LD_element_coil.html. Acesso em: 21 nov. 2026.

______. **TON (FB)**. *In*: ABB. **Standard (LIB)**: biblioteca CODESYS Application, versão 2.9.0. [S. l.]: ABB, 2025a. Disponível em: https://help.plc.abb.com/Libraries/AB290/CODESYS%20-%20Application/Common/Standard.library_Library/Timer/TON.html. Acesso em: 21 nov. 2026.

______. **TP (FB)**. *In*: ABB. **Standard (LIB)**: biblioteca CODESYS Application, versão 2.9.0. [S. l.]: ABB, 2025b. Disponível em: https://help.plc.abb.com/Libraries/AB290/CODESYS%20-%20Application/Common/Standard.library_Library/Timer/TP.html. Acesso em: 21 nov. 2026.

______. **CTU (FB)**. *In*: ABB. **Standard (LIB)**: biblioteca CODESYS Application, versão 2.9.0. [S. l.]: ABB, 2025c. Disponível em: https://help.plc.abb.com/Libraries/AB290/CODESYS%20-%20Application/Common/Standard.library_Library/Counter/CTU.html. Acesso em: 21 nov. 2026.

AMERICAN NATIONAL STANDARDS INSTITUTE. **IEC 61131-3:2013**: programmable controllers: part 3: programming languages. [S. l.]: ANSI Webstore, [s. d.]. Disponível em: https://webstore.ansi.org/Standards/IEC/IEC61131Ed2013. Acesso em: 21 nov. 2026.

BOLTON, W. **Programmable logic controllers**. 6. ed. Oxford: Newnes, 2015.

CODESYS GROUP. **Task configuration**. *In*: CODESYS GROUP. **CODESYS Essentials**: CODESYS Online Help, versão 3.5.22.0. Kempten: CODESYS Group, [s. d.]a. Disponível em: https://content.helpme-codesys.com/en/CODESYS%20Development%20System/_cds_f_task_configuration.html. Acesso em: 21 nov. 2026.

______. **Basics**. *In*: CODESYS GROUP. **CODESYS 4**: ajuda online: programação em diagrama ladder. Kempten: CODESYS Group, [s. d.]b. Disponível em: https://content.helpme-codesys.com/en/CODESYS%204/_c4ld_basics.html. Acesso em: 21 nov. 2026.

______. **Adapting the elements**. *In*: CODESYS GROUP. **CODESYS 4**: ajuda online: programação em diagrama ladder. Kempten: CODESYS Group, [s. d.]c. Disponível em: https://content.helpme-codesys.com/en/CODESYS%204/_c4ld_edit_elements.html. Acesso em: 21 nov. 2026.

ELSEVIER. **Programmable logic controllers**. 6th edition. [S. l.]: Elsevier, 2015. Disponível em: https://shop.elsevier.com/books/catalog/isbn/9780128029299. Acesso em: 21 nov. 2026.

HANSSEN, D. H. **Programmable logic controllers**: a practical approach to IEC 61131-3 using CODESYS. Tradução: Dan Lufkin. Chichester: John Wiley & Sons, 2015. ISBN 978-1-118-94924-5.

INTERNATIONAL ELECTROTECHNICAL COMMISSION. **IEC 61131-3**: programmable controllers: part 3: programming languages. 2. ed. Geneva: IEC, 2003.

______. **IEC 61131-3:2013**: programmable controllers: part 3: programming languages. Edition 3.0. Geneva: IEC, 20 fev. 2013. Disponível em: https://webstore.iec.ch/publication/4552. Acesso em: 21 nov. 2026.

______. **IEC 61131-3:2025**: programmable controllers: part 3: programming languages. Edition 4.0. Geneva: IEC, 22 maio 2025. Disponível em: https://webstore.iec.ch/en/publication/68533. Acesso em: 21 nov. 2026.

JACK, H. **Automating manufacturing systems with PLCs**. Version 5.1. [S. l.]: Hugh Jack, 2008. Disponível em: http://claymore.engineer.gvsu.edu/~jackh/books.html. Acesso em: 21 nov. 2026.

______. **Automating manufacturing systems with PLCs**. Edição impressa. [S. l.]: Lulu, [s. d.]. Disponível em: https://www.lulu.com/shop/hugh-jack/automating-manufacturing-systems-with-plcs/paperback/product-1q66yep4.html. Acesso em: 21 nov. 2026.

KOLLMORGEN. **CTUD / CTUDr**. *In*: KOLLMORGEN. **KAS 3.05**: ajuda online: biblioteca PLC. [S. l.]: Kollmorgen, [s. d.]. Disponível em: https://webhelp.kollmorgen.com/kas3.05/Content/PLC_library/ctud.htm. Acesso em: 21 nov. 2026.

NOGUEIRA, S. Z. S. **Comandos elétricos**: o seu guia prático e definitivo. 1. ed. Rio de Janeiro: Edição do autor, 2019.

PETRUZELLA, F. D. **Controladores lógicos programáveis**. Tradução: Romeu Abdo. Revisão técnica: Antonio Pertence Júnior. 4. ed. Porto Alegre: AMGH, 2014. ISBN 978-85-8055-283-6.

PLCOPEN. **Coding guidelines**: technical paper, version 1.0, official release. [S. l.]: PLCopen, abr. 2016. Software Construction Guidelines.

______. **Enhanced IEC 61131-3**: proven standard fit for future. [S. l.]: PLCopen, [s. d.]. Disponível em: https://plcopen.org/node/196. Acesso em: 21 nov. 2026.

ROCKWELL AUTOMATION. **PLC-5 programmable controllers**: instruction set reference. [S. l.]: Rockwell Automation, nov. 1998. Publicação 1785-6.1.

______. **Studio 5000 Logix Designer**: release notes, versão 38.00.00. [S. l.]: Rockwell Automation, set. 2025. Disponível em: https://compatibility.rockwellautomation.com/GeneratedReleaseNote.aspx?v1=66022. Acesso em: 21 nov. 2026.

______. **Tasks**. *In*: ROCKWELL AUTOMATION. **Studio 5000 Logix Designer online help**: versão 38.00. [S. l.]: Rockwell Automation, [s. d.]a. Disponível em: https://www.rockwellautomation.com/en-ua/docs/studio-5000-logix-designer/38-00/contents-ditamap/studio-5000-logix-designer/tasks--programs--and-routines/tasks.html. Acesso em: 21 nov. 2026.

______. **Examine if Closed (XIC)**. *In*: ROCKWELL AUTOMATION. **Studio 5000 Logix Designer online help**: versão 38.00. [S. l.]: Rockwell Automation, [s. d.]b. Disponível em: https://www.rockwellautomation.com/en-fi/docs/studio-5000-logix-designer/38-00/contents-ditamap/instruction-set/bit-instructions1/examine-if-closed--xic-1.html. Acesso em: 21 nov. 2026.

______. **Output Energize (OTE)**. *In*: ROCKWELL AUTOMATION. **Studio 5000 Logix Designer online help**: versão 38.00. [S. l.]: Rockwell Automation, [s. d.]c. Disponível em: https://www.rockwellautomation.com/en-fi/docs/studio-5000-logix-designer/38-00/contents-ditamap/instruction-set/bit-instructions1/output-energize--ote-11.html. Acesso em: 21 nov. 2026.

______. **Output Latch (OTL)**. *In*: ROCKWELL AUTOMATION. **Studio 5000 Logix Designer online help**: versão 38.00. [S. l.]: Rockwell Automation, [s. d.]d. Disponível em: https://www.rockwellautomation.com/en-dk/docs/studio-5000-logix-designer/38-00/contents-ditamap/instruction-set/bit-instructions1/output-latch--otl-1.html. Acesso em: 21 nov. 2026.

______. **Bit instructions**. *In*: ROCKWELL AUTOMATION. **Studio 5000 Logix Designer online help**: versão 38.00. [S. l.]: Rockwell Automation, [s. d.]e. Disponível em: https://www.rockwellautomation.com/en-gb/docs/studio-5000-logix-designer/38-00/contents-ditamap/instruction-set/bit-instructions1.html. Acesso em: 21 nov. 2026.

______. **Timer On Delay (TON)**. *In*: ROCKWELL AUTOMATION. **Studio 5000 Logix Designer online help**: versão 38.00. [S. l.]: Rockwell Automation, [s. d.]f. Disponível em: https://www.rockwellautomation.com/en-dk/docs/studio-5000-logix-designer/38-00/contents-ditamap/instruction-set/timer-and-counter-instructions/timer-on-delay--ton-.html. Acesso em: 21 nov. 2026.

______. **Retentive Timer On (RTO)**. *In*: ROCKWELL AUTOMATION. **Studio 5000 Logix Designer online help**: versão 38.00. [S. l.]: Rockwell Automation, [s. d.]g. Disponível em: https://www.rockwellautomation.com/en-dk/docs/studio-5000-logix-designer/38-00/contents-ditamap/instruction-set/timer-and-counter-instructions/retentive-timer-on--rto-.html. Acesso em: 21 nov. 2026.

______. **Retentive Timer On with Reset (RTOR)**. *In*: ROCKWELL AUTOMATION. **Studio 5000 Logix Designer online help**: versão 38.00. [S. l.]: Rockwell Automation, [s. d.]h. Disponível em: https://www.rockwellautomation.com/en-dk/docs/studio-5000-logix-designer/38-00/contents-ditamap/instruction-set/timer-and-counter-instructions/retentive-timer-on-with-reset--rtor-.html. Acesso em: 21 nov. 2026.

______. **Count Up (CTU)**. *In*: ROCKWELL AUTOMATION. **Studio 5000 Logix Designer online help**: versão 38.00. [S. l.]: Rockwell Automation, [s. d.]i. Disponível em: https://www.rockwellautomation.com/en-be/docs/studio-5000-logix-designer/38-00/contents-ditamap/instruction-set/timer-and-counter-instructions/count-up--ctu-.html. Acesso em: 21 nov. 2026.

SCHNEIDER ELECTRIC. **Modicon M241 Logic Controller**: programming guide. EIO0000001432.07. [S. l.]: Schneider Electric, mar. 2018.

______. **Modicon M241 Logic Controller**: programming guide. EIO0000003059, versão 10. [S. l.]: Schneider Electric, 1 abr. 2026. Disponível em: https://www.se.com/us/en/download/document/EIO0000003059. Acesso em: 21 nov. 2026.

______. **System and task watchdogs**. *In*: SCHNEIDER ELECTRIC. **EcoStruxure Machine Expert V2.2**: Modicon M241 Logic Controller: programming guide: ajuda online. [S. l.]: Schneider Electric, [s. d.]a. Disponível em: https://product-help.schneider-electric.com/Machine%20Expert/V2.2/en/m241prg/m241prg/D-RU-0004615.html. Acesso em: 21 nov. 2026.

______. **Task types**. *In*: SCHNEIDER ELECTRIC. **EcoStruxure Machine Expert V2.1**: Modicon M241 Logic Controller: programming guide: ajuda online. [S. l.]: Schneider Electric, [s. d.]b. Disponível em: https://product-help.schneider-electric.com/Machine%20Expert/V2.1/en/m241prg/m241prg/D-SE-0008842.html. Acesso em: 21 nov. 2026.

SIEMENS. **Programming guideline for S7-1200/1500**: TIA Portal. Entry ID: 81318674, V1.6. [S. l.]: Siemens Industry Online Support, dez. 2018. Disponível em: https://support.industry.siemens.com/cs/ww/en/view/81318674. Acesso em: 21 nov. 2026.

______. **S7-1500**: getting started. Edição 10/2021, A5E03981761-AD. [S. l.]: Siemens, out. 2021.

WEG. **WLP**: manual del usuario. Serie WLP V10.00. [S. l.]: WEG, jun. 2016.

______. **WLP**: user's manual. Series WLP V10.00. [S. l.]: WEG, jun. 2016.

<!-- --8<-- [end:referencias] -->
