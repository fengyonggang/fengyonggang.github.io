---
layout: post
title: Spring Security OAuth2
categories: jekyll
---

This is the user guide for the support for [OAuth 2.0][oauth-2.0]. For OAuth 1.0, everything is different, so [see its user guide][see-its-user-guide].

这篇文章是介绍[OAuth2.0][oauth-2.0]的，而对于OAuth1.0则完全不一样，[OAuth1.0请参考][see-its-user-guide]

This user guide is divided into two parts, the first for the OAuth 2.0 provider, the second for the OAuth 2.0 client. For both the provider and the client, the best source of sample code is the [integration tests][integration-tests] and [sample apps][sample-apps].

这篇文章分为两个部分：OAuth 2.0 provider 和 OAuth 2.0 client。 对于provider和client，最好的样例代码请参考[integration tests][integration-tests] 和 [sample apps][sample-apps]。

### OAuth 2.0 Provider

The OAuth 2.0 provider mechanism is responsible for exposing OAuth 2.0 protected resources. The configuration involves establishing the OAuth 2.0 clients that can access its protected resources independently or on behalf of a user. The provider does this by managing and verifying the OAuth 2.0 tokens used to access the protected resources. Where applicable, the provider must also supply an interface for the user to confirm that a client can be granted access to the protected resources (i.e. a confirmation page).

### OAuth 2.0 Provider Implementation

The provider role in OAuth 2.0 is actually split between Authorization Service and Resource Service, and while these sometimes reside in the same application, with Spring Security OAuth you have the option to split them across two applications, and also to have multiple Resource Services that share an Authorization Service. The requests for the tokens are handled by Spring MVC controller endpoints, and access to protected resources is handled by standard Spring Security request filters. The following endpoints are required in the Spring Security filter chain in order to implement OAuth 2.0 Authorization Server:

- `AuthorizationEndpoint` is used to service requests for authorization. Default URL: `/oauth/authorize`.
- `TokenEndpoint` is used to service requests for access tokens. Default URL: `/oauth/token`.

The following filter is required to implement an OAuth 2.0 Resource Server:

- The `OAuth2AuthenticationProcessingFilter` is used to load the Authentication for the request given an authenticated access token.

For all the OAuth 2.0 provider features, configuration is simplified using special Spring OAuth @Configuration adapters. There is also an XML namespace for OAuth configuration, and the schema resides at http://www.springframework.org/schema/security/spring-security-oauth2.xsd. The namespace is http://www.springframework.org/schema/security/oauth2.

[oauth-2.0]: https://tools.ietf.org/html/draft-ietf-oauth-v2
[see-its-user-guide]: http://projects.spring.io/spring-security-oauth/docs/oauth1.html
[integration-tests]: https://github.com/spring-projects/spring-security-oauth/tree/master/tests
[sample-apps]: https://github.com/spring-projects/spring-security-oauth/tree/master/samples/oauth2
