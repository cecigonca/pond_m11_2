# CPU de 8 bits no Digital

Implementação de uma **CPU de 8 bits** no simulador **Digital** (hneemann/Digital). O projeto integra um **datapath** completo, um **controller** microprogramado (sequencer + ROM de microcódigo) e uma **ULA** com operações aritméticas e lógicas básicas. Também evidente o ciclo **FETCH/EXECUTE**.

## Objetivo
- Construir e demonstrar uma CPU de 8 bits
- Evidenciar o papel do **controller** e do ciclo **FETCH/EXECUTE**
- Mostrar como programar a **ROM** e como o **PC** (Program Counter) muda com saltos

## Componentes: funções e definições

* **CLK (clock):** controla o ritmo da CPU, fazendo os componentes avançarem entre os estados de execução
* **WBus (8 bits):** barramento principal por onde todos os dados circulam entre os registradores, a memória e a ULA
* **PC (Program Counter):** guarda o endereço da próxima instrução que será buscada na memória
* **MAR (Memory Address Register):** armazena o endereço que será usado para acessar a memória
* **ROM (memory.dig):** guarda o programa e os dados que a CPU executa
* **IR (Instruction Register):** armazena a instrução atual que está sendo executada (opcode + operando)
* **Registradores A e B:** armazenam temporariamente os dados usados nas operações; o **A** funciona como acumulador principal
* **ULA:** realiza as operações aritméticas e lógicas da CPU (soma, subtração, multiplicação, divisão, incremento e decremento)
* **OUT:** mostra o resultado final das operações realizadas pela CPU
* **Controller:** coordena todos os sinais de controle, definindo o que cada componente deve fazer em cada etapa (fetch e execute)
* **Flags:** indicam condições do resultado (zero ou negativo) usadas em instruções de salto condicional

**Arquivos principais**
- `CPU.dig` (top-level): integra controller + datapath; inclui **BUS_SNIFFER** no WBus
- `data-path.dig`, `controller.dig`, `sequencer.dig`, `memory.dig`, `ula.dig`
- Blocos de operações: `8bit-adder.dig`, `8bit-subtractor.dig`, `8bit-multiplicator.dig`, `divisor.dig` (+ `partial_division.dig`), `8bit-incrementor.dig`, `8bit-decrementor.dig`, `full-adder.dig`, `half-adder.dig`, `PC.dig`, `resgistrator.dig`

## Operações e conexão com a CPU final
- Os blocos de **soma, subtração, multiplicação, divisão, incremento e decremento** são testados isoladamente e integrados em `ula.dig`
- No datapath, **A** e **B** alimentam a ULA; `ULA_op` define a operação; o **resultado retorna para A** (`RegA_Ld`) quando requerido
- O **controller** coordena o momento de leitura/escrita nos registradores e no WBus, evitando conflitos

## Instruções e organização
**Formato da instrução (ROM):** 1 byte = `OOOO AAAA`  
- **Opcode (OOOO)**: 4 bits altos (verbo da operação)  
- **Operando (AAAA)**: 4 bits baixos (endereço/imediato curto)
Dados para LDA/ADD/SUB etc. ficam na própria ROM, no **endereço indicado pelo operando**

### Datapath
Fluxo geral: **PC → MAR → ROM → IR → (A/B/ULA) → OUT**, conectado pelo **WBus**. O controller garante um único motorista no barramento por vez

### Registradores (A/B/OUT/PC/MAR/IR)
- Estrutura genérica com sinais de `*_Ld` (load) e saídas tri-state para o WBus quando aplicável
- **PC**: `PC_En` para incremento sequencial; `PC_Ld` para saltos
- **MAR**: `Mar_Ld` com endereço proveniente de PC ou operando do IR
- **IR**: `IR_Ld` na fase de fetch; expõe **opcode** e **operando** para o controller

### Memória (`memory.dig`)
- ROM programável (8×256). Conteúdo editável em **hex** no Digital

# FETCH e EXECUTE
## Como funciona
**FETCH (fixo):**
- **T1**: `PC → MAR`
- **T2**: `ROM → IR` **e** `PC_En`

**EXECUTE (depende do opcode):**
- **LDA x** : `IR.addr → MAR` → `Mem → A`
- **ADD x** : `Mem → B` → `ULA_op=ADD` → `A ← ULA`
- **SUB/INC/DEC/MULT/DIV** : mesmo padrão, trocando `ULA_op`
- **OUT** : `A → OUT`
- **JMP x** : `PC_Ld ← x`
- **JumpIf x** : `PC_Ld ← x` se (`Z_Flag`/`is_NEG`) verdadeira

## Conclusão
A CPU prioriza clareza didática: **fetch** padronizado, **execute** por opcode, **ULA modular**, **controller microprogramado** e **ROM** facilmente editável. O uso de **WBus único** e **BUS_SNIFFER** simplifica a depuração, tornando visíveis os bytes trafegando a cada T-estado.


## Vídeo do funcionamento
