---
layout: post
title:  "Asymmetric (Public-key) JWT setup with Django REST Framework"
date:   2017-04-05 8:00:00 -0600
categories: 
---
Example
----------
A completed example can be found [here][gh-source]

Info
----------
Django REST Framework JWT defaults to HS256 which is a [Symmetric-key algorithm][wiki-symmetric-key] utilizing the secret key in `settings.py`. This is an example using RS256 which is an [Asymmetric algorithm][wiki-asym-key] that uses a public/private key pair.  

Advantages
---------
Utilizing an Asymmetric algorithm allows you to have clients who can verify the JWT signature without having to expose the private key because client's only need the public key to verify the signature. Symmetric-key algorithms (HS256) use the same key for signing and verifying the signature requiring control over both the server and client. With an asymmetric algorithm control is only needed over the private key and the public key can be safely given to clients for signature verification. 

Installation
----------

Using pip install [REST framework JWT Auth][drf-jwt-gh] and cryptography
{% highlight shell %}
pip install djangorestframework-jwt
pip install cryptography
{% endhighlight %}

Setup
----------

Navigate to the project's root directory (where `manage.py` is located) and generate the private and public keys using opensh
{% highlight shell %}
openssl genrsa -out privkey.pem 2048
openssl rsa -in privkey.pem -pubout -out pubkey.pem
{% endhighlight %}

In `settings.py` make sure REST_FRAMEWORK is using `rest_framework_jwt.authentication.JSONWebTokenAuthentication`

{% highlight python %}
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticated',
    ),
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_jwt.authentication.JSONWebTokenAuthentication',
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.BasicAuthentication',
    ),
}
{% endhighlight %}

Still in `settings.py` enter the following settings to use our newly generated private and public keys.

{% highlight python %}
JWT_AUTH = {
    'JWT_PUBLIC_KEY': open(BASE_DIR + '/pubkey.pem', 'rb').read(),
    'JWT_PRIVATE_KEY': open(BASE_DIR + '/privkey.pem', 'rb').read(),
    'JWT_ALGORITHM': 'RS256',
}
{% endhighlight %}

In your `urls.py` add the JWT endpoints
{% highlight python %}
from rest_framework_jwt.views import obtain_jwt_token, refresh_jwt_token, verify_jwt_token

urlpatterns = [
    # ...
    url(r'^auth-token/$', obtain_jwt_token),
    url(r'^auth-token-refresh/$', refresh_jwt_token),
    url(r'^auth-token-verify/$', verify_jwt_token),
]
{% endhighlight %}

Usage
----------
Test that the webservice is properly generating tokens.
{% highlight shell %}
curl -X POST -H "Content-Type: application/json" -d '{"username":"admin","password":"password123"}' http://localhost:8000/auth-token/
{% endhighlight %}

Verify the token with
{% highlight shell %}
curl -X POST -H "Content-Type: application/json" -d '{"token":"<EXISTING_TOKEN>"}' http://localhost:8000/auth-token-verify/
{% endhighlight %}

To access protected api urls add the header: `Authorization: JWT <your_token>`.
{% highlight shell %}
curl -H "Authorization: JWT <your_token>" http://localhost:8000/protected-url/
{% endhighlight %}

Additionally you can use the [JWT Website][jwt-web] to verify the signature using only the public key. On the JWT website change the Algorithm to RS256, paste your token and public key. If the signature is verified, congratulations! You've correctly setup JWT with Asymmetric keys. 

[gh-source]: https://github.com/goodmase/rsa-drf-jwt-example
[drf-jwt-gh]: https://github.com/GetBlimp/django-rest-framework-jwt
[jwt-web]: https://jwt.io/
[wiki-symmetric-key]: https://en.wikipedia.org/wiki/Symmetric-key_algorithm
[wiki-asym-key]: https://en.wikipedia.org/wiki/Public-key_cryptography
