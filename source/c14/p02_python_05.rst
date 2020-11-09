[python] 配置日志记录的几种方式
=================================

开发者通常可以通过以下三种方式配置日志记录：

1. ``Python`` 显式创建日志记录器（ ``logger`` ）、处理程序( ``handler`` )、格式化程序( ``formatter`` )

2. 创建日志配置文件并使用 ``fileConfig()`` 函数来读取它

3. 创建日志配置信息字典并传递给 ``dictConfig()`` 函数。

注：具体API信息请参照 https://docs.python.org/zh-cn/3/library/logging.html#formatter-objects

**方式一：**

.. code-block:: python

    import logging
    import sys

    # create logger
    logger = logging.getLogger(__name__)
    logger.setLevel(logging.DEBUG)

    # create console handler and set level to debug
    handler = logging.StreamHandler(sys.stdout)
    handler.setLevel(logging.DEBUG)

    # create `formatter`
    formatter = logging.Formatter('%(asctime)s - %(thread)d - %(name)s - %(levelname)s - %(message)s')

    # add `formatter` to `handler`
    handler.setFormatter(formatter)

    # add `handler` to  `logger`
    logger.addHandler(handler)

    # test
    logger.debug('debug message')
    logger.info('info message')
    logger.warning('warning message')
    logger.error('error message')
    logger.critical('critical message')

..

输出如下::

    2020-11-07 10:58:55,540 - 12396 - __main__ - DEBUG - debug message
    2020-11-07 10:58:55,540 - 12396 - __main__ - INFO - info message
    2020-11-07 10:58:55,541 - 12396 - __main__ - WARNING - warning message
    2020-11-07 10:58:55,541 - 12396 - __main__ - ERROR - error message
    2020-11-07 10:58:55,541 - 12396 - __main__ - CRITICAL - critical message

..

**方式二：**

文件配置 logging.conf

.. code-block:: ini

    [loggers]
    keys=root,simpleExample

    [handlers]
    keys=consoleHandler

    [formatters]
    keys=simpleFormatter

    [logger_root]
    level=DEBUG
    handlers=consoleHandler

    [logger_simpleExample]
    level=DEBUG
    handlers=consoleHandler
    qualname=simpleExample
    propagate=0

    [handler_consoleHandler]
    class=StreamHandler
    level=DEBUG
    formatter=simpleFormatter
    args=(sys.stdout,)

    [formatter_simpleFormatter]
    format=%(asctime)s - %(name)s - %(levelname)s - %(message)s
    datefmt=

..

.. code-block:: python

    import logging
    import logging.config

    logging.config.fileConfig('logging.conf')

    # create logger
    logger = logging.getLogger('simpleExample')

    # 'application' code
    logger.debug('debug message')
    logger.info('info message')
    logger.warning('warn message')
    logger.error('error message')
    logger.critical('critical message')

..

输出::

    2020-11-07 10:58:55,540 - 12396 - __main__ - DEBUG - debug message
    2020-11-07 10:58:55,540 - 12396 - __main__ - INFO - info message
    2020-11-07 10:58:55,541 - 12396 - __main__ - WARNING - warning message
    2020-11-07 10:58:55,541 - 12396 - __main__ - ERROR - error message
    2020-11-07 10:58:55,541 - 12396 - __main__ - CRITICAL - critical messag

..

**方式三（用处较少）：**

示例：

.. code-block:: python

    {
        'version': 1,
        'disable_existing_loggers': False,
        'formatters': {
            'standard': {
                'format': '%(asctime)s [%(levelname)s] %(name)s: %(message)s'
            },
        },
        'handlers': {
            'default': {
                'level': 'INFO',
                'formatter': 'standard',
                'class': 'logging.StreamHandler',
                'stream': 'ext://sys.stdout',  # Default is stderr
            },
        },
        'loggers': {
            '': {  # root logger
                'handlers': ['default'],
                'level': 'INFO',
                'propagate': True
            },
            'my.packg': {
                'handlers': ['default'],
                'level': 'WARN',
                'propagate': False
            },
        }
    }

..