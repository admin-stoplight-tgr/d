# Cliente Oracle
## Introducción


## Cómo usar la librería.

La librería ha sido desplegada como un layer de tal manera que cualquier lambda que tenga los permisos necesarios podrá usarla (más abajo se explica cómo definir los permisos).

Para usarla desde una lambda lo único necesario es incluir la librería y luego llamar al método notifica con los parámetros que corresponda. Los parámetros son:


A continuación se muestra el código de una lambda que realiza 2 consultas a la base de datos y una invocación de un package.

```javascript
const {OracleProvider, OracleTypes} = require('tgr-sdk/clients/dss')
let provider = new OracleProvider();

module.exports.handler = async () => {
   let params = [
        {
            "sql": "select * from sae.sae_personas where rut = #rut1 or rut = #rut2",
            "params": [
                {"parameterType": "IN", "type": OracleTypes.NUMERIC, "name": "#rut1", "value": 11222333},
                {"parameterType": "IN", "type": OracleTypes.NUMERIC, "name": "#rut2", "value": 33222111}
            ]
        },
        {
            "sql": "select * from sae.sae_personas where rut = #rut1 or rut = #rut2",
            "params": [
                {"parameterType": "IN", "type": OracleTypes.NUMERIC, "name": "#rut1", "value": 44555666},
                {"parameterType": "IN", "type": OracleTypes.NUMERIC, "name": "#rut2", "value": 77888999}
            ]
        },
        {
            "sql": "AWS_CLIENTE.PKG_PER_CLIENTES_AWS.Get_Persona( #rut, #dv, #idPersona )",
            "params": [
                {"parameterType": "OUT", "type": OracleTypes.NUMERIC, "name": "#idPersona"},
                {"parameterType": "IN", "type": OracleTypes.NUMERIC, "name": "#rut", "value": 99888777},
                {"parameterType": "IN", "type": OracleTypes.VARCHAR, "name": "#dv", "value": "K"}
            ]
        }
    ]
    
    try {
      let resultado = await provider.call(params);
        
      console.log('Resultado primer select:', resultado[0])
      /* 
      Resultado primer select: {
          resultset: [
              {
                  ID_TIPO_CONTRIBUYENTE: 2,
                  ID_PERSONA: ...
              },
              {
                  ID_TIPO_CONTRIBUYENTE: 1,
                  ID_PERSONA: ...
              }
          ]
      }
      */
      
      console.log('Resultado segundo select:', resultado[1])
      /* 
      Resultado segundo select: {
          resultset: [
              {
                  ID_TIPO_CONTRIBUYENTE: 2,
                  ID_PERSONA: ...
              },
              {
                  ID_TIPO_CONTRIBUYENTE: 1,
                  ID_PERSONA: ...
              }
          ]
      }
      */
      
      console.log('Resultado invocacion package:', resultado[2])
      /* 
      Resultado invocacion package: { 
          '#idPersona': 4978198 
        } 
      */
  
  } catch (e) {
      console.error(e.message)
  }
}
```


La librería asume la existencia de la variable de ambiente:

`process.env.DSS_TIERRA_ORACLE_LAMBDA`

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
     DSS_TIERRA_ORACLE_LAMBDA: ${ssm:/tgr/common/dss/tierra/oracle/lambda}
```


## Permisos Lambda

Al usar la librería, internamente se hace una llamada a una lambda DSS que implementa la invocación a Oracle. Para que esto funcione el rol de ejecución de la lambda debe tener permisos para invocar dicha lambda. La autorización se define en estos momentos a nivel de terraform. El rol de la lambda tiene que tener el siguiente permiso:

```json
statement {
 actions = [
   "lambda:InvokeFunction"
 ]
 resources = [
   "arn:aws:lambda:us-east-1:186180787863:function:${var.dss_tierra_oracle_lambda}"
 ]
}
```

El valor de *dss_tierra_oracle_lambda* se obtiene desde el parámetro SSM:

```json
data "aws_ssm_parameter" "dss_tierra_oracle_lambda" {
  name = "/tgr/common/dss/tierra/oracle/lambda"
}
```

