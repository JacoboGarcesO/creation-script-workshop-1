# Consultas NoSQL para Sistema Bancario en MongoDB

A continuación se presentan 5 enunciados de consultas basados en las colecciones NoSQL del sistema bancario, junto con las soluciones utilizando operaciones avanzadas de MongoDB.

## 1. Análisis de Saldos por Tipo de Cuenta

**Enunciado:** El departamento financiero necesita un informe que muestre el saldo total, promedio, máximo y mínimo por cada tipo de cuenta (ahorro y corriente) para evaluar la distribución de fondos en el banco.

**Consulta MongoDB:**
```javascript
db.Clientes.aggregate([
  {
    $unwind: "$cuentas"
  },
  {
    $group: {
      _id: "$cuentas.tipo_cuenta",
      saldoTotal: { $sum: "$cuentas.saldo" },
      saldoPromedio: { $avg: "$cuentas.saldo" },
      saldoMaximo: { $max: "$cuentas.saldo" },
      saldoMinimo: { $min: "$cuentas.saldo" }
    }
  },
  {
    $project: {
      _id: 0,
      tipo_cuenta: "$_id",
      saldoTotal: 1,
      saldoPromedio: 1,
      saldoMaximo: 1,
      saldoMinimo: 1
    }
  }
])
resultado en mongo compas:
{
  saldoTotal: 1500000,
  saldoPromedio: 1500000,
  saldoMaximo: 1500000,
  saldoMinimo: 1500000,
  tipo_cuenta: 'ahorro'
}
{
  saldoTotal: 3000000,
  saldoPromedio: 3000000,
  saldoMaximo: 3000000,
  saldoMinimo: 3000000,
  tipo_cuenta: 'corriente'
}
```

## 2. Patrones de Transacciones por Cliente

**Enunciado:** El equipo de análisis de comportamiento necesita identificar los patrones de transacciones de cada cliente, mostrando la cantidad y el monto total de transacciones por tipo (depósito, retiro, transferencia) para cada cliente.

**Consulta MongoDB:**
```javascript
db.Transacciones.aggregate([
  {
    $group: {
      _id: {
        cliente: "$cliente_ref",
        tipoTransaccion: "$tipo_transaccion"
      },
      cantidad: {
        $sum: 1
      },
      montoTotal: {
        $sum: "$monto"
      }
    }
  },
  {
    $group: {
      _id: "$_id.cliente",
      patrones: {
        $push: {
          tipoTransaccion: "$_id.tipoTransaccion",
          cantidad: "$cantidad",
          montoTotal: "$montoTotal"
        }
      }
    }
  },
  {
    $lookup: {
      from: "Clientes",
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
      cliente: {
        nombre: "$cliente.nombre",
        cedula: "$cliente.cedula"
      },
      patrones: 1
    }
  }
])

resultado de la cosulta en mongo compas:
{
  patrones: [
    {
      tipoTransaccion: 'deposito',
      cantidad: 1,
      montoTotal: 200000
    },
    {
      tipoTransaccion: 'retiro',
      cantidad: 1,
      montoTotal: 50000
    },
    {
      tipoTransaccion: 'transferencia',
      cantidad: 1,
      montoTotal: 300000
    }
  ],
  cliente: {
    nombre: 'Juan Pérez',
    cedula: '1098765432'
  }
}
```

## 3. Clientes con Múltiples Tarjetas de Crédito

**Enunciado:** El departamento de riesgo crediticio necesita identificar a los clientes que poseen más de una tarjeta de crédito, mostrando sus datos personales, cantidad de tarjetas y el detalle de cada una.

