# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql
select t.nombre, t.cantidad, t.total
from
(select cli.nombre, count(cue.num_cuenta) as cantidad, sum(cue.saldo) as total
from cliente cli
inner join cuenta cue on cli.id_cliente = cue.id_cliente
group by cli.nombre) as t
where t.cantidad > 1
order by t.total desc;
```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql
select clie.nombre, 
(
select COALESCE(sum(tra.monto),0)
from cliente cli
inner join cuenta cue on cli.id_cliente = cue.id_cliente
inner join transaccion tra on tra.num_cuenta = cue.num_cuenta
inner join deposito dep on tra.id_transaccion = dep.id_transaccion
where cli.id_cliente = clie.id_cliente
) as "Total deposito",
(
select COALESCE(sum(tra.monto),0)
from cliente cli
inner join cuenta cue on cli.id_cliente = cue.id_cliente
inner join transaccion tra on tra.num_cuenta = cue.num_cuenta
inner join retiro ret on tra.id_transaccion = ret.id_transaccion
where cli.id_cliente = clie.id_cliente
) as "Total retiro"
from cliente as clie;
```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql
select cli.nombre, cue.num_cuenta
from cliente cli
inner join cuenta cue on cli.id_cliente = cue.id_cliente
left join tarjeta tar on tar.num_cuenta = cue.num_cuenta 
where tar.id_tarjeta is null;
```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql
select 
(select avg(saldo)
from cuenta cue
inner join cuentaahorro cah on cue.num_cuenta = cah.num_cuenta
where (
	select count(t.num_cuenta)
	from transaccion as t
	where cue.num_cuenta = t.num_cuenta and (EXTRACT(EPOCH FROM (CURRENT_TIMESTAMP - fecha)) / (24 * 60 * 60))::INTEGER < 30
) > 0) as "Promedio saldo ahorro",
(select avg(saldo)
from cuenta cue
inner join cuentacorriente cco on cue.num_cuenta = cco.num_cuenta
where (
	select count(t.num_cuenta)
	from transaccion as t
	where cue.num_cuenta = t.num_cuenta and (EXTRACT(EPOCH FROM (CURRENT_TIMESTAMP - fecha)) / (24 * 60 * 60))::INTEGER < 30
) > 0) as "Promedio saldo corriente";
```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql
select cli.nombre
from cliente as cli
where 
(
select count(*) 
from cuenta cue
inner join transaccion tra on cue.num_cuenta = tra.num_cuenta
inner join transferencia tran on tra.id_transaccion = tran.id_transaccion
where cli.id_cliente = cue.id_cliente
) > 0 and (
select count(*) 
from cuenta cue
inner join transaccion tra on cue.num_cuenta = tra.num_cuenta
inner join retiro ret on tra.id_transaccion = ret.id_transaccion
where 
cli.id_cliente = cue.id_cliente and 
ret.canal = 'cajero') = 0 ;
```
