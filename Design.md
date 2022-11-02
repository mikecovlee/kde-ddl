## Basic Idea
KDE-DDL的基础单位是工程，每个工程最基础的包括两个文件，分别是一个JSON和一个CovScript源文件，分别描述数据源的基础定义和数据转换逻辑
```
{
    "Name": "Test Datasource",
    "Source": "ODBC/HTTP/FTP/OBJECT",
    "Destination": "SQL/OBJECT",
    "Transformer": "test"
}
```
JSON文件用于描述基础定义，其中Source字段描述了数据来源，Destination字段描述了数据去向，Transformer字段描述了源文件中用于数据处理的类名
```
package test_proj
import kde_ddl as ddl
# Initialize Literals
ddl.init()
class test1 extends ddl.sql_datasource
    var db_conn = null
    function construct(__base_args, __source_args) override
        parent.construct(__base_args...)
        ddl.odbc_connect(db_conn, __source_args...)
    end
    # 当目标为SQL类型时，dest为CSDBC对象
    function transform(dest)
    
    end
end
class test2 extends ddl.nonsql_datasource
    var http_response = null
    function construct(__base_args, __source_args) override
        parent.construct(__base_args...)
        ddl.http_get(http_response, __source_args...)
    end
    # 当目标为Object类型时，dest为OStream对象
    function transform(dest)
    
    end
end
```
编写完JSON文件后，使用KDE-DDL工具生成基础代码框架，仅需填写transofrm函数即可

`kde_ddl`包还提供了大量数据处理工具，均使用流式API定义

SQL类型数据:
```
# Filter
var data = db_conn
    .from("test_table")
    .filter({
        "age": [](age)->age>10
    })
# Aggregation
var (salary, ct) = db_conn
    .from("test_table")
    .aggregate({
        ddl.avg("salary"),
        ddl.count()
    })
# Group
var (salary, ct) = db_conn
    .from("test_table")
    .group({
        "title",
        "country"
    })
    .aggregate({
        ddl.avg("salary"),
        ddl.count()
    })
```