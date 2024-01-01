# How Eureka server auto configuaration works in spring boot and starts
The @EnableEurekaServer annotation. This returns a bean class called Marker. This Marker is an empty class. There is nothing inside that class.

# Why do we turn off the fetch registry from the server side?
We want a central eureka server. If we register then it will also register in the registry and we don't want this. Because by default it always searching for another discovery server listening on port 8761.

# How a client get all the registry info during startup
Before starting a client, it requests the registry info to the Eureka server for the registry info. The Eureka server repond with the info through the rest template and http status code 200. The url is http://localhost:8761/eureka/apps

This url returns the address list in a xml format.

# Heartbeat and lease renewal
Each 30sec the clients sends signal that they are alive. We can also change the default 30s interval. If a service don't send any signal then it is removed from the discovery server's registry.

* Default settings for heartbeat interval

    lease-renewal-interval-in-seconds

    The service will send the heartbeat in each this second

* Default settings for service expiration: The service will be removed from the service discovery after this second.

    lease-expiration-duration-in-seconds

```yaml
eureka:
    instance:
        lease-renewal-interval-in-seconds: 60
```

# Note: everything the netflix eureka server does, it always do by hitting the rest endpoint. For example, remove an service endpoint is 
```bash
DELETE /eureka/v2/apps/appID/instanceID
```

We can get all the rest endpoints from this link

https://github.com/Netflix/eureka/wiki/Eureka-REST-operations

## Note: if we want json response instead xml, put accept: application/json in the request header.

# Delta fetch
If a service is up and running, if it wants to fetch if any new service is registered then fetch the whole registry is a bad idea. The services always communicates with the server each 30s for new update. But for the new service, it hits to another endpoint, it is . The difference is calculated using the date-time. For example, if the interval is 5min and last checked at 6:00. Now, in 6:05, the service will returned what services are connected in between 6:00 and 6:05.

## Note: We can see the all register/un-register things in the log and realize how the rest api is used in discovery service.

If the discovery services are off then the services will throw exception that they are sending heartbeat to unknown host.

# The call stack order
* GET /eureka/apps - fetch the entire registry
* POST /eureka/apps/APP_NAME - register itself in the server registry with details

After 30s fetch details
* GET /eureka/apps - fetch the entire registry
* PUT /eureka/apps/APP_NAME/{service/app url} - send heartbeat
* GET /eureka/apps/delta - check are there any new services registered

# The flow
* Client services gets the updated registry after 30s. When a service needs to communicate with other service it gets the address from its own registry, don't ask to the discovery server. Now, if the discovery server is down and the other service is also down then the service try to connect with other discovery server instance which is running.
