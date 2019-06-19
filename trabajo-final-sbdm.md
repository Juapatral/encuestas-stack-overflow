# <center> Trabajo final Sistema de Bases de Datos Masivos </center>

## <center> Análisis de las encuestas de Stack Overflow entre los años 2011 y 2017 </center>

---

### <center> Posgrado de Analítica, 2019-1 </center>
###  <center> Integrantes </center>​<br>
<center> Jhon Anderson Londoño Herrera<br>
Juan Pablo Trujillo Alviz 
</center><br>

<center> 22 de junio de 2019 </center>

<br>

## **Índice**

---

1. [Motivación](#motivación)

    1.1. [Licenciamiento](#licenciamiento)

2. [Metodología](#metodología)

3. [Desarrollo](#desarrollo)

    3.1. [Establecer conexión](#establecer-conexión)

    3.2. [Limpieza de tablas](#unificación-y-limpieza-de-las-tablas)

    3.3. [Preguntas y respuestas](#preguntas-y-respuestas)

    3.4. [Representación visual de la bodega de datos](#representación-visual-de-la-bodega-de-datos)

    3.5 [Pronóstico de la encuesta 2018](#pronóstico-2018)

4. [URL's de interés](#urls-de-interes)

<br>

## **Motivación**
---

Este es el trabajo final del módulo Sistemas de Bases de Datos Masivos del posgrado de Analítica de la Universidad Nacional de Colombia, Sede Medellín. La guía del módulo puede ser consultado en esta [página web](https://sebastian-gomez.com/bigdata/) creada por el profesor MsC. Sebastián Gómez. El trabajo final, sus preguntas y lineamientos pueden ser consultados [aquí](https://drive.google.com/file/d/18uzP5uGg7OhbEDQIECtT8XtA4jr3NTf9/view).

>Analizar tendencias dentro del mundo de la tecnología se ha convertido en uno de los más grandes retos de la industria, prever cuál será el siguiente y más éxitoso lenguaje de programación o cuál sería el salario ideal para un desarrollador por tecnología se convertirá en una tarea crítica durante los próximos años. Stack Overflow es un sitio web ampliamente utilizado por la comunidad de desarrolladores de software, en la cual otros desarrolladores pueden encontrar soluciones a problemas de programación en diferentes lenguajes. Este sitio realiza desde el 2011 una encuesta a sus usuarios para observar y analizar tendencias en la industria de la tecnología y el software: https://insights.stackoverflow.com/survey/2019. La más reciente fue durante el mismo 2019. 

>Las encuestas han sido cuidadosamente almacenadas y expuestas en directorios de datos abiertos. Para este análisis se utilizarán 7 conjuntos de datos masivos completamente libres y accesibles a través de `BigQuery` que son las encuestas de los años 2011 a 2017.

Las encuestas pueden ser consultadas a través de *Google Cloud BigQuery* o están disponibles de manera gratuita en [este enlace](https://insights.stackoverflow.com/survey). (Importante revisar el licenciamiento de las bases de datos antes de trabajar con ellas)

[Regresar](#índice)

### **Licenciamiento**

Todas las encuestas desde el 2011 al 2019 están disponibles [aquí](https://insights.stackoverflow.com/survey).

Estas bases de datos son publicables a través de la Licencia de Bases de Datos Abiertas (ODbL por sus siglas en inglés) y los términos y condiciones pueden ser consultados [aquí](http://opendatacommons.org/licenses/odbl/1.0/), mientras que todos los derechos individuales sobre el contenido de la base de datos están licenciados a través de la Licencia de Contenido de Bases de Datos (DbCL por sus siglas en inglés) y los términos y condiciones pueden ser consultados [aquí](http://opendatacommons.org/licenses/dbcl/1.0/).

Este tipo de licenciamiento da derecho a compartir, adaptar y crear trabajos derivados de las encuestas mientras se le atribuya toda la fuente a Stack Overflow, se mantenga abierta la base de datos resultante y continúe con el licenciamiento ODbL.

**Agradecimientos**

La fuente primaria de las bases de datos consultadas es Stack Overflow y contiene información sobre las encuestas realizadas por esta organización. La base de datos resultante, que en este documento se llama "encuestas", así como todos los resultados obtenidos de ella, están disponible gracias a la Licencia de Bases de Datos Abiertas ([ODbL](https://opendatacommons.org/licenses/odbl/1.0/)).
<br>

[Regresar](#índice)

## **Metodología**
---

Para el desarrollo de este trabajo se propone utilizar el lenguaje de programación libre *Python* en su versión 3.7 de *IPython* y el editor de cuadernos de código, igualmente libre, de *Jupyter* a través de *Google Colab*. En caso de que se vaya a utilizar directamente el código de manera remota, se propone utilizar *Jupyter* a través de la instalación del software libre [*Anaconda Navigator*](https://www.anaconda.com/distribution/) ya que viene predeterminado con los paquetes de *Python* más comunes para realizar ciencia de datos con los binarios apropiados para la máquina donde se instale. 

**Instalar paquetes necesarios**

Se propone utilizar una conexión directa a *Google BigQuery* a través de una API privada, generando las credenciales de acceso. Para ello se utiliza el paquete de *Python google-cloud-bigquery*, el cual se puede instalar ejecutando el comando en la `terminal del equipo`:

```terminal
pip install google-cloud.bigquery
```
En caso de que se tenga instalado *Anaconda Navigator*, se recomienda instalar el paquete a través de la sección de ambientes virtuales dentro del navegador o con el  comando dentro de la `terminal de Anaconda Prompt`:

```terminal
conda install -c conda-forge google-cloud-bigquery
```

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
credenciales = service_account.Credentials.from_service_account_file('path/filename.json')
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

[Regresar](#índice)

## **Desarrollo**
---

### **Estado actual de las tablas de las encuestas**

Se realiza la consulta a través de *Google BigQuery* utilizando una cuenta educativa para acceder a la información de las encuestas de Stack Overflow de los años 2011 a 2017.

Las encuestas entre los años 2011 a 2015 cuentan con una estructura similar: si la pregunta es de una única respuesta, esta se guarda en una sola columna, mientras que si la pregunta permitía múltiples respuestas, e.g. seleccione de los siguientes los lenguajes de programación que domine, cada una de las posibles respuestas se guarda en una columna. Los nombres de las columnas son genéricos (*string_field_**n***, donde *n* es el número de la columna), las preguntas y sus posibles respuestas están, en general, en las primeras dos filas de cada conjunto de datos.

Las encuestas de los años 2016 y 2017 manejan un esquema similar donde los nombres de las columnas corresponden a las preguntas. Si la pregunta contiene múltiples respuestas, estas se guardan en una sola columna separadas por un punto y coma (";").

Finalmente, las preguntas y respuestas varían año a año, tanto en la estructura gramatical de cada una como en el número total de preguntas. Los conjuntos de datos cuentan entre 50 campos (el más corto) hasta más de 200 campos (el más largo).

### **Identificación de campos en común a trabajar**

Los esquemas de las encuestas son variantes a lo largo de los años, por lo que se requiere una verificación, en su mayor parte manual, de las preguntas y respuestas comunes entre los años a analizar. 

Por esta razón, se eligen las siguientes variables comunes:

+ País
+ Años de experiencia programando
+ Ocupación
+ Salario
+ Lenguajes de programación o tecnologías que domina

[Regresar](#índice)

### **Unificación y limpieza de las tablas**

Cada encuesta presenta diferentes tipos de datos para cada una de las variables, por lo que se realizó un proceso de limpieza y homologación de datos utilizando funciones de los paquetes de *re*, *numpy*, *itertools* y *pandas*, además de crear unas funciones propias para automatizar el proceso. 

Este proceso de limpieza puede ser consultado en el cuaderno de *Jupyter*, accesible **[aquí PENDIENTE](https://google.com)**, en la subsección de *Limpieza de las tablas*.

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

### **Preguntas y respuestas**

~~En esta sección van las preguntas, las imágenes de las respuestas y un corto enunciado. Todavía no está terminado~~

De la información de la encuesta se propone responder las siguientes preguntas:

1. ¿Cuántas personas respondieron cada encuesta?

    ![query0](https://juapatral.github.io/encuestas-stack-overflow/imagenes/query0.PNG)

    >Se observa que la cantidad de personas que respondieron la encuesta es creciente en el tiempo, donde en el 2011 respondieron un poco menos de 3 mil personas, mientras que en los dos últimos años fueron más de 50 mil. 

2. ¿Cuáles son los diez países que más respondieron las encuestas?

    ![query1](https://juapatral.github.io/encuestas-stack-overflow/imagenes/query1.PNG)

    >Estados Unidos de América es el país con mayor número de encuestados (cerca de 38 mil). Le siguen India y Reino Unido con un aproximado de 15 mil encuestados cada uno. Esto se ve reflejado en la industria tecnológica y de servicios de telecomunicaciones que tienen cada uno de estos países.

3. ¿Cuántos son los programadores que dominan cada uno de los lenguajes o tecnologías? **(Jhon)**


4. ¿Cuáles son los cinco lenguajes de programación mas usados por encuesta? **(Jhon)**


5. ¿Cuál es el rango salarial mas común por encuesta?

    ![query4](https://juapatral.github.io/encuestas-stack-overflow/imagenes/query4.PNG)

    >Una vez se homologaron los salarios en rangos salariales, se observa que en los primeros años los salarios eran mucho mayores debido a que no tantos países respondían la encuesta. Una vez se incorporaron más países, el rango salarial disminuyó, a excepción del 2017, donde se muestra un incremento en el rango de compensaciones.
    

6. ¿Cuáles son las tres pincipales ocupaciones por encuesta?

    ![query5](https://juapatral.github.io/encuestas-stack-overflow/imagenes/query5.PNG)

    >Se observa que en casi todos los años, los estudiantes o profesionales no dedicados al desarrollo de código son los que más respondieron las encuestas. Cabe notar que para el año 2017 se creo la opción de "Desarrollador profesional", ocupación que puede contener muchas otras, explicando así el dato atípico para dicho año. 

7. ¿Cuáles son las tres principales ocupaciones por pais? **(Jhon)**


8. ¿Cuál es el principal rango salarial en cada país de los diez países con más respuestas?

    ![query7](https://juapatral.github.io/encuestas-stack-overflow/imagenes/query7.PNG)

    >Los países con más nivel de desarrollo económico como Estados Unidos, Reino Unido y Australia, presentan los salarios más grandes, mientras que países menos desarrollados como India y Polonía registran menores rangos salariales. 

9. ¿Cuál es el lenguaje con más crecimiento en las encuestas? **(Jhon)** ***(Muy importante para la predicción)***


10. ¿Cuál es el lenguaje mas popular en cada país de los diez países con mas programadores?

    ![query9](https://juapatral.github.io/encuestas-stack-overflow/imagenes/query9.PNG)

    >Se observa que JavaScript es el lenguaje preponderante en cada uno de los países, ocupando entre el 30 % y el 50 % del total de personas que respondieron la encuesta en cada país. Solo en "Europa: otro" se observa mayor participación por SQL.

11. ¿Cuáles son las cinco ocupaciones más populares de los programadores que dominan *Python* por encuesta? **(Jhon)**


12. ¿Cuáles son las tres ocupaciones con más programadores en los dos más altos rangos salariales?

    <center>

    ![query10_2](https://juapatral.github.io/encuestas-stack-overflow/imagenes/query10_2.PNG)

    </center>

    >Los encuestados con mayores rangos salariales se dedican a ser desarrolladores profesionales o dedicarse completamente al desarrollo en entorno Web.

13. ¿Cuál es el país más experto e inexperto programando por encuesta? **(Jhon)**


14. ¿Cuál es el número promedio de lenguajes que saben los programadores por encuesta?

    ![query12](https://juapatral.github.io/encuestas-stack-overflow/imagenes/query12.PNG)

    >Se observa que, en promedio, las personas que contestaron la encuesta dominan entre 3 y 8 lenguajes de programación. Este dato es muy variante entre los años debido a las opciones de respuesta que tienen las preguntas relacionadas con los lenguajes o tecnologías que dominan. Se observa que para 2013 y 2014, estas respuestas eran muy específicas (alrededor de 10 posibles), mientras que de 2015 en adelante se presentan más de 30 opciones.
    También es de interés conocer algunos datos puntuales acerca de *Colombia* y su participación en las encuestas de *Stack Overflow*
    <br>

15. ¿Cuánto es el salario más común en Colombia por encuesta?

    ![query13](https://juapatral.github.io/encuestas-stack-overflow/imagenes/query13.PNG)

    >Los encuestados solo respondieron que son de Colombia a partir del año 2014. Se observa que el rango salarial más común es Less than 20K, mientras que para el año 2017 se presentan salarios muy elevados, similar a la distribución a nivel global.


16. ¿Cuál es la ocupacion más popular en Colombia por encuesta? **(Jhon)**


17. ¿Cuáles son los tres lenguajes más usados en Colombia por encuesta?

    ![query15](https://juapatral.github.io/encuestas-stack-overflow/imagenes/query15.PNG)

    >En Colombia, el lenguaje de programación más popular es JavaScript y muestra una clara tendencia de crecimiento, mientras que SQL en su versión estandar o MySQL también ocupa una posición importante. 

[Regresar](#índice)

### **Representación visual de la bodega de datos**

A continuación se representa de forma gráfica la estructura de la bodega de datos a través de tres modelos: Cubo, Estrella y Malinowski.

***Modelo del cubo***

Para este modelo se toman tres dimensiones de la bodega de datos:

+ Año de la encuesta
+ País de la persona que responde la encuesta
+ Rango salarial de la persona que responde la encuesta

Como existen muchas categorías por dimensión, se toman los años 2011, 2012 y 2017, los países Colombia, México y Ecuador y los rangos salariales 0, 120K - 140K, More than 140K.

*<center> Imagen del modelo del cubo </center>*

<center>

![Imagen del cubo](https://juapatral.github.io/encuestas-stack-overflow/imagenes/modelo-cubo.PNG)

</center>

***Modelo de estrella***

Para este modelo se identifican todas las dimensiones (o columnas para como se constituyó la bodega de datos) y el hecho que en este caso son las mismas encuestas.

*<center> Imagen del modelo de estrella </center>*

<center>

![Imagen de estrella](https://juapatral.github.io/encuestas-stack-overflow/imagenes/Estrella.PNG)

</center>

***Modelo de Malinowski***

El modelo de Malinowski es similar al modelo estrella, cambiando la notación del hecho, las medidas del hecho y las dimensiones. Estas últimas se pueden desagregar a nivel de atributos, pero para este ejercicio no es posible debido a la forma como está constituida la bodega de datos.

*<center> Imagen del modelo de Malinowski </center>*

<center>

![Imagen de Malinowski](https://juapatral.github.io/encuestas-stack-overflow/imagenes/Malinowski.PNG)

</center>

[Regresar](#índice)

### **Pronóstico 2018**

Uno de los objetivos de este trabajo es pronosticar los resultados de la encuesta del año 2018 utilizando una tabla de la bodega de datos con al menos 50.000 registros. Para el desarrollo de este objetivo, se tomará el conjuto de datos completo y los siguientes supuestos de crecimiento.

*Nota:* los resultados de las encuestas 2018 y 2019 están disponibles en la [sitio oficial](https://insights.stackoverflow.com/survey) de Stack Overflow. Sin embargo, no se tomarán en cuenta dichas encuestas ni los resultados para establecer los pronósticos ni retroalimentarlos.

#### Supuestos

Lo primero que hay que pronosticar es cuántas personas podrían responder la encuesta para el año 2018. Para ello, se tomará una tendencia lineal de la cantidad de personas que respondieron la encuesta. 

| columna | criterio |
| --- | --- |
| country | 

[Regresar](#índice)