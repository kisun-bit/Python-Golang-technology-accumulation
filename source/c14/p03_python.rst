14.3 Python
=========================

日志记录工具
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

随着项目的单(微)服务化，各个服务间相互通信的场景不断扩大，此时对于主服务而言，日志的管理显得尤为重要。而 *python* \
也提供了 *logging* 库来支持多种日志记录管理方式。笔者最初在做远程 *ssh* 服务时，采用了下述方式做日志对象输出：

.. code-block:: python

    import logging

    def get_logger(logger_name):
        """得到日志对象
        """
        logger = logging.getLogger(logger_name)
        logger.setLevel(logging.DEBUG)
        formatter = logging.Formatter('[ %(asctime)s ] - %(levelname)s - %(message)s')
        if not logger.handlers:
            file_log_handler = logging.FileHandler(settings.VIEW_LOG_PATH, encoding=settings.DEFAULT_CHARSET)
            file_log_handler.setLevel(logging.DEBUG)
            file_log_handler.setFormatter(formatter)
            logger.addHandler(file_log_handler)
        return logger
..


补充:

======== ======
等级     意义
-------- ------
DEBUG    程序运行上下文及关键信息，通常仅仅在DEBUG时使用
INFO     程序正常运行，执行到关键操作时的输出信息
WARNING  程序超出预期运行，但在可控边界内
ERROR    程序发生错误，影响部分功能的继续使用
CRITICAL 程序已几近崩溃，严重错误
======== ======

麻雀虽小，五脏俱全。通过代码可以知道，一个日志对象，其相关关键对象如下：

 * **logger**；对外最上层的日志记录对象，可自定制额外的日志信息，使用看 *logging.info* 源码注释
 * **handler**: 处理程序将日志记录（由记录器创建）发送到适当的目的地（文件或控制台） 
 * **filter**: 筛选器提供了更细粒度的功能，用于确定要输出的日志记录(该对象优先级较低)
 * **formatter**: 格式化程序在最终输出中指定日志记录的格式

因此，对于日志记录配置而言，上述四个对象是紧密相关的。

*logging* 库下的 *config* 提供的诸多配置方式(例如 *Dictionary Schema Details*、*User-defined objects*、*Configuration file format* 等)，也与上述四个关键参数对象紧密相关，这里以项目中常见的 *Configuration file format* 模式说明：

.. code-block:: ini
    
    # log.ini
    # 声明logger，配置logger的handler, 配置handler的formatter
    [loggers]
    keys=root

    [handlers]
    keys=fileHandler

    [formatters]
    keys=simpleFormatter

    [logger_root]
    level=NOTSET
    handlers=fileHandler

    [handler_fileHandler]
    class=logging.handlers.RotatingFileHandler  # 可以进入logging.handlers查找需要的处理器
    args=(r'*.log', 'a', 30*1024*1024, 5)
    level=DEBUG
    formatter=simpleFormatter

    [formatter_simpleFormatter]
    format=%(asctime)s - %(name)s - %(levelname)s - %(message)s
    datefmt=
..


.. code-block:: python
    
    # test.py
    import logging
    import logging.config
    
    current_dir = os.path.abspath(os.path.dirname(__file__))
    log_conf_path = os.path.join(current_dir, 'logging.ini')

    logging.config.fileConfig(log_conf_path)


    def get_logger(name):
        return logging.getLogger(name)

    
    def test_log():
        _l = get_logger(__name__)
        _l.info('attention %s', 'known_issue', exc_info=1)

..

你可能会问，那低优先级的筛选器 *filter* 又是怎样的应用场景呢？

这里我们摘取来自 https://www.programcreek.com/python/example/3364/logging.Filter 上的例子做说明 \ 
*_filter_log* 方法实现了屏蔽来自 *API* 目录运行环境下的日志记录，通过重写 *logging.Filter* 下 *filter* 方法的方式

.. code-block:: python

    import logging

    def _filter_log(self):
        """Disables logging in the discovery API to avoid excessive logging."""

        class _ChildLogFilter(logging.Filter):
        """Filter to eliminate info-level logging when called from this module."""

            def __init__(self, filter_levels=None):
                super(_ChildLogFilter, self).__init__()
                self._filter_levels = filter_levels or set(logging.INFO)
                # Get name without extension to avoid .py vs .pyc issues
                self._my_filename = os.path.splitext(
                    inspect.getmodule(_ChildLogFilter).__file__)[0]

            def filter(self, record):
                if record.levelno not in self._filter_levels:
                    return True
                callerframes = inspect.getouterframes(inspect.currentframe())
                for f in callerframes:
                    if os.path.splitext(f[1])[0] == self._my_filename:
                        return False
                    return True

        googleapiclient.discovery.logger.addFilter(_ChildLogFilter({logging.INFO}))

..