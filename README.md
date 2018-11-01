<p>
    <strong>#Permisos</strong>
</p>
<p>
    export HADOOP_USER_NAME=hdfs
</p>
<p>
    <strong>#Creacion del directorio</strong>
</p>
<p>
    hdfs dfs -mkdir /tmp/challenge
</p>
<p>
    <strong>#Listar las tablas</strong>
</p>
<p>
    sqoop list-tables -connect
    jdbc:mysql://34.205.65.241:3306/ecommerce_cloudera -username bootcamp -P
</p>
<p>
    <strong>#Importaci</strong>
    <strong>ó</strong>
    <strong>n de los datos desde MySql</strong>
</p>
<p>
    sqoop import -connect jdbc:mysql://34.205.65.241/ecommerce_cloudera -table
    product_transaction -username bootcamp -P -hive-import
</p>
<p>
    <strong>#Obtenci</strong>
    <strong>ó</strong>
    <strong>n del Log</strong>
</p>
<p>
    wget http://34.205.65.241/access.log
</p>
<p>
    <strong>#Ingesta en Hdfs</strong>
</p>
<p>
    hdfs dfs -put /home/ec2-user/access.log /tmp/challenge/logs
</p>
<p>
    <strong>#Crear tabla en hive desde el archivo de log</strong>
</p>
<p>
    
    drop table if exists access_log2;
    
    CREATE EXTERNAL TABLE access_log2 (
    ip STRING,
    time_local STRING,
    method STRING,
    uri STRING,
    protocol STRING,
    status STRING,
    bytes_sent STRING,
    referer STRING,
    useragent STRING)
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
    WITH SERDEPROPERTIES (
    'input.regex'='^(\\S+) \\S+ \\S+ \\[([^\\[]+)\\] "(\\w+) (\\S+) (\\S+)"
    (\\d+) (\\d+) "([^"]+)" "([^"]+)".*'
    STORED AS TEXTFILE
    LOCATION '/tmp/challenge/logs'
</p>
<p>
    <strong>#Crear Tabla Agrupada de visualizaciones en hive</strong>
</p>
<p>
    create table visualizaciones as select split(uri,'=')[1] as producto,
    count(*) as visual from access_log2 group by split(uri,'=')[1]
</p>
<p>
    <strong>#Crear Tabla de Conversion en hive</strong>
</p>
<p>
    create table conversion as select a.product_id,
    cast((sum(a.product_cantity)/b.visual)*100 as string) as conversion_rate
    from product_transaction a inner join visualizaciones b on
    a.product_id=b.producto
    group by a.product_id, b.visual
</p>
<p>
    <strong>#Exportar en Sqoop desde hive a la tabla conversion_9 en Mysql</strong>
</p>
<p>
    sqoop export --connect jdbc:mysql://34.205.65.241/ecommerce_cloudera
    -username bootcamp -P -table conversion_9 -hcatalog-table conversion
</p>
