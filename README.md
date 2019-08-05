# SAP Leonardo IoT SDK for Node.js

**Table of Contents**
- [Description](#description)
- [Requirements](#requirements)
- [Feature Overview](#feature-overview)
- [Download and Installation](#download-and-installation)
- [Samples](#samples)
- [Authorization Concept](#authorization-concept)
- [FAQ](#faq)
- [Known Issues](#known-issues)
- [License](#license)

## Description
The SAP Leonardo IoT SDK for Node.js implements the consumer side usage of the most frequent used APIs of SAP Leonardo IoT. 
Also rarely used APIs which are not covered by designated service class functions can be called from this library by using a generic request facade.
All API calls will be automatically enriched with authorization information to ensure a simple consumption SAP Leonardo IoT services.

To get started using the SAP Leonardo IoT SDK check out the [Download and Installation](#download-and-installation) getting started guide.

## Requirements
It is required to have access to an SAP Leonardo IoT account to make use of this SDK.

## Feature Overview

### Models
|                   | CREATE | READ | UPDATE | DELETE |
|-------------------|--------|------|--------|--------|
| Package           | X      | X    |        | X      |
| Property Set Type | X      | X    |        | X      |
| Thing Type        | X      | X    |        | X      |
| Thing             | X      | X    |        | X      |
| Object Group      | X      | X    |        | X      |

### Time Series Data
|                        | CREATE | READ | DELETE |
|------------------------|--------|------|--------|
| Time Series Store      | X      | X    | X      |
| Time Series Cold Store | X      | X    | X      |

## Download and Installation
This getting started guide will help you to make first usage of the SAP Leonardo IoT SDK within your existing Node.js project.

### Prerequisites
Before starting with the SDK installation, please make sure that following prerequisites are fulfilled to ensure a successful flow through this getting started guide:

- Install your preferred editor or IDE.
- Basic JavaScript and NPM knowledge
- Local NPM installation
- SAP Leonardo IoT account (service key access)

### 1) Install the SAP Leonardo IoT SDK
If you do not already have a NPM project including a `package.json` file, let's create a new one by creating a new empty folder and running the following command within your prefered command line tool
```
$ npm init
```

Next you have to set the SAP registry as source for @sap scoped modules within your project
```
$ npm config set @sap:registry https://npm.sap.com
```

Afterwards the SAP Leonardo IoT SDK can be installed and added to your new application's dependencies
```
$ npm install SAP/leonardo-iot-sdk-nodejs --save
```

### 2) Setup authorization
Each request to SAP Leonardo IoT services requires an authorization token provided by an UAA instance to ensure authorization and authentication. As the recommended authorization configuration relates to the runtime platform, we have to distinguish in this step between cloud foundry and local applications:

##### Local application
For local setup copy the file `default-env-template.json` from this github repository into your project root directory. Afterwards copy the full content of your Leonardo IoT service key information from your subaccount's space ([service key creation documentation](https://help.sap.com/viewer/195126f4601945cba0886cbbcbf3d364/latest/en-US/a41c28db0cf449059d48c23fa5f7b24b.html)) and paste it into the placeholder part of the template file. Now rename this template file to `default-env.json` so it fulfills the naming convention.

##### Cloud Foundry application
Add the Leonardo IoT service binding into the `manifest.yml` file which is used for application deployment:
```
   services:
    - <LeonardoIoTServiceName>
``` 

### 3) Create SAP Leonardo IoT Client
Next please define a .js file which acts as application entry point (referenced by package.json). This file is named `index.js` in case you are using the default NPM project settings. Now let's create a SAP Leonardo IoT client which will support you in accessing Leonardo IoT services within your code:
```js
const LeonardoIoT = require('@sap/leonardo-iot-sdk');
const client = new LeonardoIoT();
```

### 4) Usage for service interaction
The client is now able to communicate with SAP Leonardo IoT services as it is fetching access credentials from the authorization setup file `default-env.json` or from the Leonardo IoT service binding (step 2) dependent to your choosen runtime. There is no more configuration required. Now you are able to perform your first service interaction using the Leonardo IoT client. Here is a runable sample which can be copy & pasted into your index.js file:
```js
const LeonardoIoT = require('@sap/leonardo-iot-sdk');
const client = new LeonardoIoT();

async function main(){
	let things = await client.getThings();
	
	if (things.value.length < 1) {
		console.log("No thing existing");
	}
	
	// Log all thing IDs and names
	for (const [index, thing] of things.value.entries()) {
		console.log("#" + (index + 1) + "\nID: " + thing._id + "\nName: " + thing._name + "\n");
	}
}

main();
```

### 5) Run your code
After taking care about the project setup, client installation and implementation of the application entry point, you are ready to run your code:
```console
$ node index.js
```

Congratulation! You just performed your first API call to a SAP Leonardo IoT service via the SDK for Node.js. 

Here are some more examples of different API calls via Leonardo IoT client:
```js
// Read existing things
let things = await client.getThings();

// Create thing
await client.createThing(thingPayload);

// Read thing snapshot data
let snapshot = await client.getThingSnapshot(thingId);

// Delete thing
await client.deleteThing(thingId);
```

**HINT: All offered functions are documented with JSDoc and give you more information about parameters and API usage. Also each function referes to the API related SAP Help documentation containing full API specification and payload examples**

As some of the APIs also provide more functionality like ordering, selecting designated fields or top / skip functionality, there is always the option to enhance function calls by handover of custom query parameters:
```js
const things = await client.getThings(
    { 
      '$select': '_id,_name',
      '$orderby': '_id',
      '$top': 10
    },
    { 
      'resolveWithFullResponse': false,
      'headers' : {
        'Accept-Language' : 'en-US'
      }
    }
);
```

In case you want to forward access tokens from incoming request of your users (i.e. express app) you are also able to use these tokens for accessing SAP Leonardo IoT services:
```js
const things = await client.getThings(null, 
    { 
      'jwt': <token>
    }
);
```

## Samples
We created a a few quick start samples demonstrating the SAP Leonardo IoT JavaScript SDK functionality in a easy and simple adoptable way. Please have a look into the [***sample***](./sample) subdirectory for further information and coding samples.

## Authorization Concept
Each request to SAP Leonardo IoT services requires an authorization token provided by an UAA instance to ensure authorization and authentication.
For more details please have a look in to the [SAP Help documentation](https://help.sap.com/viewer/65de2977205c403bbc107264b8eccf4b/Cloud/en-US/51ec15a8979e497fbcaadf80da9b63ba.html) or this [blog post](https://blogs.sap.com/2018/08/31/how-to-get-an-access-token-from-the-xsuaa-service-for-external-api-accesses-using-the-password-grant-with-client-and-user-credentials-method).

As this SDK is taking care of authorization handling, it is required to provide tenant specific configuration (service key) to ensure a successful client credential flow. This configuration is read from the runtime environment by using the `VCAP_SERVICES` variable when deployed to Cloud Foundry, and using the *default-env.json* file when running locally. This means that you do not have to adapt any coding and could deploy the same code base to different spaces for different subaccount.

**HINT**: Please handle this information very carefully and conscientious. Never publish it into any repository or file server and do not share it without any permission.

### Cloud Foundry environment

**Option 1: Leonardo IoT service binding**
A very flexible and secure way of providing credentials is by fetching the tenant configuration from a service broker service binding of your application. This is the default option when no explicit service name is provided in SAP Leonardo IoT client instantiation.
To make use of this option add the service broker service binding into the `manifest.yml` file, which is used for application deployment:
```
   services:
    - <LeonardoIoTServiceName>
```

Next you can directly create a SAP Leonardo IoT client within your coding without providing any other information:
```js
const LeonardoIoT = require('@sap/leonardo-iot-sdk');
const client = new LeonardoIoT();
```



**Option 2: User provided service binding**
Especially in the case that you want to run an application which is handling data of multiple SAP Leonardo IoT tenants (i.e. migrate data from your production subaccount to your development subaccount) you will make use of user provided service configurations. Here you have the possibility to create a SAP Leonardo IoT client which is fetching its configuration from a manual provided service.

So first you have to bind the user provided services to your application within the related `manifest.yml` file:
```
   services:
    - leonardo-iot-prod-account
    - leonardo-iot-dev-account
```

Next you can create instances of the SAP Leonardo IoT client within your coding by providing the service name in the instantiation:
```js
const LeonardoIoT = require('@sap/leonardo-iot-sdk');
const productionClient = new LeonardoIoT('leonardo-iot-prod-account');
const developmentClient = new LeonardoIoT('leonardo-iot-dev-account');
```

Be aware that the instantiation will fail in case the named service is not provided in your environment. The user provided service itself has to contain all tenant and landscape related information, best practice is to copy & paste the service key of your subaccount, and can be freely named to whatever fits your needs.

### Local environment

**Provide environment in default-env.json file**
The same mechanisms as descriped in the Cloud Foundry environment also are taking place for local running applications. The SAP Leonardo IoT SDK requires some credential and endpoint configuration, as it can be used for different tenants in different environments on different data centers. All these information is part of a Leonardo IoT service key which can be generated by a subaccount admin. The content of this service key has to be copied into a `default-env.json` file, which has to be placed on the projects root folder, so the SDK can also run on a local setup. If this file already exists as you may also include the SAP approuter into your project, feel free to just expand the existing file.

This example shows how the `default-env.json` could look like for using the Leonardo IoT service binding:
```
{
  "VCAP_SERVICES": {
    "iotae": [
      {
        "name": <ANY NAME>,
        "tags": [
          "leonardoiot"
        ],
        "credentials": <PASTE LEONARDO IOT SERVICE KEY HERE>
      }
    ]
  }
}
```

The following example shows a setup using user provided services:
This example shows how the `default-env.json` could look like for using the Leonardo IoT service binding:
```
{
  "VCAP_SERVICES": {
    "user-provided": [
      {
        "name": <ANY NAME, USE THIS NAME IN CLINET INSTANTIATION>,
        "credentials": <PASTE LEONARDO IOT SERVICE KEY HERE>
      }
    ]
  }
}
```

The SAP Leonardo IoT service binding is identified by the *leonardoiot* tag, so make sure that this service contains this tag in local setup. As user provided services are identified by their name, there is no need for any tag information.

## FAQ

### How can I call a service which is not covered by this SDK?
The SAP Leonardo IoT client offers a general `request` function which is also used by the SDK internally. This function gives you full options to access SAP Leonardo IoT services without caring about authorization (same authorization concept as for all other calls used):

```js
const LeonardoIoT = require('@sap/leonardo-iot-sdk');
const client = new LeonardoIoT();

const url = 'https://tm-data-mapping.cfapps.eu10.hana.ondemand.com/v1/assignments';
const assignments = await client.request({url});
```

There are also multiple ways how to build your custom URL, feel free to choose your prefered one:
```js
// Custom URL with navigator support, most recommended but not available for all services
let url = client.navigator.tmDataMapping() + '/v1/assignments';

// Custom URL with navigator destination offering
let url = client.navigator.getDestination('tm-data-mapping') + '/v1/assignments';

// Custom URL without navigator, least recommended
let url = 'https://tm-data-mapping.cfapps.eu10.hana.ondemand.com/v1/assignments';
```

### How can I add query options to my service call?
Many services offer the option to provide query options i.e. filter, order or top / skip parameters to modify the expected result set. Mostly all parameters are optional, but in case you want to make use of it call the function like shown here:

```js
// read things without query parameters
const things = await client.getThings();

// read things with query parameters
const things = await client.getThings({ '$select': '_id,_name', '$orderby': '_id', '$top': 10 });
```

### How can I make use of user tokens for SAP Leonardo IoT service calls?
Every service call offers the option to forward authorization credentials by adding it into the request config parameters of the function call. So in case you handle an incoming user request for fetching data from SAP Leonardo IoT, you can forward the token like this:
```js
// Forward token from incoming request
const things = await client.getThings(null, {'jwt': request.headers.authorization});

// Forward self generated token
const myToken = getToken();
const things = await client.getThings(null, {'jwt': myToken});
```

### How can I use this SDK for different tenants within a single application?
You can create a SAP Leonardo IoT client for a specific tenant by providing the name of the user provided service, which contains all tenant related configurations (tenant service key). In case you are testing locally, don't forget to add the user provided service in the `default-env.json` file:
```js
const LeonardoIoT = require('@sap/leonardo-iot-sdk');

// Client using user provided service configuration with name 'dev-tenant'
const clientDevTenant = new LeonardoIoT('dev-tenant');

// Client using user provided service configuration with name 'test-tenant'
const clientTestTenant = new LeonardoIoT('test-tenant');
```

## Known Issues
Please make use of the well known Github Issues functionality to check existing and report new issues related to the SAP Leonardo IoT SDK.

## License
Copyright (c) 2019 SAP SE or an SAP affiliate company. All rights reserved.

This file is licensed under the Apache Software License, v. 2 except as noted otherwise in the [LICENSE file](LICENSE).