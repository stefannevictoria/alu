# ALU 8 bits — Ponderada de Arquitetura de Computadores


Este repositório contém a implementação de uma **ALU (Unidade Lógica e Aritmética) de 8 bits**, desenvolvida como atividade ponderada da disciplina de Computação.

A ALU é o componente central de um processador, sendo ela que executa todos os cálculos matemáticos e lógicos. Esta implementação foi feita usando a ferramenta de simulação de circuitos digitais **Digital** e implementa sete operações completas com seletor de operação e registradores.

---

## Operações implementadas

| SEL | Operação       | Entrada       | Saída                        |
|-----|----------------|---------------|------------------------------|
| 000 | Soma           | AC + N        | AC (8 bits)                  |
| 001 | Subtração      | AC - N        | AC (8 bits)                  |
| 010 | Multiplicação  | AC × N        | AC (8 LSB) e MQ (8 MSB)      |
| 011 | Divisão        | AC ÷ N        | AC (Resto) e MQ (Quociente)  |
| 100 | Shift lógico   | AC            | AC (8 bits)                  |
| 101 | NAND           | AC NAND N     | AC (8 bits)                  |
| 110 | XOR            | AC XOR N      | AC (8 bits)                  |

---

## Como foi feito

### Ferramenta utilizada

- **Digital** — simulador de circuitos digitais ([https://github.com/hneemann/Digital](https://github.com/hneemann/Digital))

### Estrutura dos arquivos

```
📁 ALU/
├── ALU.dig                  ← circuito principal com todas as operações
├── divisor_8bits.dig        ← divisor Restoring Division
├── multiplidor_8bits.dig    ← multiplicador array (produtos parciais)
├── nand.dig                 ← porta lógica nand (8bits)
├── passo_div.dig            ← um passo da divisão (subcircuito)
├── README.md
├── register_8bits.dig       ← registrador de 8 bits com flip-flops D
├── shift.dig                ← shift lógico left/right
├── somador_1bit.dig         ← Full Adder de 1 bit (bloco base)
├── somador_8bits.dig        ← somador de 8 bits (8x Full Adder encadeados)
├── somador_16bits.dig       ← somador de 16 bits
├── somador_16bits.dig       ← somador de 16 bits (usado na multiplicação)
├── subtrator_8bits.dig      ← subtrator usando complemento de 2
└── xor.dig                  ← porta lógica nand (8bits)
```

---

### Soma

A soma foi implementada a partir do zero, começando pelo **Full Adder de 1 bit**. Ele recebe dois bits A e B e um carry de entrada Cin, e produz o resultado S e o carry de saída Co.

- **S** é calculado com uma porta XOR de três entradas — XOR retorna 1 quando um número ímpar de entradas é 1, que é exatamente o comportamento da soma binária
- **Co** é calculado com três portas AND (uma para cada par A-B, A-Cin, B-Cin) e uma porta OR — o carry é gerado quando pelo menos dois bits de entrada são 1

O **somador de 8 bits** encadeia 8 Full Adders, onde o Co de cada um entra como Cin do próximo.


### Subtração

O subtrator reutiliza o somador aplicando o **complemento de 2**. Para calcular A - B, o circuito faz A + (~B) + 1:

- Os bits de B são invertidos por portas **XOR** controladas por um sinal SUB
- O mesmo sinal SUB é conectado no **Cin do primeiro somador**, fornecendo o +1 do complemento de 2 sem nenhum circuito extra
- Um **LED** conectado no bit 7 da saída acende quando o resultado é negativo, pois em complemento de 2 o MSB indica o sinal


### Multiplicação

Implementada com o método **array multiplier**, equivalente à multiplicação manual em binário. Para cada bit de B, é calculado um Produto Parcial (PP) usando 8 portas AND com os bits de A. Os produtos parciais são somados em cascata com somadores de 16 bits, cada um deslocado pela posição correta.

O deslocamento (shift) é feito pela própria conexão dos fios nas posições corretas. O resultado de 16 bits é dividido: bits 0-7 vão para o AC e bits 8-15 vão para o MQ.

### Divisão

Implementada com o algoritmo **Restoring Division**. O dividendo é processado bit a bit, do mais significativo para o menos significativo. A cada passo:

1. O parcial atual sofre um shift left e recebe o próximo bit do dividendo
2. O divisor é subtraído do parcial
3. Se o resultado for positivo: o bit do quociente é 1 e o resultado é mantido
4. Se o resultado for negativo: o bit do quociente é 0 e o parcial é restaurado (daí o nome "restoring")

Um subcircuito `passo_div` encapsula toda essa lógica com um merger de shift, o subtrator, um MUX para restaurar, e uma porta NOT para gerar o bit do quociente. São encadeados 8 instâncias — uma por bit do dividendo. O resto final vai para AC e o quociente vai para MQ.


### Shift Lógico

O shift é feita pela conexão de fios:

- **Shift left**: cada bit vai para a posição acima, bit 0 recebe constante 0, bit 7 é descartado
- **Shift right**: cada bit vai para a posição abaixo, bit 7 recebe constante 0, bit 0 é descartado

Um MUX controlado pela entrada DIR seleciona a direção: 0 para esquerda, 1 para direita.


### NAND e XOR

Operações lógicas diretas usando as portas de 8 bits do Digital:

- **XOR**: retorna 1 quando os bits correspondentes de AC e N são diferentes
- **NAND**: AND bit a bit de AC e N com o resultado invertido


### Registradores

Os registradores AC e MQ foram implementados com **8 flip-flops D em paralelo**, todos compartilhando o mesmo clock. Sem clock, a saída Q mantém o valor anterior. Com um pulso de clock, captura o valor atual da entrada D. Esse subcircuito é importado duas vezes na ALU — um para AC e outro para MQ.


### ALU completa

Todas as operações rodam em paralelo. Um **MUX de 8 entradas** para o AC e outro para o MQ selecionam qual resultado aparece na saída conforme o valor de SEL (3 bits). As saídas passam pelos registradores antes de sair, e o resultado só é confirmado com um pulso de clock.

---

## Como executar

1. Baixe e instale o **Digital**: [https://github.com/hneemann/Digital/releases](https://github.com/hneemann/Digital/releases)
2. Clone este repositório
3. Abra o arquivo `ALU.dig` no Digital
4. Clique em **Simulação → Iniciar**
5. Defina os valores de **AC**, **N** e **SEL**
6. Clique no **CLK** para confirmar o resultado nos registradores

---
 
## Vídeo de apresentação
 
[![Vídeo da ponderada](https://img.youtube.com/vi/ghw8AQYY_y4/0.jpg)](https://www.youtube.com/watch?v=ghw8AQYY_y4)

> Clique na imagem acima para assistir à apresentação completa da ALU.
