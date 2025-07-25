# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql
SELECT c.nombre, count(cu.*) AS numcuentas, sum(cu.saldo) AS saldototal
FROM cliente c
INNER JOIN cuenta cu ON c.id_cliente= cu.id_cliente
GROUP BY c.nombre
HAVING count(cu.*) > 1
ORDER BY saldototal DESC
```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql
SELECT c.nombre, 
	SUM(CASE WHEN tipo_transaccion = 'deposito' THEN monto ELSE 0 END) AS total_depositos,
	SUM(CASE WHEN tipo_transaccion = 'retiro' THEN monto ELSE 0 END) AS total_retiros
FROM transaccion t
INNER JOIN cuenta cu on t.num_cuenta=cu.num_cuenta
INNER JOIN cliente c on cu.id_cliente=c.id_cliente
WHERE t.tipo_transaccion IN ('deposito','retiro')
GROUP BY c.nombre
```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql
SELECT c.nombre, cu.num_cuenta,t.num_cuenta
FROM cliente c
INNER JOIN cuenta cu ON c.id_cliente= cu.id_cliente
LEFT JOIN tarjeta t ON cu.num_cuenta=t.num_cuenta
WHERE t.num_cuenta IS NULL
```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql
SELECT c.nombre,
	AVG(CASE WHEN tipo_cuenta = 'ahorro' THEN saldo ELSE 0 END) AS promedio_ahorro,
	AVG(CASE WHEN tipo_cuenta = 'corriente' THEN saldo ELSE 0 END) AS promedio_corriente
FROM cliente c
INNER JOIN cuenta cu ON c.id_cliente= cu.id_cliente
INNER JOIN transaccion t ON cu.num_cuenta=t.num_cuenta
WHERE t.fecha >= NOW() - INTERVAL '30 days'
GROUP BY c.nombre
```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql
SELECT c.nombre, t.tipo_transaccion, t.id_transaccion, r.canal
FROM cliente c
INNER JOIN cuenta cu ON c.id_cliente= cu.id_cliente
INNER JOIN transaccion t ON cu.num_cuenta=t.num_cuenta
INNER JOIN retiro r ON t.id_transaccion=r.id_transaccion
WHERE r.canal <> 'cajero'
```