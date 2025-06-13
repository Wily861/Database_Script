# 📁 Scripts SQL Profesionales para Proyectos Reales

¡Hola! Soy **Wily Duvan Villamil Rey** — Administrador de Bases de Datos e Ingeniero de Datos Junior 💻📊.
En este espacio comparto scripts que he aplicado en entornos reales, desarrollados para organizaciones de alto impacto como **Alkosto**, **BTG Pactual**, **CEIPA**, **Cardio Infantil**, **Novaventa**, **Metro de Medellín**, entre otros 🏢🚀.

Estos scripts reflejan buenas prácticas en diseño, optimización y mantenimiento de bases de datos, con enfoque en herramientas como **Oracle 18c XE**, **PostgreSQL** **SQL Server** y más.

🧠 **Mi objetivo:** proporcionar soluciones claras, eficientes y escalables, aplicables a contextos reales del mundo empresarial y académico.

---


## 🔐 Otorgamiento de privilegios en el esquema `PRDN`

```sql
-- Otorga privilegios SELECT sobre TABLAS y VISTAS del esquema PRDN
SELECT
   'GRANT SELECT ON ' || owner || '.' || object_name || ' TO "SERV_APP";' AS grant_stmt
FROM
   dba_objects
WHERE
   owner = 'PRDN'
   AND object_type IN ('TABLE', 'VIEW');

-- Otorga privilegios EXECUTE sobre PROCEDIMIENTOS del esquema PRDN
SELECT
   'GRANT EXECUTE ON ' || owner || '.' || object_name || ' TO "SERV_APP";' AS grant_stmt
FROM
   dba_objects
WHERE
   owner = 'PRDN'
   AND object_type = 'PROCEDURE';
```

---

## 👨‍💼 Acceso y limpieza en la base de datos `dbOzono`

```sql
-- 🧑‍💻 Crear usuario Leadpasivos
CREATE USER Leadpasivos WITH PASSWORD 'h8A2WuDS';

-- 🔑 Permitir conexión a la base de datos "dbOzono"
GRANT CONNECT ON DATABASE "dbOzono" TO "usrDivisasRead";

-- 📦 Otorgar uso sobre el esquema flujodivisasgiros
GRANT USAGE ON SCHEMA flujodivisasgiros TO "usrDivisasRead";

-- 👁️ Otorgar permiso de lectura sobre todas las tablas del esquema
GRANT SELECT ON ALL TABLES IN SCHEMA "flujodivisasgiros" TO "usrDivisasRead";

-- 🧹 Eliminar función ya no requerida
DROP FUNCTION IF EXISTS flujos_cdt.registrotipo1(VARCHAR);
```

---

## 📌 Identificación de sesiones bloqueadas en SQL Server

```sql
SELECT
    L.request_session_id AS [ID de sesión bloqueada],
    L.resource_type AS [Tipo de recurso bloqueado],
    L.resource_database_id AS [ID de base de datos],
    DB_NAME(L.resource_database_id) AS [Nombre de base de datos],
    L.resource_description AS [Descripción del recurso bloqueado],
    L.request_mode AS [Modo de bloqueo],
    ST.text AS [Consulta bloqueada],
    AT.name AS [Nombre de la tabla bloqueada],
    P.program_name AS [Nombre del programa bloqueado]
FROM
    sys.dm_tran_locks L
JOIN
    sys.dm_exec_requests R ON L.request_session_id = R.session_id
CROSS APPLY
    sys.dm_exec_sql_text(R.sql_handle) AS ST
JOIN
    sys.dm_exec_sessions S ON R.session_id = S.session_id
JOIN
    sys.dm_tran_active_transactions AT ON R.transaction_id = AT.transaction_id
LEFT JOIN
    sys.sysprocesses P ON R.session_id = P.spid
WHERE
    L.request_session_id <> @@SPID
ORDER BY
    L.request_session_id,
    L.resource_type;
```

---

## 🛠️ Modificaciones estructurales y de formato en columnas


