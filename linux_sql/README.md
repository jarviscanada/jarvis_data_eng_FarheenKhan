# Linux Cluster Monitoring Agent
## Intrduction
The Linux Resource Cluster Monitoring Agent serves as a fundamental solution designed to monitor resources within a cluster environment. Through the execution of various Bash scripts, essential tasks such as registering host container information and updating real-time usage data are performed. This program caters specifically to the Linux Cluster Administration team (LCA), enabling them to monitor nodes within our Linux cluster and gather pertinent information for informed load- and storage-balancing decisions. Key technologies incorporated in this initiative encompass Git for version control, Linux Bash scripting, Docker for hosting and containerizing databases, and PostgreSQL for secure data storage and querying.
## Quick Start
>P**lease ensure that you have execute permissions for all these files, if not chmod**.<br>
>**Start a psql instance using psql_docker.sh**<br>
>./scripts/psql_docker.sh create [username] [password]<br>
>**or if instance already exists**<br>
>./scripts/psql_docker.sh start<br></br>
> **Create tables using ddl.sql**<br>
>export PGPASSWORD=[password]<br>
>psql -h localhost -U [username] -c "CREATE DATABASE host_agent;" #(skip if already created)<br>
>psql -h localhost -U [username] -d host_agent -f ./sql/ddl.sql<br></br>
> **Insert hardware specs data into the DB using host_info.sh**<br>
>./scripts/host_info.sh localhost 5432 host_agent [username] [password]<br></br>
> **Insert hardware usage data into the DB using host_usage.sh** <br>
>./scripts/host_usage.sh localhost 5432 host_agent [username] [password]<br></br>
>**Crontab setup (inside square brackets is what to input in the file)**<br>
>crontab -e [* * * * * bash /global/path/to/host_usage.sh localhost 5432 host_agent postgres password]<br>
>**To stop cron job**<br>
>crontab - e []
## Implementation
The Linux Resource Cluster Monitoring Agent program was established by containerizing a psql instance using Docker. We developed Bash scripts to imbue the psql instance within the container with necessary functionalities. Once the container was operational, ddl.sql facilitated the creation of tables (host_info and host_usage). The host's hardware information was incorporated into the database using the host_info.sh script, while the host's resource usage was similarly inserted and stored through the execution of the host_usage.sh script. To automate the process of collecting the host's resource usage, a crontab job was configured to run the host_usage.sh script at minute intervals. This setup ensured real-time updates and continuous population of our psql instance.
<br></br>
## Architecture
![arc.png](..%2Fassets%2Farc.png)<br>
As shown in the diagram, each node runs an instance of the agent (via a cron job) that communicates with a PostgreSQL database.
## Scripts
### psql_docker.sh
This script is designed to manage the lifecycle of a PostgreSQL Docker container, allowing you to create, start, or stop the container based on the provided command-line arguments. It uses Docker volumes to persist data between container instances.<br></br>
### ddl.sql
This script is used to create the schema of the host_info and host_usage tables to be used in application . It also allows user to store data collected from host_info.sh and host_usage.sh <br></br>
### host_info.sh
This script collects the host hardware info and inserts it into the host_info table in database. It will be run only once, at install time. We need to provide the host, port, database name, username, and password as parameters when executing the command.<br></br>
### host_usage.sh
This script collects the current host usage (CPU and Memory) and inserts into the host_usage table in database. It will be triggered by cron at a regular interval, such as once per minute.We need to provide host, port, db_name, username and password as arguments when executing the command.<br></br>
## Database Modelling
### host_info Table
| Column Name | Data Type | Constraint |
|-------------|-----------|------------|
|id|SERIAL|Primary Key|
|hostname|VARCHAR|NOT NULL, UNIQUE|
|cpu_number|INT2|NOT NULL|
|cpu_architecture|VARCHAR|NOT NULL|
|cpu_model|VARCHAR|NOT NULL|
|cpu_mhz|FLOAT8|NOT NULL|
|l2_cache|INT4|NOT NULL|
|timestamp|TIMESTAMP|NULL|
|total_mem|INT4|NULL|
### host_usage Table
|Column Name|Data Type|Constraint|
|-----------|----------|--------|
|timestamp|TIMESTAMP|NOT NULL|
|host_id|SERIAL|FOREIGN KEY|
|memory_free|INT4|NOT NULL|
|cpu_idle|INT2|NOT NULL|
|cpu_kernel|INT2|NOT NULL|
|disk_io|INT4|NOT NULL|
|disk_availaible|INT4|NOT NULL|

## Test
For testing the bash scripts, all testing was done in the terminal using the -x flag during executing, which helped debug line-by-line what and where the bug could be occurring. The next step was to test database schema and this was done by inserting sample queries. Both the tables  host_info and host_usage were checked by running a select statement as SELECT * FROM host_info; and SELECT * FROM host_usage; A crontab job was tested periodically after every minute, to see that the records were inserted in host_usage table <br></br>
## Deployment
Deployment was done using Git, Github, Docker, and crontab.</br>

- Git and Github to deploy the codes</br>
- Docker to deploy the PSQL database</br>
- Crontab to execute the bash script
## Improvements
- Add more scripts to sort processes so that user can filter out processes using more memory to help optimize usage.
- host_info is set to run once, and this should change so that if there is ever a system change, it can change the information in the database accordingly.
- Remove old data if it is no longer relevant, like in the case of a hardware update.














      

 