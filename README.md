# Laboratorio 2 - Sumador/Resta en Verilog

## Integrante
- **Juan David Velasco Sarmiento**

## Materia
- Arquitectura de Procesadores

## Objetivo del Laboratorio
El objetivo de este laboratorio es diseñar e implementar en Verilog un circuito que permita realizar operaciones aritméticas de suma y resta de dos números de 4 bits. El diseño utiliza la técnica de complemento a 2 para facilitar la resta, aprovechando la arquitectura del sumador en cascada. Este proyecto sirve para profundizar en conceptos fundamentales de la aritmética digital y el diseño modular en sistemas digitales.

## Descripción del Funcionamiento

El laboratorio está compuesto por tres módulos principales, que se interconectan para formar el circuito completo:

1. **Módulo `full_adder`**  
   - **Función:** Implementa un sumador completo de un bit.  
   - **Detalle:** Realiza la operación de suma a nivel de bit, considerando la entrada de acarreo. Su función es esencial para la suma de bits individuales en cada posición.

2. **Módulo `ripple_adder_4bit`**  
   - **Función:** Suma dos números de 4 bits utilizando instancias del módulo `full_adder`.  
   - **Detalle:** Se encadena la operación de varios sumadores completos (en una configuración llamada "sumador en cascada" o ripple carry adder) para realizar la operación de suma de manera secuencial, propagando los acarreos entre los bits de menor peso a mayor peso.

3. **Módulo `lab2`**  
   - **Función:** Es el módulo principal que integra el sumador y además permite realizar operaciones de suma o resta dependiendo de una señal de control `Sel`.  
   - **Detalle:**  
     - Cuando `Sel = 0`, el circuito opera directamente como un sumador, sumando A y B.
     - Cuando `Sel = 1`, el circuito efectúa la operación de resta (A - B). Esto se logra al generar el complemento a 2 de B. Primero se calcula el complemento a 1 mediante una operación XOR controlada, y luego se suma 1 (mediante la conexión del bit `Sel` al acarreo de entrada, Cin).

## Diagrama del Circuito

El diagrama de bloques del circuito se compone de los siguientes elementos:

- **Entrada A (4 bits) y entrada B (4 bits):** Son los operandos para la operación aritmética.
- **Señal de Selección `Sel`:** Determina la operación: suma (Sel = 0) o resta (Sel = 1).
- **Complemento a 1 de B:** Se genera mediante una operación XOR que invierte los bits de B cuando `Sel` es 1.
- **Sumador en Cascada:** Se realiza la suma entre A y B (o su complemento) utilizando el módulo `ripple_adder_4bit`.
- **Salida S (4 bits) y acarreo de salida `Cout`:** Se obtienen los resultados finales de la operación aritmética.

![Diagrama del Circuito](Juand\Downloads.png)


## Archivos del Proyecto

- `lab2.v`: Módulo principal que integra la operación de suma/resta.
- `ripple_adder_4bit.v`: Sumador en cascada de 4 bits construido a partir del sumador completo.
- `full_adder.v`: Sumador completo básico para la suma a nivel de bit.

## Código Fuente

### Módulo `lab2`
```verilog
module lab2(
    input [3:0] A,
    input [3:0] B,
    input Sel,
    output [3:0] S,
    output Cout
);
    wire [3:0] B_comp1;

    // Complemento a 1 de B usando XOR controlado por Sel
    assign B_comp1 = B ^ {4{Sel}};

    // Sumador con Cin = Sel (realiza complemento a 2 al sumar 1)
    ripple_adder_4bit adder(
        .A(A),
        .B(B_comp1),
        .Cin(Sel),
        .S(S),
        .Cout(Cout)
    );
endmodule
module ripple_adder_4bit(
    input [3:0] A,
    input [3:0] B,
    input Cin,
    output [3:0] S,
    output Cout
);
    wire [3:0] c;

    full_adder fa0(.a(A[0]), .b(B[0]), .cin(Cin), .sum(S[0]), .cout(c[0]));
    full_adder fa1(.a(A[1]), .b(B[1]), .cin(c[0]), .sum(S[1]), .cout(c[1]));
    full_adder fa2(.a(A[2]), .b(B[2]), .cin(c[1]), .sum(S[2]), .cout(c[2]));
    full_adder fa3(.a(A[3]), .b(B[3]), .cin(c[2]), .sum(S[3]), .cout(Cout));
endmodule
module full_adder(
    input a,
    input b,
    input cin,
    output sum,
    output cout
);
    assign sum = a ^ b ^ cin;
    assign cout = (a & b) | (a & cin) | (b & cin);
endmodule
```
## Pruebas y Resultados

Se realizaron diversas pruebas en ambiente de simulación para verificar el correcto funcionamiento del circuito, utilizando diferentes combinaciones de entrada para A y B, tanto en modo suma como en modo resta:

### Suma (Sel = 0):

- **Ejemplo 1:**  
  - A = 0101 (5)  
  - B = 0011 (3)  
  - **Resultado esperado:** S = 1000 (8)

- **Ejemplo 2:**  
  - A = 1100 (12)  
  - B = 0010 (2)  
  - **Resultado esperado:** S = 1110 (14)

### Resta (Sel = 1):

- **Ejemplo 1:**  
  - A = 0101 (5)  
  - B = 0011 (3)  
  - **Resultado esperado:** S = 0010 (2) *(con la lógica de complemento a 2)*

- **Ejemplo 2:**  
  - A = 1001 (9)  
  - B = 0100 (4)  
  - **Resultado esperado:** S = 0101 (5)

Los resultados obtenidos durante la simulación confirmaron la correcta implementación del circuito, cumpliendo tanto la función aritmética de suma como la de resta.

## Discusión y Conclusiones

### Versatilidad del Diseño
La utilización de la señal de selección `Sel` para decidir entre suma y resta demuestra la eficiencia de poder reutilizar el mismo circuito aritmético. Al invertir los bits de B y sumar uno, se habilita la operación de complemento a 2 sin necesidad de un bloque aritmético adicional.

### Modularidad
La división del diseño en módulos (`full_adder`, `ripple_adder_4bit` y `lab2`) mejora la organización del código y facilita el proceso de depuración y ampliación del proyecto. Además, permite la reutilización de componentes en futuros diseños.

### Facilidad de Simulación
Se recomienda el uso de testbenches automatizados para la simulación de circuitos digitales, lo que agiliza la verificación de todas las combinaciones posibles de entradas y detecta posibles errores en la implementación.

### Potencial de Expansión
Aunque este laboratorio se enfoca en operaciones aritméticas básicas en números de 4 bits, la estructura modular del diseño permite escalar el circuito a mayores anchos de datos, lo que es crucial en el desarrollo de unidades aritméticas para procesadores.