```sql
-- Alterar temporalmente la columna 'fecha_fin_poliza' a tipo VARCHAR(8)
ALTER TABLE dbo.Temp_Hdi_Generales_ETL
ALTER COLUMN fecha_fin_poliza VARCHAR(8);
```

```sql
-- Convertir los datos de la columna 'fecha_fin_poliza' al formato 'YYYYMMDD'
UPDATE dbo.Temp_Hdi_Generales_ETL
SET fecha_fin_poliza = CONVERT(VARCHAR(8), CONVERT(DATE, fecha_fin_poliza), 112);
```

```sql
-- Alterar la columna 'fecha_fin_poliza' nuevamente a tipo NUMERIC(8, 0)
ALTER TABLE dbo.Temp_Hdi_Generales_ETL
ALTER COLUMN fecha_fin_poliza NUMERIC(8, 0);
```

```sql
-- Cambiar el tipo de la columna 'FechaFin' a DATETIME
ALTER TABLE cAfiliadoHDI
ALTER COLUMN FechaFin DATETIME;
```

```sql
-- Convertir los datos de la columna 'FechaFin' al tipo DATETIME
-- Asumiendo que los valores están en formato 'YYYYMMDD'
UPDATE cAfiliadoHDI
SET FechaFin = TRY_CONVERT(DATETIME, FechaFin);
```
---
## 🧹 Limpieza e Inicialización de Campos de Error

```sql
-- Inicializar la columna 'DESCERROR' donde su valor sea 'Ok'
UPDATE [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL]
SET DESCERROR = NULL
WHERE DESCERROR = 'Ok';
```

```sql
--Inicializar columnas 'DESCERROR' y 'NUMERROR' con valores vacíos
UPDATE t1 
SET DESCERROR = ''
FROM [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL] t1;

UPDATE t1 
SET NUMERROR = ''
FROM [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL] t1;
```
---
## 🗑️ Eliminación de Registros Vacíos o Nulos Masivos

 ```sql
-- Eliminar registros saltos de linea
delete t1
from [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL] t1 where 
    t1.codigo_plan_negocio_especial is null and
    t1.cod_sucursal is null and
    t1.cod_ramo is null and
    t1.ramo is null and
    t1.poliza is null and
    t1.certificado is null and
    t1.fecha_inicio_poliza is null and
    t1.fecha_fin_poliza is null and
    t1.nro_identificacion_tomador is null and
    t1.nombre_tomador is null and
    t1.nro_identificacion_asegurado is null and
    t1.nombre_asegurado is null and
    t1.amparo_garantia is null and
    t1.descripcion_amparo_garantia is null and
    t1.Placa is null and
    t1.modelo is null and
    t1.marca is null and
    t1.clase is null and
    t1.tipo_de_vehiculo is null and
    t1.chasis is null and
    t1.direccion_riesgo is null and
    t1.departamento_riesgo is null and
    t1.ciudad_riesgo is null and
    t1.intermediario is null;
```
---

## 📏 Validación de Longitudes de Campos
```sql
-- codigo_plan_negocio_especial
IF EXISTS (SELECT 1 
           FROM [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL] t1 
           WHERE LEN(CAST(t1.codigo_plan_negocio_especial AS VARCHAR)) > 4)
BEGIN
    UPDATE [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL]
    SET codigo_plan_negocio_especial = CAST(LEFT(CAST(codigo_plan_negocio_especial AS VARCHAR), 4) AS NUMERIC)
    WHERE LEN(CAST(codigo_plan_negocio_especial AS VARCHAR)) > 4
END
```

```sql
-- poliza
IF EXISTS (SELECT 1 
           FROM [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL] t1 
           WHERE LEN(CAST(t1.poliza AS VARCHAR(50))) > 50)
BEGIN
    UPDATE [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL]
    SET poliza = CAST(LEFT(CAST(poliza AS VARCHAR(50)), 50) AS NUMERIC)
    WHERE LEN(CAST(poliza AS VARCHAR(50))) > 50
END
```

