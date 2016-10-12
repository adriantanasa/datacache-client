# datacache-client

[![NPM Version][npm-image]][npm-url]
[![NPM Downloads][downloads-image]][downloads-url]
[![Build Status][travis-image]][travis-url]

NodeJS request-promise based client for IBM Bluemix Data Cache service.

## Setup

```
npm install datacache-client
```

Using the DataCache storage connector:

```javascript
var DataCacheClient = require('datacache-client');
var dcClient = new DataCacheClient();

// save data to storage
dcClient.put('key-name', {item: 'test'})
	.then(function(resp) {
    	// data saved - continue your logic
        debug('Data saved to storage at key "key-name"');
	}).catch(function(err) {
    	// something went wrong
        debug('Failed to save data to storage');
    });

// get data from storage
dcClient.get('key-name')
	.then(function(resp) {
    	// data loaded in resp.body
        var obj = resp.body;
        debug('Object returned with item = %s', obj.item);
	}).catch(function(err) {
    	// no data found
        debug('Failed to retrieve data from storage');
    });
    
// destroy data
dcClient.destroy('key-name')
	.then(function(resp) {
    	// data deleted from storage
        debug('Data deleted for '"key-name"');
	}).catch(function(err) {
    	// no data found
        debug('Failed to delete data from storage');
    });
```
## API

### .put(key, data, ttl)

**key** - string - name of the key to put data under in DataCache

**data** - Object|string - Data to be saved to cache service - based on the contentType parameter for the client it saves it as JSON or plain text

**ttl** - *optional* - overrides the client Time To Live default value (seconds)

**returns:** Promise

### .get(key)

**key** - string - key name to be used for retrieval of data from cache storage

**returns:** Promise - resolved value has the full response (check resolveWithFullResponse parameter for request-promise)

ex:
resolved.body - Object retrieved from cache
resolved.statusCode - Status code from REST API

### .destroy(key)

**key** - string - key name to be used for deletion

**returns:** Promise - resolved value has the full response (check resolveWithFullResponse parameter for request-promise)

## Storage Options

Bellow is an example with the full list of parameters - default values for optional ones:

```javascript
var store = new DataCacheClient({
        // required parameters when no custom client provided
        restResource: 'http://dcsdomain.bluemix.net/resources/datacaches/{gridName}',
        restResourceSecure: 'https://dcsdomain.bluemix.net/resources/datacaches/{gridName}',
        gridName: '{gridName}',
        username: '{username}',
        password: '{password}',
        // optional parameters - default values
        mapName: '{gridName}',
        eviction: 'LUT',
        locking: 'optimistic',
        contentType: 'application/json',
        secure: true,
        ttl: 3600,
        cfenvServiceName: null
    }
);

```

### Bluemix environment

The datacache client is looking first for DataCache service cfenv values. For the Bluemix NodeJS app with a DataCache service associated the required parameters are read from ENV variables (credentials): 

Environment Variables > VCAP_SERVICES

```json
{
    "system_env_json": {
      "VCAP_SERVICES": {
         "DataCache-dedicated": [
            {
               "credentials": {
                 "catalogEndPoint": "...",
                 "restResource": "http://ip-numeric/resources/datacaches/SYS_GENERATED_GRIDNAME",
                 "restResourceSecure": "https://sdomain.bluemix.net/resources/datacaches/SYS_GENERATED_GRIDNAME",
                 "gridName": "SYS_GENERATED_GRIDNAME",
                 "username": "sysGeneratedUsername",
                 "password": "sysGeneratedPass"
               },
               "name": "datacache-service-name",
               "tags": []
            }
         ]
      }
   },
}
```

### restResources/restResourceSecure
defaults: VCAP_SERVICES credentials values

Depending on the "secure" value, one of them is required if not found in ENV variables by cfenv.


### mapName
For a Bluemix application it is required to have the same value as for "gridName". A resource is identified with a complete URI as:

http://secure.domain/resources/datacaches/SYS_GENERATED_GRIDNAME/MAP_NAME.EVICTION.LOCK/SESSION_KEY

ex:
https://ecaas3.w3ibm.bluemix.net/resources/datacaches/Ae7hjz7tQjuxF44ncLAvuQGH/Ae7hjz7tQjuxF44ncLAvuQGH.LUT.O/sess:t7OQGXZ3x8TNp269-lf-wsdnUBx5OcU6

For non-Bluemix environments can be customized as a namespace for data.

### eviction
- 'LUT' - default - expires based on the Last Update Time
- 'NONE' - data is stored indefinitly (until si programaticaly deleted)
- 'LAT' - expires based on Last Access Time

### locking
- 'optimistic' - default
- 'pessimistic'

### contentType
- 'application/json' - default - turns on the JSON encoder/decoder for session data
- other - saves session data as plain text

### secure
- true - default - uses 'restResourceSecure' as store entrypoint
- false  - uses 'restResource' as store entrypoint

### ttl
- default storage time to live per client (seconds)

### cfenvServiceName
- allows using multiple Data Cache services for same application - loads credentials from ENV using service name;

```
var dcClient = new DataCacheClient({'cfenvServiceName': 'datacache-service-name'});
```


## Contributing

PR code needs to pass the lint check and unit test

```
npm test
```
PR code should be covered by UT

```
npm run coverage
```
## Debugging
The module uses debug npm module - in order to turn the debuging on follow the steps:

Local environment:
```
$> DEBUG=datacache-client npm start
```

CF environmment (Bluemix) - using manifest.yml

```yaml
applications:
- path: .
# ...
  env:
    DEBUG: datacache-client
  services:
  - datacache-service-name
```

### Resources

- http://www.ibm.com/support/knowledgecenter/SSTVLU_8.6.0/com.ibm.websphere.extremescale.doc/tdevrest.html
- https://console.ng.bluemix.net/docs/services/DataCache/index.html#datac001


[npm-image]: https://img.shields.io/npm/v/datacache-client.svg
[npm-url]: https://npmjs.org/package/datacache-client
[travis-image]: https://img.shields.io/travis/adriantanasa/datacache-client/master.svg
[travis-url]: https://travis-ci.org/adriantanasa/datacache-client
[downloads-image]: https://img.shields.io/npm/dm/datacache-client.svg
[downloads-url]: https://npmjs.org/package/datacache-client
