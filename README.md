# Installing Log collection from Apache Web Server to Splunk via Universal Forwarder.

This document outlines the process for properly installing and configuring Apache and Splunk to handle logs from Apache. ,This will use Splunk's Apache TA and by following these steps, the Web CIM data model will be populated.

Why this matters here.

1. Download Splunk TA and install where needed.
    1. Download Splunkbase app: https://splunkbase.splunk.com/app/3186/
    1. Follow installation instructions: https://docs.splunk.com/Documentation/AddOns/released/ApacheWebServer/Install
1. Configure app to collection logs from Apache Web Servers. (example deployment app in repository)
    1. Download default web app.
    1. Configure as needed for environment
    2. 
```
[monitor:///var/log/apache2/*error.log*]
disabled = 0
blacklist = \.gz$
sourcetype = apache:error

[monitor:///var/log/apache2/*access.log*]
disabled = 0
#sourcetype = access_common
sourcetype = apache:access
blacklist = \.gz$
```
    3. Upload to deployment server, configure server class, and deploy app.
1. Configure Apache
    1. Download apache configuration file
    1. Copy config file
    1. Configure logging to use Splunk log format
1. Validate

Notes:
Can use syslog forwarding, such as from syslog-ng and json format to forward to HEC.

Custom logging, available variables.
http://httpd.apache.org/docs/current/mod/mod_log_config.html

Using an Apache reverse proxy.
On the Apache proxy server, use the command "a2enconf remoteip". The Apache server will then start passing the remote IP to the server serving HTTP content.
The server must now handle that. "a2enconf remoteip"
To each apache host file, add
	RemoteIPHeader X-Forwarded-For
	RemoteIPInternalProxy 192.168.237.2
Finally, in the log configuration file, the splunk_json or splunk_kv log format must use the %a in place %h.

/etc/apache2/conf-enabled/splunk_log.conf 
LogFormat "{\"time\":\"%{%s}t.%{usec_frac}t\", \"bytes_in\":\"%I\", \"bytes_out\":\"%O\", \"cookie\":\"%{Cookie}i\", \"server\":\"%v\", \"dest_port\":\"%p\", \"http_content_type\":\"%{Content-type}i\", \"http_method\":\"%m\", \"http_referrer\":\"%{Referer}i\", \"http_user_agent\":\"%{User-agent}i\", \"ident\":\"%l\", \"response_time_microseconds\":\"%D\", \"client\":\"%a\", \"status\":\"%>s\", \"uri_path\":\"%U\", \"uri_query\":\"%q\", \"user\":\"%u\"}" splunk_json

LogFormat "time=%{%s}t.%{usec_frac}t, bytes_in=%I, bytes_out=%O, cookie=\"%{Cookie}i\", server=%v, dest_port=%p, http_content_type=\"%{Content-type}i\", http_method=\"%m\", http_referrer=\"%{Referer}i\", http_user_agent=\"%{User-agent}i\", ident=\"%l\", response_time_microseconds=%D, client=%a, status=%>s, uri_path=\"%U\", uri_query=\"%q\", user=\"%u\"" splunk_kv
