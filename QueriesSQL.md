# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```
SELECT cli.nombre, COUNT(cue.num_cuenta) AS cantidad_cuentas, SUM(cue.saldo) AS saldo_total
FROM cliente cli
JOIN cuenta cue ON cli.id_cliente = cue.id_cliente
GROUP BY cli.id_cliente, cli.nombre
HAVING COUNT(cue.num_cuenta) > 1
ORDER BY saldo_total DESC;

```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```
SELECT cli.nombre, 
	COALESCE(SUM(CASE WHEN tra.tipo_transaccion = 'deposito' THEN tra.monto END), 0) AS total_depositos,
    COALESCE(SUM(CASE WHEN tra.tipo_transaccion = 'retiro' THEN tra.monto END), 0) AS total_retiros
FROM cliente cli
JOIN cuenta cue ON cli.id_cliente = cue.id_cliente
JOIN transaccion tra ON cue.num_cuenta = tra.num_cuenta
WHERE tra.tipo_transaccion IN ('deposito', 'retiro')
GROUP BY cli.id_cliente, cli.nombre
ORDER BY cli.nombre;

```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```
SELECT cue.num_cuenta, cli.nombre
FROM cuenta cue
JOIN cliente cli ON cue.id_cliente = cli.id_cliente
LEFT JOIN tarjeta tar ON cue.num_cuenta = tar.num_cuenta
WHERE tar.num_cuenta IS NULL;

```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```
SELECT cue.tipo_cuenta, ROUND(AVG(cue.saldo), 2) AS saldo_promedio
FROM cuenta cue
WHERE cue.num_cuenta IN (
        SELECT DISTINCT tra.num_cuenta
        FROM transaccion tra
        WHERE tra.fecha >= CURRENT_DATE - INTERVAL '30 days'
    )
GROUP BY cue.tipo_cuenta;

```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```
SELECT DISTINCT cli.nombre
FROM cliente cli
JOIN cuenta cue ON cli.id_cliente = cue.id_cliente
JOIN transaccion ts ON cue.num_cuenta = ts.num_cuenta
JOIN transferencia tf ON tf.id_transaccion = ts.id_transaccion
WHERE cli.id_cliente NOT IN (
    SELECT DISTINCT cl2.id_cliente
    FROM cliente cl2
    JOIN cuenta cue2 ON cl2.id_cliente = cue2.id_cliente
    JOIN transaccion ts_retiro ON cue2.num_cuenta = ts_retiro.num_cuenta
    JOIN retiro ret ON ret.id_transaccion = ts_retiro.id_transaccion
    WHERE ret.canal = 'cajero'
);

```
