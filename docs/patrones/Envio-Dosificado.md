# Envio Dosificado

Este patrón define un mecanismo de procesamiento dosificado de objetos masivos. Dado un conjunto de objetos, el mecanismo permite procesar subconjuntos de objetos por minuto. Un caso de uso es invocar una API externa que tiene puede atender una cantidad acotada de transacciones por minuto.

## Mecanismo

El mecanismo consiste en los siguientes pasos:
- Un productor encola todos los objetos en una cola de dosificacion.
- Una lambda dosificadora es invocada cada 1 minuto y desencola mensajes desde la cola. Esta lambda tiene 2 parametros: una cuota N que corresponde al máximo de invocaciones por minuto y una cantidad M que corresponde a la cantidad de objetos que envía en cada invocacion.
- Entonces, cada 1 minuto el dosificador invoca de manera asincrona una cantidad N de veces a una lambda receptora pasando en cada invocacion un numero M de mensajes.
- La lambda receptora recibe entonces M mensajes y los procesa.

## Implementación

El siguiente código corresponde al productor. Está implementado como una lambda que encola un gran numero de objetos.

```js
let SQS = require('aws-sdk/clients/sqs');
let sqs = new SQS({region: 'us-east-1'});

let handler = async (event) => {
    let objetos = ...;

    for (const objeto of objetos) {
        await sqs.sendMessage({
            MessageBody: JSON.stringify(objeto),
            QueueUrl: process.env.QUEUE_URL
        }).promise()
    }
};

module.exports = {
    handler
};
```

La siguiente lambda implementa al dosificador, el cual utiliza una estrategia de consumo de la cola que esta implementada en la libreria *tgr-sdk/clients/queue-workers*.

```js
const {QueueWorkers} = require('tgr-sdk/helpers/queue-workers')

let N = 60;
let M = 5
let channel = {type: QueueWorkers.LAMBDA_CHANNEL, arn: process.env.INVOKER_ARN}

let workers = new QueueWorkers(process.env.QUEUE_URL)
module.exports.handler = bottleneck.getDosificadorFunction({perMinute: N, perInvocation: M, channel});
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
      Resource: arn:aws:lambda:${self:provider.region}:*:function:${self:custom.prefix}-invoker

package:
  exclude:
    - package-lock.json
    - package.json

functions:
  producer:
    name: ${self:custom.prefix}-producer
    handler: src/handlers/producer.handler
    environment:
      QUEUE_URL: !Ref ColaEnvioDosificado

  consumer:
    name: ${self:custom.prefix}-consumer
    handler: src/handlers/consumer.handler
    timeout: 30
    environment:
      QUEUE_URL: !Ref ColaEnvioDosificado
      INVOKEE_NAME: ${self:custom.prefix}-invokee
    #events:
    #  - schedule: "rate(1 minute)"

  invokee:
    name: ${self:custom.prefix}-invoker
    handler: src/handlers/invoker.handler
    timeout: 30

resources:
  Resources:
    ColaEnvioDosificado:
      Type: "AWS::SQS::Queue"
      Properties:
        QueueName: ${self:custom.prefix}-envio-dosificado

```

