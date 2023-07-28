# Assessment-f2x
Propuesta de solución al ejercicio - f2x-v2.0.0

# Contexto

La solución propuesta provee funcionalidades para el registro de usuarios (onboarding), registro de vehículos (enrolamiento), transacciones de pago de servicios para vehículos (parqueadero, peajes), consulta de saldos y movimientos (balances), registro de medios de pago y la posibilidad de realizar recargas a la cuenta FlyPass.

# Contenedores

![image](https://github.com/4dagio/assessment-f2x/assets/3275936/27b97f0b-35d2-4fac-aece-d469eaaab426)

La solución consiste en los siguientes contenedores:
- **[X] Frontend:** Interfaz de usuario para realizar las operaciones (staff y usuarios)
- **[X] Servicio de Onboarding:** Registra nuevos usuarios en el sistema.
- **[X] Servicio de Enrolamiento:** Permite a los usuarios registrar y asociar vehículos en su cuenta.
- **[X] Servicio de Transacciones:** Pasarela de pagos para servicios de vehículos.
- **[X] Servicio de Balances:** Permite consultar el saldo de la cuenta y los movimientos realizados.
- **[X] Servicio de Medios de Pago:** Permite registrar una tarjeta de crédito o realizar una recarga en la cuenta.
- **[X] Servicio de Recargas:** Permite enviar dinero a la cuenta del usuario.
- **[X] Broker de Eventos:** Orquesta la comunicación entre servicios y maneja eventos asíncronos.
- **[X] Servicio de Autenticación**: Maneja la autenticación y autorización de usuarios.

# Componentes

![image](https://github.com/4dagio/assessment-f2x/assets/3275936/746e3efb-6727-41c7-b499-0bffb12592e5)
# Descripción de la solución

### Onboarding : Sing-up/sing-in
- El usuario inicia sesión en la aplicación web - (Sin no existe puede registrarse con correo/contraseña o  a través de Oauth2.0 )
- El usuario se crea en el grupo de usuarios de Cognito y los atributos de usuario se rellenan basándose en las asignaciones de atributos (nombres, datos personales y data relevante).
- Tras una autenticación correcta, Cognito recibirá un token de acceso JWT .
- Se devuelve un token JWT de Cognito a la aplicación.
- El front  realiza una llamada al API Gateway.
- La aplicación extrae el token de identificación del JWT y pasa el token en la cabecera de autorización de la API.
- La pasarela de API invoca el autorizador Lambda personalizado y pasa el token para su posterior validación (servicios internos).
- 
### Medios de pago:
El dominio de **Registrar Medios de Pago** se encargaría de manejar todas las operaciones relacionadas con la adición y gestión de medios de pago de un usuario en el sistema.
- Agregar un nuevo medio de pago: Los usuarios deben poder registrar nuevos medios de pago, como tarjetas de crédito o débito.
- Ver medios de pago existentes: Los usuarios deben poder ver los medios de pago que han registrado anteriormente.
- Eliminar un medio de pago: Los usuarios deben poder eliminar un medio de pago que ya no quieren usar.
- Actualizar un medio de pago: Los usuarios deben poder actualizar la información de un medio de pago, como la fecha de vencimiento de una tarjeta de crédito.

Objeto de dominio

```javascript
{
    "userId": "123",  // El ID del usuario al que pertenece este medio de pago
    "paymentMethodId": "456",  // Un ID único para este medio de pago
    "type": "credit card",  // El tipo de medio de pago (por ejemplo, "credit card", "debit card", "bank account", etc.)
    "cardNumber": "**** **** **** 1234",  // El número de la tarjeta, probablemente necesitarías guardar solo los últimos 4 dígitos por razones de seguridad
    "expiryDate": "12/2025",  // La fecha de vencimiento de la tarjeta
    "cardholderName": "John Doe",  // El nombre del titular de la tarjeta
    "isDefault": true  // Un booleano que indica si este es el medio de pago predeterminado para el usuario
}
```

### Servicios
El dominio Administrar **Servicios** podría manejar la configuración y gestión de todos los servicios asociados y sus correspondientes tarifas. Estos pueden incluir servicios como estacionamiento, tarifas de transacciones, peajes, cuotas de manejo, entre otros.

- Las funcionalidades principales de este dominio son:
- Agregar un nuevo servicio: Los administradores deben poder agregar nuevos servicios y definir sus tarifas correspondientes.
- Ver los servicios existentes y sus tarifas: Los administradores deben poder ver todos los servicios disponibles y sus tarifas asociadas.
- Actualizar un servicio existente: Los administradores deben poder actualizar la información de un servicio existente, incluyendo la tarifa asociada.
- Eliminar un servicio: Los administradores deben poder eliminar un servicio que ya no se ofrece.

Objeto de dominio
```javascript
{
    "serviceId": "123",  // Un ID único para este servicio
    "serviceName": "Parking",  // El nombre del servicio
    "serviceDescription": "Parking fee",  // Una descripción del servicio
    "fee": 10.00,  // La tarifa asociada con el servicio
    "feeType": "hourly",  // El tipo de tarifa (por ejemplo, "hourly", "daily", "transaction", etc.)
    "isActive": true  // Un booleano que indica si este servicio está activo
}
```
### Billetera
El dominio de la Billetera podría encargarse de gestionar las operaciones relacionadas con la billetera digital del usuario. La billetera digital tiene la cantidad de dinero que un usuario tiene disponible.
Las principales funcionalidades pueden incluir:
- Recargar la billetera: Los usuarios deben poder recargar su billetera usando los medios de pago registrados.
- Ver el saldo de la billetera: Los usuarios deben poder ver cuánto dinero tienen en su billetera.
- Ver el historial de transacciones de la billetera: Los usuarios deben poder ver un registro de todas las recargas y gastos realizados con su billetera.

Objeto de dominio

```javascript
Wallet: {
    "userId": "123",  // El ID del usuario propietario de la billetera
    "walletId": "456",  // Un ID único para esta billetera
    "balance": 100.00,  // El saldo actual de la billetera
    "currency": "USD"  // La moneda en la que se mantiene el saldo
}

Transaction: {
    "transactionId": "789",  // Un ID único para esta transacción
    "walletId": "456",  // El ID de la billetera a la que pertenece esta transacción
    "amount": 10.00,  // La cantidad de dinero involucrada en la transacción
    "transactionType": "recharge",  // El tipo de transacción (por ejemplo, "recharge", "spend")
    "timestamp": "2023-07-24T12:34:56Z"  // El momento en que se realizó la transacción
}
```
### Transacciones
Para las transacciones de débito y crédito el proceso sería:

**Eventos de débito**

- Cuando se solicita una transacción de débito, por ejemplo, cuando un usuario paga por un servicio, una función de Lambda podría crear un evento 'DebitRequested' y enviarlo a EventBridge.
- EventBridge podría tener una regla configurada para enrutar eventos 'DebitRequested 'a una cola de SQS llamada'DebitQueue'.
- Una función de Lambda 'DebitProcessor' podría estar configurada para ser activada por los mensajes en 'DebitQueue'. Esta función tendría la lógica para verificar si el usuario tiene suficiente saldo en su billetera, descontar el dinero de la billetera y registrar la transacción en la tabla de DynamoDB.

**Eventos de crédito**

- Cuando un usuario recarga su billetera, una función de Lambda podría crear un evento 'CreditRequested' y enviarlo a 'EventBridge'.
- 'EventBridge' podría tener una regla configurada para enrutar eventos 'CreditRequested' a una cola de SQS llamada 'CreditQueue'.
- Una función de Lambda 'CreditProcessor' podría estar configurada para ser activada por los mensajes en 'CreditQueue'. Esta función tendría la lógica para agregar dinero al saldo de la billetera del usuario, sincronizar la información con el medio de pago y registrar la transacción en la tabla de DynamoDB.

Este diseño permite un procesamiento asíncrono de las transacciones de débito y crédito. Además, al utilizar colas de SQS, se pueden manejar correctamente los picos de tráfico y garantizar que todas las transacciones se procesen de forma fiable, incluso si hay problemas temporales con otros componentes del sistema. utilizando tecnicas como la idempotencia con el idTransaction y el datetime 

En el caso de que una transacción falle (por ejemplo, si un usuario intenta gastar más dinero del que tiene en su billetera), la función de Lambda podría devolver el mensaje a la cola SQS para ser reintentado más tarde, o podría enviar un evento de error a EventBridge para ser manejado por otro componente del sistem

# Estilo de arquitectura 
### Serverless
**- Ventajas:**

1 - **Sin administración de servidores:** No es necesario provisionar, mantener o administrar servidores. AWS lo hace por nosotros.

2 - **Escalabilidad automática:** La infraestructura serverless se escala automáticamente en función de la demanda.

3 - **Costo eficiente:** Con el modelo de precios de pago por uso, sólo pagas por el tiempo de ejecución de tus funciones. No pagas por la infraestructura ociosa.

4 - **Alta disponibilidad:** Los proveedores de servicios en la nube ofrecen alta disponibilidad y recuperación ante desastres con sus servicios serverless.


# Observabilidad 

Desacople de componentes: En una arquitectura serverless basada en eventos en AWS, utilizaremos los servicios:

- Con **CloudWatch** supervisamos  métricas de rendimiento de tus funciones Lambda, colas SQS y DynamoDB. crear alarmas que se activan si alguna de estas métricas supera un umbral que podría indicar un problema. (umbrales de procesamiento o memoria)

- Con usar **X-Ray** rastreamos las peticiones que ingresan hasta que se completan, pasando por todas las funciones Lambda, SQS y DynamoDB que se utilizan para procesar la solicitud. Esto nos facilita identificar cuellos de botella y problemas de rendimiento.

- los logs se enviaran desde las funciones Lambda a **CloudWatch Logs.** Estos logs pueden incluir información sobre eventos de transacciones, errores y cualquier otra cosa que pueda ser útil para entender lo que está pasando en tu sistema. con el fin de implementar modelos predictivos


## Scaffolder
Con la arquitectura hexagonal, también conocida como Ports and Adapters, el dominio de la aplicación (lógica de negocio y modelos de dominio) se coloca en el centro de la aplicación y se aísla de las infraestructuras y tecnologías externas.

```javascript
/wallet-service
|-- src
|   |-- application
|   |   |-- use_cases
|   |   |   |-- rechargeWallet.js
|   |   |   |-- debitWallet.js
|   |   |-- ports
|   |   |   |-- walletRepository.js
|   |-- domain
|   |   |-- wallet.js
|   |   |-- transaction.js
|   |-- infrastructure
|   |   |-- adapter
|   |   |   |-- dynamoDBWalletRepository.js
|   |   |-- api
|   |   |   |-- handler.js
|   |-- config
|   |   |-- dbConfig.js
|-- package.json
```

### Descripción del arquetipo la lambda wallet
src/application/use_cases: En esta carpeta, implementaremos los casos de uso, que implementan la lógica de negocio. puede tener dos archivos aquí: rechargeWallet.js y debitWallet.js.

src/application/ports: Aquí se definen las interfaces para los adaptadores. una interfaz walletRepository.js que defina los métodos para interactuar con la base de datos.

src/domain: Aquí se colocan los modelos de dominio. En este caso un modelo wallet.js para representar una billetera y un modelo transaction.js para representar una transacción.

src/infrastructure/adapter: En esta carpeta se implementan los adaptadores que interactúan con infraestructuras externas. Aquí es donde implementarías dynamoDBWalletRepository.js, que implementa la interfaz walletRepository.js utilizando DynamoDB.

src/infrastructure/api: Este es nuestro controlador para el API de Lambda. handler.js podría ser el punto de entrada a la función Lambda y se invocan los casos de uso apropiados en función del evento de entrada.

src/config: este archivo tiene la configuración de la aplicación, como la configuración de la base de datos o llamado a AWS secret.

package.json: Este es el archivo de configuración del proyecto Node.js, dependencias del proyecto.
