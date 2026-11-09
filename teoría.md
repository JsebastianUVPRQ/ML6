# Fundamentos teóricos de ML6

## Modelado probabilístico del lenguaje a nivel de caracteres

> **Propósito del documento.** Este texto reúne la teoría matemática y
> computacional necesaria para explicar un proyecto de modelado de
> lenguaje basado en secuencias de caracteres. Está organizado para
> servir como material de estudio y como base de una exposición. La
> descripción concreta de la implementación debe contrastarse con el
> código y los resultados reales del repositorio.

------------------------------------------------------------------------

## 1. ¿Qué es un modelo de lenguaje?

Un **modelo de lenguaje** asigna probabilidades a secuencias de
símbolos. En lenguaje natural, esos símbolos pueden ser palabras,
subpalabras o caracteres.

Si una secuencia está formada por los símbolos

\[ x_1,x_2,`\ldots`{=tex},x_T, \]

el modelo busca estimar la probabilidad de observar esa secuencia:

\[ P(x_1,x_2,`\ldots`{=tex},x_T). \]

Esta probabilidad permite comparar secuencias y, especialmente, estimar
cuál es el siguiente símbolo más probable dado un contexto anterior.

### 1.1 Modelado a nivel de caracteres

En un modelo a nivel de caracteres, cada símbolo es un carácter: una
letra, un espacio, un signo de puntuación, un dígito, etc. Por ejemplo,
la palabra `casa` se representa como:

\[ (c,a,s,a). \]

El objetivo de predicción consiste en estimar:

\[ P(x\_{t+1}`\mid `{=tex}x_1,`\ldots`{=tex},x_t), \]

es decir, la probabilidad del siguiente carácter conociendo los
anteriores.

**Ventajas:**

-   No requiere un diccionario cerrado de palabras.
-   Puede representar palabras desconocidas, nombres propios y errores
    ortográficos.
-   Permite estudiar patrones locales de escritura, como combinaciones
    frecuentes de letras.

**Limitaciones:**

-   Las dependencias entre caracteres pueden ser largas y difíciles de
    capturar.
-   Una secuencia de caracteres plausible no necesariamente forma una
    frase con sentido.
-   El modelo no representa directamente conceptos, sintaxis o
    significado.

------------------------------------------------------------------------

## 2. Probabilidad condicional y regla de la cadena

La probabilidad condicional mide la probabilidad de un evento cuando se
conoce que otro evento ocurrió:

\[
P(A`\mid `{=tex}B)=`\frac{P(A\cap B)}{P(B)}`{=tex},`\qquad `{=tex}P(B)\>0.
\]

En un modelo de lenguaje, interesa la probabilidad de un carácter dado
el contexto. Por ejemplo:

\[ P(`\text{a}`{=tex}`\mid `{=tex}`\text{c}`{=tex}) \]

representa la probabilidad de que aparezca `a` después de `c`, según los
datos de entrenamiento.

La **regla de la cadena** permite expresar la probabilidad de una
secuencia como producto de probabilidades condicionales:

\[ P(x_1,`\ldots`{=tex},x_T) =
P(x_1)`\prod`{=tex}*{t=2}\^{T}P(x_t`\mid `{=tex}x_1,`\ldots`{=tex},x*{t-1}).
\]

Esta identidad es general. Sin embargo, condicionar cada carácter a toda
la historia puede requerir demasiados datos y memoria. Por eso se
introducen aproximaciones basadas en contextos más cortos.

------------------------------------------------------------------------

## 3. Cadenas de Markov y modelos de orden finito

Una **cadena de Markov** es un proceso en el que el futuro depende del
pasado únicamente a través de un estado presente. En su forma de primer
orden:

\[ P(X\_{t+1}`\mid `{=tex}X_t,X\_{t-1},`\ldots`{=tex},X_1) =
P(X\_{t+1}`\mid `{=tex}X_t). \]

Esta propiedad se conoce como **propiedad de Markov**.

Aplicada al lenguaje, supone que para predecir el siguiente carácter
basta con conocer el carácter actual. Es una simplificación: en el
lenguaje real, el contexto más lejano también puede influir.

### 3.1 Modelo de orden 1

El modelo de orden 1 aproxima la distribución del siguiente carácter
mediante:

