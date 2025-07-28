# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql

select cl.id_cliente, cl.nombre, count (cu.num_cuenta) as cantidad_cuentas, sum(cu.saldo) as saldo_total from cliente cl
inner join cuenta cu on cl.id_cliente = cu.id_cliente
group by cl.id_cliente, cl.nombre
having count (cu.num_cuenta) > 1
order by sum(cu.saldo) desc

```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql

select cl.id_cliente, cl.nombre, sum(tr.monto) as monto_total, tr.tipo_transaccion from cliente cl 
inner join cuenta cu  on cl.id_cliente = cu.id_cliente
inner join Transaccion tr on tr.num_cuenta = cu.num_cuenta
group by cl.id_cliente, cl.nombre, tr.tipo_transaccion
having tipo_transaccion in ('deposito','retiro')
order by cl.id_cliente, sum(tr.monto) desc

```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql

select cu.num_cuenta, ta.id_tarjeta, ta.numero_tarjeta, ta.tipo_tarjeta  from Cuenta cu 
left join Tarjeta ta on cu.num_cuenta = ta.num_cuenta
where ta.num_cuenta is null

```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql

select cu.tipo_cuenta, AVG(saldo) as promedio from Transaccion tr 
inner join Cuenta cu on tr.num_cuenta = cu.num_cuenta
where  (tr.fecha >= GETDATE() -30)
group by cu.tipo_cuenta

```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql

select cl.id_cliente, cl.nombre, tr.tipo_transaccion, rt.canal from cliente cl 
inner join cuenta cu on cu.id_cliente = cl.id_cliente 
inner join Transaccion tr on cu.num_cuenta = tr.num_cuenta
left join retiro rt on rt.id_transaccion =tr.id_transaccion
where tipo_transaccion in ('transferencia','retiro')
     and not exists
	 (select cl.id_cliente, cl.nombre, tr.tipo_transaccion, rt.canal from cliente cl 
	 inner join cuenta cu on cu.id_cliente = cl.id_cliente 
	 inner join Transaccion tr on cu.num_cuenta = tr.num_cuenta
	 left join retiro rt on rt.id_transaccion =tr.id_transaccion
	 where rt.canal in ('cajero'))
group by  cl.id_cliente, cl.nombre, tr.tipo_transaccion, rt.canal

```