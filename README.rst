graphql-ws-aiohttp
===========================

|code_ci| |coverage| |pypi_version| |license|

Port of `graphql-ws-next`_ for `graphql-core v3`_.

A GraphQL WebSocket server and client to facilitate GraphQL queries, mutations and subscriptions over WebSocket (for Python 3.6+).
This code is based on the current implementation of `subscriptions-transport-ws <https://github.com/apollographql/subscriptions-transport-ws>`_.

============== ==============================================================
PyPI           ``pip install graphql-ws-aiohttp``
Source code    https://github.com/dls-controls/graphql-ws-aiohttp
Changelog      https://github.com/dls-controls/graphql-ws-aiohttp/blob/master/CHANGELOG.rst
============== ==============================================================


Getting Started
===============

Start by installing the package using pip:

.. code: shell

    pip install graphql-ws-next

Or, by using your favorite package manager, like `Poetry <https://github.com/sdispater/poetry>`_:

.. code: shell

    poetry add graphql-ws-next


With ``aiohttp``
================

Usage with ``aiohttp`` is simple:

.. code: python

    import aiohttp.web
    import graphql_ws
    from graphql_ws.aiohttp import AiohttpConnectionContext

    async def handle_subscriptions(
        request: aiohttp.web.Request
    ) -> aiohttp.web.WebSocketResponse:
        wsr = aiohttp.web.WebSocketResponse(protocols=(graphql_ws.WS_PROTOCOL,))
        request.app["websockets"].add(wsr)
        await wsr.prepare(request)
        await request.app["subscription_server"].handle(wsr, None)
        request.app["websockets"].remove(wsr)
        return wsr

    def make_app(schema: graphql.GraphQLSchema) -> aiohttp.web.Application:
        app = aiohttp.web.Application()
        app.router.add_get("/subscriptions", handle_subscriptions)

        app["subscription_server"] = graphql_ws.SubscriptionServer(
            schema, AiohttpConnectionContext
        )
        app["websockets"] = set()

        async def on_shutdown(app):
            await asyncio.wait([wsr.close() for wsr in app["websockets"]])

        app.on_shutdown.append(on_shutdown)
        return app

    if __name__ == '__main__':
        app = make_app(schema)  # you supply your GraphQLSchema
        aiohttp.web.run_app()


For other frameworks
====================

Adding support for other web frameworks is simple.
A framework must provide a concrete implementation of ``graphql_ws.abc.AbstractConnectionContext``, and then it's ready to use with the ``SubscriptionServer``.

Usage
=====

Using `apollo-link-ws <https://github.com/apollographql/apollo-link/tree/master/packages/apollo-link-ws>`_ you can opt to use websockets for queries and mutations in addition to subscriptions.

Use it with GraphiQL
====================

Look in the `demo<./demo>_` directory to see usage examples for GraphiQL.
Due to the implementation of the javascript client for GraphiQL (`GraphiQL-Subscriptions-Fetcher <https://github.com/apollographql/GraphiQL-Subscriptions-Fetcher>`_), queries and mutations will not be handled over websocket.

Contributing
============

This project uses `Poetry <https://github.com/sdispater/poetry>`_, so to contribute, simply fork and clone this repository, and then set up your virtual environment using:

.. code: shell:

    cd graphql-ws-next
    poetry develop .

If you don't yet have Poetry installed, please follow the `documentation for installation <https://poetry.eustace.io/docs/#installation>`_.

Code formatting is done via `black <https://github.com/ambv/black>`_, and code should be well-typed using `mypy <https://github.com/python/mypy>`_.


License
=======
This package is licensed under the MIT License.

.. _`graphql-ws-next`: https://github.com/dfee/graphql-ws-next
.. _`graphql-core v3`: https://github.com/graphql-python/graphql-core

.. |code_ci| image:: https://github.com/dls-controls/graphql-ws-aiohttp/workflows/Code%20CI/badge.svg?branch=master
    :target: https://github.com/dls-controls/graphql-ws-aiohttp/actions?query=workflow%3A%22Code+CI%22
    :alt: Code CI

.. |coverage| image:: https://codecov.io/gh/dls-controls/graphql-ws-aiohttp/branch/master/graph/badge.svg
    :target: https://codecov.io/gh/dls-controls/graphql-ws-aiohttp
    :alt: Test Coverage

.. |pypi_version| image:: https://img.shields.io/pypi/v/graphql-ws-aiohttp.svg
    :target: https://pypi.org/project/graphql-ws-aiohttp
    :alt: Latest PyPI version

.. |license| image:: https://img.shields.io/badge/License-Apache%202.0-blue.svg
    :target: https://opensource.org/licenses/Apache-2.0
    :alt: Apache License
