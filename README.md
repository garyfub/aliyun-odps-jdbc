
# ODPS JDBC

[![Build Status](https://travis-ci.org/aliyun/aliyun-odps-jdbc.svg?branch=master)](https://travis-ci.org/aliyun/aliyun-odps-jdbc)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.aliyun.odps/odps-jdbc/badge.svg)](https://maven-badges.herokuapp.com/maven-central/com.aliyun.odps/odps-jdbc)

## Chinese Docs

[使用 odps-jdbc 接入 ODPS，不再从零开始](https://yq.aliyun.com/articles/5588)

## Installation

Generally, there are two ways to use ODPS JDBC driver in your project.

1.The first one is to use the standalone library: 

* Download the with-dependencies-jar from [release page](https://github.com/aliyun/aliyun-odps-jdbc/releases).
* Checkout the [CHANGELOG](https://github.com/aliyun/aliyun-odps-jdbc/blob/master/CHANGELOG.md).

2.The second is to rely on maven to resolve the dependencies for you:

```xml
<dependency>
  <groupId>com.aliyun.odps</groupId>
  <artifactId>odps-jdbc</artifactId>
  <version>VERSION</version>
</dependency>
```

## Getting Started

Using ODPS JDBC driver is just as using other JDBC drivers. It contains the following few steps:

1\. Explictly load the ODPS JDBC driver using `Class.forName()`:

```java
Class.forName("com.aliyun.odps.jdbc.OdpsDriver");
```


2\. Connect to the ODPS by creating a `Connection` object with the JDBC driver:


```java
Connection conn = DriverManager.getConnection(url, accessId, accessKey);
```

The ODPS server works with RESTful API, so the url looks like:

```java
String url = "jdbc:odps:ENDPOINT?project=PROJECT_NAME&charset=UTF-8";
```

The connection properties can also be passed through `Properties`. For example:

```java
Properties config = new Properties();
config.put("access_id", "...");
config.put("access_key", "...");
config.put("project_name", "...");
config.put("charset", "...");
Connection conn = DriverManager.getConnection("jdbc:odps:<endpoint>", config);
```


3\. Submit SQL to ODPS by creating `Statement` object and using its `executeQuery()` method:

```java
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery("SELECT foo FROM bar");
```

4\. Process the result set.

For example:

```java
while (rs.next()) {
    ...
}
```



### Connection String


|   URL key   |  Property Key  | Description                              |
| :---------: | :------------: | :--------------------------------------- |
| `endpoint`  |  `end_point`   | the endpoint of the ODPS cluster         |
|  `project`  | `project_name` | the current ODPS project                 |
| `accessId`  |  `access_id`   | the id to access the ODPS service        |
| `accessKey` |  `access_key`  | the authentication key                   |
|  `logview`  | `logview_host` | the host domain of the log view appeared in the log history |
| `lifecycle` |  `lifecycle`   | the lifecycle of the temp table using in query |
|  `charset`  |   `charset`    | the charset of the string                |
| `loglevel`  |  `log_level`   | the level of debug infomartion debug/info/fatal |

## Example

### JDBC Client Sample Code

```java
import java.sql.SQLException;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.Statement;
import java.sql.DriverManager;

public class OdpsJdbcClient {
  private static String driverName = "com.aliyun.odps.jdbc.OdpsDriver";

  /**
   * @param args
   * @throws SQLException
   */
  public static void main(String[] args) throws SQLException {
    try {
      Class.forName(driverName);
    } catch (ClassNotFoundException e) {
      e.printStackTrace();
      System.exit(1);
    }

    // fill in the information here
    String accessId = "your_access_id";
    String accessKey = "your_access_key";
    Connection conn = DriverManager.getConnection("jdbc:odps:https://service.odps.aliyun.com/api?project=<your_project_name>", accessId, accessKey);
    Statement stmt = conn.createStatement();
    String tableName = "testOdpsDriverTable";
    stmt.execute("drop table if exists " + tableName);
    stmt.execute("create table " + tableName + " (key int, value string)");

    String sql;
    ResultSet rs;

    // insert a record
    sql = String.format("insert into table %s select 24 key, 'hours' value from (select count(1) from %s) a", tableName, tableName);
    System.out.println("Running: " + sql);
    int count = stmt.executeUpdate(sql);
    System.out.println("updated records: " + count);

    // select * query
    sql = "select * from " + tableName;
    System.out.println("Running: " + sql);
    rs = stmt.executeQuery(sql);
    while (rs.next()) {
      System.out.println(String.valueOf(rs.getInt(1)) + "\t" + rs.getString(2));
    }

    // regular query
    sql = "select count(1) from " + tableName;
    System.out.println("Running: " + sql);
    rs = stmt.executeQuery(sql);
    while (rs.next()) {
      System.out.println(rs.getString(1));
    }

    // do not forget to close
    stmt.close();
    conn.close();
  }
}
```

### Running the JDBC Sample Code

```shell
# compile the client code
javac OdpsJdbcClient.java

# run the program with specifying the class path
java -cp odps-jdbc-*-with-dependencies.jar:. OdpsJdbcClient
```

### Setting SQL task properties

```java
stmt.execute("set biz_id=xxxxxx");
stmt.execute("set odps.sql.mapper.split.size=512");
```

## Third-party Integration

It is also recommended to use ODPS by using other third-party BI tools or DB visualizer that supports JDBC.

For example:

* [SQLWorkbench/J]()
* [Squrriel SQL]()
* [Pentaho]()


## Getting Involved

The project is under construction (and not fully JDBC-compliant). If you dicover any good features which have not been implemented, please fire me an [Email](mailto:yichao.cheng@alibaba-inc.com) or just pull a request.

### Architecture

[![](mindmap-thumb.png)](mindmap.pdf)

### Build and run unitest

1.Build from source locally:

```
git clone ....
cd odps-jdbc
mvn install -DskipTests
```

2.Copy out a configuration file:

```
cp ./src/test/resources/conf.properties.example ./src/test/resources/conf.properties
```

3.Fill in your connection strings:

```
access_id=...
access_key=...
end_point=...
project_name=...
logview_host=...
charset=UTF-8
```

4.Run maven test command (or just test it in IntelliJ IDEA):

```
mvn test
```

### Data Type Mapping

Currenty, there are six kinds of ODPS data types can be accessed from ODPS JDBC. They can be accessed by the getters of `ResultSet` like `getInt()` and `getTime()`. The following table reflects the mapping between JDBC data type and ODPS data type:


| ODPS Type | Java Type       | JDBC Interface                    |   JDBC    |
| :-------: | :-------------- | :-------------------------------- | :-------: |
|  BIGINT   | Long            | int, short, long                  |  BIGINT   |
|  DOUBLE   | Double          | double, float                     |  DOUBLE   |
|  BOOLEAN  | Boolean         | boolean                           |  BOOLEAN  |
| DATETIME  | util.Date       | sql.Date, sql.Time, sql.Timestamp | TIMESTAMP |
|  STRING   | byte[]          | String                            |  VARCHAR  |
|  DECIMAL  | math.BigDecimal | math.BigDecimal                   |  DECIMAL  |


### Type Conversion

The implicit type conversion follows the rule:


|    ODPS    | BIGINT | DOUBLE | BOOLEAN | DATETIME | STRING | DECIMAL |
| :--------: | :----: | :----: | :-----: | :------: | :----: | :-----: |
|  boolean   |   Y    |   Y    |    Y    |          |   Y    |         |
|    byte    |   Y    |   Y    |         |          |        |    Y    |
|    int     |   Y    |   Y    |         |          |   Y    |    Y    |
|   short    |   Y    |   Y    |         |          |   Y    |    Y    |
|    long    |   Y    |   Y    |         |          |   Y    |    Y    |
|   double   |   Y    |   Y    |         |          |   Y    |    Y    |
|   float    |   Y    |   Y    |         |          |   Y    |    Y    |
| BigDecimal |        |        |         |          |   Y    |    Y    |
|   String   |   Y    |   Y    |    Y    |    Y     |   Y    |    Y    |
|  byte\[\]  |   Y    |   Y    |    Y    |    Y     |   Y    |    Y    |
|    Date    |        |        |         |    Y     |   Y    |         |
|    Time    |        |        |         |    Y     |   Y    |         |
| Timestamp  |        |        |         |    Y     |   Y    |         |




## Authors && Contributors

- [Cheng Yichao](https://github.com/onesuper)
- [Li Ruibo](https://github.com/lyman)
- [Wen Shaojin](https://github.com/wenshao)

## License

licensed under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0.html)
