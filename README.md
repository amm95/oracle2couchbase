# oracle2couchbase

## Goal

From a set of Oracle database tables, extract the data and import them into a Couchbase database.
Each row will be transformed to a JSON Document and stored in Cocuhbase.

The Java utility will do direct connection to Oracle and Couchbase. There are no extra import/export files in the process.

To execute, use the libary **oracle2couchbase.jar** published in the [release](https://github.com/mahurtado/oracle2couchbase/releases) oh this repository. There are some external dependecies required explained below.

## Conventions

* Each JSON Document will include a root-element equal to the name of the table
* All the attribute names are lower-case
* Couchbase is a key-value document database. The format of the **key** will be derived from the relational primary key as follows:

`[table name in lower case]::[value of field1 of the PK]::[value of field 2 of the PK]:: ...`

* Numeric, boolean and text data types will be preserved in the transformation
* Oracle Date and Timestamps types will be stored in Couchbase as milliseconds since January 1, 1970, 00:00:00 GMT 

## Transformation Example

![](https://github.com/mahurtado/oracle2couchbase/blob/master/oracle2couchbase_1.jpg)

Oracle row in a table:

location_id | street_address | postal_code | city	| state_province | country_id
----------- | -------------- | ----------- | ----	| -------------- | ----------
1200 | 2017 Shinjuku-ku | 1689 | Tokyo | Tokyo Prefecture | JP
**primary key** |

JSON Document to be stored in Couchbase:

**Key**
```
locations::1200
```
**Value** 
```
  "locations": {
    "street_address": "2017 Shinjuku-ku",
    "state_province": "Tokyo Prefecture",
    "postal_code": "1689",
    "city": "Tokyo",
    "location_id": 1200,
    "country_id": "JP"
  }
```
  
## Dependencies

* **Couchbase Client Java SDK**. You can get it [here](http://developer.couchbase.com/documentation/server/4.0/sdks/java-2.2/download-links.html). 
In particular the following libraries are required:
  * couchbase-java-client-2.2.2.jar
  * couchbase-core-io-1.2.2.jar
  * rxjava-1.0.15.jar
* **Oracle thin driver**. Download from [here](http://www.oracle.com/technetwork/apps-tech/jdbc-112010-090769.html) - requires login -.
You can also find thin driver in any Oracle database installation, under [ORACLE_HOME]/jdbc/lib. In particular the following library are required:
  * ojdbc6.jar

## Usage

Command line:
```
java -cp [path_to]/oracle2couchbase.jar:[path_to]/couchbase-java-client-2.2.2.jar:[path_to]/couchbase-core-io-1.2.2.jar:
[path_to]/rxjava-1.0.15.jar:[path_to]/ojdbc6.jar 
-Dparameter1=value1 -Dparameter2=value2 ... com.oracle2couchbase.Loader
```

Parameters:
* **cbClusterAddress**: Machine name/IP of the Couchbase server/list of servers. Default value: 127.0.0.1
* **cbBucketName**: Couchbase bucket name. Default value: "default"
* **cbBucketPassword**: Couchbase bucket password.
* **oraAddress**: Machine name/IP of the Oracle server. Default value: 127.0.0.1
* **oraPort**: Port of the Oracle servers. Default value: 1521
* **oraUser**: Oracle database User
* **oraPassword**: Oracle database password
* **oraSid**: Oracle SID value
* **oraService**: Oracle Service value
* **oraTables**: Comma separated list of the tables to export

Example:

```
java -cp ./oracle2couchbase.jar:./lib/couchbase-java-client-2.2.2.jar:./lib/couchbase-core-io-1.2.2.jar:
./lib/rxjava-1.0.15.jar:./lib/ojdbc6.jar 
-DcbClusterAddress=couchbaseMachine -DcbBucketName=HR -DoraAddress=oracleMachine -DoraUser=HR -DoraPassword=oracle 
-DoraService=XE -DoraTables=COUNTRIES,DEPARTMENTS,EMPLOYEES,JOBS,JOB_HISTORY,LOCATIONS,REGIONS com.oracle2couchbase.Loader
````