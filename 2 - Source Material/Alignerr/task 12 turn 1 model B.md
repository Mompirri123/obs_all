

# summary

  - Root causes (two issues, both in the DuckDB Python client):
	  - duckdb.typing is deprecated and removed
	  - Starting in DuckDB 1.4
		  - import duckdb no longer auto-populates duckdb.typing.
		  - Explicitly importing it emits DeprecationWarning: duckdb.typing is deprecated … use duckdb.sqltypes instead
		  - The replacement module is duckdb.sqltypes with an identical set of type constants.
	  - UHUGEINT (unsigned 128-bit integer) was added to DuckDB but had no corresponding entry in the UDFType enum, so any UDF using that type could not be declared

## Fix applied (udf.py):

| What                         | Before                                            | After                                                                   | lol |
| ---------------------------- | ------------------------------------------------- | ----------------------------------------------------------------------- | --- |
| Import                       | import duckdb.typing                              | import duckdb.sqltypes                                                  | ü   |
| UDFType enum                 | 26 members, no UHUGEINT                           | 27 members, includes UHUGEINT = 12                                      | ö   |
| to_duckdb_type()             | 87-line if-elif chain referencing duckdb.typing.* | Single dict lookup into _UDFTYPE_TO_DUCKDB, defined once after the enum | ä   |
| All DuckDBPyType annotations | duckdb.typing.DuckDBPyType (×6)                   | duckdb.sqltypes.DuckDBPyType                                            | üöä |

- The dict-based lookup (_UDFTYPE_TO_DUCKDB) is a one-to-one drop-in: adding a future type only requires one new line in the enum and one in the dict — no branching logic to maintain
