# Rate Limiting

The Gloo Platform Rate Limiting API allows for very fine grained configuration to meet the needs of the end user. First a user must decide the type of rate limiting they want to enforce. Below is some common types of rate limiting.

Types of rate limiting
* Global inbound - rate limit x number of requests per second
* User based - User can make x number of requests per second
* Per backend - Prevent too many requests from overwhelming a given endpoint

Links:
  - [Rate Limit Docs](https://docs.solo.io/gloo-mesh-enterprise/latest/policies/rate-limit/)
  - [RateLimitServerConfig API](https://docs.solo.io/gloo-mesh-enterprise/latest/reference/api/ratelimit_server_config/#ratelimitserverconfigspec)
  - [RateLimitServerSettings API](https://docs.solo.io/gloo-mesh-enterprise/latest/reference/api/ratelimit_server_settings/)
  - [RateLimit API](https://docs.solo.io/gloo-mesh-enterprise/latest/reference/api/ratelimit/)
  - [RateLimitClientConfig API](https://docs.solo.io/gloo-mesh-enterprise/latest/reference/api/ratelimit_client_config/)
  - [RateLimitPolicy API](https://docs.solo.io/gloo-mesh-enterprise/latest/reference/api/ratelimit_policy/)

## Install Gloo Platform Addons

The Gloo Platform Addons are extentions that helm enable certain features that are offered within. The Gloo Platform addons contain a set of applications and cache to enable rate limiting.

![Gloo Platform Addon Components](../images/gloo-platform-addons.png)

1. Install Gloo Platform Addon applications

```sh
kubectl create namespace gloo-mesh-addons
kubectl label namespace gloo-mesh-addons istio.io/rev=$ISTIO_REVISION

helm upgrade --install gloo-mesh-addons gloo-mesh-agent/gloo-mesh-agent \
  --namespace gloo-mesh-addons \
  --set glooMeshAgent.enabled=false \
  --set rate-limiter.enabled=true \
  --set ext-auth-service.enabled=true \
  --version $GLOO_PLATFORM_VERSION

kubectl apply -f 10-rate-limiting/gloo-mesh-addons-servers.yaml
```

2. Apply the RateLimitPolicy to enable rate limiting
```sh
kubectl apply -f 10-rate-limiting/rate-limit-policy.yaml
```

3. Refresh UI a few times should see it get rate limited. 

4. Optional: see the error with curl
```sh
for i in {1..6}; do curl -iksS -X GET https://$GLOO_GATEWAY_HTTPS | tail -n 10; done
```

4. Remove the RateLimitPolicy
```sh
kubectl delete -f 10-rate-limiting/rate-limit-policy.yaml
```