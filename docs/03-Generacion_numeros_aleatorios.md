# Generación de números pseudoaleatorios con distribución uniforme {#cap3}




Como ya se comentó, los distintos métodos de simulación requieren disponer de secuencias de números pseudoaleatorios que imiten las propiedades de generaciones independientes de una distribución $\mathcal{U}(0,1)$. 
En primer lugar nos centraremos en el caso de los generadores congruenciales. A pesar de su simplicidad, podrían ser adecuados en muchos casos y constituyen la base de los generadores avanzados habitualmente considerados.
Posteriormente se dará una visión de las diferentes herramientas para estudiar la calidad de un generador de números pseudoaleatorios.

## Generadores congruenciales (lineales) {#gen-cong}

Se basan en la idea de considerar una combinación lineal de los últimos $k$ enteros generados y calcular su resto al dividir por un entero fijo $m$. 
En el método congruencial simple (de orden $k = 1$), partiendo de una semilla inicial $x_0$, el algoritmo secuencial es el siguiente:
$$\begin{aligned}
x_{i}  & = (ax_{i-1}+c) \operatorname{mod} m \\
u_{i}  & = \dfrac{x_{i}}{m} \\
i  & =1,2,\ldots
\end{aligned}$$ 
donde $a$ (*multiplicador*), $c$ (*incremento*) y $m$ (*módulo*) son parámetros enteros del generador fijados de antemano. Si $c=0$ el generador se denomina congruencial *multiplicativo* (Lehmer, 1951) y en caso contrario se dice que es *mixto* (Rotenburg, 1960).

Este método está implementado en el fichero *RANDC.R*, tratando de imitar el funcionamiento de un generador de R (aunque de un forma no muy eficiente):

