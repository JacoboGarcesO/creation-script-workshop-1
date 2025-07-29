# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql
SELECT
    cli.id_cliente,
    cli.nombre,
    COUNT(cta.id_cliente) AS numero_de_cuentas,
    SUM(cta.saldo) as saldototal
FROM
    Cliente cli
JOIN
    Cuenta cta ON cli.id_cliente = cta.id_cliente
GROUP BY
    cli.id_cliente, cli.nombre
HAVING COUNT(cta.num_cuenta) > 1
ORDER BY
    saldototal DESC;
```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql
--- Version humana Juandiego Muñoz Ospina Punto 2

SELECT
	cli.id_cliente,
	cli.nombre,
    SUM(t_deposito.monto_total_deposito) AS monto_total_depositos_por_cuenta,
    SUM(r_retirado.valor_retirado) AS monto_total_retirado
FROM
    cliente cli
JOIN
    cuenta c ON cli.id_cliente = c.id_cliente
LEFT JOIN (
    SELECT
        tran.num_cuenta,
        SUM(tran.monto) AS monto_total_deposito
    FROM
        transaccion tran
    INNER JOIN
        deposito d ON d.id_transaccion = tran.id_transaccion
    GROUP BY
        tran.num_cuenta
) AS t_deposito ON c.num_cuenta = t_deposito.num_cuenta

LEFT JOIN(
	SELECT
        t.num_cuenta, SUM(t.monto) AS valor_retirado
    FROM
        transaccion t
    INNER JOIN
        retiro r ON r.id_transaccion = t.id_transaccion
        GROUP by t.num_cuenta
        )
        AS r_retirado ON c.num_cuenta = r_retirado.num_cuenta
GROUP BY
     cli.id_cliente,cli.nombre;


--- version mejorada por ia puntos 2 y comprendida por Juan diego Muñoz Ospina
SELECT
    c.id_cliente,
    c.nombre,
    COALESCE(SUM(CASE WHEN t.tipo_transaccion = 'deposito' THEN t.monto ELSE 0 END), 0) AS total_depositos,
    COALESCE(SUM(CASE WHEN t.tipo_transaccion = 'retiro' THEN t.monto ELSE 0 END), 0) AS total_retiros
FROM Cliente c
JOIN Cuenta cta ON c.id_cliente = cta.id_cliente
LEFT JOIN Transaccion t ON cta.num_cuenta = t.num_cuenta
GROUP BY c.id_cliente, c.nombre
ORDER BY c.nombre;
```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql
--- Version humana Juandiego Muñoz Ospina Punto 3

SELECT c.num_cuenta FROM cuenta c

EXCEPT

SELECT t.num_cuenta FROM tarjeta t

---version con ayuda ia puntos 3 comprendida por Juan Diego Muñoz Ospina
SELECT
    cta.num_cuenta,
    cli.nombre AS nombre_titular,
    cta.tipo_cuenta
FROM Cuenta cta
JOIN Cliente cli ON cta.id_cliente = cli.id_cliente
LEFT JOIN Tarjeta t ON cta.num_cuenta = t.num_cuenta
WHERE t.num_cuenta IS NULL;
```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql
--- Version humana Juandiego Muñoz Ospina Punto 4

SELECT
    cta.tipo_cuenta,
    COUNT(cta.num_cuenta) AS cantidad_cuentas_activas,
    AVG(cta.saldo) AS saldo_promedio
FROM
    Cuenta cta
WHERE EXISTS (
    SELECT 1 FROM Transaccion t WHERE t.num_cuenta = cta.num_cuenta AND t.fecha >= ('2023-04-15'::timestamp - INTERVAL '30 days') --- esta subconsulta require apoyo en fuentes externas y las transaciones están datadas en el 2023, por lo que resto 30 días a la que se supone fecha actual AA JUAN DIEGO M

)--- buen dato conocer este tipo de complejidad, No lo vimos en clase.
GROUP BY cta.tipo_cuenta;
```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql
SELECT c.nombre FROM cliente c
INNER JOIN cuenta cu ON cu.id_cliente = c.id_cliente
INNER JOIN 
(select t.num_cuenta from transaccion t
INNER JOIN retiro r ON r.id_transaccion = t.id_transaccion
WHERE t.descripcion <> 'Retiro en cajero'
GROUP BY t.num_cuenta) AS transa_sin_retiros ON transa_sin_retiros.num_cuenta = cu.num_cuenta
GROUP BY c.nombre; 
```