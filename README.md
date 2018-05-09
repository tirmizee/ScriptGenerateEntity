# Generate Attribute Entity 
    USE schema_name;  
    GO  
    CREATE PROCEDURE generateAttributeEntity   
    @table nvarchar(50)   
    AS   

    SET NOCOUNT ON;  

    SELECT 

    'private ' + 
    CASE t.Name 
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
      ELSE ''
    END + ' ' + c.name +';'

    FROM    
      sys.columns c
    INNER JOIN 
      sys.types t ON c.user_type_id = t.user_type_id
    LEFT OUTER JOIN 
      sys.index_columns ic ON ic.object_id = c.object_id AND ic.column_id = c.column_id
    LEFT OUTER JOIN 
      sys.indexes i ON ic.object_id = i.object_id AND ic.index_id = i.index_id
    WHERE
      c.object_id = OBJECT_ID(@table) 
      
    GO  

# Generate Column 
    USE schema_name;  
    GO  
    CREATE PROCEDURE generateColumn   
    @table nvarchar(50)   
    AS   

    SET NOCOUNT ON;  
	
	SELECT 'public static final String TABLE_' + UPPER(@table) + ' = "' + @table + '";'

	UNION ALL

	SELECT 'public static final String COL_' + UPPER (c.Name) + ' = "' +  c.Name + '";'
	FROM sys.columns c
		JOIN sys.objects o ON o.object_id = c.object_id
	WHERE o.object_id = OBJECT_ID(@table)

	UNION ALL

	SELECT '' 

	UNION ALL

	SELECT 'public static final String ' + UPPER (c.name) + ' = "' + o.name + '.' + c.Name + '";'
		FROM sys.columns c 
			JOIN sys.objects o ON o.object_id = c.object_id
		WHERE o.object_id = OBJECT_ID(@table)
      
    GO  
    
   # Generate Map Column
    USE schema_name;  
    GO  
    
    CREATE PROCEDURE generateColumn   
    @table nvarchar(50)   
    
    AS
    SET NOCOUNT ON;  
    
    SELECT 'map.put(COL_' + UPPER (c.Name) + ' , param.get' +  c.Name + '());'
	FROM sys.columns c
		JOIN sys.objects o ON o.object_id = c.object_id
	WHERE o.object_id = OBJECT_ID(@table)
      
    GO  
