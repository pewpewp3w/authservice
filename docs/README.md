### Configuration Options


##### message `Config` (config/config.proto)

The top-level configuration object. For a simple example, see the [sample JSON in the bookinfo configmap template](https://github.com/istio-ecosystem/authservice/blob/master/bookinfo-example/config/authservice-configmap-template-for-authn-and-authz.yaml).

| Field | Description | Type |
| ----- | ----------- | ---- |
| chains | Each incoming http request is matched against the list of filters in the chain, in order, until a matching filter is found. The first matching filter is then applied to the request. After the first match is made, other filters in the chain are ignored. Order of chain declaration is therefore important. At least one `FilterChain` is required in this array. | (slice of) FilterChain |
| listen_address | The IP address for the Authservice to listen for incoming requests to process. Required. | string |
| listen_port | The TCP port for the Authservice to listen for incoming requests to process. Required. | int32 |
| log_level | The verbosity of logs generated by the Authservice. Must be one of `trace`, `debug`, `info', 'error' or 'critical'. Required. | string |
| threads | The number of threads in the thread pool to use for processing. The main thread will be used for accepting connections, before sending them to the thread-pool for processing. The total number of running threads, including the main thread, will be N+1. Required. | uint32 |
| trigger_rules | List of trigger rules to decide if the Authservice should be used to authenticate the request. The Authservice authentication happens if any one of the rules matched. If the list is not empty and none of the rules matched, the request will be allowed to proceed without Authservice authentication. The format and semantics of `trigger_rules` are the same as the `triggerRules` setting on the Istio Authentication Policy (see https://istio.io/docs/reference/config/security/istio.authentication.v1alpha1). CAUTION: Be sure that your configured `OIDCConfig.callback` and `OIDCConfig.logout` paths each satisfies at least one of the trigger rules, or else the Authservice will not be able to intercept requests made to those paths to perform the appropriate login/logout behavior. Optional. Leave this empty to always trigger authentication for all paths. | (slice of) TriggerRule |
| default_oidc_config | Global configuration of OIDC. This value will be applied to all filter definition when it defined as `oidc_override`. Optional. | oidc.OIDCConfig |
| allow_unmatched_requests | If true will allow the the requests even no filter chain match is found. Default false. Optional. | bool |



##### message `Filter` (config/config.proto)

A filter configuration.

| Field | Description | Type |
| ----- | ----------- | ---- |
| type | The type of filter. Currently, the only valid types are `oidc` and `mock`. Required. | oneof |
| oidc | An OpenID Connect filter configuration. | oidc.OIDCConfig |
| oidc_override | This value will be used when `default_oidc_config` exists. It will override values of them. If that doesn't exist, this configuration will be rejected. | oidc.OIDCConfig |
| mock | Mock filter configuration for testing and letting AuthService run even if no OIDC providers are configured. | mock.MockConfig |



##### message `FilterChain` (config/config.proto)

A chain of one or more filters that will sequentially process an HTTP request.

| Field | Description | Type |
| ----- | ----------- | ---- |
| name | A user-defined identifier for the processing chain used in log messages. Required. | string |
| match | A rule to determine whether an HTTP request should be processed by the filter chain. If not defined, the filter chain will match every request. Optional. | Match |
| filters | The configuration of one of more filters in the filter chain. When the filter chain matches an incoming request, then this list of filters will be applied to the request in the order that they are declared. All filters are evaluated until one of them returns a non-OK response. If all filters return OK, the envoy proxy is notified that the request may continue. The first filter that returns a non-OK response causes the request to be rejected with the filter's returned status and any remaining filters are skipped. At least one `Filter` is required in this array. | (slice of) Filter |



##### message `LogoutConfig` (config/oidc/config.proto)

When specified, the Authservice will destroy the Authservice session when a request is made to the configured path.

| Field | Description | Type |
| ----- | ----------- | ---- |
| path | A http request path that the Authservice matches against to initiate logout. Whenever a request is made to that path, the Authservice will remove the Authservice-specific cookies and respond with a redirect to the configured `redirect_uri`. Removing the cookies causes the user to be unauthenticated in future requests. If the service application has its own logout controller, then it may be desirable to have its logout controller redirect to this path. If the service application does not need its own logout controller, then the application's logout button/link's href can GET or POST directly to this path. Required. | string |
| redirect_uri | A URI specifying the destination to which the Authservice will redirect any request made to the logout `path`. For example, it may be desirable to redirect the logged out user to the homepage of the service application, or to the [logout endpoint of the OIDC Provider](https://openid.net/specs/openid-connect-session-1_0.html#RPLogout). As with all redirects, the user's browser will perform a GET to this URI. Required. | string |



##### message `Match` (config/config.proto)

Specifies how a request can be matched to a filter chain.

| Field | Description | Type |
| ----- | ----------- | ---- |
| header | The name of the http header used to match against. Required. | string |
| criteria | The criteria by which to match. Must be one of `prefix` or `equality`. Required. | oneof |
| prefix | The expected prefix. If the actual value of the header starts with this prefix, then it will be considered a match. | string |
| equality | The expected value. If the actual value of the header exactly equals this value, then it will be considered a match. | string |



##### message `MockConfig` (config/mock/config.proto)

Mock filter config. The only thing which can be defined is whether it allows or rejects any request it matches.

| Field | Description | Type |
| ----- | ----------- | ---- |
| allow | Boolean specifying whether the filter should return OK for any request it matches. Defaults to false (not OK). | bool |



##### message `OIDCConfig` (config/oidc/config.proto)

The configuration of an OpenID Connect filter that can be used to retrieve identity and access tokens via the standard authorization code grant flow from an OIDC Provider.

| Field | Description | Type |
| ----- | ----------- | ---- |
| authorization_uri | The OIDC Provider's [authorization endpoint](https://openid.net/specs/openid-connect-core-1_0.html#AuthorizationEndpoint). Required. | string |
| token_uri | The OIDC Provider's [token endpoint](https://openid.net/specs/openid-connect-core-1_0.html#TokenEndpoint). Required. | string |
| callback_uri | This value will be used as the `redirect_uri` param of the authorization code grant [Authentication Request](https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest). This URL must be one of the Redirection URI values for the Client pre-registered at the OIDC provider. Note: The Istio gateway's VirtualService must be prepared to ensure that this URL will get routed to the service so that the Authservice can intercept the request and handle it (see [example](https://github.com/istio-ecosystem/authservice/blob/master/bookinfo-example/config/bookinfo-gateway.yaml)). Required. | string |
| JwksFetcherConfig | This message defines a setting to allow asynchronous retrieval and update of the JWK for JWT validation at regular intervals. | message |
| jwks_uri | Request URI that has the JWKs. Required. | string |
| periodic_fetch_interval_sec | Request interval to check whether new JWKs are available. If not specified, default to 1200 seconds, 20min. Optional. | uint32 |
| jwks_config |  | oneof |
| jwks | The JSON JWKS response from the OIDC provider’s `jwks_uri` URI which can be found in the OIDC provider's [configuration response](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfigurationResponse). Note that this JSON value must be escaped when embedded in a json configmap (see [example](https://github.com/istio-ecosystem/authservice/blob/master/bookinfo-example/config/authservice-configmap-template.yaml)). Used during token verification. | string |
| jwks_fetcher | Configuration to allow JWKs to be retrieved and updated asynchronously at regular intervals. | JwksFetcherConfig |
| client_id | The OIDC client ID assigned to the filter to be used in the [Authentication Request](https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest). Required. | string |
| client_secret | The OIDC client secret assigned to the filter to be used in the [Authentication Request](https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest). Required. | string |
| scopes | Additional scopes passed to the OIDC Provider in the [Authentication Request](https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest). The `openid` scope is always sent to the OIDC Provider, and does not need to be specified here. Required, but an empty array is allowed. | (slice of) string |
| cookie_name_prefix | A unique identifier of the Authservice's browser cookies. Can be any string. Needed when multiple services in the same domain are each protected by their own Authservice, in which case each service's Authservice should have a unique value to avoid cookie name conflicts. Also needed when an Authservice is configured with multiple `oidc` filters (across multiple `chains`), each sharing a Redis server for their session storage, to avoid having those `oidc` filters read/write the same sessions in Redis. Optional. | string |
| id_token | The configuration for adding ID Tokens as headers to requests forwarded to a service. Required. | TokenConfig |
| access_token | The configuration for adding Access Tokens as headers to requests forwarded to a service. Optional. | TokenConfig |
| logout | When specified, the Authservice will destroy the Authservice session when a request is made to the configured path. Optional. | LogoutConfig |
| absolute_session_timeout | The Authservice associates obtained OIDC tokens with a session ID in a session store. It also stores some temporary information during the login process into the session store, which will be removed when the user finishes the login. This configuration option sets the number of seconds since a user's session with the Authservice has started until that session should expire. When configured to `0`, which is the default value, the session will never timeout based on the time that it was started, but can still timeout due to being idle. When both `absolute_session_timeout` and `idle_session_timeout` are zero, then sessions will never expire. These settings do not affect how quickly the OIDC tokens contained inside the user's session expire. Optional. | uint32 |
| idle_session_timeout | The Authservice associates obtained OIDC tokens with a session ID in a session store. It also stores some temporary information during the login process into the session store, which will be removed when the user finishes the login. This configuration option sets the number of seconds since the most recent incoming request from that user until the user's session with the Authservice should expire. When configured to `0`, which is the default value, session expiration will not consider idle time, but can still consider timeout based on maximum absolute time since added. When both `absolute_session_timeout` and `idle_session_timeout` are zero, then sessions will never expire. These settings do not affect how quickly the OIDC tokens contained inside the user's session expire. Optional. | uint32 |
| trusted_certificate_authority | When specified, the Authservice will trust the specified Certificate Authority when performing HTTPS calls to the Token Endpoint of the OIDC Identity Provider. Optional. | string |
| proxy_uri | The Authservice makes two kinds of direct network connections directly to the OIDC Provider. Both are POST requests to the configured `token_uri` of the OIDC Provider. The first is to exchange the authorization code for tokens, and the other is to use the refresh token to obtain new tokens. Configure the `proxy_uri` when both of these requests should be made through a web proxy. The format of `proxy_uri` is `http://proxyserver.example.com:8080`, where `:<port_number>` is optional. Userinfo (usernames and passwords) in the `proxy_uri` setting are not yet supported. The `proxy_uri` should always start with `http://`. The Authservice will upgrade the connection to the OIDC provider to HTTPS using an HTTP CONNECT request to the proxy server. The proxy server will see the hostname and port number of the OIDC provider in plain text in the CONNECT request, but all other communication will occur over an encrypted HTTPS connection negotiated directly between the Authservice and the OIDC provider. See also the related `trusted_certificate_authority` configuration option. Optional. | string |
| redis_session_store_config | When specified, the Authservice will use the configured Redis server to store session data. Optional. | RedisConfig |



##### message `RedisConfig` (config/oidc/config.proto)

When specified, the Authservice will use the configured Redis server to store session data

| Field | Description | Type |
| ----- | ----------- | ---- |
| server_uri | The Redis server uri, e.g. "tcp://127.0.0.1:6379" | string |



##### message `StringMatch` (config/config.proto)

Describes how to match a given string. Match is case-sensitive.

| Field | Description | Type |
| ----- | ----------- | ---- |
| match_type |  | oneof |
| exact | exact string match. | string |
| prefix | prefix-based match. | string |
| suffix | suffix-based match. | string |
| regex | ECMAscript style regex-based match as defined by [EDCA-262](http://en.cppreference.com/w/cpp/regex/ecmascript). Example: "^/pets/(.*?)?" | string |



##### message `TokenConfig` (config/oidc/config.proto)

Defines how a token obtained through an OIDC flow is forwarded to services.

| Field | Description | Type |
| ----- | ----------- | ---- |
| header | The name of the header that Authservice adds to the request when forwarding to services. The value of this header will contain the `preamble` and the token. This value is case-insensitive, as http header names are case-insensitive. Note that this value must be `Authorization` for the [Istio Authentication Policy](https://istio.io/docs/tasks/security/authn-policy/) to inspect the token. Required. | string |
| preamble | The authentication scheme of the token. For example, when the preamble is `Bearer` and `header` is `Authorization`, the following header will be added to the request to the service: `Authorization: Bearer ID_TOKEN_VALUE`. Note that this value must be `Bearer`, case-sensitive, when header is `Authorization`. Optional. | string |



##### message `TriggerRule` (config/config.proto)

Trigger rule to match against a request. The trigger rule is satisfied if and only if both rules, excluded_paths and include_paths are satisfied.

| Field | Description | Type |
| ----- | ----------- | ---- |
| excluded_paths | List of paths to be excluded from the request. The rule is satisfied if request path does not match to any of the path in this list. Optional. | (slice of) StringMatch |
| included_paths | List of paths that the request must include. If the list is not empty, the rule is satisfied if request path matches at least one of the path in the list. If the list is empty, the rule is ignored, in other words the rule is always satisfied. Optional. | (slice of) StringMatch |



