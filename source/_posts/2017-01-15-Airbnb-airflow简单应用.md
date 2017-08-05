---
title: Airbnb/airflow之不简单介绍
tags:
  - airflow
  - big data
date: 2017-01-15 20:01:30
categories:
---


# 认识airflow

目前在一家互联网金融公司数据部门工作，人手很少，很多大数据的架构玩不来，一切只能自己研究。
做数据分析就得涉及到数据的ETL，OLTP的数据库不能拿来就分析。首先是extraction，老同事用了kettle作为导数工具，用cron定时运行。
不得不说kettle的GUI确实强大，但我却对它没有太多好感，你懂的。其次，这种上百行定时任务看了就头疼。如果让我来维护简直比吃了苍蝇还恶心。
直到在网上发现了airflow这款流程管理工具，Python写的，类似于Luigi, Oozie, Azkaban。目前还在Apache孵化中。社区和gitter都很活跃。

安装很简单直接`pip install airflow`，然后根据使用目的不同还需要安装其他组件。如果做分布式架构，还需要安装mysql, celery，
还有一个可选方案是mesos，但是我没有用过。
安装完成后会在home目录下生成一个airflow文件夹。这样就算安装成功了。如果要做分布式，需要在每一台机器上都安装。

## 常用概念
- **DAG**：翻译过来是有向无环图，做过大数据处理的都知道。下面是airflow官方解释
    >In Airflow, a DAG – or a Directed Acyclic Graph – is a collection of all the tasks you want 
    to run, organized in a way that reflects their relationships and dependencies.
- **webserver**：这个是airflow的数据可视化页面
- **scheduler**：流程调度器，负责监控所有任务和dag，并触发任务
- **worker**：和celery worker概念一样，负责在分布式环境下对任务进行处理
- **airflow.cfg**：airflow所有配置都在这里了, 需要重点学习
- **connections**：airflow为你预置了一些数据库和各种服务的连接模板，你只需要在一个集中的位置管理好这些连接，就可以在任意地方使用了

