<br/>

<img src="https://uploads-ssl.webflow.com/614b1fe22fa8b90ef41aeffe/6265cb48f9496b1cefc9ab75_logotipo-mbit-39.png" width="200px" align="right" CLASS="TextWrap" style="background-color:#2a3f3f;">

# <font color="#2a3f3f" size=6>Proyecto de Consolidación 01</font>

## <font color="#2a3f3f" size=4>Creación de una aplicación apificada basada en microservicios</font>


<div style="text-align: right">
<!--<font color="#2a3f3f" size=3>Javier Cózar - javier.cozar@mbitschool.com</font><br>-->
<font color="#2a3f3f" size=3>Alberto Martos Rueda - alberto.martos@alu.mbitschool.com</font><br>
<font color="#2a3f3f" size=3>Máster en Data Engineering </font><br>
</div>

## <font color="#2a3f3f" size=4>Introducción</font>

En este proyecto se ha creado una aplicación basada en microservicios, utilizando la tecnología Docker para ejecutar cada microservicio en un entorno reproducible.

La aplicación permite subir fotos, las cuales se etiquetan automáticamente, y se almacenan en una carpeta además de en una base de datos donde existe toda esta información (paths a imágenes y etiquetas asignadas).
Con esta información, la aplicación permite la busqueda de imágenes a través de unas etiquetas concretas.

Existen los siguientes microservicios:

- **Base de datos:** Base de datos MySQL 8.0.
- **API:** implementada en Flask y servida mediante waitress.

<img src="https://raw.githubusercontent.com/AlbertoMR17/01-Aplicacion-apificada-basada-en-microservicios/main/diagram.png" width="2000px" style="float: right; margin-left: 100px; margin-bottom: 100px;">

## <font color="#2a3f3f" size=4>Base de datos</font>

Se ha usado una base de datos MySQL versión 8.0 haciendo uso de la imagen de dockerhub. La base de datos se compone de dos tablas

-**Tabla *pictures*** que contiene una fila por cada imagen almacenada en el sistema. Tiene las siguientes columnas:

- `id`: Columna de strings (36 caracteres) que se corresponden con un uuid único para cada imagen. Es la **Primary Key**.
- `path`: String que identifica el path donde está almacenada la imagen.
- `date`: String que identifica la fecha en la que se creó la imagen, en formato `YYYY-MM-DD HH:MM:SS`.


-**Tabla *tags*** que contiene las tags asociadas a cada imagen. Tiene las siguientes columnas:

- `tag`: Nombre de la tag asociada a la imagen (como mucho 32 caracteres).
- `picture_id`: uuid de la imagen que está asociada a la tag. Es una **Foreign Key**.
- `confidence`: Nivel de confianza de la tag asociada a la imagen.
- `date`: String que identifica la fecha en la que se creó la imagen, en formato `YYYY-MM-DD HH:MM:SS`.

- La **Primary Key** está compuesta por las columnas `tag` y `picture_id`

## <font color="#2a3f3f" size=4>API</font>

