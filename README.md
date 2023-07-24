# assessment-f2x
Propuesta de soluciòn al ejercicio - f2x-v2.0.0


# scaffolder
# Contexto
La solucion propuesta provee funcionalidades para el registro de usuarios (onboarding), registro de vehículos (enrolamiento), transacciones de pago de servicios para vehículos (parqueadero, peajes), consulta de saldos y movimientos (balances), registro de medios de pago y la posibilidad de realizar recargas a la cuenta FlyPass.

# Onboarding
![image](https://github.com/4dagio/assessment-f2x/assets/3275936/7263c75a-06a8-4684-aff3-c053e23c7c40)

![image](https://github.com/4dagio/assessment-f2x/assets/3275936/065a742d-aae6-46dd-b927-81f2c8c6f2d1)



# Contenedores

![image](https://github.com/4dagio/assessment-f2x/assets/3275936/27b97f0b-35d2-4fac-aece-d469eaaab426)


La solución consiste en los siguientes contenedores:
- [X] Frontend: Interfaz de usuario para realizar las operaciones (staff y usuarios)
- [X] Servicio de Onboarding: Registra nuevos usuarios en el sistema.
- [X] Servicio de Enrolamiento: Permite a los usuarios registrar y asociar vehículos en su cuenta.
- [X] Servicio de Transacciones: Pasarela de pagos para servicios de vehículos.
- [X] Servicio de Balances: Permite consultar el saldo de la cuenta y los movimientos realizados.
- [X] Servicio de Medios de Pago: Permite registrar una tarjeta de crédito o realizar una recarga en la cuenta.
- [X] Servicio de Recargas: Permite enviar dinero a la cuenta del usuario.
- [X] Broker de Eventos: Orquesta la comunicación entre servicios y maneja eventos asíncronos.
- [X] Servicio de Autenticación: Maneja la autenticación y autorización de usuarios.

# Componentes

![image](https://github.com/4dagio/assessment-f2x/assets/3275936/6c7e8a89-40dc-40b4-98a6-cadc54762767)


## Scaffolder

```javascript
/src
  /application
    /use_cases
      user_use_case.ts
  /domain
    /entities
      user.ts
    /repositories
      user_repository.ts
  /infrastructure
    /db
      /mongodb
        user_repository.ts
    /api
      /controllers
        user_controller.ts
      /routes
        user_routes.ts
  /interfaces
    /adapters
      user_adapter.ts
    /dtos
      user_dto.ts
  index.ts
/package.json
/tsconfig.json
```
