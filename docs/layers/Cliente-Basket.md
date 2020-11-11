# Cliente Baskets

## Introducción
Esta librería permite simplificar el proceso de envío de solicitudes (requests) a APIS externas por parte de los servicios de la TGR. Especialmente cuando se requiere realizar invocaciones a servicios que tienen una cantidad acotada de transacciones que pueden atender por minuto.

El funcionamiento se asemeja a un gran contenedor de transacciones, las que son despachadas en grupos pequeños hasta que se consumen todas las operaciones.


Los siguientes pasos, resumen la operación en general:
- Se inicializa la instancia indicando: un nombre descriptivo y una máximo de transacciones por minuto.
- Se envían las transacciones a la librería.
- Se suscribe el API de consumo al Topic SNS donde recibirá las transacciones a envíar al servicio externo.



## Cómo usar la librería

### Definición y envío de transacciones.

```json
const {basket} = require('tgr-sdk/clients/baskets')

const eventType = "ConsultaSRCeI"
const quota = 100

module.exports.handler = async () => {
   try {
       const b = new basket(eventType, quota);
       ...
       const consultas = foo();  
       b.process(consultas);
   } catch (e) {
       console.log(e.message)
   }
}

```

### Recepción transacciones.

#### Serverless
Serveless.yml
```json
...

functions:
  suscripcion:
    name: tgr-lab-suscripcionClienteBasket
    handler: consumer.handler
    deadLetter:
      sqs:  tgr-lab-suscripcionClienteBasket-DLQ      
    timeout: 30
    events:
      - sns:
          arn: !Ref TopicBasket
          topicName: TGR-#{custom::env}-TOPIC-BASKET-API
          filterPolicy:
            eventType: "ConsultaSRCeI"

```
consumer.js
```json
const axios = require('axios');
const qs = require('qs');

module.exports.handler = async (event) => {
  const msj = event.Records[0];
   try {
      const obj = mjs.Sns.Message;      
      ...
      const config = {
            method: 'POST',
            url: TOKEN_URL,
            headers: {HEADERS},
            data: qs.stringify(obj)
        }
      const res = await axios(config);
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


