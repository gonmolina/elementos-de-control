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

# Ecuaciones de Cinética Puntual

+++

## Ecuaciones de cinética de un reactor nuclear

+++

Las ecuaciones de cinética puntual del núcleo de un reactor a un grupo de energía y a un grupo de precursores sin fuente de neutrones son:

$$
\begin{align}
\frac{dn(t)}{dt}&=\frac{\rho(t)-\beta}{\Lambda}n(t)&+\lambda c(t)\\
\frac{dc(t)}{dt}&=\frac{\beta}{\Lambda}n(t)&-\lambda c(t)
\end{align}
$$

con $n$ el flujo neutrónico normalizado (en realidad es población de neutrones, pero bajo ciertas condiciones se puede considerar proporcional al flujo neutrónico o potencia nuclear), $c$ la concentración de precursores, y $\rho$ la reactividad introducida en el mismo por las barras de control. Los parámetros del sistema son: $\Lambda = 1,76e^{-4} s$ (tiempo de reproducción de los neutrones rápidos), $\lambda = 0,076 1/s$ (constante de decaimiento de los precursores) y $\beta = 765$pcm (fracción de neutrones retardados que son emitidos por los precursores).

```{code-cell} ipython3
import numpy as np
import control as ctrl
import matplotlib.pyplot as plt
```

```{code-cell} ipython3
def cp_derivs(t, x, u, params={}):
    L = params.get('L', 1.76e-4)                 # tiempo de reprod. de neotrnues
    l = params.get('l', 0.0761)                 # constante de dec de prec.
    b = params.get('b', 765e-5)                  # fracción neutr. ret. emitidos por prec.
    
    n, c = x
    r = u[0]
    
    dn=(r-b)/L*n+l*c
    dc=b/L*n-l*c
    
    return [dn, dc]

def cp_salidas(t, x, u, params={}):
    return [x[0]]

```

+++ {"user_expressions": []}

Ahora, definamos el sistema no lineal

```{code-cell} ipython3
cp_nl = ctrl.NonlinearIOSystem(cp_derivs, cp_salidas, name='cin. puntual',
                                    inputs=('rho'), outputs=('n'), state=('n', 'c'))
```

+++ {"user_expressions": []}

Es común trabajar con el sistema sobre un punto de equilibrio normalizado con $n=1$. Buscaremos entonces el valor de reactividad para el cual la población de nuetrones se mantiene constante en 1. Nuevamente aclaramos que el cálculo del punto de equilibrio queda fuera del alcance del curso, pero no así su verificación.

```{code-cell} ipython3
:tags: [hide-cell]

xe, ue = ctrl.find_eqpt(cp_nl, [1,0], u0=[0], y0=[1], iy=[0])
xe, ue
```

+++ {"user_expressions": []}

El valor de $\rho$ de equilibrio es `ue` y los estados de equilibrio son `xe`. Para verificar debemos ver que las derivadas de los estados sean todas 0 y que la salida sea 1 para `xe` y `ue`.

```{code-cell} ipython3
print(cp_derivs(0, xe, ue), cp_salidas(0, xe, ue))
```

+++ {"user_expressions": []}

Finalmente obtenemos el sistema linealizado en torno al punto de equilibrio.

```{code-cell} ipython3
cp_l = ctrl.linearize(cp_nl, xe, ue)
cp_l
```

## Simulación numérica

Vamos a definir primero nuestra entrada, que será un escalón de reactividad de 10pcm en $t=1$.

```{code-cell} ipython3
t=np.arange(0,8,0.02)
u = []

for i in t:
    if i<=1:
        u.append(0.)
    else:
        u.append(10e-5)
u=np.array(u)
```

+++ {"user_expressions": []}

Graficamos la señal de reactividad que acabamos de crear:

```{code-cell} ipython3
fig, ax = plt.subplots(figsize=(10, 4))
ax.plot(t,u)
ax.set_title('Escalón unitario en $t=1$')
ax.set_xlabel('Tiempo [s]')
ax.set_ylabel('Valor de $u$');
ax.grid()
```

+++ {"user_expressions": []}

Realicemos ahora la simulación numérica del modelo lineal y no lineal.

```{code-cell} ipython3
sr1_l = ctrl.forced_response(cp_l, t, u, xe)
sr1_nl= ctrl.input_output_response(cp_nl, t, u, xe, solve_ivp_kwargs={'method':'BDF'})
```

+++ {"user_expressions": []}

Vamos a graficar sobre la misma figura ambos respuesta para poder comparar.
Para esto debemos agregar a la respuesta del sistema lineal el valor de equilibrio de la misma manera que lo hicimos en el ejemplo anterior.

```{code-cell} ipython3
:tags: [hide-input]

fig = plt.figure(figsize=(10,4))
ax = fig.add_axes([0.1, 0.1, 0.8, 0.8])
ax.plot(sr1_nl.t,sr1_nl.outputs, label='Modelo no lineal')
ax.plot(sr1_l.t, sr1_l.outputs, label='Modelo lineal')
ax.grid()
ax.legend()
ax.set_title('Respuesta al escalón de 10pcm')
ax.set_xlabel('Tiempo [s]')
ax.set_ylabel('Flujo neutrónico');
```

Vemos que el sistema linealizado es una muy buena aproximación del no lineal. Esto es por que la perturbación de 10pcm es suficientemente chica y el sistema no se aleja demasiado del punto del cual linealizamos.

+++

## Probemos aumentado un poco más la reactividad de entrada a 200 pcm

+++ {"user_expressions": []}

Redefinimos la entrada para que tome un valor escalón de 200pcm.

```{code-cell} ipython3
t=np.arange(0,8,0.02)
u = []

for i in t:
    if i<=1:
        u.append(0.)
    else:
        u.append(200e-5)
u=np.array(u)
```

+++ {"user_expressions": []}

Y realizamos ambas simulaciones.

```{code-cell} ipython3
sr2_l = ctrl.forced_response(cp_l, t, u, xe)
sr2_nl= ctrl.input_output_response(cp_nl, t, u, xe, solve_ivp_kwargs={'method':'BDF'})
```

+++ {"user_expressions": []}

Y los resultados los vemos en las siguientes gráficas:

```{code-cell} ipython3
:tags: [hide-input]

fig = plt.figure(figsize=(10,4))
ax = fig.add_axes([0.1, 0.1, 0.8, 0.8])
ax.plot(sr2_nl.t,sr2_nl.outputs, label='Modelo no lineal')
ax.plot(sr2_l.t, sr2_l.outputs, label='Modelo lineal')
ax.grid()
ax.legend()
ax.set_title('Respuesta al escalón de 200pcm')
ax.set_xlabel('Tiempo [s]')
ax.set_ylabel('Flujo neutrónico');
```

Vemos como ahora la aproximación es bastante peor a la que teníamos cuando lo perturbamos con 10 pcm.