```r
# Generador congruencial de números pseudoaleatorios
# ==================================================

# --------------------------------------------------
# initRANDC(semilla,a,c,m)
#   Selecciona el generador congruencial
#   Por defecto RANDU de IBM con semilla del reloj
#   OJO: No se hace ninguna verificación de los parámetros
initRANDC <- function(semilla=as.numeric(Sys.time()), a=2^16+3, c=0, m=2^31) {
  .semilla <<- as.double(semilla) %% m  #Cálculos en doble precisión
  .a <<- a
  .c <<- c
  .m <<- m
  return(invisible(list(semilla=.semilla,a=.a,c=.c,m=.m))) #print(initRANDC())
}

# --------------------------------------------------
# RANDC()
#   Genera un valor pseudoaleatorio con el generador congruencial
#   Actualiza la semilla (si no existe llama a initRANDC)
RANDC <- function() {
    if (!exists(".semilla", envir=globalenv())) initRANDC()
    .semilla <<- (.a * .semilla + .c) %% .m
    return(.semilla/.m)
}

# --------------------------------------------------
# RANDCN(n)
#   Genera un vector de valores pseudoaleatorios con el generador congruencial
#   (por defecto de dimensión 1000)
#   Actualiza la semilla (si no existe llama a initRANDC)
RANDCN <- function(n=1000) {
    x <- numeric(n)
    for(i in 1:n) x[i]<-RANDC()
    return(x)
    # return(replicate(n,RANDC()))  # Alternativa más rápida    
}

initRANDC(543210)       # Fijar semilla 543210 para reproductibilidad
```


    
\BeginKnitrBlock{remark}<div class="remark">\iffalse{} <span class="remark"><em>Nota: </em></span>  \fi{}Para evitar problemas computacionales, se recomienda emplear
el método de Schrage (ver Bratley *et al.*, 1987; L'Ecuyer, 1988)</div>\EndKnitrBlock{remark}


Ejemplos:

-   $c=0$, $a=2^{16}+3=65539$ y $m=2^{31}$, generador `RANDU` de IBM
    (**no recomendable**).

-   $c=0$, $a=7^{5}=16807$ y $m=2^{31}-1$ (primo de Mersenne), Park y Miller (1988)
    “minimal standar”, empleado por las librerías IMSL y NAG.
    
-   $c=0$, $a=48271$ y $m=2^{31}-1$ actualización del “minimal standar” 
    propuesta por Park, Miller y Stockmeyer (1993).
    
Los parámetros y la semilla determinan los valores generados:
$$x_{i}=\left(  a^{i}x_0+c\frac{a^{i}-1}{a-1}\right)  \operatorname{mod}m$$

A pesar de su simplicidad, una adecuada elección de los parámetros
permite obtener de manera eficiente secuencias de números
“aparentemente” i.i.d. $\mathcal{U}(0,1)$.

El procedimiento habitual solía ser escoger $m$ de forma que la operación del módulo se pudiese realizar de forma muy eficiente, para posteriormente seleccionar $c$ y $a$ de forma que el período fuese lo más largo posible (o suficientemente largo).


\BeginKnitrBlock{theorem}<div class="theorem"><span class="theorem" id="thm:unnamed-chunk-4"><strong>(\#thm:unnamed-chunk-4) </strong></span>(Hull y Dobell, 1962)

Un generador congruencial tiene período máximo ($p=m$) si y solo si:

1.  $c$ y $m$ son primos relativos (i.e. $m.c.d.(c, m) = 1$).

2.  $a-1$ es múltiplo de todos los factores primos de $m$ (i.e.
    $a \equiv 1 \operatorname{mod}q$, para todo $q$ factor primo de $m$).

3.  Si $m$ es múltiplo de $4$, entonces $a-1$ también lo ha de
    ser (i.e. $m \equiv 0 \operatorname{mod}4\Rightarrow a \equiv
    1 \operatorname{mod}4$).
    
.</div>\EndKnitrBlock{theorem}

Algunas consecuencias:

-   Si $m$ primo, $p=m$ si y solo si $a=1$.

-   Un generador multiplicativo no cumple la condición 1 ($m.c.d.(0, m)=m$).


\BeginKnitrBlock{theorem}<div class="theorem"><span class="theorem" id="thm:unnamed-chunk-5"><strong>(\#thm:unnamed-chunk-5) </strong></span>
Un generador multiplicativo tiene período máximo ($p=m-1$) si:

1.  $m$ es primo.

2.  $a$ es una raiz primitiva de $m$ (i.e. el menor entero $q$ tal
    que $a^{q}=1\operatorname{mod}m$ es $q=m-1$).

.    </div>\EndKnitrBlock{theorem}

Además de preocuparse de la longitud del ciclo, las secuencias
generadas deben aparentar muestras i.i.d. $\mathcal{U}(0,1)$. 

Uno de los principales problemas es que los valores generados pueden mostrar una clara estructura reticular.
Este es el caso por ejemplo del generador RANDU de IBM muy empleado en la década de los 70 (ver Figura \@ref(fig:randu)). 


```r
system.time(u <- RANDCN(9999))  # Generar
```

```
##    user  system elapsed 
##    0.01    0.00    0.02
```

```r
xyz <- matrix(u, ncol = 3, byrow = TRUE)

library(plot3D)
points3D(xyz[,1], xyz[,2], xyz[,3], colvar = NULL, phi = 60, theta = -50, pch = 21, cex = 0.2)
```

\begin{figure}[!htb]

{\centering \includegraphics[width=0.7\linewidth]{03-Generacion_numeros_aleatorios_files/figure-latex/randu-1} 

}

\caption{Grafico de dispersión de tripletas del generador RANDU de IBM (contenidas en 15 planos)}(\#fig:randu)
\end{figure}

```r
# Alternativamente se podría utilizar la función `plot3d` del paquete `rgl`,
# y pulsando con el ratón se podría rotar la figura para ver los hiperplanos:
# library(rgl)
# plot3d(xyz) 
```

En general todos los generadores de este tipo van a presentar estructuras reticulares.
Marsaglia (1968) demostró que las $k$-uplas de un generadores multiplicativo están contenidas en a lo sumo $\left(k!m\right)^{1/k}$ hiperplanos paralelos (para más detalles sobre la estructura reticular, ver por ejemplo Ripley, 1987, sección 2.7).
Por tanto habría que seleccionar adecuadamente $m$ y $c$ ($a$ solo influiría en la pendiente) de forma que la estructura reticular sea impreceptible teniendo en cuenta el número de datos que se pretende generar (por ejemplo de forma que la distancia mínima entre los puntos sea próxima a la esperada en teoría).

Se han propuesto diversas pruebas (ver Sección \@ref(calgen)) para
determinar si un generador tiene problemas de este tipo y se han
realizado numerosos estudios para determinadas familias (e.g. Park y
Miller, 1988, estudiaron que parámetros son adecuados para $m=2^{31}-1$).

-   En cualquier caso, se recomienda considerar un “periodo de
    seguridad” $\approx \sqrt{p}$ para evitar este tipo de problemas.

-   Aunque estos generadores tiene limitaciones en su capacidad para
    producir secuencias muy largas de números i.i.d. $\mathcal{U}(0,1)$,
    es un elemento básico en generadores más avanzados.


### Otros generadores

Se han considerado diversas extensiones del generador congruencial lineal simple:

-   Lineal múltiple: 
    $x_{i}= a_0 + a_1 x_{i-1} + a_2 x_{i-2} + \cdots + a_{k} x_{i-k} \operatorname{mod} m$,
    con periodo $p\leq m^{k}-1$.

-   No lineal: 
    $x_{i} = f\left(  x_{i-1}, x_{i-2}, \cdots, x_{i-k} \right) \operatorname{mod} m$. 
    Por ejemplo $x_{i} = a_0 + a_1 x_{i-1} + a_2 x_{i-1}^2 \operatorname{mod} m$.

-   Matricial: 
    $\boldsymbol{x}_{i} = A_0 + A_1\boldsymbol{x}_{i-1} 
    + A_2\boldsymbol{x}_{i-2} + \cdots 
    + A_{k}\boldsymbol{x}_{i-k} \operatorname{mod} m$.
    
El generador *Mersenne-Twister* (Matsumoto y Nishimura, 1998), empleado por defecto en R, de periodo $2^{19937}-1$ y equidistribution en 623 dimensiones, se puede expresar como un generador congruencial matricial lineal.

Un caso particular del generador lineal múltiple son los denominados *generadores de registros desfasados* (más relacionados con la Criptografía).
Se generan bits de forma secuencial considerando $m=2$ y $a_{i} \in \left \{ 0,1\right \}$ y se van combinando $l$ bits para obtener valores en el intervalo $(0, 1)$, por ejemplo $u_i = 0 . x_{it+1} x_{it+2} \ldots x_{it+l}$, siendo $t$ un parámetro denominado *aniquilación* (Tausworthe, 1965). 
Los cálculos se pueden realizar rápidamente mediante operaciones lógicas (los sumandos de la combinación lineal se traducen en un "o" exclusivo XOR), empleando directamente los registros del procesador (ver por ejemplo, Ripley, 1987, Algoritmo 2.1).

Otras alternativas consisten en la combinanción de varios generadores, las más empleadas son:

-   Combinar las salidas: por ejemplo $u_{i}=\sum_{i=1}^L u_{i}^{(l)} \operatorname{mod} 1$, donde $u_{i}^{(l)}$ es el $i$-ésimo valor obtenido con el generador $l$.

-   Barajar las salidas: por ejemplo se crea una tabla empleando un generador y se utiliza otro para seleccionar el índice del valor que se va a devolver y posteriormente actualizar.

El generador *L'Ecuyer-CMRG* (L'Ecuyer, 1999), empleado como base para la generación de múltiples secuencias en el paquete `parallel`, combina dos generadores concruenciales lineales múltiples de orden $k=3$ (el periodo aproximado es $2^{191}$).
    
\BeginKnitrBlock{exercise}<div class="exercise"><span class="exercise" id="exr:unnamed-chunk-6"><strong>(\#exr:unnamed-chunk-6) </strong></span></div>\EndKnitrBlock{exercise}

Considera el generador congruencial definido por: 
$$\begin{aligned}
x_{n+1}  & =(5x_{n}+1)\ \operatorname{mod}\ 512,\nonumber\\
u_{n+1}  & =\frac{x_{n+1}}{512},\ n=0,1,\dots\nonumber
\end{aligned}$$
(de ciclo máximo).

NOTA: El algoritmo está implementado en el fichero *RANDC.R* y se muestra en la Sección \@ref(gen-cong).

a)  Generar 500 valores de este generador, obtener el tiempo de CPU,
    representar su distribución mediante un histograma (en escala
    de densidades) y compararla con la densidad teórica.
   
    
    ```r
    initRANDC(321, 5, 1, 512)       # Fijar semilla para reproductibilidad
    nsim <- 500
    system.time(u <- RANDCN(nsim))  # Generar
    ```
    
    ```
    ##    user  system elapsed 
    ##       0       0       0
    ```
    
    ```r
    hist(u, freq = FALSE)
    abline(h = 1)                   # Equivalente a curve(dunif(x, 0, 1), add = TRUE)
    ```
    
    \begin{figure}[!htb]
    
    {\centering \includegraphics[width=0.7\linewidth]{03-Generacion_numeros_aleatorios_files/figure-latex/ejcona-1} 
    
    }
    
    \caption{Histograma de los valores generados}(\#fig:ejcona)
    \end{figure}

    En este caso concreto la distribución de los valores generados es aparentemente más uniforme de lo que cabría esperar, lo que induciría a sospechar de la calidad de este gerenador.

b)  Calcular la media de las simulaciones (`mean`) y compararla con
    la teórica.
    
    La aproximación por simulación de la media teórica es:
    
    
    ```r
    mean(u)
    ```
    
    ```
    ## [1] 0.4999609
    ```
    
    La media teórica es 0.5. 
    Error absoluto $\ensuremath{3.90625\times 10^{-5}}$.

c)  Aproximar (mediante simulación) la probabilidad del intervalo
    $(0.4;0.8)$ y compararla con la teórica.

    La probabilidad teórica es 0.8 - 0.4 = 0.4
    
    La aproximación mediante simulación:
    
    
    ```r
    sum((0.4 < u) & (u < 0.8))/nsim
    ```
    
    ```
    ## [1] 0.402
    ```
    
    ```r
    mean((0.4 < u) & (u < 0.8))     # Alternativa
    ```
    
    ```
    ## [1] 0.402
    ```


Análisis de la calidad de un generador {#calgen}
--------------------------------------

Para verificar si un generador tiene las propiedades estadísticas
deseadas hay disponibles una gran cantidad de test de hipótesis
(baterías de contrastes) y métodos gráficos:

-   Contrastes genéricos de bondad de ajuste y aleatoriedad.

-   Contrastes específicos para generadores aleatorios.

Se trata principalmente de contrastar si las muestras generadas son
i.i.d. $\mathcal{U}\left(0,1\right)$ (análisis univariante).
Aunque los métodos más avanzados tratan normalmente de
contrastar si las $k$-uplas:

$$(U_{t+1},U_{t+2},...,U_{t+k-1}); \ t=(i-1)k, \ i=1,...,m$$

son i.i.d. $\mathcal{U}\left(0,1\right)^{k}$ (uniformes
independientes en el hipercubo; análisis multivariante).

**Nos centraremos en los métodos genéricos**.
Pueden usarse en:

-   Evaluación de generadores aleatorios

-   Evaluación de generadores de variables aleatorias

-   Modelado de entradas de modelos de simulación

Uno de los contrastes más conocidos es el test ji-cuadrado de bondad de ajuste
(`chisq.test` para el caso discreto). 
Aunque si la variable de interés es continua, habría que discretizarla 
(con la correspondiente perdida de información). 
Por ejemplo, se podría emplear la siguiente función 
(que imita a las incluídas en `R`):



```r
#-------------------------------------------------------------------------------
# chisq.test.cont(x, distribution, nclasses, output, nestpar,...)
#-------------------------------------------------------------------------------
# Realiza el test ji-cuadrado de bondad de ajuste para una distribución continua
# discretizando en intervalos equiprobables.
# Parámetros:
#   distribution = "norm","unif",etc
#   nclasses = floor(length(x)/5)
#   output = TRUE
#   nestpar = 0= nº de parámetros estimados
#   ... = parámetros distribución
# Ejemplo:
#   chisq.test.cont(x, distribution="norm", nestpar=2, mean=mean(x), sd=sqrt((nx-1)/nx)*sd(x))
#-------------------------------------------------------------------------------
chisq.test.cont <- function(x, distribution = "norm", nclasses = floor(length(x)/5), 
    output = TRUE, nestpar = 0, ...) {
    # Funciones distribución
    q.distrib <- eval(parse(text = paste("q", distribution, sep = "")))
    d.distrib <- eval(parse(text = paste("d", distribution, sep = "")))
    # Puntos de corte
    q <- q.distrib((1:(nclasses - 1))/nclasses, ...)
    tol <- sqrt(.Machine$double.eps)
    xbreaks <- c(min(x) - tol, q, max(x) + tol)
    # Gráficos y frecuencias
    if (output) {
        xhist <- hist(x, breaks = xbreaks, freq = FALSE, lty = 2, border = "grey50")
        curve(d.distrib(x, ...), add = TRUE)
    } else {
        xhist <- hist(x, breaks = xbreaks, plot = FALSE)
    }
    # Cálculo estadístico y p-valor
    O <- xhist$counts  # Equivalente a table(cut(x, xbreaks)) pero más eficiente
    E <- length(x)/nclasses
    DNAME <- deparse(substitute(x))
    METHOD <- "Pearson's Chi-squared test"
    STATISTIC <- sum((O - E)^2/E)
    names(STATISTIC) <- "X-squared"
    PARAMETER <- nclasses - nestpar - 1
    names(PARAMETER) <- "df"
    PVAL <- pchisq(STATISTIC, PARAMETER, lower.tail = FALSE)
    # Preparar resultados
    classes <- format(xbreaks)
    classes <- paste("(", classes[-(nclasses + 1)], ",", classes[-1], "]", 
        sep = "")
    RESULTS <- list(classes = classes, observed = O, expected = E, residuals = (O - 
        E)/sqrt(E))
    if (output) {
        cat("\nPearson's Chi-squared test table\n")
        print(as.data.frame(RESULTS))
    }
    if (any(E < 5)) 
        warning("Chi-squared approximation may be incorrect")
    structure(c(list(statistic = STATISTIC, parameter = PARAMETER, p.value = PVAL, 
        method = METHOD, data.name = DNAME), RESULTS), class = "htest")
}
```
Por ejemplo, continuando con el generador congruencial anterior, obtendríamos:


```r
chisq.test.cont(u, distribution = "unif", 
                nclasses = 10, nestpar = 0, min = 0, max = 1)
```



\begin{center}\includegraphics[width=0.7\linewidth]{03-Generacion_numeros_aleatorios_files/figure-latex/unnamed-chunk-10-1} \end{center}

```
## 
## Pearson's Chi-squared test table
##                          classes observed expected  residuals
## 1  (-1.490116e-08, 1.000000e-01]       51       50  0.1414214
## 2  ( 1.000000e-01, 2.000000e-01]       49       50 -0.1414214
## 3  ( 2.000000e-01, 3.000000e-01]       49       50 -0.1414214
## 4  ( 3.000000e-01, 4.000000e-01]       50       50  0.0000000
## 5  ( 4.000000e-01, 5.000000e-01]       51       50  0.1414214
## 6  ( 5.000000e-01, 6.000000e-01]       51       50  0.1414214
## 7  ( 6.000000e-01, 7.000000e-01]       49       50 -0.1414214
## 8  ( 7.000000e-01, 8.000000e-01]       50       50  0.0000000
## 9  ( 8.000000e-01, 9.000000e-01]       50       50  0.0000000
## 10 ( 9.000000e-01, 9.980469e-01]       50       50  0.0000000
```

```
## 
## 	Pearson's Chi-squared test
## 
## data:  u
## X-squared = 0.12, df = 9, p-value = 1
```

**Importante**:

Empleando los métodos genéricos del modo habitual, desconfiamos del
generador si la muestra/secuencia no se ajusta a la distribución
teórica ($p$-valor $\leq \alpha$).
En este caso además, **también se sospecha si se ajusta demasiado
bien** a la distribución teórica ($p$-valor $\geq1-\alpha$).

Otro contraste de bondad de ajuste muy conocido 
es el test de Kolmogorov-Smirnov, implementado en `ks.test`.

\BeginKnitrBlock{exercise}<div class="exercise"><span class="exercise" id="exr:unnamed-chunk-11"><strong>(\#exr:unnamed-chunk-11) </strong></span></div>\EndKnitrBlock{exercise}

Continuando con el generador congruencial anterior: 


```r
initRANDC(321, 5, 1, 512)       # Fijar semilla para reproductibilidad
nsim <- 500
system.time(u <- RANDCN(nsim))  # Generar
```

    
a)  Realizar el contraste de Kolmogorov-Smirnov para estudiar el
    ajuste a una $\mathcal{U}(0,1)$.
    
    
    ```r
    # Distribución empírica
    curve(ecdf(u)(x), type = "s", lwd = 2)
    curve(punif(x, 0, 1), add = TRUE)
    ```
    
    
    
    \begin{center}\includegraphics[width=0.7\linewidth]{03-Generacion_numeros_aleatorios_files/figure-latex/unnamed-chunk-13-1} \end{center}
    
    ```r
    # Test de Kolmogorov-Smirnov
    ks.test(u, "punif", 0, 1)
    ```
    
    ```
    ## 
    ## 	One-sample Kolmogorov-Smirnov test
    ## 
    ## data:  u
    ## D = 0.0033281, p-value = 1
    ## alternative hypothesis: two-sided
    ```
    
b)  Obtener el gráfico secuencial y el de dispersión retardado, ¿se
    observa algún problema?

    Gráfico secuencial:
    
    
    ```r
    plot(as.ts(u))
    ```
    
    
    
    \begin{center}\includegraphics[width=0.7\linewidth]{03-Generacion_numeros_aleatorios_files/figure-latex/unnamed-chunk-14-1} \end{center}
    
    Gráfico de dispersión retardado:
    
    
    ```r
    plot(u[-nsim],u[-1])
    ```
    
    
    
    \begin{center}\includegraphics[width=0.7\linewidth]{03-Generacion_numeros_aleatorios_files/figure-latex/unnamed-chunk-15-1} \end{center}

c)  Estudiar las correlaciones del vector $(u_{i},u_{i+k})$, con
    $k=1,...,10$. Contrastar si son nulas.

    Correlaciones:
    
    
    ```r
    acf(u)
    ```
    
    
    
    \begin{center}\includegraphics[width=0.7\linewidth]{03-Generacion_numeros_aleatorios_files/figure-latex/unnamed-chunk-16-1} \end{center}
    
    Test de Ljung-Box:
    
    
    ```r
    Box.test(u, lag = 10, type = "Ljung")
    ```
    
    ```
    ## 
    ## 	Box-Ljung test
    ## 
    ## data:  u
    ## X-squared = 22.533, df = 10, p-value = 0.01261
    ```


### Repetición de contrastes

Los contrastes se plantean habitualmente desde el punto de vista de
la inferencia estadística en la práctica: se realiza una prueba
sobre la única muestra disponible. Si se realiza una única prueba, 
en las condiciones de $H_0$ hay
una probabilidad $\alpha$ de rechazarla.
En simulación tiene mucho más sentido realizar un gran número de
pruebas:

-   La proporción de rechazos debería aproximarse al valor de
    $\alpha$(se puede comprobar para distintos valores de $\alpha$).

-   La distribución del estadístico debería ajustarse a la teórica
    bajo $H_0$(se podría realizar un nuevo contraste de bondad
    de ajuste).

-   Los *p*-valores obtenidos deberían ajustarse a una
    $\mathcal{U}\left(0,1\right)$ (se podría realizar también un
    contraste de bondad de ajuste).

Este procedimiento es también el habitual para validar un método de
contraste de hipótesis por simulación.

\BeginKnitrBlock{example}<div class="example"><span class="example" id="exm:unnamed-chunk-18"><strong>(\#exm:unnamed-chunk-18) </strong></span></div>\EndKnitrBlock{example}

Consideramos el generador congruencial RANDU:


```r
# Valores iniciales
initRANDC(543210)   # Fijar semilla para reproductibilidad
# set.seed(543210)
n <- 500
nsim <- 1000
estadistico <- numeric(nsim)
pvalor <- numeric(nsim)

# Realizar contrastes
for(isim in 1:nsim) {
  u <- RANDCN(n)    # Generar
  # u <- runif(n)
  tmp <- chisq.test.cont(u, distribution="unif", 
                        nclasses=100, output=FALSE, nestpar=0, min=0, max=1)
  estadistico[isim] <- tmp$statistic
  pvalor[isim] <- tmp$p.value
}
```

Proporción de rechazos:


```r
# cat("\nProporción de rechazos al 1% =", sum(pvalor < 0.01)/nsim, "\n")
cat("\nProporción de rechazos al 1% =", mean(pvalor < 0.01), "\n")
```

```
## 
## Proporción de rechazos al 1% = 0.014
```

```r
# cat("Proporción de rechazos al 5% =", sum(pvalor < 0.05)/nsim, "\n")
cat("Proporción de rechazos al 5% =", mean(pvalor < 0.05), "\n")
```

```
## Proporción de rechazos al 5% = 0.051
```

```r
# cat("Proporción de rechazos al 10% =", sum(pvalor < 0.1)/nsim, "\n")
cat("Proporción de rechazos al 10% =", mean(pvalor < 0.1), "\n")
```

```
## Proporción de rechazos al 10% = 0.112
```

Análisis del estadístico contraste:


```r
# Histograma
hist(estadistico, breaks = "FD", freq=FALSE)
curve(dchisq(x,99), add=TRUE)
```



\begin{center}\includegraphics[width=0.7\linewidth]{03-Generacion_numeros_aleatorios_files/figure-latex/unnamed-chunk-21-1} \end{center}

```r
# Test ji-cuadrado
# chisq.test.cont(estadistico, distribution="chisq", nclasses=20, nestpar=0, df=99)
# Test de Kolmogorov-Smirnov
ks.test(estadistico, "pchisq", df=99)
```

```
## 
## 	One-sample Kolmogorov-Smirnov test
## 
## data:  estadistico
## D = 0.023499, p-value = 0.6388
## alternative hypothesis: two-sided
```

Análisis de los p-valores:


```r
# Histograma
hist(pvalor, freq=FALSE)
abline(h=1) # curve(dunif(x,0,1), add=TRUE)
```



\begin{center}\includegraphics[width=0.7\linewidth]{03-Generacion_numeros_aleatorios_files/figure-latex/unnamed-chunk-22-1} \end{center}

```r
# Test ji-cuadrado
# chisq.test.cont(pvalor, distribution="unif", nclasses=20, nestpar=0, min=0, max=1)
# Test de Kolmogorov-Smirnov
ks.test(pvalor, "punif",  min=0, max=1)
```

```
## 
## 	One-sample Kolmogorov-Smirnov test
## 
## data:  pvalor
## D = 0.023499, p-value = 0.6388
## alternative hypothesis: two-sided
```

Adicionalmente, si queremos estudiar la proporción de rechazos (el *tamaño del contraste*) para los posibles valores de $\alpha$, podemos emplear la distribución empírica del $p$-valor (proporción de veces que resultó menor que un determinado valor):


```r
# Distribución empírica
curve(ecdf(pvalor)(x), type = "s", lwd = 2, 
      xlab = 'Nivel de significación', ylab = 'Proporción de rechazos')
abline(a=0, b=1, lty=2)   # curve(punif(x, 0, 1), add = TRUE)
```



\begin{center}\includegraphics[width=0.7\linewidth]{03-Generacion_numeros_aleatorios_files/figure-latex/unnamed-chunk-23-1} \end{center}


### Baterías de contrastes

Contrastes específicos para generadores aleatorios:

-   Diehard tests (The Marsaglia Random Number CDROM):
    [http://www.stat.fsu.edu/pub/diehard](http://www.stat.fsu.edu/pub/diehard).

-   TestU01: 
    [http://www.iro.umontreal.ca/simardr/testu01/tu01.html](http://www.iro.umontreal.ca/simardr/testu01/tu01.html).

-   NIST test suite: 
    [http://csrc.nist.gov/groups/ST/toolkit/rng](http://csrc.nist.gov/groups/ST/toolkit/rng).

-   Dieharder (paquete `RDieHarder`):
    [http://www.phy.duke.edu/rgb/General/dieharder.php](http://www.phy.duke.edu/rgb/General/dieharder.php)

-   Entidad Certificadora (gratuita):
    [CAcert](http://www.cacert.at/random).

Documentación adicional:

-   Randomness Tests: A Literature Survey
    [http://www.ciphersbyritter.com/RES/RANDTEST.HTM](http://www.ciphersbyritter.com/RES/RANDTEST.HTM)-

-   Marsaglia, Tsang (2002). Some Difficult-to-pass Tests of Randomness: 
    [http://www.jstatsoft.org/v07/i03](http://www.jstatsoft.org/v07/i03)   
    [http://www.csis.hku.hk/cisc/download/idetect](http://www.csis.hku.hk/cisc/download/idetect)
    

Ejercicios de fin de práctica
-----------------------------


\BeginKnitrBlock{exercise}<div class="exercise"><span class="exercise" id="exr:unnamed-chunk-24"><strong>(\#exr:unnamed-chunk-24) </strong></span></div>\EndKnitrBlock{exercise}
Uno de los primeros generadores fue el denominado método de los
cuadrados medios propuesto por Von Neumann (1946). Con este
procedimiento se generan números pseudoaleatorios de 4 dígitos de la
siguiente forma:

i.  Se escoge un número de cuatro dígitos $x_0$ (semilla).

ii.   Se eleva al cuadrado ($x_0^2$) y se toman los cuatro dígitos
    centrales ($x_1$).

iii.   Se genera el número pseudo-aleatorio
    como$$u_1=\frac{x_1}{10^{4}}.$$

iv.  Volver al paso ii y repetir el proceso.

Para obtener los $k$ (número par) dígitos centrales de $x_{i}^2$
se puede utilizar que:
$$x_{i+1}=\left\lfloor \left(  x_{i}^2-\left\lfloor \dfrac{x_{i}^2}{10^{(2k-\frac{k}2)}}\right\rfloor 10^{(2k-\frac{k}2)}\right)
/10^{\frac{k}2}\right\rfloor$$ 

El algoritmo está implementado en el fichero *RANDVN.R*:

```r
# Generador Von Neumann de números pseudoaleatorios
# =================================================

# -------------------------------------------------
# initRANDVN(semilla,n)
#   Inicia el generador 
#   n número de digitos centrales, 4 por defecto (debe ser un nº par)
#   Por defecto semilla del reloj
#   OJO: No se hace ninguna verificación de los parámetros
initRANDVN <- function(semilla = as.numeric(Sys.time()), n = 4) {
  .semilla <<- as.double(semilla) %% 10^n  # Cálculos en doble precisión
  .n <<- n
  .aux <<- 10^(2*n-n/2)
  .aux2 <<- 10^(n/2)
  return(invisible(list(semilla=.semilla,n=.n)))
}

# -------------------------------------------------
# RANDVN()
#   Genera un valor pseudoaleatorio con el generador de Von Neumann
#   Actualiza la semilla (si no existe llama a initRANDVN)
RANDVN <- function() {
    if (!exists(".semilla", envir=globalenv())) initRANDVN()
    z <- .semilla^2
    .semilla <<- trunc((z-trunc(z/.aux)*.aux)/.aux2)
    return(.semilla/10^.n)
}

# -------------------------------------------------
# RANDVNN(n)
#   Genera un vector de valores pseudoaleatorios con el generador congruencial
#   (por defecto de dimensión 1000)
#   Actualiza la semilla (si no existe llama a initRANDVN)
RANDVNN <- function(n = 1000) {
    x <- numeric(n)
    for(i in 1:n) x[i] <- RANDVN()
    return(x)
    # return(replicate(n,RANDVN()))  # Alternativa más rápida
}
```

Estudiar las características del
generador de cuadrados medios a partir de una secuencia de 500
valores. Emplear únicamente métodos gráficos.

\BeginKnitrBlock{exercise}<div class="exercise"><span class="exercise" id="exr:unnamed-chunk-26"><strong>(\#exr:unnamed-chunk-26) </strong></span></div>\EndKnitrBlock{exercise}
Considerando el generador congruencial multiplicativo de parámetros
$a=7^{5}=16807$, $c=0$ y $m=2^{31}-1$. ¿Se observan los mismos problemas 
que con el algoritmo RANDU al considerar las tripletas $(x_{k},x_{k+1},x_{k+2})$?
