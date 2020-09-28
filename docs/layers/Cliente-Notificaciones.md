# Cliente Notificaciones
## Introducción

Tesorería quiere poder notificar a sus contribuyentes cuando algo relevante para ellos ocurre, por ejemplo, informarle cuando TGR les ha enviado un depósito o cuando hay alguna notificación legal. Con este propósito se ha definido una librería que encapsula el envío de una notificación. Por el momento esta librería está solo disponible para ser utilizada dentro de una lambda en nuestro entorno AWS.

## Cómo usar la librería.

La librería ha sido desplegada como un layer por lo cual sin mucho esfuerzo cualquier lambda que tenga los permisos necesarios podrá usarla (más abajo se explica cómo definir los permisos).

Para usarla desde una lambda lo único necesario es incluir la librería y luego llamar al método notifica con los parámetros que corresponda. Los parámetros son:

* rutDestinatario: el rut del contribuyente que es notificado
* asunto: Título de la notificación
* contenido: Información muy breve de lo que se notifica. Idealmente acompañar un link donde se pueda encontrar más información.

A continuación se muestra el código de una lambda que notifica a un rut 11222333-K:

```javascript
const {notifica} = require('tgr-sdk/clients/notificaciones')

let rutDestinatario = "11222333-K"
let asunto = "Usted tiene un depósito"
let contenido = "Tesorería le ha depositado la suma de $1000. Mas detalles en <a>https://www.tgr.cl/…</a>"

module.exports.handler = async () => {
   try {
       await notifica({rutDestinatario, asunto, contenido})
   } catch (e) {
       console.log(e.message)
   }
}

```

La librería soporta múltiples formatos de rut, los siguientes son ruts válidos: 11222333-K, 11222333-k, 11.222.333-K

La librería asume la existencia de la variable de ambiente:

`process.env.NOTIFICACIONES_ENDPOINT`

Esta variable tiene que ser definida dentro de los parámetros que se incluyen en la definición de la lambda.  Esto se hace en a través del archivo serverless.yml. Además en la definición de la lambda se tiene que incluir el layer donde está definida la librería. A continuación se muestra un ejemplo de archivo serverless.yml:

```javascript
service: mi-servicio

provider:
 name: aws
 runtime: nodejs12.x
 
functions:
 miFuncion:
   handler: index.handler
   layers:
     - ${ssm:/tgr/common/tgr-sdk-layer-arn}
   environment:
     NOTIFICACIONES_ENDPOINT: ${ssm:/tgr/common/notificaciones/endpoint}

```


## Permisos Lambda

Al usar la librería, internamente se hace una llamada al endpoint de la aplicación de notificaciones usando credenciales IAM de AWS. Para que esto funcione el rol de ejecución de la lambda debe tener permisos para invocar dicho endpoint. La autorización se define en estos momentos a nivel de terraform. El rol de la lambda tiene que tener el siguiente permiso:

```json
statement {
 actions = [
   "appsync:GraphQL"
 ]
 resources = [
   "arn:aws:appsync:us-east-1:*:apis/${graphql-api-id}/*"
 ]
}
```

El valor de graphql-api-id se obtiene desde el parámetro SSM:

`/tgr/common/notificaciones/graphql-api-id`


