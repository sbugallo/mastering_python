5. Code instrumentation and monitoring
**************************************

Our work does not end on writing an application and deploying it to target the execution
environment. It is possible to write an application, which after deployment will not require
any further maintenance, although it is very unlikely. In reality, we need to ensure that it is
properly observed for errors and performance.

To be sure that your product works as expected, you need to properly handle application
logs and monitor the necessary application metrics. This often includes the following:

- Monitoring web application access logs for various HTTP status codes
- A collection of process logs that may contain information about runtime errors and various warnings
- Monitoring usage of system resources (CPU load, memory, network traffic, I/O performance, disk usage, and so on) on the remote hosts where the application is run
- Monitoring application-level performance and metrics that are business performance indicators (customer acquisition, revenue, conversion rates, and so on)
- Luckily, there are a lot of free tools available for instrumenting your code and monitoring its performance. Most of them are very easy to integrate.

5.1. Logging errors – Sentry/Raven
++++++++++++++++++++++++++++++++++

The truth is painful. No matter how precisely your application is tested, your code will
eventually fail at some point. This can be anything: unexpected exception, resource
exhaustion, crash of some backing service, network outage, or simply an issue in the
external library. Some of the possible issues (such as resource exhaustion) can be predicted
and prevented in advance with proper monitoring. Unfortunately, there will
always be something that passes your defenses, no matter how much you try.

What you can do instead is to prepare for such scenarios and make sure that no error
passes unnoticed. In most cases, any unexpected failure scenario results in an exception
raised by the application and logged through the logging system. This can
be ``stdout``, ``stderr``, log file, or whatever output you have configured for logging.
Depending on your implementation, this may or may not result in the application quitting
with some system exit code.

You could, of course, depend solely on the log files stored in the filesystem for finding and
monitoring your application errors. Unfortunately, observing errors in plain textual form is
quite painful and does not scale well beyond anything more complex than running code in
development. You will eventually be forced to use some services designed for log collection
and analysis. Proper log processing is very important for other reasons (that will be
explained a bit later) but does not work well for tracking and debugging errors. The reason
is simple. The most common form of error logs is just Python stack trace. If you stop only
on that, you will shortly realize that it is not enough in finding the root cause of your issues.
This is especially true when errors occur in unknown patterns or in certain load conditions.

What you really need is as much context information about the error occurrence as
possible. It is also very useful to have a full history of the errors that occurred in the
production environment that you can browse and search in some convenient way.

One of the most common tools that gives such capabilities is Sentry
(`https://getsentry.com <https://getsentry.com>`_). It is a battle-tested service for tracking exceptions and
collecting crash reports. It is available as open source, written in Python, and originated as a tool for
backend web developers. Now, it outgrew its initial ambitions and has support for many
more languages, including PHP, Ruby, and JavaScript but still stays the most popular tool
of choice for many Python web developers.

.. tip::

    It is common that web applications do not exit on unhandled exceptions
    because HTTP servers are obliged to return an error response with a
    status code from the 5XX group if any server error occurs. Most Python
    web frameworks do such things by default. In such cases, the exception is,
    in fact, handled either on the internal web framework level or by the
    WSGI server middleware. Anyway, this will usually still result in the
    exception stack trace being printed (usually on standard output).

The Sentry is available as a paid software-as-a-service model, but it is open source, so it can
be hosted for free on your own infrastructure. The library that provides integration with
Sentry is ``sentry-sdk`` (available on PyPI). If you haven't worked with it yet and want to
test it but have no access to your own Sentry server, then you can easily sign up for a free
trial on Sentry's on-premise service site. Once you have access to a Sentry server and have
created a new project, you will obtain a string called Data Source Name (DSN). This DSN
string is the minimal configuration setting needed to integrate your application with sentry.
It contains protocol, credentials, server location, and your organization/project identifier in
the following form:

.. code-block:: bash

    '{PROTOCOL}://{PUBLIC_KEY}:{SECRET_KEY}@{HOST}/{PATH}{PROJECT_ID}'

Once you have DSN, the integration is pretty straightforward, as shown in the following
code:

.. code-block:: python

    import sentry_sdk

    sentry_sdk.init(dsn='https://<key>:<secret>@app.getsentry.com/<project>')

    try:
        1 / 0
    except Exception as e:
        sentry_sdk.capture_exception(e)

