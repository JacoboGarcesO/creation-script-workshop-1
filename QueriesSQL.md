# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql

SELECT 
    c.id_cliente,
    c.nombre,
    COUNT(ct.num_cuenta) AS cantidad_cuentas,
    SUM(ct.saldo) AS saldo_total
FROM 
    Cliente c
JOIN 
    Cuenta ct ON c.id_cliente = ct.id_cliente
GROUP BY 
    c.id_cliente, c.nombre
HAVING 
    COUNT(ct.num_cuenta) > 1
ORDER BY 
    saldo_total DESC;

```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql

SELECT 
    cl.id_cliente,
    cl.nombre,
    SUM(CASE WHEN t.tipo_transaccion = 'deposito' THEN t.monto END) AS Total_Depositos,
    SUM(CASE WHEN t.tipo_transaccion = 'retiro' THEN t.monto END) AS Total_Retiros,
    SUM(CASE WHEN t.tipo_transaccion = 'deposito' THEN t.monto END) -
    SUM(CASE WHEN t.tipo_transaccion = 'retiro' THEN t.monto END) AS Balance
FROM 
    Cliente cl
JOIN 
    Cuenta c ON cl.id_cliente = c.id_cliente
JOIN 
    Transaccion t ON c.num_cuenta = t.num_cuenta
GROUP BY 
    cl.id_cliente, cl.nombre;

```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql

SELECT 
    c.num_cuenta,
    c.id_cliente,
    cli.nombre,
    c.tipo_cuenta,
    c.saldo,
    c.fecha_apertura,
    t.fecha_expiracion 
FROM 
    Cuenta c
JOIN 
    Cliente cli ON c.id_cliente = cli.id_cliente
LEFT JOIN 
    Tarjeta t ON c.num_cuenta = t.num_cuenta
WHERE 
	t.fecha_expiracion < CURRENT_DATE ;

```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql
SELECT 
    c.num_cuenta,
    c.id_cliente,
    cli.nombre,
    c.tipo_cuenta,
    c.saldo,
    c.fecha_apertura,
    t.fecha_expiracion,
    CURRENT_DATE -30 as fecha
FROM 
    Cuenta c
JOIN 
    Cliente cli ON c.id_cliente = cli.id_cliente
LEFT JOIN 
    Tarjeta t ON c.num_cuenta = t.num_cuenta
WHERE 
	t.fecha_expiracion < CURRENT_DATE ;

SELECT 
    c.tipo_cuenta,
    ROUND(AVG(c.saldo), 2) AS saldo_promedio
FROM 
    Cuenta c
JOIN 
    Transaccion t ON c.num_cuenta = t.num_cuenta
WHERE 
    t.fecha >= CURRENT_DATE - 30
GROUP BY 
    c.tipo_cuenta;

```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql

SELECT DISTINCT cli.id_cliente, cli.nombre, cli.cedula
FROM Cliente cli
JOIN Cuenta c ON cli.id_cliente = c.id_cliente
JOIN Transaccion t ON c.num_cuenta = t.num_cuenta
JOIN Transferencia tf ON t.id_transaccion = tf.id_transaccion
WHERE cli.id_cliente NOT IN (
    SELECT DISTINCT cli2.id_cliente
    FROM Cliente cli2
    JOIN Cuenta c2 ON cli2.id_cliente = c2.id_cliente
    JOIN Transaccion t2 ON c2.num_cuenta = t2.num_cuenta
    JOIN Retiro r ON t2.id_transaccion = r.id_transaccion
    WHERE LOWER(r.canal) = 'transferencia' 
    	and LOWER(r.canal) <> 'cajero'
);

```