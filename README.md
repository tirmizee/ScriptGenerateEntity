# GENERATE SCRIPT 
	ALTER PROCEDURE [dbo].[generateRepository]   
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
