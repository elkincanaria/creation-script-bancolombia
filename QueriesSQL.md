# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql
SELECT CLT.NOMBRE, CLT.CEDULA, CTA.ID_CLIENTE, COUNT(CTA.NUM_CUENTA) AS NUM_CUENTAS,SUM(CTA.SALDO) AS SALDO_TOTAL  
FROM CLIENTE CLT
INNER JOIN CUENTA CTA ON CLT.ID_CLIENTE=CTA.ID_CLIENTE
GROUP BY CLT.NOMBRE, CLT.CEDULA, CTA.ID_CLIENTE
HAVING COUNT(CTA.NUM_CUENTA)>1 
ORDER BY SUM(CTA.SALDO)
```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql
SELECT CLT.NOMBRE, CLT.CEDULA, CTA.ID_CLIENTE, 
SUM(CASE WHEN TIPO_TRANSACCION='deposito' THEN TRX.MONTO ELSE 0 END) AS TOTAL_DEP,
SUM(CASE WHEN TIPO_TRANSACCION='retiro' THEN TRX.MONTO ELSE 0 END) AS TOTAL_RET  
FROM CLIENTE CLT
INNER JOIN CUENTA CTA ON CLT.ID_CLIENTE=CTA.ID_CLIENTE
INNER JOIN TRANSACCION TRX ON TRX.NUM_CUENTA=CTA.NUM_CUENTA
GROUP BY CLT.NOMBRE, CLT.CEDULA, CTA.ID_CLIENTE
ORDER BY ID_CLIENTE
```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql
SELECT CLT.NOMBRE, CLT.CEDULA, CTA.ID_CLIENTE, CTA.NUM_CUENTA
FROM CLIENTE CLT
INNER JOIN CUENTA CTA ON CLT.ID_CLIENTE=CTA.ID_CLIENTE
LEFT JOIN TARJETA TRJ ON CTA.NUM_CUENTA=TRJ.NUM_CUENTA
WHERE TRJ.NUM_CUENTA IS NULL
```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql
SELECT CTA.TIPO_CUENTA, AVG(CTA.SALDO) AS SALDO_PROM
FROM CUENTA CTA
WHERE CTA.NUM_CUENTA IN 
   (SELECT DISTINCT TRX.NUM_CUENTA
    FROM TRANSACCION TRX
    WHERE TRX.FECHA >= CURRENT_DATE - INTERVAL '30 days')
GROUP BY CTA.TIPO_CUENTA;
```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql
SELECT DISTINCT CLT.NOMBRE, CLT.CEDULA FROM CLIENTE CLT
INNER JOIN CUENTA CTA ON CLT.ID_CLIENTE=CTA.ID_CLIENTE
INNER JOIN TRANSACCION TRX ON CTA.NUM_CUENTA=TRX.NUM_CUENTA
INNER JOIN RETIRO RTR ON TRX.ID_TRANSACCION=RTR.ID_TRANSACCION
WHERE NOMBRE NOT IN(
SELECT CLT.NOMBRE FROM CLIENTE CLT
INNER JOIN CUENTA CTA ON CLT.ID_CLIENTE=CTA.ID_CLIENTE
INNER JOIN TRANSACCION TRX ON CTA.NUM_CUENTA=TRX.NUM_CUENTA
INNER JOIN RETIRO RTR ON TRX.ID_TRANSACCION=RTR.ID_TRANSACCION
WHERE RTR.CANAL='cajero')
```
