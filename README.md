# Processador de 8 bits no Simulador Digital

Este projeto contempla a implementação completa de uma Unidade Central de Processamento de 8 bits, desenvolvida do zero utilizando o ambiente educacional **Digital**. O núcleo do sistema opera sob os preceitos clássicos computacionais, integrando lógica e aritmética com fluxo de dados em um barramento comum.

## Visão Geral do Projeto

A característica mais marcante deste microprocessador é o compartilhamento do endereçamento. Utilizando o preceito de Von Neumann, o mesmo Contador de Programa (PC) é responsável por apontar simultaneamente para as memórias de instruções e para a memória de dados (EEPROMs). Isso enxuga o projeto elétrico e consolida o tráfego em tempo contínuo de forma sequencial.

A arquitetura usa palavras nativas de 8 bits para operações, com os sinais de controle centralizados por um opcode em bloco de 3 bits, gerando a base para um conjunto de 8 instruções básicas processadas direto no hardware.

## Estrutura de Hardware e Módulos

Para otimizar o entendimento lógico e estrutural, podemos segmentar os componentes em três subsistemas principais:

### 1. Memória e Endereçamento
* **Bancos de Memória (EEPROMs de Dupla Porta):** Estão alocadas paralelamente. Uma fita guarda o *software* (opcodes ou diretrizes), enquanto o banco ao lado serve os *valores/dados* daquele determinado comando.
* **Contador Principal (PC):** O motor sequencial responsável por varrer os endereços das EEPROMs.
* **Registradores de Estabilização (MARs):** Antes de o PC bater diretamente na memória, o valor passa por instâncias estabilizadoras (Memory Address Registers). A principal utilidade no circuito digital é mitigar o atraso de propagação dos flip-flops, evitando buscar dados falsos durante curtas interferências do Clock.

### 2. Caminho de Dados e Acumulação Múltipla
* **Unidade Lógica Computacional (ALU):** É a engrenagem transformadora que mastiga as contas. Ela consome dois vetores de 8 bits por vez para devolver uma resposta coerente com a solicitação da CPU.
* **Acumulador (AC):** É o registrador principal da operação. Todo estado intermediário entre duas etapas e contas passa e finaliza atracado nele.
* **Extensão Lógica Múltipla (MQ):** Registrador de apoio empregado unicamente quando as contas espirram para fora da capacidade máxima (extrapolam o teto do 8º bit). Ideal para acolher o resto de divisões e expansão da matriz em multiplicações de LSB/MSB.
* **Barramento Coletivo:** Uma trilha limpa batizada via *Tunnels* de rede, permitindo que todas essas peças consigam trocar as respostas e inputs sem a criação de linhas redundantes.

### 3. Loop de Execução e Estado (Control Unit)
A Unidade de Controle atua como uma máquina de estados infinita baseada em um Contador de Cadeia (*Ring Counter*) fechado em 5 passos lógicos:
* Composto por uma malha de **Flip-Flops D (D_FF_AS)** em anel. Diferente de contadores binários brutos, o pulso no anel é limpo, em que estritamente apenas um sinalizador sobe para Alto (nível Lógico 1) em cada ciclo de tempo.
* **A Extração (Fetch):** Ocorre nas ondas iniciais, destravando as passagens das memórias. O opcode desce limpo e fica preso no **Instruction Register (IR)** da base de controle.
* **A Resolução (Execute):** Nas ondas seguintes do anel lógico, a ALU é despertada por portas baseadas em XOR da Control Unit. O dado é calculado, jogado no ônibus principal, e depois engatado no Acumulador (onda final, engatilhando PC++).

## ISA: Instruction Set Architecture

A linguagem do processo mapeou estritamente 8 tipos de ações que englobam aritmética cardinal e manejo booleano de arranjos binários:

#### Operações Numéricas
| Hexa/Bin | Instrução | Dinâmica | Conclusivo em AC |
| :---: | :---: | :--- | :--- |
| `000` | **ADD** | Soma aritmética do Acumulador com Op | Resultado Direto |
| `001` | **SUB** | Subtrai do Acumulador a referência Op | Resultado Direto |
| `010` | **MUL** | A união (AC * Op) quebra em dois vetores | Guarda parte Menor/LSB |
| `011` | **DIV** | Realização de separação com piso cego | Quociente Bruto |

#### Booleanos e Movimentação Espacial
| Hexa/Bin | Instrução | Dinâmica | Conclusivo em AC |
| :---: | :---: | :--- | :--- |
| `100` | **SHL** | Empuxa os vetores globalmente à esquerda | Multiplicação por raiz (2x) |
| `101` | **SHR** | Expurga vetores da linha global p/ direita | Divisão Absoluta por (2x) |
| `110` | **NAND** | Portas integradas negando `AC & Op` | Produto Negado (Lógico) |
| `111` | **XOR** | Equivalência paritária excludente | Disjunção do Acumulador |

---

## Prova de Conceito Computacional

O projeto acompanha um esquema prático para validar a simulação em cadeia funcional. A premissa do programa abaixo realiza os seguintes passos no registrador AC de forma cumulativa, resolvendo algo equivalente a: `[(5 * 2) + 14] / 2`:

1. **Endereço `0x00`:** `000` [ADD] - Traz o operando **5** para o acumulador livre. *(AC altera de vazio para `5`)*.
2. **Endereço `0x01`:** `010` [MUL] - Envia a requisição de multiplicar o acumulador por **2**. *(AC altera para `10`)*.
3. **Endereço `0x02`:** `000` [ADD] - Injeta mais operandos de base no circuito, com a quantia sendo **14**. *(AC vira `24`)*.
4. **Endereço `0x03`:** `101` [SHR] - Desliza os chips todos na escala binária subtraindo posições. Efetua tecnicamente uma divisão exata por 2. Operando da memória pode ser arbitrário ou nulo. *(AC final: `12`)*.
