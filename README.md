# Installing Log collection from Apache Web Server to Splunk via Universal Forwarder.

This document outlines the process for properly installing and configuring Apache and Splunk to handle logs from Apache. ,This will use Splunk's Apache TA and by following these steps, the Web CIM data model will be populated.

Why this matters here.

1. Download Splunk TA and install where needed.
    1. Download Splunkbase app: https://splunkbase.splunk.com/app/3186/
    1. Follow installation instructions: https://docs.splunk.com/Documentation/AddOns/released/ApacheWebServer/Install
1. Configure app to collection logs from Apache Web Servers. (example deployment app in repository)
    1. Deploy app or monitor stanza in inputs.conf similar to below.
```
[monitor:///var/log/apache2/*error.log*]
disabled = 0
blacklist = \.gz$
sourcetype = apache:error

[monitor:///var/log/apache2/*access.log*]
disabled = 0
sourcetype = apache:access
blacklist = \.gz$
```
3. Configure Apache
    1. Download apache configuration file splunk_log.conf file from this repository and copy to /etc/apache2/conf-available on Apache server.
    1. Use `a2enconf splunk_log` to enable configuration
    1. Configure logging to use Splunk log format editing the configuration file for the Apache server located in /etc/apache2/sites-enabled. Alter the logging line to  `   CustomLog ${APACHE_LOG_DIR}/access.log splunk_json`. Note that the filename can be changed, but should match the monitor statement above.
    1. Restart apache

Optional: The CIM Web data model can be accelerated, but should only be done if apps will use this data.

Custom logging Apache documentation shows available variables.
http://httpd.apache.org/docs/current/mod/mod_log_config.html

**Using an Apache reverse proxy.**
1. On the Apache proxy server and Apache instance serving content, use the command `a2enmod remoteip`. The Apache server will then start passing the remote IP to the server serving HTTP content.
1. To each apache host file, add to the VirtualHost stanza:
```
	RemoteIPHeader X-Forwarded-For
	RemoteIPInternalProxy 192.168.237.2
```
1. In the log configuration file (/etc/apache2/conf-enabled/splunk_log.conf), the splunk_json or splunk_kv log format must use the %a in place %h. See below.
```
LogFormat "{\"time\":\"%{%s}t.%{usec_frac}t\", \"bytes_in\":\"%I\", \"bytes_out\":\"%O\", \"cookie\":\"%{Cookie}i\", \"server\":\"%v\", \"dest_port\":\"%p\", \"http_content_type\":\"%{Content-type}i\", \"http_method\":\"%m\", \"http_referrer\":\"%{Referer}i\", \"http_user_agent\":\"%{User-agent}i\", \"ident\":\"%l\", \"response_time_microseconds\":\"%D\", \"client\":\"%a\", \"status\":\"%>s\", \"uri_path\":\"%U\", \"uri_query\":\"%q\", \"user\":\"%u\"}" splunk_json

LogFormat "time=%{%s}t.%{usec_frac}t, bytes_in=%I, bytes_out=%O, cookie=\"%{Cookie}i\", server=%v, dest_port=%p, http_content_type=\"%{Content-type}i\", http_method=\"%m\", http_referrer=\"%{Referer}i\", http_user_agent=\"%{User-agent}i\", ident=\"%l\", response_time_microseconds=%D, client=%a, status=%>s, uri_path=\"%U\", uri_query=\"%q\", user=\"%u\"" splunk_kv
```
