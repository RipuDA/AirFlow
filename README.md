# AirFlow
Setup Airflow on AWS EC2

	Spin of EC2 instance (Ubuntu)
	Run the updates by running the following command
    o	sudo apt-get update (this is to run the updates for the o/s)
    o	sudo apt-get install python-setuptools (This is to install python)
    o	sudo apt-get install python-pip (This is to install python package management tool)
    o	sudo pip install --upgrade pip (This will ensure the latest version of the pip is installed)
	Airflow will require database to store the meta-data information. So for the same we will use the PostgreSQL(Sqlite is the default db). Following are the steps
    o	sudo apt-get install postgresql postgresql-contrib
    o	sudo -u postgres psql
	Next steps are to configure the database to be used for the airflow. This involves creating database, creating roles, granting privileges etc.
    o	sudo -u postgres psql

CREATE ROLE ubuntu;
CREATE DATABASE airflow;
GRANT all privileges on database airflow to ubuntu;
ALTER ROLE ubuntu SUPERUSER;
ALTER ROLE ubuntu CREATEDB;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public to ubuntu;

	Connect to the above database and get all the connection related information which would be required to configure the airflow to use this database
    o	postgres-# \c airflow
    o	airflow=# \conninfo
Message : You are connected to database "airflow" as user "postgres" via socket in "/var/run/postgresql" at port "5432"

	Change settings in pg_hb.conf file for required configuration as per Airflow. 
	Run command SHOW hba_file to find location of pg_hba.conf file
/etc/postgresql/10/main/ 
	Exit out of postgresql (using \q) and change the following 2 files
    o	postgresql.conf (in this in approx. in line number 59 uncomment listen_addresses and replace localhost by ‘*’. This is to make sure it listens from the ip address where airflow is running. You can put specific ip address as well
    o	pg_hba.conf (In this file, we need to ensure that database can be access from anywhere. So  change the ipv4 address from local ip to 0.0.0/0. And update the access method type from md5 to trust so that we donot have to put the credentials)

	We will restart PostgreSQL to load changes
    o	sudo service postgresql restart
	Set AIRFLOW_HOME environment variable to ~/airflow
    o	export AIRFLOW_HOME=~/airflow
	Install all the below dependencies (refer to the link for the required packages : https://airflow.apache.org/installation.html)
    o	sudo apt-get install libmysqlclient-dev ( for airflow airflow mysql )
    o	sudo apt-get install libssl-dev ( for airflow cryptograph package)
    o	sudo apt-get install libkrb5-dev (  for airflow kerbero package )
    o	sudo apt-get install libsasl2-dev ( for airflow hive package )
    o	sudo pip install 'apache-airflow[celery]'	
    o	sudo pip install 'apache-airflow[github_enterprise]'	
    o	sudo pip install 'apache-airflow[hive]'	
    o	sudo pip install sqlalchemy
    o	sudo pip uninstall marshmallow-sqlalchemy
    o	sudo pip install marshmallow-sqlalchemy==0.17.1
    o	sudo apt-get install python-psycopg2
	Next step is to install apache airflow
    o	sudo pip install apache-airflow
    o	Login into the database and use the below
        ALTER ROLE ubuntu WITH LOGIN;
    o	sudo airflow initdb
	Now airflow.cfg file should be generated in airflow home directory, we will tweak some configuration here to get better airflow functionality. We will be using CeleryExecutor instead of SequentialExecutor which come by default with airflow. 
	Open the airflow config file and make the following changes
    o	sql_alchemy_conn = postgresql+psycopg2://ubuntu@localhost:5432/airflow
    o	executor = CeleryExecutor
    o	broker_url = amqp://guest:guest@localhost:5672//
    o	celery_result_backend = amqp://guest:guest@localhost:5672//
	Initialize the DB again to reflect these configuration changes
    o	airflow initdb
	Rabbitmq is a message broker, that required to rerun airflow dags with celery.  Rabbitmq can be installed with following command.
    o	sudo apt install rabbitmq-server
	Update the rabbit mq configuration file by uncommenting the NODE_IP_Address and update the values as below. The config file can be found at etc/rabbitmq/rabbitmq-env.conf
    o	NODE_IP_ADDRESS=0.0.0.0
	Start RabbitMQ service
    o	sudo service rabbitmq-server start

	Make directory to place the dags and then run the airflow
    o	mkdir -p /home/ubuntu/airflow/dags/
    o	airflow webserver
o	airflow scheduler
o	airflow worker
	Below command will print Airflow process ID now kill it using command
    o	cat $AIRFLOW_HOME/airflow-webserver.pid
	Airflow can be killed by running the below commands
    o	sudo kill -9 {process_id of airflow}

