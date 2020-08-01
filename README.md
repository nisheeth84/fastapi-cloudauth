# FastAPI Cloud Auth


## Features

* [X] Verify access/id token
* [X] Authenticate permission based on scope (or groups) within access token 
* [X] Get login user info (name, email, etc.) within ID token
* [X] Dependency injection for verification/getting user, powered by [FastAPI](https://github.com/tiangolo/fastapi)
* [X] Support for:
    * [X] [AWS Cognito](https://aws.amazon.com/jp/cognito/)
    * [X] [Auth0](https://auth0.com/jp/)
    * [] [Firebase Auth]()

## Example (AWS Cognito)

### Pre-requirement

* Check `region` and `userPoolID` of AWS Cognito that you manage to
* Create a user assigned `read:users` permission in AWS Cognito 
* Get Access/ID token for the created user

NOTE: access token is valid for verification and scope-based authentication. ID token is valid for verification and getting user info.

### Create it

Create a file main.py with:

```
import os
from fastapi import FastAPI, Depends
from fastapi_auth.cognito import Cognito, CognitoCurrentUser, CognitoClaims

app = FastAPI()
auth = Cognito(region=os.environ["REGION"], userPoolId=os.environ["USERPOOLID"])


@app.get("/", dependencies=[Depends(auth.scope("read:users"))])
def secure():
    # access token is valid
    return "Hello"


get_current_user = CognitoCurrentUser(
    region=os.environ["REGION"], userPoolId=os.environ["USERPOOLID"]
)


@app.get("/user/")
def secure_user(current_user: CognitoClaims = Depends(get_current_user)):
    # ID token is valid
    return f"Hello, {current_user.username}"
```

Run the server with:

```console
$ uvicorn main:app

INFO:     Started server process [15332]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
```

### Interactive API Doc

Go to http://127.0.0.1:8000/docs.

You will see the automatic interactive API documentation (provided by Swagger UI).

`Authorize :unlock:` button can be available at the endpoints injected dependency.

You can put token and try endpoint interactively.

![Swagger UI](https://github.com/tokusumi/fastapi-cloudauth/tree/master/docs/src/authorize_in_doc.jpg)

