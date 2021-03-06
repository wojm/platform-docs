.. .. meta::
   :description: Hasura Auth mobile provider
   :keywords: hasura, users, signup, login, mobile, verify mobile


Mobile / password based authentication
====================================

This provider supports mobile/password based authentication.  The user's mobile
is verified by sending an OTP via SMS to the user's mobile number. To use this
provider there are extra steps to be performed to verify the user's mobile.

.. note::

  For this provider to send an OTP via SMS, you have to :doc:`enable an SMS provider <../../../notify/sms/index>` in
  the Hasura notify microservice.

Configuration
-------------

You can configure Mobile/password provider settings in the :doc:`conf/auth.yaml <../../../project/directory-structure/conf/auth.yaml>` file in your project. Find a top level key called ``mobilePassword`` in the ``auth.yaml``. By default the mobile-password conf looks like this:

.. snippet:: yaml
    :filename: auth.yaml

    mobilePassword:
      # Template for the SMS that is sent. This is a Jinja template. Leave the
      # "{{otp}}" as it is. It will be used by the auth service to inject the
      # actual token.
      smsTemplate: |
        Verify your acccount with {{ cluster.name }}! Your OTP is {{ "{{otp}}" }}.
      # OTP expiry time in minutes
      otpExpiryTime: "15"
      # OTP length is optional. default value is 6
      # otpLength: "4"

You can modify it as you wish and then apply the modifcations to the cluster by running a git push:

.. code-block:: bash

  $ git add conf/auth.yaml
  $ git commit -m "Changed conf for Mobile/Password provider"
  $ git push hasura master

API
---

Signup
~~~~~~

To signup a user, make a request to the signup endpoint : ``/v1/signup``.

.. code-block:: http

   POST auth.<cluster-name>.hasura-app.io/v1/signup HTTP/1.1
   Content-Type: application/json

   {
     "provider" : "mobile-password",
     "data" : {
        "mobile": "9876543210",
        "country_code": "91",
        "password": "somepass123"
     }
   }

If the request is successful, Hasura auth will send a verification SMS to the
given mobile number and will return a response with the user details.


This will not login the user automatically (unlike the ``username`` provider),
because at this point the mobile verification is pending.

The above user details will not have ``auth_token`` set.

Typical response of the ``/v1/signup`` request is :

.. code-block:: http

   HTTP/1.1 200 OK
   Content-Type: application/json

   {
     "auth_token": null,
     "mobile": "9876543210",
     "hasura_roles": [
       "user"
     ],
     "hasura_id": 79
   }


* ``auth_token`` is the authentication token of the user for the current
  session. This is null because at this point the mobile verification is
  pending, hence no session is created for the user.

* ``hasura_roles`` is a list of all roles assigned to the user.

* ``hasura_id`` is the Hasura identifier of the user.


Verify mobile
~~~~~~~~~~~~~

To verify the mobile number, Hasura auth will send an SMS with a one time
password or OTP to the user's mobile number, and within a configurable amount of
time, the user has to submit the OTP to a Hasura auth API endpoint to verify
the mobile number.

To verify the mobile number, make the following request.

.. code-block:: http

   POST auth.<cluster-name>.hasura-app.io/v1/providers/mobile-password/verify-otp HTTP/1.1
   Content-Type: application/json

   {
     "mobile": "9876543210",
     "country_code": "91",
     "otp": "123456"
   }

The response of the mobile verification endpoint indicates success or failure.
If it is successful, then your application should ask the user to login.

.. code-block:: http

   HTTP/1.1 200 OK
   Content-Type: application/json

   {
     "message" : "success"
   }


Login
~~~~~

To login a user make a request to the login endpoint: ``/v1/login``.

.. code-block:: http

   POST auth.<cluster-name>.hasura-app.io/v1/login HTTP/1.1
   Content-Type: application/json

   {
     "provider": "mobile-password",
     "data": {
        "mobile": "9876543210",
        "country_code": "91",
        "password": "somepass123"
     }
   }


Typical response of the ``/v1/login`` request is :

.. code-block:: http

   HTTP/1.1 200 OK
   Content-Type: application/json

   {
     "auth_token": "b4b345f980ai4acua671ac7r1c37f285f8f62e29f5090306",
     "mobile": "9876543210",
     "hasura_id": 79,
     "hasura_roles": [
         "user"
     ]
   }

* ``auth_token`` is the authentication token of the user for the current
  session.
* ``hasura_roles`` is an array of all roles assigned to the user.

* ``hasura_id`` is the Hasura identifier of the user.


Get user info
~~~~~~~~~~~~~

To get the logged in user's details, or to check if a session token is valid
you can use this endpoint.

Make a request to the endpoint: ``/v1/user/info``.

.. code-block:: http

   GET auth.<cluster-name>.hasura-app.io/v1/user/info HTTP/1.1
   Content-Type: application/json
   Authorization: Bearer <auth_token>


Typical response is :

.. code-block:: http

   HTTP/1.1 200 OK
   Content-Type: application/json

   {
     "auth_token": "b4b345f980ai4acua671ac7r1c37f285f8f62e29f5090306",
     "mobile": "9876543210",
     "hasura_id": 79,
     "hasura_roles": [
         "user"
     ]
   }


* ``auth_token`` is the authentication token of the user for the current
  session.
* ``hasura_roles`` is an array of all roles assigned to the user.

* ``hasura_id`` is the Hasura identifier of the user.


Logout
~~~~~~

To logout a user, make the following request.

.. code-block:: http

   POST auth.<cluster-name>.hasura-app.io/v1/user/logout HTTP/1.1
   Authorization: Bearer <auth_token>

.. note::
    The logout request is a POST request with an empty body.


Change password
~~~~~~~~~~~~~~~

If the user is logged in, they can change their password using the following
endpoint.

.. code-block:: http

   POST auth.<cluster-name>.hasura-app.io/v1/user/change-password HTTP/1.1
   Authorization: Bearer <auth_token>

   {
     "old_password": "oldpassword",
     "new_password": "newpassword"
   }

.. _forgot_password_mobile_password:

Forgot password / password reset
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If a user has forgotten their password, it can be reset.

.. note::

  This process is meant for users who have forgotten their password and
  can't login. For logged-in user to change their password use
  ``/v1/user/change-password`` endpoint.

To reset a password first a reset OTP has to be obtained. This is done by sending
a forgot password SMS to the user's mobile.

To send a forgot password SMS make a request to
``/v1/providers/mobile-password/forgot-password`` endpoint with the user's
mobile number.

.. code-block:: http

   POST auth.<cluster-name>.hasura-app.io/v1/providers/mobile-password/forgot-password HTTP/1.1
   Content-Type: application/json

   {
     "mobile" : "9876543210",
     "country_code" : "91"
   }

After obtaining the OTP, your application should make auth API call to the
``/v1/providers/mobile-password/reset-password`` endpoint to reset the user's password.

The reset password endpoint takes the OTP and the new password of the user.
  
.. code-block:: http

   POST auth.<cluster-name>.hasura-app.io/v1/providers/mobile-password/reset-password HTTP/1.1
   Content-Type: application/json

   {
     "mobile" : "9876543210",
     "country_code" : "91",
     "otp": "1231",
     "password": "newpass123"
   }
