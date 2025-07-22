# Gloo Gateway Reproducer Template

## Installation

Add Gloo Gateway Helm repo:
```
helm repo add glooe https://storage.googleapis.com/gloo-ee-helm
```

Export your Gloo Gateway License Key to an environment variable:
```
export GLOO_GATEWAY_LICENSE_KEY={your license key}
```

Install Gloo Gateway:
```
cd install
./install-gloo-gateway-with-helm.sh
```

> NOTE
> The Gloo Gateway version that will be installed is set in a variable at the top of the `install/install-gloo-gateway-with-helm.sh` installation script.

## Setup the environment

Run the `install/setup.sh` script to setup the environment:

- Create the required namespaces
- Deploy the Gateway
- Deploy the HTTPBin application
- Deploy the Reference Grants
- Deploy the HTTPRoute
- Deploy the JTW RouteOption

```
./setup.sh
```

## Access the HTTPBin application

Create the JWT required to access the application

```
export ALICE_TOKEN=$(./install/create-jwt-with-scopes.sh ./install/private-key.pem alice ops)
```

```
./curl-request-k8s-gw-api.sh
```

or

```
curl -v -H "Authorization: Bearer $ALICE_TOKEN" http://api.example.com:81/get
```


## Requirement

Be able to grant access based on a, possible, partial match of scopes. Say you have the following scopes in your JWT, as per spec https://datatracker.ietf.org/doc/html/rfc6749#section-3.3, space-delimited

[
  {
    "alg": "RS256",
    "typ": "JWT"
  },
  {
    "iss": "solo.io",
    "org": "solo.io",
    "sub": "alice",
    "team": "ops",
    "scope": "app1.resource1:read app1.resource1:update app1.resource2:read app1.resource2:update app2resource1:read app2.resource1:update"
  }
]

And say you want to grant access to a route if:
- the JWT contains "app1.resource1:read AND app1.resource1.update"
- the JWT contains "app1.resource1:read OR app1.rsource1.update"


And for another route you want to grant access if:
- the JWT contains app2.resource.read