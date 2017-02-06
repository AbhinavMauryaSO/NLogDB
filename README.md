# NLogDB
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.nlog-project.org/schemas/NLog.xsd NLog.xsd"
      autoReload="true"
      throwExceptions="false"
      internalLogLevel="Error" internalLogFile="d:\tempFile\log.txt">
 <targets >

   <target xsi:type="AsyncWrapper"
        name="db"
        queueLimit="1"
        timeToSleepBetweenBatches="1"
        batchSize="3"
        overflowAction="Discard">
     <target xsi:type="Database" name="db">
       <!-- SQL command to be executed for each entry -->
       <commandText>INSERT INTO [LogEntries](TimeStamp, Message, Level, Logger) VALUES(getutcdate(), @msg, @level, @logger)</commandText>

       <!-- parameters for the command -->
       <parameter name="@msg" layout="${message}" />
       <parameter name="@level" layout="${level}" />
       <parameter name="@logger" layout="${logger}" />

       <!-- connection string -->
       <dbProvider>System.Data.SqlClient</dbProvider>
       <connectionString>server=.\SQLEXPRESS;database=logdb;integrated security=sspi;</connectionString>

       commands to install database
       <install-command>
         <text>CREATE DATABASE logdb</text>
         <connectionString>server=.\SQLEXPRESS;database=master;integrated security=sspi</connectionString>
         <ignoreFailures>true</ignoreFailures>
       </install-command>

       <install-command>
         <text>
           CREATE TABLE LogEntries(
           id int primary key not null identity(1,1),
           TimeStamp datetime2,
           Message nvarchar(max),
           level nvarchar(10),
           logger nvarchar(128))
         </text>
       </install-command>

       commands to uninstall database
       <uninstall-command>
         <text>DROP DATABASE logdb</text>
         <connectionString>server=.\SQLEXPRESS;database=logdb;integrated security=sspi</connectionString>
         <ignoreFailures>true</ignoreFailures>
       </uninstall-command>
     </target>
   </target>
  </targets>

  <rules>
    <logger name="*" level="Warn" writeTo="db" />
  </rules>

</nlog>
