## teradata ##

![](https://github.com/gnosia93/alibaba-interview/blob/main/images/teradata-archi.png)

* Node − It is the basic unit in Teradata System. Each individual server in a Teradata system is referred as a Node. A node consists of its own operating system, CPU, memory, own copy of Teradata RDBMS software and disk space. A cabinet consists of one or more Nodes.

* Parsing Engine − Parsing Engine is responsible for receiving queries from the client and preparing an efficient execution plan. The responsibilities of parsing engine are −
   - Receive the SQL query from the client
   - Parse the SQL query check for syntax errors
   - Check if the user has required privilege against the objects used in the SQL query
   - Check if the objects used in the SQL actually exists
   - Prepare the execution plan to execute the SQL query and pass it to BYNET
   - Receives the results from the AMPs and send to the client

* Message Passing Layer − Message Passing Layer called as BYNET, is the networking layer in Teradata system. It allows the communication between PE and AMP and also between the nodes. It receives the execution plan from Parsing Engine and sends to AMP. Similarly, it receives the results from the AMPs and sends to Parsing Engine.

* Access Module Processor (AMP) − AMPs, called as Virtual Processors (vprocs) are the one that actually stores and retrieves the data. AMPs receive the data and execution plan from Parsing Engine, performs any data type conversion, aggregation, filter, sorting and stores the data in the disks associated with them. Records from the tables are evenly distributed among the AMPs in the system. Each AMP is associated with a set of disks on which data is stored. Only that AMP can read/write data from the disks.

* BTEQ utility is a powerful utility in Teradata that can be used in both batch and interactive mode. It can be used to run any DDL statement, DML statement, create Macros and stored procedures. BTEQ can be used to import data into Teradata tables from flat file and it can also be used to extract data from tables into files or reports.
* 파일 추출: BTEQ, Fast Export 또는 TPT(Teradata Parallel Transporter)를 통해 일반적으로 CSV 형식의 Teradata 테이블에서 플랫 파일로 데이터를 추출합니다. TPT는 데이터 처리량 측면에서 가장 효율적이므로 가능한 한 항상 TPT를 사용합니다.
* 인덱스, containt 가 있고, 테이블 해시방식으로 샤딩해서 저장하고, row wise 로 데이터를 저장한다.
* 시스템 카탈로그 테이블이 있어서 각종 메타 정보를 조회할 수 있다.

### data integration ###

* [oracle to teradata synchronization /w ogg](https://www.oracle.com/webfolder/technetwork/tutorials/obe/fmw/goldengate/11g/ggs_sect_config_winux_ora_to_ux_tera.pdf)



### 1. 데이터 마이그레이션 ###

* 데이터마이그레이션은 SCT 의 데이터 데이터 에이전트를 활용한다. 데이타의 타입중 변경이 안되는 데이터 부터 먼저 옮겨놓는다. (마스터) 트랜잭셩성 fact 의 경우 대부분이 이력에 해당하므로 수정 보다는 입력이 주류이다..
* 마스터 데이터, 트랜잭션 데이터를 구분해서 update, delete, insert 에 대비한다. 
https://docs.aws.amazon.com/SchemaConversionTool/latest/userguide/agents.html#agents.Installing.AgentSettings

### 2. ETL (dagta integration) ###
   - AS-IS    
       테라데이터는 row-wise 로 데이터를 저장하고 인덱스를 지원한다. 데이터들은 레드쉬프트와 동일하게 샤딩되어져 있다.
       오라클 OLTP 에서 ogg 를 이용해서 CDC 로 변경 데이터를 테라데이터에 카피하고 있고, 프로시저를 이용하여 ETL 을 처리한다.
   - TO-BE
       레드쉬프트는 인덱스를 지원하지 않기 때문에, CDC 로 레드쉬프트에 데이터를 붙는 것은 비효율적이다. (update, delete 시 full scan)  
       etl 역시 프로시저로 작성하는 것은 원본 테이블 전체를 읽는 것과 같아서 추천하지 않는다.
    
   oracle --> s3 (staging) --> redshift 구조 (fact/dimen, summary)
   
          dms              etl
   
   * 데이터는 AWS ETL을 통해 Amazon Simple Storage Service(Amazon S3)에 적재해 전처리하고, Amazon Redshift에 통합한 후 기존에 사용 중인 OLAP(Oracle Hyperion)를 통해 분석

   * DMS 설정은 dataformat 은 파퀘이, 날짜 기반(년월일시) 파티셔닝 사용
          - https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Target.S3.html#CHAP_Target.S3.DatePartitioning
          - https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Target.S3.html#CHAP_Target.S3.Configuring



### 3. 마이그레이션 고려사항 ###
  - 기존 환경 분석 (1개월)
  - 이행 (3개월)
    - 인스턴스 타입 선정 및 데이터 볼륨계산
    - 레드쉬프트 스키마 설계 (분산키, 소트키)
    - 이행 연습 (정합성 체크, 다운타임 단축, 어플리케이션 검증, SQL 성능 검증 및 튜닝)
    - 코드성 데이터 수정 (프로시저, UDF, 매크로)
  - ETL 파이프라인 설계 (3개월)
    - CDC
    - ETL
    - Kafka 커넥터
    - 기타 
  - 안정화 (1개월)
    - 성능 모니터링
    - SQL 튜닝(?)
    - 정합성 오류 수정.   
    

### 4. Issue Fixes for Migration Challenges ### 

* We identified the issue in the Schema Conversion Tool (SCT) related to data type length. To resolve this, we modified the string length twice: first to match the source DDL and then to conform to the target DDL requirements.  
* To avoid collation errors, we changed the database/schema to be case-insensitive.
* For large tables exceeding 20GB, we utilized the virtual partitioning option and applied a date column filter to effectively migrate the data.  
* In Windows, we disabled session timeouts to ensure uninterrupted data migration during background processes.
* To reduce the time required for migration, we provisioned three additional high-resource instances, allowing us to migrate data in parallel and meet the project timeline.  We adjusted the minimum and maximum memory values in the SCT configuration file, resulting in an increase in the speed of the data migration process. 


### 5. Utility 사용법 ###

* [Teradata Parallel Transporter(TPT) - TPTEXPORT - Explanation 2022](https://www.youtube.com/watch?v=NSIqogUoBQU)
* [Table Upload with TPT to S3](https://docs.teradata.com/r/Enterprise_IntelliFlex_Lake_VMware/Teradata-Tools-and-Utilities-Access-Module-Reference-20.00/Teradata-Access-Module-for-S3/TPT-Log-Support)




