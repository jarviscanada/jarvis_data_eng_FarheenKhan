# Linux Cluster Monitoring Agent
## Intrduction
The Linux Resource Cluster Monitoring Agent serves as a fundamental solution designed to monitor resources within a cluster environment. Through the execution of various Bash scripts, essential tasks such as registering host container information and updating real-time usage data are performed. This program caters specifically to the Linux Cluster Administration team (LCA), enabling them to monitor nodes within our Linux cluster and gather pertinent information for informed load- and storage-balancing decisions. Key technologies incorporated in this initiative encompass Git for version control, Linux Bash scripting, Docker for hosting and containerizing databases, and PostgreSQL for secure data storage and querying.
## Quick Start
1. Start a psql instance using psql_docker.sh

   How do we create a psql instance if we do not have one?
`./scripts/psql_docker.sh create [username] [password]`
<br></br>
   If a psql docker instance has been created, start or stop the instance using the following command:

   `./scripts/psql_docker.sh (start | stop)`<br></br>
2. Create host_info and host_usage table in DB using ddl.sql bash script
`psql -h localhost -U postgres -p 5432 -d host_agent -f sql/ddl.sql`
<br></br>
3. Insert hardware specs data into the DB using the host_info.sh bash script</br>
`./scripts/host_info.sh psql_host psql_port db_name psql_user psql_password`
<br></br>
4. Insert hardware usage data into the DB using the host_usage.sh bash script</br>
` ./scripts/host_usage.sh psql_host psql_port db_name psql_user psql_password`<br></br>
5. Setup Crontab to periodically collect hardware usage data for current psql instance in one-minute intervals</br>
   Create a crontab job using the following command:</br>
`crontab -e`</br>
   Edit the crontab job and insert the following command:</br>
`* * * * * bash /home/centos/dev/jrvs/bootcamp/linux_sql/host_agent/scripts/host_usage.sh`
<br></br>
   How do we stop the crontab job?<br></br>
-> <font color=lightblue>Stop the crontab job by editing and removing the job or stopping the cron service.<br></br></font>
   Why are the files not executable?<br></br>
-> <font color=lightblue>Use chmod to alter permissions of files and add executable permissions if they are missing.</font>
## Implementation
The Linux Resource Cluster Monitoring Agent program was established by containerizing a psql instance using Docker. We developed Bash scripts to imbue the psql instance within the container with necessary functionalities. Once the container was operational, ddl.sql facilitated the creation of tables (host_info and host_usage). The host's hardware information was incorporated into the database using the host_info.sh script, while the host's resource usage was similarly inserted and stored through the execution of the host_usage.sh script. To automate the process of collecting the host's resource usage, a crontab job was configured to run the host_usage.sh script at minute intervals. This setup ensured real-time updates and continuous population of our psql instance.
<br></br>












      

 