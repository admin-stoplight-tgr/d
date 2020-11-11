# Cliente Bottleneck

## Introducción
Esta librería permite simplificar el proceso de envío de solicitudes (requests) a APIS externas por parte de los servicios de la TGR. Especialmente cuando se requiere realizar invocaciones a servicios que tienen una cantidad acotada de transacciones que pueden atender por minuto.

El funcionamiento se asemeja a un gran contenedor de transacciones, las que son despachadas en grupos pequeños hasta que se consumen todas las operaciones.


Los siguientes pasos, resumen la operación en general:
- Se inicializa la instancia indicando: un nombre descriptivo y una máximo de transacciones por minuto.
- Se envían las transacciones a la librería.
- Se suscribe el API de consumo al Topic SNS donde recibirá las transacciones a envíar al servicio externo.



## Cómo usar la librería

### Definición y envío de transacciones.

producer.js
```js
const {Bottleneck} = require('tgr-sdk/clients/bottleneck')

let bottleneck;

module.exports.handler = async (orden) => {
   try {
        if(bottleneck === undefined) {
          bottleneck = new Bottleneck(process.env.EVENT_NAME, process.env.BOTTLENECK_ENDPOINT)

          await bottleneck.configure({
            notification: {
              perMinute: 60,
              size: 10,
              channel: {
                type: 'LAMBDA',
                arn: process.env.CONSUMER_LAMBDA_ARN
              }
            }
          })
       }
       ...
       await bottleneck.submitElement(orden);
       ...
   } catch (e) {
       console.log(e.message)
   }
}
```

consumer.js
```js
module.exports.handler = async (group) => {
  ...
  foreach(element in group) {
    console.log(element) // un elemento del grupo
  }
  ...
}
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
      - BOTTLENECK_ENDPOINT: ${ssm:/tgr/common/bottleneck/api/endpoint}
      - EVENT_NAME: "ORDENES_PAGO"
      - CONSUMER_LAMBDA_ARN: !Ref consumer

  consumer:
    name: ...
    handler: consumer.handler
```


## Permisos Lambda
No requiere permisos especiales.

## Cola Dead Letter
Esta cola almacenaráá las operaciones que por algúún motivo no hayan podido ser procesadas por el consumidor suscrito a SNS para un posterior procesamiento


