### Errores conocidos

### ORA-12705
Además de lo indicado por [Burleson](http://www.dba-oracle.com/t_ora_12705_resolution.htm "Burleson"),
si una vez revisado todo no es el NLS_LANG ni en el host ni en el cliente,
revisar si hay varios ORACLE_HOME y el listener está levantado
con el software de otra versión (en mi caso,11.2 para una base de datos 12.1)
