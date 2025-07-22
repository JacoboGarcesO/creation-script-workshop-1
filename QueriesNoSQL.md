# Consultas NoSQL para Sistema Bancario en MongoDB

A continuación se presentan 5 enunciados de consultas basados en las colecciones NoSQL del sistema bancario, junto con las soluciones utilizando operaciones avanzadas de MongoDB.

## 1. Análisis de Saldos por Tipo de Cuenta

**Enunciado:** El departamento financiero necesita un informe que muestre el saldo total, promedio, máximo y mínimo por cada tipo de cuenta (ahorro y corriente) para evaluar la distribución de fondos en el banco.

**Consulta MongoDB:**
```javascript
Para la consulta es necesario descomponer las cuentas, para lo anterior usamos $unwind 
db.Clientes.aggregate([
  {$unwind: "$cuentas"},
  {
    $group: {
      _id: "$cuentas.tipo_cuenta",
      tot_sal: {$sum: "$cuentas.saldo"},
      pro_sal: {$avg: "$cuentas.saldo"},
      sal_max: {$max: "$cuentas.saldo"},
      sal_min: {$min: "$cuentas.saldo"}
    }
  }
])					  
```

## 2. Patrones de Transacciones por Cliente

**Enunciado:** El equipo de análisis de comportamiento necesita identificar los patrones de transacciones de cada cliente, mostrando la cantidad y el monto total de transacciones por tipo (depósito, retiro, transferencia) para cada cliente.

**Consulta MongoDB:**
```javascript
db.Transacciones.aggregate([
  {
    $group: {
      _id: {
        id_cli: "$cliente_ref",
        tip_tra: "$tipo_transaccion"
      },
      can_tra: { $sum: 1 },
      val_tra: { $sum: "$monto" }
    }
  }
])
```

## 3. Clientes con Múltiples Tarjetas de Crédito

**Enunciado:** El departamento de riesgo crediticio necesita identificar a los clientes que poseen más de una tarjeta de crédito, mostrando sus datos personales, cantidad de tarjetas y el detalle de cada una.

**Consulta MongoDB:**
```javascript
Para la consulta es necesario descomponer las cuentas y las tarjetas, para lo anterior usamos $unwind 
db.Clientes.aggregate([
  {$unwind: "$cuentas"},
  {$unwind: "$cuentas.tarjetas"},
  {$match: {"cuentas.tarjetas.tipo_tarjeta": "credito"}},
  {
    $group: {
      _id: "$_id",
      nom_cli: {$first: "$nombre"},
      can_tar: {$sum: 1},
      inf_tar: {$push: "$cuentas.tarjetas"}
    }
  },
  {$match: {can_tar: {$gt: 1}} 
}
])
```

## 4. Análisis de Medios de Pago más Utilizados

**Enunciado:** El departamento de marketing necesita conocer cuáles son los medios de pago más utilizados para depósitos, agrupados por mes, para orientar sus campañas promocionales.

**Consulta MongoDB:**
```javascript
Para agrupar por mes es necesario convertir la fecha a tipo DATE y desencadenar la fecha para obtener el mes
db.Transacciones.aggregate([
  {$match: {tipo_transaccion: "deposito"}},
  {
    $addFields: {
    new_fec: {$toDate: "$fecha"}
  }
  },
  {
    $group: {
      _id: {
        mes: {$dateToString: {format: "%m", date: "$new_fec"}},
        med_pag: "$detalles_deposito.medio_pago"
      },
      can_med: {$sum: 1},
    }
  },
  {$sort: {"_id.mes": 1, can_med: -1}}
])
```

## 5. Detección de Cuentas con Transacciones Sospechosas

**Enunciado:** El departamento de seguridad necesita identificar cuentas con patrones de transacciones sospechosas, definidas como aquellas que tienen más de 3 retiros en un mismo día con un monto total superior a 1,000,000 COP.

**Consulta MongoDB:**
```javascript
Para agrupar por dia es necesario convertir la fecha a tipo DATE y desencadenar la fecha para obtener el dia
db.Transacciones.aggregate([
  {
    $match: {tipo_transaccion: "retiro"}
  },
  {
    $addFields: {
      fec_dia: {$dateToString: {format: "%Y-%m-%d", date: {$toDate: "$fecha"}}
      }
    }
  },
  {
    $group: {
      _id: {
        num_cue: "$num_cuenta",
        fec_mov: "$fec_dia"
      },
      can_ret: {$sum: 1},
      mon_tot: {$sum: "$monto"},
    }
  },
  {
    $match: {
      can_ret: {$gt: 3},
      mon_tot: {$gt: 1000000}
    }
  }
])
```