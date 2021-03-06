.. .. meta::
   :description: How microservices work on a Hasura cluster
   :keywords: hasura, getting started, step 2

Viewing microservice logs
=========================

All STDOUT output, like ``print()``, ``console.log()``, etc. from microservices gets
captured as logs.

The command ``hasura microservice logs``, or ``hasura ms logs`` for short, will get logs for your microservice.


**Fetch all the logs till now of microservice `<my-app>` in one shot:**

.. code-block:: bash

   $ hasura ms logs <my-app>

**Fetch the last 100 log lines of microservice `<my-app>`:**

.. code-block:: bash

   $ hasura ms logs <my-app> --tail 100

**Stream (follow) logs of microservice `<my-app>`:**

The command will not exit and log lines will keep coming up on your terminal:

.. code-block:: bash

   $ hasura ms logs <my-app> -f

**Get logs of a hasura microservice, say `auth`**

.. code-block:: bash

   $ hasura ms logs auth -n hasura


For more details, read the full CLI reference for :doc:`hasura microservice logs <../hasuractl/hasura_platform:microservice_logs>`.
