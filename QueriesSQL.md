# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql
select c.id_cliente, c.nombre, COUNT(ct.num_cuenta) as cantidad_cuentas, SUM(ct.saldo) as saldo_total from Cliente c 
inner join Cuenta ct on c.id_cliente = ct.id_cliente
group by c.id_cliente, c.nombre
having COUNT(ct.num_cuenta) > 1 
order by saldo_total desc;
```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql
select cl.nombre,
    COALESCE(SUM(case when t.tipo_transaccion = 'deposito' then t.monto end), 0) as total_depositos,
    COALESCE(SUM(case when t.tipo_transaccion = 'retiro' then t.monto end), 0) as total_retiros
from Cliente cl
inner join Cuenta c on cl.id_cliente = c.id_cliente
inner join Transaccion t on c.num_cuenta = t.num_cuenta
group by cl.nombre;
```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql
select c.num_cuenta, c.id_cliente from cuenta c
inner join cliente cl on c.id_cliente = cl.id_cliente
left join tarjeta t on c.num_cuenta = t.num_cuenta
where t.num_cuenta is null;
```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql
select c.tipo_cuenta, avg(c.saldo) as saldo_promedio from cuenta c
inner join transaccion t on c.num_cuenta = t.num_cuenta
where t.fecha >= current_date - interval '30 days'
group by c.tipo_cuenta;
```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql
select cl.id_cliente, cl.nombre, cl.correo from cliente cl
join cuenta c on cl.id_cliente = c.id_cliente
join transaccion t1 on c.num_cuenta = t1.num_cuenta
join transferencia tr on t1.id_transaccion = tr.id_transaccion
where cl.id_cliente not in (
        select cl2.id_cliente from cliente cl2
        join cuenta c2 on cl2.id_cliente = c2.id_cliente
        join transaccion t2 on c2.num_cuenta = t2.num_cuenta
        join retiro r on t2.id_transaccion = r.id_transaccion
        where r.canal = 'cajero');
```