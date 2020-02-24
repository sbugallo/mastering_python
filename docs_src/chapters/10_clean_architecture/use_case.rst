3. Use case
***********

As an example of how we might organize the components of our application, and how the
previous concepts might work in practice, we present the following simple example.
The use case is that there is an application for delivering food, and this application has a
specific service for tracking the status of each delivery at its different stages. We are going
to focus only on this particular service, regardless of how the rest of the application might
appear. The service has to be really simple—a REST API that, when asked about the status
of a particular order, will return a JSON response with a descriptive message.
We are going to assume that the information about each particular order is stored in a
database, but this detail should not matter at all.
Our service has two main concerns for now: getting the information about a particular
order (from wherever this might be stored), and presenting this information in a useful way
to the clients (in this case, delivering the results in JSON format, exposed as a web service).
As the application has to be maintainable and extensible, we want to keep these two
concerns as hidden as possible and focus on the main logic. Therefore, these two details are
abstracted and encapsulated into Python packages that the main application with the core
logic will use, as shown in the following diagram:
In the following sections, we briefly demonstrate how the code might appear, in terms of
the packages mainly, and how to create services from these, in order to finally see what
conclusions we can infer.
[ 295 ]Clean Architecture
Chapter 10
The code
The idea of creating Python packages in this example is to illustrate how abstracted and
isolated components can be made, in order to work effectively. In reality, there is no actual
need for these to be Python packages; we could just create the right abstractions as part of
the "delivery service" project, and, while the correct isolation is preserved, it will work
without any issues.
Creating packages makes more sense when there is logic that is going to be repeated and is
expected to be used across many other applications (that will import from those packages)
because we want to favor code reuse. In this particular case, there are no such
requirements, so it might be beyond the scope of the design, but such distinction still makes
more clear the idea of a "pluggable architecture" or component, something that is really a
wrapper abstracting technical details we don't really want to deal with, much less depend
upon.
The storage package is in charge of retrieving the data that is required, and presenting
this to the next layer (the delivery service) in a convenient format, something that is suitable
for the business rules. The main application should now know where this data came from,
what its format is, and so on. This is the entire reason why we have such an abstraction in
between so the application doesn't use a row or an ORM entity directly, but rather
something workable.
Domain models
The following definitions apply to classes for business rules. Notice that they are meant to
be pure business objects, not bound to anything in particular. They aren't models of an
ORM, or objects of an external framework, and so on. The application should work with
these objects (or objects with the same criteria).
In each case, the dosctring documents the purpose of each class, according to the business
rule:
from typing import Union
class DispatchedOrder:
"""An order that was just created and notified to start its
delivery."""
status = "dispatched"
def __init__(self, when):
self._when = when
[ 296 ]Clean Architecture
Chapter 10
def message(self) -> dict:
return {
"status": self.status,
"msg": "Order was dispatched on {0}".format(
self._when.isoformat()
),
}
class OrderInTransit:
"""An order that is currently being sent to the customer."""
status = "in transit"
def __init__(self, current_location):
self._current_location = current_location
def message(self) -> dict:
return {
"status": self.status,
"msg": "The order is in progress (current location:
{})".format(
self._current_location
),
}
class OrderDelivered:
"""An order that was already delivered to the customer."""
status = "delivered"
def __init__(self, delivered_at):
self._delivered_at = delivered_at
def message(self) -> dict:
return {
"status": self.status,
"msg": "Order delivered on {0}".format(
self._delivered_at.isoformat()
),
}
class DeliveryOrder:
def __init__(
self,
delivery_id: str,
[ 297 ]Clean Architecture
Chapter 10
status: Union[DispatchedOrder, OrderInTransit, OrderDelivered],
) -> None:
self._delivery_id = delivery_id
self._status = status
def message(self) -> dict:
return {"id": self._delivery_id, **self._status.message()}
From this code, we can already get an idea of what the application will look like—we want
to have a DeliveryOrder object, which will have its own status (as an internal
collaborator), and once we have that, we will call its message() method to return this
information to the user.
Calling from the application
Here is how these objects are going to be used in the application. Notice how this depends
on the previous packages ( web and storage ), but not the other way round:
from storage import DBClient, DeliveryStatusQuery, OrderNotFoundError
from web import NotFound, View, app, register_route
class DeliveryView(View):
async def _get(self, request, delivery_id: int):
dsq = DeliveryStatusQuery(int(delivery_id), await DBClient())
try
result = await dsq.get()
except OrderNotFoundError as e:
raise NotFound(str(e)) from e
return result.message()
register_route(DeliveryView, "/status/<delivery_id:int>")
In the previous section, the domain objects were shown and here the code for the
application is displayed. Aren't we missing something? Sure, but is it something we really
need to know now? Not necessarily.
The code inside the storage and web packages was deliberately left out (although the
reader is more than encouraged to look at it—the repository for the book contains the full
example). Also, and this was done on purpose, the names of such packages were chosen so
as not to reveal any technical detail— storage and web .
[ 298 ]Clean Architecture
Chapter 10
Look again at the code in the previous listing. Can you tell which frameworks are being
used? Does it say whether the data comes from a text file, a database (if so, of what type?
SQL? NoSQL?), or another service (the web, for instance)? Assume that it comes from a
relational database. Is there any clue to how this information is retrieved (manual SQL
queries? Through an ORM?)?
What about the web? Can we guess what frameworks are used?
The fact that we cannot answer any of those questions is probably a good sign. Those are
details, and details ought to be encapsulated. We can't answer those questions unless we
take a look at what's inside those packages.
There is another way of answering the previous questions, and it comes in the form of a
question itself: why do we need to know that? Looking at the code, we can see that there is
a DeliveryOrder , created with an identifier of a delivery, and it that has a get() method,
which returns an object representing the status of the delivery. If all of this information is
correct, that's all we should care about. What difference does it make how is it done?
The abstractions we created make our code declarative. In declarative programming, we
declare the problem we want to solve, not how we want to solve it. It's the opposite
of imperative, in which we have to make all the steps required explicit in order to get
something (for instance connect to the database, run this query, parse the result, load it into
this object, and so on). In this case, we are declaring that we just want to know the status of
the delivery given by some identifier.
These packages are in charge of dealing with the details and presenting what the
application needs in a convenient format, namely objects of the kind presented in the
previous section. We just have to know that the storage package contains an object that,
given an ID for a delivery and a storage client (this dependency is being injected into this
example for simplicity, but other alternatives are also possible), it will
retrieve DeliveryOrder which we can then ask to compose the message.
This architecture provides convenience and makes it easier to adapt to changes, as it
protects the kernel of the business logic from the external factors that can change.
Imagine we want to change how the information is retrieved. How hard would that be? The
application relies on an API, like the following one:
dsq = DeliveryStatusQuery(int(delivery_id), await DBClient())
[ 299 ]Clean Architecture
Chapter 10
So it would be about just changing how the get() method works, adapting it to the new
implementation detail. All we need is for this new object to return DeliveryOrder on
its get() method and that would be all. We can change the query, the ORM, the database,
and so on, and, in all cases, the code in the application does not need to change!
Adapters
Still, without looking at the code in the packages, we can conclude that they work as
interfaces for the technical details of the application.
In fact, since we are seeing the application from a high-level perspective, without needing
to look at the code, we can imagine that inside those packages there must be an
implementation of the adapter design pattern (introduced in Chapter 9 , Common Design
Patterns). One or more of these objects is adapting an external implementation to the API
defined by the application. This way, dependencies that want to work with the application
must conform to the API, and an adapter will have to be made.
There is one clue pertaining to this adapter in the code for the application though. Notice
how the view is constructed. It inherits from a class named View that comes from our web
package. We can deduce that this View is, in turn, a class derived from one of the web
frameworks that might be being used, creating an adapter by inheritance. The important
thing to note is that once this is done, the only object that matters is our View class, because,
in a way, we are creating our own framework, which is based on adapting an existing one
(but again changing the framework will mean just changing the adapters, not the entire
application).
The services
To create the service, we are going to launch the Python application inside a Docker
container. Starting from a base image, the container will have to install the dependencies
for the application to run, which also has dependencies at the operating system level.
This is actually a choice because it depends on how the dependencies are used. If a package
we use requires other libraries on the operating system to compile at installation time, we
can avoid this simply by building a wheel for our platform of the library and installing this
directly. If the libraries are needed at runtime, then there is no choice but to make them part
of the image of the container.
[ 300 ]Clean Architecture
Chapter 10
Now, we discuss one of the many ways of preparing a Python application to be run inside a
Docker container. This is one of numerous alternatives for packaging a Python project into
a container. First, we take a look at what the structure of the directories looks like:
.
├── Dockerfile
├── libs
│
├── README.rst
│
├── storage
│
└── web
├── Makefile
├── README.rst
├── setup.py
└── statusweb
├── __init__.py
└── service.py
The libs directory can be ignored since it's just the place where the dependencies are
placed (it's displayed here to keep them in mind when they are referenced in the setup.py
file, but they could be placed in a different repository and installed remotely via pip ).
We have Makefile with some helper commands, then the setup.py file, and the
application itself inside the statusweb directory. A common difference between packaging
applications and libraries is that while the latter specify their dependencies in
the setup.py file, the former have a requirements.txt file from where dependencies are
installed via pip install -r requirements.txt . Normally, we would do this in the
Dockerfile , but in order to keep things simpler in this particular example, we will assume
that taking the dependencies from the setup.py file is enough. This is because, besides this
consideration, there are a lot more considerations to be taken into account when dealing
with dependencies, such as freezing the version of the packages, tracking indirect
dependencies, using extra tools such as pipenv , and more topics that are beyond the scope
of the chapter. In addition, it is also customary to make the setup.py file read
from requirements.txt for consistency.
Now we have the content of the setup.py file, which states some details of the application:
from setuptools import find_packages, setup
with open("README.rst", "r") as longdesc:
long_description = longdesc.read()
install_requires = ["web", "storage"]
setup(
[ 301 ]Clean Architecture
Chapter 10
name="delistatus",
description="Check the status of a delivery order",
long_description=long_description,
author="Dev team",
version="0.1.0",
packages=find_packages(),
install_requires=install_requires,
entry_points={
"console_scripts": [
"status-service = statusweb.service:main",
],
},
)
The first thing we notice is that the application declares its dependencies, which are the
packages we created and placed under libs/ , namely web and storage , abstracting and
adapting to some external components. These packages, in turn, will have dependencies, so
we will have to make sure the container installs all the required libraries when the image is
being created so that they can install successfully, and then this package afterward.
The second thing we notice is the definition of the entry_points keyword argument
passed to the setup function. This is not strictly mandatory, but it's a good idea to create an
entry point. When the package is installed in a virtual environment, it shares this directory
along with all its dependencies. A virtual environment is a structure of directories with the
dependencies of a given project. It has many subdirectories, but the most important ones
are:
<virtual-env-root>/lib/<python-version>/site-packages
<virtual-env-root>/bin
The first one contains all the libraries installed in that virtual environment. If we were to
create a virtual environment with this project, that directory would contain the web ,
and storage packages, along with all its dependencies, plus some extra basic ones and the
current project itself.
[ 302 ]Clean Architecture
Chapter 10
The second, /bin/ , contains the binary files and commands available when that virtual
environment is active. By default, it would just be the version of Python, pip , and some
other basic commands. When we create an entry point, a binary with that declared name is
placed there, and, as a result, we have that command available to run when the
environment is active. When this command is called, it will run the function that is
specified with all the context of the virtual environment. That means it is a binary we can
call directly without having to worry about whether the virtual environment is active, or
whether the dependencies are installed in the path that is currently running.
The definition is the following one:
"status-service = statusweb.service:main"
The left-hand side of the equals sign declares the name of the entry point. In this case, we
will have a command named status-service available. The right-hand side declares how
that command should be run. It requires the package where the function is defined,
followed by the function name after : . In this case, it will run the main function declared in
statusweb/service.py .
This is followed by a definition of the Dockerfile:
FROM python:3.6.6-alpine3.6
RUN apk add --update \
python-dev \
gcc \
musl-dev \
make
WORKDIR /app
ADD . /app
RUN pip install /app/libs/web /app/libs/storage
RUN pip install /app
EXPOSE 8080
CMD ["/usr/local/bin/status-service"]
The image is built based on a lightweight Python image, and then the operating system
dependencies are installed so that our libraries can be installed. Following the previous
consideration, this Dockerfile simply copies the libraries, but this might as well be
installed from a requirements.txt file accordingly. After all the pip install
commands are ready, it copies the application in the working directory, and the entry point
from Docker (the CMD command, not to be confused with the Python one) calls the entry
point of the package where we placed the function that launches the process.
[ 303 ]Clean Architecture
Chapter 10
All the configuration is passed by environment variables, so the code for our service will
have to comply with this norm.
In a more complex scenario involving more services and dependencies, we will not just run
the image of the created container, but instead declare a docker-compose.yml file with
the definitions of all the services, base images, and how they are linked and interconnected.
Now that we have the container running, we can launch it and run a small test on it to get
an idea of how it works:
$ curl http://localhost:8080/status/1
{"id":1,"status":"dispatched","msg":"Order was dispatched on
2018-08-01T22:25:12+00:00"}
Analysis
There are many conclusions to be drawn from the previous implementation. While it might
seem like a good approach, there are cons that come with the benefits; after all, no
architecture or implementation is perfect. This means that a solution such as this one cannot
be good for all cases, so it will pretty much depend on the circumstances of the project, the
team, the organization, and more.
While it's true that the main idea of the solution is to abstract details as much as possible, as
we shall see some parts cannot be fully abstracted away, and also the contracts between the
layers imply an abstraction leak.
The dependency flow
Notice that dependencies flow in only one direction, as they move closer to the kernel,
where the business rules lie. This can be traced by looking at the import statements. The
application imports everything it needs from storage, for example, and in no part is this
inverted.
Breaking this rule would create coupling. The way the code is arranged now means that
there is a weak dependency between the application and storage. The API is such that we
need an object with a get() method, and any storage that wants to connect to the
application needs to implement this object according to this specification. The dependencies
are therefore inverted—it's up to every storage to implement this interface, in order to
create an object according to what the application is expecting.
[ 304 ]Clean Architecture
Chapter 10
Limitations
Not everything can be abstracted away. In some cases, it's simply not possible, and in
others, it might not be convenient. Let's start with the convenience aspect.
In this example, there is an adapter of the web framework of choice to a clean API to be
presented to the application. In a more complex scenario, such a change might not be
possible. Even with this abstraction, parts of the library were still visible to the application.
Adapting an entire framework might not only be hard but also not possible in some cases.
It's not entirely a problem to be completely isolated from the web framework because,
sooner or later, we will need some of its features or technical details.
The important takeaway here is not the adapter, but the idea of hiding technical details as
much as possible. That means, that the best thing that was displayed on the listing for the
code of the application was not the fact that there was an adapter between our version of
the web framework and the actual one, but instead the fact that the latter was not
mentioned by name in any part of the visible code. The service was made clear that web
was just a dependency (a detail being imported), and revealed the intention behind what it
was supposed to do. The goal is to reveal the intention (as in the code) and to defer details
as much as possible.
As to what things cannot be isolated, those are the elements that are closest to the code. In
this case, the web application was using the objects operating within them in an
asynchronous fashion. That is a hard constraint we cannot circumvent. It's true that
whatever is inside the storage package can be changed, refactored, and modified, but
whatever these modifications might be, it still needs to preserve the interface, and that
includes the asynchronous interface.
Testability
Again, much like with the code, the architecture can benefit from separating pieces into
smaller components. The fact that dependencies are now isolated and controlled by
separate components leaves us with a cleaner design for the main application, and now it's
easier to ignore the boundaries to focus on testing the core of the application.
We could create a patch for the dependencies, and write unit tests that are simpler (they
won't need a database), or to launch an entire web service, for instance. Working with pure
domain objects means it will be easier to understand the code and the unit tests. Even the
adapters will not need that much testing because their logic should be very simple.
[ 305 ]Clean Architecture
Chapter 10
Intention revealing
These details included keeping functions short, concerns separated, dependencies isolated,
and assigning the right meaning to abstractions in every part of the code. Intention
revealing was a critical concept for our code—every name has to be wisely chosen, clearly
communicating what it's supposed to do. Every function should tell a story.
A good architecture should reveal the intent of the system it entails. It should not mention
the tools it's built with; those are details, and as we discussed at length, details should be
hidden, encapsulated.