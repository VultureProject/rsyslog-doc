********************************************
omsentinel: Sentinel Output Module
********************************************

===========================  ===========================================================================
**Module Name:**             **omsentinel**
**Author:**                  `Rainer Gerhards <https://rainer.gerhards.net/>`_ <rgerhards@adiscon.com>
===========================  ===========================================================================


Purpose
=======

This module provides native support for logging to Sentinel.


Configuration Parameters
========================

.. note::

   Parameter names are case-insensitive.


Action Parameters
-----------------

tenant_id
^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "none", "no", "none"

tenant_id for authentication purpose.


client_secret
^^^^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "none", "no", "none"

client_secret for authentication purpose.

client_id
^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "none", "no", "none"

client_id for authentication purpose.

scope
^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "https://monitor.azure.com/.default", "no", "none"

resource scope.


grant_type
^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "client_credentials", "no", "none"

authentication method.


dcr
^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "none", "no", "none"

Data Collection Rule to ingest data.


dce
^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "none", "no", "none"

Data Collection Endpoint.



stream_name
^^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "none", "no", "none"

Name of the stream where to ingest data.

healthchecktimeout
^^^^^^^^^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "integer", "3500", "no", "none"

The time after which the health check will time out in milliseconds.



batch
^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "binary", "off", "no", "none"


This parameter activates batching mode, which queues messages and sends them as a single request. There are several related parameters that specify the format and size of the batch: they are batch.format_, batch.maxbytes_, and batch.maxsize_.

Note that rsyslog core is the ultimate authority on when a batch must be submitted, due to the way that batching is implemented. This plugin implements the `output plugin transaction interface <https://www.rsyslog.com/doc/v8-stable/development/dev_oplugins.html#output-plugin-transaction-interface>`_. There may be multiple batches in a single transaction, but a batch will never span multiple transactions. This means that if batch.maxsize_ or batch.maxbytes_ is set very large, you may never actually see batches hit this size. Additionally, the number of messages per transaction is determined by the size of the main, action, and ruleset queues as well.

Additionally, due to some open issues with rsyslog and the transaction interface, batching requires some nuanced retry_ configuration.


batch.format
^^^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "newline", "no", "none"

This parameter specifies how to combine multiple messages into a single batch. Valid options are *newline* (default), *jsonarray*, *kafkarest*, and *lokirest*.

Each message on the "Inputs" line is the templated log line that is fed into the omsentinel action, and the "Output" line describes the resulting payload sent to the configured HTTP server.

1. *jsonarray* - Builds a JSON array containing all messages in the batch. This mode requires that each message is parseable JSON, since the plugin parses each message as JSON while building the array.

