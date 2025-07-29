# Documento JSON NoSQL para Sistema Bancario

A continuación se presenta un ejemplo de cómo se estructuraría la información del sistema bancario en un formato NoSQL basado en documentos (MongoDB).

## Colección: Clientes

```json
{
  "_id": "ObjectId('507f1f77bcf86cd799439011')",
  "nombre": "Juan Pérez",
  "cedula": "1098765432",
  "correo": "juan.perez@email.com",
  "direccion": "Calle 123 #45-67, Medellín, Colombia",
  "cuentas": [
    {
      "num_cuenta": 1000000001,
      "tipo_cuenta": "ahorro",
      "saldo": 1500000.00,
      "fecha_apertura": "2022-01-15",
      "detalles_ahorro": {
        "tasa_interes": 2.5,
        "limite_retiros": 3
      },
      "tarjetas": [
        {
          "id_tarjeta": 1,
          "numero_tarjeta": "4539123456781234",
          "tipo_tarjeta": "debito",
          "fecha_emision": "2022-01-15",
          "fecha_expiracion": "2027-01-31"
        }
      ]
    },
    {
      "num_cuenta": 1000000002,
      "tipo_cuenta": "corriente",
      "saldo": 3000000.00,
      "fecha_apertura": "2022-02-20",
      "detalles_corriente": {
        "limite_sobregiro": 1000000.00,
        "comision_mensual": 12000.00
      },
      "tarjetas": [
        {
          "id_tarjeta": 2,
          "numero_tarjeta": "5412345678901234",
          "tipo_tarjeta": "debito",
          "fecha_emision": "2022-02-20",
          "fecha_expiracion": "2027-02-28"
        },
        {
          "id_tarjeta": 3,
          "numero_tarjeta": "4024007123456789",
          "tipo_tarjeta": "credito",
          "fecha_emision": "2022-03-10",
          "fecha_expiracion": "2027-03-31"
        }
      ]
    }
  ]
}
,
{
  "_id": "ObjectId('507f1f77bcf86cd799439012')",
  "nombre": "juan diego mo",
  "cedula": "1099888777",
  "correo": "juan.diego.mo@email.com",
  "direccion": "Carrera 80 #10-15, Bogotá, Colombia",
  "cuentas": [
    {
      "num_cuenta": 2000000001,
      "tipo_cuenta": "corriente",
      "saldo": 4500000.00,
      "fecha_apertura": "2023-01-20",
      "detalles_corriente": {
        "limite_sobregiro": 2500000.00,
        "comision_mensual": 18000.00
      },
      "tarjetas": [
        {
          "id_tarjeta": 101,
          "numero_tarjeta": "4024007111112222",
          "tipo_tarjeta": "credito",
          "fecha_emision": "2023-02-01",
          "fecha_expiracion": "2028-02-28"
        },
        {
          "id_tarjeta": 102,
          "numero_tarjeta": "4024007133334444",
          "tipo_tarjeta": "credito",
          "fecha_emision": "2023-02-01",
          "fecha_expiracion": "2028-02-28"
        },
        {
          "id_tarjeta": 103,
          "numero_tarjeta": "4024007155556666",
          "tipo_tarjeta": "credito",
          "fecha_emision": "2023-05-15",
          "fecha_expiracion": "2028-05-31"
        },
        {
          "id_tarjeta": 104,
          "numero_tarjeta": "4024007177778888",
          "tipo_tarjeta": "credito",
          "fecha_emision": "2023-08-20",
          "fecha_expiracion": "2028-08-31"
        },
        {
          "id_tarjeta": 105,
          "numero_tarjeta": "4024007199990000",
          "tipo_tarjeta": "credito",
          "fecha_emision": "2023-10-10",
          "fecha_expiracion": "2028-10-31"
        }
      ]
    }
  ]
}

```

## Colección: Transacciones

```json
{
  "_id": "ObjectId('607f1f77bcf86cd799439022')",
  "num_cuenta": 1000000001,
  "fecha": "2023-05-10T14:30:45.000Z",
  "monto": 200000.00,
  "tipo_transaccion": "deposito",
  "descripcion": "Depósito de nómina",
  "detalles_deposito": {
    "medio_pago": "transferencia"
  },
  "cliente_ref": "ObjectId('507f1f77bcf86cd799439011')"
}
```

```json
{
  "_id": "ObjectId('607f1f77bcf86cd799439023')",
  "num_cuenta": 1000000001,
  "fecha": "2023-05-15T10:20:30.000Z",
  "monto": 50000.00,
  "tipo_transaccion": "retiro",
  "descripcion": "Retiro en cajero automático",
  "detalles_retiro": {
    "canal": "cajero"
  },
  "cliente_ref": "ObjectId('507f1f77bcf86cd799439011')"
}
```

```json
{
  "_id": "ObjectId('607f1f77bcf86cd799439024')",
  "num_cuenta": 1000000002,
  "fecha": "2023-05-20T16:45:12.000Z",
  "monto": 300000.00,
  "tipo_transaccion": "transferencia",
  "descripcion": "Pago de servicios",
  "detalles_transferencia": {
    "cuenta_destino": 1000000003
  },
  "cliente_ref": "ObjectId('507f1f77bcf86cd799439011')"
}
```

```json
{
  "_id": "ObjectId('607f1f77bcf86cd799439025')",
  "num_cuenta": 1000000002,
  "fecha": "2023-05-20T16:45:12.000Z",
  "monto": 3000000.00,
  "tipo_transaccion": "transferencia",
  "descripcion": "Pago de servicios",
  "detalles_transferencia": {
    "cuenta_destino": 1000000003
  },
  "cliente_ref": "ObjectId('507f1f77bcf86cd799439011')"
}
```

```json
{
  "_id": "ObjectId('607f1f77bcf86cd799439026')",
  "num_cuenta": 1000000002,
  "fecha": "2023-05-20T16:45:12.000Z",
  "monto": 30000000.00,
  "tipo_transaccion": "retiro",
  "descripcion": "Pago de servicios",
  "detalles_transferencia": {
    "cuenta_destino": 1000000003
  },
  "cliente_ref": "ObjectId('507f1f77bcf86cd799439011')"
}
```
```json
{
  "_id": "ObjectId('607f1f77bcf86cd799439027')",
  "num_cuenta": 1000000002,
  "fecha": "2023-05-20T16:45:12.000Z",
  "monto": 30000000.00,
  "tipo_transaccion": "retiro",
  "descripcion": "Pago de servicios",
  "detalles_transferencia": {
    "cuenta_destino": 1000000003
  },
  "cliente_ref": "ObjectId('507f1f77bcf86cd799439011')"
}
```
```json
{
  "_id": "ObjectId('607f1f77bcf86cd799439028')",
  "num_cuenta": 1000000002,
  "fecha": "2023-05-20T16:45:12.000Z",
  "monto": 30000000.00,
  "tipo_transaccion": "retiro",
  "descripcion": "Pago de servicios",
  "detalles_transferencia": {
    "cuenta_destino": 1000000003
  },
  "cliente_ref": "ObjectId('507f1f77bcf86cd799439011')"
}
```json