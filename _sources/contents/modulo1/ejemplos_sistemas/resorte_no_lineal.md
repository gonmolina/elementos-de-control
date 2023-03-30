---
jupytext:
  formats: ipynb,md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.14.5
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

+++ {"user_expressions": []}

# Sistema masa resorte amortiguador

+++ {"user_expressions": []}

## Ecuaciones de movimiento

+++ {"user_expressions": []}

Supongamos que el resorte no cumple la ley de Hook, sino que produce una fuerza que es:

$$f_k = Kx^3$$

+++ {"user_expressions": []}

Entonces las ecuaciones de movimiento son

$$ ma(t) = F+mg - f_b - f_k \implies m \ddot x(t) = F + mg - b \dot x(t) - K x^3(t)$$

+++ {"user_expressions": []}

Podemos escribir las ecuaciones de estado, suponiendo los estados son $\dot x = x_1$ y $x = x_2$. Entonces resultan:

$$
  \left\{
    \begin{array}{ll}
      \dot x_1(t) =  & -\dfrac{b}{m} x_1(t) - \dfrac{K} {m} x_2^3(t) +\dfrac{F(t)}{m} + g\\
      \dot x_2(t) =  & x_1(t)
    \end{array}
  \right.
$$

+++ {"user_expressions": []}

## Implementación del modelo en Python

Este ejemplo implementaremos utilizando un función que no permite modelos sistemas dinámicos escritos como ecuaciones de estado. Para esto importaremos algunos "paquetes" para Python.

```{code-cell} ipython3
:tags: [hide-input]

import numpy as np
import control as ctrl
import matplotlib.pyplot as plt
# descomentar la siguiente linea para ver la imagen e una ventana emergente
# %matplotlib qt5
```

```{code-cell} ipython3
def resorte_derivs(t, x, u, params={}):
    # Ajuste de los parámetros del modelo b=1, m=3, K=2, g=9.8
    m = params.get('m', 3.)                 # mass
    b = params.get('b', 1.)                 # gravitational constant, m/s^2
    K = params.get('K', 2)                  # coefficient of rolling friction
    g = params.get('g', 9.8)                # drag coefficient
    
    dx =np.zeros((2,), dtype=np.float64)
    dx[0] = -b/m*x[0]-K/m*x[1]**3+u[0]/m+g
    dx[1] = x[0]
    
    return dx   

def resorte_salidas(t, x, u, params={}):
    return [x[1]]
```

+++ {"user_expressions": []}

A partir de las funciones anteriores (ecuaciones de estado), podemos definir el modelo del sistema en Python.

```{code-cell} ipython3
resorte_nl = ctrl.NonlinearIOSystem(resorte_derivs, resorte_salidas, name='resorte',
                                    inputs=('F'), outputs=('pos'), state=('vel', 'pos'))
```

+++ {"user_expressions": []}

Para poder simular el sistema nos queda definir:
- en que tiempos queremos tener la respuesta del sistema 
- cuanto vale la entrada para los tiempos definidos anteriormente

Para esto vamos a suponer que queremos las salidas cada 0.01 segundos, entre 0 y 10 segundos.

La salida vamos a tomar un escalón unitario en $t=1$, es decir que $F=0$ para $t\leq 1$ y $F=1$ para $t>1$.

Estas dos condiciones la definimos en la siguiente celda.

```{code-cell} ipython3
t=np.arange(0,80,0.1)
u = []

for i in t:
    if i<=40:
        u.append(0.)
    else:
        u.append(1.)
```

+++ {"user_expressions": []}

Analicemos la señal que acabamos de crear. Para esto vamos a ver los valores que toma $u$ en función del tiempo $t$.

```{code-cell} ipython3
fig, ax = plt.subplots(figsize=(10, 4))
ax.plot(t,u)
ax.set_title('Escalón unitario en $t=1$')
ax.set_xlabel('Tiempo [s]')
ax.set_ylabel('Valor de $u$');
ax.grid()
```

+++ {"user_expressions": []}

Podemos ver que la señal que creamos es efectivamente lo que queríamos. Una señal que vale 0 para tiempos menores que 1 y 1 para tiempos mayores a 1.

+++ {"user_expressions": []}

Ahora tenemos todo lo necesario para simular el sistema dinámico. Para esto usaremos otra función del paquete de control.

```{code-cell} ipython3
resp = ctrl.input_output_response(resorte_nl, t, U=u)
```

```{code-cell} ipython3
fig, ax = plt.subplots(figsize=(10, 4))
ax.plot(resp.t,resp.outputs)
ax.set_title('Respuesta del sistema a un escalón unitario')
ax.set_xlabel('Tiempo [s]')
ax.set_ylabel('Valor de la pos');
ax.grid()
```

+++ {"user_expressions": []}

Analicemos ahora que sucede en esta figura.

La posición de la masa es 0. O sea consideramos que el resorte esta en una posición de re reposo. Sin embargo,la masa tiene aplicada la fuerza de gravedad. Arrancar la simulación sería como soltar la masa desde la posición de reposo del resorte.

Cuando se suelte la masa cae y empieza a oscilar. Luego de un tiempo se estabiliza en torno a una posición que llamaremos de equilibrio. En esta posición, si no cambia la fuerza aplicada a la masa esta se quedaría quieta.

En t=40, cuando entra la fuerza, comienza nuevamente la evolución dinámica del sistema.

Muchas veces se estudia al sistema desde el punto de equilibrio, es decir cuando está quieto. Para esto debemos obtener en que punto del espacio de estados el sistema se queda quieto. Mirando la figura anterior podemos notar que si no entrara la fuerza en $t=40$ el sistema se estabilizaría en torno a una posición cercana a 2.5. En esa posición, para $u=0$, *el sistema se queda quieto*. Matemáticamente, que el sistema se quede quieto significa que todas las derivadas de las variables de estado son igual 0.

Entonces, podríamos calcular de forma exacta este punto evaluando para `x` se hace 0 la función `resorte_derivs`. El calculo de este punto queda fuera del alcance del curso.

```{code-cell} ipython3
:tags: [hide-cell]

import scipy.optimize as opt
x0=opt.fsolve(lambda x: resorte_derivs(0,x,[0]), [0,2], xtol=1e-20)
print(x0, resorte_derivs(0,x0,[0]))
```

+++ {"user_expressions": []}

Ahora el punto de equilibrio fue calculado y se encuentra guardado en la variable `x0`, repitamos la simulación anterior, pero comenzando la simulación en este punto de equilibrio.

```{code-cell} ipython3
resp_eq = ctrl.input_output_response(resorte_nl, t, U=u, X0=x0)
```

```{code-cell} ipython3
fig, ax = plt.subplots(figsize=(10, 4))
ax.plot(resp_eq.t,resp_eq.outputs)
ax.set_title('Respuesta del sistema a un escalón unitario')
ax.set_xlabel('Tiempo [s]')
ax.set_ylabel('Valor de la pos');
ax.grid()
```

+++ {"user_expressions": []}

## Verificación de la no linealidad

Para verificar la no linealidad debemos mostrar que no se verifica superposición. Una forma simple de verificar superposición es simular el sistema frente a dos entradas, de forma que la segunda entrada sea el doble de la primera. Si el sistema es lineal, **la salida de la segunda simulación deberá presentar una desviación respecto al punto de equilibrio del doble de primera**.

```{code-cell} ipython3
resp1 = ctrl.input_output_response(resorte_nl, t, X0=x0, U=u)
resp2 = ctrl.input_output_response(resorte_nl, t, X0=x0, U=2*np.array(u))
```

```{code-cell} ipython3
:tags: [hide-input]


fig, ax = plt.subplots(figsize=(10, 4))
ax.plot(resp1.t,resp1.outputs, label="salida 1")
ax.plot(resp2.t,resp2.outputs, label="salida 2")
ax.legend()
ax.set_title('Respuesta del sistema a un escalón unitario')
ax.set_xlabel('Tiempo [s]')
ax.set_ylabel('Valor de la pos');
ax.grid()
```

+++ {"user_expressions": []}

Vemos que la primer salida el valor final es aproximadamente 2.7 y para la segunda es 2.91. El valor inicial de ambas es 2.45. Por lo tanto tenemos que:

$$2.7-2.45=0.25$$

$$2.91-2.45=0.46$$

Vemos que la desviación de la  salida no es exactamente el doble. Pero esta bastante cerca de serlo.

+++ {"user_expressions": []}

Hagamos una tercera verificación, pero ahora hagamos la entrada 5 veces más grande.

```{code-cell} ipython3
:user_expressions: []

resp3 = ctrl.input_output_response(resorte_nl, t, X0=x0, U=5*np.array(u))
```

```{code-cell} ipython3
:tags: [hide-input]
:user_expressions: []

fig, ax = plt.subplots(figsize=(10, 4))
ax.plot(resp1.t,resp1.outputs, label="salida 1")
ax.plot(resp3.t,resp3.outputs, label="salida 3")
ax.legend()
ax.set_title('Respuesta del sistema a un escalón unitario')
ax.set_xlabel('Tiempo [s]')
ax.set_ylabel('Valor de la pos');
ax.grid()
```

+++ {"user_expressions": []}

Para esta tercer simulación el valor final es:

$$3.41-2.46 = 0.95$$

Si el sistema fuese lineal la salida debería esperada es 5 veces la diferencia de la primer simulación:

$$5\times 0.25 = 1.25$$

Ahora podemos notar de forma más clara la presencia de la no linealidad.

+++ {"user_expressions": []}

```{important}
En general la presencia de la no linealidad se hace más notoria cuando las desviaciones desde el punto de equilibrio son más grandes.
```

+++ {"user_expressions": []}

## Sistema linealizado

Generalmente los sistemas no lineales son difíciles de estudiar, analizar y sacar conclusiones. Por lo que en el area de control de sistemas dinámicos, **en general se trabaja con sistemas linealizados en torno al puno de equilibrio**. La linealización de un sistema no lineal escapa al alcance de este curso, pero se muestra una forma de hacerla en la celda oculta debajo.

```{code-cell} ipython3
resorte_l = ctrl.linearize(resorte_nl, x0, [0])
```

+++ {"user_expressions": []}

Los sistemas lineales son más fáciles de simular y el paquete de control que utilizaremos posee herramientas para hacer simulaciones frente a la señales más comunes como son el escalón y el impulso.

```{code-cell} ipython3
sr = ctrl.step_response(resorte_l)
ir = ctrl.impulse_response(resorte_l)
```

```{code-cell} ipython3
:tags: [hide-input]

fig, ax = plt.subplots(figsize=(10, 4))
ax.plot(sr.t, sr.outputs, label="Respuesta al escalón")
ax.plot(ir.t, ir.outputs, label="Respuesta al impulso")
ax.legend()
ax.set_title('Respuestas del sistema lineal')
ax.set_xlabel('Tiempo [s]')
ax.set_ylabel('Desv de la pos');
ax.grid()
```

+++ {"user_expressions": []}

Para simular el sistema frente a entradas arbitrarias, también tenemos una función util para sistemas lineales:

```{code-cell} ipython3
fr = ctrl.forced_response(resorte_l, t, u)
```

```{code-cell} ipython3
:tags: [hide-input]

fig, ax = plt.subplots(figsize=(10, 4))
ax.plot(fr.t,fr.outputs)
ax.set_title('Respuesta del sistema a un escalón unitario')
ax.set_xlabel('Tiempo [s]')
ax.set_ylabel('Valor de la desv. de la pos');
ax.grid()
```

+++ {"user_expressions": []}

Podemos ver que la evolución luce similar a la de la simulación no lineal. Sin embargo, cuando miramos el eje $y$ vemos que tiene valores diferentes. Esto es porque la evolución es la desviación respecto del punto de equilibrio.

Para que ambas simulaciones sean comparables debemos sumar la valor de la salida de la simulación lineal, el valor de la salida en el punto de equilibrio.

Haciendo esto y superponiendo con la simulación no lineal tenemos:

```{code-cell} ipython3
fig, ax = plt.subplots(figsize=(10, 4))
ax.plot(fr.t,fr.outputs+x0[1])
ax.plot(resp_eq.t,resp_eq.outputs)
ax.set_title('Respuesta del sistema a un escalón unitario')
ax.set_xlabel('Tiempo [s]')
ax.set_ylabel('Valor de la desv. de la pos')
ax.grid();
```

+++ {"user_expressions": []}

Vemos que las evoluciones son parecidas, aunque no iguales. A medida que se aumente el valor del escalón, el sistema se alejará más del punto de equilibrio y las diferencias serán cada vez más notorias.
