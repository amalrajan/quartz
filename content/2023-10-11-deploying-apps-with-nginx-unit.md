---
title: Deploying apps with NGINX unit
date: 2023-10-11
categories: [Software Engineering]
tags: [deployment]
---

An app server is different from a web server. An app server executes application logic, manages transactions, and communicates with backend databases, while a web server primarily serves static content, handles HTTP requests, and may also manage SSL encryption and caching.

NGINX is a part of my most recent employer, F5. While working at F5, I was an integral part of the team that aimed to package the NGINX Unit app server so that it is easily consumed.


### Configuration

NGINX Unit uses configuration in JSON format. A typical configuration has two main components: the listener and the application section.


```json
{
    "listeners": {
        "*:80": {
            "pass": "applications/flask"
        }
    },

    "applications": {
        "flask": {
            "type": "python 3.Y",
            "path": "/path/to/app/",
            "home": "/path/to/app/venv/",
            "module": "wsgi",
            "callable": "app"
        }
    }
}
```

### Get the current configuration

To see what apps are running on Unit, it is as simple as issuing a GET request:


```bash
curl -X GET --unix-socket /var/run/control.unit.sock localhost/config
```

### Run a new app

To run a new app on Unit, it is again, as simple as sending a POST request. The `config.json` is the new unit configuration with the new application configuration added.

```bash
curl -X PUT --unix-socket /var/run/control.unit.sock --data-binary @config.json localhost/config
```

### Restarting an app

I'm actually impressed with how they manage to redeploy an app with almost zero downtime.

```bash
curl -X GET --unix-socket /var/run/control.unit.sock localhost/control/applications/<application-id>/restart
```


### Thoughts

I wonder why NGINX Unit is not as widely adopted as, say, Tomcat for Java or Gunicorn for Python runtime. It is fast, secure, and modular. You only need to learn one type of configuration for multiple app deployments. If you find this interesting, head over to [https://unit.nginx.org/](https://unit.nginx.org/) to learn more.
