# Especificación: API de Transferencia entre cuentas

## 1. Visión general
Motor de transferencias internas book-to-book. Mueve dinero entre dos cuentas del mismo banco sin duplicidad, sin perderlo y sin dejarlo a medias

## 2. Requisitos funcionales
- Cuentas : Carga precargada (seeds) con el modelo
    `account_id(Guid)`
    `account_currency(string)`
    `account_balance(decimal)`
    `account_status(int)`

- Transferencias: Operaciones entre cuentas respetando el modelo:
    `transfer_id(int)`
    `transfer_account_origin_id(Guid)`
    `transfer_account_destiny_id(Guid)`
    `transfer_ammount(decimal)`
    `transfer_state(int)`
    `transfer_reference(string)`

- Historial: El historial de movimientos debe de mantenerse con la siguiente especificación de modelado:
    `movement_id(int)`
    `account_id(Guid)`
    `movement_status(string)`
    `movement_date(date)`

## 3. Reglas de negocio
- Aprovechando la tecnologia y la arquitectura aplicada CLEAN + DDD se deberan respetar los aggregados y sus limites de contexto para respetar las siguientes reglas:

- Saldos NO Negativos:
    `Para los movimientos de salida de una cuenta se debe implementar a nivel dominio la validación del balance igual a cero`

- Atomicidad débito / crédito:
    `Implementar patrón Unit of Work para respetar la atomicidad entre cuentas, como la comunicación será asincrona se validara al final que ambos movimientos hayan ocurrido satisfactoriamente si uno falla se revierte o no se aplica el movimiento`

- Conservación del dinero:
    `Cada movimiento debe de mantener integro el dinero creado en los seeds es decir solo contamos con 2 cuentas la Cuenta A tiene un ammount $100 y la Cuenta B un ammount $50 la suma $ 150 se tendrá que respetar a lo largo del tiempo en las transferencias entre ellas.`

    `SUM(balances) = SUM(balances_iniciales)`

- Idempotencia:
    - El modelo de idempotencia se creará con la estructura SHA256 y con el siguiente modelo:

    `transfer_account_origin_id(Guid)`
    `transfer_account_destiny_id(Guid)`
    `transfer_ammount(decimal)`
    `transfer_state(int)`
    `endpoint`

- Ademas de almaccenar la solicitud y respuesta del primer envío
    `requestData`
    `responseData`

## 5. Estados de Transferencia
- Maquina de estados:
    - INITAL: Se crea desde el inicio de un movimiento
    - PENDING: Se cambia al momento en que se consultan los balances entre cuentas
    - COMPLETED: Se actualiza al momento de que el movimiento pasa las reglas pertinentes 
    - FAILED: Se asenta en caso de que no cumpla con las reglas de validación

    - * Un movimiento en estado COMPLETED o FAILED jamas regresan a INITIAL o PENDING son estados terminales

## 4. Validaciones de Dominio
- `Montos`: Dentro de Account respetar 2 decimales en Account
- `Movimientos`: 
    - Validar que las cuentas sean diferentes para account_origin y account_destity
    - Las cuentas de un movimiento obligatoriamente deben de estar en ESTATUS Activo
    - La moneda entre transferencia debe de coincidir ente ambas
    - Agregar una referencia de máximo 140 caracteres.


## 4. Concurrencia
- PENDIENTE ...
