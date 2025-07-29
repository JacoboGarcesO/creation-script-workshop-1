# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql
SELECT c.id_cliente, c.cedula, COUNT(cu.num_cuenta) AS num_cuentas, SUM(cu.saldo) AS saldo_acumulado  
FROM Cliente c
INNER JOIN Cuenta cu ON c.id_cliente = cu.id_cliente
GROUP BY c.id_cliente, c.cedula
HAVING COUNT(cu.num_cuenta) >= 2
ORDER BY saldo_acumulado DESC;
```  

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql
SELECT c.id_cliente, c.cedula,
	COALESCE(SUM(CASE WHEN t.tipo_transaccion = 'deposito' THEN t.monto END), 0) AS total_depositos, 
	COALESCE(SUM(CASE WHEN t.tipo_transaccion = 'retiro' THEN t.monto END), 0) AS total_retiros
FROM Cliente c
INNER JOIN Cuenta cu ON c.id_cliente  = cu.id_cliente
INNER JOIN Transaccion t ON cu.num_cuenta = t.num_cuenta
WHERE t.tipo_transaccion IN ('deposito', 'retiro')
GROUP BY c.id_cliente, c.cedula;
```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql
SELECT c.num_cuenta
FROM Cuenta c
LEFT JOIN Tarjeta t ON c.num_cuenta = t.num_cuenta
WHERE t.id_tarjeta IS NULL;

//Otra posible forma de hacerlo seria

SELECT c.num_cuenta
FROM Cuenta c

EXCEPT

SELECT t.num_cuenta
FROM Tarjeta t;
```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql
SELECT 
    tipo_cuenta,
    AVG(saldo) AS saldo_promedio
FROM (
    SELECT DISTINCT cu.num_cuenta, cu.tipo_cuenta, cu.saldo
    FROM Cuenta cu
    INNER JOIN Transaccion t ON cu.num_cuenta = t.num_cuenta
    WHERE t.fecha >= NOW() - INTERVAL '30' DAY
)
GROUP BY tipo_cuenta;
```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql
SELECT DISTINCT c.id_cliente, c.cedula
FROM Cliente c
INNER JOIN Cuenta cu ON c.id_cliente = cu.id_cliente
INNER JOIN Transaccion t ON cu.num_cuenta = t.num_cuenta
INNER JOIN Transferencia tf ON tf.id_transaccion  = t.id_transaccion

EXCEPT

SELECT DISTINCT c.id_cliente, c.cedula
FROM Cliente c
INNER JOIN Cuenta cu ON c.id_cliente = cu.id_cliente
INNER JOIN Transaccion t ON cu.num_cuenta = t.num_cuenta
INNER JOIN Retiro r ON r.id_transaccion = t.id_transaccion
WHERE canal = 'cajero';
```