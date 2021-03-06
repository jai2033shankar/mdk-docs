======================
Web Client Integration
======================

The MDK provide integration points for common HTTP client frameworks.
The number of supported frameworks will grow over time.

Hooking up the MDK will do the following:

1. Timeouts will be extracted from the MDK session and used to set the request timeout.
2. The given MDK session will be transmitted to the destination server via a ``X-MDK-CONTEXT`` HTTP header.

The session may contain sensitive information, so this functionality should only be used within servers under your own control.

.. contents:: Integrations
   :local:

JavaScript
==========

Request
-------

To use MDK with the `Request <https://github.com/request/request>`_ library you can ``npm install datawire_mdk_request``.
In particular, ``forMDKSession`` creates an object that looks like a ``request`` object but has timeout and MDK session header support:

.. code-block:: javascript

   var mdk = require('datawire_mdk').mdk;
   var process = require('process');
   var mdk_request = require('datawire_mdk_request');

   mdk = mdk.start();
   process.on('exit', function () {
       mdk.stop();
   });

   var mdkSession = mdk.session();
   mdkSession.setTimeout(1.0);

   var requestMDK = mdk_request.forMDKSession(mdkSession);
   requestMDK("http://example.com", function (error, response, body) {
       console.log(body);
   });
   // You can also do requestMDK.get(...), etc..


Python
======

Requests
--------

To use MDK with the `Requests <https://requests.readthedocs.io>`_ library you can use the ``mdk.requests.requests_session`` API, which creates a `requests.Session <http://requests.readthedocs.io/en/master/user/advanced/#session-objects>`_ object.
For example, if you're using the MDK Flask integration:

.. code-block:: python

   from flask import g, Flask

   from mdk.flask import mdk_setup
   from mdk.requests import requests_session

   app = Flask(__name__)

   @app.route("/")
   def proxy():
       # Lookup backend server using MDK Discovery.
       node = g.mdk_session.resolve("backend_service", "1.0")

       # Use requests to do a HTTP GET that includes the MDK
       # session context and a timeout derived from the session
       # timeout:
       req_ssn = requests_session(g.mdk_session)
       return req_ssn.get(node.address).text

   if __name__ == '__main__':
       mdk_setup(app, timeout=10.0)
       app.run()


Ruby
====

Faraday
-------

To use MDK with the `Faraday <https://github.com/lostisland/faraday>`_ library you can ``gem install faraday_mdk`` and then:

.. code-block:: ruby

   require 'faraday'
   require 'mdk'
   require 'faraday_mdk'

   mdk = ::Quark::Mdk.start
   session = mdk.session
   session.setTimeout(1.0)
   conn = Faraday.new(:url => ARGV[0]) do |faraday|
     # Add middleware for the MDK session:
     faraday.request :mdk_session, session
     faraday.adapter  Faraday.default_adapter
   end

   response = conn.get
   puts(response.body)
   mdk.stop