\[ P(X\_{t+1}=j`\mid `{=tex}X_t=i). \]

Se construye contando cuántas veces aparece cada carácter (j)
inmediatamente después de un carácter (i).

Sea (N(i,j)) el número de transiciones observadas de (i) a (j). La
estimación de máxima verosimilitud es:

\[ `\widehat `{=tex}P(j`\mid `{=tex}i) =
`\frac{N(i,j)}{\sum_{k\in\mathcal A}N(i,k)}`{=tex}, \]

donde (`\mathcal `{=tex}A) es el alfabeto o conjunto de símbolos
considerado.

La matriz (P) formada por estas probabilidades es una **matriz de
transición**. Cada fila corresponde al carácter actual y cada columna al
carácter siguiente. Para toda fila con al menos una transición
observada:

\[ `\sum`{=tex}\_{j`\in`{=tex}`\mathcal `{=tex}A}P(j`\mid `{=tex}i)=1.
\]

Por tanto, cada fila representa una distribución de probabilidad.

### 3.2 Modelo de orden 2

El modelo de orden 2 conserva dos caracteres de contexto:

\[ P(X\_{t+1}=k`\mid `{=tex}X\_{t-1}=i,X_t=j). \]

La estimación por frecuencias es:

\[ `\widehat `{=tex}P(k`\mid `{=tex}i,j) = `\frac{N(i,j,k)}`{=tex}
{`\sum`{=tex}\_{r`\in`{=tex}`\mathcal `{=tex}A}N(i,j,r)}, \]

donde (N(i,j,k)) cuenta las apariciones de la secuencia de tres
caracteres (ijk).

En lugar de una matriz bidimensional sencilla, puede representarse como
una colección de distribuciones: una por cada par de caracteres de
contexto.

El orden 2 suele capturar patrones locales más específicos. Por ejemplo,
la probabilidad de una vocal después de `q` depende mucho de si el
carácter anterior fue `u`. A cambio, necesita más observaciones: hay
muchos más contextos posibles y algunos aparecerán pocas veces o nunca.

### 3.3 Comparación entre órdenes

  ------------------------------------------------------------------------------------------------
  Aspecto                 Orden 1                         Orden 2
  ----------------------- ------------------------------- ----------------------------------------
  Contexto                Un carácter                     Dos caracteres

  Probabilidad estimada   (P(x\_{t+1}`\mid `{=tex}x_t))   (P(x\_{t+1}`\mid `{=tex}x\_{t-1},x_t))

  Complejidad             Menor                           Mayor

  Datos por contexto      Generalmente más abundantes     Más dispersos

  Patrones capturados     Dependencias inmediatas         Patrones locales de mayor longitud

  Riesgo                  Contexto demasiado limitado     Contextos sin observaciones
  ------------------------------------------------------------------------------------------------

Aumentar el orden no garantiza automáticamente mejores textos. El
resultado depende del tamaño y la diversidad del corpus, del tratamiento
de contextos no observados y del criterio usado para evaluar la
generación.

------------------------------------------------------------------------

## 4. Corpus, alfabeto y preprocesamiento

Un **corpus** es el conjunto de textos utilizados para estimar el
modelo. En un proyecto de este tipo, los libros digitales pueden
funcionar como fuentes de secuencias de entrenamiento.

Antes de contar transiciones, los documentos deben convertirse en texto
y normalizarse. Las decisiones de preprocesamiento afectan directamente
las probabilidades aprendidas.

### 4.1 Extracción y limpieza

Un flujo habitual es:

1.  Leer los archivos de entrada.
2.  Extraer el contenido textual y descartar elementos estructurales que
    no forman parte de la narración.
3.  Unificar convenciones de codificación.
4.  Decidir cómo tratar mayúsculas, espacios, saltos de línea y
    puntuación.
5.  Definir el alfabeto permitido.
6.  Convertir el texto en una secuencia de caracteres.

No existe una única normalización correcta. Por ejemplo, convertir todo
a minúsculas reduce el tamaño del alfabeto, pero elimina la distinción
entre nombres propios e inicios de oración. Eliminar tildes simplifica
los símbolos, pero pierde información ortográfica del español.

### 4.2 El alfabeto

El alfabeto (`\mathcal `{=tex}A) es el conjunto de caracteres que el
modelo reconoce. Su tamaño se denota:

\[ \|`\mathcal `{=tex}A\|=V. \]

Un alfabeto más amplio conserva más detalles, pero aumenta la cantidad
de contextos posibles. Un alfabeto reducido simplifica el modelo, aunque
puede eliminar información relevante.

Es importante que el tratamiento del texto de entrenamiento y el de
generación sea consistente: ambos deben utilizar las mismas reglas de
normalización y el mismo alfabeto.

------------------------------------------------------------------------

## 5. Estimación de probabilidades y datos escasos

Las frecuencias relativas permiten estimar probabilidades a partir de
observaciones. Si un contexto (c) aparece (n(c)) veces y va seguido del
carácter (j) en (n(c,j)) ocasiones:

\[ `\widehat `{=tex}P(j`\mid `{=tex}c)=`\frac{n(c,j)}{n(c)}`{=tex}. \]

El problema aparece cuando un contexto no se ha observado. En ese caso
(n(c)=0), por lo que la fracción no está definida. También puede ocurrir
que un carácter posible tenga probabilidad estimada cero por no haber
aparecido en el corpus, aunque sí pudiera aparecer en otro texto.

### 5.1 Suavizado de Laplace

Una solución sencilla es añadir una constante positiva (`\alpha`{=tex})
a cada conteo:

\[ `\widehat `{=tex}P\_{`\alpha`{=tex}}(j`\mid `{=tex}c) =
`\frac{n(c,j)+\alpha}`{=tex} {n(c)+`\alpha `{=tex}V}. \]

Cuando (`\alpha=1`{=tex}), se conoce como **suavizado de Laplace**. El
método evita probabilidades exactamente nulas para los símbolos del
alfabeto, aunque puede asignar demasiada masa a eventos poco frecuentes.
Otros métodos de suavizado pueden funcionar mejor en corpus grandes.

Si no se implementa suavizado, el sistema debe definir explícitamente
qué hacer cuando encuentra un contexto sin transiciones observadas:
elegir un contexto de menor orden, reiniciar la secuencia, seleccionar
otro estado inicial o detener la generación.

------------------------------------------------------------------------

## 6. Predicción y generación de texto

La **predicción** y la **generación** están relacionadas, pero no son lo
mismo.

-   **Predicción:** dado un contexto, se calcula una distribución de
    probabilidad para el siguiente carácter.
-   **Generación:** se elige un carácter de esa distribución, se añade a
    la secuencia y se actualiza el contexto. El proceso se repite.

### 6.1 Selección del siguiente carácter

Una estrategia es elegir siempre el carácter de mayor probabilidad:

\[
x\_{t+1}=`\arg`{=tex}`\max`{=tex}\_{j`\in`{=tex}`\mathcal `{=tex}A}P(j`\mid `{=tex}c_t).
\]

Es una selección determinista. Puede producir secuencias repetitivas
porque, ante el mismo contexto, siempre toma la misma decisión.

Otra estrategia es el **muestreo**, en el que el siguiente carácter se
sortea de acuerdo con la distribución estimada:

\[ x\_{t+1}`\sim `{=tex}P(`\cdot`{=tex}`\mid `{=tex}c_t). \]

El muestreo introduce variabilidad. Los caracteres frecuentes aparecen
más a menudo, pero también pueden aparecer alternativas menos probables.

### 6.2 Proceso iterativo

``` mermaid
flowchart TD
    A["Corpus de entrenamiento"] --> B["Preprocesamiento"]
    B --> C["Conteo de contextos y transiciones"]
    C --> D["Estimación de probabilidades"]
    D --> E["Elegir contexto inicial"]
    E --> F["Consultar distribución del siguiente carácter"]
    F --> G["Seleccionar carácter"]
    G --> H["Añadir carácter al texto"]
    H --> I{"¿Se alcanzó la longitud deseada?"}
    I -- "No" --> J["Actualizar contexto"]
    J --> F
    I -- "Sí" --> K["Texto generado"]
```

La elección del contexto inicial también importa. Puede definirse
manualmente, tomarse de un texto semilla o seleccionarse de entre los
contextos observados. Una generación reproducible debe documentar tanto
esa elección como la semilla aleatoria utilizada, si hay muestreo.

------------------------------------------------------------------------

