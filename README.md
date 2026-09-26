# Una introducción a los Modelos Gráficos Probabilistas

Este repositorio contiene las implementaciones computacionales desarrolladas como parte de mi tesis de maestría **"Una introducción a los Modelos Gráficos Probabilistas"**.

El objetivo de las implementaciones es ilustrar algunos de los principales métodos de aprendizaje e inferencia en redes bayesianas, así como estudiar su comportamiento bajo diferentes condiciones, incluyendo datos ruidosos, datos faltantes y variables ocultas.

Los experimentos se realizan principalmente sobre redes bayesianas discretas con variables binarias y datos simulados, lo que permite comparar las estructuras y parámetros estimados con los utilizados para generar los datos.

---

## Contenido

Las implementaciones se agrupan en los siguientes temas:

1. [Generación de datos](#generación-de-datos)
2. [Aprendizaje de estructura](#aprendizaje-de-estructura)
   - [Algoritmo K2](#algoritmo-k2)
   - [Algoritmo SGS](#algoritmo-sgs)
   - [Algoritmo PC](#algoritmo-pc)
3. [Aprendizaje de parámetros](#aprendizaje-de-parámetros)
   - [Máxima verosimilitud y suavizamiento laplaciano](#máxima-verosimilitud-y-suavizamiento-laplaciano)
   - [Expectation-Maximization](#expectation-maximization)
4. [EM con inferencia variacional](#em-con-inferencia-variacional)
5. [EM estructural](#em-estructural)

---

# Generación de datos

Para los experimentos principales se utiliza una red bayesiana compuesta por diez variables binarias

$$X_1,\ldots,X_{10},$$

cuyos valores pertenecen a {0,1}.

A partir de una estructura y de tablas de probabilidad condicional previamente especificadas se generan muestras de la distribución conjunta mediante la factorización

$$P(X_1,\ldots,X_n)=\prod_{i=1}^{n}P(X_i\mid Pa(X_i)).$$

---

# Aprendizaje de estructura

El repositorio incluye implementaciones de métodos de aprendizaje de estructura pertenecientes tanto a la familia de **búsqueda y puntuación** como a la familia de **métodos basados en restricciones**.

## Algoritmo K2

Se implementa el algoritmo **K2** para aprender la estructura de una red bayesiana a partir de un orden causal previamente especificado.

K2 construye de manera incremental el conjunto de padres de cada variable, seleccionando aquellos que producen una mejora en una función de puntuación local.

Se estudian tres órdenes causales:

- un orden compatible con la red verdadera;
- un orden alternativo;
- un orden incompatible con la dirección causal utilizada para generar los datos.

Para cada orden se consideran dos escenarios:

- datos sin ruido;
- datos con ruido tipo bit-flip.

El experimento permite observar la sensibilidad del algoritmo K2 respecto al orden causal proporcionado.

---

## Algoritmo SGS

También se implementa el algoritmo **SGS (Spirtes–Glymour–Scheines)**.

SGS pertenece a los métodos de aprendizaje basados en restricciones y utiliza pruebas de independencia condicional para eliminar aristas de un grafo completo inicial.

Las pruebas realizadas tienen la forma

$$X_i \perp X_j \mid S,$$

donde $S$ es un subconjunto de las variables restantes.

En la implementación se utilizan pruebas chi-cuadrada de independencia condicional con nivel de significancia

$$\alpha = 0.05.$$

Se consideran diferentes órdenes de recorrido de las variables con el objetivo de analizar posibles variaciones en las orientaciones obtenidas.

También se estudia el comportamiento del algoritmo cuando los datos contienen ruido.

---

## Algoritmo PC

Se implementa además el algoritmo **PC**, otro método basado en pruebas de independencia condicional.

A diferencia de SGS, PC restringe los conjuntos condicionantes a subconjuntos de las adyacencias actuales de cada nodo. Esto reduce considerablemente el número de pruebas necesarias en redes poco densas.

En los experimentos se consideran conjuntos condicionantes de tamaño máximo 3 y se estudian:

- diferentes órdenes de las variables;
- datos sin ruido;
- datos con ruido.

Los resultados se comparan con los obtenidos mediante SGS y con la estructura verdadera de la red.

---

# Aprendizaje de parámetros

Una vez fijada la estructura de la red bayesiana, se estiman sus tablas de probabilidad condicional.

Para variables binarias se estiman probabilidades de la forma

$$P(X_i=1\mid Pa(X_i)).$$

## Máxima verosimilitud y suavizamiento laplaciano

El estimador de máxima verosimilitud utilizado es

$$\widehat P(X_i=1\mid pa_i)=\frac{N(X_i=1,pa_i)}{N(pa_i)}.$$

Para evitar probabilidades iguales a cero o uno debidas a configuraciones poco observadas, también se implementa suavizamiento laplaciano:

$$\widehat P_{\mathrm{Lap}}(X_i=1\mid pa_i)=\frac{N(X_i=1,pa_i)+1}{N(pa_i)+2}.$$

Las tablas estimadas se comparan con las tablas verdaderas utilizando:

- MAE (Mean Absolute Error);
- RMSE (Root Mean Squared Error);
- error máximo.
---

# Expectation-Maximization

Se implementa el algoritmo **Expectation-Maximization (EM)** para estimar los parámetros de redes bayesianas con datos incompletos.

Se consideran dos escenarios.

## Datos faltantes

A partir de la base de datos completa se eliminan aleatoriamente algunos valores individuales.

Para cada registro se separan las variables observadas y faltantes:

$$X_{\mathrm{obs}},\qquad X_{\mathrm{mis}}.$$

### Etapa E

Se calcula

$$P(X_{\mathrm{mis}} \mid X_{\mathrm{obs}}, \theta^{(t)})$$

mediante enumeración exacta de todas las configuraciones posibles de las variables faltantes.

### Etapa M

Las probabilidades posteriores obtenidas en la etapa E se utilizan para calcular conteos esperados y actualizar las tablas de probabilidad condicional.

Para una variable binaria:

```math
\widehat{P}(X_i=1 \mid \mathrm{Pa}(X_i)=u)
=
\frac{N_{1,u}^{*}}{N_{0,u}^{*}+N_{1,u}^{*}}
```

---

## Variable oculta

También se analiza un escenario en el que una variable completa de la red no es observada.

En el experimento se considera $X_6$ como variable oculta. Sus valores deben inferirse utilizando únicamente las demás variables observadas.

---

# EM con inferencia variacional

Se implementan dos versiones del algoritmo EM para estudiar el costo de la inferencia realizada durante la etapa E:

- **EM con inferencia exacta**;
- **EM con inferencia variacional mean-field**.

Para estos experimentos se generan bases con distintas proporciones de valores faltantes.

---

# EM estructural

Finalmente, se implementa el algoritmo **Structural EM** para aprender simultáneamente la estructura y los parámetros de una red bayesiana a partir de datos incompletos.

El experimento utiliza las diez variables binarias de la red original y datos en los que cada observación contiene exactamente un valor faltante.

Debido a que Structural EM realiza una búsqueda local, se utilizan diferentes estructuras iniciales.

Se realizan tres ejecuciones partiendo de distintos árboles dirigidos aleatorios.


---

# Requisitos

Las implementaciones fueron desarrolladas en Python.

Las principales bibliotecas utilizadas en los experimentos incluyen herramientas para:

- manipulación de datos;
- cálculo numérico;
- pruebas estadísticas;
- construcción y visualización de grafos;
- generación de gráficas.


---

# Ejecución

Los experimentos pueden ejecutarse de manera independiente.

Cada implementación contiene los parámetros necesarios para reproducir los experimentos presentados en la tesis.

---

# Tesis

Este repositorio contiene el código asociado con la tesis:

**Una introducción a los Modelos Gráficos Probabilistas**

Tesis de maestría.

Centro de Investigación en Matemáticas (CIMAT).

---

# Autor

**Luz María Salazar Manjarrez**
