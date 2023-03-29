# Appendix D

Figure 2. MIDAS API Service Architecture
![MIDAS API service architecture diagram](img/MIDAS-API-service-architecture.png "MIDAS API service architecture showing API connections passing through the CEC firewall and load balancer")
Source: California Energy Commission

This diagram shows the general architecture of the MIDAS API. MIDAS API requests pass through the CEC firewall and load balancer before connecting to the API server. When the API server is responding to a request, it queries the MIDAS database and provides the response back to the user. All connections use Secure Socket Layer (SSL). API connections require that each user request a token using username and password authentication. The token is then used to authenticate requests. Tokens expire after 10 minutes. Passwords stored in the MIDAS database use one-way hashing with key stretching and salting.

Access to the web tool shown in the diagram is only available to CEC employees. Connections to the web tool also pass through the CEC firewall and require authentication.
