# <center> Trabajo final Sistema de Bases de Datos Masivos </center>

## <center> Análisis de las encuestas de Stack Overflow entre los años 2011 a 2017 </center>

---

### <center> Posgrado de Analítica, 2019-1 </center>

###  <center> Integrantes </center>​
<center> Jhon Anderson Londoño Herrera<br>
Juan Pablo Trujillo Alviz 
</center>

<br>

<center> 22 de junio de 2019 </center>

<br>

## Índice

---

1. [Motivación](#motivación)

> 1.1[Licenciamiento](#licenciamiento)

2. [Metodología](#metodología)


3. [Desarrollo](#desarrollo)

> 3.1 [Establecer conexión](#establecer-conexión)

> 3.2 [Limpieza de tablas](#limpieza-de-las-tablas)

> 3.3 [SQL](#15-sql)

> 3.4 [Representación visual de la bodega de datos](#representación-visual-de-la-bodega-de-datos)

4. [URL's de interés](#urls-de-interes)

<br>

## Motivación
---

Este es el trabajo final del módulo Sistemas de Bases de Datos Masivos del posgrado de Analítica de la Universidad Nacional de Colombia, Sede Medellín. La guía del módulo puede ser consultado en esta [página web](https://sebastian-gomez.com/bigdata/) creada por el profesor MsC. Sebastián Gómez. El trabajo final, sus preguntas y lineamientos pueden ser consultados [aquí](https://drive.google.com/file/d/18uzP5uGg7OhbEDQIECtT8XtA4jr3NTf9/view).

Analizar tendencias dentro del mundo de la tecnología se ha convertido en uno de los más grandes retos de la industria, prever cuál será el siguiente y más éxitoso lenguaje de programación o cuál sería el salario ideal para un desarrollador por tecnología se convertirá en una tarea crítica durante los próximos años. Stack Overflow es un sitio web ampliamente utilizado por la comunidad de desarrolladores de software, en la cual otros desarrolladores pueden encontrar soluciones a problemas de programación en diferentes lenguajes. Este sitio realiza desde el 2011 una encuesta a sus usuarios para observar y analizar tendencias en la industria de la tecnología y el software: https://insights.stackoverflow.com/survey/2019. La más reciente fue durante el mismo 2019. 

Las encuestas han sido cuidadosamente almacenadas y expuestas en directorios de datos abiertos. Para este análisis se utilizarán 7 conjuntos de datos masivos completamente libres y accesibles a través de BigQuery que son las encuestas de los años **2011 a 2017**.

Las encuestas pueden ser consultadas a través de *Google Cloud BigQuery* o están disponibles de manera gratuita en [este enlace](https://insights.stackoverflow.com/survey). (Importante revisar el licenciamiento de las bases de datos antes de trabajar con ellas)

#### Licenciamiento

Todas las encuestas desde el 2011 al 2019 están disponibles [aquí](https://insights.stackoverflow.com/survey).


Estas bases de datos son publicables a través de la Licencia de Bases de Datos Abiertas (ODbL por sus siglas en inglés) y los términos y condiciones pueden ser consultados [aquí](http://opendatacommons.org/licenses/odbl/1.0/), mientras que todos los derechos individuales sobre el contenido de la base de datos están licenciados a través de la Licencia de Contenido de Bases de Datos (DbCL por sus siglas en inglés) y los términos y condiciones pueden ser consultados [aquí](http://opendatacommons.org/licenses/dbcl/1.0/).

Este tipo de licenciamiento da derecho a compartir, adaptar y crear trabajos derivados de las encuestas mientras se le atribuya toda la fuente a Stack Overflow, se mantenga abierta la base de datos resultante y continúe con el licenciamiento ODbL.

**Agradecimientos**

La fuente primaria de las bases de datos consultadas es Stack Overflow y contiene información sobre las encuestas realizadas por esta organización. La base de datos resultante, que en este documento se llama "encuestas", así como todos los resultados obtenidos de ella, están disponible gracias a la Licencia de Bases de Datos Abiertas ([ODbL](https://opendatacommons.org/licenses/odbl/1.0/)).

<br>

## Metodología
---

**Instalar paquetes necesarios**


Se propone utilizar una conexión directa a *Google BigQuery* a través de una API privada, generando las credenciales de acceso. Para ello se utiliza el paquete de python *google-cloud-bigquery* el cual se puede instalar ejecutando el comando en la terminal:

```terminal
pip install google-cloud.bigquery
```
En caso de que se tenga instalado *Anaconda Navigator*, se recomienda instalar el paquete con el siguiente comando dentro de la terminal de Anaconda:

```terminal
conda install -c conda-forge google-cloud-bigquery
```

**Establecer conexión desde equipo local**

Una vez se instale el paquete necesario, se puede realizar la conexión ejecutando los siguientes códigos en python:

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

En caso que esté ejecutando los comandos en *Google Colab*, es necesario realizar la carga del archivo a *Google Drive* y se llama de la siguiente forma.

```terminal
# subir el archivo de credenciales a su google drive
# para mas informacion ir a la seccion de anexos dentro de metodologia

# importar la libreria google.colab
from google.colab import drive

# habilitar el uso de tu drive y abrir el vínculo que sale como resultado
drive.mount('/content/gdrive') # ingrese la contraseña generada

# cambiar el directorio
cd gdrive/'My drive'/

# cargar el archivo de credenciales desde Drive
credenciales = service_account.Credentials.from_service_account_file('filename.json')
```

**Ejecutar queries**

Una vez la conexión sea exitosa, se pueden ejecutar las rutinas de SQL de la siguiente forma:

```terminal
# se escribe el query como texto crudo
query = '''
SELECT *
FROM `dataset.table`
'''

# se ejecuta el query
consulta = cliente.query(query)

# se traen los resultados como un objeto Rowiterator
resultado = consulta.result()

# si se quiere visualizar los resultados como un dataframe de pandas
import pandas as pd

tabla = resultado.to_dataframe()
tabla.head()
```
<br>

**Anexos**

* Para más información sobre el paquete *google-cloud-bigquery*, [dar click aquí](https://googleapis.github.io/google-cloud-python/latest/index.html).

* Para más información sobre cómo realizar la conexión con Google BigQuery, [dar click aquí](https://www.blendo.co/blog/access-data-google-bigquery-python-r/).

* Para más información sobre cómo generar las credenciales de acceso, [dar click aquí](https://stackoverflow.com/questions/43004904/accessing-gae-log-files-using-google-cloud-logging-python).

* Para más información sobre cómo cargar archivos a Google Colab, [dar click aquí](https://colab.research.google.com/notebooks/io.ipynb#scrollTo=u22w3BFiOveA).

<br>

## Desarrollo
---

### Estado actual de las tablas de las encuestas

Se realiza la consulta a través de *Google BigQuery* utilizando una cuenta educativa para acceder a la información de las encuestas de Stack Overflow de los años 2011 a 2017.

Las encuestas entre los años 2011 a 2015 cuentan con una estructura similar: si la pregunta es de una única respuesta, esta se guarda en una sola columna, mientras que si la pregunta permitía múltiples respuestas, e.g. seleccione de los siguientes los lenguajes de programación que domine, cada una de las posibles respuestas se guarda en una columna. Los nombres de las columnas son genéricos (*string_field_**n***, donde *n* es el número de la columna), las preguntas y sus posibles respuestas están, en general, en las primeras dos filas de cada conjunto de datos.

Las encuestas de los años 2016 y 2017 manejan un esquema similar a los años, sin embargo, los nombres de las columnas corresponden a las preguntas. Si la pregunta contiene múltiples respuestas, estas se guardan en una sola columna separadas por un punto y coma (";").

Finalmente, las preguntas y respuestas varían año a año, tanto en la estructura gramatical de cada una como en el número. Los conjuntos de datos cuentan entre 50 campos (el más corto) hasta más de 200 campos.


### Identificación de campos en común

Los esquemas de las encuestas son variantes a lo largo de los años, por lo que se requiere una verificación, en su mayor parte manual, de las preguntas y respuestas comunes entre los años a analizar. 



### 15 SQL

### Representación visual de la bodega de datos

***Modelo del cubo***

Para este modelo se toman tres dimensiones de la bodega de datos:

+ Año de la encuesta
+ País de la persona que responde la encuesta
+ Rango salarial de la persona que responde la encuesta

Como existen muchas categorías por dimensión, se toman los años 2011, 2012 y 2017, los países Colombia, México y Ecuador y los rangos salariales 0, 120K - 140K, More than 140K.

*<center> Modelo del cubo </center>*

![Imagen del cubo](imagenes/modelo-cubo.png)

***Modelo de estrella***

Para este modelo se identifican todas las dimensiones (o columnas para como se constituyó la bodega de datos) y el hecho que en este caso son las mismas encuestas.

*<center> Modelo de estrella </center>*

![Imagen de estrella](imagenes/estrella.png)

***Modelo de Malinowski***

El modelo de Malinowski es similar al modelo estrella, cambiando la notación del hecho, las medidas del hecho y las dimensiones. Estas últimas se pueden desagregar a nivel de atributos, pero para este ejercicio no es posible debido a la forma como está constituida la bodega de datos.

*<center> Modelo de Malinowski </center>

![Imagen de estrella](imagenes/malinowski.png)