````markdown
# 📘 HTTP Error Code Transformation Using Kong Post Function Plugin

---

## 📋 Table of Contents

- [🛠️ Prerequisites](#-prerequisites)
- [🚀 Create a Sample Test Service](#-create-a-sample-test-service)
- [🔗 Add a Route to the Service](#-add-a-route-to-the-service)
- [🧪 Test the API Before Adding the Post Function Plugin](#-test-the-api-before-adding-the-post-function-plugin)
- [➕ Add Post Function Plugin](#-add-post-function-plugin)
- [🧪 Test API After Adding Post Function Plugin](#-test-api-after-adding-post-function-plugin)
- [📝 Summary](#-summary)

---

## 🛠️ Prerequisites

- Kong Gateway running locally (Admin API on port 8001, Proxy on port 8000)
- Admin token available (`<admin-token>`)
- `curl` command-line tool installed
- Internet connectivity to access `https://httpbin.konghq.com`

---

## 🚀 Create a Sample Test Service

Create a sample service that points to the HTTPBin status endpoint:

```bash
curl -s -X POST http://localhost:8001/plugin-demos/services \
  -H 'Kong-Admin-Token:<admin-token>' \
  -d "name=post_function_demo_svc" \
  -d "url=https://httpbin.konghq.com/status"
````

**Output:**

```json
{
  "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "created_at": 1752239999,
  "updated_at": 1752239999,
  "name": "post_function_demo_svc",
  "url": "https://httpbin.konghq.com/status",
  ...
}
```

---

## 🔗 Add a Route to the Service

Create a route named `post_function_demo_rt` for the service:

```bash
curl -i -X POST http://localhost:8001/plugin-demos/services/post_function_demo_svc/routes \
  -H 'Kong-Admin-Token:<admin-token>' \
  --data "name=post_function_demo_rt" \
  --data "paths[]=/http-change-code"
```

**Output:**

```http
HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8
Content-Length: 456
...
{
  "id": "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy",
  "name": "post_function_demo_rt",
  "paths": ["/http-change-code"],
  "service": {
    "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  },
  ...
}
```

---

## 🧪 Test the API Before Adding the Post Function Plugin

Test the API endpoint for status code 200:

```bash
curl -i http://localhost:8000/http-change-code/200
```

**Output:**

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 0
Connection: keep-alive
Server: gunicorn/19.9.0
Date: Fri, 11 Jul 2025 12:55:19 GMT
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
X-Kong-Upstream-Latency: 178
X-Kong-Proxy-Latency: 1
Via: 1.1 kong/3.9.0.1-enterprise-edition
X-Kong-Request-Id: 63814467e3c577789de10b62e60fdb6d
```

Test the API endpoint for status code 403:

```bash
curl -i http://localhost:8000/http-change-code/403
```

**Output:**

```http
HTTP/1.1 403 FORBIDDEN
Content-Type: text/html; charset=utf-8
Content-Length: 0
Connection: keep-alive
Server: gunicorn/19.9.0
Date: Fri, 11 Jul 2025 12:55:26 GMT
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
X-Kong-Upstream-Latency: 178
X-Kong-Proxy-Latency: 0
Via: 1.1 kong/3.9.0.1-enterprise-edition
X-Kong-Request-Id: 4efab6713f944441a5ec71657030f300
```

---

## ➕ Add Post Function Plugin

Add the `post-function` plugin to intercept the response and convert HTTP 403 to 401:

```bash
curl -i -X POST http://localhost:8001/plugin-demos/services/post_function_demo_svc/plugins \
  -H 'Kong-Admin-Token:<admin-token>' \
  --data "name=post-function" \
  --data "config.header_filter=return function() if kong.response.get_status() == 403 then kong.response.set_status(401) end end"
```

**Output:**

```http
HTTP/1.1 201 Created
Date: Fri, 11 Jul 2025 13:23:50 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: http://ec2-52-66-65-107.ap-south-1.compute.amazonaws.com:8002
X-Kong-Admin-Request-ID: b3def33429717246e270137dc7370f4c
vary: Origin
Access-Control-Allow-Credentials: true
Content-Length: 604
X-Kong-Admin-Latency: 18
Server: kong/3.9.0.1-enterprise-edition

{
  "route": null,
  "service": {
    "id": "6ab2205b-99fb-4ddf-bfc3-395f75106343"
  },
  "id": "b016b8ac-d2dc-43fe-8154-31f24fb044d4",
  "consumer": null,
  "ordering": null,
  "consumer_group": null,
  "instance_name": null,
  "created_at": 1752240230,
  "updated_at": 1752240230,
  "config": {
    "ws_handshake": [],
    "access": [],
    "header_filter": [
      "return function() if kong.response.get_status() == 403 then kong.response.set_status(401) end end"
    ],
    "body_filter": [],
    "ws_upstream_frame": [],
    "ws_client_frame": [],
    "ws_close": [],
    "certificate": [],
    "log": [],
    "rewrite": []
  },
  "protocols": [
    "grpc",
    "grpcs",
    "http",
    "https"
  ],
  "enabled": true,
  "name": "post-function",
  "tags": null
}
```

---

## 🧪 Test API After Adding Post Function Plugin

Test for HTTP 200 (should remain unchanged):

```bash
curl -i http://localhost:8000/http-change-code/200
```

**Output:**

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 0
Connection: keep-alive
Server: gunicorn/19.9.0
Date: Fri, 11 Jul 2025 13:26:09 GMT
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
X-Kong-Upstream-Latency: 178
X-Kong-Proxy-Latency: 2
Via: 1.1 kong/3.9.0.1-enterprise-edition
X-Kong-Request-Id: db50971e93041e5a5cb4437d699f874a
```

Test for HTTP 401:

```bash
curl -i http://localhost:8000/http-change-code/401
```

**Output:**

```http
HTTP/1.1 401 UNAUTHORIZED
Content-Length: 0
Connection: keep-alive
Server: gunicorn/19.9.0
Date: Fri, 11 Jul 2025 13:26:12 GMT
WWW-Authenticate: Basic realm="Fake Realm"
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
X-Kong-Upstream-Latency: 180
X-Kong-Proxy-Latency: 1
Via: 1.1 kong/3.9.0.1-enterprise-edition
X-Kong-Request-Id: 9ece0a76ee46003cdba7ab63cc256e73
```

Test for HTTP 403 (should be transformed to 401):

```bash
curl -i http://localhost:8000/http-change-code/403
```

**Output:**

```http
HTTP/1.1 401 Unauthorized
Content-Type: text/html; charset=utf-8
Content-Length: 0
Connection: keep-alive
Server: gunicorn/19.9.0
Date: Fri, 11 Jul 2025 13:26:15 GMT
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
X-Kong-Upstream-Latency: 196
X-Kong-Proxy-Latency: 1
Via: 1.1 kong/3.9.0.1-enterprise-edition
X-Kong-Request-Id: 5f34a67ac9dc0f0aaa0d82720d4a1e7d
```

---

## 📝 Summary

This guide demonstrated how to:

* Create a sample Kong service and route.
* Validate HTTP status codes returned by the upstream service.
* Add Kong's **post-function** plugin with a Lua snippet that intercepts HTTP responses.
* Transform HTTP response status from **403 Forbidden** to **401 Unauthorized** dynamically.
* Confirm the plugin behavior with multiple test requests, ensuring only the 403 responses are modified.


---


