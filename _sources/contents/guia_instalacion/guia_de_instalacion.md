# Guía de instalación de las herramientas computacionales

## Verificación del tipo de sistema operativo (32 o 64 bits)

### En Windows 10 y Windows 8.1

Selecciona el botón Inicio y después, ``Configuración > Sistema > Acerca de``. A la derecha, en Especificaciones del dispositivo, consulta Tipo de sistema.

### En Windows 7

Selecciona el botón Inicio, haz clic con el botón derecho en  
``Equipo`` y selecciona `Propiedades`. En `Sistema`, consulta el `tipo de sistema`.

## Descargar Miniconda de acuerdo al tipo de sistema operativo

Seguir el enlace de [Miniconda](https://docs.conda.io/en/latest/miniconda.html). Elegir la versión Miniconda con con el Python más reciente `Python 3.X` (donde X es el número entero más grande) y que sea acorde al tipo de sistema operativo instalado es su máquina según lo averiguado en el punto anterior.

- Para sistemas operativos Windows de 64 bits hacer [click aquí](https://repo.anaconda.com/miniconda/Miniconda3-py310_23.1.0-1-Windows-x86_64.exe).

- Para sistemas operativos Windows de 32 bits hacer [click aquí](https://repo.anaconda.com/miniconda/Miniconda3-latest-Windows-x86.exe).

## Instalar Miniconda

Seguir las recomendaciones por defecto dadas por el instalador. Una vez terminada la instalación tendremos Python instalado en nuestro sistema. Además tendremos en el menú de inicio un ítem
que se llama **Anaconda Powershell Prompt**, el cual abriremos para continuar con la instalación de las herramientas del curso.



## Instalación de las herramientas especificas del curso

Para continuar con la instalación necesitaremos descargar el archivo [eecc.yml](https://drive.google.com/file/d/1x9sAdSsXrPut3RiX7sintGzfyloEq3De/view?usp=share_link) haciendo click sobre el enlace anterior. Una vez descargado, dirigirse con la consola de Anaconda Powershell Prompt abierta a la carpeta de descargas donde se encuentra el archivo `eecc.yml` descargado anteriormente. En general, para dirigirse a esta carpeta debemos tipear en la consola de Anaconda Powershell:

```bash
cd ~\Downloads\
```

Para verificar que el archivo se encuentre ahí, podemos tipear:

```bash
dir eecc.yml
```

Si nos encontramos en la carpeta donde está el archivo este deberá aparecer luego del comando anterior.

Finalmente para instalar todas las herramientas necesarias en el curso debemos ejecutar en este comando:

```bash
conda env create -f eecc.yml
```

````{warning} 
En caso de que el comando anterior falle, deberá abrir una nueva consola de PowerShell como Administrador y ejecutar

```powershell
set-executionpolicy unrestricted
```
Luego debe contestar Si a Todo. Ahora podremos cerrar la consola de PowerShell como Administrador y continuar con la instalación.
````

Una vez terminado este paso tendremos instaladas todas las herramientas necesarias para la materia. Es necesario tener paciencia ya que este paso puede tardar un tiempo largo dependiendo de la velocidad de conexión. 

### Crear acceso directo a Jupyter Lab

Una vez instaladas todas las herramientas podremos utilizar `jupyter-lab` abriendo la consola de Anaconda Powershell y ejecutando el siguiente comando:

```powershell	
conda activate eecc
jupyter lab
```

escritorio. 

## NUEVO! Aplicación JupyterLab de escritorio

Recientemente se lanzó una nueva aplicación JupyterLab de escritorio. Esta aplicación crea un lanzador desde el menu de inicio en windows y se pueden abrir los Notebooks con esta aplicación directamente desde el explorador de archivos. Para instalarlos de debe descargar el siguiente [instalador](https://github.com/jupyterlab/jupyterlab-desktop/releases/latest/download/JupyterLab-Setup-Windows.exe) y ejecutarlo.

Para poder visualizar correctamente los cuadernos del curso será necesario instalar dos paquetes de Python extra. Para esto, po única vez cuando se abra la aplicación JupyterLab se de hacer lo siguiente:

1. Abrir el lanzador de JupyterLab. Para esto seleccionar desde la barra de menú de la aplicación `File > New Launcher`. Deberías veralgo similar a esta figura (con menos opciones):

```{figure} JupyterLabLauncher.png
:width: 400px
:align: center
:alt: JupyterLabLauncher

Lanzador de JupyterLab
```
2. En la ventana que aparece, seleccionar la opción Python3 (ipykernel) debajo de `Notebook`.
3. Cerrar la consola o el cuaderno y comenzar a utilizar la aplicación.

Una vez abierto el cuaderno con que el se quiere trabajar, se debe seleccionar el kernel `Python 3 (eecc)`, ya que este kernel es el que tiene todas las herramientas de la materia. Esto se hace haciendo click en el recuadro rojo que se muestra en la figura:

```{figure} KernelSelect.png
:width: 400px
:align: center
:alt: KernelSelect

Selector de Kernel en JupyterLab
```











