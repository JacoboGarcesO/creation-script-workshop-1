# Consultas NoSQL para Sistema Bancario en MongoDB

A continuación se presentan 5 enunciados de consultas basados en las colecciones NoSQL del sistema bancario, junto con las soluciones utilizando operaciones avanzadas de MongoDB.

## 1. Análisis de Saldos por Tipo de Cuenta

**Enunciado:** El departamento financiero necesita un informe que muestre el saldo total, promedio, máximo y mínimo por cada tipo de cuenta (ahorro y corriente) para evaluar la distribución de fondos en el banco.

**Consulta MongoDB:**
```javascript

db.clientes.aggregate([
  { $unwind: "$cuentas" }, // Descompone el arreglo de cuentas
  {
    $group: {
      _id: "$cuentas.tipo_cuenta",
      total_saldo: { $sum: "$cuentas.saldo" },
      saldo_promedio: { $avg: "$cuentas.saldo" },
      saldo_maximo: { $max: "$cuentas.saldo" },
      saldo_minimo: { $min: "$cuentas.saldo" }
    }
  },
  {
    $project: {
      _id: 0,
      tipo_cuenta: "$_id",
      total_saldo: 1,
      saldo_promedio: 1,
      saldo_maximo: 1,
      saldo_minimo: 1
    }
  }
])


```

## 2. Patrones de Transacciones por Cliente

**Enunciado:** El equipo de análisis de comportamiento necesita identificar los patrones de transacciones de cada cliente, mostrando la cantidad y el monto total de transacciones por tipo (depósito, retiro, transferencia) para cada cliente.

**Consulta MongoDB:**
```javascript

db.transacciones.aggregate([
  {
    $group: {
      _id: {
        cliente: "$cliente_ref",
        tipo_transaccion: "$tipo_transaccion"
      },
      cantidad_transacciones: { $sum: 1 },
      monto_total: { $sum: "$monto" }
    }
  },
  {
    $group: {
      _id: "$_id.cliente",
      transacciones_por_tipo: {
        $push: {
          tipo_transaccion: "$_id.tipo_transaccion",
          cantidad: "$cantidad_transacciones",
          monto_total: "$monto_total"
        }
      }
    }
  },
  {
    $lookup: {
      from: "clientes",
      localField: "_id",
      foreignField: "_id",
      as: "cliente"
    }
  },
  {
    $unwind: "$cliente"
  },
  {
    $project: {
      _id: 0,
      cliente_id: "$cliente._id",
      nombre: "$cliente.nombre",
      transacciones_por_tipo: 1
    }
  }
])


```

## 3. Clientes con Múltiples Tarjetas de Crédito

**Enunciado:** El departamento de riesgo crediticio necesita identificar a los clientes que poseen más de una tarjeta de crédito, mostrando sus datos personales, cantidad de tarjetas y el detalle de cada una.

**Consulta MongoDB:**
```javascript

db.clientes.aggregate([
  // Descomponer las cuentas
  { $unwind: "$cuentas" },
  // Descomponer las tarjetas de cada cuenta
  { $unwind: "$cuentas.tarjetas" },
  // Filtrar solo las tarjetas de tipo "credito"
  {
    $match: {
      "cuentas.tarjetas.tipo_tarjeta": "credito"
    }
  },
  // Agrupar por cliente
  {
    $group: {
      _id: "$_id",
      nombre: { $first: "$nombre" },
      cedula: { $first: "$cedula" },
      correo: { $first: "$correo" },
      direccion: { $first: "$direccion" },
      tarjetas_credito: {
        $push: "$cuentas.tarjetas"
      },
      cantidad_tarjetas_credito: { $sum: 1 }
    }
  },
  // Filtrar solo aquellos con más de una tarjeta de crédito
  {
    $match: {
      cantidad_tarjetas_credito: { $gt: 1 }
    }
  },
  // Proyectar el resultado limpio
  {
    $project: {
      _id: 0,
      nombre: 1,
      cedula: 1,
      correo: 1,
      direccion: 1,
      cantidad_tarjetas_credito: 1,
      tarjetas_credito: 1
    }
  }
])


```

## 4. Análisis de Medios de Pago más Utilizados

**Enunciado:** El departamento de marketing necesita conocer cuáles son los medios de pago más utilizados para depósitos, agrupados por mes, para orientar sus campañas promocionales.

**Consulta MongoDB:**
```javascript

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

## 5. Detección de Cuentas con Transacciones Sospechosas

**Enunciado:** El departamento de seguridad necesita identificar cuentas con patrones de transacciones sospechosas, definidas como aquellas que tienen más de 3 retiros en un mismo día con un monto total superior a 1,000,000 COP.

**Consulta MongoDB:**
```javascript

db.transacciones.aggregate([
  {
    $match: {
      tipo_transaccion: "retiro"
    }
  },
  {
    $addFields: {
      fecha_convertida: {
        $cond: {
          if: { $eq: [{ $type: "$fecha" }, "date"] },
          then: "$fecha",
          else: { $toDate: "$fecha" }
        }
      }
    }
  },
  {
    $addFields: {
      fecha_dia: {
        $dateToString: {
          format: "%Y-%m-%d",
          date: "$fecha_convertida"
        }
      }
    }
  },
  {
    $group: {
      _id: {
        num_cuenta: "$num_cuenta",
        fecha: "$fecha_dia"
      },
      cantidad_retiros: { $sum: 1 },
      monto_total: { $sum: "$monto" },
      transacciones: { $push: "$$ROOT" }
    }
  },
  {
    $match: {
      cantidad_retiros: { $gt: 3 },
      monto_total: { $gt: 1000000 }
    }
  },
  {
    $project: {
      _id: 0,
      num_cuenta: "$_id.num_cuenta",
      fecha: "$_id.fecha",
      cantidad_retiros: 1,
      monto_total: 1,
      transacciones: {
        _id: 1,
        fecha: 1,
        monto: 1,
        descripcion: 1,
        "detalles_retiro.canal": 1,
        cliente_ref: 1
      }
    }
  }
])


```