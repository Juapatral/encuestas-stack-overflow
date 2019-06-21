# **<center> Análisis de las encuestas de Stack Overflow 2011-2017 </center>**
---

### **<center> Posgrado de Analítica 2019-1 </center>**
<center> Sistema de Bases de Datos Masivos </center><br>

###  **<center> Integrantes: </center>**
​
<center> Jhon Anderson Londoño Herrera<br>
Juan Pablo Trujillo Alviz 
</center><br>

<center> 22 de junio de 2019 </center>

<br>

## **Índice**

---

1. [Motivación](#1-motivación)

    1.1. [Licenciamiento](#11-licenciamiento)

2. [Metodología](#2-metodología)

3. [Desarrollo](#3-desarrollo)

    3.1. [Revisión de las encuestas](#31-revisión-de-las-encuestas)

    3.2. [Limpieza de tablas](#32-unificación-y-limpieza-de-las-tablas)

    3.3. [Preguntas y respuestas](#33-preguntas-y-respuestas)

    3.4. [Representación visual de la bodega de datos](#34-representación-visual-de-la-bodega-de-datos)

    3.5 [MapReduce: lenguajes por programador](#35-número-de-programadores-por-lenguaje-de-programación-utilizando-la-metodología-de-mapreduce)

    3.6 [Pronóstico de la encuesta 2018](#36-pronóstico-2018)

    3.7 [Incremento de la bodega de datos](#37-incremento-de-la-bodega-de-datos)

    3.8 [Modificación de indicadores](#38-modificación-de-indicadores)

4. [URL's de interés](#4-url-de-interés)

5. [Contacto](#5-contacto)

<br>

## **1. Motivación**
---

Este es el trabajo final del módulo Sistemas de Bases de Datos Masivos del posgrado de Analítica de la Universidad Nacional de Colombia, Sede Medellín. La guía del módulo puede ser consultada en esta [página web](https://sebastian-gomez.com/bigdata/) creada por el profesor MsC. Sebastián Gómez. El trabajo final, sus preguntas y lineamientos pueden ser consultados [aquí](https://drive.google.com/file/d/18uzP5uGg7OhbEDQIECtT8XtA4jr3NTf9/view).

>Analizar tendencias dentro del mundo de la tecnología se ha convertido en uno de los más grandes retos de la industria, prever cuál será el siguiente y más éxitoso lenguaje de programación o cuál sería el salario ideal para un desarrollador por tecnología se convertirá en una tarea crítica durante los próximos años. Stack Overflow es un sitio web ampliamente utilizado por la comunidad de desarrolladores de software, en la cual otros desarrolladores pueden encontrar soluciones a problemas de programación en diferentes lenguajes. Este sitio realiza desde el 2011 una encuesta a sus usuarios para observar y analizar tendencias en la industria de la tecnología y el software: https://insights.stackoverflow.com/survey/2019. La más reciente fue durante el mismo 2019.

>Las encuestas han sido cuidadosamente almacenadas y expuestas en directorios de datos abiertos. Para este análisis se utilizarán 7 conjuntos de datos masivos completamente libres y accesibles a través de `BigQuery` que son las encuestas de los años 2011 a 2017.

Las encuestas pueden ser consultadas a través de *Google Cloud BigQuery* o están disponibles de manera gratuita en [este enlace](https://insights.stackoverflow.com/survey).

### **1.1. Licenciamiento**

Las bases de datos de las encuestas son publicadas a través de la Licencia de Bases de Datos Abiertas (`ODbL` por sus siglas en inglés) y los términos y condiciones pueden ser consultados [aquí](http://opendatacommons.org/licenses/odbl/1.0/), mientras que todos los derechos individuales sobre el contenido de la base de datos están licenciados a través de la Licencia de Contenido de Bases de Datos (`DbCL` por sus siglas en inglés) y los términos y condiciones pueden ser consultados [aquí](http://opendatacommons.org/licenses/dbcl/1.0/).

Este tipo de licenciamiento da derecho a compartir, adaptar y crear trabajos derivados de las encuestas mientras se le atribuya toda la fuente a Stack Overflow, se mantenga abierta la base de datos resultante y continúe con el licenciamiento `ODbL`.

**Agradecimientos**

La fuente primaria de las bases de datos consultadas es Stack Overflow y contiene información sobre las encuestas realizadas por esta organización. La base de datos resultante, que en este documento se llama "encuestas", así como todos los resultados obtenidos de ella, están disponible gracias a la Licencia de Bases de Datos Abiertas `ODbL`.
<br>

[Regresar](#índice)

## **2. Metodología**
---

Para el desarrollo de este trabajo se propone utilizar el lenguaje de programación libre *Python* en su versión 3.7 de *IPython* y el editor de cuadernos de código, igualmente libre, de *Jupyter* a través de *Google Colab*. En caso de que se vaya a utilizar directamente el código de manera local, se recomienda utilizar *Jupyter* a través de la instalación del software libre [*Anaconda Navigator*](https://www.anaconda.com/distribution/), ya que viene predeterminado con los paquetes de *Python* más comunes para realizar ciencia de datos y con los binarios apropiados para la máquina donde se instale. 

**Instalar paquetes necesarios**

Se propone utilizar una conexión directa a *Google Cloud BigQuery* a través de una API privada, generando las credenciales de acceso. Para ello se utiliza el paquete de *Python google-cloud-bigquery*, el cual se puede instalar ejecutando el comando en la `terminal del equipo`:

```terminal
pip install google-cloud-bigquery
```
En caso de que se tenga instalado *Anaconda Navigator*, se recomienda instalar el paquete a través de la sección de ambientes virtuales dentro del navegador o con el  comando dentro de la `terminal de Anaconda Prompt`:

```terminal
conda install -c conda-forge google-cloud-bigquery
```

Dentro de *Google Colab* no es necesario hacer la instalación ya que viene dentro de los paquetes predeterminados.

**Establecer conexión desde equipo local**

Una vez se instale el paquete necesario, se puede realizar la conexión ejecutando los siguientes códigos en *Python*:

```terminal
from google.cloud import bigquery
from google.oauth2 import service_account

# cargar el archivo de credenciales descargado 
credenciales = service_account.Credentials.from_service_account_file(.../filepath/filename.json)

# identificar el nombre del proyecto
id_proyecto = 'nombre_del_proyecto'

# realizar la conexion
cliente = bigquery.Client(credentials = credenciales, project = id_proyecto)
```

**Establecer conexión desde Google Colab**

En caso que esté ejecutando los comandos en *Google Colab*, es necesario realizar la carga del archivo a *Google Drive* y se establece la conexión de la siguiente forma:

```terminal
# subir el archivo de credenciales a su google drive

# importar la libreria google.colab
from google.colab import drive

# habilitar el uso de tu drive y abrir el vínculo que sale como resultado
drive.mount('/content/gdrive') # ingrese la contraseña generada

# cambiar el directorio
cd gdrive/'My drive'/

# cargar el archivo de credenciales desde Drive
credenciales = service_account.Credentials.from_service_account_file('path/filename.json')

# para mas informacion ir a la seccion de anexos dentro de metodologia
```

**Ejecutar queries**

Una vez la conexión sea exitosa, se pueden ejecutar las rutinas de SQL de la siguiente forma:

```terminal
# Seleccionar todo de la tabla 'Tabla' dentro de la bodega 'BodegaDeDatos'

# se escribe el query como texto crudo
query = '''
SELECT *
FROM `BodegaDeDatos.Tabla`
'''

# se ejecuta el query
consulta = cliente.query(query)

# se traen los resultados como un objeto Rowiterator
resultado = consulta.result()

# si se quiere visualizar los resultados como un DataFrame de Pandas
import pandas as pd

tabla = resultado.to_dataframe()
tabla.head()
```

Si se sigue el proceso descrito y es exitoso, se tendrá la información de `Tabla` en la variable *`tabla`* como un DataFrame.

**Anexos**

* Para más información sobre el paquete *google-cloud-bigquery*, [dar click aquí](https://googleapis.github.io/google-cloud-python/latest/index.html).

* Para más información sobre cómo realizar la conexión con Google BigQuery, [dar click aquí](https://www.blendo.co/blog/access-data-google-bigquery-python-r/).

* Para más información sobre cómo generar las credenciales de acceso, [dar click aquí](https://stackoverflow.com/questions/43004904/accessing-gae-log-files-using-google-cloud-logging-python).

* Para más información sobre cómo cargar archivos a Google Colab, [dar click aquí](https://colab.research.google.com/notebooks/io.ipynb#scrollTo=u22w3BFiOveA).

[Regresar](#índice)

## **3. Desarrollo**
---

El cuaderno de *Jupyter* con el código de programación en *Python* está disponible en este [enlace](https://github.com/Juapatral/encuestas-stack-overflow/blob/master/trabajo_final_sbdm_20191.ipynb). 

### **3.1. Revisión de las encuestas**

#### Estado actual de las tablas de las encuestas

Se realiza la consulta a través de *Google CLoud BigQuery*, utilizando una cuenta educativa para acceder a la información de las encuestas de Stack Overflow de los años 2011 a 2017.

Las encuestas entre los años 2011 a 2015 cuentan con una estructura similar. Si la pregunta es de una única respuesta, esta se guarda en una sola columna; mientras que si la pregunta permite múltiples respuestas, por ejemplo *"seleccione todos los lenguajes de programación que domine de la siguiente lista"*, cada una de las posibles respuestas se guarda en una columna. Los nombres de las columnas son genéricos (*string_field_**n***, donde *n* es el número de la columna), las preguntas y sus posibles respuestas están, en general, en las primeras dos filas de cada conjunto de datos.

Las encuestas de los años 2016 y 2017 manejan un esquema similar, donde los nombres de las columnas corresponden a las preguntas. Si la pregunta contiene múltiples respuestas, estas se guardan en una sola columna separadas por un punto y coma (";").

Finalmente, las preguntas y respuestas varían año a año, tanto en la estructura gramatical de cada una como en el número total de preguntas. Los conjuntos de datos tienen entre 50 campos (el más corto) hasta más de 200 campos (el más largo).

#### Identificación de campos en común a trabajar

Los esquemas de las encuestas son variantes a lo largo de los años, por lo que se requiere una verificación (en su mayor parte manual) de las preguntas y respuestas comunes entre los años a analizar. 

Por esta razón, se eligen las siguientes variables comunes:

+ País
+ Años de experiencia programando
+ Ocupación
+ Salario
+ Lenguajes de programación o tecnologías que domina

[Regresar](#índice)

### **3.2. Unificación y limpieza de las tablas**

Cada encuesta presenta diferentes tipos de datos para cada una de las variables, por lo que se realizó un proceso de limpieza y homologación de datos utilizando funciones de los paquetes de *re*, *numpy*, *itertools* y *pandas*, además de crear unas funciones propias para automatizar el proceso. 

Este proceso de limpieza puede ser consultado en el cuaderno de [***Jupyter***](https://github.com/Juapatral/encuestas-stack-overflow/blob/master/trabajo_final_sbdm_20191.ipynb), en la subsección de *Limpieza de las tablas*.

Después de la limpieza de cada una de las encuestas, el resultado es la bodega de datos ***encuestas*** que cuenta con una única tabla con el siguiente esquema:

| Campo | Tipo  | Descripción |
| --- | --- | --- |
|survey| integer | Año de realización de la encuesta|
| country | string | País que indica la persona que responde la encuesta|
| years_programming |string | Rango de años programando: Rather not say, less than 2 years, 2 - 5 years, 6 - 10 years, 11 or more years|
| occupation| string | Puesto o área de trabajo|
| salary | string | Rango salarial en USD: 0, Less than 20K, 20K - 40K, (rangos de 20K en 20K), 120K - 140K, More than 140K|
| programming_language| string | Lenguajes de programación o tecnologías que la persona dice saber manejar, separados por punto y coma (";")|

[Regresar](#índice)

### **3.3. Preguntas y respuestas**

~~En esta sección van las preguntas, las imágenes de las respuestas y un corto enunciado. Todavía no está terminado~~

De la información de la encuesta se propone responder las siguientes preguntas:

1. ¿Cuántas personas respondieron cada encuesta?

    ![query0](imagenes/query0.PNG)

    >Se observa que la cantidad de personas que respondieron la encuesta es creciente en el tiempo. En el 2011 respondieron un poco menos de 3 mil personas, mientras que en los dos últimos años fueron más de 50 mil. 

2. ¿Cuáles son los diez países que más respondieron las encuestas?

    ![query1](imagenes/query1.PNG)

    >Estados Unidos de América es el país con mayor número de encuestados (cerca de 38 mil). Le siguen India y Reino Unido con un aproximado de 15 mil encuestados cada uno. Esto se ve reflejado en la industria tecnológica y de servicios de telecomunicaciones que tienen cada uno de estos países.

3. ¿Cuántos son los programadores que dominan cada uno de los lenguajes o tecnologías? **(Jhon)**


4. ¿Cuáles son los cinco lenguajes de programación mas usados por encuesta? **(Jhon)**


5. ¿Cuál es el rango salarial mas común por encuesta?

    ![query4](imagenes/query4.PNG)

    >Una vez se homologaron los salarios en rangos salariales, se observa que en los primeros años los salarios eran mucho mayores debido a que no tantos países respondían la encuesta. Una vez se incorporaron más países, el rango salarial disminuyó, a excepción del 2017, donde se muestra un incremento en el rango de compensaciones.
    

6. ¿Cuáles son las tres pincipales ocupaciones por encuesta?

    ![query5](imagenes/query5.PNG)

    >Se observa que en casi todos los años, los estudiantes o profesionales no dedicados al desarrollo de código son los que más respondieron las encuestas. Cabe notar que para el año 2017 se creo la opción de "Desarrollador profesional", ocupación que puede contener muchas otras, explicando así el dato atípico para dicho año. 

7. ¿Cuáles son las tres principales ocupaciones por pais? **(Jhon)**


8. ¿Cuál es el principal rango salarial en cada país de los diez países con más respuestas?

    ![query7](imagenes/query7.PNG)

    >Los países con más nivel de desarrollo económico como Estados Unidos, Reino Unido y Australia, presentan los salarios más grandes, mientras que países menos desarrollados como India y Polonía registran menores rangos salariales. 

9. ¿Cuál es el lenguaje con más crecimiento en las encuestas? **(Jhon)** ***(Muy importante para la predicción)***


10. ¿Cuál es el lenguaje mas popular en cada país de los diez países con mas programadores?

    ![query9](imagenes/query9.PNG)

    >Se observa que JavaScript es el lenguaje preponderante en cada uno de los países, ocupando entre el 30 % y el 50 % del total de personas que respondieron la encuesta en cada país. Solo en "Europa: otro" se observa mayor participación por SQL.

11. ¿Cuáles son las cinco ocupaciones más populares de los programadores que dominan *Python* por encuesta? **(Jhon)**


12. Dentro de los dos rangos salariales más altos, ¿cuáles son las tres ocupaciones con mayor número  de programadores?

    ![query11](imagenes/query11.PNG)

    >Los encuestados con mayores rangos salariales se dedican a ser desarrolladores profesionales o completamente al desarrollo en entorno Web.

12. ¿Cuál es el país más experto e inexperto programando?

    ![query12](imagenes/query12.PNG)

    >Estados Unidos tiene la mayor cantidad de programadores con más años de experiencia, mientras que India tiene la mayor cantidad con menos años de experiencia. Estos resultados son consistentes ya que son los dos países que tienen mayor número de encuestas respondidas. 


14. ¿Cuál es el número promedio de lenguajes que saben los programadores por encuesta?

    ![query13](imagenes/query13.PNG)

    >Se observa que, en promedio, las personas que contestaron la encuesta dominan entre 3 y 8 lenguajes de programación. Este dato es muy variante entre los años debido a las opciones de respuesta que tienen las preguntas relacionadas con los lenguajes o tecnologías que dominan. Se observa que para 2013 y 2014, estas respuestas eran muy específicas (alrededor de 10 posibles), mientras que de 2015 en adelante se presentan más de 30 opciones.
    También es de interés conocer algunos datos puntuales acerca de *Colombia* y su participación en las encuestas de *Stack Overflow*
    <br>

15. ¿Cuánto es el salario más común en Colombia por encuesta?

    ![query14](imagenes/query14.PNG)

    >Los encuestados solo respondieron que son de Colombia a partir del año 2014. Se observa que el rango salarial más común es Less than 20K, mientras que para el año 2017 se presentan salarios muy elevados, similar a la distribución a nivel global.


16. ¿Cuál es la ocupacion más popular en Colombia por encuesta? **(Jhon)**


17. ¿Cuáles son los tres lenguajes más usados en Colombia por encuesta?

    ![query16](imagenes/query16.PNG)

    >En Colombia, el lenguaje de programación más popular es JavaScript y muestra una clara tendencia de crecimiento, mientras que SQL en su versión estandar o MySQL también ocupa una posición importante. 

Todas  gráficas se encuentran dentro de la carpeta de *imágenes* en la rama principal de [este repositorio](https://github.com/Juapatral/encuestas-stack-overflow), identificadas con el nombre *"query**n**.PNG"*, donde n es el número de la pregunta menos 1. Ejemplo, para la pregunta 1, la imagen se llama *"query0.PNG"*.

[Regresar](#índice)

### **3.4. Representación visual de la bodega de datos**

A continuación se representa de forma gráfica la estructura de la bodega de datos a través de tres modelos: Cubo, Estrella y Malinowski.

***Modelo del cubo***

Para este modelo se toman tres dimensiones de la bodega de datos:

+ Año de la encuesta
+ País de la persona que responde la encuesta
+ Rango salarial de la persona que responde la encuesta

Como existen muchas categorías por dimensión, se toman los años 2011, 2012 y 2017, los países Colombia, México y Ecuador y los rangos salariales 0, 120K - 140K, More than 140K.

*<center> Imagen del modelo del cubo </center>*


![Imagen del cubo](imagenes/modelo-cubo.PNG)


***Modelo de estrella***

Para este modelo se identifican todas las dimensiones (o columnas para como se constituyó la bodega de datos) y el hecho que en este caso son las mismas encuestas.

*<center> Imagen del modelo de estrella </center>*

![Imagen de estrella](imagenes/Estrella.PNG)


***Modelo de Malinowski***

El modelo de Malinowski es similar al modelo estrella, cambiando la notación del hecho, las medidas del hecho y las dimensiones. Estas últimas se pueden desagregar a nivel de atributos, pero para este ejercicio no es posible debido a la forma como está constituida la bodega de datos.

*<center> Imagen del modelo de Malinowski </center>*


![Imagen de Malinowski](https://juapatral.github.io/encuestas-stack-overflow/imagenes/Malinowski.PNG)


[Regresar](#índice)

### **3.5. Número de programadores por lenguaje de programación utilizando la metodología de MapReduce**

~~Espacio para escribir: **Jhon**~~

[Regresar](#índice)

### **3.6. Pronóstico 2018**

Uno de los objetivos de este trabajo es pronosticar los resultados de la encuesta del año 2018 utilizando una tabla de la bodega de datos con al menos 50.000 registros. Para el desarrollo de este objetivo, se tomará el conjuto de datos completo y los siguientes supuestos de crecimiento.

*Nota:* los resultados de las encuestas 2018 y 2019 están disponibles en el [sitio oficial](https://insights.stackoverflow.com/survey) de Stack Overflow. Sin embargo, no se tomarán en cuenta dichas encuestas ni sus resultados para establecer los pronósticos.

#### Supuestos

Lo primero que hay que pronosticar es cuántas personas podrían responder la encuesta para el año 2018. Para ello, se tomará una tendencia lineal de la cantidad de personas que respondieron la encuesta. 

Para las variables de interés se tienen los siguientes supuestos:

| columna | criterio |
| --- | --- |
| country | Participación de cada país a lo largo de los años.|
| years_programming | Participación de cada rango de experiencia y su tasa de crecimiento.|
| occupation | Participación de cada ocupación y su tasa de crecimiento.|
| salary | Participación por cada país.|
| programming_language| Tasa de crecimiento de cada uno y promedio de número de lenguajes o tecnologías que domina cada encuestado.| 

#### Resultados preliminares

~~Espacio para escribir: **JHON JP**~~

#### Ejecución del código

~~Espacio para escribir: **JHON JP**~~

[Regresar](#índice)

### **3.7. Incremento de la bodega de datos**

Suponga que, por alguna causa ajena a este ejercicio, una respuesta de una encuesta de alguno de los años quedó por for fuera de la bodega de datos. Es de interés ingresar las respuestas, limpiarlas para que sean acordes al esquema de la tabla *encuestas* y recalcular todos los indicadores definidios en la sección de [preguntas y respuestas](#33-preguntas-y-respuestas). 

Se propone realizar la inscripción de cada respuesta a través de una entrada definida por el usuario, la cual es validada si cumple con las características de cada tipo de dato. Para ello, se crea una función propia en *Python* con la siguiente documentación:

    '''
    ingresar_registro()

    Esta función inicia automáticamente la validación de una nueva respuesta a la tabla encuestas, donde al usuario se le solicita ingresar un dato a la vez. Si el dato es válido, se inicia el proceso de limpieza de dicho 
    dato. 

    Si todos los datos son válidos, esta función devuelve un pandas.DataFrame con la misma estructura de encuestas. En caso contrario, devuelve None.
    '''

 Para ingresar el registro validado a la tabla *encuestas*, se puede ejecutar el siguiente código:

```terminal
# se ingresa el nuevo dato
nuevo_dato = ingresar_registro()

# se verifica si el dato es valido
if nuevo_dato is None:
    print('\nIngrese datos correctos por favor')
    
else:
    encuestas = pd.concat([encuestas,nuevo_dato], ignore_index = True)

# se devuelve el final de la tabla para verificar ingreso exitoso
encuestas.tail()
```   

Después de definir la metodología para ingresar una respuesta más a cualquiera de los años, se vuelven a hacer todos los cálculos para cada una de las gráficas. Para este recálculo, se utiliza la funcionalidad del *magic* *`%rerun`* de *IPython* de la siguiente manera:

```terminal
# este es un ejemplo para repetir la ultima celda que contenga el patron consulta15

%rerun -g consulta15
```

Utilizando este *magic* se pueden realizar nuevamente las gráficas de todas las preguntas y respuestas de la *[sección 3.3.](#33-preguntas-y-respuestas)* 

Para más información acerca de la funcionalidad el *magic `%rerun`*, dar click [aquí](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-rerun).


**Ejemplo**

Se quiere añadir la siguiente respuesta:

|columna|nuevo dato|
|---|---|
|survey|2013|
|country|Colombia|
|years_programming|Less than 2 years|
|occupation|Student|
|salary|Less than 20K|
|programming_language|Python;R;PostgreSQL;Tableau|

Se ejecuta la función *`ingresar_registro()`* para crear el nuevo objeto de clase *pandas.DataFrame* y después se añade a la tabla de encuestas, utilizando los códigos definidos previamente. Ademas, las gráficas de las preguntas  se actualizan después de ejecutar el *magic %rerun*.

Se presentan dos ejemplos de la actualización de los indicadores:

Para la pregunta 15: ¿cuánto es el salario más común en Colombia por encuesta?

![query14](imagenes/query14.PNG)
![query14-v2](imagenes/query14-v2.PNG)

Se observa la creación de una nueva columna para el registro del año 2013, identificando el rango salarial.

De forma similar, para la pregunta 17: ¿cuáles son los tres lenguajes más usados en Colombia por encuesta?

![query16](imagenes/query16.PNG)
![query16-v2](imagenes/query16-v2.PNG)

Se agregaron los lenguajes de programación y el año de respuesta de la encuesta al inicio de la gráfica.

Toda la actualización de las demás gráficas se encuentran dentro de la carpeta de *imágenes* en la rama principal de [este repositorio](https://github.com/Juapatral/encuestas-stack-overflow), identificadas con el nombre *"query**n**-v2.PNG"*, donde n es el número de la pregunta menos 1. Ejemplo, para la pregunta 1, la imagen se llama *"query0-v2.PNG"*. 

[Regresar](#índice)

### **3.8 Modificación de indicadores**

Suponga que, por alguna causa ajena a este ejercicio, los datos ingresados en la tabla *encuestas* no son reales y deben ser modificados. Es de interés ingresar las respuestas, limpiarlas para que sean acordes al esquema de la tabla *encuestas* y recalcular todos los indicadores definidios en la sección de [preguntas y respuestas](#33-preguntas-y-respuestas).  

Se propone que la modificación de los datos esté dada por la identificación de cuáles registros se quieren modificar y por qué valores o textos deben ser reemplazados. Para ello, se crean dos funciones propias en *Python*: *`cambiar_lista(args)`* y *`cambiar_dataframe(arg)`*. Sus respectivas documentaciones son las siguientes:

```terminal
# funcion cambiar lista

"""cambiar_lista(lista, patron, reemplazo, cambios = None, entero = False, reg_expr = True)
  
  Esta función busca patron sobre la lista y cambia los elementos por reemplazo, en los primeros n registros definidos por cambios. 
  
  lista     (list) lista sobre la que se quiere modificar información
  
  patron    (str)  patrón a buscar sobre la lista, puede ser una expresión regular si reg_expr es True
  
  reemplazo (str)  valor a reemplazar para todos los elementos encontrados con patron
  
  cambios   (int)  Por defecto None. cantidad de cambios a realizar, si es None, se realizan todos los cambios que sean posible
  
  entero    (bool) Por defecto False. se establece si la lista se devuelve como enteros
  
  reg_expr  (bool) Por defecto True. se establece si patron es una expersion regular o si se quiere buscar explícitamente
  """

# funcion cambiar_dataframe

  """cambiar_dataframe(dataframe, columna, patron, reemplazo, dataframe_reemplazo = None, cambios = None, entero = False, reg_expr = True):
  
  Cambia todos los elementos de la columna columna de dataframe, utilizando la funcion cambiar_lista
  definida anteriormente (wrapper)
  
  dataframe             (dataframe) dataframe sobre el cual realizar el cambio
  
  columna               (str)       columna a la cual se quiere realizar el cambio
  
  patron                (str)       (cambiar_lista) patrón a buscar sobre la lista, puede ser una expresión regular si reg_expr es True
  
  reemplazo             (str)       (cambiar_lista) valor a reemplazar para todos los elementos encontrados con patron
  
  dataframe_reemplazo   (dataframe) dataframe objeto del reemplazo
  
  cambios               (int)       (cambiar_lista) Por defecto None. cantidad de cambios a realizar, si es None, se realizan todos los cambios que sean posible
  
  entero                (bool)      (cambiar_lista) Por defecto False. se establece si la lista se devuelve como enteros
  
  reg_expr              (bool)      (cambiar_lista) Por defecto True. se establece si patron es una expersion regular o si se quiere buscar explícitamente
  """
```
Al igual que en la *[sección 3.7](#37-incremento-de-la-bodega-de-datos)*, una vez se realizan las modificaciones sobre la bodega de datos se vuelven a hacer todos los cálculos para cada una de las gráficas. Para este recálculo, se utiliza la funcionalidad del *magic* *`%rerun`* de *IPython*.

Después de definir la metodología, se vuelven a calcular cada una de las gráficas definidas para las preguntas y respuestas de la *[sección 3.3.](#33-preguntas-y-respuestas)*

**Ejemplo**

Se quiere actualizar la información del rango salarial más común en Colombia por año de encuesta y convertirlos en el rango salarial *"120K - 140K"*. 

Para realizar el proceso de modificación se ejecutan los siguientes comandos:

```terminal
# EJEMPLO
# para las encuestas de colombia, cambiar para que el rango salarial mas alto en todos los anios sea 120K - 140K

encuestas2 = encuestas[encuestas['country'] == 'Colombia']
cambiar_dataframe(encuestas2,
                  'salary',                 
                  'Less than 20K', 
                  '120K - 140K',
                  dataframe_reemplazo = encuestas,
                  cambios = None
                 )
```

Después de actualizada la tabla *encuestas*, las gráficas de las preguntas  se actualizan al ejecutar el *magic %rerun*.

Para visualizar los resultados se toma como muestra la pregunta pregunta 15: ¿cuánto es el salario más común en Colombia por encuesta?

![query14](imagenes/query14.PNG)
![query14-v3](imagenes/query14-v3.PNG)

Se observa la modificación de los rangos salariales y el incremento de la cantidad de encuestas respondidas por año. 

Toda la actualización de las demás gráficas se encuentran dentro de la carpeta de *imágenes* en la rama principal de [este repositorio](https://github.com/Juapatral/encuestas-stack-overflow), identificadas con el nombre *"query**n**-v3.PNG"*, donde n es el número de la pregunta menos 1. Ejemplo, para la pregunta 1, la imagen se llama *"query0-v3.PNG"*. 

[Regresar](#índice)

## **4. URL de interés**
---
A continuación se listan unas páginas web donde se puede encontrar documentación sobre las funciones y procedimientos realizados en este trabajo.

---
**Paquete *pandas***

[Documentación del paquete *pandas* de *Python*](https://pandas.pydata.org/pandas-docs/stable/index.html)

[Cómo concatenar dataframes con *pandas*](https://pandas.pydata.org/pandas-docs/stable/user_guide/merging.html)

[Cómo combinar diferentes columnas de texto con *pandas*](https://stackoverflow.com/questions/19377969/combine-two-columns-of-text-in-dataframe-in-pandas-python)

[Cómo contar patrones de caracteres con *pandas*](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.str.count.html)

[Cómo renombrar columnas con *pandas*](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.rename.html)

[Cómo convertir índices en columnas con *pandas*](https://stackoverflow.com/questions/20461165/how-to-convert-index-of-a-pandas-dataframe-into-a-column)

[Cómo anidar múltiples métodos con *pandas*](https://tomaugspurger.github.io/method-chaining.html)

[Cómo filtrar por múltiples elementos con *pandas*](https://stackoverflow.com/questions/12096252/use-a-list-of-values-to-select-rows-from-a-pandas-dataframe)

[Cómo cambiar datos de un dataframe con *pandas*](https://stackoverflow.com/questions/45093241/how-to-replace-part-of-dataframe-in-pandas)

[Cómo cambiar un dato específico de un dataframe con *pandas*](https://stackoverflow.com/questions/31569384/set-value-for-particular-cell-in-pandas-dataframe-with-iloc)

---
**Expresiones regulares *re***

[Documentación del paquete *re* de *Python*](https://docs.python.org/3/library/re.html)

[Cómo trabajar con expresiones regulares con *re*](https://www.guru99.com/python-regular-expressions-complete-tutorial.html)

---

**Graficar *matplotlib***

[Documentacion del paquete *matplotlib* de *Python*](https://matplotlib.org/3.1.0/index.html#)

[Cómo guardar una gráfica de *matplotlib* usando *savefig*](https://matplotlib.org/3.1.0/api/_as_gen/matplotlib.pyplot.savefig.html)

---

***Markdown***

[Documentación sobre el formato *Markdown*](https://www.markdownguide.org/getting-started)

[Cómo crear tablas en formato Markdown](https://www.tablesgenerator.com/markdown_tables)

[Cómo aplicar formato de texto en *Markdown*](https://support.discordapp.com/hc/en-us/articles/210298617-Markdown-Text-101-Chat-Formatting-Bold-Italic-Underline-)

---

***Jupyter***

[Cómo añadir imágenes a un cuaderno de *Jupyter*](https://stackoverflow.com/questions/10628262/inserting-image-into-ipython-notebook-markdown)

[Cómo repetir comandos de *Jupyter* en otra celda](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-rerun)

[Cómo ingresar datos por usuario a cuaderno de *Jupyter*](https://stackoverflow.com/questions/34968112/how-to-give-jupyter-cell-standard-input-in-python)

---

**Varios**

[Ejemplo de trabajar con tipos de dato *None* o *Nonetype*](https://stackoverflow.com/questions/23086383/how-to-test-nonetype-in-python)

[Ejemplo de añadir elementos a un diccionario](https://thispointer.com/python-how-to-add-append-key-value-pairs-in-dictionary-using-dict-update/)

[Guía de estilo para código *Python*](https://www.python.org/dev/peps/pep-0008/)

---

[Regresar](#índice)

## **5. Contacto**

Si existe alguna duda, sugerencia o comentario sobre este informe, el código de programación o [información legal](#11-licenciamiento) al respecto, así como reportar enlaces no funcionales o código erroneo, por favor escribir a alguno de los siguientes correos:

+ Juan Pablo Trujillo Alviz: *jptrujilloa@unal.edu.co*
+ Jhon Anderson Londoño Herrera: *jalondonh@unal.edu.co*

[Regresar](#índice)