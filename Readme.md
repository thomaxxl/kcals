# ApiLogicServer + Keycloak

## Info

This repo contains a demo for apilogicserver integration with keycloak oidc JWT authentication.

## Run:

`docker-compose up`

This will run 2 containers on the docker host:
- keycloak (http://localhost:8080)
- apilogicserver project (http://localhost:5656)

## Test:

```bash
# keycloak realm named "kcals"
KC_BASE=http://localhost:8080/realms/kcals

# oidc token endpoint
TOKEN_ENDPOINT=$(curl ${KC_BASE}/.well-known/openid-configuration | jq -r .token_endpoint)

# retrieve an access token by logging in 
TOKEN=$(curl ${TOKEN_ENDPOINT} -d 'grant_type=password&client_id=alsclient' -d 'username=demo' -d 'password=demo' | jq -r .access_token)

# test the authentication
curl http://localhost:5656/api/Category -H "Authorization: Bearer ${TOKEN}" | jq .

```

## Implementation

- the `$PWD/projects` was mounted at `/projects` in the ApiLogicServer container
- A project named [`KCALS`](projects/KCALS) was created (default nw, with authentication):

```bash
mkdir projects
chmod 777 projects # we need to be able to write to this directory from the container
docker run  $PWD/projects:/projects -it apilogicserver/api_logic_server bash -c "ApiLogicServer create --project_name=/projects/KCALS --db_url= ; ApiLogicServer add-auth --project_name=/projects/KCALS"
```

For users to be able to authenticate with JWTs signed by keycloak, we have to download the JWK signing key from keycloak and use that to validate the JWTs. 
JWT validation is implemented in [projects/KCALS/security/system/authentication.py](projects/KCALS/security/system/authentication.py). 

By default, apilogicserver authentication uses a user database. Our users are defined in keycloak however. I had to change [auth_provider.py](auth_provider.py) for this to (kinda) work.

## React-Admin

Nginx is used to host the safrs-react-admin frontend at http://localhost/admin-app .
