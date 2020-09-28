# Guia API

## Definiciones

### API
Application Programming Interface, es un conjunto de subrutinas, funciones, y procedimientos que ofrece cierta biblioteca para ser utilizado por otro software como una capa de abstracción.

### REST
Representational State Transfer, Se utiliza para describir cualquier interfaz entre sistemas que utilice directamente http para obtener datos o indicar la ejecución de operaciones sobre datos, en cualquier formato (XML, JSON, etc) sin las abstracciones adicionales de los protocolos basados en patrones de intercambio de mensajes como SOAP.


### OpenAPI
Es un estándar para definir contratos de Api. 

### URI
Es una cadena de caracteres que identifica los recursos de una red de forma unívoca.La diferencia respecto a un localizador de recursos uniforme (URL) es que estos últimos hacen referencia a recursos que, de forma general, pueden variar en el tiempo.

Normalmente estos recursos son accesibles en una red o sistema. Los URI pueden ser localizador de recursos uniforme (URL), uniform resource name (URN), o ambos.

Un URI consta de las siguientes partes:

* Esquema: nombre que se refiere a una especificación para asignar los identificadores, e.g. urn:, tag:, cid:. En algunos casos también identifica el protocolo de acceso al recurso, por ejemplo http:, mailto:, ftp:, etc.
* Autoridad: elemento jerárquico que identifica la autoridad de nombres (por ejemplo //www.example.com).
* Ruta: Información usualmente organizada en forma jerárquica, que identifica al recurso en el ámbito del esquema URI y la autoridad de nombres (e.g. /domains/example).
* Consulta: Información con estructura no jerárquica (usualmente pares "clave=valor") que identifica al recurso en el ámbito del esquema URI y la autoridad de nombres. El comienzo de este componente se indica mediante el carácter '?'.
* Fragmento: Permite identificar una parte del recurso principal, o vista de una representación del mismo. El comienzo de este componente se indica mediante el carácter '#'.

Aunque se acostumbra llamar URL a todas las direcciones web, URI es un identificador más completo y por eso es recomendado su uso en lugar de la expresión URL.

Un URI se diferencia de un URL en que permite incluir en la dirección una subdirección, determinada por el “fragmento”.


## Operaciones

Los verbos HTTP comprenden una parte importante para nuestra restricción de "interfaz uniforme" y nos proporciona la contraparte de la acción del recurso basado en sustantivos.  Los verbos HTTP primario o mas utilizados son POST, GET, PUT, PATCH y DELETE.  Estos corresponden a la operación de creación, lectura actualización y eliminación (CRUD).  Otros utilizados con menos frecuencia con OPTIONS y HEAD.

A continuación se muestra una tabla que resume los valores de retorno recomendados de los métodos HTTP primarios en combinación con los URI de recursos:


Verbo HTTP | CRUD | Coleccion Completa (ej: /contribuyentes) | Item Específico (ej: /contribuyentes/{id})
-----------|------|------------------------------------------|--------------------------------------------
 POST | Create | 201 (Created) | 404 (Not Found), 409 (Conflict) si el recurso ya existe
 GET | Read | 200 (OK), lista de contribuyentes. Se debe usar paginación, ordenamiento y filtrado para navegar una lista muy grande. | 200 (OK), un contribuyente. 404 (Not Found), si no fue encontrado o es un identificador invalido.
 PUT | Update/Replace | 405 (Method Not Allowed), a menos que desee actualizar / reemplazar cada recurso en toda la colección. | 200 (OK) o 204 (No Content). 404 (Not Found), si el identificador no fue encontrado o es invalido.
 PATCH | Update/Modify | 405 (Method Not Allowed), a menos que desee modificar la colección en sí. | 200 (OK) o 204 (No Content). 404 (Not Found), si el identificador no fue encontrado o es invalido.
 DELETE | Delete | 405 (Method Not Allowed), a menos que desee eliminar toda la colección, no es deseable. | 200 (OK). 404 (Not Found)), si el identificador no fue encontrado o es invalido.


## Requisitos Clave para una Api REST

* Debe ser amigable par ael desarrollados y de ser explorable a través de una barra de dirección de del navegador

* Debe ser simple, intuitivo y consistente para hacer que la adopción no solo sea fácil sino también agradable

* debe ser eficiente, manteniendo el equilibrio con los otros requisitos

## Diseño de una API Rest

1. Use el slash (/) para indicar que hay una relación jerárquica entre los recursos. no coloque más de 3 recursos por jerarquía.  Nunca deje un slash al final.



Recurso | URI
--------|-------
API Portal | https://api.tgr.cl/portal/v1 
Coleccion de Clientes  | https://api.tgr.cl/portal/v1/clientes 
Un cliente específico| https://api.tgr.cl/portal/v1/clientes/{idCliente}
Colección de cuentas | https://api.tgr.cl/portal/v1/clientes/{idClientes}/cuentas
 


