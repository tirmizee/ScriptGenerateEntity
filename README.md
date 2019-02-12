# Sample script generate code from database

# SCRIPT GENERATE FOR MSSQL
	CREATE PROCEDURE [dbo].[generateRepository]   
	@table nvarchar(50)   
	AS   

	SET NOCOUNT ON;  
	-- TABLE ENTITY
	SELECT 
	'private ' + 
	CASE LOWER(LEFT(t.Name ,1))+SUBSTRING(t.Name ,2,LEN(t.Name ))
	  when 'nvarchar'   then 'String'
	  when 'varchar'    then 'String'
	  when 'int'        then 'Integer'
	  when 'bigint'     then 'Long'
	  when 'tinyint'    then 'Boolean'
	  when 'datetime'   then 'java.sql.Date'
	  when 'timestamp'  then 'java.sql.Timestamp'
	  when 'numeric'    then 'BigDecimal'
	  when 'float'      then 'Double'
	  when 'real'       then 'Float'
	  when 'decimal'    then 'BigDecimal'
	  ELSE ''
	END + ' ' + c.name +';' AS Entity
	FROM    
	  sys.columns c
	INNER JOIN 
	  sys.types t ON c.user_type_id = t.user_type_id
	LEFT OUTER JOIN 
	  sys.index_columns ic ON ic.object_id = c.object_id AND ic.column_id = c.column_id
	LEFT OUTER JOIN 
	  sys.indexes i ON ic.object_id = i.object_id AND ic.index_id = i.index_id
	WHERE
	  c.object_id = OBJECT_ID(@table);

	-- TABLE COLUMN
	SELECT 'public static final String TABLE_' + UPPER(@table) + ' = "' + @table + '";' AS [Column]
	UNION ALL
	SELECT 'public static final String COL_' + UPPER (c.Name) + ' = "' +  c.Name + '";' AS [Column]
	FROM sys.columns c
		JOIN sys.objects o ON o.object_id = c.object_id
	WHERE o.object_id = OBJECT_ID(@table)
	UNION ALL
	SELECT '' AS [Column]
	UNION ALL
	SELECT 'public static final String ' + UPPER (c.name) + ' = "' + o.name + '.' + c.Name + '";' AS [Column]
	FROM sys.columns c 
		JOIN sys.objects o ON o.object_id = c.object_id
	WHERE o.object_id = OBJECT_ID(@table);

	-- TABLE MAP COLUMN
	SELECT 'map.put(COL_' + UPPER (c.Name) + ' , param.get' +  c.Name + '());' AS MapColumn
	FROM sys.columns c
		JOIN sys.objects o ON o.object_id = c.object_id
	WHERE o.object_id = OBJECT_ID(@table);

	-- TABLE RESULT SET
	SELECT 
	@table + '.set' + c.name + '(rs.get' +
	CASE t.Name 
		when 'nvarchar'   then 'String'
		when 'varchar'    then 'String'
		when 'int'        then 'Int'
		when 'bigint'     then 'Long'
		when 'tinyint'    then 'Boolean'
		when 'datetime'   then 'Date'
		when 'timestamp'  then 'Timestamp'
		when 'numeric'    then 'BigDecimal'
		when 'decimal'    then 'BigDecimal'
		when 'float'      then 'Double'
		when 'real'       then 'Float'
		ELSE ''
	END + '(COL_' + upper(c.name) + '));' AS ResultSet
	FROM    
		sys.columns c
	INNER JOIN 
		sys.types t ON c.user_type_id = t.user_type_id
	LEFT OUTER JOIN 
		sys.index_columns ic ON ic.object_id = c.object_id AND ic.column_id = c.column_id
	LEFT OUTER JOIN 
		sys.indexes i ON ic.object_id = i.object_id AND ic.index_id = i.index_id
	WHERE
		c.object_id = OBJECT_ID(@table);