.. important::

    The old library for Sentry integration is Raven. It is still maintained and
    available on PyPI but is being phased out, so it is best to start your Sentry
    integration using the newer ``python-sdk`` package. It is possible though
    that some framework integrations or Raven extensions haven't been
    ported to new SDK, so in such situations, integration using Raven is still a
    feasible integration path.

Sentry SDK has numerous integrations with most popular Python frameworks such as
Django, Flask, Celery, or Pyramid to make integration easier. These integrations will
automatically provide additional context that is specific to the given framework. If your
web framework of choice does not have a dedicated support, the ``sentry-sdk`` package
provides generic WSGI middleware that makes it compatible with any WSGI-based web
servers, as shown in the following code:

.. code-block:: python

    from sentry_sdk.integrations.wsgi import SentryWsgiMiddleware


    sentry_sdk.init(dsn='https://<key>:<secret>@app.getsentry.com/<project>')

    # ...
    # note: application is some WSGI application object defined earlier
    application = SentryWsgiMiddleware(application)

The other notable integration is the ability to track messages logged through Python's built-
in ``logging`` module. Enabling such support requires only the following few additional lines
of code:

.. code-block:: python

    import logging
    import sentry_sdk
    from sentry_sdk.integrations.logging import LoggingIntegration

    sentry_logging = LoggingIntegration(
        level=logging.INFO,
        event_level=logging.ERROR,
    )
    sentry_sdk.init(
        dsn='https://<key>:<secret>@app.getsentry.com/<project>',
        integrations=[sentry_logging],
    )

Capturing of logging messages may have caveats, so make sure to read the official
documentation on that topic if you are interested in such a feature. This should save you
from unpleasant surprises.

The last note is about running your own Sentry as a way to save some money. There ain't no
such thing as a free lunch. You will eventually pay additional infrastructure costs and Sentry
will be just another service to maintain. Maintenance = additional work = costs! As your
application grows, the number of exceptions grow, so you will be forced to scale Sentry as
you scale your product. Fortunately, this is a very robust project, but will not give you any
value if overwhelmed with too much load. Also, keeping Sentry prepared for a catastrophic
failure scenario where thousands of crash reports per second can be sent is a real challenge.
So you must decide which option is really cheaper for you, and whether you have enough
resources to do all of this by yourself. There is, of course, no such dilemma if security
policies in your organization deny sending any data to third parties. If so, just host it on
your own infrastructure. There are costs, of course, but ones that are definitely worth
paying.

5.2. Monitoring system and application metrics
++++++++++++++++++++++++++++++++++++++++++++++

When it comes to monitoring performance, the amount of tools to choose from may be
overwhelming. If you have high expectations, then it is possible that you will need to use a
few of them at the same time.

**Munin** (`http://munin-monitoring.org <http://munin-monitoring.org>`_) is one of the popular choices used
by many organizations regardless of the technology stack they use. It is a great tool for analyzing
resource trends and provides a lot of useful information, even with a default installation
without additional configuration. Its installation consists of the following two main
components:

- The Munin master that collects metrics from other nodes and serves metrics graphs
- The Munin node that is installed on a monitored host, which gathers local metrics and sends it to the Munin master

The master node and most of the plugins are written in Perl. There are also node
implementations in other languages: munin-node-c is written in C
(`<https://github.com/munin-monitoring/munin-c>`_ ) and munin-node-python is written in
Python (`<https://github.com/agroszer/munin-node-python>`_). Munin comes with a huge
number of plugins available in its ``contrib`` repository. This means it provides out-of-the-
box support for most of the popular databases and system services. There are even plugins
for monitoring popular Python web servers, such as uWSGI or Gunicorn.
The main drawback of Munin is the fact that it serves graphs as static images and actual
plotting configuration is included in specific plugin configurations. This does not help in
creating flexible monitoring dashboards and comparing metric values from different
sources at the same graph. But this is the price we need to pay for simple installation and
versatility. Writing your own plugins is quite simple. There is the ``munin-python`` package
(`<http://python-munin.readthedocs.org/en/latest/>`_ ) that helps to write Munin plugins in
Python.