2. Use guiones (-) para separar nombres largos en segmentos; evite usar underscores (_). Por ejemplo:
 https://api.path.com/gestion-clientes/v1/clientes
https://api.path.com/sistema-automatizado-egresos/v1
https://api.path.com/publicacion-remates/v1

3. Para los URI use letras minúsculas. Evite otros estilos de nomenclatura como CamelCase o UPPERCASE.

4. No incluya extensiones de archivo. Si necesita indicar o resaltar el tipo de archivo, utilice el header Content-Type, de modo que se pueda determinar cómo procesar el contenido

5. Coloque los conceptos de negocio siempre en español con estilo camelCase (idPersona, digitoVerificador, etc.). Para los metadatos (message, error, code, data, etc.) debe usar idioma inglés.

6. El número de versión (v1) debe mantenerse igual, aunque se modifique la API, a no ser que el cambio provoque pérdida de compatibilidad con clientes anteriores. Ejemplo:


v1 | v1 | v1 | v1 | v2 | v3
---|----|----|----|----|----
Se crea la versión inicial | Se añade un nuevo recurso o método | Parámetros de entrada adicionales (opcionales) | Más info. en la invocación de un recurso | Se modifica un valor de entrada o salida | Se borra un recurso


En los casos de evolución hacia una nueva versión, convivirán por un período de tiempo las versiones existentes y se notificará a los clientes para que sean actualizados. Por la naturaleza y la complejidad de este cambio de versión, no ocurrirán frecuentemente.

7. Diseñe la API pensando en los consumidores y no en la estructura de sus datos.


## Nomenclatura de recursos REST

Los recursos mapean un conjunto de entidades por tanto, las colecciones se identifican en plural y siempre con sustantivos. ¿Por qué? Los sustantivos se asocian con objetos que tienen propiedades a diferencia de los verbos que representan acciones o los adjetivos que describen características
 
Para incrementar la claridad y el análisis, se dividirán los recursos en las categorías colección y documento. Siempre coloque un recurso en una de estas categorías para que sea más fácil aplicar las convenciones de nombrado. Además, evite diseñar recursos que puedan caer en más de una categoría o ruta.
Categoría colección
Las colecciones serían los conjuntos de recursos o documentos. Use sustantivos en plural para nombrar las colecciones. Ejemplos:

* https://api.path.com/gestion-clientes/v1/clientes
* https://api.path.com/sistema-automatizado-egresos/v1/egresos
* https://api.path.com/publicacion-remates/v1/publicaciones

## Categoría documento

Un documento es un concepto similar a una instancia de una clase o un registro de una base de datos. En REST, representa un único recurso dentro de una colección que contiene atributos. Use identificadores para los documentos, generalmente las llaves del negocio o llave subrogada. Este sustantivo sólo se utiliza en el diseño, porque en tiempo de ejecución, el valor dependerá de los datos. Ejemplos:
* https://api.path.com/gestion-clientes/v1/clientes/11111111-1
* https://api.path.com/sistema-automatizado-egresos/v1/egresos/cod9x4c
* https://api.path.com/publicacion-remates/v1/publicaciones/SDaHlafIq

### Nota sobre los identificadores:

Los identificadores de documento deben ser únicos en la colección a la que pertenecen. Se recomienda que sus valores sean llaves subrogadas, llaves únicas o Universally Unique IDentifier (uuid).


## Gestión de CRUD sobre recursos
Para indicar que una determinada función se va a realizar sobre un recurso, no es necesario incluir esta en el URI. Los métodos HTTP Request (GET, POST, PUT, DELETE) deben ser usados para indicar cuál función es realizada. A continuación se incluye una tabla resumen con las operaciones por categorías y la acción del CRUD realizada.

verbo | colección | documento
------|-----------|-----------
POST | Crear nuevo documento | No válido
GET | Obtener lista de documentos | Obtener el documento
PUT | No válido | Actualizar el documento
DELETE | No válido | Eliminar el documento

En los documentos, no se utiliza POST porque los documentos se crean a través de las colecciones. En el caso de las colecciones no es válido PUT o DELETE porque las funciones para actualizar o eliminar completamente una colección no deberían estar expuestas a través de una API pública.

## Datos de entrada
En la tabla siguiente se representan los datos de entrada según la solicitud

