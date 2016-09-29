<properties linkid="" urlDisplayName="" pageTitle="MySQL Database on Azure Service overview | Microsoft Azure" metaKeywords="Azure Cloud, technical documentation, files and resources, MySQL, database, quick start guide, Azure MySQL, MySQL PaaS, Azure MySQL PaaS, Azure MySQL Service, Azure RDS" description="MySQL Database on Azure Service Overview lists the features and advantages of using the MySQL Database on Azure cloud database service." metaCanonical="" services="MySQL" documentationCenter="Services" title="" authors="" solutions="" manager="" editor="" />

<tags ms.service="mysql" ms.date="07/05/2016" wacn.date="07/05/2016" wacn.lang="cn" />

> [AZURE.LANGUAGE]
- [Chinese](/documentation/articles/mysql-database-tech-overview/)
- [English](/documentation/articles/mysql-database-enus-tech-overview/)

# MySQL Database on Azure Services overview #

## What is MySQL Database on Azure? ##
MySQL Database on Azure is the MySQL cloud database service that we launched on the Azure platform. It is a type of relational database service that is fully compatible with MySQL protocols and provides users with a fully-managed database service that features stable performance, rapid deployment, high availability, and high security. To cater to different user performance requirements, the MySQL on Azure service provides six different versions that offer performance in multiples from low to high.

## What are the advantages of MySQL Database on Azure? ##
The MySQL Database on Azure service is a fully-managed database platform service that enables you to focus on service logic development without worrying about infrastructure details. Contrast this efficiency with building your own local database or using solutions created with virtual machines (VMs).

### MySQL Database on Azure has the following core advantages: ###

#### Supports flexible upscaling/downscaling and simplifies the purchase experience ####

You no longer need to keep recalculating your database server configuration requirements or worry about how to adjust various aspects of the configuration to achieve better database performance. The MySQL Database on Azure service is available in several versions. You can choose the version that matches your current performance requirements and then flexibly upscale or downscale as actual operating conditions change. If your service scenario has a high read/write ratio, we recommend that you create a read-only instance and separate reading and writing, so that your service will have better horizontal scaling. For specifics, see: [Create and use read-only instances](https://www.azure.cn/documentation/articles/mysql-database-read-replica).

#### Excellent monitoring experience ####

MySQL Database on Azure provides more than 20 types of monitoring indicators to help you to monitor real-time indicators and historical data for every aspect of database performance. The indicators help you to better understand database usage, identify problems, and make adjustments in a timely manner. You can also open and download database logs, which help database administrators to better manage and optimize the database. For specifics, see: [Monitor and manage MySQL Database on Azure](https://www.azure.cn/documentation/articles/mysql-database-operation-monitoring-metrics).

#### Excellent database reliability and security ####

MySQL Database on Azure uses Azure’s geo-redundant storage to provide high reliability for data. The service automatically synchronizes and replicates three local copies in the same region. MySQL Database on Azure also uses an asynchronous method to make three offsite copies in regions that are more than 1,000 kilometers away to ensure that a high level of reliability is maintained for your data. MySQL on Azure also provides excellent backup and restore solutions. MySQL Database on Azure provides a firewall function that can whitelist your client-side public IP address (or IP address block) and supports SSL connections for database access. For more detailed information on SSL configuration, see [Use SSL to securely access MySQL Database on Azure](https://www.azure.cn/documentation/articles/mysql-database-ssl-connection). To be more specific, we provide **full backup**, **incremental backup** and **database restore** functions:



- **Full backup** - MySQL Database on Azure automatically performs a full backup of your MySQL database every day at a time that you specify. Backups are retained for 30 days. You can also instantly make full backups manually. Storage space for backups is provided completely free of charge, so you don’t need to pay any additional fees for backups.
- **Incremental backup** - MySQL Database on Azure automatically backs up new sections or changes to the database for the last seven days to enable rollback based on any point in time.
- **Database restore** - You can restore the database to any time point in the last seven days, or you can restore a full backup. MySQL Database on Azure can restore to the current database instance or a new instance.

(For details about backup and restore methods, see: [Backup and restore MySQL Database on Azure](https://www.azure.cn/documentation/articles/mysql-database-point-in-time-restore). )

#### Provides high availability, supports disaster recovery, and ensures service continuity ####

MySQL Database on Azure uses a high-availability architecture that guarantees database server availability rates in excess of 99.9% and delivers rapid failover. MySQL Database on Azure also provides disaster recovery solutions to ensure service continuity. For details, see [MySQL on Azure service continuity solutions](https://www.azure.cn/documentation/articles/mysql-database-business-continuity-disaster-recovery).

#### Rapid deployment with no need for manual infrastructure maintenance ####

MySQL Database on Azure makes it easy to deploy a MySQL database server that has high availability in just one or two minutes. It also provides an automatic software patch function that means you don’t need to perform any maintenance on the database service. To find out how to quickly deploy a MySQL database server, refer to the [Create and use your first MySQL cloud database server](https://www.azure.cn/documentation/articles/mysql-database-get-started) introduction course.

#### Supports familiar platforms and tools ####

MySQL Database on Azure is compatible with MySQL protocols and supports MySQL 5.5, MySQL 5.6 and MySQL 5.7. You can perform database development and integration by using common platforms and technologies that support MySQL. You can also use familiar management tools such as Workbench and Navicat.

#### Supports offsite database synchronization to meet enterprise needs for hybrid cloud construction ####

MySQL Database on Azure supports offsite database synchronization. For example, you can set a MySQL instance outside Azure as the primary database and an instance running on Azure as the secondary database. You can then use standard MySQL primary/secondary data synchronization methods to sync the data to Azure. This enables MySQL Database on Azure to meet a range of hybrid cloud scenario requirements for business. Database synchronization features can also help you deliver a superior and more flexible cloud migration experience. For the specific configuration steps, see [Synchronize the replication of a local database to MySQL Database on Azure](https://www.azure.cn/documentation/articles/mysql-database-data-replication).

#### Supports automated development ####

MySQL Database on Azure supports the use of PowerShell (see [Use PowerShell to rapidly create a MySQL cloud database)](https://www.azure.cn/documentation/articles/mysql-database-etoe-powershell) or REST-API (see [Technical documentation for REST-API ](https://www.azure.cn/documentation/articles/mysql-database-api-createserver)) to directly perform automated deployment development.

<!---HONumber=AcomDC_0919_2016_MySql-->
