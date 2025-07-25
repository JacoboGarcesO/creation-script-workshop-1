# Consultas NoSQL para Sistema Bancario en MongoDB

A continuación se presentan 5 enunciados de consultas basados en las colecciones NoSQL del sistema bancario, junto con las soluciones utilizando operaciones avanzadas de MongoDB.

## 1. Análisis de Saldos por Tipo de Cuenta

**Enunciado:** El departamento financiero necesita un informe que muestre el saldo total, promedio, máximo y mínimo por cada tipo de cuenta (ahorro y corriente) para evaluar la distribución de fondos en el banco.

**Consulta MongoDB:**
```
db.clientes.aggregate([
  { 
    $unwind: "$cuentas" // Desenlaza cada cuenta como documento independiente
  },
  {
    $group: {
      _id: "$cuentas.tipo_cuenta",
      total_saldo: { $sum: "$cuentas.saldo" },
      promedio_saldo: { $avg: "$cuentas.saldo" },
      saldo_maximo: { $max: "$cuentas.saldo" },
      saldo_minimo: { $min: "$cuentas.saldo" },
      cantidad_cuentas: { $sum: 1 } // opcional si quieres ver el número de cuentas por tipo
    }
  },
  {
    $project: {
      _id: 0,
      tipo_cuenta: "$_id",
      total_saldo: 1,
      promedio_saldo: 1,
      saldo_maximo: 1,
      saldo_minimo: 1,
      cantidad_cuentas: 1
    }
  }
])
```

## 2. Patrones de Transacciones por Cliente

**Enunciado:** El equipo de análisis de comportamiento necesita identificar los patrones de transacciones de cada cliente, mostrando la cantidad y el monto total de transacciones por tipo (depósito, retiro, transferencia) para cada cliente.

**Consulta MongoDB:**
```
db.transacciones.aggregate([
  {
    $group: {
      _id: {
        cliente: "$cliente_ref",
        tipo: "$tipo_transaccion"
      },
      total_monto: { $sum: "$monto" },
      cantidad_transacciones: { $sum: 1 }
    }
  },
  {
    $group: {
      _id: "$_id.cliente",
      transacciones_por_tipo: {
        $push: {
          tipo_transaccion: "$_id.tipo",
          total_monto: "$total_monto",
          cantidad: "$cantidad_transacciones"
        }
      }
    }
  },
  {
    $lookup: {
      from: "clientes",
      localField: "_id",
      foreignField: "_id",
      as: "cliente_info"
    }
  },
  {
    $unwind: "$cliente_info"
  },
  {
    $project: {
      _id: 0,
      nombre_cliente: "$cliente_info.nombre",
      cedula: "$cliente_info.cedula",
      transacciones_por_tipo: 1
    }
  }
])
```

## 3. Clientes con Múltiples Tarjetas de Crédito

**Enunciado:** El departamento de riesgo crediticio necesita identificar a los clientes que poseen más de una tarjeta de crédito, mostrando sus datos personales, cantidad de tarjetas y el detalle de cada una.

**Consulta MongoDB:**
```
db.clientes.aggregate([
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
  }
])
```

## 4. Análisis de Medios de Pago más Utilizados

**Enunciado:** El departamento de marketing necesita conocer cuáles son los medios de pago más utilizados para depósitos, agrupados por mes, para orientar sus campañas promocionales.

**Consulta MongoDB:**
```
db.transacciones.aggregate([
  {
    $match: { tipo_transaccion: "deposito" } // Filtrar solo depósitos
  },
  {
    $addFields: {
      mes: { $dateToString: { format: "%Y-%m", date: { $toDate: "$fecha" } } }
    }
  },
    {
    $group: {
      _id: {
        mes: "$mes",
        medio_pago: "$detalles_deposito.medio_pago",
		cantidad: { $sum: 1 }
      }      
    }
  }
])
```

## 5. Detección de Cuentas con Transacciones Sospechosas

**Enunciado:** El departamento de seguridad necesita identificar cuentas con patrones de transacciones sospechosas, definidas como aquellas que tienen más de 3 retiros en un mismo día con un monto total superior a 1,000,000 COP.

**Consulta MongoDB:**
```
db.transacciones.aggregate([
  {
    $match: { tipo_transaccion: "retiro" }
  },
  {
    $addFields: {
      fecha_dia: { $dateToString: { format: "%Y-%m-%d", date: { $toDate: "$fecha" } } }
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
      transacciones: 1
    }
  }
])
```