# SCRIPT GENERATE FOR MYSQL
	CREATE DEFINER=`root`@`localhost` PROCEDURE `generateRepository`(IN tableName VARCHAR(255))
	BEGIN
		SELECT 
			concat(
				'public static final String TABLE_', 
		    UPPER (table_name),
		    ' = "',
		    table_name,
		    '";' 
			) AS ColumnNane 
		FROM information_schema.columns where table_name = tableName
		UNION
		SELECT 
			concat(
				'public static final String COL_', 
		    UPPER (column_name),
		    ' = "',
		    column_name,
		    '";' 
			) AS ColumnNane 
		FROM information_schema.columns where table_name = tableName
	    UNION
		SELECT 
			concat(
				'public static final String ', 
		    UPPER (column_name),
		    ' = "',
		    table_name,
		    '.',
		    column_name,
		    '";' 
			) AS ColumnNane 
		FROM information_schema.columns where table_name = tableName;

	    SELECT 
			concat(
				'private ' , 
		    CASE DATA_TYPE
					WHEN 'varchar'     THEN 'String'
			WHEN 'bigint'      THEN 'Long'
					WHEN 'datetime'    THEN 'Timestamp'
			WHEN 'date'        THEN 'java.sql.Date'
			ELSE 'String'
		    END ,
		    ' ' ,
		    column_name ,
		    ';'
			) AS Entity 
		FROM information_schema.columns where table_name = tableName;

	    SELECT 
			concat(
				'map.put(COL_', 
		    UPPER (column_name),
		    ', param.get',
		    CONCAT(UCASE(LEFT(column_name, 1)),SUBSTRING(column_name, 2)), 
		    '());'
			) AS MapColumn
	    FROM information_schema.columns where table_name = tableName;

	     SELECT 
			CONCAT(
				CONCAT(LCASE(LEFT(table_name, 1)),SUBSTRING(table_name, 2)) ,
		    '.set' ,
		    CONCAT(UCASE(LEFT(column_name, 1)),SUBSTRING(column_name, 2)),
		    '(rs.get',
				CASE DATA_TYPE
					WHEN 'varchar'     THEN 'String'
			WHEN 'bigint'      THEN 'Long'
					WHEN 'datetime'    THEN 'Timestamp'
			WHEN 'date'        THEN 'Date'
			ELSE 'String'
		    END ,
		    '(COL_',
		    UPPER (column_name),
		    '));'
		) AS ResultSet
	    FROM information_schema.columns where table_name = tableName;

	END
	
# SCRIPT GENERATE FOR DB2/AS400
	SELECT 
	 'public static final String TABLE_'
	 CONCAT 
	 UPPER (TABLE_NAME) 
	 CONCAT
	 ' = "' 
	 CONCAT
	 TABLE_NAME 
	 CONCAT
	 '";' 
	 AS ColumnNane 
	FROM sysibm.sqlcolumns WHERE TABLE_NAME = 'VPRCPF' AND TABLE_SCHEM = 'TDEVDTA'
	UNION
	SELECT 
		 'public static final String COL_'
		 CONCAT 
		 UPPER (COLUMN_NAME) 
		 CONCAT
		 ' = "' 
		 CONCAT
		 COLUMN_NAME 
		 CONCAT
		 '";' 
		 AS ColumnNane 
	FROM sysibm.sqlcolumns WHERE TABLE_NAME = 'VPRCPF' AND TABLE_SCHEM = 'TDEVDTA'
	UNION
	SELECT 
		'public static final String '
		CONCAT 
		UPPER (COLUMN_NAME)
		CONCAT
		' = "'
		CONCAT
		TABLE_NAME
		CONCAT
		'.'
		CONCAT
		COLUMN_NAME
		CONCAT
		'";' 
		 AS ColumnNane 
	FROM sysibm.sqlcolumns WHERE TABLE_NAME = 'VPRCPF' AND TABLE_SCHEM = 'TDEVDTA';
	-- SELECT * FROM sysibm.sqlcolumns WHERE TABLE_NAME = 'VPRCPF' AND TABLE_SCHEM = 'TDEVDTA';
	SELECT 
		'private ' 
		CONCAT 
		CASE LOWER(TYPE_NAME)
	    WHEN 'varchar'     THEN 'String'
	    WHEN 'char'        THEN 'String'
			WHEN 'bigint'      THEN 'Long'
	    WHEN 'decimal'     THEN 'BigDecimal'
			WHEN 'datetime'    THEN 'java.sql.Date'
	    WHEN 'timestamp'   THEN 'Timestamp'
			WHEN 'date'        THEN 'Date'
			ELSE 'String'
		END 
		CONCAT
		' ' 
		CONCAT
		COLUMN_NAME 
		CONCAT
		';'
		AS Entity 
	FROM sysibm.sqlcolumns WHERE TABLE_NAME = 'VPRCPF' AND TABLE_SCHEM = 'TDEVDTA';
	-- SELECT * FROM sysibm.sqlcolumns WHERE TABLE_NAME = 'VPRCPF' AND TABLE_SCHEM = 'TDEVDTA';
	SELECT 
		'map.put(COL_'
		CONCAT 
		UPPER (COLUMN_NAME)
		CONCAT
		', param.get'
		CONCAT
		CONCAT(UCASE(LEFT(COLUMN_NAME, 1)),SUBSTRING(COLUMN_NAME, 2))
		CONCAT 
		'());'
		AS MapColumn
	FROM sysibm.sqlcolumns WHERE TABLE_NAME = 'VPRCPF' AND TABLE_SCHEM = 'TDEVDTA';
	-- SELECT * FROM sysibm.sqlcolumns WHERE TABLE_NAME = 'VPRCPF' AND TABLE_SCHEM = 'TDEVDTA';
	SELECT 
		CONCAT(LCASE(LEFT(TABLE_NAME, 1)),SUBSTRING(TABLE_NAME, 2)) 
		CONCAT
		'.set' 
		CONCAT
		CONCAT(UCASE(LEFT(COLUMN_NAME, 1)),SUBSTRING(COLUMN_NAME, 2))
		CONCAT
		'(rs.get'
		CONCAT
		CASE LOWER(TYPE_NAME)
			WHEN 'varchar'     THEN 'String'
	    WHEN 'char'        THEN 'String'
			WHEN 'bigint'      THEN 'Long'
	    WHEN 'decimal'     THEN 'BigDecimal'
			WHEN 'datetime'    THEN 'Timestamp'
	    WHEN 'timestamp'   THEN 'Timestamp'
			WHEN 'date'        THEN 'Date'
			ELSE 'String'
		END 
		CONCAT
		'(COL_'
		CONCAT
		UPPER (COLUMN_NAME)
		CONCAT
		'));'
		AS ResultSet
	FROM sysibm.sqlcolumns WHERE TABLE_NAME = 'VPRCPF' AND TABLE_SCHEM = 'TDEVDTA';
	
