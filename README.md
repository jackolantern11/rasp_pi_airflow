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



## Helpful Guides

* [Airflow Docs](https://airflow.apache.org/docs/apache-airflow/stable/start.html)

* [The Crusty Engineer P1](http://www.thecrustyengineer.com/post/setting_up_airflow_on_a_raspberry_pi_4_part_1)

* [The Crusty Engineer P2](http://www.thecrustyengineer.com/post/setting_up_airflow_on_a_raspberry_pi_4_part_2)