Unfortunately, the architecture of Munin that assumes that there is always a separate
monitoring daemon process on every host that is responsible for collection of metrics may
not be the best solution for monitoring custom application performance metrics. It is indeed
very easy to write your own Munin plugins, but under the assumption that the monitoring
process can already report its performance statistics in some way.

If you want to collect some custom application-level metrics, it might be necessary to
aggregate and store them in some temporary storage until reporting to a custom Munin
plugin. It makes creation of custom metrics more complicated, so you might want to
consider other solutions for such purposes.

The other popular solution that makes it especially easy to collect custom metrics is StatsD
(`<https://github.com/etsy/statsd>`_). It's a network daemon written in Node.js that listens
to various statistics such as counters, timers, and gauges. It is very easy to integrate, thanks
to the simple protocol based on UDP. It is also easy to use the Python package
named ``statsd`` for sending metrics to the StatsD daemon, as follows:

.. code-block: python

    >>> import statsd
    >>> c = statsd.StatsClient('localhost', 8125)
    >>> c.incr('foo') # Increment the 'foo' counter
    >>> c.timing('stats.timed', 320) # Record a 320ms 'stats.timed'




Because UDP is a connectionless protocol, it has a very low performance overhead on the
application code, so it is very suitable for tracking and measuring custom events inside the
application code.

Unfortunately, StatsD is the only metrics collection daemon, so it does not provide any
reporting features. You need other processes that are able to process data from StatsD in
order to see the actual metrics graphs. The most popular choice is Graphite
(`<http://graphite.readthedocs.org>`_). It does mainly the following two things:

- Stores numeric time-series data
- Renders graphs of this data on demand

Graphite provides you with the ability to save graph presets that are highly customizable.
You can also group many graphs into thematic dashboards. Graphs are, similar to Munin,
rendered as static images, but there is also the JSON API that allows other frontends to read
graph data and render it by other means.

One of the great dashboard plugins integrated with Graphite is Grafana
(`<http://grafana.org>`_). It is really worth trying because it has way better usability than
plain Graphite dashboards. Graphs provided in Grafana are fully interactive and easier to
manage.

Graphite is unfortunately a bit of a complex project. It is not a monolithic service and
consists of the following three separate components:

- **Carbon**: This is a daemon written using Twisted that listens for time-series data.
- **whisper**: This is a simple database library for storing time-series data.
- **graphite webapp**: This is a Django web application that renders graphs on-
  demand as static images (using Cairo library) or as JSON data.

When used with the StatsD project, the ``statsd`` daemon sends its data to
the ``carbon`` daemon. This makes the full solution a rather complex stack of various
applications, where each of them is written using completely different technology. Also,
there are no preconfigured graphs, plugins, and dashboards available, so you will need to
configure everything by yourself. This is a lot of work at the beginning and it is very easy to
miss something important. This is the reason why it might be a good idea to use Munin as a
monitoring backup, even if you decide to have Graphite as your core monitoring service.

Another good monitoring solution for arbitrary metric collection is Prometheus. It has a
completely different architecture than Munin and StatsD. Instead of relying on monitored
applications or daemons to push metrics in configured intervals, Prometheus actively pulls
metrics directly from the source using the HTTP protocol. This requires monitored services
to store (and sometimes preprocess) metrics internally and expose them on HTTP
endpoints.

Fortunately, Prometheus comes with a handful of libraries for various languages and
frameworks to make this kind of integration as easy as possible. There are also various
exporters that act as bridges between Prometheus and other monitoring systems. So, if you
already use other monitoring solutions, it is usually very easy to migrate gradually to a
Prometheus architecture. Prometheus also wonderfully integrates with Grafana.

5.3. Dealing with application logs
++++++++++++++++++++++++++++++++++

While solutions such as Sentry are usually way more powerful than ordinary textual output
stored in files, logs will never die. Writing some information to a standard output or file is
one of the simplest things that an application can do and this should never be
underestimated. There is a risk that messages sent to Sentry by Raven will not get
delivered. The network can fail. Sentry's storage can get exhausted or may not be able to
handle the incoming load. Your application might crash before any message is sent (with a
segmentation fault, for example). These are only a few of the possible scenarios.

