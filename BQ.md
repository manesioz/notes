Biggest things to consider:
1. It does not have Primary Keys! This means that there may be repeated rows.
2. Be careful when using the LIMIT clause in a sub-query (since there are no PK's and BQ randomly chooses the rows if no ORDER BY clause is given, each time you run it you can get different records.
3. When using LAST_VALUE in window/analytic functions, make sure to specify the window range
4. User-defined functions (UDFs) can be used to perform more complex aggregations/calculations, which are written in SQL or vanilla JS
5. Rarely use "SELECT *" since it is costly and usually unnecessary
6. If the table is date sharded, ALWAYS use _TABLE_SUFFIX when limiting the queries range, and if the table is partitioned, ALWAYS use _PARTITIONTIME when limiting queries range

NOTE: in Apache Airflow there is no Standard SQL support for BigQueryCheck() operators (only legacy SQL)


Examples

//Chained sub-queries
WITH VoltageValues AS (
	SELECT HadwareId, Voltage
	FROM DebugTable
	WHERE HardwareId IN (SELECT DISTINCT HardwareId FROM DebugTable ORDER BY HardwareId LIMIT 50)
)
, min_values AS (
	SELECT HardwareId, MIN(Voltage) AS MinVoltage
	FROM VoltageValues
	GROUP BY HardwareId
)
, max_values AS (
	SELECT HardwareId, MAX(Voltage) AS MaxVoltage
	FROM VoltageValues
	GROUP BY HardwareId
)

SELECT min_values.HardwareId, Voltage, MinVoltage, MaxVoltage
FROM min_values
JOIN max_values
ON min_values.HardwareId = max_values.HardwareId;