# SCRIPT GENERATE FOR ORACLE
	CREATE VIEW GENERATE_REPO AS
	SELECT 
    	'#########################COLUMNS#########################' AS GEBERATE,
	    table_name
	FROM all_tables
	UNION ALL
	SELECT 
	'public static final String TB_' || upper(table_name) || ' = "' ||table_name || '";'  AS GEBERATE,
	table_name
	FROM all_tables 
	UNION ALL
	SELECT 
	    'public static final String COL_' || upper(cols.column_name) || ' = "' || cols.column_name || '";' AS GEBERATE,
	    tb.table_name
	FROM all_tables tb 
	INNER JOIN all_tab_columns cols ON tb.table_name = cols.table_name
	UNION ALL
	SELECT 
	    'public static final String ' || upper(cols.column_name) || ' = "' ||tb.table_name || '.' || cols.column_name || '";' AS GEBERATE,
	    tb.table_name
	FROM all_tables tb 
	INNER JOIN all_tab_columns cols ON tb.table_name = cols.table_name
	UNION ALL
	SELECT 
	    '#########################ENTITY#########################' AS GEBERATE,
	    table_name
	FROM all_tables
	UNION ALL
	SELECT 
	    (
		'private ' ||
		CASE cols.data_type
		    WHEN 'NUMBER'    THEN 'Integer'
		    WHEN 'VARCHAR'   THEN 'String'
		    WHEN 'VARCHAR2'  THEN 'String'
		WHEN 'DATE'      THEN 'java.sql.Date'
		END || ' ' ||
		lower(cols.column_name) || ';'

	    ) AS GEBERATE,
	    tb.table_name
	FROM all_tables tb
	INNER JOIN all_tab_columns cols ON tb.table_name = cols.table_name
	UNION ALL
	SELECT 
	    '#########################RESULT_SET#########################' AS GEBERATE,
	    table_name
	FROM all_tables
	UNION ALL
	SELECT 
	    (
		LOWER(cols.table_name) || '.set' || INITCAP(cols.column_name) || '(rs.get' ||
		CASE cols.data_type
		    WHEN 'NUMBER'    THEN 'Int'
		    WHEN 'VARCHAR'   THEN 'String'
		    WHEN 'VARCHAR2'  THEN 'String'
		END  || 
		'(COL_' || upper(cols.column_name) || '));'
	    ) AS GEBERATE,
	    tb.table_name
	FROM all_tables tb
	INNER JOIN all_tab_columns cols ON tb.table_name = cols.table_name
	UNION ALL
	SELECT 
	    '#########################MAP#########################' AS GEBERATE,
	    table_name
	FROM all_tables
	UNION ALL
	SELECT 
	    'map.put(COL_' || upper(cols.column_name) || ', param.get' ||  INITCAP(cols.column_name) || '());'  AS GEBERATE,
	    tb.table_name
	FROM all_tables tb
	INNER JOIN all_tab_columns cols ON tb.table_name = cols.table_name
