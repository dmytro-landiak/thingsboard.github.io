---
layout: docwithnav-mqtt-broker
title: HTTP Service Authentication
description: HTTP Service Authentication documentation

http-provider-control:
  0:
    image: /images/mqtt-broker/security/auth-providers/http/http-provider-control-1.png
    title: 'Go to the Broker Settings card on the Home page to switch authentication providers.'
  1:
    image: /images/mqtt-broker/security/auth-providers/http/http-provider-control-2.png
    title: 'On the Authentication > Providers page, use the switch button in the table’s right column to enable or disable providers.'
  2:
    image: /images/mqtt-broker/security/auth-providers/http/http-provider-control-3.png
    title: 'Select the HTTP row, and click the "Edit" button to configure the provider.'
    
http-provider-config:
  0:
    image: /images/mqtt-broker/security/auth-providers/http/http-provider-config-1.png
    title: 'Configure Endpoint URL, Request method and Credentials.'
  1:
    image: /images/mqtt-broker/security/auth-providers/http/http-provider-config-2.png
    title: 'Configure Headers and Body.'
  2:
    image: /images/mqtt-broker/security/auth-providers/http/http-provider-config-3.png
    title: 'Configure Default client type and Default authorization rules.'
  3:
    image: /images/mqtt-broker/security/auth-providers/http/http-provider-config-4.png
    title: 'Configure Advanced settings.'

http-provider-session:
  0:
    image: /images/mqtt-broker/security/auth-providers/http/http-provider-session-1.png
    title: 'Session details displaying the client was authenticated via the HTTP provider and has APPLICATION type.'
  1:
    image: /images/mqtt-broker/security/auth-providers/http/http-provider-session-2.png
    title: 'Session subscriptions list.'

---

{% include docs/mqtt-broker/security/authentication/http.md %}
