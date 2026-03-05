
{% capture client-certificate-cn %}

**Note: New in v2.3**

For clients authenticated via X.509 Certificate Chain (mTLS), TBMQ now automatically includes the client's identity in messages processed by Integrations. 
When a message passes an integration topic filter and is delivered to an external system, the outgoing JSON payload will include the certificate's Common Name in a new field:
`"clientCertCn": "CLIENT_CERTIFICATE_COMMON_NAME"`.
{% endcapture %}
{% include templates/info-banner.md content=client-certificate-cn %}
