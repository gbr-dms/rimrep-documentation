# Krakend Rules
This doc explains krakend rules and how to update/add Krakend rules. Krakend rules are available as a simple json configuration.
Krakend json configuration is managed in `rimrep-flux` repo under `/apps` directory. 
Like all other apps, it will have a base yaml/configuration and the environment specific files are maintained under respective environment specific folders.
Update the environment specific `krakend.json` for respective environments.

## Krakend JSON
Example krakend json file skeleton is provided below. 
```json
{
  "version": 3,
  "timeout": "10s",
  "plugin": {
    "pattern": ".so",
    "folder": "./"
  },
  "extra_config": {
    "router": {
      "return_error_msg": true
    }
  },
  "endpoints": [] # Important part of Krakend rules
}
```

We will the attributes definiton here,
- `version` is the version of Krakend json
- `timeout` is the timeout for all request via Krakend
- `plugin` to load custom plugin. Here it is used for loading aws plugin which comes with Krakend image from [rimrep-krakend-plugin](https://github.com/aodn/rimrep-krakend-plugin) to access S3 buckets. DO NOT REMOVE IT OR ELSE BUCKET ACCESS WILL FAIL.
- `extra_config` to capture the error from routes
- `endpoints` holds all URL rules and will see it in detail in the next section.

## Krakend Endpoints Rules
There are 4 types of rules configured and are,
- Apps (pygeoapi/stacfastapi) public urls
- Apps (pygeoapi/stacfastapi) private/protected urls
- S3 public bucket urls
- S3 private bucket urls

### Apps public urls
Pygeoapi and Stac fast api public urls comes under this category and the json configuration looks like below. 
It is applicable only for the url patterns https://pygeoapi.development.reefdata.io/collection.
```json
{
      "endpoint": "/collections", # URL access via browser/curl
      "output_encoding": "no-op", # no-op encoding will act as proxy
      "input_query_strings": [ 
        "*" # allows all query string in request to backend app
      ],
      "input_headers": [
        "*" # allows all request headers to backend app
      ],
      "backend": [
        {
          "url_pattern": "/collections", # backend URL path
          "host": [
            "pygeoapi.pygeoapi.svc.cluster.local:5000" # backend URL host, here it is service endpoint of pygeoapi
          ]
        }
      ]
}
```
Another one example with placeholder. The difference here is `/{1}` added to endpoint and backend. 
The above URL only captures URL like https://pygeoapi.development.reefdata.io/collection but not https://pygeoapi.development.reefdata.io/collection/abs-lgas-2021.
In order to capture such urls placeholder is used. If the url path have three parts, then another placholder `/{2}` needs to be added. 
Likewise new rules to be added when the url path is getting increased.
**NOTE: WILDCARD IS SUPPORTED ONLY IN KRAKEND ENTERPRISE VERSION**
```json
{
      "endpoint": "/collections/{1}", # URL access via browser/curl with placholder
      "output_encoding": "no-op", 
      "input_query_strings": [ 
        "*" 
      ],
      "input_headers": [
        "*" 
      ],
      "backend": [
        {
          "url_pattern": "/collections/{1}", # backend URL path with placeholder passed from frontend
          "host": [
            "pygeoapi.pygeoapi.svc.cluster.local:5000"
          ]
        }
      ]
}
```
### Apps private/protected urls
Pygeoapi and Stac fast api private/protected dataset urls comes under this category and the json configuration looks like below.
The difference between public and private dataset url is the presence of `extra_config.auth/validator` responsible for validating the `Authoriation: Bearer <token>` header.
In the below example, URL https://pygeoapi.development.reefdata.io/collection/noaa-crw-dhw is protected and can be accessed only by the user with `reef-admin` role.
```json
{
      "endpoint": "/collections/noaa-crw-dhw",
      "output_encoding": "no-op",
      "input_query_strings": [
        "*"
      ],
      "input_headers": [
        "*"
      ],
      "extra_config": {
        "auth/validator": {
          "alg": "RS256", 
          "jwk_url": "https://keycloak.development.reefdata.io/realms/rimrep-development/protocol/openid-connect/certs",
          "disable_jwk_security": true,
          "issuer": "https://keycloak.development.reefdata.io/realms/rimrep-development",
          "cache": true,
          "roles_key": "realm_role", # role name key in jwt response
          "roles": [
            "reef-admin" # role in jwt response
          ],
          "operation_debug": true
        }
      },
      "backend": [
        {
          "url_pattern": "/collections/noaa-crw-dhw",
          "host": [
            "pygeoapi.pygeoapi.svc.cluster.local:5000"
          ]
        }
      ]
    }

```


## How to add Krakend rules
Most of the URLs for apps and buckets are already covered with or without placeholder rules so no specific rules needed if a new dataset is added unless if the URL pattern is different from the configured rules.
In such cases, add the new url pattern to endpoints by copying any of the available rules. Add `extra_config.auth/validator` to the rules if it needs to be protected and add `extra_config.plugin/req-resp-modifier` if it is for bucket URL.
