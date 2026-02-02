# Summary

## Problem
 -  The udf.py file imports and uses duckdb.typing throughout
 - which was deprecated in DuckDB >= 1.4.0 in favor of duckdb.sqltypes
 - The deprecation warning states:
	 - duckdb.typing is deprecated and will be removed in a future version. 
	 - Please use duckdb.sqltypes instead.
 - When duckdb.typing is eventually removed in a future DuckDB release
	 - all UDF registration and type casting would fail with an ImportError.  

## Fix Applied

### In udf.py:

- Replaced the import (line 7-12): 
- Before:
	- import duckdb.typing
	
- After:#
	- try
		- import duckdb.sqltypes as duckdb_types
	- except ImportError:
		- import duckdb.typing as duckdb_types:
			- This uses the new duckdb.sqltypes module when available (DuckDB >= 1.4.0)
			- with a fallback to the old duckdb.typing for older versions (e.g., the minimum supported 1.2.0)
				- affecting
					- UDFType.to_duckdb_type() method (26 type constants)
					- return type annotations on:
						- UDFStructType
						- UDFListType
						- UDFMapType
						- UDFAnyParameters.
			- The fix maintains full backward compatibility with DuckDB >= 1.2.0
			- while being forward-compatible with future DuckDB versions that will remove duckdb.typing.