## 配置airflow.cfg
需要分别对webserver和celery sections进行配置, 直接上我的配置
    ```conf
    [core]
    # The home folder for airflow, default is ~/airflow
    airflow_home = /root/airflow

    # The folder where your airflow pipelines live, most likely a
    # subfolder in a code repository
    dags_folder = /root/airflow/dags

    # The folder where airflow should store its log files. This location
    base_log_folder = /root/airflow/logs

    # Airflow can store logs remotely in AWS S3 or Google Cloud Storage. Users
    # must supply a remote location URL (starting with either 's3://...' or
    # 'gs://...') and an Airflow connection id that provides access to the storage
    # location.
    remote_base_log_folder =
    remote_log_conn_id =
    # Use server-side encryption for logs stored in S3
    encrypt_s3_logs = False
    # deprecated option for remote log storage, use remote_base_log_folder instead!
    # s3_log_folder =

    # The executor class that airflow should use. Choices include
    # SequentialExecutor, LocalExecutor, CeleryExecutor
    executor = CeleryExecutor

    # The SqlAlchemy connection string to the metadata database.
    # SqlAlchemy supports many different database engine, more information
    # their website
    sql_alchemy_conn = mysql://username:password@databasehost:3306/airflow?charset=utf8

    # The SqlAlchemy pool size is the maximum number of database connections
    # in the pool.
    sql_alchemy_pool_size = 5

    # The SqlAlchemy pool recycle is the number of seconds a connection
    # can be idle in the pool before it is invalidated. This config does
    # not apply to sqlite.
    sql_alchemy_pool_recycle = 3600

    # The amount of parallelism as a setting to the executor. This defines
    # the max number of task instances that should run simultaneously
    # on this airflow installation
    parallelism = 32

    # The number of task instances allowed to run concurrently by the scheduler
    dag_concurrency = 16

    # Are DAGs paused by default at creation
    dags_are_paused_at_creation = True

    # When not using pools, tasks are run in the "default pool",
    # whose size is guided by this config element
    non_pooled_task_slot_count = 128

    # The maximum number of active DAG runs per DAG
    max_active_runs_per_dag = 16

    # Whether to load the examples that ship with Airflow. It's good to
    # get started, but you probably want to set this to False in a production
    # environment
    load_examples = False

    # Where your Airflow plugins are stored
    plugins_folder = /root/airflow/plugins

    # Secret key to save connection passwords in the db
    fernet_key = cryptography_not_found_storing_passwords_in_plain_text

    # Whether to disable pickling dags
    donot_pickle = False

    # How long before timing out a python file import while filling the DagBag
    dagbag_import_timeout = 30


    [operators]
    # The default owner assigned to each new operator, unless
    # provided explicitly or passed via `default_args`
    default_owner = Airflow


    [webserver]
    # The base url of your website as airflow cannot guess what domain or
    # cname you are using. This is used in automated emails that
    # airflow sends to point links to the right web server
    base_url = http://localhost:8080

    # The ip specified when starting the web server
    web_server_host = 0.0.0.0

    # The port on which to run the web server
    web_server_port = 8080

    # The time the gunicorn webserver waits before timing out on a worker
    web_server_worker_timeout = 120

    # Secret key used to run your flask app
    secret_key = temporary_key

    # Number of workers to run the Gunicorn web server
    workers = 4

    # The worker class gunicorn should use. Choices include
    # sync (default), eventlet, gevent
    worker_class = gevent

    # Expose the configuration file in the web server
    expose_config = true

    # Set to true to turn on authentication:
    # http://pythonhosted.org/airflow/installation.html#web-authentication
    authenticate = False

    # Filter the list of dags by owner name (requires authentication to be enabled)
    filter_by_owner = False

    [email]
    email_backend = airflow.utils.email.send_email_smtp

    [smtp]
    # If you want airflow to send emails on retries, failure, and you want to use
    # the airflow.utils.email.send_email_smtp function, you have to configure an smtp
    # server here
    smtp_host = smtp.domain.com
    smtp_starttls = True
    smtp_ssl = False
    smtp_user = airflow@domain.com
    smtp_port = 25
    smtp_password = xt5l3hIyWkqqiEDbufwe
    smtp_mail_from = airflow@airflow.com

    [celery]
    # This section only applies if you are using the CeleryExecutor in
    # [core] section above

    # The app name that will be used by celery
    celery_app_name = airflow.executors.celery_executor

    # The concurrency that will be used when starting workers with the
    # "airflow worker" command. This defines the number of task instances that
    # a worker will take, so size up your workers based on the resources on
    # your worker box and the nature of your tasks
    celeryd_concurrency = 16

    # When you start an airflow worker, airflow starts a tiny web server
    # subprocess to serve the workers local log files to the airflow main
    # web server, who then builds pages and sends them to users. This defines
    # the port on which the logs are served. It needs to be unused, and open
    # visible from the main web server to connect into the workers.
    worker_log_server_port = 8793

    # The Celery broker URL. Celery supports RabbitMQ, Redis and experimentally
    # a sqlalchemy database. Refer to the Celery documentation for more
    # information.
    broker_url = amqp://user:pass@masterhost:5672/airflowvhost

    # Another key Celery setting
    celery_result_backend = db+mysql://user:pass@databasehost:3306/airflow

    # Celery Flower is a sweet UI for Celery. Airflow has a shortcut to start
    # it `airflow flower`. This defines the port that Celery Flower runs on
    flower_port = 5555

    # Default queue that tasks get assigned to and that worker listen on.
    default_queue = default

    [scheduler]
    # Task instances listen for external kill signal (when you clear tasks
    # from the CLI or the UI), this defines the frequency at which they should
    # listen (in seconds).
    job_heartbeat_sec = 5

    # The scheduler constantly tries to trigger new tasks (look at the
    # scheduler section in the docs for more information). This defines
    # how often the scheduler should run (in seconds).
    scheduler_heartbeat_sec = 5

    # Statsd (https://github.com/etsy/statsd) integration settings
    # statsd_on =  False
    # statsd_host =  localhost
    # statsd_port =  8125
    # statsd_prefix = airflow

    # The scheduler can run multiple threads in parallel to schedule dags.
    # This defines how many threads will run. However airflow will never
    # use more threads than the amount of cpu cores available.
    max_threads = 1

    [mesos]
    # Mesos master address which MesosExecutor will connect to.
    master = localhost:5050

    # The framework name which Airflow scheduler will register itself as on mesos
    framework_name = Airflow

    # Number of cpu cores required for running one task instance using
    # 'airflow run <dag_id> <task_id> <execution_date> --local -p <pickle_id>'
    # command on a mesos slave
    task_cpu = 1

    # Memory in MB required for running one task instance using
    # 'airflow run <dag_id> <task_id> <execution_date> --local -p <pickle_id>'
    # command on a mesos slave
    task_memory = 256

    # Enable framework checkpointing for mesos
    # See http://mesos.apache.org/documentation/latest/slave-recovery/
    checkpoint = False

    # Failover timeout in milliseconds.
    # When checkpointing is enabled and this option is set, Mesos waits
    # until the configured timeout for
    # the MesosExecutor framework to re-register after a failover. Mesos
    # shuts down running tasks if the
    # MesosExecutor framework fails to re-register within this timeframe.
    # failover_timeout = 604800

    # Enable framework authentication for mesos
    # See http://mesos.apache.org/documentation/latest/configuration/
    authenticate = False

    # Mesos credentials, if authentication is enabled
    # default_principal = admin
    # default_secret = admin

    ```
我用了rabbitmq作为broker, 安装在master机器上。配置文件要在不同机器间保持一致。

## 初始化并运行
初始化命令：`airflow initdb`
然后在master上分别运行`airflow scheduler`, `airflow webserver`,这时访问浏览器localhost:port就可以看到airflow的可视化页面了

