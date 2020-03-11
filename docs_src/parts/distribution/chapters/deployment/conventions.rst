4. Common conventions and practices
***********************************

There are a set of common conventions and practices for deployment that not every
developer may know but are obvious for anyone who did some operations in their life. As
explained in this chapter's introduction, it is crucial to know at least a few of them, even if
you are not responsible for code deployment and operations, because it will allow you to
make better design decisions during the development.

4.1. The filesystem hierarchy
+++++++++++++++++++++++++++++

The most obvious conventions that may come into your mind are probably about filesystem
hierarchy and user naming. If you are looking for such suggestions here, then you will be
disappointed. There is, of course, a **Filesystem Hierarchy Standard (FHS)** that defines the
directory structure and directory contents in Unix and Unix-like operating systems, but it is
really hard to find the actual OS distribution that is fully compliant with FHS. If system
designers and programmers cannot obey such standards, it is very hard to expect the same
from its administrators. During my experience, I've seen application code deployed almost
everywhere it is possible, including nonstandard custom directories in the root filesystem
level. Almost always the people behind such decisions had really strong arguments for
doing so. The only suggestions in this matter that I can give you are as follows:

- Choose wisely and avoid surprises.
- Be consistent across all of the available infrastructure of your project.
- Try to be consistent across your organization (the company you work in).

What really helps is to document conventions for your project. Just remember to make sure
that this documentation is accessible for every interested team member and that everyone
knows that such a document exists.


4.2. Isolation
++++++++++++++

Reasons for isolation as well as recommended tools were already discussed.
These are: better environment reproducibility
and solving the inevitable problems of dependency conflicts. For the purpose of
deployments, there is only one important thing to add. You should always isolate project
dependencies for each release of your application. In practice, it means that, whenever you
deploy a new version of the application, you should create a new isolated environment for
this release (using ``virtualenv`` or ``venv``). Old environments should be left for some time on
your hosts, so that, in case of issues, you can easily perform a rollback to one of the older
versions of your application.

Creating fresh environments for each release helps in managing their clean state and
compliance with a list of provided dependencies. By fresh environment we mean creating a
new directory tree in the filesystem instead of updating already existing files.
Unfortunately, it may make it a bit harder to perform things such as the graceful reload of
services, which is much easier to achieve if the environment is updated in place.

4.3. Using process supervision tools
++++++++++++++++++++++++++++++++++++

Applications on remote servers are never usually expected to quit. If it is a web application,
its HTTP server process will indefinitely wait for new connections and requests and will
exit only if some unrecoverable error occurs.

It is, of course, not possible to run it manually in shell and have a never-ending SSH
connection. Using ``nohup``, ``screen``, or ``tmux`` to semi-daemonize the process is not an option.
Doing so is like designing your service to fail.

What you need is to have some process supervision tool that can start and manage your
application process. Before choosing the right one, you need to make sure it does the
following things:

- Restarts the service if it quits
- Reliably tracks its state
- Captures its stdout / stderr streams for logging purposes
- Runs a process with specific user/group permissions
- Configures system environment variables

Most of the Unix and Linux distributions have some built-in tools/subsystems for process
supervision such as ``initd`` scripts, ``upstart``, and ``runit``. Unfortunately, in most cases, they
are not well suited for running user-level application code and are really hard to maintain.
In particular, writing reliable ``init.d`` scripts is a real challenge because it requires a lot of
Bash scripting that is hard to do it right. Some Linux distributions such as Gentoo have a
redesigned approach to ``init.d`` scripts, so writing them is a lot easier. Anyway, locking
yourself to a specific OS distribution just for the purpose of a single process supervision
tool is not a good idea.

Two popular tools in the Python community for managing application processes are
Supervisor (`http://supervisord.org <http://supervisord.org>`_ ) and Circus
(`https://circus.readthedocs.org/en/latest/ <https://circus.readthedocs.org/en/latest/>`_ ).
They are both very similar in
configuration and usage. Circus is a bit younger than Supervisor because it was created to
address some weaknesses of the latter. They both can be configured in simple INI-like
configuration format. They are not limited to running Python processes and can be
configured to manage any application. It is hard to say which one is better because they
both provide very similar functionality. Anyway, Supervisor does not run on Python 3, so it
does not get our approval. While it is not a problem to run Python 3 processes under
Supervisor's control, I will take it as an excuse and feature only the example of the Circus
configuration.

