.. _hasura_platform:cluster_template-context:

Hasura CLI: hasura platform:cluster template-context
----------------------------------------------------

Show the context used to template files

Synopsis
~~~~~~~~


Get the context for a cluster that is used to render templates for configuration.

::

  hasura platform:cluster template-context [flags]

Examples
~~~~~~~~

::


    # Get the context for cluster called 'hasura'
    $ hasura cluster template-context


Options
~~~~~~~

::

  -c, --cluster string   alias of the cluster to be contacted
  -h, --help             help for template-context

Options inherited from parent commands
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

      --project string   hasura project directory where the commands should be executed. (default: current directory)

SEE ALSO
~~~~~~~~

* :ref:`hasura platform:cluster <hasura_platform:cluster>` 	 - Manage hasura clusters

*Auto generated by spf13/cobra*