What is less likely is that your application won't be able to log messages that are going to be
written to the filesystem. It is still possible, but let's be honest, if you face such a condition
where logging fails, probably you have a lot more burning issues than some missing log
messages.

Remember that logs are not only about errors. Many developers used to think about logs
only as a source of data that is useful when debugging issues and/or that can be used to
perform some kind of forensics.

Definitely, less of them try to use it as a source for generating application metrics or to do
some statistical analysis. But logs may be a lot more useful than that. They can even be a
core of the product implementation. A great example of building a product with logs is
Amazon's article presenting example architecture for the real-time bidding service, where
everything is centered around access log collection and processing.
See `<https://aws.amazon.com/blogs/aws/real-time-ad-impression-bids-using-dynamodb/>`_

5.3.1. Basic low-level log practices
------------------------------------

The Twelve-Factor App manifesto says that logs should be treated as event streams. So, the
log file is not a log by itself, but only an output format. The fact that they are streams means
they represent time ordered events. In raw, they are typically in a plaintext format with one
line per event, although in some cases they may span across multiple lines (this is typical
for any back traces related to runtime errors).

According to the Twelve-Factor App methodology, the application should never be aware
of the format in which logs are stored. This means that writing to the file, or log rotation
and retention should never be maintained by the application code.

These are the responsibilities of the environment in which the applications is run. This may
be confusing because a lot of frameworks provide functions and classes for managing log
files as well as rotation, compression, and retention utilities. It is tempting to use them
because everything can be contained in your application code base, but actually it is an
anti-pattern that should be avoided.

The best practices for dealing with logs are as follows:

- The application should always write logs unbuffered to the standard output (``stdout``).
- The execution environment should be responsible for collection and routing of
  logs to the final destination.

The main part of the mentioned execution environment is usually some kind of process
supervision tool. The popular Python solutions, such as Supervisor or Circus, are the first
ones responsible for dealing with log collection and routing. If logs are to be stored in the
local filesystem, then only they should write to actual log files.

Both Supervisor and Circus are also capable of handling log rotation and retention for
managed processes but you should really consider whether this is a path that you want to
take. Successful operations are mostly about simplicity and consistency. Logs of your own
application are probably not the only ones that you want to process and archive. If you use
Apache or NGINX as a reverse proxy, you might want to collect their access logs.

You might also want to store and process logs for caches and databases. If you are running
some popular Linux distribution, then the chances are very high that each of these services
have their own log files processed (rotated, compressed, and so on) by the popular utility
named ``logrotate``. My strong recommendation is to forget about Supervisor's and Circus'
log rotation capabilities for the sake of consistency with other system
services. ``logrotate`` is way more configurable and also supports compression.

.. tip::

    There is an important thing to know when using ``logrotate`` with
    Supervisor or Circus. Rotation of logs will always happen while process
    Supervisor still has open descriptor to rotated logs. If you don't take
    proper countermeasures, then new events will be still written to the file
    descriptor that was already deleted by ``logrotate``. As a result, nothing
    more will be stored in a filesystem. Solutions to this problem are quite
    simple. Configure ``logrotate`` for log files of processes managed by
    Supervisor or Circus with the ``copytruncate`` option. Instead of moving
    the log file after rotation, it will copy it and truncate the original file to
    zero size in place. This approach does not invalidate any of the existing
    file descriptors and processes that are already running can write to log
    files uninterrupted. Supervisor can also accept the ``SIGUSR2`` signal that
    will make it reopen all of the file descriptors. It may be included as
    the ``postrotate ``script in the ``logrotate`` configuration. This second
    approach is more economical in the terms of I/O operations, but is also
    less reliable and harder to maintain.

5.3.2. Tools for log processing
-------------------------------

If you have no experience in working with big amounts of logs, you will eventually gain it
when working with a product that has some substantial load. You will shortly notice that a
simple approach based on storing them in files and backing them up in some persistent
storage for later retrieval is not enough. Without proper tools, this will become crude and
expensive. Simple utilities such as ``logrotate`` help you only to ensure that the hard disk is
not overloaded by the ever-increasing amount of new events, although splitting and
compressing log files only helps in the data archival process but does not make data
retrieval or analysis simpler.

