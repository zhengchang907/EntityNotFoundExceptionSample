# Issue report

## Subject

Get "EntityNotFoundException" when calling EntityManager.refresh()

## What issue is it

In our code, after EntityManager.persist(), we call EntityManager.merge() and then EntityManager.refresh() to get the inserted data.

However, EntityManager.merge() is calling "SELECT @@IDENTITY" which returns wrong ID value when the table we perform EntityManager.persist() has triggers configured to call INSERT of other tables.

## What is the environment

OS: Windows 10

IDE: Eclipse IDE for Enterprise Java and Web Developers - 2021-03
JDK: 8

Liberty Server: WebSphere Application Server
Server version: 21.0.0.7

Database used: Microsoft Azure SQL Database
JDBC driver: mssql-jdbc-8.2.2.jre8.jar

## Steps to reproduce this

We have a sample repo for reproducing the issue, the steps are as follows:

1. Download & install [Eclipse IDE for Enterprise Java and Web Developers - 2021-03](https://www.eclipse.org/downloads/packages/release/2021-03/r/eclipse-ide-enterprise-java-and-web-developers)
Configure a local Liberty Server
1. Download [mssql-jdbc-8.2.2.jre8.jar](https://github.com/microsoft/mssql-jdbc/releases/download/v8.2.2/mssql-jdbc-8.2.2.jre8.jar) and put it under <Local Liberty Server>/usr/shared/resources/
1. Creat an Azure SQL Server and Database following: [Quickstart: Create an Azure SQL Database single database](https://docs.microsoft.com/en-us/azure/azure-sql/database/single-database-create-quickstart?tabs=azure-portal)
1. Checkout the [sample repo](https://github.com/zhengchang907/EntityNotFoundExceptionSample) and open it in Eclipse
1. Replace local Liberty Server configure file with the repo provided server.xml, update the dataSource credentials with Azure SQL Database
1. Use the repro provide create.sql to create COFFEE and COFFEE_AUDIT tables, then create trigger
1. `mvn clean & install`, then deploy the application to local Liberty Server
1. In the popup windows try to add a new Coffee

**If the exception didn't show up, it's because the new inserted entry(by the trigger) in COFFEE_AUDIT table has the AUDIT_ID value equals to a record's ID value in COFFEE table, run the following SQL queries to insert a few random entries then retry:**

1. Connect to the Azure SQL Database's Query Editor following: [Quickstart: Use the Azure portal's query editor (preview) to query an Azure SQL Database](https://docs.microsoft.com/en-us/azure/azure-sql/database/connect-query-portal)
1. Run the following SQL queries to add 5 entries to `COFFEE_AUDIT` table:

    ```sql
    INSERT INTO [dbo].[COFFEE_AUDIT] ([COFFEE_ID], [NAME], [PRICE]) VALUES (1, 'COFFEE1', 11.1);
    INSERT INTO [dbo].[COFFEE_AUDIT] ([COFFEE_ID], [NAME], [PRICE]) VALUES (2, 'COFFEE2', 22.2);
    INSERT INTO [dbo].[COFFEE_AUDIT] ([COFFEE_ID], [NAME], [PRICE]) VALUES (3, 'COFFEE3', 33.3);
    INSERT INTO [dbo].[COFFEE_AUDIT] ([COFFEE_ID], [NAME], [PRICE]) VALUES (4, 'COFFEE4', 44.4);
    INSERT INTO [dbo].[COFFEE_AUDIT] ([COFFEE_ID], [NAME], [PRICE]) VALUES (5, 'COFFEE5', 55.5);
    ```

## Error logs

```text
[2022/2/24  12:15:28:895 CST] 00000022 eclipselink   I   CWWJP9990I: [eclipselink] EclipseLink, version: Eclipse Persistence Services - 2.7.8.v20201217-ecdf3c32c4
 
[2022/2/24  12:15:30:945 CST] 00000022 DatabaseHelpe I   DSRA8203I: Database product name : Microsoft SQL Server
 
[2022/2/24  12:15:30:946 CST] 00000022 DatabaseHelpe I   DSRA8204I: Database product version : 12.00.2327
 
[2022/2/24  12:15:30:946 CST] 00000022 DatabaseHelpe I   DSRA8205I: JDBC driver name  : Microsoft JDBC Driver 8.2 for SQL Server
 
[2022/2/24  12:15:30:946 CST] 00000022 DatabaseHelpe I   DSRA8206I: JDBC driver version  : 8.2.2.0
 
[2022/2/24  12:15:31:993 CST] 00000022 sql           3   [eclipselink.sql] SELECT ID, NAME, PRICE FROM COFFEE
 
[2022/2/24  12:15:32:803 CST] 00000034 webapp        I com.ibm.ws.webcontainer.webapp.WebApp log SRVE0292I: Servlet Message - [javaee-cafe]:.No state saving method defined, assuming default server state saving
 
[2022/2/24  12:15:45:770 CST] 00000038 CafeRepositor I   Finding all coffees.
 
[2022/2/24  12:15:45:771 CST] 00000038 sql           3   [eclipselink.sql] SELECT ID, NAME, PRICE FROM COFFEE
 
[2022/2/24  12:15:51:663 CST] 0000002f CafeRepositor I   Finding all coffees.
 
[2022/2/24  12:15:51:664 CST] 0000002f sql           3   [eclipselink.sql] SELECT ID, NAME, PRICE FROM COFFEE
 
[2022/2/24  12:15:52:260 CST] 00000030 CafeRepositor I   Persisting the new coffee cafe.model.entity.Coffee[id=null, name=AA, price=14.0].
 
[2022/2/24  12:15:52:264 CST] 00000030 sql           3   [eclipselink.sql] INSERT INTO COFFEE (NAME, PRICE) VALUES (?, ?)
 
bind => [AA, 14.0]
 
[2022/2/24  12:15:52:562 CST] 00000030 sql           3   [eclipselink.sql] SELECT @@IDENTITY
 
[2022/2/24  12:15:53:163 CST] 00000030 CafeRepositor I   Getting managed coffee:  
 
                                 cafe.model.entity.Coffee[id=13, name=AA, price=14.0]
 
[2022/2/24  12:15:53:164 CST] 00000030 sql           3   [eclipselink.sql] SELECT ID, NAME, PRICE FROM COFFEE WHERE (ID = ?)
 
bind => [13]
 
[2022/2/24  12:15:53:460 CST] 00000030 BusinessExcep E   CNTR0020E: EJB threw an unexpected (non-declared) exception during invocation of method "refreshCoffee" on bean "BeanId(mssql#javaee-cafe.war#CafeRepository, null)". Exception data: javax.persistence.EntityNotFoundException: Entity no longer exists in the database: cafe.model.entity.Coffee[id=13, name=AA, price=14.0].
 
at org.eclipse.persistence.internal.jpa.EntityManagerImpl.refresh(EntityManagerImpl.java:1160)
 
at org.eclipse.persistence.internal.jpa.EntityManagerImpl.refresh(EntityManagerImpl.java:1042)
 
at com.ibm.ws.jpa.management.JPAExEmInvocation.refresh(JPAExEmInvocation.java:347)
 
at com.ibm.ws.jpa.management.JPAEntityManager.refresh(JPAEntityManager.java:191)
 
at cafe.model.CafeRepository.refreshCoffee(CafeRepository.java:37)
```

## What we tried

No bugs found in the following approaches:

Using entityManager.createNativeQuery()
Using Hibernate
Using Plain JDBC

