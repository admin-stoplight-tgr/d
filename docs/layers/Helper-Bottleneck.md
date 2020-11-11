# Helper Bottleneck

## Introducción.
Este helper permite la dosificación de las invocaciones a servicios externos que poseen una capacidad limitada o restringida en la ejecución de las solicitudes recibidas.

Para lograr este objetivo, esta librería se conecta a un repositorio donde estaán almacenadas las transacciones que deben ser envíadas y realiza un proceso de agrupación de dichas operaciones para que estás sean entregadas a una función Lambda específica.

Esta librería requiere de los siguientes elementos para su funcionamiento:

- Cola SQS, Contenedor de las transacciones a enviar al servicio externo
- Paramétros para la agrupación de las transacciones
- Una Lambda para entregar las transacciones agrupadas

## Casos de Uso.
- Envío de ordenes de pago a servidores onPremise.
- Consumo de servicios externos (APIs)
- Ejecución masiva de consultas a bases de datos.

## Cómo usar la librería

### Definición y envío de transacciones.
consumer.js
```js
const {Bottleneck} = require('tgr-sdk/clients/bottleneck')

let bottleneck = new Bottleneck(process.env.QUEUE,{groups: 60, size: 10, channel:{type:LAMBDA, arn:process.env.INVOKER_ARN})

module.exports.handler = bottleneck.getDequeueFunction()
```

### Recepción transacciones.

#### Serverless
serveless.yml
```yml
...

functions:
  producer:
    name: ...
    handler: consumer.handler
    environment:
      - QUEUE: !Ref my-queue

  consumer:
    name: ...
    handler: consumer.handler
    environment: 
      - INVOKER_ARN: !Ref invoker
    event:
      schedule:...

  invoker:
    name: ...
    handler: invoker.handler
```

## Permisos Lambda
No requiere permisos especiales.