Let's assume that we want to run the ``webxample`` application
using ``gunicorn`` webserver under Circus control. In production, we would
probably run Circus under an applicable system-level process supervision tool
(``initd``, ``upstart``, and ``runit``), especially if it was installed from the system packages
repository. For the sake of simplicity, we will run this locally inside of the virtual
environment. The minimal configuration file (here named ``circus.ini``) that allows us to
run our application in Circus looks like this:

.. code-block:: ini

    [watcher:webxample]
    cmd = /path/to/venv/dir/bin/gunicorn webxample.conf.wsgi:application
    numprocesses = 1

Now, the circus process can be run with this configuration file as the execution argument:

.. code-block:: bash

    $ circusd circus.ini
    2016-02-15 08:34:34 circus[1776] [INFO] Starting master on pid 1776
    2016-02-15 08:34:34 circus[1776] [INFO] Arbiter now waiting for commands
    2016-02-15 08:34:34 circus[1776] [INFO] webxample started
    [2016-02-15 08:34:34 +0100] [1778] [INFO] Starting gunicorn 19.4.5
    [2016-02-15 08:34:34 +0100] [1778] [INFO] Listening at: http://127.0.0.1:8000 (1778)
    [2016-02-15 08:34:34 +0100] [1778] [INFO] Using worker: sync
    [2016-02-15 08:34:34 +0100] [1781] [INFO] Booting worker with pid: 1781

Now, you can use the ``circusctl`` command to run an interactive session and control all
managed processes using simple commands. Here is an example of such a session:

.. code-block:: bash

    $ circusctl
    circusctl 0.13.0
    webxample: active
    (circusctl) stop webxample
    ok
    (circusctl) status
    webxample: stopped
    (circusctl) start webxample
    ok
    (circusctl) status
    webxample: active

Of course, both of the mentioned tools have a lot more features available. All of them are
explained in their documentation, so before making your choice, you should read them
carefully.

4.4. Application code running in user space
+++++++++++++++++++++++++++++++++++++++++++

Your application code should be always run in user space. This means it must not be
executed under super-user privileges. If you design your application following the Twelve-
Factor App, it is possible to run your application under a user that has almost no privileges.
The conventional name for the user that owns no files and is in no privileged groups
is ``nobody``; anyway, the actual recommendation is to create a separate user for each
application daemon. The reason for that is system security. It is to limit the damage that a
malicious user can do if it gains control over your application process. In Linux, processes
of the same user can interact with each other, so it is important to have different
applications separated at the user level.

4.5. Using reverse HTTP proxies
+++++++++++++++++++++++++++++++

Multiple Python WSGI-compliant web servers can easily serve HTTP traffic all by
themselves without the need of any other web server on top of them. It is still very common
to hide them behind a reverse proxy such as NGINX or Apache. A reverse proxy creates an
additional HTTP server layer that proxies requests and responses between clients and your
application and appears to your Python server as though it is the requesting client. Reverse
proxies are useful for the following variety of reasons:

- TLS/SSL termination is usually better handled by top-level web servers such as NGINX and Apache. This allows the Python application to speak only simple HTTP protocol (instead of HTTPS), so complexity and configuration of secure communication channels are left for the reverse proxy.
- Unprivileged users cannot bind low ports (in the range of 0-1000), but the HTTP protocol should be served to the users on port 80, and HTTPS should be served on port 443. To do this, you must run the process with super-user privileges. Usually, it is safer to have your application serving on a high port or on a Unix domain socket and use that as an upstream for reverse proxy that is run under the more privileged user.
- Usually, NGINX can serve static assets (images, JS, CSS, and other media) more efficiently than Python code. If you configure it as a reverse proxy, then it is only a few more lines of configuration to serve static files through it.
- When a single host needs to serve multiple applications from different domains, Apache or NGINX are indispensable for creating virtual hosts for different domains served on the same port.
- Reverse proxies can improve performance by adding additional caching layers or can be configured as simple load balancers. Reverse proxies can also apply compression (for example, gzip) to responses in order to limit the amount of required network bandwidth.

Some of the web servers actually are recommended to be run behind a proxy such as
NGINX. For example, ``gunicorn`` is a very robust WSGI-based server that can give
exceptional performance results if its clients are fast as well. On the other hand, it does not
handle slow clients well, so it is easily susceptible to the denial of service attacks based on a
slow client connection. Using a proxy server that is able to buffer slow clients is the best
way to solve this problem.

