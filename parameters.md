---
title: Parameters
description: Built-in parameter store to keep secrets encrypted and available only during runtime.
---

Ampt's built-in parameter store allows developers to store the parameters in a secure way, and to use them programmatically with the `params` interface of Ampt SDK. All the parameters are encrypted both at transit and at rest and only decrypted during runtime.

!!! note
When you modify a parameter value in the Ampt Dashboard, the changes are instantly applied to all running environments that require the parameter. No restart or anything required to flush the parameters.
!!!

## Parameter Scopes

Some parameters apply to all apps within an organization, while others might be specific to a particular project or application. Additionally, an application will probably use a different value of a parameter in production and development.

**Organization-level parameters** are defined under "Organization Settings" in the Ampt Dashboard. Every environment of every app will inherit those parameters automatically. Organization-level parameters can be overridden at the app and environment levels. For example, you can define an API key to be used for a third-party vendor as `THIRD_PARTY_API_KEY` and you can override this parameter for the production environment.

**App-level parameters** are defined for each application under App Settings. This is helpful when you need to declare a parameter that must be used by all environments of an application. For example, you can define the `STRIPE KEY` for the billing app, and it will be applied solely to this app's environments.

All the parameters defined at the organization and app level are inherited at the environment level and applied to the apps running in those environments. You can't define a new parameter at environment level but you can override the inherited parameters' values. For example, you can override the API keys required for your personal sandbox.

## Reserved Parameters

Ampt reserves some of the keys that are automatically populated by our runtime. The following parameters below are not allowed to be defined on the Ampt Dashboard.

| KEY          | Value                                              |
| ------------ | -------------------------------------------------- |
| ORG_NAME     | The organization the application belongs to        |
| PROJECT_NAME | The name of the project the app is part of         |
| APP_NAME     | The name of the application                        |
| ENV_NAME     | The name of the environment (e.g. prod, dev, etc.) |
| AMPT_URL     | The URL of the application generated by Ampt       |
| AMPT_REGION  | The AWS region the environment is running in       |

## Reading parameters programmatically

Developers can access the parameters injected to the runtime by using the `params` interface.

```javascript
import { params } from "@ampt/sdk";
import { api } from "@ampt/api";

api("my-api")
  .router("/test-data")
  .get("/", async (event) => {
    // reading a custom parameter in an API handler
    console.log(params("KEY"));

    // reading reserved parameters is same as reading the custom parameters
    console.log(params("AMPT_URL"));

    // return all params
    const parameterList = params().list();

    // exported params are available as environment variables
    console.log(process.env.AMPT_URL);
  });
```

## Exporting parameters as environment variables

Developers can export parameters to an app's environment variables when required by certain frameworks or SDKs.

!!! caution
Exporting secrets to environment variables is generally considered a bad security practice. Whenever possible, secure parameters should be accessed directly using the `params` interface.
!!!

```javascript
import { params } from "@ampt/sdk";
// exporting at init time

params().export(); // export all parameters as environment variables
params().export("KEY"); // export specific key to process.env
params().export(["AMPT_URL", "KEY"]); // export list of keys to process.env

api("my-api")
  .router("/test-data")
  .get("/", async (event) => {
    // exported params are available as environment variables
    console.log(process.env.AMPT_URL);
  });
```