```sql
-- certificado
IF EXISTS (SELECT 1 
           FROM [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL] t1 
           WHERE LEN(t1.certificado) > 50)
BEGIN
    UPDATE [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL]
    SET certificado = SUBSTRING(certificado, 1, 50)
    WHERE LEN(certificado) > 50
END
```

```sql
 -- fecha_inicio_poliza
IF EXISTS (SELECT 1 
           FROM [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL] 
           WHERE LEN(CAST(fecha_inicio_poliza AS VARCHAR(8))) > 8)
BEGIN
    UPDATE [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL]
    SET fecha_inicio_poliza = CAST(SUBSTRING(CAST(fecha_inicio_poliza AS VARCHAR(8)), 1, 8) AS NUMERIC)
    WHERE LEN(CAST(fecha_inicio_poliza AS VARCHAR(8))) > 8
END
```

```sql
-- fecha_fin_poliza
IF EXISTS (SELECT 1 
           FROM [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL] 
           WHERE LEN(CAST(fecha_fin_poliza AS VARCHAR(8))) > 8)
BEGIN
    UPDATE [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL]
    SET fecha_fin_poliza = CAST(SUBSTRING(CAST(fecha_fin_poliza AS VARCHAR(8)), 1, 8) AS NUMERIC)
    WHERE LEN(CAST(fecha_fin_poliza AS VARCHAR(8))) > 8
END
```

```sql
-- Placa
IF EXISTS (SELECT 1 
           FROM [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL] 
           WHERE LEN(Placa) > 50)
BEGIN
    UPDATE [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL]
    SET Placa = SUBSTRING(Placa, 1, 50)
    WHERE LEN(Placa) > 50
END
```
---

## 🗂️ Formato de los campos

```sql
-- codigo_plan_negocio_especial 
IF((SELECT COUNT(*) FROM [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL] t1 WHERE (ISNUMERIC(t1.codigo_plan_negocio_especial) = 0) ) > 0)
BEGIN 
    UPDATE t1 
    SET t1.NUMERROR = 1, t1.DESCERROR = DESCERROR + 'motivo: formato incorrecto' 
    FROM [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL] t1 WHERE ISNUMERIC(t1.codigo_plan_negocio_especial) = 0
END
```

```sql
-- poliza 
IF((SELECT COUNT(*) FROM [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL] t1  WHERE (ISNUMERIC(t1.poliza) = 0) ) > 0)
BEGIN 
    UPDATE t1 
    SET t1.NUMERROR = 1,  t1.DESCERROR = DESCERROR + 'motivo: formato incorrecto' 
    FROM [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL] t1 WHERE ISNUMERIC(t1.poliza) = 0
END
```

```sql
-- certificado 
IF((SELECT COUNT(*) FROM [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL] t1  WHERE (ISNUMERIC(t1.certificado) = 0) ) > 0)
BEGIN 
    UPDATE t1 
    SET t1.NUMERROR = 1,  t1.DESCERROR = DESCERROR + 'motivo: formato incorrecto' 
    FROM [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL] t1 WHERE ISNUMERIC(t1.certificado) = 0
END
```

```sql
-- fecha_inicio_poliza 
IF((SELECT COUNT(*) FROM [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL] t1 WHERE (ISDATE(t1.fecha_inicio_poliza) = 0) )> 0)
BEGIN 
    UPDATE t1 
    SET t1.NUMERROR = 1, t1.DESCERROR = DESCERROR + 'motivo: formato incorrecto' 
    FROM [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL] t1 WHERE ISDATE(t1.fecha_inicio_poliza) = 0
END
```
---