|        | Coleccion | Documento 
|--------|-----------|-----------
| POST   | {<br> "data" : {<br>      //atributos<br>  }<br>} | No valido 
| GET    | No aplica | No aplica 
| PUT    | No valido | {<br>  "data" : {<br>          key : value,<br>          //atributos<br>   }<br>} 
| DELETE | No valido | No valido


## Datos de salida

Para facilitar la comprensión de las APIs y los datos que devuelven, se incorporan los mensajes de respuesta y HTTP Status Code esperados para las peticiones de recursos.
* 200: Se procesó correctamente la solicitud.
* 201: Se creó el objeto correctamente.
* 400: Ocurrió un error desde el cliente. Error de validación. El mensaje debe ir dirigido a resolver el error por el cliente.
* 404: Se intenta acceder a un recurso que no existe.
* 500: Ocurrió un error en el servidor. Error en el servidor, debe asociarse con uno de los códigos proporcionados.
 
En la tabla siguiente se representan los datos de salida según la solicitud. No se incluyen los campos
 
 
|  |Recurso | Código HTTP | Body
|--|--------|-------------|------
GET | colección | 200 | {<br>  "total": "",<br>  "page": "",<br>  "limit": "",<br>  "data": [<br>	{...},<br>	{...}<br>  ]<br>}<br>
POST | colección | 201 | empty 
GET | documento | 200 | {<br>  "data": {}<br>}<br>
PUT, DELETE | documento | 200 | empty
GET, PUT, DELETE | documento | 404 | {<br>  "errors": [<br>	{<br>  	"message": "Recurso no existe"<br>	},<br>	{}<br>  ]<br>}<br>
ANY | recurso | 400 | {<br>  "errors": [<br>	{<br>  	"message": "abcdef"<br>	},<br>	{}<br>  ]<br>}<br>
ANY | recurso | 500 | {<br>  "errors": [<br>	{<br>  	"message": "abcdef",<br>  	"code": 500xxx,<br>  	"id":"uuid",1]<br>	},<br>	{}<br>  ]<br>}<br>

## Formatos

### Fecha 

Las fechas deben almacenarse con el valor correspondiente a UTC/GMT +0/Zulu Time.  El formato para almacenarlas debe ser ISO 8601 o timestamp en milisegundos.  Así se evitan problemas de interpretación antes cambios de hora y se sigue un estándar internacional.


`timestamp: "1554385220000"


```json
{
    "iso8601" : "2019-04-04T13:40:20Z",
    "format" : "yyyy-MM-ddTHH:mm:ssZ" 
}
```


### Ordenar

Para realizar el ordenamiento se debe utilizar el operador "?" y la palabra clave "sort", ingresando el campo por el cual se desar ordenar.  El ordenamiento por defecto debe ser en orden ascendente, si se desea que sea de forma descendente se debe especificar.

#### Ejemplo

Obteniendo clientes ordenado por rut:

* Solicitud normal -> https://api.path.com/gestion-clientes/v1/clientes​?sort=rut

* Solicitud descendente -> https://api.path.com/gestion-clientes/v1/clientes​?sort=rut,desc

### Paginar
se obtiene la página 1, con 10 elementos. Para la página n, el primer valor mostrado es el elemento (n*limit+1). Por omisión se asume page = 1, limit = 10

#### Ejemplo

https://api.path.com/sistema-automatizado-egresos/v1/egresos​?page=1&limit=10

### Filtrado

Realizar filtros por campo especificado.

#### Ejemplo 
* Se filtra por el campo fecha -> https://api.path.com/publicacion-remates/v1/publicaciones​?fecha=2019-03-05 

### Selección

Se devuelven solamente los campos especificados.

#### Ejemplo
https://api.path.com/publicacion-remates/v1/publicaciones​?fields=fecha,juzgado 

### Seguridad

La seguridad será establecida utilizando el protocolo para autorización OAuth2, con el ​grant type c​lient_credentials en los casos de clientes confidenciales. En otros casos, deben evaluarse modos alternativos de resguardar las APIs. Los datos asociados a la seguridad se deberán enviar a través del Header de la solicitud (request). Los campos que se envían son:

* grant_type, es el tipo de concesión usado para la interacción con el servidor de Token
* scope, significa el alcance que tendrá el token
* client_id, es el Oauth Client que fue creado a nivel de DP
* client_secret, clave asignada al Oauth Client

#### Ejemplo

* grant_type : client_credentials
* scope : /nombre-api/v1/*
* client_id : ​OauthCuentaClient
* client_secret :  ​TGR.passw0rd

Donde
​  ​
* client_credentials :​ ​es un valor constante para el tipo de autenticación que se maneja por el momento.
* /nombre-api/v1/* : ​abarca el primer nivel del URI del API (donde se especifica el nombre de la API) y el segundo nivel del URI (donde se especifica la versión correspondiente del servicio).
* OauthCuentaClient : ​Es el Oauth Client Profile que se configura en el DataPower. 
* TGR.passw0rd : ​Contraseña asignada al Oauth Client Profile en el DataPower.



