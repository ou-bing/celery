.. _intro:

========================
 Celery 简介
========================

.. contents::
    :local:
    :depth: 1

什么是任务队列?
====================

任务队列是一种用于跨线程/机器分布式工作的机制。

任务队列的输入是一个被称为任务的工作单元，专用的工作进程持续监控任务队列，以便执行新的工作。

Celery 通过消息进行通信，通常使用代理作为客户端和工作者之间的中介。
为了启动一个任务，客户端会向队列添加一条消息，然后代理会将该消息交付给工作者。

一个 Celery 系统为了高可用，可以由多个横向扩展的代理和工作者构成，

Celery 使用 Python 编写，但是该协议可以用任何语言实现。
除了 Python 以外，这里还有使用 Node.js 构建的 node-celery_ 和 node-celery-ts_，以及一个 `PHP 客户端`_。

Language interoperability can also be achieved
exposing an HTTP endpoint and having a task that requests it (webhooks).

.. _`PHP 客户端`: https://github.com/gjedeer/celery-php
.. _node-celery: https://github.com/mher/node-celery
.. _node-celery-ts: https://github.com/IBM/node-celery-ts

我需要什么?
===============

.. sidebar:: Version Requirements
    :subtitle: Celery version 5.3 runs on

    - Python ❨3.8, 3.9, 3.10, 3.11❩
    - PyPy3.8+ ❨v7.3.11+❩

    Celery 4.x 是最后一个支持 Python 2.7 的版本。
    Celery 5.x 需要 Python 3.6 或更新的版本。
    Celery 5.1.x 也需要 Python 3.6 或更新的版本。
    Celery 5.2.x 需要 Python 3.7 或更新的版本。


    如果你在运行旧版本的 Python，那么你也需要使用旧版本的 Celery：

    - Python 2.7 或 Python 3.5: Celery 4.4 或更早的版本。
    - Python 2.6: Celery 3.1 或更早的版本。
    - Python 2.5: Celery 3.0 或更早的版本。
    - Python 2.4: Celery 2.2 或更早的版本。

    Celery 是一个小资金项目，所以我们不支持 Microsoft Windows，请不要打开该平台相关的缺陷。

*Celery* 需要一个消息传输模块来发送和接受消息，RabbitMQ 和 Redis 就是功能完备的传输代理。

不过我们也支持一些实验性质的解决方案，例如在进行本地开发时使用 SQLite。

*Celery* 可以在单机上运行，也可以在多台机器上运行，甚至可以跨数据中心运行。

入门
===========

如果这是你第一次尝试使用 Celery，或者你使用过以前的版本但没有跟上 3.1 版本的发展，那么你应该阅读我们的入门教程：

- :ref:`first-steps`
- :ref:`next-steps`

Celery 是…
==========

.. _`邮件列表`: https://groups.google.com/group/celery-users

.. topic:: \

    - **简单地说**

        Celery 易于使用和维护，并且它 *不需要配置文件* 。

        它有一个活跃、友好的社区，你可以在里面进行讨论并寻求支持，例如使用 `邮件列表`_ 和 :ref:`IRC 频道 <irc-channel>`。

        下面是一个你可以构建的最简单的应用示例：

        .. code-block:: python

            from celery import Celery

            app = Celery('hello', broker='amqp://guest@localhost//')

            @app.task
            def hello():
                return 'hello world'

    - **高可用**

        工作者和客户端将在连接丢失或失败时自动重试，

        一些代理也可以通过 *主/主* 、 *主/备* 同步的方式来支持高可用。

    - **高速**

        单个 Celery 进程可以在亚毫秒级的延迟下，每分钟处理数百万个任务（基于 librabbitmq 库使用 RabbitMQ，并且优化过配置）。

    - **灵活的**

        *Celery* 几乎每一个部分都可以单独使用或进行扩展，
        
        例如自定义池、序列化器、压缩方案、日志、调度器、消费者、生产者、传输代理等等的实现。


.. topic:: 它支持

    .. hlist::
        :columns: 2

        - **代理**

            - :ref:`RabbitMQ <broker-rabbitmq>`, :ref:`Redis <broker-redis>`,
            - :ref:`Amazon SQS <broker-sqs>`, and more…

        - **并发**

            - prefork (multiprocessing),
            - Eventlet_, gevent_
            - thread (multithreaded)
            - `solo` (single threaded)

        - **结果存储**

            - AMQP, Redis
            - Memcached,
            - SQLAlchemy, Django ORM
            - Apache Cassandra, Elasticsearch, Riak
            - MongoDB, CouchDB, Couchbase, ArangoDB
            - Amazon DynamoDB, Amazon S3
            - Microsoft Azure Block Blob, Microsoft Azure Cosmos DB
            - File system

        - **序列化**

            - *pickle*, *json*, *yaml*, *msgpack*.
            - *zlib*, *bzip2* compression.
            - Cryptographic message signing.

特性
========

