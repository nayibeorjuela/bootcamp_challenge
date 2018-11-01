# bootcamp_challenge

#Permisos
export HADOOP_USER_NAME=hdfs

#Creacion del directorio
hdfs dfs -mkdir /tmp/challenge

#Listar las tablas
sqoop list-tables -connect jdbc:mysql://34.205.65.241:3306/ecommerce_cloudera -username bootcamp -P 

#Importación de los datos desde MySql
sqoop import -connect jdbc:mysql://34.205.65.241/ecommerce_cloudera -table product_transaction  -username bootcamp -P -hive-import

#Obtención del Log
wget http://34.205.65.241/access.log

#Ingesta en Hdfs
hdfs dfs -put /home/ec2-user/access.log /tmp/challenge/logs

#Crear tabla en hive del log
drop table if exists access_log2;
CREATE EXTERNAL TABLE access_log2 (
        ip                STRING,
        time_local        STRING,
        method            STRING,
        uri               STRING,
        protocol          STRING,
        status            STRING,
        bytes_sent        STRING,
        referer           STRING,
        useragent         STRING
        )
    ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
    WITH SERDEPROPERTIES (
    'input.regex'='^(\\S+) \\S+ \\S+ \\[([^\\[]+)\\] "(\\w+) (\\S+) (\\S+)" (\\d+) (\\d+) "([^"]+)" "([^"]+)".*'
)
STORED AS TEXTFILE
LOCATION '/tmp/challenge/logs'

#Crear Tabla Agrupada de visualizaciones
create table visualizaciones as select split(uri,'=')[1] as producto, count(*) as visual  from access_log2 group by split(uri,'=')[1]

#Crear Tabla de Conversion
create table conversion as select a.product_id, cast((sum(a.product_cantity)/b.visual)*100 as string) as conversion_rate
from product_transaction a inner join visualizaciones b on a.product_id=b.producto
group by a.product_id, b.visual

#Exportar en Sqoop
sqoop export --connect jdbc:mysql://34.205.65.241/ecommerce_cloudera -username bootcamp -P -table conversion_9 -hcatalog-table conversion