When working with distributed systems that span across multiple nodes, it is nice to have a
single central point from which all logs can be retrieved and analyzed. This requires a log
processing flow that goes way beyond simple compression and backing up. Fortunately,
this is a well-known problem so there are many tools available that aim to solve it.

One of the popular choices among many developers is **Logstash**. This is the log collection
daemon that can observe active log files, parse log entries, and send them to the backing
service in a structured form. The choice of backing stays almost always the
same: **Elasticsearch**. Elasticsearch is the search engine built on top of Lucene. Among text
search capabilities, it has a unique data aggregation framework that fits extremely well into
the purpose of log analysis. The other addition to this pair of tools is **Kibana**. It is a very
versatile monitoring, analysis, and visualization platform for Elasticsearch. The way that
these three tools complement each other is the reason why almost always they are used
together as a single stack for log processing.

The integration of existing services with Logstash is very simple because it can listen on
existing log file changes for the new events with only minimal changes in your logging
configuration. It parses logs in textual form and has preconfigured support for some of the
popular log formats, such as Apache/NGINX access logs. Logstash can be complemented
with Beats. Beats are log shippers compatible with Logstash input protocols that can collect
not only raw log data from files (Filebeat) but also various system metrics (Metricbeat) and
even audit user activities on hosts (Auditbeat).

The other solution that seems to fill some of Logstash gaps is Fluentd. It is an alternative log
collection daemon that can be used interchangeably with Logstash in the mentioned log
monitoring stack. It also has an option to listen and parse log events directly in log files, so
integration requires only a little effort. In contrast to Logstash, it handles reloads very well
and even does not need to be signaled if log files were rotated. Anyway, the most
advantage comes from using one of its alternative log collection options that will require
some substantial changes to logging configuration in your application.

Fluentd really treats logs as event streams (as recommended by the Twelve-Factor App).
The file-based integration is still possible but it is only kind of backward compatible for
legacy applications that treat logs mainly as files. Every log entry is an event and it should
be structured. Fluentd can parse textual logs and has multiple plugin options to handle,
including the following:

- Common formats (Apache, NGINX, and syslog)
- Arbitrary formats specified using regular expressions or handled with custom
  parsing plugins
- Generic formats for structured messages such as JSON

The best event format for Fluentd is JSON because it adds the least amount of overhead.
Messages in JSON can also be passed almost without any change to the backing service
such as Elasticsearch or the database.

The other very useful feature of Fluentd is the ability to pass event streams using transports
other than a log file written to the disk. The following are the most notable built-in input
plugins:

- ``in_udp``: With this plugin, every log event is sent as UDP packets.
- ``in_tcp``: With this plugin, events are sent through TCP connection.
- ``in_unix``: With this plugin, events are sent through a Unix domain socket
  (named socket).
- ``in_http``: With this plugin, events are sent as HTTP POST requests.
- ``in_exec``: With this plugin, Fluentd process executes an external command
  periodically to pull events in the JSON or MessagePack format.
- ``in_tail``: With this plugin, Fluentd process listens for an event in a textual file.

Alternative transports for log events may be especially useful in situations where you need
to deal with poor I/O performance of machine storage. It is very often on cloud computing
services that the default disk storage has a very low number of
**Input Output Operations Per Second (IOPS)** and you need to pay a lot of money for better disk performance.

If your application outputs a large amount of log messages, you can easily saturate your
I/O capabilities, even if the data size is not very high. With alternate transports, you can use
your hardware more efficiently because you leave the responsibility of data buffering only
to a single process-log collector. When configured to buffer messages in memory instead of
disk, you can even completely get rid of disk writes for logs, although this may greatly
reduce the consistency guarantees of collected logs.

Using different transports seems to be slightly against the 11 th rule of the Twelve-Factor
App methodology. Treating logs as event streams when explained in detail suggests that
the application should always log only through a single standard output stream (``stdout``).
It is still possible to use alternate transports without breaking this rule. Writing
to ``stdout`` does not necessarily mean that this stream must be written to file.

You can leave your application logging that way and wrap it with an external process that
will capture this stream and pass it directly to Logstash or Fluentd without engaging the
filesystem. This is an advanced pattern that may not be suitable for every project. It has the
obvious disadvantage of higher complexity, so you need to consider for yourself whether it
is really worth doing.