.. topic:: \

    .. hlist::
        :columns: 2

        - **监控**

            监控事件流由工作者发出，内置以及外部的工具使用它们来实时告诉你集群正在做什么。

            :ref:`阅读更多… <guide-monitoring>`.

        - **工作流**

            简单和复杂的工作流可以由一组强大的原语组成，我们称之为画布。

            例如：分组、链接、分块等等。

            :ref:`阅读更多… <guide-canvas>`.

        - **时间和速率限制**

            您可以控制每秒、每分钟、每小时可以执行的任务数量，也可以控制允许任务运行多长时间。

            这可以其设置为默认值，也可以对指定工作者或某种任务类型单独进行设置。

            :ref:`阅读更多… <worker-time-limits>`.

        - **调度**

            你可以以秒或 :class:`~datetime.datetime` 规定任务的运行时间，
            或者你也可以使用定时任务基于简单间隔处理定期事件，
            以及支持分钟、小时、每周某天、每个月某天和每天某天的 Crontab 表达式。

            :ref:`阅读更多… <guide-beat>`.

        - **资源泄漏保护**

            对于用户任务的资源泄露，:option:`--max-tasks-per-child <celery worker --max-tasks-per-child>`
            选项是有用的。
            
            像内存或文件描述符泄露，都是你无法控制的。

            :ref:`阅读更多… <worker-max-tasks-per-child>`.

        - **用户组件**

            每个工作者组件都可以被定制,并且额外的组件可以由用户自己定义。

            工作者使用“引导步骤”生成 - 一个依赖关系图，可以实现对工作程序内部的细粒度控制。

.. _`Eventlet`: http://eventlet.net/
.. _`gevent`: http://gevent.org/

框架集成
=====================

Celery 很容易与 web 框架集成，其中一些框架甚至有集成包：
    +--------------------+------------------------+
    | `Pyramid`_         | :pypi:`pyramid_celery` |
    +--------------------+------------------------+
    | `Pylons`_          | :pypi:`celery-pylons`  |
    +--------------------+------------------------+
    | `Flask`_           | not needed             |
    +--------------------+------------------------+
    | `web2py`_          | :pypi:`web2py-celery`  |
    +--------------------+------------------------+
    | `Tornado`_         | :pypi:`tornado-celery` |
    +--------------------+------------------------+
    | `Tryton`_          | :pypi:`celery_tryton`  |
    +--------------------+------------------------+

对于 `Django`_ 可以阅读 :ref:`django-first-steps`.

集成包并不是必要的，但它们可以使开发变得更容易。
有时它们会添加重要的钩子，比如在 fork(2) 处关闭数据库连接。

.. _`Django`: https://djangoproject.com/
.. _`Pylons`: http://pylonshq.com/
.. _`Flask`: http://flask.pocoo.org/
.. _`web2py`: http://web2py.com/
.. _`Bottle`: https://bottlepy.org/
.. _`Pyramid`: http://docs.pylonsproject.org/en/latest/docs/pyramid.html
.. _`Tornado`: http://www.tornadoweb.org/
.. _`Tryton`: http://www.tryton.org/
.. _`tornado-celery`: https://github.com/mher/tornado-celery/

快速跳转
==========

.. topic:: 我想要 ⟶

    .. hlist::
        :columns: 2

        - :ref:`获取任务的返回值 <task-states>`
        - :ref:`在我的任务中使用日志 <task-logging>`
        - :ref:`了解最佳实践 <task-best-practices>`
        - :ref:`创建自定义任务的基类 <task-custom-classes>`
        - :ref:`为一个任务组添加回调 <canvas-chord>`
        - :ref:`将任务分成几块 <canvas-chunks>`
        - :ref:`优化工作者 <guide-optimizing>`
        - :ref:`查看内置任务状态列表 <task-builtin-states>`
        - :ref:`创建自定义任务状态 <custom-states>`
        - :ref:`设置自定义任务名称 <task-names>`
        - :ref:`跟踪一个开始的任务 <task-track-started>`
        - :ref:`重试一个失败的任务 <task-retry>`
        - :ref:`获取当前任务的标识 <task-request-info>`
        - :ref:`了解任务被交付到哪个队列 <task-request-info>`
        - :ref:`查看正在运行的工作者列表 <monitoring-control>`
        - :ref:`清空所有消息 <monitoring-control>`
        - :ref:`检查工作者在做什么 <monitoring-control>`
        - :ref:`查看某个工作者注册了哪些任务 <monitoring-control>`
        - :ref:`将任务迁移到新的代理 <monitoring-control>`
        - :ref:`查看事件消息的类型列表 <event-reference>`
        - :ref:`为 Celery 做贡献 <contributing>`
        - :ref:`了解可用性的相关配置 <configuration>`
        - :ref:`获取使用 Celery 的个人和公司列表 <res-using-celery>`
        - :ref:`编写我自己的远程控制命令 <worker-custom-control-commands>`
        - :ref:`在运行时更改工作者的队列 <worker-queues>`

.. topic:: 跳转到 ⟶

    .. hlist::
        :columns: 4

        - :ref:`代理 <brokers>`
        - :ref:`应用程序 <guide-app>`
        - :ref:`任务 <guide-tasks>`
        - :ref:`调用任务 <guide-calling>`
        - :ref:`工作者 <guide-workers>`
        - :ref:`后台运行 <daemonizing>`
        - :ref:`监控 <guide-monitoring>`
        - :ref:`优化 <guide-optimizing>`
        - :ref:`安全 <guide-security>`
        - :ref:`路由 <guide-routing>`
        - :ref:`配置 <configuration>`
        - :ref:`Django <django>`
        - :ref:`贡献 <contributing>`
        - :ref:`信号 <signals>`
        - :ref:`常见问题解答 <faq>`
        - :ref:`API 手册 <apiref>`

.. include:: ../includes/installation.txt
