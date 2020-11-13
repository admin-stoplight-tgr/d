# Envio Dosificado

Este patrón define un mecanismo de procesamiento dosificado de objetos masivos. Dado un conjunto de objetos, el mecanismo permite procesar subconjuntos de objetos por minuto. Un caso de uso es invocar una API externa que puede atender una cantidad acotada de transacciones por minuto.

## Mecanismo

El mecanismo consiste en los siguientes pasos:
- Un productor va encolando objetos en una cola de dosificacion.
- Una lambda dosificadora es invocada cada 1 minuto y desencola mensajes. Esta lambda tiene 2 parametros: una cuota N que corresponde al máximo de invocaciones por minuto y una cantidad M que corresponde a la cantidad de objetos que envía en cada invocacion.
- Entonces, cada 1 minuto el dosificador invoca de manera asincrona una cantidad N de veces a una lambda receptora pasando en cada invocacion un numero M de mensajes.
- La lambda receptora recibe entonces M mensajes y los procesa.

## Implementación

El siguiente código corresponde al productor. Está implementado como una lambda que encola un gran numero de objetos.

```js
let SQS = require('aws-sdk/clients/sqs');
let sqs = new SQS({ region: 'us-east-1' });

let AMOUNT = 1000;

module.exports.handler = async(event) => {
    for (let i = 0; i < AMOUNT; ++i) {
        await sqs.sendMessage({
            MessageBody: JSON.stringify({
                key: `key-${Math.random()*AMOUNT}`,
                value: `value-${Math.random()*AMOUNT}`
            }),
            QueueUrl: process.env.QUEUE_URL
        }).promise()
    }
};
```

La siguiente lambda implementa al dosificador, el cual utiliza una estrategia de consumo de la cola que esta implementada en la libreria *tgr-sdk/clients/queue-workers*.

```js
const { DosificadorQueueWorker, DosificadorChannelTypes } = require('tgr-sdk/helpers/queue-workers')

const N = 60;
const M = 5

let worker = new DosificadorQueueWorker({
    queueURL: process.env.QUEUE_URL,
    perMinute: N,
    perInvocation: M,
    channel: {
        type: DosificadorChannelTypes.LAMBDA_CHANNEL,
        lambdaName: process.env.FUNCTION_NAME
    }
})

module.exports.handler = worker.dequeue
```

El siguiente codigo muestra la configuración usando el framework serverless.

```yml
service: envio-dosificado

custom: ${file(./config.yml)}

provider:
  name: aws
  runtime: nodejs12.x
  region: us-east-1
  stackName: ${self:custom.prefix}
  iamRoleStatements:
    - Effect: 'Allow'
      Action:
        - 'sqs:sendMessage'
        - 'sqs:receiveMessage'
        - 'sqs:deleteMessage'
      Resource:
        - 'Fn::GetAtt': [ColaEnvioDosificado, Arn]
    - Effect: 'Allow'
      Action:
        - 'lambda:InvokeFunction'
      Resource: arn:aws:lambda:${self:provider.region}:*:function:${self:custom.prefix}-receptor

package:
  exclude:
    - package-lock.json
    - package.json

functions:
  productor:
    name: ${self:custom.prefix}-productor
    handler: src/handlers/productor.handler
    environment:
      QUEUE_URL: !Ref ColaEnvioDosificado

  dosificador:
    name: ${self:custom.prefix}-dosificador
    handler: src/handlers/dosificador.handler
    layers:
     - ${ssm:/tgr/common/tgr-sdk-layer-arn}
    environment:
      QUEUE_URL: !Ref ColaEnvioDosificado
      FUNCTION_NAME: ${self:custom.prefix}-receptor
    #events:
    #  - schedule: "rate(1 minute)"

  receptor:
    name: ${self:custom.prefix}-receptor
    handler: src/handlers/receptor.handler

resources:
  Resources:
    ColaEnvioDosificado:
      Type: "AWS::SQS::Queue"
      Properties:
        QueueName: ${self:custom.prefix}-envio-dosificado

```

