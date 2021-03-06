<properties
	pageTitle="IoT Hub device management twin queries | Microsoft Azure"
	description="Azure IoT Hub for device management tutorial describing how to use queries to find device twins."
	services="iot-hub"
	documentationCenter=".net"
	authors="juanjperez"
	manager="timlt"
	editor=""/>

<tags
 ms.service="iot-hub"
 ms.devlang="dotnet"
 ms.topic="article"
 ms.tgt_pltfrm="na"
 ms.workload="na"
 ms.date="04/29/2016"
 ms.author="juanpere"/>

# Tutorial: How to find device twins using queries with node.js (preview)

[AZURE.INCLUDE [iot-hub-device-management-query-selector](../../includes/iot-hub-device-management-query-selector.md)]

Azure IoT device management enables you to find device twins, the service representation of a physical device, with queries. You can query based on device properties, service properties or tags in the device twin. You can query using tags and properties:

-   To query for device twins using tags, you pass an array of strings and the query returns the set of devices which are tagged with all of those strings.

-   To query for device twins using service properties or device properties, you use a JSON query expression.

For more information on the device twin and queries, please refer to [Overview of Azure IoT Hub device management][lnk-dm-overview].

## Running the device query sample

The following sample extends the [Get started with Azure IoT Hub device management][lnk-get-started] tutorial functionality. Starting from having the different simulated devices running, it will use a query to find specific devices.

### Prerequisites 

Before running this sample, you must have completed the steps in [Get started with Azure IoT Hub device management][lnk-get-started]. That means your simulated devices must be running. If you completed the process before, please restart your simulated devices now.

### Starting the sample

To start the sample, you need to run `registry_queryDevices.js`. This will execute a few different queries. Follow the steps below to start the sample:

1.  Let the simulated devices run for at least 1 minute. This ensures that the device properties in the twin are synchronized with the physical device. Please see [Tutorial: How to use the device twin][lnk-twin-tutorial] for more details on the synchronization.

2.  From the root folder where you cloned the **azure-iot-sdks** repository, navigate to the **azure-iot-sdks/node/service/samples** directory.  

3.  Open **registry_queryDevices.js** and replace the placeholder with your IoT Hub connection string.

4.  Run `node registry_queryDevices.js`.

You should see output in the command line window showing the result of queries for device objects using tags, service properties and device properties.

## Query structure (JSON)

Queries on device properties and service properties are executed with a JSON string to represent the query itself. The JSON string is composed 4 parts. Below is an explanation of each part and the C\# object which can  serialized into the correct JSON string.

- **Project**: The expression that designates the fields from the device object to include in the query result set (equivalent to SELECT in SQL):

	```
	var projectQuery = {
	    "project": {
	        "all": false,
	        "properties": [{
	            "name": "CustomerId",
	            "type": "service"
	        }, {
	            "name": "Weight",
	            "type": "service"
	        }]
	    }
	}
	```

- **Filter**: The expression that limits the device objects included in the query result set (equivalent to WHERE in SQL):

	```
	var filterQuery = {
	  filter: {
	    property: {
	      name: "CustomerId",
	      type: "service"
	    },
	    value: "123456",
	    comparisonOperator: "eq",
	    type: "comparison"
	  },
	  project: null,
	  aggregate: null,
	  sort: null
	};
	```

- **Aggregate**: The expression that determines how to group the query result set (equivalent to GROUPBY in SQL):

	```
	var aggregateQuery = {
	filter: null,
	project: null,
	aggregate: {
	keys: [
	  {
	    name: "CustomerId",
	    type: "service"
	  }
	],
	properties: [
	  {
	    operator: "sum",
	    property: {
	      name: "Weight",
	      type: "service"
	    },
	    columnName: "TotalWeight"
	  }
	]
	},
	sort: null
	};
	```

- **Sort**: The expression definition which property should be used to sort the query result set (equivalent to ORDER BY in SQL). If sort is null, **deviceID** is used by default:

```
  var sortQuery = {
  filter: null,
  project: null,
  aggregate: null,
  sort: [
      {
          order: "asc",
          property: {
              name: "QoS",
              type: "service"
          }
      }
  ]
  };
```

### Limitations

There are some limitations in the public preview implementation of queries.

-   There is no validation on the query expression.

-   Queries are case sensitive.

-   Only 100 devices will be returned when using query expressions to query by service or device properties. An example of how to implement paging is available in [our query library][lnk-query-samples].

More details on the syntax and available fields for the JSON are [available][lnk-query-expression-guide]. You can also see sample queries in our [query expressions library][lnk-query-samples].

### Query by device and service properties

Once you have the JSON query expression, you can query for the device twins. Call **queryDevices** and check the **result** object for aggregate queries and the **devices** object for all other queries.

```
registry.queryDevices(query, done)
```

### Query by tags

Querying by tags enables you to find device objects without using a JSON query expression. If more than one tag is passed, only device objects with all the tags will be returned. The second parameter is **maxCount**, the maximum number of devices to be returned. The maximum value for **maxCount** is 1000.

```
registry.queryDevicesByTags(['bacon'], 100, done)
```

### Device Implementation

The query is enabled by the Azure IoT Hub device management client library. As long as your device properties are synchronized (as described in [Tutorial: How to use the device twin][lnk-twin-tutorial]), you can query on them. Device properties are available only after the physical device connects to IoT Hub and provides initial values.

## Next steps

To learn more about the Azure IoT Hub device management features you can go through the tutorials:

- [How to use the device twin][lnk-twin-tutorial]
- [How to use device jobs to update device firmware][lnk-jobs-tutorial]



<!-- images and links -->
[lnk-dm-overview]: iot-hub-device-management-overview.md
[lnk-get-started]: iot-hub-device-management-get-started.md
[lnk-twin-tutorial]: iot-hub-device-management-device-twin.md
[lnk-jobs-tutorial]: iot-hub-device-management-device-jobs.md
[lnk-query-spec]: https://github.com/Azure/azure-iot-sdks/blob/dmpreview/node/service/devdoc/query_expression_requirements.md
[lnk-query-samples]: https://github.com/Azure/azure-iot-sdks/blob/dmpreview/doc/get_started/dm_queries/query-samples.md
[lnk-query-expression-guide]:https://github.com/Azure/azure-iot-sdks/blob/dmpreview/node/service/devdoc/query_expression_requirements.md
