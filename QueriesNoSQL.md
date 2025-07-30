# Consultas NoSQL para Sistema Bancario en MongoDB

A continuación se presentan 5 enunciados de consultas basados en las colecciones NoSQL del sistema bancario, junto con las soluciones utilizando operaciones avanzadas de MongoDB.

## 1. Análisis de Saldos por Tipo de Cuenta

**Enunciado:** El departamento financiero necesita un informe que muestre el saldo total, promedio, máximo y mínimo por cada tipo de cuenta (ahorro y corriente) para evaluar la distribución de fondos en el banco.

**Consulta MongoDB:**
```
    db.banco.aggregate([
    { $unwind: "$cuentas" },
    {
        $group: {
        _id: "$cuentas.tipo_cuenta",
        total_saldo: { $sum: "$cuentas.saldo" },
        promedio_saldo: { $avg: "$cuentas.saldo" },
        max_saldo: { $max: "$cuentas.saldo" },
        min_saldo: { $min: "$cuentas.saldo" },
        cantidad_cuentas: { $sum: 1 }
        }
    },
    {
        $project: {
        _id: 0,
        tipo_cuenta: "$_id",
        total_saldo: 1,
        promedio_saldo: 1,
        max_saldo: 1,
        min_saldo: 1,
        cantidad_cuentas: 1
        }
    }
    ])

```

## 2. Patrones de Transacciones por Cliente

**Enunciado:** El equipo de análisis de comportamiento necesita identificar los patrones de transacciones de cada cliente, mostrando la cantidad y el monto total de transacciones por tipo (depósito, retiro, transferencia) para cada cliente.

**Consulta MongoDB:**
```
db.banco.aggregate([
  { $unwind: "$cuentas" },
  { $unwind: "$cuentas.transacciones" },
  {
    $group: {
      _id: {
        cliente: "$nombre",
        tipo_transaccion: "$cuentas.transacciones.tipo"
      },
      total_monto: { $sum: "$cuentas.transacciones.monto" },
      cantidad: { $sum: 1 }
    }
  },
  {
    $group: {
      _id: "$_id.cliente",
      transacciones: {
        $push: {
          tipo: "$_id.tipo_transaccion",
          total_monto: "$total_monto",
          cantidad: "$cantidad"
        }
      }
    }
  },
  {
    $project: {
      _id: 0,
      cliente: "$_id",
      transacciones: 1
    }
  }
])
```

## 3. Clientes con Múltiples Tarjetas de Crédito

**Enunciado:** El departamento de riesgo crediticio necesita identificar a los clientes que poseen más de una tarjeta de crédito, mostrando sus datos personales, cantidad de tarjetas y el detalle de cada una.

**Consulta MongoDB:**
```
db.banco.aggregate([
  { $unwind: "$cuentas" },
  { $unwind: "$cuentas.tarjetas" },
  { $match: { "cuentas.tarjetas.tipo_tarjeta": "credito" } },
  {
    $group: {
      _id: "$_id",
      nombre: { $first: "$nombre" },
      cedula: { $first: "$cedula" },
      correo: { $first: "$correo" },
      direccion: { $first: "$direccion" },
      tarjetas_credito: { $push: "$cuentas.tarjetas" },
      cantidad_tarjetas_credito: { $sum: 1 }
    }
  },
  { $match: { cantidad_tarjetas_credito: { $gt: 1 } } },
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
```
db.banco.aggregate([
  { $unwind: "$cuentas" },
  { $unwind: "$cuentas.transacciones" },
  { $match: { "cuentas.transacciones.tipo": "deposito" } },
  {
    $addFields: {
      mes: {
        $dateToString: { format: "%Y-%m", date: "$cuentas.transacciones.fecha" }
      }
    }
  },
  {
    $group: {
      _id: {
        mes: "$mes",
        medio_pago: "$cuentas.transacciones.medio_pago"
      },
      cantidad: { $sum: 1 },
      total_monto: { $sum: "$cuentas.transacciones.monto" }
    }
  },
  {
    $project: {
      _id: 0,
      mes: "$_id.mes",
      medio_pago: "$_id.medio_pago",
      cantidad: 1,
      total_monto: 1
    }
  },
  { $sort: { mes: 1, cantidad: -1 } }
])
```

## 5. Detección de Cuentas con Transacciones Sospechosas

**Enunciado:** El departamento de seguridad necesita identificar cuentas con patrones de transacciones sospechosas, definidas como aquellas que tienen más de 3 retiros en un mismo día con un monto total superior a 1,000,000 COP.

**Consulta MongoDB:**
```
db.banco.aggregate([
  { $unwind: "$cuentas" },
  { $unwind: "$cuentas.transacciones" },
  { $match: { "cuentas.transacciones.tipo": "retiro" } },
  {
    $addFields: {
      fecha_dia: {
        $dateToString: { format: "%Y-%m-%d", date: "$cuentas.transacciones.fecha" }
      }
    }
  },
  {
    $group: {
      _id: {
        num_cuenta: "$cuentas.num_cuenta",
        fecha_dia: "$fecha_dia"
      },
      nombre: { $first: "$nombre" },
      cedula: { $first: "$cedula" },
      total_retiros: { $sum: "$cuentas.transacciones.monto" },
      cantidad_retiros: { $sum: 1 },
      detalles: {
        $push: {
          monto: "$cuentas.transacciones.monto",
          fecha: "$cuentas.transacciones.fecha"
        }
      }
    }
  },
  {
    $match: {
      cantidad_retiros: { $gt: 3 },
      total_retiros: { $gt: 1000000 }
    }
  },
  {
    $project: {
      _id: 0,
      nombre: 1,
      cedula: 1,
      num_cuenta: "$_id.num_cuenta",
      fecha: "$_id.fecha_dia",
      cantidad_retiros: 1,
      total_retiros: 1,
      detalles: 1
    }
  }
])
```
