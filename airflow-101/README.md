## Demo Setup

**For a local MYSQL setup**

Run up a local MySQL database which has the following connection settings

```
finch|docker run --name mysql -d \
   -p 3306:3306 \
   -e MYSQL_ROOT_PASSWORD=change-me \
   --restart unless-stopped \
   mysql:8
```

You can connect to this MySql server running on 127.0.0.1:3306, connecting as root with a password of change-me. You can also create a connection from within VSCode.

> *Tip!* If you get errors such as "ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)", use the "--protocol tcp" 

You will need to enable access to your MySQL database from Airflow. Running this worked for me. **Remember, this is only playing around/demo, never never ever do this for something in real life**

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'change-me';
SET GLOBAL local_infile=1;
FLUSH PRIVILEGES;
```

**For a local PostgreSQL setup**

Run up a local PostgreSQL database by creating the follder Docker Compose file (as postgres-local.yml)

```
version: '3'
volumes:
  psql:
services:
  psql:
    image: postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: change-me
    volumes:
      - psql:/var/lib/postgresql/data 
    ports:
      - 5555:5432
```

```
finch|docker compose -p local-postgres -f postgres-local.yml up
```

This will make postgres avilable locally on port 5555, with the username of postgres/change-me


Start MWAA by running the following command from within the aws-mwaa-local-runner folder

```
./mwaa-local-env start
```

And after a few minutes, you should be able to connect to the localhost on port 8080, logging in as admin/test.

The folders you will be working with are as follows

```
├── README.md  <-- this file
├── aws-mwaa-local-runner
│   ├── db-data <--the postgresql database that mwaa-local-runner will use, spinning up as a container
│   ├── mwaa-local-env <-- the control script
│   ├── docker
│   │   ├── Dockerfile
│   │   ├── config
│   │   │   ├── airflow.cfg <-- the Apache Airflow configuration file, modified to hot load DAGs every 5 sec
│   │   │   ├── constraints.txt
│   │   │   ├── mwaa-base-providers-requirements.txt
│   │   │   └── webserver_config.py
│   │   │   └── .env.localrunner <-- where you define env variabls, including things like AWS Credentials that are passed into the Docker containers
│   │   ├── docker-compose-local.yml <-- the core Docker Compose file used when starting a local mwaa-local-runner
│   │   ├── docker-compose-resetdb.yml
│   │   ├── docker-compose-sequential.yml
│   │   └── script <-- key scripts used to boot Airflow and setup AWS stuff
│   │       ├── bootstrap.sh
│   │       ├── entrypoint.sh
│   │       ├── generate_key.sh
│   │       ├── run-startup.sh
│   │       ├── shell-launch-script.sh
│   │       ├── systemlibs.sh
│   │       └── verification.sh
│   └── startup_script
│       └── startup.sh
└── workflow <-- this is where all the working files are sourced (DAGs, requirements.txt, etc)
    ├── dags
    │   ├── example_dag_with_taskflow_api.py
    │   ├── info-environment.py
    │   └── jokes.py
    ├── plugins
    └── requirements
        └── requirements.txt
```

**Setting up Airflow stuff**

You need to set up the following in the default_aws connection, under the extras add the following based on your aws region

{"region_name": "eu-west-1"}

**MySQL**

You need to configure the default_mysql as follows

```
hostname - host.docker.internal (Docker) / 192.168.5.2 (Finch)
database - bad_jokes
login - root
password - change-me
port - 3306
Extras - {"local_infile": "true"}
```

**PostgreSQL**

```
hostname - host.docker.internal (Docker) / 192.168.5.2 (Finch)
database - bad_jokes
login - postgres
password - change-me
port - 5555

```

> Apache Airflow provider is downgraded as a bug that seems to ignore the local_infile was introduced. 3.2.1 is an older, known working version of the provider

## To stop the demo

Stop the running MWAA local runner (CTRL + C) and then check to make sure there are no dangling containers.

Stop the MySQL|Postgres database using the following command

```
docker kill {container image}
```



From ~/Projects/nordevcon/airflow-101/finch-mwaa-local-runner ->  ./mwaa-finch-local-env start
From ~/Projects/nordevcon/airflow-101/local-postgres -> finch compose -p local-postgres -f postgres-local.yml up
