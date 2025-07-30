# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql

SELECT
  COUNT(cu.num_cuenta) AS cantidad_cuentas,
  SUM(cu.saldo) AS saldo_total,
  cli.cedula
FROM cliente cli
JOIN cuenta cu ON cli.id_cliente = cu.id_cliente
GROUP BY cli.cedula
HAVING COUNT(cu.num_cuenta) > 2
ORDER BY saldo_total;

```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql

SELECT
consulta_depositos.cedula,
consulta_depositos.monto_deposito,
consulta_retiro.monto_retiro
FROM
	(SELECT
	SUM(tra.monto) AS monto_deposito,
	cli.cedula
	FROM
	cliente cli,
	cuenta cu,
	transaccion tra
	WHERE
	cli.id_cliente = cu.id_cliente
	AND cu.num_cuenta = tra.num_cuenta
	AND tra.tipo_transaccion = 'deposito'
	GROUP BY cli.cedula) consulta_depositos,	
	(SELECT
	SUM(tra.monto) AS monto_retiro,
	cli.cedula
	FROM
	cliente cli,
	cuenta cu,
	transaccion tra
	WHERE
	cli.id_cliente = cu.id_cliente
	AND cu.num_cuenta = tra.num_cuenta
	AND tra.tipo_transaccion = 'retiro'
	GROUP BY cli.cedula ) consulta_retiro
WHERE
consulta_depositos.cedula = consulta_retiro.cedula;

```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql

SELECT
* FROM
cuenta cu
where
cu.num_cuenta not in (select t.num_cuenta from tarjeta t);


```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql
SELECT t.num_cuenta, t.id_transaccion
FROM transaccion t
WHERE t.tipo_transaccion = 'transferencia'
  AND NOT EXISTS (
    SELECT t2.num_cuenta 
    FROM transaccion t2
    JOIN retiro r ON t2.id_transaccion = r.id_transaccion
    WHERE t2.num_cuenta = t.num_cuenta
      AND r.canal = 'cajero'
  );


```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql
SELECT AVG(t.monto ) promedioCAhorro
FROM 
transaccion t,
cuentaahorro ch
where
t.num_cuenta = ch.num_cuenta
and t.fecha >= '2023-03-01 00:00:00';

SELECT AVG(t.monto ) promedioCAhorro
FROM 
transaccion t,
cuentacorriente ch
where
t.num_cuenta = ch.num_cuenta
and t.fecha >= '2023-03-01 00:00:00';


```