## 7. Teorema de Bayes

El teorema de Bayes relaciona probabilidades condicionales:

\[ P(A`\mid `{=tex}B) = `\frac{P(B\mid A)P(A)}{P(B)}`{=tex},
`\qquad `{=tex}P(B)\>0. \]

En esta expresión:

-   (P(A)) es la probabilidad *a priori* de (A).
-   (P(B`\mid `{=tex}A)) es la verosimilitud de observar (B) si (A) es
    cierto.
-   (P(A`\mid `{=tex}B)) es la probabilidad *a posteriori* de (A)
    después de observar (B).
-   (P(B)) es la probabilidad del dato observado.

En un modelo de caracteres, Bayes puede servir para invertir una
condición o comparar hipótesis. Por ejemplo, puede calcularse la
probabilidad de un carácter anterior dadas ciertas observaciones
posteriores, siempre que se definan claramente los eventos y las
probabilidades involucradas.

**Precaución conceptual:** normalizar conteos de transiciones para
obtener (P(j`\mid `{=tex}i)) es una estimación de probabilidad
condicional. No constituye por sí sola una aplicación del teorema de
Bayes. Para afirmar que se usa Bayes, debe mostrarse qué evento actúa
como hipótesis, cuál es la evidencia y cómo se aplica la fórmula.

------------------------------------------------------------------------

## 8. Álgebra lineal y descomposición en valores singulares (SVD)

Las frecuencias y transiciones pueden organizarse en matrices. El
álgebra lineal permite analizar esas representaciones, comparar patrones
y estudiar estructuras de menor dimensión.

### 8.1 Matriz de frecuencias o transiciones

Una matriz (M) reúne valores asociados a pares de símbolos o contextos.
Por ejemplo, una matriz de conteos de transición puede tener:

\[ M\_{ij}=N(i,j). \]

Al normalizar cada fila se obtiene una matriz de probabilidades de
transición, siempre que las filas tengan conteos positivos.

### 8.2 Descomposición en valores singulares

La descomposición en valores singulares expresa una matriz real (M)
como:

\[ M=U`\Sigma `{=tex}V\^`\top`{=tex}. \]

-   \(U\) contiene vectores singulares izquierdos.
-   \(V\) contiene vectores singulares derechos.
-   (`\Sigma`{=tex}) es una matriz diagonal cuyos elementos son los
    valores singulares, ordenados de mayor a menor.

Los valores singulares indican la importancia relativa de distintas
direcciones o patrones presentes en la matriz.

### 8.3 Aproximación de rango reducido

Si se conservan únicamente los (r) valores singulares más grandes, se
obtiene una aproximación:

\[ M_r=U_r`\Sigma`{=tex}\_rV_r\^`\top`{=tex}. \]

Esta aproximación puede resumir la estructura dominante de los datos y
reducir dimensionalidad. No obstante, aplicar SVD a una matriz de
transiciones no mejora automáticamente la generación. Para que afecte al
modelo, hay que especificar cómo se transforma la matriz aproximada en
probabilidades válidas y cómo se evalúa el efecto.

------------------------------------------------------------------------

## 9. Evaluación de los modelos

Una exposición rigurosa debe explicar cómo se determina si un modelo
funciona bien. La apariencia de un fragmento generado puede ser
ilustrativa, pero no basta como evaluación.

### 9.1 Separación entre entrenamiento y prueba

Una estrategia consiste en dividir el corpus en conjuntos de
entrenamiento y prueba. Las probabilidades se estiman con el primero y
se evalúan sobre el segundo. Esto ayuda a medir si el modelo captura
regularidades que no se limitaron a los ejemplos usados para
construirlo.

La división debe evitar fugas de información. Si fragmentos casi
idénticos aparecen en ambos conjuntos, la evaluación puede resultar
demasiado optimista.

### 9.2 Verosimilitud y perplejidad

Para una secuencia de prueba (x_1,`\ldots`{=tex},x_T), la
log-verosimilitud media condicional puede escribirse como:

\[ `\frac{1}{T-1}`{=tex}`\sum`{=tex}\_{t=2}\^{T}
`\log `{=tex}P(x_t`\mid `{=tex}c_t). \]

La **perplejidad** es:

\[ `\operatorname{PP}`{=tex}(x\_{1:T}) = `\exp`{=tex}`\left`{=tex}(
-`\frac{1}{T-1}`{=tex}
`\sum`{=tex}\_{t=2}\^{T}`\log `{=tex}P(x_t`\mid `{=tex}c_t)
`\right`{=tex}). \]

En general, una perplejidad menor indica que el modelo asigna mayor
probabilidad a los caracteres observados en el conjunto de prueba. Las
comparaciones deben usar el mismo corpus de prueba, la misma
tokenización y convenciones equivalentes para los contextos no
observados.

### 9.3 Evaluación cualitativa

También pueden examinarse:

-   Legibilidad de las secuencias.
-   Frecuencia de repeticiones y ciclos.
-   Presencia de patrones ortográficos plausibles.
-   Comportamiento ante contextos no observados.
-   Diferencias entre los textos generados por modelos de distintos
    órdenes.

La evaluación cualitativa complementa, pero no reemplaza, las métricas
cuantitativas.

------------------------------------------------------------------------

## 10. Limitaciones y posibles mejoras

Un modelo de Markov de orden bajo aprende regularidades locales. Por eso
puede producir combinaciones de letras parecidas a las del corpus sin
reproducir necesariamente su gramática, coherencia temática o
significado.

Entre las limitaciones principales están:

1.  **Memoria contextual limitada:** el modelo solo considera uno o dos
    caracteres anteriores.
2.  **Dispersión de datos:** al aumentar el orden, hay más contextos
    posibles y menos observaciones por contexto.
3.  **Dependencia del corpus:** el modelo reproduce sesgos, vocabulario
    y convenciones de los textos de entrenamiento.
4.  **Sensibilidad al preprocesamiento:** eliminar signos, tildes o
    espacios cambia las distribuciones estimadas.
5.  **Contextos no observados:** algunos estados no tienen transiciones
    conocidas.
6.  **Evaluación insuficiente:** ejemplos atractivos no demuestran por
    sí solos que el modelo generalice.

Mejoras posibles incluyen suavizado, retroceso a modelos de orden
inferior, corpus más amplios y variados, evaluación con perplejidad,
comparación sistemática de órdenes y métodos que modelen dependencias
más largas.

------------------------------------------------------------------------

## 11. Cómo conectar la teoría con el trabajo práctico

Para presentar el proyecto, conviene vincular cada concepto con una
operación verificable del programa. La tabla siguiente es una guía: debe
completarse con los nombres de funciones, archivos y resultados que
efectivamente existan en el repositorio.

  -----------------------------------------------------------------------
  Concepto teórico        Pregunta que debe       Evidencia útil para la
                          responder la            exposición
                          implementación          
  ----------------------- ----------------------- -----------------------
  Corpus                  ¿Qué documentos se      Lista de archivos y
                          usan?                   cantidad de texto

  Preprocesamiento        ¿Qué caracteres se      Ejemplos antes y
                          conservan o             después
                          transforman?            

  Frecuencias             ¿Cómo se cuentan los    Tabla o histograma
                          caracteres?             

  Markov de orden 1       ¿Cómo se cuentan y      Matriz de transición
                          normalizan pares?       

  Markov de orden 2       ¿Cómo se cuentan        Ejemplos de contextos
                          ternas?                 de dos caracteres

  Predicción              ¿Cómo se obtiene la     Predicción para un
                          distribución siguiente? contexto concreto

  Generación              ¿Cómo se selecciona y   Texto generado y
                          añade cada carácter?    configuración

  Bayes                   ¿Qué eventos se         Cálculo numérico con
                          relacionan mediante la  eventos definidos
                          fórmula?                

  SVD                     ¿Qué matriz se          Valores singulares o
                          descompone y para qué?  reconstrucción

  Evaluación              ¿Cómo se compara el     Métricas y ejemplos
                          desempeño?              bajo el mismo protocolo
  -----------------------------------------------------------------------

### 11.1 Ejemplo pequeño de estimación

Supóngase que, en un corpus simplificado, el carácter `a` aparece
seguido de `s` seis veces, de `r` tres veces y de `l` una vez. Entonces:

\[ N(a,s)=6,`\quad `{=tex}N(a,r)=3,`\quad `{=tex}N(a,l)=1. \]

El total de transiciones