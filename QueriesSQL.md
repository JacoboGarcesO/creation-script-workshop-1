# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql
select c.id_cliente, count(distinct ct.num_cuenta) as numero_de_cuentas, sum(ct.saldo) as saldo_total 
from cliente c 
join cuenta ct on c.id_cliente = ct.id_cliente 
group by c.id_cliente 
having count(distinct ct.num_cuenta) > 1
order by saldo_total desc
```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql
with depositos_data as (
	select c.id_cliente, sum(t.monto) as total
	from cliente c 
	join cuenta ct on c.id_cliente = ct.id_cliente 
	join transaccion t on t.num_cuenta  = ct.num_cuenta 
	join deposito d on d.id_transaccion = t.id_transaccion 
	group by c.id_cliente
),
retiros_data as (
	select c.id_cliente, sum(t.monto) as total
	from cliente c 
	join cuenta ct on c.id_cliente = ct.id_cliente 
	join transaccion t on t.num_cuenta  = ct.num_cuenta 
	join retiro r on r.id_transaccion = t.id_transaccion 
	group by c.id_cliente
)
select c.id_cliente, c.nombre, d.total as total_depositos, r.total as total_retiros
from cliente c
join depositos_data d on d.id_cliente = c.id_cliente 
join retiros_data r on r.id_cliente = c.id_cliente 
```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql
select c.id_cliente, c.nombre
from cliente c 
join cuenta ct on ct.id_cliente = c.id_cliente 
where not exists (
	select 1
	from tarjeta 
	where num_cuenta is not null
) 
```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql
select 
    ct.tipo_cuenta,
    count(distinct ct.num_cuenta) as total_cuentas,
    round(avg(ct.saldo), 2) as saldo_promedio
from cuenta ct
join transaccion t on t.num_cuenta = ct.num_cuenta
where t.fecha >= now() - interval '30 days'
group by ct.tipo_cuenta
order by saldo_promedio DESC;
```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql
with clientes_transferencias as (
    select distinct c.id_cliente, c.nombre
    from cliente c
    join cuenta ct on c.id_cliente = ct.id_cliente
    join transaccion t on t.num_cuenta = ct.num_cuenta
    join transferencia tr on tr.id_transaccion = t.id_transaccion
),
clientes_retiros_cajero as (
    select distinct c.id_cliente
    from cliente c
    join cuenta ct on c.id_cliente = ct.id_cliente
    join transaccion t on t.num_cuenta = ct.num_cuenta
    join retiro r on r.id_transaccion = t.id_transaccion
    where r.canal = 'cajero'
)
select 
    ct.id_cliente,
    ct.nombre,
    COUNT(*) as total_transferencias
from clientes_transferencias ct
left join clientes_retiros_cajero crc on ct.id_cliente = crc.id_cliente
where crc.id_cliente is null
group by ct.id_cliente, ct.nombre
order by total_transferencias desc;
```