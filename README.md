[![FIWARE Banner](https://fiware.github.io/tutorials.Subscriptions/img/fiware.png)](https://www.fiware.org/developers)
[![NGSI LD](https://img.shields.io/badge/NGSI-LD-d6604d.svg)](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.03.01_60/gs_cim009v010301p.pdf)

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Subscriptions.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)
[![JSON LD](https://img.shields.io/badge/JSON--LD-1.1-f06f38.svg)](https://w3c.github.io/json-ld-syntax/) <br/>
[![Documentation](https://img.shields.io/readthedocs/ngsi-ld-tutorials.svg)](https://ngsi-ld-tutorials.rtfd.io)

This tutorial teaches NGSI-LD users about how to create and manage context data subscriptions. The tutorial builds on the
entities and [Smart Farm](https://github.com/FIWARE/tutorials.Getting-Started/tree/NGSI-LD)
application created in the previous examples to enable users to
understand the [NGSI-LD](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.03.01_60/gs_cim009v010301p.pdf) Subscribe/Notify paradigm and how to
use NGSI subscriptions within their own code.

The tutorial refers to devices and actions made within the browser combined with [cUrl](https://ec.haxx.se/)
commands. The cUrl commands are also available as
[Postman documentation](https://fiware.github.io/tutorials.Subscriptions/).

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/18ea15017244c70d1fe4)

## Contents

<details>
<summary><strong>Details</strong></summary>

- [Subscribing to Changes of State](#subscribing-to-changes-of-state)
  * [Entities within a smart Agrifood system](#entities-within-a-smart-agrifood-system)
  * [Farm Management Information System frontend](#farm-management-information-system-frontend)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
  * [Docker](#docker)
  * [Cygwin](#cygwin)
- [Start Up](#start-up)
- [Using Subscriptions](#using-subscriptions)
  * [Setting up a simple Subscription](#setting-up-a-simple-subscription)
- [Subscription CRUD Actions](#subscription-crud-actions)
    + [Creating a Subscription](#creating-a-subscription)
    + [Delete a Subscription](#delete-a-subscription)
    + [Update an Existing Subscription](#update-an-existing-subscription)
    + [List all Subscriptions](#list-all-subscriptions)
    + [Read the detail of a Subscription](#read-the-detail-of-a-subscription)
-   [Next Steps](#next-steps)

</details>

# Subscribing to Changes of State

> 'Another sandwich!' said the King.
>
> 'There's nothing but hay left now,' the Messenger said, peeping into the bag.
>
> 'Hay, then,' the King murmured in a faint whisper.
>
> Alice was glad to see that it revived him a good deal. 'There's nothing like eating hay when you're faint,' he remarked to her, as he munched away.
>
> — Lewis Carroll (Through the Looking-Glass and What Alice Found There)

Within the FIWARE platform, an entity represents the state of a physical or conceptual object which exists in the real
world. Every smart solution needs to know the current state of these object at any given moment in time.

The context of each of these entities is constantly changing. For example, within the smart farm example, the
context will change as animals and vehicles move, soil dries out, tasks are allocated on the farm and completed and so on. For a smart solution based on IoT
sensor data, this issue is even more pressing as the system will constantly be reacting to changes in the real world.

Until now all the operations we have used to change the state of the system have been **synchronous** - changes have
been made by directly by a user or application and they have been informed of the result. The Orion Context Broker
offers also an **asynchronous** notification mechanism - applications can subscribe to changes of context information so
that they can be informed when something happens. This means the application does not need to continuously poll or
repeat query requests.

Use of the subscription mechanism will therefore reduce both the volume of requests and amount of data being passed
between components within the system. This reduction in network traffic will improve the overall responsiveness.

## Entities within a smart Agrifood system

The relationship between our entities is defined as shown:

![](https://fiware.github.io/tutorials.Subscriptions/img/ngsi-ld-entities.png)

## Farm Management Information System frontend

In a previous tutorial, a simple Node.js Express application
was created. This tutorial will use the monitor page to watch the status of recent requests, and the devices page to alter the machines on the farm. Once the services are running these pages can be accessed from the following URLs:

#### Event Monitor

The event monitor can be found at: `http://localhost:3000/app/monitor`

![FIWARE Monitor](https://fiware.github.io/tutorials.Subscriptions/img/monitor.png)

#### Device Monitor

For the purpose of this tutorial, a series of dummy agricultural IoT devices have been created, which will be attached
to the context broker. Details of the architecture and protocol used can be found in the
[IoT Sensors tutorial](https://github.com/FIWARE/tutorials.IoT-Sensors/tree/NGSI-LD) The state of each device can be
seen on the UltraLight device monitor web page found at: `http://localhost:3000/device/monitor`

![FIWARE Monitor](https://fiware.github.io/tutorials.Subscriptions/img/farm-devices.png)

# Architecture

This application will make use of two FIWARE components - the
[Orion-LD Context Broker](https://fiware-orion.readthedocs.io/en/latest/)and the
[IoT Agent for UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/). Usage of any NGSI-LD Context Broker is sufficient
for an application to qualify as _“Powered by FIWARE”_.

Currently, the Orion-LD Context Broker relies on open source [MongoDB](https://www.mongodb.com/) technology to keep
persistence of the context data it holds. To request context data from external sources, a simple **Context Provider
NGSI proxy** has also been added. To visualize and interact with the Context we will add a simple Express **Frontend**
application

Therefore, the architecture will consist of four elements:

-   The [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) which will receive requests using
    [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/gitlab/NGSI-LD/NGSI-LD/raw/master/spec/updated/full_api.json)
-   The FIWARE [IoT Agent for UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/) which will receive
    southbound requests using
    [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/gitlab/NGSI-LD/NGSI-LD/raw/master/spec/updated/full_api.json)
    and convert them to
    [UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
    commands for the devices
-   The underlying [MongoDB](https://www.mongodb.com/) database:
    -   Used by the Orion Context Broker to hold context data information such as data entities, subscriptions and
        registrations
-   The **Tutorial Application** does the following:
    -   Offers static `@context` files defining the context entities within the system.
    -   Acts as set of dummy [agricultural IoT devices](https://github.com/FIWARE/tutorials.IoT-Sensors/tree/NGSI-LD)
        using the
        [UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
        protocol running over HTTP.

Since all interactions between the elements are initiated by HTTP requests, the entities can be containerized and run
from exposed ports.

![](https://fiware.github.io/tutorials.Subscriptions/img/architecture-ld.png)

The necessary configuration information can be seen in the services section of the associated `docker-compose.yml` file.
It has been described in a previous tutorial

# Prerequisites

## Docker

To keep things simple both components will be run using [Docker](https://www.docker.com). **Docker** is a container
technology which allows to different components isolated into their respective environments.

-   To install Docker on Windows follow the instructions [here](https://docs.docker.com/docker-for-windows/)
-   To install Docker on Mac follow the instructions [here](https://docs.docker.com/docker-for-mac/)
-   To install Docker on Linux follow the instructions [here](https://docs.docker.com/install/)

**Docker Compose** is a tool for defining and running multi-container Docker applications. A
[YAML file](https://raw.githubusercontent.com/Fiware/tutorials.Entity-Relationships/master/docker-compose.yml) is used
configure the required services for the application. This means all container services can be brought up in a single
command. Docker Compose is installed by default as part of Docker for Windows and Docker for Mac, however Linux users
will need to follow the instructions found [here](https://docs.docker.com/compose/install/)

You can check your current **Docker** and **Docker Compose** versions using the following commands:

```console
docker-compose -v
docker version
```

Please ensure that you are using Docker version 18.03 or higher and Docker Compose 1.21 or higher and upgrade if
necessary.

## Cygwin

We will start up our services using a simple bash script. Windows users should download [cygwin](http://www.cygwin.com/)
to provide a command-line functionality similar to a Linux distribution on Windows.

# Start Up

All services can be initialized from the command-line by running the bash script provided within the repository. Please
clone the repository and create the necessary images by running the commands as shown:

```console
git clone https://github.com/FIWARE/tutorials.Subscriptions.git
cd tutorials.Subscriptions
git checkout NGSI-LD

./services create;
./services orion;
```

This command will also import seed data from the previous
Farm Management Information System example on startup, and provision a series of dummy devices on the farm.

> :information_source: **Note:** If you want to clean up and start over again you can do so with the following command:
>
> ```console
> ./services stop
> ```

# Using Subscriptions

To follow the tutorial correctly please ensure you have the follow two pages available on separate tabs in your browser before you
enter any cUrl commands.

#### FMIS System

Details of various buildings around the farm can be found in the tutorial application. Open `http://localhost:3000/app/farm/urn:ngsi-ld:Building:farm001` to display a building with an associated filling sensor and thermostat.

#### Event Monitor

The event monitor can be found at: `http://localhost:3000/app/monitor`.

## Setting up a simple Subscription

Within the Farm Management Information System, imagine that the farmer wants a contractor to refill his barn with hay when the level has reduced below a set level. It would be possible
to set up the system so that the contractor was constantly polling for new information, however hay is not removed very
frequently so this would be a waste of resources and create a lot of unnecessary data traffic.

The alternative is to create a subscription which will POST a payload to a "well-known" URL whenever a value has
changed. A new subscription can be added by making a POST request to the `/ngsi-ld/v1/subscriptions/` endpoint as shown below:

#### :one: Request:

```console
curl -L -X POST 'http://localhost:1026/ngsi-ld/v1/subscriptions/' \
-H 'Content-Type: application/ld+json' \
-H 'NGSILD-Tenant: openiot' \
--data-raw '{
  "description": "Notify me of low feedstock on Farm:001",
  "type": "Subscription",
  "entities": [{"type": "FillingSensor"}],
  "watchedAttributes": ["filling"],
  "q": "filling>0.6;filling<0.8;controllingAsset==urn:ngsi-ld:Building:farm001",
  "notification": {
    "attributes": ["filling", "controllingAsset"],
    "format": "keyValues",
    "endpoint": {
      "uri": "http://tutorial:3000/subscription/low-stock-farm001",
      "accept": "application/json"
    }
  },
   "@context": "http://context-provider:3000/data-models/ngsi-context.jsonld"
}'
```

The body of the POST request consists of two parts, the first section of the request (consisting of `entities`, `type`, `watchedAttributes` and
`q`)states that the subscription will be checked whenever the `filling`  attribute of  a **FillingSensor** entity is altered. This is further refined by the `q` parameter so that the actual subscription is only fired for any **FillingSensor** entity linked to the **Building** `urn:ngsi-ld:Building:farm001`  and only when the `filling` attribute drops below 0.8

The notification section of the body states that once the conditions of the subscription have been met, a POST
request containing all affected **FillingSensor** entities will be sent to the URL
`http://tutorial:3000/subscription/low-stock-farm001` which is handled by the contractor's own system.

It should be noted that the subscription is using the `NGSILD-Tenant` header because the IoT Devices have been provisioned using a separate tenant to the buildings for now. Tenants allow for context data to be distributed across separate databases and allow multiple application clients to access the same context broker but keep their own data sets apart.

Go to the Device Monitor `http://localhost:3000/app/farm/urn:ngsi-ld:Building:farm001` and start removing hay from the barn. Nothing happens until the barn is half-empty, then a request is sent to
`subscription/low-stock-farm001` as shown:

#### `http://localhost:3000/app/monitor`

![](https://fiware.github.io/tutorials.Subscriptions/img/products-subscription.png)

#### Subscription Payload:

```json
{
 "id": "urn:ngsi-ld:Notification:5fd0f3824eb81930c97005d8",
 "type": "Notification",
 "subscriptionId": "urn:ngsi-ld:Subscription:5fd0ee554eb81930c97005c1",
 "notifiedAt": "2020-12-09T15:55:46.520Z",
 "data": [
  {
   "id": "urn:ngsi-ld:Device:filling001",
   "type": "FillingSensor",
   "controllingAsset": "urn:ngsi-ld:Building:farm001",
   "filling": 0.59
  }
 ]
}
```

Code within the Farm Management Information System handles received the POST request as shown:

```javascript
const NOTIFY_ATTRIBUTES = ['controllingAsset', 'type', 'filling', 'humidity', 'temperature'];

router.post('/subscription/:type', (req, res) => {
    monitor('notify', req.params.type + ' received', req.body);
    _.forEach(req.body.data, (item) => {
        broadcastEvents(req, item, NOTIFY_ATTRIBUTES);
    });
    res.status(204).send();
});

function broadcastEvents(req, item, types) {
    const message = req.params.type + " received";
    _.forEach(types, (type) => {
        if (item[type]) {
            req.app.get("io").emit(item[type], message);
        }
    });
}
```

This business logic emits socket I/O events to any registered parties (such as the contractor who will then refill the barn.)



#### :two: Request:

This second subsscription will fire when the `filling` level is between 0.6 and 0.4. The `format` attribute has been altered to inform the subscriber using NGSI-LD normalized format.

```console
curl -L -X POST 'http://localhost:1026/ngsi-ld/v1/subscriptions/' \
-H 'Content-Type: application/ld+json' \
-H 'NGSILD-Tenant: openiot' \
--data-raw '{
  "description": "Notify me of low feedstock on Farm:001",
  "type": "Subscription",
  "entities": [{"type": "FillingSensor"}],
  "watchedAttributes": ["filling"],
  "q": "filling>0.4;filling<0.6;controllingAsset==urn:ngsi-ld:Building:farm001",
  "notification": {
    "attributes": ["filling", "controllingAsset"],
    "format": "normalized",
    "endpoint": {
      "uri": "http://tutorial:3000/subscription/low-stock-farm001-ngsild",
      "accept": "application/json"
    }
  },
   "@context": "http://context-provider:3000/data-models/ngsi-context.jsonld"
}'
```


#### Subscription Payload:

When a `low-stock-farm001-ngsild` event is fired, the response is as shown:

```json
{
 "id": "urn:ngsi-ld:Notification:5fd0fa684eb81930c97005f3",
 "type": "Notification",
 "subscriptionId": "urn:ngsi-ld:Subscription:5fd0f69b4eb81930c97005db",
 "notifiedAt": "2020-12-09T16:25:12.193Z",
 "data": [
  {
   "id": "urn:ngsi-ld:Device:filling001",
   "type": "FillingSensor",
   "filling": {
    "type": "Property",
    "value": 0.25,
    "unitCode": "C62",
    "observedAt": "2020-12-09T16:25:12.000Z"
   },
   "controllingAsset": {
    "type": "Relationship",
    "object": "urn:ngsi-ld:Building:farm001",
    "observedAt": "2020-12-09T16:25:12.000Z"
   }
  }
 ]
}
```

Because the `accept` attribute has been set to `application/json`, the `@context` is sent as a `Link` header rather than an attribute within the payload body.

#### :three: Request:

Context brokers may offer additional custom payload formats (typically prefixed with an `x-`). The Orion-LD broker offers a backwards compatible **NGSI-v2** payload option for legacy systems.

This third subsscription will fire when the `filling` level is below 0.4. The `format` attribute has been altered to inform the subscriber using NGSI-v2 normalized format.


```console
curl -L -X POST 'http://localhost:1026/ngsi-ld/v1/subscriptions/' \
-H 'Content-Type: application/ld+json' \
-H 'NGSILD-Tenant: openiot' \
--data-raw '{
  "description": "Notify me of low feedstock on Farm:001",
  "type": "Subscription",
  "entities": [{"type": "FillingSensor"}],
  "watchedAttributes": ["filling"],
  "q": "filling<0.4;controllingAsset==urn:ngsi-ld:Building:farm001",
  "notification": {
    "attributes": ["filling", "controllingAsset"],
    "format": "x-ngsiv2-normalized",
    "endpoint": {
      "uri": "http://tutorial:3000/subscription/low-stock-farm001-ngsiv2",
      "accept": "application/json"
    }
  },
   "@context": "http://context-provider:3000/data-models/ngsi-context.jsonld"
}'
```

#### Subscription Payload:

When a `low-stock-farm001-ngsiv2` event is fired, the response is a normalzed NGSI-v2 payload as shown:

```json
{
 "subscriptionId": "urn:ngsi-ld:Subscription:5fd1f31e8b9b83697b855a5d",
 "data": [
  {
   "id": "urn:ngsi-ld:Device:filling001",
   "type": "https://uri.etsi.org/ngsi-ld/default-context/FillingSensor",
   "https://w3id.org/saref#fillingLevel": {
    "type": "Property",
    "value": 0.33,
    "metadata": {
     "unitCode":  "C62",
     "accuracy": {
      "type": "Property",
      "value": 0.05
     },
     "observedAt": "2020-12-10T10:11:57.000Z"
    }
   },
   "https://uri.etsi.org/ngsi-ld/default-context/controllingAsset": {
    "type": "Relationship",
    "value": "urn:ngsi-ld:Building:farm001",
    "metadata": {
     "observedAt": "2020-12-10T10:11:57.000Z"
    }
   }
  }
 ]
}
```

As can be seen, by default the attributes are returned using URN long names. It is also possible to request that the Orion-LD context broker pre-applies a compaction operation to the payload.

-  `x-nsgiv2-keyValues` - Key Value pairs with URN attribute names
-  `x-nsgiv2-keyValues-compacted` - Key Value pairs with short name attribute aliases
-  `x-ngsiv2-normalized` - NGSI-v2 normalized payload with URN attribute names
-  `x-ngsiv2-normalized-compacted`- NGSI-v2 normalized payload pairs with short name attribute aliases

The set of available custom formats will vary between Context Brokers.


# Subscription CRUD Actions

The **CRUD** operations for subscriptions map on to the expected HTTP verbs under the `/ngsi-ld/v1/subscriptions/` endpoint.

-   **Create** - HTTP POST
-   **Read** - HTTP GET
-   **Update** - HTTP PATCH
-   **Delete** - HTTP DELETE

The `<subscription-id>` is auto generated when the subscription is created and returned in Header of the POST response
to be used by the other operation thereafter.

### Creating a Subscription

This example creates a new subscription. The subscription will fire an asynchronous notification to a URL whenever the
context is changed and the conditions of the subscription - Any Changes to Product prices - are met.

New subscriptions can be added by making a POST request to the `/ngsi-ld/v1/subscriptions/` endpoint.

The subject section of the request states that the subscription will be fired whenever the price attribute of any
Product entity is altered.

The notification section of the body states that a POST request containing all affected entities will be sent to the
`http://tutorial:3000/subscription/price-change` endpoint.

#### :four: Request:

```console
curl -L -X POST 'http://localhost:1026/ngsi-ld/v1/subscriptions/' \
-H 'Content-Type: application/ld+json' \
--data-raw '{
  "description": "Notify me of all product price changes",
  "type": "Subscription",
  "entities": [{"type": "Product"}],
  "watchedAttributes": ["price"],
  "notification": {
    "format": "keyValues",
    "endpoint": {
      "uri": "http://tutorial:3000/subscription/price-change",
      "accept": "application/json"
    }
  },
   "@context": "http://context-provider:3000/data-models/ngsi-context.jsonld"
}'
```

### Delete a Subscription

This example deletes the Subscription with `id=urn:ngsi-ld:Subscription:5fd228838b9b83697b855a72` from the context.

Subscriptions can be deleted by making a DELETE request to the `/ngsi-ld/v1/subscriptions/<subscription-id>` endpoint.

#### :five: Request:

```console
curl -X DELETE \
  --url 'http://localhost:1026/ngsi-ld/v1/subscriptions/urn:ngsi-ld:Subscription:5fd228838b9b83697b855a72'
```

### Update an Existing Subscription

This example amends an existing subscription with the ID `urn:ngsi-ld:Subscription:5fd228838b9b83697b855a72` and updates the notification URL.

Subscriptions can be updated making a PATCH request to the `/ngsi-ld/v1/subscriptions/<subscription-id>` endpoint.

#### :six: Request:

```console
curl -iX PATCH \
  --url 'http://localhost:1026/ngsi-ld/v1/subscriptions/urn:ngsi-ld:Subscription:5fd228838b9b83697b855a72' \
  --header 'content-type: application/json' \
  --data '{
  "notification": {
    "format": "normalized",
    "endpoint": {
      "uri": "http://tutorial:3000/subscription/price-change",
      "accept": "application/json"
    }
  }
}'
```

### List all Subscriptions

This example lists all subscriptions by making a GET request to the `/ngsi-ld/v1/subscriptions/` endpoint.

The notification section of each subscription will also include the last time the conditions of the subscription were
met, and whether associated the POST action was successful.

#### :seven: Request:

```console
curl -X GET \
  --url 'http://localhost:1026/ngsi-ld/v1/subscriptions/
```

### Read the detail of a Subscription

This example obtains the full details of a subscription with a given ID.

The response includes additional details in the notification section showing the last time the conditions of the
subscription were met, and whether associated the POST action was successful.

Subscription details can be read by making a GET request to the `/ngsi-ld/v1/subscriptions/<subscription-id>` endpoint.

#### :eight: Request:

```console
curl -X GET \
  --url 'http://localhost:1026/ngsi-ld/v1/subscriptions/5aead3361587e1918de90aba'
```

# Next Steps

Want to learn how to add more complexity to your application by adding advanced features? You can find out by reading
the other [tutorials in this series](https://fiware-tutorials.rtfd.io)

---

## License

[MIT](LICENSE) © 2020 FIWARE Foundation e.V.
