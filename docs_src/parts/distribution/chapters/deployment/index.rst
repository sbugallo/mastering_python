Deployment
==========

Even perfect code (if it exists) is useless if it is not able to run. So, in order to serve any
purpose, our code needs to be installed on the target machine (computer) and executed.
The process of making a specific version of your application or service available to end
users is called deployment.

In the case of desktop applications, this seems to be simple as your job ends with providing
a downloadable package with an optional installer, if necessary. It is the user's
responsibility to download and install the package in their environment. Your
responsibility is to make this process as easy and convenient as possible. Proper packaging
is still not a simple task, but some tools were already explained in the previous chapter.

Surprisingly, things get more complicated when your code is not a standalone product. If
your application only provides a service that is being sold to users, then it is your
responsibility to run it on your own infrastructure. This scenario is typical for a web
application or any X as a service product. In such a situation, the code is deployed to set off
remote machines that are physically accessible to the developers. This is especially true if
you are already a user of cloud computing services such as Amazon Web Services (AWS)
or Heroku.

.. include:: twelve_factor.rst
.. include:: approaches.rst
.. include:: package_index.rst
.. include:: conventions.rst
.. include:: instrumentation.rst
