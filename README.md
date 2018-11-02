
## Monitoring Product Releases with WSO2-SP
Stream Processor, APIM Manager, Identity Server and Enterprise Integrator are the four products of WSO2. All these products are open source products. In every week there is a expected release for each of these products (usually each Tuesday). 
The aim of this project is to monitor the product releases of these products with WSO2 Stream Processor. Through a Siddhi application it will connect to GitHub API and will fetch tag name, release name and published date of the latest release for each product. 

## Features

 - Releases are displayed in a timeline widget including product type, tag name, release date and gap.
 - Sends an E-mail to the product teams if the release is late.

## How to Setup
**Step 01 -** Clone or Download the Project Monitoring Product Releases with WSO2 -SP.

**Step 02 -** Install [Oracle Java SE Development Kit (JDK)].(https://www.oracle.com/technetwork/java/javase/downloads/index.html) version 1.8* and set the JAVA_HOME environment variable.

**Step 03 -** Download the latest [WSO2 SP](https://github.com/wso2/product-sp/releases)

**Step 04 -** Navigate to the `<SP_HOME>/bin` directory and issue the following command to start the WSO2 Stream Processor Studio.
 -   For Windows: `editor.bat`
 -   For Linux: `sh editor.sh`


**Step 05 -** Access the Stream Processor Studio via the `http://localhost:<EDITOR_PORT>/editor` URL. (The default URL is `http://localhost:9390/editor` .)

**Step 06 -** Import the Siddhi application (MonitoringProducReleasesWithWSO2SP.siddhi) to the editor.

**Step 07 -** Configure the database

 - Download and install [MySQL Server](http://dev.mysql.com/downloads/).
 - Download the [MySQL JDBC driver](http://dev.mysql.com/downloads/connector/j/).
 - Unzip the downloaded MySQL driver zipped archive, and copy the MySQL JDBC driver JAR (`mysql-connector-java-x.x.xx-bin.jar`) into the `<SP_HOME>/lib` directory.
 - Enter the following command in a terminal/command window, where `username` is the username you want to use to access the databases.  
`mysql -u username -p`

 -   When prompted, specify the password you are using to access the databases with the username you specified.
 - Add the following configuration under the **Data Sources Configuration** section of the `<SP_HOME>/conf/editor/deployment.yaml` file.

       - name: ProductReleasesDB
        description: Datasource used for Motitoring the product releases!!!
        jndiConfig:
          name: jdbc/ProductReleasesDB
          useJndiReference: true
        definition:
         type: RDBMS
         configuration:
           jdbcUrl: 'jdbc:mysql://localhost:3306/ProductReleasesDB'
           username: root
           password: root
           driverClassName: com.mysql.jdbc.Driver
           maxPoolSize: 50
           idleTimeout: 60000
           connectionTestQuery: SELECT 1
           validationTimeout: 30000
           isAutoCommit: false

 - To create a database named `ProductReleasesDB`, issue the following commands from the terminal.

    mysql> create database ProductReleasesDB;
    mysql> use ProductReleasesDB;
    mysql> source <SP_HOME>/wso2/editor/dbscripts/metrics/mysql.sql;
    mysql> grant all on ProductReleasesDB.* TO username@localhost identified by "password";

 - To create the database tables, issue the following commands from the terminal.
 - 

    mysql> CREATE TABLE SPReleaseDetailsTable (tag_name VARCHAR(50), release_name VARCHAR(100), release_date VARCHAR(50), release_expected_date VARCHAR(50), next_release_expected_date VARCHAR(50), gap INTEGER(50), release_alert_count INTEGER(50), product_type INTEGER(10));
     
    mysql> CREATE TABLE APIMReleaseDetailsTable (tag_name VARCHAR(50), release_name VARCHAR(100), release_date VARCHAR(50), release_expected_date VARCHAR(50), next_release_expected_date VARCHAR(50), gap INTEGER(50), release_alert_count INTEGER(50), product_type INTEGER(10));
     
    mysql> CREATE TABLE ISReleaseDetailsTable (tag_name VARCHAR(50), release_name VARCHAR(100), release_date VARCHAR(50), release_expected_date VARCHAR(50), next_release_expected_date VARCHAR(50), gap INTEGER(50), release_alert_count INTEGER(50), product_type INTEGER(10));

    mysql> CREATE TABLE EIReleaseDetailsTable (tag_name VARCHAR(50), release_name VARCHAR(100), release_date VARCHAR(50), release_expected_date VARCHAR(50), next_release_expected_date VARCHAR(50), gap INTEGER(50), release_alert_count INTEGER(50), product_type INTEGER(10));

    mysql> CREATE TABLE ReleaseDetailsTable (tag_name VARCHAR(50), release_name VARCHAR(100), release_date VARCHAR(50), release_expected_date VARCHAR(50), next_release_expected_date VARCHAR(50), gap INTEGER(50), release_alert_count INTEGER(50), product_type INTEGER(10));

 - In order to function the application properly the database tables should contain values for at least one row. This is because it requires the next_release_expected_date from the table to calculate the gap. To insert values to the tables use the following commands from the terminal. (**Note -** Edit the VALUES in the insert command according to your requirement).

 - 

    mysql> INSERT INTO SPReleaseDetailsTable (tag_name, release_name, release_date, release_expected_date, next_release_expected_date, gap, release_alert_count, product_type) VALUES ('4.3.0-M1', 'WSO2 Stream Processor 4.3.0-M1 Release!', '2018-07-12 20:32:55', '2018-07-10 00:00:01', '2018-07-17 00:00:01', -68, 0, 1);
    
    mysql> INSERT INTO APIMReleaseDetailsTable (tag_name, release_name, release_date, release_expected_date, next_release_expected_date, gap, release_alert_count, product_type) VALUES ('v2.6.0-m1', 'WSO2 API Manager 2.6.0-M1 Released!', '2018-08-04 09:25:03', '2018-07-31 00:00:01', '2018-08-07 00:00:01', -105, 0, 2);

    mysql> INSERT INTO ISReleaseDetailsTable (tag_name, release_name, release_date, release_expected_date, next_release_expected_date, gap, release_alert_count, product_type) VALUES ('v5.7.0-alpha', 'WSO2 Identity Server 5.7.0 Alpha', '2018-08-03 09:48:16', '2018-07-31 00:00:01', '2018-08-07 00:00:01', -81, 0, 3);
    
    mysql> INSERT INTO EIReleaseDetailsTable (tag_name, release_name, release_date, release_expected_date, next_release_expected_date, gap, release_alert_count, product_type) VALUES ('v6.4.0-m5', 'WSO2 Enterprise Integrator 6.4.0-M5', '2018-08-06 05:30:07', '2018-07-31 00:00:01', '2018-08-07 00:00:01', -149, 0, 4);
    
**Note -** Insert at least one data set for one product type in to the ReleaseDetailsTable.
      
    mysql> INSERT INTO ReleaseDetailsTable (tag_name, release_name, release_date, release_expected_date, next_release_expected_date, gap, release_alert_count, product_type) VALUES ('4.3.0-M1', 'WSO2 Stream Processor 4.3.0-M1 Release!', '2018-07-12 20:32:55', '2018-07-10 00:00:01', '2018-07-17 00:00:01', -68, 0, 1);
    
    mysql> INSERT INTO ReleaseDetailsTable (tag_name, release_name, release_date, release_expected_date, next_release_expected_date, gap, release_alert_count, product_type) VALUES ('v2.6.0-m1', 'WSO2 API Manager 2.6.0-M1 Released!', '2018-08-04 09:25:03', '2018-07-31 00:00:01', '2018-08-07 00:00:01', -105, 0, 2);
    
    mysql> INSERT INTO ReleaseDetailsTable (tag_name, release_name, release_date, release_expected_date, next_release_expected_date, gap, release_alert_count, product_type) VALUES ('v5.7.0-alpha', 'WSO2 Identity Server 5.7.0 Alpha', '2018-08-03 09:48:16', '2018-07-31 00:00:01', '2018-08-07 00:00:01', -81, 0, 3);
    
    mysql> INSERT INTO ReleaseDetailsTable (tag_name, release_name, release_date, release_expected_date, next_release_expected_date, gap, release_alert_count, product_type) VALUES ('v6.4.0-m5', 'WSO2 Enterprise Integrator 6.4.0-M5', '2018-08-06 05:30:07', '2018-07-31 00:00:01', '2018-08-07 00:00:01', -149, 0, 4);

 - Add the following configuration under the **Data Sources Configuration** section of the `<SP_HOME>/conf/dashboard/deployment.yaml` file.

  - 

    name: ProductReleasesDB
        description: Datasource used for Monitoring the product releases!!!
        jndiConfig:
          name: jdbc/ProductReleasesDB
          useJndiReference: true
        definition:
          type: RDBMS
          configuration:
            jdbcUrl: 'jdbc:mysql://localhost:3306/ProductReleasesDB'
            username: root
            password: root
            driverClassName: com.mysql.jdbc.Driver
            maxPoolSize: 50
            idleTimeout: 60000
            connectionTestQuery: SELECT 1
            validationTimeout: 30000
            isAutoCommit: false

**Step 08 -** Add the custom widgets in to the dashboard portal

 - To install the dependencies required to to build your widget, navigate to the `<WIDGET_ROOT>` directory and issue the following command.

  

 - `npm install`

 - Go to the `<WIDGET_ROOT>` directory and issue the following command to build the widget.

 - `npm run build`
 -   Once the build is successful the final widget directory is created in the `<WIDGET_ROOT>/dist`  directory. Copy the  `<WIDGET_ROOT>/dist/<WIDGET_NAME>`  directory into the  `<SP_HOME>/wso2/dashboards/deployment/web-ui-apps/portal/extensions/widgets`  directory.
 - Perform the above mention steps under Step-08 for all the five widgets.

**Step 09 -** Now log in to the Dashboard portal and create a new dashboard .

 - Navigate to the `<SP_HOME>/bin` directory and issue the following command to start the WSO2 Stream Processor Studio.
On Windows: `dashboard.bat --run`
On Linux/Mac OS: `sh dashboard.sh`

 - You can access the dashboard via the following URL.  
    `https://<Your_IP>:9643/portal`
    

 - Login to the dashboard by entering `admin` as both the username and the password.
 
 **Step 10 -** Now you can create a dashboard and add the custom widgets of the project by following  **step 4** of [Presenting Data](https://docs.wso2.com/display/SP430/Presenting+Data)

 

			 




