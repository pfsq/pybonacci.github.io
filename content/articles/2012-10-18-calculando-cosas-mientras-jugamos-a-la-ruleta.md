---
title: Calculando cosas mientras jugamos a la ruleta
date: 2012-10-18T20:50:20+00:00
author: Kiko Correoso
slug: calculando-cosas-mientras-jugamos-a-la-ruleta
tags: monte carlo, montecarlo, números aleatorios, pi, python, shapely

Hoy vamos a tratar una serie de simulaciones que deben su nombre al lujoso casino de Monte Carlo y a la capacidad de la ruleta de comportarse como un generador de números aleatorios. Vamos a ver varios ejemplos que ilustran el cálculo aproximado de áreas y del número $pi$ mediante una simulación Monte Carlo.

En primer lugar vamos a calcular un área considerando el siguiente problema: Tenemos un recuadro que mide 10 km de lado y dentro de este recuadro tenemos un polígono de lados irregulares que, por la razón que sea, no sabemos cuanto ocupa, es decir, cuál es su área.

Para resolver este problema de forma aproximada vamos a usar [shapely](http://pybonacci.org/2012/09/20/buscando-esa-playa-en-la-isla-a-mediodia-usando-shapely/), además de los omnipresentes numpy y matplotlib. Empezamos importando todo lo que necesitamos:

    :::python
    from shapely.geometry import Polygon, Point
    import numpy as np
    import matplotlib.pyplot as plt

Generamos los dos polígonos, el cuadrado que sí sabemos lo que mide (poligono_externo) y el polígono con área desconocida (polígono interno):

    :::python
    poligono_externo = Polygon(((0,0),(0,10),(10,10),(10,0),(0,0)))
    poligono_interno = Polygon(((3,2),(5,7),(9,6),(6,5),(7,1),(3,2)))

Si dibujamos ambos polígonos tendremos algo parecido a lo siguiente:

    :::python
    plt.ion()
    plt.xlim(-1,11)
    plt.ylim(-1,11)
    plt.plot(poligono_externo.boundary.xy[0], poligono_externo.boundary.xy[1])
    plt.plot(poligono_interno.boundary.xy[0], poligono_interno.boundary.xy[1])

![poligonos](http://pybonacci.org/images/2012/10/poligonos.png)

Ahora creamos una función que será la que se encarga de hacer el cálculo:

    :::python
    def simulacion_mc(num_pruebas):
        xvect = np.random.rand(num_pruebas)*10.
        yvect = np.random.rand(num_pruebas)*10.
        cont = 0
        for x, y in zip(xvect, yvect):
            if poligono_interno.contains(Point(x, y)):
                cont += 1
        return poligono_externo.area * cont / num_pruebas

¿Qué hemos hecho en la anterior pieza de código? Hemos generado unos valores aleatorios para la variable **_x_** y para la variable **_y_** de la posición. Comprobamos si cada uno de los puntos generados se encuentran dentro del polígono de área desconocida y los contamos. Finalmente devolvemos el cálculo del área aproximada. El área se calcula aprovechando que la probabilidad de que un punto esté dentro del área del polígono interior será proporcional al área de ese polígono:

$frac{Area\_{text{poligono exterior}}}{Area\_{text{poligono interior}}}proptofrac{text{puntos contenidos en el poligono exterior}}{text{puntos contenidos en el poligono interior}} = frac{text{num pruebas}}{text{cont}} $

Por tanto, si despejamos el Área del polígono interior de la anterior fórmula tenemos que el área aproximada la podemos obtener de la siguiente forma:

$Area\_{text{poligono interior}} = Area\_{text{poligono exterior}}frac{text{cont}}{text{num pruebas}} $

Donde _num_pruebas_ es el número de puntos (posiciones) que usamos en la simulación (todos contenidos dentro del polígono exterior) y _cont_ es el número de puntos que han 'caído' en el polígono interior.

Una simulación de 1000 puntos lo que hace es lo que podemos ver en la siguiente gráfica:

    :::python
    num_pruebas = 1000
    xvect = np.random.rand(num_pruebas)*10.
    yvect = np.random.rand(num_pruebas)*10.
    plt.xlim(-1,11)
    plt.ylim(-1,11)
    plt.plot(poligono_externo.boundary.xy[0], poligono_externo.boundary.xy[1])
    plt.plot(poligono_interno.boundary.xy[0], poligono_interno.boundary.xy[1])
    plt.plot(xvect, yvect, 'r.')

![poligono_puntos](http://pybonacci.org/images/2012/10/poligono_puntos.png)

El número de puntos dentro del polígono interno sería _cont_ en la función que hemos definido anteriormente mientras que el número de puntos dentro del polígono externo serían todos los puntos usados, es decir, _num_pruebas._

Vamos a hacer varias simulaciones con diferentes números de puntos y representar el valor del área obtenida en función del número de puntos usado:

    :::python
    ## Aviso, este cálculo puede tardar bastante...
    resultados = [simulacion_mc(i) for i in np.arange(0, 10000, 10) + 10]
    plt.plot(np.arange(0,10000,10) + 10, resultados)
    plt.xlabel(u'número de puntos aleatorios usados')
    plt.ylabel(u'Área del polígono interior')
    plt.show()

El resultado sería el siguiente:

![Montecarlo](http://pybonacci.org/images/2012/10/montecarlo.png)

Vemos que los valores obtenidos están en torno a 15 y pico y que hay menos dispersión a medida que aumentamos el número de puntos usados en la simulación. Si ahora usamos la propiedad **area** de la clase Polygon podemos ver el área que tiene el polígono interno:

    :::python
    print poligono_interno.area

El resultado, si hemos hecho todo bien, sería 15.5 km². Parecido a los resultados de nuestras simulaciones!!!

Todo el código junto (sin la generación de gráficas) quedaría de la siguiente forma:

    :::python
    from shapely.geometry import Polygon, Point
    import numpy as np
    import matplotlib.pyplot as plt
    ## Definimos dos polígonos, uno del que conoceremos su área
    ## y un segundo de área desconocida (a priori)
    poligono_externo = Polygon(((0,0),(0,10),(10,10),(10,0),(0,0)))
    poligono_interno = Polygon(((3,2),(5,7),(9,6),(6,5),(7,1),(3,2)))
    ## Función que calcula nuestra área aproximada
    def simulacion_mc(num_pruebas):
        xvect = np.random.rand(num_pruebas)*10.
        yvect = np.random.rand(num_pruebas)*10.
        cont = 0
        for x, y in zip(xvect, yvect):
            if poligono_interno.contains(Point(x, y)):
                cont += 1
        return poligono_externo.area * cont / num_pruebas
    ## Lanzamos varias simulaciones
    ## Aviso, este cálculo puede tardar bastante...
    resultados = [simulacion_mc(i) for i in np.arange(0, 10000, 10) + 10]

Ahora dejo un código para calcular $pi$ usando 10000 puntos aleatorios. Si alguien no lo entiende que pregunte en los comentarios pero se trata de la misma metodología usada anteriormente (pistas: [1](http://upload.wikimedia.org/wikipedia/commons/8/84/Pi_30K.gif), [2](http://niallohiggins.com/2007/07/05/monte-carlo-simulation-in-python-1/)):

    :::python
    import numpy as np
    import matplotlib.pyplot as plt
    ## Función que calcula nuestra área aproximada
    def simulacion_mc(num_pruebas):
        xvect = np.random.rand(num_pruebas)
        yvect = np.random.rand(num_pruebas)
        cont = 0
        for x, y in zip(xvect, yvect):
            if np.sqrt(x * x + y * y) &lt;= 1:
                cont += 1
        return 4. * cont / num_pruebas
    print simulacion_mc(10000)

Espero que a alguien le resulte útil.

P.D.: Escribiendo las fórmulas en latex de más arriba no encontraba como escribir el símbolo $propto$ y lo he encontrado gracias a [ésta web](http://detexify.kirelabs.org/classify.html) (algún día hablaremos sobre metodologías para hacer lo que se hace en esa web :-))