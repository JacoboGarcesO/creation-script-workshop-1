# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql
SELECT * FROM (
	SELECT cli.id_cliente, cli.nombre, COUNT(cli.id_cliente) NUM_CUE, SUM(cue.saldo) SAL_ACU
	FROM cliente cli
	INNER JOIN cuenta cue ON cli.id_cliente = cue.id_cliente
	GROUP BY cli.id_cliente, cli.nombre)
WHERE NUM_CUE > 1
ORDER BY SAL_ACU DESC
;
```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql
SELECT cli.id_cliente, 
SUM(CASE 
	WHEN tra.tipo_transaccion = 'deposito' THEN tra.monto
	ELSE 0 END) AS TOL_DEP,
SUM(CASE 
	WHEN tra.tipo_transaccion = 'retiro' THEN tra.monto
	ELSE 0 END) AS TOL_RET
FROM cliente cli
INNER JOIN cuenta cue ON cli.id_cliente = cue.id_cliente
INNER JOIN transaccion tra ON cue.num_cuenta = tra.num_cuenta
WHERE tra.tipo_transaccion IN ('deposito', 'retiro')
GROUP BY cli.id_cliente
;
```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql
SELECT cue.num_cuenta
FROM cuenta cue
WHERE cue.num_cuenta NOT IN (SELECT tar.num_cuenta
			     FROM tarjeta tar)
;
```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql
SELECT cue.tipo_cuenta, AVG(saldo) PRO_SALDO
FROM cuenta cue
INNER JOIN (SELECT num_cuenta, COUNT(id_transaccion) NUM_TRA
			FROM transaccion
			WHERE TO_CHAR(fecha::date, 'YYYYMMDD')::date >= TO_CHAR(fecha::date, 'YYYYMMDD')::date - INTERVAL '30 days'
			--CURRENT_DATE - INTERVAL '30 days'
			GROUP BY num_cuenta) tra
			ON tra.num_cuenta = cue.num_cuenta
AND tra.num_tra > 0
AND cue.tipo_cuenta IN ('ahorro', 'corriente')
GROUP BY cue.tipo_cuenta
;
```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql
SELECT cli.id_cliente, MAX(cli.nombre)
FROM cliente cli
INNER JOIN cuenta cue ON cue.id_cliente = cli.id_cliente
INNER JOIN transaccion tra ON tra.num_cuenta = cue.num_cuenta
WHERE cli.id_cliente NOT IN (SELECT DISTINCT cli2.id_cliente
						  FROM cliente cli2
						  INNER JOIN cuenta cue2 ON cue2.id_cliente = cli2.id_cliente
						  INNER JOIN transaccion tra2 ON tra2.num_cuenta = cue2.num_cuenta
						  WHERE tra2.tipo_transaccion = 'retiro'
						  AND tra2.descripcion LIKE '%cajero%')
GROUP BY cli.id_cliente					  
;
```