Mind that, with proper infrastructure, it is possible to almost completely get rid of reverse
proxies in your architecture. Nowadays, things such as SSL termination and compression
can be easily handled with load balancing services such as AWS Load Balancer. Static and
media assets are also better served through Content Delivery Networks (CDNs) that can
also be used to cache other responses of your service.

The mentioned requirement to serve HTTP/HTTPS traffic on low 80/433 ports (that cannot
be bound by unprivileged users) is also no longer a problem if the only entry points that
your clients communicate with are your load balancers and CDN. Still, even with that kind
of architecture, it does not necessarily mean that your system does not facilitate reverse
proxies at all. For instance, many load balancers support proxy protocol. It means that a
load balancer may appear to your application as though it is the requesting client. In such
scenarios, the load balancer acts as it were in fact a reverse proxy.

4.6. Reloading processes gracefully
+++++++++++++++++++++++++++++++++++

The ninth rule of Twelve-Factor App methodology deals with process disposability and
says that you should maximize robustness with fast start up times and graceful shutdowns.
While fast start up time is quite self-explanatory, the graceful shutdowns require some
additional discussion.

In the scope of web applications, if you terminate the server process in a non-graceful way,
it will quit immediately without the time to finish processing requests and reply with
proper responses to connected clients. In the best scenario case, if you use some kind of
reverse proxy, then the proxy might reply to the connected clients with some generic error
response (for example, 502 Bad Gateway), even though it is not the right way to notify
users that you have restarted your application and have deployed a new release.

According to the Twelve-Factor App, the web serving process should be able to quit
gracefully upon receiving the Unix ``SIGTERM`` signal. This means the server should stop
accepting new connections, finish processing all of the pending requests, and then quit with
some exit code when there is nothing more to do.

Obviously, when all of the serving processes quit or start their shutdown procedure, you
are not able to process new requests any longer. This means your service will still
experience an outage. So there is an additional step you need to perform—start new
workers that will be able to accept new connections while the old ones are gracefully
quitting. Various Python WSGI-compliant web server implementations allow you to reload
the service gracefully without any downtime.

The most popular Python web servers are Gunicorn and uWSGI, which provide the
following functionality:

- Gunicorn's master process upon receiving the ``SIGHUP`` signal (``kill -HUP <process-pid>``) will start new workers (with new code and configuration) and attempt a graceful shutdown on the old ones.
- uWSGI has at least three independent schemes for doing graceful reloads. Each of them is too complex to explain briefly, but its official documentation provides full information on all of the possible options.

Today, graceful reloads are a standard in deploying web applications. Gunicorn seems to
have an approach that is the easiest to use but also leaves you with the least flexibility.
Graceful reloads in uWSGI on the other hand allow much better control on reloads but
require more effort to automate and set up. Also, how you handle graceful reloads in your
automated deploys is also affected by what supervision tools you use and how they are
configured. For instance, in Gunicorn, graceful reloads are as simple as the following:

.. code-block:: bash

    kill -HUP <gunicorn-master-process-pid>

But, if you want to properly isolate project distributions by separating virtual environments
for each release and configure process supervision using symbolic links (as presented in
the ``fabfile`` example earlier), you will shortly notice that this feature of Gunicorn may not
work as expected. For more complex deployments, there is still no system-level solution
available that will work for you out-of-the-box. You will always have to do a bit of hacking
and sometimes this will require a substantial level of knowledge about low-level system
implementation details.

In such complex scenarios, it is usually better to solve the problem on a higher level of
abstraction. If you finally decide to run your applications as containers and distribute new
releases as new container images (it is strongly advised), then you can leave the
responsibility of graceful reloads to your container orchestration system of choice (for
example, Kubernetes) that can usually handle various reloading strategies out-of-the-box.

Even without advanced container orchestration systems, you can do graceful reloading on
the infrastructure level. For instance, AWS Elastic Load Balancer is able to gracefully switch
traffic from your old application instances (for example, EC2 hosts) to new ones. Once old
application instances receive no new traffic and are done handling their requests, they can
be simply terminated without any observable outage to your service. Other cloud
providers, of course, usually provide analogous features in their service portfolio.