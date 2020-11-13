---
tags: [servicio]
---

# Cliente Eventos

## Introducción

Tesorería requiere tener un registro de cada evento relevante que vayan ocurriendo en los procesos de negocio que manejan las aplicaciones. Para esto se ha definido una librería que registra los eventos relevantes que ocurren en cada aplicación. Por el momento esta librería está solo disponible para ser utilizada dentro de una lambda en nuestro entorno AWS.

## Cómo usar la librería.

La librería ha sido desplegada como un layer por lo cual sin mucho esfuerzo cualquier lambda que tenga los permisos necesarios podrá usarla (más abajo se explica cómo definir los permisos).

Para usarla desde una lambda lo único necesario es incluir la librería y luego llamar al método evento con los parámetros que corresponda. Los parámetros son:
evento: el tipo de evento para registrar en BigQuery
aplicacion: Nombre de la Aplicación
data: el contenido de la información  a enviar a BigQuery en formato JSON

A continuación se muestra el código de una lambda que registra un evento:

```javascript
const {evento} = require('tgr-sdk/clients/eventos')

let evento = "CargaArchivos"
let aplicacion = "carga-egresos"
let data = {"hola": "mundo"}

module.exports.handler = async () => {
   try {
       await evento({evento, aplicacion , data })
   } catch (e) {
       console.log(e.message)
   }
}
```

La librería asume la existencia de las siguientes variable de ambiente en el proyecto que requiere usar el layer:

```javascript
process.env.TGR_EVENTS_TOPIC
```

A continuación se muestra un ejemplo de archivo serverless.yml:

```yaml
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
     TGR_EVENTS_TOPIC: ${ssm:/tgr/common/eventos/topic}
```

## Permisos CodeBuild

Para que pueda hacer el deploy automático se requiere agregar el permiso a nivel de terraform en el rol de ejecución para codebuild en el módulo de deployment:

```javascript
statement {
    actions = [
        "ssm:GetParameters",
        "ssm:GetParameter"
    ]
    resources = [
        "arn:aws:ssm:*:*:parameter/tgr/common/*"
      ]
  }
```

## Permisos Lambda

Al usar la librería, internamente se hace una llamada a un topic SNS. Para que esto funcione el rol de ejecución de la lambda debe tener permisos para invocar el servicio SNS. La autorización se define en estos momentos a nivel de terraform. El rol de ejecución de la lambda tiene que tener el siguiente permiso:

```json
statement {
    sid = "ssm"
    actions = [
      "ssm:GetParameters",
      "ssm:GetParameter"
    ]
    resources = [
      "arn:aws:ssm:*:*:parameter/tgr/common/*"
    ]
}

statement {
 actions = [
   "SNS:Publish"
 ]
 resources = [
   "arn:aws:sns:us-east-1:${var.account}:tgr-${var.env}-api-events-to-gcp-topic"
 ]
}

```

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTA2MzM0NTUyN119
-->
