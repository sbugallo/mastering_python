4. Documenting web APIs
***********************

The principles for documenting web APIs are almost the same as for other kinds of
software. You want to properly target your readership, provide documentation in a way
and form that is native for the usage environment (here, as a web page), and, most of all,
make sure that readers have access to the up to date and relevant version of your
documentation.

Because of this, it is extremely important to have your documentation of web APIs
generated from the sources of the code that provides these APIs. Unfortunately, due to the
complex architecture of most web frameworks, classical documentation tools like Sphinx
are rarely useful for documenting typical HTTP endpoints of web APIs. In this context, it is
very common that auto-documentation capabilities are built into your web framework of
choice. These kind of frameworks either serve user-readable documentation by themselves
or serve a standardized API description in a machine-readable format that can be later
processed with a specialized documentation browser.

There is also another completely different philosophy for documenting web APIs, and it is
based on the idea of API prototyping. Tools for API prototyping allow you to use
documentation as a software contract that can be used as an API stub, even before service
development starts. Often, this kind of tool allows you to automatically verify if the API
structure matches the one actually implemented in the service. In this approach,
documentation may serve the additional function of an API testing tool.

4.1. Documentation as API prototype with API Blueprint
++++++++++++++++++++++++++++++++++++++++++++++++++++++

API Blueprint is a web API description language that is both human-readable and well-
defined. You can think of it like a Markdown for web service description language. It
allows documenting anything from the structure of URL paths, through body structures of
HTTP request/responses and headers, to complex request-response exchanges. The
following is an example of an imaginary Cat API described using API Blueprint:

.. code-block:: none

    FORMAT: 1A
    HOST: https://cats-api.example.com

    # Cat API
    This API Blueprint demonstrates example documentation of some imaginary Cat API.

    # Group Posts
    This section groups Cat resources.

    ## Cat [/cats/{cat_id}]

    A Cat is central and only resource utilized by Cat API.

    + Parameters
        + cat_id: `1` (string) - The id of the Cat.

    + Model (application/json)
        ```js
        {
        "data": {
        "id": "1", // note this is a string
        "breed": "Maine Coon",
        "name": "Smokey"
        ```

    ### Retrieve a Cat [GET]

    Returns a specific Cat.

    + Response 200

        [Cat][]

    ### Create a Cat [POST]

    Create a new Post object. Mentions and hashtags will be parsed out of the post text, as will bare URLs...

    + Request

        [Cat][]

    + Response 201
        [Cat][]

API Blueprint alone is nothing more than a language. Its strength really comes from the fact
that it can be easily written by hand and from the huge selection of tools supporting that
language. At the time of writing this, the official API Blueprint page lists over 70 tools
that support this language. Some of these tools can even generate functional API mock
servers that are meant to shorten development cycles, as mock servers can be used, for
instance, by frontend code, even before programmers start the development of backend
API services.

4.2. Self-documenting APIs with Swagger/OpenAPI
+++++++++++++++++++++++++++++++++++++++++++++++

While self-documenting APIs is a more traditional approach for documenting web APIs
(compared to documenting through API prototypes), we can clearly see some interesting
trends that appeared during the past few years. In the past, when API frameworks had to
support auto-documentation capabilities, it almost always meant that the framework had a
built-in API metadata structure with a custom documentation rendering engine. If someone
wanted to have multiple services auto-documented, they had to use the same framework
for every service, or decide to have very a inconsistent documentation landscape.

With the advent of microservice architectures, this approach becomes extremely
inconvenient and inefficient. Nowadays, it's very common that services within the same
projects are written using different frameworks, libraries, and even using completely
different programming languages. Having different documentation libraries for every
framework and language would produce very inconsistent documentation, as every tool
would have different strengths and weaknesses.

One approach that solves this problem requires splitting the documentation display
(rendering and browsing) from the actual documentation definition. This
approach is analogous to API prototyping because it requires a standardized API definition
language. But here, the developer rarely uses this language explicitly. It is the framework's
responsibility to create a machine-readable API definition from the structure of the code
written with this framework.

One such machine-readable web API description languages is OpenAPI. The specification
of OpenAPI is the result of the development of the popular Swagger documentation tool.
At first, it was an internal metadata format of the Swagger tool, but once it became
standardized, many tools around that specification appeared. With OpenAPI, many web
frameworks can describe their API structure using the same metadata format, so their
documentation can be rendered in the same consistent form by a single documentation
browser.