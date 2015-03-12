# ATC Api

ATC API is a Django app that allow to bridge a REST API to ATCD's thrift API.

# Setup

## Requirements

* [Django 1.7](https://github.com/django/django)
* [Django REST framework 3.X](https://github.com/tomchristie/django-rest-framework)
* [atc_thrift](../atc_thrift)

## Installation

```python
$ cd path/to/django-atc-api
pip install .
```

## Configuration

1. Edit your Django project's settings.py and add "atc_api" to your INSTALLED_APPS::

```python
    INSTALLED_APPS = (
        ...
        'atc_api',
        'rest_framework',
    )
```

2. Include the atc_api URLconf in your Django project urls.py like this::

```python
    url(r'^api/v1/', include('atc_api.urls')),
```

3. Start the development server

4. Visit http://127.0.0.1:8000/api/v1/shape/ to set/unset shaping.


Some settings like the `ATCD_HOST` and `ATCD_PORT` can be changes in your Django projects'settinggs.py::

```python
ATC_API = {
    'ATCD_HOST': 'localhost',
    'ATCD_PORT': 9090,
}
```

see [ATC api settings](atc_api/settings.py)


# API usage

Let's suppose the api is available under `/api/v1`. The core API is limited and allow to:

* Getting the shaping staus of an device by GETing `/api/v1/shape/`
* Shape a device by POSTing to `/api/v1/shape/`
* Unshape a device by sending a DELETE request to `/api/v1/shape/`

## Shaping Status

To find out if a device is shaped, you can GET `/api/v1/shape/[ip/]`

If the device is being shaped, HTTP will return 200 and the current shaping of the device.

If the device is not being shaped, HTTP will return code 404.

Examples:

* Check if I am being shaped (device not being shaped, HTTP code 404):

```sh
$ curl -H 'Accept: application/json; indent=2' http://127.0.0.1:8080/api/v1/shape/
{
  "detail": "This IP (10.0.2.2) is not being shaped"
}
```

* Check if I am being shaped (device being shaped, HTTP code 200):

```sh
$ curl -H 'Accept: application/json; indent=2' http://127.0.0.1:8080/api/v1/shape/
{
  "down": {
    "rate": 400,
    "loss": {
      "percentage": 5.0,
      "correlation": 0.0
    },
    "delay": {
      "delay": 15,
      "jitter": 0,
      "correlation": 0.0
    },
    "corruption": {
      "percentage": 0.0,
      "correlation": 0.0
    },
    "reorder": {
      "percentage": 0.0,
      "correlation": 0.0,
      "gap": 0
    }
  },
  "up": {
    "rate": 200,
    "loss": {
      "percentage": 1.0,
      "correlation": 0.0
    },
    "delay": {
      "delay": 10,
      "jitter": 0,
      "correlation": 0.0
    },
    "corruption": {
      "percentage": 0.0,
      "correlation": 0.0
    },
    "reorder": {
      "percentage": 0.0,
      "correlation": 0.0,
      "gap": 0
    }
  }
}
```

* Check if 1.1.1.1 is being shaped (device not being shaped, HTTP code 404):

```sh
$ curl -H 'Accept: application/json; indent=2' http://127.0.0.1:8080/api/v1/shape/1.1.1.1/
{
  "detail": "This IP (1.1.1.1) is not being shaped"
}
```

## Shaping a device

Shaping a device is done by posting the shaping setting payload to `/api/v1/shape/[ip/]`


Examples:

* Shape my own device, 200kb up, added latency of 10ms with 1% packet loss and 400kb down with added latency of 15ms and 5% packet loss
This will always retun HTTP code 201 on success. If the device was already being shaped, the new setting is going to be applied and the onld one deleted.

Mind the (Ctrl-D)

```sh
$ curl -X POST -d '@-' -D - -H 'Content-Type: application/json' -H 'Accept: application/json; indent=2' http://127.0.0.1:8080/api/v1/shape/
{
    "down": {
        "rate": 400,
        "loss": {
            "percentage": 5.0,
            "correlation": 0.0
        },
        "delay": {
            "delay": 15,
            "jitter": 0,
            "correlation": 0.0
        },
        "corruption": {
            "percentage": 0.0,
            "correlation": 0.0
        },
        "reorder": {
            "percentage": 0.0,
            "correlation": 0.0,
            "gap": 0
        }
    },
    "up": {
        "rate": 200,
        "loss": {
            "percentage": 1.0,
            "correlation": 0.0
        },
        "delay": {
            "delay": 10,
            "jitter": 0,
            "correlation": 0.0
        },
        "corruption": {
            "percentage": 0.0,
            "correlation": 0.0
        },
        "reorder": {
            "percentage": 0.0,
            "correlation": 0.0,
            "gap": 0
        }
    }
}
Ctrl-D
HTTP/1.1 201 CREATED
Server: gunicorn/19.2.1
Date: Fri, 27 Feb 2015 20:02:05 GMT
Connection: close
Transfer-Encoding: chunked
Vary: Accept, Cookie
Content-Type: application/json; indent=2
Allow: GET, POST, DELETE, HEAD, OPTIONS

{
  "down": {
    "rate": 400,
    "loss": {
      "percentage": 5.0,
      "correlation": 0.0
    },
    "delay": {
      "delay": 15,
      "jitter": 0,
      "correlation": 0.0
    },
    "corruption": {
      "percentage": 0.0,
      "correlation": 0.0
    },
    "reorder": {
      "percentage": 0.0,
      "correlation": 0.0,
      "gap": 0
    }
  },
  "up": {
    "rate": 200,
    "loss": {
      "percentage": 1.0,
      "correlation": 0.0
    },
    "delay": {
      "delay": 10,
      "jitter": 0,
      "correlation": 0.0
    },
    "corruption": {
      "percentage": 0.0,
      "correlation": 0.0
    },
    "reorder": {
      "percentage": 0.0,
      "correlation": 0.0,
      "gap": 0
    }
  }
}
```

or... more simply:

```sh
$ curl -H 'Accept: application/json; indent=2' http://127.0.0.1:8080/api/v1/shape/
{
    "down": {
        "rate": 400, 
        "loss": {
            "percentage": 5.0
        }, 
        "delay": {
            "delay": 15
        }, 
        "corruption": {}, 
        "reorder": {}
    }, 
    "up": {
        "rate": 200, 
        "loss": {
            "percentage": 1.0
        }, 
        "delay": {
            "delay": 10
        }, 
        "corruption": {}, 
        "reorder": {}
    }
}
CTRL-D
... same response...
```

Likely, device 1.1.1.1 could be shaped by using URL http://127.0.0.1:8080/api/v1/shape/1.1.1.1/ instead.

## Unshaping a device

Unshaping a device is done by sending a DELETE request to `/api/v1/shape/[ip]/`

Examples:

* Unshape myself (device being shaped, HTTP code 204)

```sh
$ curl -X DELETE -D - -H 'Accept: application/json; indent=2' http://127.0.0.1:8080/api/v1/shape/
HTTP/1.1 204 NO CONTENT
Server: gunicorn/19.2.1
Date: Fri, 27 Feb 2015 19:46:58 GMT
Connection: close
Vary: Accept, Cookie
Content-Length: 0
Allow: GET, POST, DELETE, HEAD, OPTIONS

```

* Unshape myself (device not being shaped, HTTP code 400):

```sh
$ curl -X DELETE -D - -H 'Accept: application/json; indent=2' http://127.0.0.1:8080/api/v1/shape/
HTTP/1.1 400 BAD REQUEST
Server: gunicorn/19.2.1
Date: Fri, 27 Feb 2015 19:43:36 GMT
Connection: close
Transfer-Encoding: chunked
Vary: Accept, Cookie
Content-Type: application/json; indent=2
Allow: GET, POST, DELETE, HEAD, OPTIONS

{
  "detail": "{'message': 'No session for IP 10.0.2.2 found', 'result': 12}"
}
```

* Unshape 1.1.1.1 (device not being shaped, HTTP code 400):

```sh
 curl -X DELETE -D - -H 'Accept: application/json; indent=2' http://127.0.0.1:8080/api/v1/shape/1.1.1.1/
HTTP/1.1 400 BAD REQUEST
Server: gunicorn/19.2.1
Date: Fri, 27 Feb 2015 19:47:57 GMT
Connection: close
Transfer-Encoding: chunked
Vary: Accept, Cookie
Content-Type: application/json; indent=2
Allow: GET, POST, DELETE, HEAD, OPTIONS

{
  "detail": "{'message': 'No session for IP 1.1.1.1 found', 'result': 12}"
}
```