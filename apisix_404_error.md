### Error: **APISIX 404 - File or Directory Not Found / 404 Route Not Found**

#### Description:
I am using Apache APISIX, deployed in a Docker container, to route traffic to a backend website with multiple pages. I have set up an upstream in APISIX pointing to the backend and created a route that matches requests directed to `http://<IP>:<PORT>/login`.

When accessing the route, the login page from the backend displays correctly. However, after entering valid credentials and attempting to log in, I receive a "404 Route Not Found" error. The login process does not complete, and APISIX fails to route to the appropriate page after login.

Both the upstream and the route are properly configured in etcd, and the backend is accessible when accessed directly. What could be the cause of this 404 error post-login, and how can the routing be fixed to ensure smooth navigation through all pages after authentication?

#### Solution:
To resolve the "404 Route Not Found" error after login, follow these steps:

1. **Update the Route Path**  
   - Ensure your route path is set to capture all sub-paths by using a wildcard. For example, if your route path is `/zabbix`, configure it as `/zabbix/*`.

2. **Enable URI Override with Regex**  
   - Access the APISIX configuration, either through the dashboard or via API.
   - Enable **Regex Matching** for the URI override.  
   - Use the following:
     - **Regexp Field**: `^/zabbix/(.*)` (replace `zabbix` with your actual path).
     - **Template Field**: `/$1`.

3. **Test the Route**  
   - Test the routing by accessing various pages of the backend, especially after login, to confirm that all pages are routed correctly.

4. **Update the Route Configuration via CLI**  
   If you are managing APISIX via CLI, you can update the route configuration using the `curl` command. Ensure that the `uri` and `regex_uri` fields are set up correctly:

   ```bash
   curl -i http://127.0.0.1:9180/apisix/admin/routes/1 -H "X-API-KEY: $admin_key" -X PUT -d '
   {
       "uri": "/zabbix/*",
       "plugins": {
           "proxy-rewrite": {
               "regex_uri": ["^/zabbix/(.*)", "/$1"]
           }
       },
       "upstream": {
           "type": "roundrobin",
           "nodes": {
               "10.92.123.7:8000": 1
           }
       }
   }'
   ```

   Ensure to replace `10.92.123.7:8000` with your actual backend server address if it's different.
