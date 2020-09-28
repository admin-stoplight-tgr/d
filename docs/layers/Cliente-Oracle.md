# Cliente Oracle
## Introducción


## Cómo usar la librería.

La librería ha sido desplegada como un layer de tal manera que cualquier lambda que tenga los permisos necesarios podrá usarla (más abajo se explica cómo definir los permisos).

Para usarla desde una lambda lo único necesario es incluir la librería y luego llamar al método notifica con los parámetros que corresponda. Los parámetros son:


A continuación se muestra el código de una lambda que ...:

```javascript
const {DSS, ORACLE} = require('tgr-sdk/clients/dss')

let dss = new DSS(ORACLE);

let sql = "select * from ...";
let params = [...];

module.exports.handler = async () => {
   try {
       await dss.query([{sql, params}])
   } catch (e) {
       console.log(e.message)
   }
}

```
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


