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
            notificationsPerMinute: 60
            notificationsSize: 10
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

  consumer:
    name: ...
    handler: consumer.handler
    events:
      - sns:
          arn: ${ssm:/tgr/common/bottleneck/sns/arn}
          topicName: ${ssm:/tgr/common/bottleneck/sns/topics/event}
          filterPolicy:
            EVENT_NAME: "ORDENES_PAGO"

```
consumer.js
```js

module.exports.handler = async (event) => {
  const record = event.Records[0];
   try {
      const group = record.Sns.Message;      
      ...
      foreach(element in group) {
        console.log(element) // un elemento del grupo
      }
      ...
       
   } catch (e) {
       console.log(e.message)
   }
}
```
## Permisos Lambda
No requiere permisos especiales.

## Cola Dead Letter
Esta cola almacenaráá las operaciones que por algúún motivo no hayan podido ser procesadas por el consumidor suscrito a SNS para un posterior procesamiento


