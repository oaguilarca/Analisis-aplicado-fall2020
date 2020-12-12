# Proyecto Análisis Aplicado
Este es un proyecto escrito en Julia. Para poder correrlo hay que seguir las siguientes instrucciones:
1. Correr en la terminal `julia`
  * Se recomienda empezar usando `julia --threads 2` o algún otro número en lugar de `2`. Este programa utiliza varios threads. Mientras más se utilicen, mayor será la velocidad. Esto está acotado por el tamaño de la matriz más grande.
2. Dentro de Julia, correr en la terminal: `include("path/to/Proyecto1.jl")`, utilizado el path al proyecto.
## Guía básica de Julia
Julia como lenguaje de programación es relativamente sencillo con sintaxis similar a lenguajes como Python, R y Matlab. Una guía de sus diferencias básicas donde están presentadas algunas de las características del lenguaje importantes para este proyecto se pueden ver [aqui](https://docs.julialang.org/en/v1/manual/noteworthy-differences/). Aquí están presentadas algunas de las 
### Tipos
El lenguaje Julia puede ser estricto con los tipos utilizados. Si la función pide un `Float64` es importante pasar algo que el compilador entiende como `Float64`, es decir, si se quiere usar un entero es importante agregar un `.0` después del número.
### Vectores
Los vectores son arreglos unidimensionales que se pueden utilizar como vectores después de importar el paquete `LinearAlgebra`. Al correr este programa se importa el paquete, pero la instrucción para importarlo es: `using LinearAlgebra`. Un vector o arreglo se define como:
```
x=[1.0, 2, 3] #Arreglo de Float64
x=[1, 2, 3] #Arreglo de Int64
x=Array{Float64}(undef, 3) #Arreglo vacío de longitud 3
x=zeros(3) #Arreglo de ceros de longitud 3
```
### Matrices
Funcionan con los mismos principios que el vector, siendo arreglos bidimensionales.
```
x=[1.0 2 3; 4 5 6; 7 8 9] #Arreglo de Float64
x=[1 2 3; 4 5 6; 7 8 9] #Arreglo de Int64
x=Array{Float64}(undef, 3, 3) #Arreglo vacío de tamaño 3x3
x=zeros(3, 3) #Arreglo de ceros de tamaño 3x3
x=Matrix{Float64}(I, 3, 3) #Matriz identidad de 3x3
```
### Funciones
Para definir una función anónima se puede hacer:
```
f(x)=x+1
```
Las funciones tienen dos tipos de parámetros: los obligatorios y los opcionales. Los parámetros obligatorios deben de pasar separados por comas en el mismo orden que en la definición de la función. Si se quiere pasar parámetros opcionales, se pone un `;` después de los parámetros obligatorios y se asignan con nombre al parámetro:
```
f(x;y=1)=x+y
f(5) #Regresa 6
f(5;y=1) #Regresa 6
f(5; y=7) #Regresa 12
```
## Documentación

### Funciones necesarias de álgebra lineal

#### is_pos_semi_def
Esta función recibe un arreglo bidimensional de `Float64` y regresa un Booleano que es `true` si la matriz es semidefinida positiva y `false` si no lo es.
##### Ejemplo:
```
julia> is_pos_semi_def([1.0 0 0; 0 1 0; 0 0 1])
true
```
#### is_pos_def
Esta función recibe un arreglo bidimensional de `Float64` y regresa un Booleano que es `true` si la matriz es definida positiva y `false` si no lo es.
##### Ejemplo:
```
julia> is_pos_def([1.0 0 0; 0 1 0; 0 0 1])
true
```
#### grad
Esta función recibe una función, un arreglo numérico de una dimensión y, opcionalmente, una `h` con valor default `1e-20`. Regresa un arreglo de `Float64` de una dimensión. Para calcularla, se usa un paso complejo, lo que limita las funciones utilizables a todas las funciones que utilizan valores absolutos pero, como ventaja, aumentan significativamente la precisión. El método está basado en [este paper](https://www.researchgate.net/publication/222112601_The_Complex-Step_Derivative_Approximation).
##### Ejemplo:
```
julia> f(x)=x[1]*x[2]
f (generic function with 1 method)

julia> grad(f, [1,3]; h=1e-10)
2-element Array{Float64,1}:
 3.0
 1.0
```

#### derivative
Esta función recibe una función, un arreglo numérico de una dimensión y, opcionalmente, una `h` con valor default `1e-20`. Regresa la parte imaginaria de la funcion con paso complejo.

##### Ejemplo:
```
julia> f(x)=x[1]*x[2]
f (generic function with 1 method)

julia> derivative(f, [1,3]; h=1e-20)
2-element Array{Float64,1}:
 0.0
 0.0
```

#### hess
Esta función recibe una función, un arreglo numérico de una dimensión y, opcionalmente, una `h` con valor default `1e-7`. Regresa un arreglo de `Float64` de 2 dimensiones. Se calcula a través de un paso complejo y diferencia centrada, lo que limita las funciones utilizables a todas las funciones que utilizan valores absolutos pero, como ventaja, aumenta significativamente la precisión. El metodo esta basado en [este programa en Matlab](https://www.mathworks.com/matlabcentral/fileexchange/18177-complex-step-hessian).
##### Ejemplo:
```
julia> f(x)=(x[1]^2)*(x[2]^2)
f (generic function with 1 method)

julia> hess(f, [1,3]; h=1e-5)
2×2 Array{Float64,2}:
 18.0  12.0
 12.0   2.0
```
#### check_optimality
Esta función recibe un gradiente, una hessiana y, opcionalmente, recibe una tolerancia `tol` con valor default `1e-20`. Regresa un booleano que es `true` si es óptimo y `false` si no lo es.
##### Ejemplo:
```
julia> f(x)=x[1]^2+x[2]^2
f (generic function with 1 method)

julia> x=[0.0, 0]
2-element Array{Float64,1}:
 0.0
 0.0

julia> check_optimality(grad(f,x),hess(f,x);tol=0.0)
true
```
#### backtracking_line_search
Es el algoritmo 3.1 del Nocedal.  Recibe una función, un arreglo numérico, otro arreglo numérico y, opcionalmente, recibe tres variables `Float64`: una `a` con valor default `1`, una `c` con valor default `1e-4` y una `p` (rho) con valor default `0.5`. Regresa un `Float64`.

#### add_identity
Busca iterativamente un número `t` tal que la matriz A+I*t sea definida positiva y regresa la matriz A+I*t. Recibe una matriz de `Float64` y, opcionalmente, un `Float64` `b` con default `1e-4`. Regresa una matriz definida positiva.

#### line_search_newton_modification
Es el algoritmo 3.2 del Nocedal. Recibe una función, un punto inicial y, opcionalmente, recibe una tolerancia `tol` con valor default `1e-4` y un máximo de iteraciones `maxit` con valor default `10000`. Regresa las coordenadas de la mejor aproximación que se logró e imprime el número de iteraciones.
##### Ejemplo
```
julia> f(x)=x[1]^2+x[2]^2
f (generic function with 1 method)

julia> x=[10.0, 100]
2-element Array{Float64,1}:
  10.0
 100.0

julia> line_search_newton_modification(f,x;tol=0.0,maxit=1000000)
Numero de iteraciones:    4
2-element Array{Float64,1}:
 0.0
 0.0
```
#### rosenbrock
Es la función de Rosenbrock. Recibe un arreglo numérico y, opcionalmente, números: `a` con valor default `1` y `b`. Regresa el resultado de la función de Rosenbrock.
##### Ejemplo
```
julia> rosenbrock([1,2]; a=2, b=200)
201
```
### Declaracion de tipos de descenso

#### BFGS
Se realiza una estructura con el método de Broyden–Fletcher–Goldfarb–Shanno. Se recibe `Q` y `c1`, `c2` y `p` (rho) valores para las condiciones de Wolfe.

##### init!
Función que inicializa a `D` para el método a partir de Q. Recibe `D`, `x` y el gradiente de f `gf`. Regresa `D`

##### step!
Función que calcula el paso para el método de BFGS. Recibe `D`, `f`, el gradiente de f, `x`, el gradiente de `f` evaluado en `x` y la hessiana ´hx´. Regresa el valor de `x` en el siguiente paso `xk` y su valor `gk`.

##### Ejemplo

#### Metodo de Newton
Se realiza una estructura con el método de Newton.

##### init!
Función que inicializa a D para el método a partir de Q. Recibe `D`, `x` y el gradiente de f `gf`. Regresa `D`

##### step!
Función que calcula el paso para el método de BFGS. Recibe `D`, `f`, el gradiente de f, `x`, el gradiente de `x` y la matriz Hessiana `Hx`. Regresa el valor de `x` en el siguiente paso `xk` y su valor `gk`.

##### Ejemplo 

#### Metodo de Newton con modificacion a la Hessiana
Se realiza una estructura con el método de Newton con modificación a la Hessiana donde recibe el valor del parámetro `a`.

##### init!
Función que inicializa a `D` para el método a partir de `Q`. Recibe `D`, `x` y el gradiente de f `gf`. Regresa `D.`

##### step!
Función que calcula el paso para el método de BFGS. Recibe `D`, `f`, el gradiente de f, `x`, el gradiente de `f` evaluado en `x` y la matriz Hessiana `Hx`. Regresa el valor de `x` en el siguiente paso `xk` y su valor `gk`.

##### Ejemplo 

#### Line Search
Se realiza una estructura con el método de Newton.

##### init!
Función que inicializa a `D` para el método a partir de `Q`. Recibe `D`, `x` y el gradiente de `f`: `gf`. Regresa `D`

##### step!
Función que calcula el paso para el método de BFGS. Recibe `D`, `f`, el gradiente de `f`, `x`, el gradiente de `f` evaluado en `x` y la matriz Hessiana `Hx`. Regresa el valor de `x` en el siguiente paso `xk` y su valor `gk`.

##### interpolate
Función que interpola. Recibe los puntos `a1`, `a2` y la función `f`. Regresa un punto entre `a1` y `a2`.

##### zoom
Función que calcula la `ak` óptima para el método. Recibe los puntos `al`, `ah`, la funcion `f`, y valores para `c1` y `c2`. Regresa el valor de `a` = `ak` que se usará.

##### minimize
Función que minimiza la alpha. Recibe la funcion `f`, los valores `c1`, `c2` y `p` inicializados en `1e-3`, `0.9` y `2` respectivamente. Regresa la alpha minimizada.

##### line_step_size
Funcion que calcula el tamaño del paso para el método de line search. Recibe la función `f`, `x` & `d`. Regresa `x` más el tamaño del paso.

##### Ejemplo 

#### Funcion general de descenso
Funcion que calcula en general el descenso y genera banderas para saber si encontró o no la solución óptima. Regresa el valor de `x` & el numero de pasos.

##### Ejemplo 

### Funciones para optimizar

#### revisar
Esta función sirve para demostrar el funcionamiento del código usando la función de Rosenbrock. Recibe un punto inicial, un valor de `a`, un valor de `b`, el método a utilizar y, opcionalmente, recibe una tolerancia `tol` con valor default `1e-10` y un máximo de iteraciones `maxit` con valor default `10000`. Imprime un reporte con el resultado exacto, el número de iteraciones, el resultado obtenido y los errores absolutos y relativos en x & y.

##### Ejemplo


### Función de las cámaras

#### costo_camara
Esta función es la función de costo que calcula la distancia del punto de la localización de las cámaras al punto del delito. Recibe las coordenadas latitud `lat`y longitud `lon` de donde se pondrían las cámaras, `x0`, los datos descargados de las coordenadas de los delitos cometidos `df` y un valor para `j`. 

#### costo_est
Esta función es la función de costo estimado para un número fijo de 20 cámaras. Recibe `x0` y los datos descargados de las coordenadas de los delitos cometidos. Imprime un reporte con los  

#### costo
Esta función es la función de costo estimado. Recibe `x0` y los datos descargados de las coordenadas de los delitos cometidos. Imprime un reporte con los  

#### rand_geo_array
Esta función genera un intervalo aleatorio de dimensión 1. Recibe los valores `mina` y `maxa` los valores mínimos y máximos de latitud, `mino` y `maxo` los valores máximos de longitud y un valor para `s`. Regresa un intervalo aleatorio dentro de la región donde se encuentran los datos.


#### mejores_camaras
Esta función es la función de 

##### Ejemplo

#### proyecto_final
Se define una estructura con las siguientes variables necesarias:
`f`: la funcion a minimizar
`gf`: la derivada de f
`x0`: el valoralor inicial
`tol`: la tolerancia
`maxit`: el maximo numero de iteraciones
`met`: el metodo que se va a usar para minimizar
`res`: el valor minimizado
`pasos`: el número de puntos por los que paso el algoritmo
Y el contructor sería la funcion que calcula según el método indicado y usa las variables anteriores y las condiciones de Wolfe `c1`, `c2` y `p` inicializadas en `1e-2`, `0.9` y `2` respectivamente. Regresa la respuesta según el método utilizado usando la función de descenso correspondiente.


##### Ejemplo

### Integrantes
* Dan Jinich
* María José Sedano
* Oscar Aguilar
* Francisco Aramburu


