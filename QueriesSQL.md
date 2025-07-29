# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql
select c.nombre , sum(cu.saldo) as saldoTotal , cc.cantidad_cuentas
from ( SELECT id_cliente, COUNT(id_cliente) AS cantidad_cuentas
       FROM cuenta
    GROUP BY id_cliente
) cc 
inner join cliente as c on cc.id_cliente = c.id_cliente  
inner join cuenta as cu on cu.id_cliente = c.id_cliente 
where cc.cantidad_cuentas >  1
group by c.nombre ,  cu.saldo , cc.cantidad_cuentas

order by cu.saldo desc

;
```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql
select c.nombre  
,   sum(case when tipo_transaccion = 'deposito' then monto else 0 end ) AS DEPOSITOS  ,  
sum(case when tipo_transaccion = 'retiro' then monto else 0 end ) AS retiros 
from cliente as c 
inner join cuenta as cu on c.id_cliente = cu.id_cliente
inner join transaccion as t on cu.num_cuenta = t.num_cuenta 
GROUP BY c.nombre  
order by c.nombre
;

```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql

select c.num_cuenta, t.numero_tarjeta 
from cuenta as c 
LEFT JOIN tarjeta as t ON t.num_cuenta = c.num_cuenta 
where t.num_cuenta is NULL
; 
```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql
select  c.id_cliente , c.tipo_cuenta , c.num_cuenta , AVG(c.saldo) as Saldo_promedio
from cuenta as c 
inner join transaccion as t on c.num_cuenta = t.num_cuenta 
where t.fecha >= CURRENT_DATE - INTERVAL '30' DAY 
GROUP by id_cliente , tipo_cuenta , c.num_cuenta 
order by id_cliente  , c.tipo_cuenta 

;
```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql
select cl.nombre , c.num_cuenta , t.tipo_transaccion , t.descripcion
from transaccion as t
inner join cuenta as c  on t.num_cuenta = c.num_cuenta 
inner join cliente as cl on cl.id_cliente = c.id_cliente
where t.descripcion like '%Transferencia%'

EXCEPT 

select cl.nombre , c.num_cuenta , t.tipo_transaccion  , t.descripcion 
from transaccion as t 
inner join cuenta as c  on t.num_cuenta = c.num_cuenta 
inner join cliente as cl on cl.id_cliente = c.id_cliente 
where t.descripcion = 'Retiro en cajero automático '

;
```