**Consulta MongoDB:**
```javascript
db.Clientes.aggregate([
  {
    $unwind: "$cuentas"
  },
  {
    $unwind: "$cuentas.tarjetas"
  },
  {
    $match: {
      "cuentas.tarjetas.tipo_tarjeta": "credito"
    }
  },
  {
    $group: {
      _id: {
        cliente: "$_id",
        nombre: "$nombre",
        cedula: "$cedula"
      },
      cantidadTarjetasCredito: {
        $sum: 1
      },
      detallesTarjetas: {
        $push: "$cuentas.tarjetas"
      }
    }
  },
  {
    $match: {
      cantidadTarjetasCredito: {
        $gt: 1
      }
    }
  },
  {
    $project: {
      _id: 0,
      cliente: {
        id: "$_id.cliente",
        nombre: "$_id.nombre",
        cedula: "$_id.cedula"
      },
      cantidadTarjetasCredito: 1,
      detallesTarjetas: 1
    }
  }
])
Nota, veo que solo tenemos un cliente con una sola tarjeta, agregue uno para probar: 
{
  cantidadTarjetasCredito: 5,
  detallesTarjetas: [
    {
      id_tarjeta: 101,
      numero_tarjeta: '4024007111112222',
      tipo_tarjeta: 'credito',
      fecha_emision: '2023-02-01',
      fecha_expiracion: '2028-02-28'
    },
    {
      id_tarjeta: 102,
      numero_tarjeta: '4024007133334444',
      tipo_tarjeta: 'credito',
      fecha_emision: '2023-02-01',
      fecha_expiracion: '2028-02-28'
    },
    {
      id_tarjeta: 103,
      numero_tarjeta: '4024007155556666',
      tipo_tarjeta: 'credito',
      fecha_emision: '2023-05-15',
      fecha_expiracion: '2028-05-31'
    },
    {
      id_tarjeta: 104,
      numero_tarjeta: '4024007177778888',
      tipo_tarjeta: 'credito',
      fecha_emision: '2023-08-20',
      fecha_expiracion: '2028-08-31'
    },
    {
      id_tarjeta: 105,
      numero_tarjeta: '4024007199990000',
      tipo_tarjeta: 'credito',
      fecha_emision: '2023-10-10',
      fecha_expiracion: '2028-10-31'
    }
  ],
  cliente: {
    id: "ObjectId('507f1f77bcf86cd799439012')",
    nombre: 'juan diego mo',
    cedula: '1099888777'
  }
}.
```

## 4. Análisis de Medios de Pago más Utilizados

**Enunciado:** El departamento de marketing necesita conocer cuáles son los medios de pago más utilizados para depósitos, agrupados por mes, para orientar sus campañas promocionales.

**Consulta MongoDB:**
```javascript
db.Transacciones.aggregate([
  {
    $match: { tipo_transaccion: "deposito" }
  },
  {
    $addFields: {
      mes: { $month: { $toDate: "$fecha" } },
      anio: { $year: { $toDate: "$fecha" } }
    }
  },
  {
    $group: {
      _id: {
        anio: "$anio",
        mes: "$mes",
        medio_pago: "$detalles_deposito.medio_pago"
      },
      cantidad: { $sum: 1 }
    }
  },
  {
    $sort: { "_id.anio": 1, "_id.mes": 1, cantidad: -1 }
  },
  {
    $project: {
      _id: 0,
      anio: "$_id.anio",
      mes: "$_id.mes",
      medio_pago: "$_id.medio_pago",
      cantidad: 1
    }
  }
])
resultado en Mongo Compas:
{
  cantidad: 1,
  anio: 2023,
  mes: 5,
  medio_pago: 'transferencia'
}
```

## 5. Detección de Cuentas con Transacciones Sospechosas

**Enunciado:** El departamento de seguridad necesita identificar cuentas con patrones de transacciones sospechosas, definidas como aquellas que tienen más de 3 retiros en un mismo día con un monto total superior a 1,000,000 COP.

**Consulta MongoDB:**
```javascript
db.Transacciones.aggregate([
  {
    $match: {
      tipo_transaccion: "retiro"
    }
  },
  {
    $group: {
      _id: {
        cuenta: "$num_cuenta",
        // Se anida $toDate dentro de $dateToString para convertir y formatear en un solo paso
        fecha: { $dateToString: { format: "%Y-%m-%d", date: { $toDate: "$fecha" } } }
      },
      totalRetiros: { $sum: 1 },
      montoTotalRetirado: { $sum: "$monto" }
    }
  },
  {
    $match: {
      totalRetiros: { $gt: 3 },
      montoTotalRetirado: { $gt: 1000000 }
    }
  },
  {
    $project: {
      _id: 0,
      cuenta: "$_id.cuenta",
      fecha: "$_id.fecha",
      totalRetiros: 1,
      montoTotalRetirado: 1
    }
  }
])

Resultado: {
  totalRetiros: 3,
  montoTotalRetirado: 63000000,
  cuenta: 1000000002,
  fecha: '2023-05-20'
}
```