# Rasp Pi 4 Airflow Local Install

Install airflow on rasp pi 4

## Basic Install Steps

1. Create 'airflow' directory  ~/airflow
2. Update rasp pi `sudo apt update && sudo apt upgrade`
3. Install required packages `sudo apt-get install python3 python3-pip`

    1. This may include the following package installs depending on what's been done on the rasp pi previously:

        * default-jre-headless
        * gcc 
        * ca-certificates 
        * build-essential 
        * ldap-utils 
        * libsasl2-dev 
        * libldap2-dev 
        * libssl-dev 
        * openssh-server 
        * libaio1 
        * libpq-dev

4. Create virtual env for airflow (`python -m venv`)
5. Activate the virtual env (`source ./activate`)
6. Install airflow `pip install "apache-airflow==2.6.1" --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-2.6.1/constraints-3.9.txt"`

    1. Airflow versions beyond 2.6.1 will run into issues due to the package [google-re2](https://pypi.org/project/google-re2/) - wheels will fail to build on rasp pi

## Service Config

Set up airflow as a service on the pi. When the pi restarts, airflow will restart automatically.

[Airflow Github systemd scripts](https://github.com/apache/airflow/tree/main/scripts/systemd)


1. Copy the files airflow-webserver.service and airflow-scheduler.service to /etc/systemd/system/
    
    * Update the airflow-webserver.service:

    ```   
    [Unit]
    Description=Airflow webserver daemon
    After=network.target vasd.service

    [Service]
    EnvironmentFile=/etc/sysconfig/airflow
    User=zfreeze
    Group=zfreeze
    Type=simple
    ExecStart=/bin/bash -c 'source /home/zfreeze/.virtualenvs/airflow/bin/activate && airflow webserver --pid /home/zfreeze/airflow/webserver.pid'
    Restart=on-failure
    RestartSec=5s
    PrivateTmp=true
        
    [Install]
    WantedBy=multi-user.target
    ```

    * Update the airflow-scheduler.service

    ```
    [Unit]
    Description=Airflow scheduler daemon
    After=network.target vasd.service

    [Service]
    EnvironmentFile=/etc/sysconfig/airflow
    User=zfreeze
    Group=zfreezes
    Type=simple
    ExecStart=/bin/bash -c 'source /home/zfreeze/.virtualenvs/airflow/bin/activate && airflow scheduler'
    Restart=always
    RestartSec=5s

    [Install]
    WantedBy=multi-user.target
    ```

2. Copy the file airflow to /etc/sysconfig/

    * Update the airflow file

    ```
    AIRFLOW_CONFIG=/home/zfreeze/airflow/airflow.cfg
    AIRFLOW_HOME=/home/zfreeze/airflow/
    ```

3. Enable and start the airflow service

    ```
    sudo systemctl daemon-reload 
    sudo systemctl enable airflow-webserver.service 
    sudo systemctl enable airflow-scheduler.service 
    sudo systemctl start airflow-webserver.service 
    sudo systemctl start airflow-scheduler.service
    ```

## Postgresql Backend Config (LocalExecutor)

This will allow tasks to run in parallel using the local executor (instead of sequentially only).
Data from sqlite will be lost with this method. But that's probably okay.

1. sudo apt update && sudo apt upgrade
2. sudo apt install postgresql
3. Set up airflow database & user using psql commands:

    **Helpful psql commands:**

    1. Connect as postgres user: `sudo su postgres` (required after initial setup)
    2. Create new user: `createuser <USERNAME> -P --interactive` (set password)
    3. Start psql connection: `psql`
    4. Create database: `CREATE DATABASE name;`
    5. Connect to specific database as user: `psql -U zfreeze -d test`
    6. Leave psql: `exit`
    7. If you create a database with the same name as your username, you can run `psql user` and it will
    auto connect you to the database
    8. Use `\d` to list all tables in database

    **Airflow DB Setup:**
    
    ```
    CREATE DATABASE airflow_db;
    CREATE USER airflow_user WITH PASSWORD 'airflow_pass';
    GRANT ALL PRIVILEGES ON DATABASE airflow_db TO airflow_user;
    -- PostgreSQL 15 requires additional privileges:
    GRANT ALL ON SCHEMA public TO airflow_user;
    ```

4. pip install 'apache-airflow[mysql]' (in airflow venv)
5. Update airflow.cfg file:
    1. sql_alchemy_conn = sql_alchemy_conn = postgresql+psycopg2://airflow_user:airflow_pass@localhost:5432/airflow_db
    2. executor = LocalExecutor
6. airflow db reset (in airflow venv)
7. airflow db init (in airflow venv)
8. Restart airflow services:
    1. sudo systemctl restart airflow-scheduler.service
    2. sudo systemctl restart airflow-webserver.service


## Helpful Websites:

* [Airflow Docs](https://airflow.apache.org/docs/apache-airflow/stable/start.html)

* [The Crusty Engineer P1](http://www.thecrustyengineer.com/post/setting_up_airflow_on_a_raspberry_pi_4_part_1)

* [The Crusty Engineer P2](http://www.thecrustyengineer.com/post/setting_up_airflow_on_a_raspberry_pi_4_part_2)

* [Postgresql on PI](https://singleboardblog.com/install-postgresql-on-raspberry-pi/)

* [Postgresql on PI](https://pimylifeup.com/raspberry-pi-postgresql/)

* [Postgresql Airflow Docs](https://airflow.apache.org/docs/apache-airflow/stable/howto/set-up-database.html#setting-up-a-postgresql-database)