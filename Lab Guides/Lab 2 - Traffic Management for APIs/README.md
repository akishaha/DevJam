![](./media/image06.png)

#Lab 2 - Traffic Management for APIs

![](./media/image18.png)

## Objectives

The goal of this lesson is to introduce you to Traffic Management
policies and applying a couple of these policies to the API Proxy you
created in the previous lesson.

The total estimated time required for this exercise: 30 minutes.

## Pre-Requisites
-   Lab 1 is completed

## Background

To maintain performance and availability across a diverse base of client
apps, it's critical to maintain app traffic within the limits of the
capacity of your APIs and backend services. It's also important to
ensure that apps don't consume more resources than permitted.

Apigee Edge provides three policies that enable you to optimize
traffic management to minimize latency for apps while maintaining the
health of backend services.

> NB, a policy in Apigee Edge is a unit of re-usable logic that performs
a certain task. There are policy types for transforming messages,
caching data, checking tokens or keys, creating tokens, and so on.

Each of the traffic management policy types addresses a distinct aspect
of traffic management. In some cases, you might use the three policy
types in a single API proxy.

In addition, Apigee Edge includes a cache, that can also be accessed via
policies, to manage traffic and handling of requests. 


### Spike Arrest 

This policy smoothes traffic spikes by dividing a limit that you define
into smaller intervals. For example, if you define a limit of 100
messages per second, the Spike Arrest policy enforces a limit of about 1
request every 10 milliseconds (1000 / 100); and 30 messages per minute
is smoothed into about 1 request every 2 seconds (60 / 30). The Spike
Arrest limit should be close to capacity calculated for either your
backend service or the API proxy itself. The limit should also be
configured for shorter time intervals, such as seconds or minutes. This
policy should be used to prevent sudden traffic bursts caused by
malicious attackers attempting to disrupt a service using a
denial-of-service (DOS) attack or by buggy client applications.  See
[SpikeArrest](http://docs.apigee.com/api-services/reference/spike-arrest-policy).

### Quota 

This policy enforces consumption limits on client apps by maintaining a
distributed 'counter' that tallies incoming requests. The counter can
tally API calls for any identifiable entity, including apps, developers,
API keys, access tokens, and so on. Usually, API keys are used to
identify client apps. This policy is computationally expensive so, for
high-traffic APIs, it should configured for longer time intervals, such
as a day or month. This policy should be used to enforce business
contracts or SLAs with developers and partners, rather than for
operational traffic management.  See [Quota](http://docs.apigee.com/api-services/reference/quota-policy).

### Concurrent Rate Limit

This policy enables traffic management between API Services and your
backend services. Some backend services, such as legacy applications,
may have strict limits on the number of simultaneous connections they
can support. This policy enforces a limit on the number of requests that
can be sent at any given time from API services to your backend
service. This number is counted across all of the distributed instances
of API Services that may be calling your backend service. Policy limits
and time duration should be configured to match the capacity available
for your backend service.  See [Concurrent Rate Limit](http://docs.apigee.com/api-services/reference/concurrent-rate-limit-policy).


### Caching Policies

Apigee Edge supports different caching policies enabling you to:

* Reduce latency: A request satisfied from the cache gets the
  representation and displays it in a shorter time. The server is more
  responsive with requests satisfied from the cache, which is closer to
  the client than the origin server.

* Reduce network traffic: Representations are reused, reducing the
  impact of processing duplicate or redundant requests. Using a cache
  also reduces the amount of bandwidth you use.

* Persist data across transactions: You can store session data for reuse
  across HTTP transactions.

* Support security:You may need "scope" access to the contents of a
  cache so it can only be accessed in a particular environment or by a
  specific API proxy.

The various caching policies supported by Apigee Edge are:

| Policy name    | Description |
| :------------- | :---------- |
| ResponseCache | Uses a cache to cache the response  from the backend resource. This reduces the number of requests to the resource. When the response is already available in cache, it is returned directly by Edge, without contacting the backend.  [ResponseCache policy](http://apigee.com/docs/api-services/reference/response-cache-policy). |
| PopulateCache | Write arbitrary data, not necessarily response content, to the cache. You can use composite keys for the cache, so that you could cache user information, client information, Nonces, and so on. Each cache item can be tagged with an individual Time-to-Live, so data can be available for 10 seconds or 10 hours or 10 days, etc. [PopulateCache](http://apigee.com/docs/api-services/reference/populate-cache-policy). |
| LookupCache | Retrieve values that have been cached with the PopulateCache policy. [LookupCache](http://apigee.com/docs/api-services/reference/lookup-cache-policy). |
| InvalidateCache | Purges values that have been cached by PopulateCache. [Invalidate Cache policy](http://apigee.com/docs/api-services/reference/invalidate-cache-policy). |


## Part 1: Adding a Spike Arrest Policy

**Estimated Time: 6 minutes**

1. Go to the Apigee Edge Management UI browser tab.

2. Ensure you're viewing the list of API Proxies.  If not, Use the side navigation bar Develope -> API Proxies to select "API Proxies".  
![](./media/select-proxies.png)

3. From the list of proxies, Click the name of *your* API proxy.

4. Click on the develop tab.  
![](./media/click-develop.png)

5. Click on Proxy Endpoints &gt; PreFlow.  
![](./media/click-preflow.png)

  Each proxy has a number of different places at which developers can
  attach policies. You can specify that your policies will run during
  request processing, or during response processing - *Request flow* or
  *Response flow* in Apigee parlance. For each of those, you can set up
  conditions to apply policies to a subset of cases - for example a
  condition will typically match a url path and verb.  `GET /hotels`
  might have a conditional flow. There is also a "preflow" and
  "postflow" which act like wildcards - the policies attached here will
  run all requests on this proxy, before or after any conditional flow,
  respectively.

6. Click on `+ Step` on the Request Flow.  
![](./media/click-plus-step.png)

  By doing this, you are adding a policy step that will run in the
  Request Pre-flow. It will run for all inbound requests. You will see a
  modal dialog that allows you to select a particular policy:   
  ![](./media/modal-add-a-step.png)  

  Feel free to scroll to examine the list of available policies. 


7. Select the ‘Spike Arrest’ policy, and click "Add".  You have now
added a policy, and you will be presented with this view:  
![](./media/spike-policy-added.png)

8. Modify the configuration of the policy, specifying "10pm" - 10 per minute - for the Rate.   
![](./media/spike-arrest-modify.png)

  This shows you the basics related to configuring policies in Apigee
  Edge. There are numerous policy types, and for each one, you may
  configure certain properties to affect their behavior.

  Your configuration should look
  something like the following. The DisplayName and name are not of critical importance:

  ```
  <?xml version="1.0" encoding="UTF-8" standalone="yes"?>
  <SpikeArrest async="false" continueOnError="false" enabled="true" name="Spike-Arrest-1">
      <DisplayName>Spike Arrest 10pm</DisplayName>
      <Properties/>
      <Identifier ref="request.header.some-header-name"/>
      <MessageWeight ref="request.header.weight"/>
      <Rate>10pm</Rate>
  </SpikeArrest>
  ```

9. Save the changes to the proxy and ensure that it is deployed
successfully to the ‘test’ environment.
![](./media/spike-save.png)

  Think of Spike Arrest as a way to generally protect against traffic
  spikes (system wide) rather than as a way to limit traffic to a
  specific number of requests *per user*. Your APIs and backend can
  handle a certain amount of traffic, and the Spike Arrest policy helps
  you smooth traffic to the general amounts you want, or to set safety
  limits.

  The runtime Spike Arrest behavior differs from what you might expect
  to see from the literal per-minute or per-second values you enter.

  For example, say you enter a rate of 30pm (30 requests per minute). In
  testing, you might think you could send 30 requests in 1 second, as
  long as they came within a minute. But that's not how the policy
  enforces the setting. Thirty requests inside a
  1-second period could be considered a mini spike in some environments.

  Instead, Spike Arrest smooths the allowed traffic by dividing your
  settings into smaller intervals:

  * **Per-minute** rates get smoothed into requests allowed intervals of
    **seconds**. For example, 30pm gets smoothed like this: 60 seconds
    (1 minute) / 30pm = 2-second intervals, or about 1 request allowed
    every 2 seconds. A second request inside of 2 seconds will fail.
    Also, a 31st request within a minute will fail.

  * **Per-second** rates get smoothed into requests allowed in intervals
    of **milliseconds**. For example, 10ps gets smoothed like this: 1000
    milliseconds (1 second) / 10ps = 100-millisecond intervals, or about
    1 request allowed every 100 milliseconds . A second request inside
    of 100ms will fail. Also, an 11th request within a second will fail.


10. Now, test the proxy.  Use Postman to quickly send more than 2 requests in 6 seconds and observe that certain requests will receive an error with the errorCode `policies.ratelimit.SpikeArrestViolation`.

11. Let's use the Trace UI to examine what's happening. Return to the Edge UI, and select the Trace tab again. 
![](./media/start-trace.png)

12. Return to Postman, and send a few more requests. Now back to the Edge UI.  Examine the trace. You should see some requests returning 500 errors, indicating that the SpikeArrest has sent back the call. 
![](./media/trace-calls.png)

13. Stop the Trace session.
![](./media/stop-trace.png)


## Part 2: Adding a Response Cache

**Estimated Time: 10 minutes**

Now we'll introduce the ResponseCache policy.  

1. Go to the Apigee Edge Management UI browser tab.

2. Select your API proxy. Select the Develop tab
![](./media/proxy-develop.png)

3. Click on Proxy Endpoints -&gt; PreFlow
![](./media/click-preflow-again.png)

4. Click on “+ Step” on the Request Flow.
![](./media/click-plus-sign-again.png)

5. From the resulting modal dialog, Select the ‘Response Cache’ policy, and click Add.
You should see something like the following: 
![](./media/add-response-cache.png)

  You have now added a second policy to the Request Pre-Flow. Remember,
  these are policies that will run for every inbound request handled by
  this proxy. They run in sequence. You get to specify the order. 

  The configuration for this policy looks like the following:
  ```
  <ResponseCache async="false" continueOnError="false" enabled="true" name="Response-Cache-1">
    <DisplayName>Response Cache-1</DisplayName>
    <Properties/>
    <CacheKey>
        <Prefix/>
        <KeyFragment ref="request.uri" type="string"/>
    </CacheKey>
    <Scope>Exclusive</Scope>
    <ExpirySettings>
        <ExpiryDate/>
        <TimeOfDay/>
        <TimeoutInSec ref="">3600</TimeoutInSec>
    </ExpirySettings>
    <SkipCacheLookup/>
    <SkipCachePopulation/>
  </ResponseCache>
  ```

  This stipulates that the cache should timeout after 3600 seconds, or
  one hour. You can modify this to whatever setting you like. As implied
  by the configuration options, you can also set the expiry to a
  particular time of day - say 3:00am Pacific time each day. This allows
  you to hold a cache all day, and refresh it at the beginning of each
  day, for example. For more information, see [the
  documentation](http://apigee.com/docs/api-services/reference/response-cache-policy).

  Apigee Edge provides a default cache resource that can be used for
  quick testing, which is what is being used in this lesson. Cache
  policies like ResponseCache can also used named cache resources.  A
  Named cache resource can be manipulated administratively, outside of
  policy control. For examine, if you would like to clear a cache
  administratively, you can do that with a named cache resource. It
  takes just a moment.

  For more information on Cache Resources, see [Manage Caches for an
  Environment](http://apigee.com/docs/api-services/content/manage-caches-environment).


10. Your Proxy Endpoints → Default → PreFlow should now look as follows:
![](./media/proxy-req-preflow.png)

11. Click the Target Endpoints → default → PostFlow
![](./media/target-post-click.png)

12. Verify that it looks as follows:
![](./media/target-postflow.png)

13. Save the changes to the API Proxy, and wait for it to successfully deploy.

14. Now, let's test the modifications you've made.  Click the Trace tab and start a Trace session.

15. From Postman, send a test ‘/GET hotels’ request from Postman using the following query parameters:
`zipcode=98101&radius=200`
![](./media/Postman-send-parameterized-query.png)

  In the Postman UI, you should see a success response, and a timing.
  ![](./media/Postman-success-1.png)

16. Wait for 6 to 10 seconds, to avoid the Spike Arrest policy from
  stopping your requests) and send the same request again from
  Postman. You should see a faster response this time.

17. Go back to the Edge UI, in the Trace view and review the transaction
  map of both the requests including the overall elapsed time to process
  both requests.

  The first transaction map will show you that the request was proxied
  to the backend. It should look as follows:
  ![](./media/api-before-cache.png)

  The view for the second transaction will show you that the response
  was served from cache. It will look as follows:
  ![](./media/api-after-cache.png)

  After configuring the Response Cache policy, as expected, after the
  initial request, the second and all other requests for the next 3600
  seconds will be served from the cache and hence avoid executing any
  other policies. Since no request is sent to the target service, the
  overall transaction time has also dropped significantly. This can also
  increase concurrency at high transaction rates.


**Questions:**

* Suppose you modified the ResponseCache policy to have a TTL of 10 seconds.  Then, suppose you sent new requests in.  At this point, Would you expect the cache to be re-filled every 10 seconds?  Explain.

* What would you need to do in order to allow an administrator the ability to clear the cache?


### For Extra Credit:

* Modify the Spike Arrest to be 10ps. Now that we've demonstrated its
  behavior, we don't need it to be so intrusive.

* Use the EdgeUI to examine the cache resources available in the Environment.  
  ![](./media/admin-environments.png)

  You should see something like this, Select test environment:
  
  ![](./media/select-test-env.png)
  
  Create a named cache resource with your initials. Then, return to your
  Proxy, and modify the ResponseCache policy to explicitly use that named
  cache resource.  hint: see [the documentation for
  the ResponseCache
  policy](http://apigee.com/docs/api-services/reference/response-cache-policy). It
  might have something to do with adding an element like this to the
  configuration: ```
  <CacheResource>name_of_cache_resource</CacheResource> ```

  Verify the behavior with 10 second TTL is now changed. Another thing
  to try: reset the TTL in the policy to 3600, and then run a few
  requests from Postman.  Verify that the cache is hot. Return to the
  cache management page, and clear the cache you created.  Then return
  to Postman, and run a new request.  You should see that the cache is
  cold.

  Keep in mind that all of the administrative actions you can perform in
  the Edge UI can also be performed via an administrative API. So you
  could set up an agent to clear the Edge cache based on a slack
  command, or running as a cron job, or part of a CI/CD flow, and so on.


## Summary

That completes this hands-on lesson. You learned how to use the Spike
Arrest to protect the environment from traffic spikes and to use the
Response Cache policy to provide a better overall experience for the API
consumer while reducing network traffic. Obviously like any other
policy, these policies must be used appropriately based upon your use
cases.