.. code-block:: text

    Inputs: {"msg": "message 1"} {"msg"": "message 2"} {"msg": "message 3"}
    Output: [{"msg": "message 1"}, {"msg"": "message 2"}, {"msg": "message 3"}]


batch.maxsize
^^^^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "Size", "100", "no", "none"

This parameter specifies the maximum number of messages that will be sent in each batch.

batch.maxbytes
^^^^^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "Size", "10485760 (10MB)", "no", "none"

batch.maxbytes and maxbytes do the same thing, maxbytes included for backwards compatibility.

This parameter specifies the maximum size in bytes for each batch.

template
^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "StdJSONFmt", "no", "none"

The template to be used for the messages.

Note that in batching mode, this describes the format of *each* individual message, *not* the format of the resulting batch. Some batch modes require that a template produces valid JSON.


retry
^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "binary", "off", "no", "none"

This parameter specifies whether failed requests should be retried using the custom retry logic implemented in this plugin. Requests returning 5XX HTTP status codes are considered retriable. If retry is enabled, set retry.ruleset_ as well.

Note that retries are generally handled in rsyslog by setting action.resumeRetryCount="-1" (or some other integer), and the plugin lets rsyslog know it should start retrying by suspending itself. This is still the recommended approach in the 2 cases enumerated below when using this plugin. In both of these cases, the output plugin transaction interface is not used. That is, from rsyslog core's point of view, each message is contained in its own transaction.


retry.ruleset
^^^^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "none", "no", "none"

This parameter specifies the ruleset where this plugin should requeue failed messages if retry_ is on. This ruleset generally would contain another omsentinel action instance.

**Important** - Note that the message that is queued on the retry ruleset is the templated output of the initial omsentinel action. This means that no further templating should be done to messages inside this ruleset, unless retries should be templated differently than first-tries. An "echo template" does the trick here.

.. code-block:: text

   template(name="tpl_echo" type="string" string="%msg%")

This retry ruleset can recursively call itself as its own retry.ruleset to retry forever, but there is no timeout behavior currently implemented.

Alternatively, the omsentinel action in the retry ruleset could be configured to support action.resumeRetryCount as explained above in the retry parameter section. The benefit of this approach is that retried messages still hit the server in a batch format (though with a single message in it), and the ability to configure rsyslog to give up after some number of resume attempts so as to avoid resource exhaustion.

Or, if some data loss or high latency is acceptable, do not configure retries with the retry ruleset itself. A single retry from the original ruleset might catch most failures, and errors from the retry ruleset could still be logged using the errorfile parameter and sent later on via some other process.

ratelimit.interval
^^^^^^^^^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "integer", "600", "no", "none"

This parameter sets the rate limiting behavior for the retry.ruleset_. Specifies the interval in seconds onto which rate-limiting is to be applied. If more than ratelimit.burst messages are read during that interval, further messages up to the end of the interval are discarded. The number of messages discarded is emitted at the end of the interval (if there were any discards). Setting this to value zero turns off ratelimiting.

ratelimit.burst
^^^^^^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "integer", "20000", "no", "none"

This parameter sets the rate limiting behavior for the retry.ruleset_. Specifies the maximum number of messages that can be emitted within the ratelimit.interval interval. For further information, see description there.


errorfile
^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "none", "no", "none"

Here you can set the name of a file where all errors will be written to. Any request that returns a 4XX or 5XX HTTP code is recorded in the error file. Each line is JSON formatted with "request" and "response" fields, example pretty-printed below.

.. code-block:: text

    {
        "request": {
            "url": "https://url.com:443/path",
            "postdata": "mypayload"
        },
        "response" : {
            "status": 400,
            "message": "error string"
        }
    }

It is intended that a full replay of failed data is possible by processing this file.

compress
^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "binary", "off", "no", "none"

When switched to "on" each message will be compressed as GZIP using zlib's deflate compression algorithm.

A "Content-Encoding: gzip" HTTP header is added to each request when this feature is used. Set the compress.level_ for fine-grained control.

compress.level
^^^^^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "integer", "-1", "no", "none"

Specify the zlib compression level if compress_ is enabled. Check the `zlib manual <https://www.zlib.net/manual.html>`_ for further documentation.

"-1" is the default value that strikes a balance between best speed and best compression. "0" disables compression. "1" results in the fastest compression. "9" results in the best compression.


tls.cacert
^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "none", "no", "none"

This parameter sets the path to the Certificate Authority (CA) bundle. Expects .pem format.

tls.mycert
^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "none", "no", "none"

This parameter sets the path to the SSL client certificate. Expects .pem format.

tls.myprivkey
^^^^^^^^^^^^^

.. csv-table::
   :header: "type", "default", "mandatory", "|FmtObsoleteName| directive"
   :widths: auto
   :class: parameter-table

   "word", "none", "no", "none"

The parameters sets the path to the SSL private key. Expects .pem format.


Statistic Counter
=================

This plugin maintains global :doc:`statistics <../rsyslog_statistic_counter>` for omsentinel that
accumulates all action instances. The statistic origin is named "omsentinel" with following counters:

- **messages.submitted** - Number of messages submitted to omsentinel. Messages resubmitted via a retry ruleset will be counted twice.

- **messages.success** - Number of messages successfully sent.

- **messages.fail** - Number of messages that omsentinel failed to deliver for any reason.

- **messages.retry** - Number of messages that omsentinel resubmitted for retry via the retry ruleset.

- **request.count** - Number of attempted HTTP requests.

- **request.success** - Number of successful HTTP requests. A successful request can return *any* HTTP status code.

- **request.fail** - Number of failed HTTP requests. A failed request is something like an invalid SSL handshake, or the server is not reachable. Requests returning 4XX or 5XX HTTP status codes are *not* failures.

- **request.status.success** - Number of requests returning 1XX or 2XX HTTP status codes.

- **request.status.fail** - Number of requests returning 3XX, 4XX, or 5XX HTTP status codes. If a requests fails (i.e. server not reachable) this counter will *not* be incremented.


Examples
========

Example 1
---------

The following example is a basic usage, first the module is loaded and authenticated


.. code-block:: text

    module(load="omsentinel")
    template(name="tpl1" type="string" string="{\"type\":\"syslog\", \"host\":\"%HOSTNAME%\"}")
    action(
        type="omsentinel"
        tenant_id="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        client_secret="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.xxxxxxx"
        client_id="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        dcr="dcr-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
        dce="xxxxxxxxxxxxxxxxxxxxx.ingest.monitor.azure.com"
        stream_name="STREAM_NAME"
        template="tpl1"
    )

