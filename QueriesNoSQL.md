# Consultas NoSQL para Sistema Bancario en MongoDB

A continuación se presentan 5 enunciados de consultas basados en las colecciones NoSQL del sistema bancario, junto con las soluciones utilizando operaciones avanzadas de MongoDB.

## 1. Análisis de Saldos por Tipo de Cuenta

**Enunciado:** El departamento financiero necesita un informe que muestre el saldo total, promedio, máximo y mínimo por cada tipo de cuenta (ahorro y corriente) para evaluar la distribución de fondos en el banco.

**Consulta MongoDB:**
```javascript

db.clientes.aggregate([
  { $unwind: "$cuentas" }, 
  { $group: {
      _id: "$cuentas.tipo_cuenta",
      saldo_total: { $sum: "$cuentas.saldo" },
      promedio: { $avg: "$cuentas.saldo" },
      saldo_maximo: { $max: "$cuentas.saldo" },
      saldo_minimo: { $min: "$cuentas.saldo" }
  }}
]);

```

## 2. Patrones de Transacciones por Cliente

**Enunciado:** El equipo de análisis de comportamiento necesita identificar los patrones de transacciones de cada cliente, mostrando la cantidad y el monto total de transacciones por tipo (depósito, retiro, transferencia) para cada cliente.

**Consulta MongoDB:**
```javascript

db.Transacciones.aggregate( [
  {
    $group: {
      _id: {
        cliente_ref: "$cliente_ref",
        tipo_transaccion: "$tipo_transaccion"
      },
      cantidad_transacciones: { $sum: 1 },
      monto_total: { $sum: "$monto" }
    }
  }
])

```

## 3. Clientes con Múltiples Tarjetas de Crédito

**Enunciado:** El departamento de riesgo crediticio necesita identificar a los clientes que poseen más de una tarjeta de crédito, mostrando sus datos personales, cantidad de tarjetas y el detalle de cada una.

**Consulta MongoDB:**
```javascript

db.clientes.aggregate([
  // Desenrollar cuentas
  { $unwind: "$cuentas" },

  // Filtrar solo tarjetas de crédito dentro de cada cuenta
  {
    $project: {
      nombre: 1,
      cedula: 1,
      tarjetas_credito: {
        $filter: {
          input: "$cuentas.tarjetas",
          as: "tarjeta",
          cond: { $eq: ["$$tarjeta.tipo_tarjeta", "credito"] }
        }
      }
    }
  },

  // Agrupar por cliente para unir tarjetas
  {
    $group: {
      _id: { nombre: "$nombre", cedula: "$cedula" },
      tarjetas_credito: { $push: "$tarjetas_credito" }
    }
  },

  // Aplanar array 
  {
    $project: {
      nombre: "$_id.nombre",
      cedula: "$_id.cedula",
      tarjetas_credito: {
        $reduce: {
          input: "$tarjetas_credito",
          initialValue: [],
          in: { $concatArrays: ["$$value", "$$this"] }
        }
      }
    }
  },

  // Contar tarjetas credito
  {
    $addFields: {
      cantidad_tarjetas: { $size: "$tarjetas_credito" }
    }
  },

  // filtrar clientes con más de una tarjeta
  {
    $match: {
      cantidad_tarjetas: { $gt: 1 }
    }
  }
]);

```

## 4. Análisis de Medios de Pago más Utilizados

**Enunciado:** El departamento de marketing necesita conocer cuáles son los medios de pago más utilizados para depósitos, agrupados por mes, para orientar sus campañas promocionales.

**Consulta MongoDB:**
```javascript

db.Transacciones.aggregate([
  // Filtrar solo depósitos
  {
    $match: { tipo_transaccion: "deposito" }
  },

  // Mostrar mes y medio de pago
  {
    $project: {
      month: { $month: { $toDate: "$fecha" } },
      medio_pago: "$detalles_deposito.medio_pago"
    }
  },

  // mes y medio de pago
  {
    $group: {
      _id: {
        month: "$month",
        medio_pago: "$medio_pago"
      },
      cantidad: { $sum: 1 }
    }
  },

  // Ordenar por fecha y uso descendente
  {
    $sort: {
      "_id.month": 1,
      cantidad: -1
    }
  }
]);

```

## 5. Detección de Cuentas con Transacciones Sospechosas

**Enunciado:** El departamento de seguridad necesita identificar cuentas con patrones de transacciones sospechosas, definidas como aquellas que tienen más de 3 retiros en un mismo día con un monto total superior a 1,000,000 COP.

**Consulta MongoDB:**
```javascript

db.Transacciones.aggregate([
  { 
    $match: { tipo_transaccion: "retiro" } 
  },
  { 
    $group: {
      _id: {
        num_cuenta: "$num_cuenta",
        dia: { $dateToString: { format: "%Y-%m-%d", date: { $toDate: "$fecha" } } }
      },
      totalRetiros: { $sum: 1 },
      montoTotal: { $sum: "$monto" },
      detalles: { $push: "$$ROOT" }
    }
  },
  { 
    $match: { 
      totalRetiros: { $gt: 3 },
      montoTotal: { $gt: 1000000 }
    }
  },
  {
    $project: {
      num_cuenta: "$_id.num_cuenta",
      dia: "$_id.dia",
      totalRetiros: 1,
      montoTotal: 1,
      detalles: 1,
      _id: 0
    }
  }
]);

```