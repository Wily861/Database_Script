**📁 Scripts SQL Profesionales para Proyectos Reales**

¡Hola! Soy Wily Duvan Villamil Rey — Administrador de Bases de Datos e Ingeniero de Datos Junior 💻📊. 
En este espacio comparto scripts que he aplicado en entornos reales, desarrollados para organizaciones de alto impacto como **Alkosto**, **BTG Pactual**, **CEIPA**, **Cardio Infantil**, **Novaventa**, **Metro de Medellín**, entre otros 🏢🚀.

Estos scripts reflejan buenas prácticas en diseño, optimización y mantenimiento de bases de datos, con enfoque en herramientas como Oracle 18c XE, PostgreSQL y más.

🧠 **Mi objetivo:** proporcionar soluciones claras, eficientes y escalables, aplicables a contextos reales del mundo empresarial y académico.

---

| Script                          | Descripción                                                  | Base de datos | Versión |
|---------------------------------|---------------------------------------------------------------|----------------|----------|
| `grant_prdn_privileges.sql`     | Otorga GRANT SELECT y EXECUTE en objetos del esquema PRDN     | Oracle         | 1.0      |


--- **Otorga privilegios SELECT sobre TABLAS y VISTAS del esquema PRDN**

SELECT 
   'GRANT SELECT ON ' || owner || '.' || object_name || ' TO "SERV_APP";' AS grant_stmt
FROM dba_objects
WHERE owner = 'PRDN'
  AND object_type IN ('TABLE', 'VIEW')

UNION


--- **Otorga privilegios EXECUTE sobre PROCEDIMIENTOS del esquema PRDN**

SELECT 
   'GRANT EXECUTE ON ' || owner || '.' || object_name || ' TO "SERV_APP";' AS grant_stmt
FROM dba_objects
WHERE owner = 'PRDN'
  AND object_type = 'PROCEDURE';

---

| Script Name                    | Autor                       | Descripción                                                                                   | Base de Datos | Versión |
|-------------------------------|-----------------------------|-----------------------------------------------------------------------------------------------|----------------|---------|
| dbOzono_access_and_cleanup.sql | Wily Duvan Villamil Rey     | Crea el usuario `Leadpasivos`, asigna privilegios a `usrDivisasRead` y elimina una función. | PostgreSQL     | 1.0     |

-- **🧑‍💻 Crear usuario Leadpasivos**

CREATE USER Leadpasivos WITH PASSWORD 'h8A2WuDS';

-- **🔑 Permitir conexión a la base de datos "dbOzono"**

GRANT CONNECT ON DATABASE "dbOzono" TO "usrDivisasRead";

-- **📦 Otorgar uso sobre el esquema flujodivisasgiros**

GRANT USAGE ON SCHEMA flujodivisasgiros TO "usrDivisasRead";

-- **👁️ Otorgar permiso de lectura sobre todas las tablas del esquema**

GRANT SELECT ON ALL TABLES IN SCHEMA "flujodivisasgiros" TO "usrDivisasRead";

-- **🧹 Eliminar función ya no requerida**

DROP FUNCTION IF EXISTS flujos_cdt.registrotipo1(VARCHAR);

---

| Script Name                     | Autor                       | Descripción                                                                                   | Base de Datos | Versión |
|---------------------------------|-----------------------------|-----------------------------------------------------------------------------------------------|--------------|---------|
| session_lock_analysis.sql       | Wily Duvan Villamil Rey     | Identifica sesiones bloqueadas en SQL Server, mostrando detalles del recurso y consulta afectada. | SQL Server   | 1.0     |



-- 📌 Identificar sesiones bloqueadas en SQL Server
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
    JOIN sys.dm_exec_requests R ON L.request_session_id = R.session_id
    CROSS APPLY sys.dm_exec_sql_text(R.sql_handle) AS ST
    JOIN sys.dm_exec_sessions S ON R.session_id = S.session_id
    JOIN sys.dm_tran_active_transactions AT ON R.transaction_id = AT.transaction_id
    LEFT JOIN sys.sysprocesses P ON R.session_id = P.spid
WHERE  
    L.request_session_id <> @@SPID
ORDER BY  
    L.request_session_id, L.resource_type;


  