Se ha implementado una API en Flask. Se ha hecho uso de la imagen de python en [dockerhub](https://hub.docker.com/_/python) con la tag `3.11`.
Se ha construido nuestro propio `Dockerfile` para crear la imagen que contenga nuestro código custom que, además de implementar la funcionalidad necesaria con Flask, sirva la API utilizando `waitress` en el puerto 80 del contenedor.
Se proporciona el archivo `requirements.txt` con los paquetes necesarios para implementar la API.

La API hace uso de tres endpoints:


<!--#### POST image-->
#### <font color="#2a3f3f" size=2>POST image</font>

Este endpoint espera como input un body que es un `json` con un campo `data` que es la imagen codificada en base64.
También se especifica un _query parameter_ llamado `min_confidence`, que sirve para exigir ese valor de certeza a las etiquetas generadas para la imagen.
Su valor por defecto es `80`.
Una vez se recibe se utiliza un [servicio cloud mediante una API](https://imagga.com/) para extraer tags a partir de esta imagen. Esta API requiere que le pasemos la imagen como una URL pública, por lo que se usa en primer lugar otro [servicio cloud mediante una API](https://docs.imagekit.io/) para subir temporalmente esta imagen a la nube.
Una vez recibida la respuesta, se almacena la información en la base de datos:

  - En la tabla `pictures` se almacena el path de la imagen que tiene un id asociado. También se almacena la fecha en la que se ha introducido como un string en formato `YYYY-MM-DD HH:MM:SS`.
  - En la tabla `tags` se almacen una fila por cada tag asociado a la imagen, donde se tienen los campos `tag`, `picture_id` (foreign key) y la fecha en la que se ha introducido la imagen como un string en formato `YYYY-MM-DD HH:MM:SS`. También se almacena qué `confidence` tiene esta tag para la imagen.

La **respuesta** de este endpoint debe ser un **json** con los siguientes campos:

- `id`: identificador de la imagen
- `size`: tamaño de la imagen en KB
- `date`: fecha en la que se registró la imagen, en formato `YYYY-MM-DD HH:MM:SS`
- `tags`: lista de objetos identificando las tags asociadas a la imágen. Cada objeto tendrá el siguiente formato:
    - `tag`: nombre de la tag
    - `confidence`: confianza con la que la etiqueta está asociada a la imagen
- `data`: imagen como string codificado en base64
-`imagekit.io`: ir a dashboard, crearnos una cuenta, y al loguearnos ir a **Developer options**. Ahí veremos nuestro `URL-endpoint`, y unas credenciales por defecto ya creadas (`Public Key` y `Private Key`). Aunque se puede utilizar `requests`, vamos a usar la librería para Python que nos proporciona la plataforma que simplifica el proceso de autenticación. Para ello debemos instalar la librería `imagekitio`


<!--#### GET images-->
#### <font color="#2a3f3f" size=2>GET images</font>

Este endpoint sirve para obtener una lista de imágenes que cumplan un filtro. Se proporcionará mediante _query parameters_ la siguiente información:

- `min_date`/`max_date`: opcionalmente se puede indicar una fecha mínima y máxima, en formato `YYYY-MM-DD HH:MM:SS`, para obtener imágenes cuya fecha de registro esté entre ambos valores. Si no se proporciona `min_date` no se filtrará ninguna fecha inferiormente. Si no se proporciona `max_date` no se filtrará ninguna fecha superiormente.

- `tags`: optionalmente se puede indicar una lista de tags. Las imágenes devueltas serán aquellas que incluyan **todas** las tags indicadas. El formato de este campo será un string donde las tags estarán separadas por comas, por ejemplo `"tag1,tag2,tag3"`. Si no se proporciona ninguna tag, no se devolverá ninguna imagen.

La **respuesta** del endpoint será una lista de objetos imagen con los siguientes campos:

- `id`: identificador de la imagen
- `size`: tamaño de la imagen en KB
- `date`: fecha en la que se registró la imagen, en formato `YYYY-MM-DD HH:MM:SS`
- `tags`: lista de objetos identificando las tags asociadas a la imágen. Cada objeto tendrá el siguiente formato:
    - `tag`: nombre de la tag
    - `confidence`: confianza con la que la etiqueta está asociada a la imagen

<!--#### GET image-->
#### <font color="#2a3f3f" size=2>GET image</font>

Este endpoint sirve para descargarse una imágen y sus propiedades. Se proporcionará mediante _path parameter_ el id de la imagen y se la **respuesta** será un json con los siguientes campos:

- `id`: identificador de la imagen
- `size`: tamaño de la imagen en KB
- `date`: fecha en la que se registró la imagen, en formato `YYYY-MM-DD HH:MM:SS`
- `tags`: lista de objetos identificando las tags asociadas a la imágen. Cada objeto tendrá el siguiente formato:
    - `tag`: nombre de la tag
    - `confidence`: confianza con la que la etiqueta está asociada a la imagen
- `data`: imagen como string codificado en base64
