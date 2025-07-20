# Consultas NoSQL para Sistema Bancario en MongoDB

A continuación se presentan 5 enunciados de consultas basados en las colecciones NoSQL del sistema bancario, junto con las soluciones utilizando operaciones avanzadas de MongoDB.

## 1. Análisis de Saldos por Tipo de Cuenta

**Enunciado:** El departamento financiero necesita un informe que muestre el saldo total, promedio, máximo y mínimo por cada tipo de cuenta (ahorro y corriente) para evaluar la distribución de fondos en el banco.

**Consulta MongoDB:**
```javascript
use("nosql_advance");

db.getCollection("Clientes")
.aggregate([
    {
        $unwind: "$cuentas"
    },
    {
        $group: {
          _id: "$cuentas.tipo_cuenta",
          total: {
            $sum: "$cuentas.saldo"
          },
          promedio: {
            $avg: "$cuentas.saldo"
          },
          maximo: {
            $max: "$cuentas.saldo"
          },
          minimo: {
            $min: "$cuentas.saldo"
          }
        }
    }
]);
```

## 2. Patrones de Transacciones por Cliente

**Enunciado:** El equipo de análisis de comportamiento necesita identificar los patrones de transacciones de cada cliente, mostrando la cantidad y el monto total de transacciones por tipo (depósito, retiro, transferencia) para cada cliente.

**Consulta MongoDB:**
```javascript
use("nosql_advance");

db.getCollection("Clientes").aggregate([
    {
        $lookup: {
          from: "Transacciones",
          localField: "_id",
          foreignField: "cliente_ref",
          as: "transactions_per_client"
        }
    },
    {
        $unwind: "$transactions_per_client"
    },
    {
        $group: {
            _id: "$transactions_per_client.tipo_transaccion",
            nombre: { $first: "$nombre" },
            monto: { $sum: "$transactions_per_client.monto" },
            tolta_transactions: { $count: {} }
        }
    }
]);

```

## 3. Clientes con Múltiples Tarjetas de Crédito

**Enunciado:** El departamento de riesgo crediticio necesita identificar a los clientes que poseen más de una tarjeta de crédito, mostrando sus datos personales, cantidad de tarjetas y el detalle de cada una.

**Consulta MongoDB:**
```javascript
use("nosql_advance");

db.getCollection("Clientes")
.aggregate([
    {
        $unwind: "$cuentas",
    },
    {
        $unwind: "$cuentas.tarjetas"
    },
    {
        $group: {
          _id: "$cuentas.tarjetas.tipo_tarjeta",
          nombre: { $first: "$nombre" },
          cedula: { $first: "$cedula" },
          correo: { $first: "$correo" },
          direccion: { $first: "$direccion" },
          count: {
            $count: {}
          },
          tarjetas: { $push: "$cuentas.tarjetas" }
        }
    },
    {
        $match: {
          count: { $gt: 1 }
        }
    }
]);
```

## 4. Análisis de Medios de Pago más Utilizados

**Enunciado:** El departamento de marketing necesita conocer cuáles son los medios de pago más utilizados para depósitos, agrupados por mes, para orientar sus campañas promocionales.

**Consulta MongoDB:**
```javascript
use("nosql_advance");

db.getCollection("Transacciones").aggregate([
    {
        $match: {
          tipo_transaccion: { $eq: "deposito" },
        }
    },
    {
        $addFields: {
            fecha_converted: { $toDate: "$fecha" }
        }
    },
    {
        $group: {
            _id: { $month: "$fecha_converted" },
            medio_pago: { $first: "$detalles_deposito.medio_pago" },
        }
    }
]);
```

## 5. Detección de Cuentas con Transacciones Sospechosas

**Enunciado:** El departamento de seguridad necesita identificar cuentas con patrones de transacciones sospechosas, definidas como aquellas que tienen más de 3 retiros en un mismo día con un monto total superior a 1,000,000 COP.

**Consulta MongoDB:**
```javascript
use("nosql_advance");

db.getCollection("Transacciones").aggregate([
    {
        $match: {
          tipo_transaccion: { $eq: "deposito" },
        }
    },
    {
        $addFields: {
            fecha_converted: { $toDate: "$fecha" }
        }
    },
    {
        $group: {
            _id: {
                month: { $month: "$fecha_converted" },
                year: { $year: "$fecha_converted" },
                day: { $dayOfMonth: "$fecha_converted" }
            },
            numero_transacciones: { $count: {} },
            monto: { $sum: "$monto" },
        }
    },
    {
        $match: {
          numero_transacciones: { $gt: 3 },
          monto: { $gt: 1000000 },
        }
    }
]);
```