## --- 3. Eliminación de caracteres especiales
```sql
set dateformat dmy --- para establecer el formato de fecha

## -- Primera Actualización: Eliminar salto de línea de cada campo especificado
UPDATE [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL]
SET 
    [codigo_plan_negocio_especial] = REPLACE(REPLACE(REPLACE([codigo_plan_negocio_especial], CHAR(9), ''), CHAR(10), ''), CHAR(13), ''),
    [cod_sucursal] = REPLACE(REPLACE(REPLACE([cod_sucursal], CHAR(9), ''), CHAR(10), ''), CHAR(13), ''),
    [cod_ramo] = REPLACE(REPLACE(REPLACE([cod_ramo], CHAR(9), ''), CHAR(10), ''), CHAR(13), ''),
    [ramo] = REPLACE(REPLACE(REPLACE([ramo], CHAR(9), ''), CHAR(10), ''), CHAR(13), ''),
    [poliza] = REPLACE(REPLACE(REPLACE([poliza], CHAR(9), ''), CHAR(10), ''), CHAR(13), ''),
    [certificado] = REPLACE(REPLACE(REPLACE([certificado], CHAR(9), ''), CHAR(10), ''), CHAR(13), ''),
    [fecha_inicio_poliza] = REPLACE(REPLACE(REPLACE([fecha_inicio_poliza], CHAR(9), ''), CHAR(10), ''), CHAR(13), ''),
    [fecha_fin_poliza] = REPLACE(REPLACE(REPLACE([fecha_fin_poliza], CHAR(9), ''), CHAR(10), ''), CHAR(13), ''),
    [nro_identificacion_tomador] = REPLACE(REPLACE(REPLACE([nro_identificacion_tomador], CHAR(9), ''), CHAR(10), ''), CHAR(13), ''),
    [nombre_tomador] = REPLACE(REPLACE(REPLACE([nombre_tomador], CHAR(9), ''), CHAR(10), ''), CHAR(13), ''),
    [nro_identificacion_asegurado] = REPLACE(REPLACE(REPLACE([nro_identificacion_asegurado], CHAR(9), ''), CHAR(10), ''), CHAR(13), ''),
    [nombre_asegurado] = REPLACE(REPLACE(REPLACE([nombre_asegurado], CHAR(9), ''), CHAR(10), ''), CHAR(13), ''),
    [amparo_garantia] = REPLACE(REPLACE(REPLACE([amparo_garantia], CHAR(9), ''), CHAR(10), ''), CHAR(13), ''),
    [descripcion_amparo_garantia] = REPLACE(REPLACE(REPLACE([descripcion_amparo_garantia], CHAR(9), ''), CHAR(10), ''), CHAR(13), ''),
    [direccion_riesgo] = REPLACE(REPLACE(REPLACE([direccion_riesgo], CHAR(9), ''), CHAR(10), ''), CHAR(13), ''),
    [departamento_riesgo] = REPLACE(REPLACE(REPLACE([departamento_riesgo], CHAR(9), ''), CHAR(10), ''), CHAR(13), ''),
    [ciudad_riesgo] = REPLACE(REPLACE(REPLACE([ciudad_riesgo], CHAR(9), ''), CHAR(10), ''), CHAR(13), ''),
    [intermediario] = REPLACE(REPLACE(REPLACE([intermediario], CHAR(9), ''), CHAR(10), ''), CHAR(13), '');
```
```sql
## -- Segunda Actualización: -- Este script eliminará los caracteres especiales

UPDATE [dbo].[Temp_HDI_SEGUROS_Base_AUTOS_ETL]
SET
 [codigo_plan_negocio_especial] = REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE([codigo_plan_negocio_especial], CHAR(39), ''), CHAR(34), ''), CHAR(33), ''), ',', ''),   '&', ''), '/', ''), '*', ''), '+', ''), '[', ''), ']', ''), '\\', ''), '¡', ''), '!', ''), '¿', ''), '?', ''), '$', ''), '#', ''), '%', ''), '|', ''), '°', ''), '{', ''), '}', ''), '´', ''), '(', ''), ')', ''),
[cod_sucursal] = REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE([cod_sucursal], CHAR(39), ''), CHAR(34), ''), CHAR(33), ''), ',', ''),   '&', ''), '/', ''), '*', ''), '+', ''), '[', ''), ']', ''), '\\', ''), '¡', ''), '!', ''), '¿', ''), '?', ''), '$', ''), '#', ''), '%', ''), '|', ''), '°', ''), '{', ''), '}', ''), '´', ''), '(', ''), ')', ''),
[cod_ramo] = REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE([cod_ramo], CHAR(39), ''), CHAR(34), ''), CHAR(33), ''), ',', ''),   '&', ''), '/', ''), '*', ''), '+', ''), '[', ''), ']', ''), '\\', ''), '¡', ''), '!', ''), '¿', ''), '?', ''), '$', ''), '#', ''), '%', ''), '|', ''), '°', ''), '{', ''), '}', ''), '´', ''), '(', ''), ')', ''),
[ramo] = REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE([ramo], CHAR(39), ''), CHAR(34), ''), CHAR(33), ''), ',', ''),   '&', ''), '/', ''), '*', ''), '+', ''), '[', ''), ']', ''), '\\', ''), '¡', ''), '!', ''), '¿', ''), '?', ''), '$', ''), '#', ''), '%', ''), '|', ''), '°', ''), '{', ''), '}', ''), '´', ''), '(', ''), ')', ''),
[poliza] = REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE([poliza], CHAR(39), ''), CHAR(34), ''), CHAR(33), ''), ',', ''),   '&', ''), '/', ''), '*', ''), '+', ''), '[', ''), ']', ''), '\\', ''), '¡', ''), '!', ''), '¿', ''), '?', ''), '$', ''), '#', ''), '%', ''), '|', ''), '°', ''), '{', ''), '}', ''), '´', ''), '(', ''), ')', ''),
[certificado] = REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE([certificado], CHAR(39), ''), CHAR(34), ''), CHAR(33), ''), ',', ''),   '&', ''), '/', ''), '*', ''), '+', ''), '[', ''), ']', ''), '\\', ''), '¡', ''), '!', ''), '¿', ''), '?', ''), '$', ''), '#', ''), '%', ''), '|', ''), '°', ''), '{', ''), '}', ''), '´', ''), '(', ''), ')', ''),
[fecha_inicio_poliza] = REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE([fecha_inicio_poliza], CHAR(39), ''), CHAR(34), ''), CHAR(33), ''), ',', ''),   '&', ''), '/', ''), '*', ''), '+', ''), '[', ''), ']', ''), '\\', ''), '¡', ''), '!', ''), '¿', ''), '?', ''), '$', ''), '#', ''), '%', ''), '|', ''), '°', ''), '{', ''), '}', ''), '´', ''), '(', ''), ')', ''),
[fecha_fin_poliza] = REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE([fecha_fin_poliza], CHAR(39), ''), CHAR(34), ''), CHAR(33), ''), ',', ''),   '&', ''), '/', ''), '*', ''), '+', ''), '[', ''), ']', ''), '\\', ''), '¡', ''), '!', ''), '¿', ''), '?', ''), '$', ''), '#', ''), '%', ''), '|', ''), '°', ''), '{', ''), '}', ''), '´', ''), '(', ''), ')', ''),
[nro_identificacion_tomador] = REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE([nro_identificacion_tomador], CHAR(39), ''), CHAR(34), ''), CHAR(33), ''), ',', ''),  '&', ''), '/', ''), '*', ''), '+', ''), '[', ''), ']', ''), '\\', ''), '¡', ''), '!', ''), '¿', ''), '?', ''), '$', ''), '#', ''), '%', ''), '|', ''), '°', ''), '{', ''), '}', ''), '´', ''), '(', ''), ')', ''),
[nombre_tomador] = REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE([nombre_tomador], CHAR(39), ''), CHAR(34), ''), CHAR(33), ''), ',', ''),  '&', ''), '/', ''), '*', ''), '+', ''), '[', ''), ']', ''), '\\', ''), '¡', ''), '!', ''), '¿', ''), '?', ''), '$', ''), '#', ''), '%', ''), '|', ''), '°', ''), '{', ''), '}', ''), '´', ''), '(', ''), ')', ''),
[nro_identificacion_asegurado] = REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE([nro_identificacion_asegurado], CHAR(39), ''), CHAR(34), ''), CHAR(33), ''), ',', ''),  '&', ''), '/', ''), '*', ''), '+', ''), '[', ''), ']', ''), '\\', ''), '¡', ''), '!', ''), '¿', ''), '?', ''), '$', ''), '#', ''), '%', ''), '|', ''), '°', ''), '{', ''), '}', ''), '´', ''), '(', ''), ')', ''),
[nombre_asegurado] = REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE([nombre_asegurado], CHAR(39), ''), CHAR(34), ''), CHAR(33), ''), ',', ''),  '&', ''), '/', ''), '*', ''), '+', ''), '[', ''), ']', ''), '\\', ''), '¡', ''), '!', ''), '¿', ''), '?', ''), '$', ''), '#', ''), '%', ''), '|', ''), '°', ''), '{', ''), '}', ''), '´', ''), '(', ''), ')', ''),
[amparo_garantia] = REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE([amparo_garantia], CHAR(39), ''), CHAR(34), ''), CHAR(33), ''), ',', ''),  '&', ''), '/', ''), '*', ''), '+', ''), '[', ''), ']', ''), '\\', ''), '¡', ''), '!', ''), '¿', ''), '?', ''), '$', ''), '#', ''), '%', ''), '|', ''), '°', ''), '{', ''), '}', ''), '´', ''), '(', ''), ')', ''),
[descripcion_amparo_garantia] = REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE([descripcion_amparo_garantia], CHAR(39), ''), CHAR(34), ''), CHAR(33), ''), ',', ''),   '&', ''), '/', ''), '*', ''), '+', ''), '[', ''), ']', ''), '\\', ''), '¡', ''), '!', ''), '¿', ''), '?', ''), '$', ''), '#', ''), '%', ''), '|', ''), '°', ''), '{', ''), '}', ''), '´', ''), '(', ''), ')', ''),
[direccion_riesgo] = REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE([direccion_riesgo], CHAR(39), ''), CHAR(34), ''), CHAR(33), ''), ',', ''),   '&', ''), '/', ''), '*', ''), '+', ''), '[', ''), ']', ''), '\\', ''), '¡', ''), '!', ''), '¿', ''), '?', ''), '$', ''), '#', ''), '%', ''), '|', ''), '°', ''), '{', ''), '}', ''), '´', ''), '(', ''), ')', ''),
[departamento_riesgo] = REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE([departamento_riesgo], CHAR(39), ''), CHAR(34), ''), CHAR(33), ''), ',', ''),  '&', ''), '/', ''), '*', ''), '+', ''), '[', ''), ']', ''), '\\', ''), '¡', ''), '!', ''), '¿', ''), '?', ''), '$', ''), '#', ''), '%', ''), '|', ''), '°', ''), '{', ''), '}', ''), '´', ''), '(', ''), ')', ''),
[ciudad_riesgo] = REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE([ciudad_riesgo], CHAR(39), ''), CHAR(34), ''), CHAR(33), ''), ',', ''),   '&', ''), '/', ''), '*', ''), '+', ''), '[', ''), ']', ''), '\\', ''), '¡', ''), '!', ''), '¿', ''), '?', ''), '$', ''), '#', ''), '%', ''), '|', ''), '°', ''), '{', ''), '}', ''), '´', ''), '(', ''), ')', ''),
[intermediario] = REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE([intermediario], CHAR(39), ''), CHAR(34), ''), CHAR(33), ''), ',', ''),  '&', ''), '/', ''), '*', ''), '+', ''), '[', ''), ']', ''), '\\', ''), '¡', ''), '!', ''), '¿', ''), '?', ''), '$', ''), '#', ''), '%', ''), '|', ''), '°', ''), '{', ''), '}', ''), '´', ''), '(', ''), ')', '');
```
