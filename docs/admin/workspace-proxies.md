# Workspace Proxies

> Workspace proxies are in an [experimental state](../contributing/feature-stages.md#experimental-features) and the behavior is subject to change. Use [GitHub issues](https://github.com/coder/coder) to leave feedback. This experiment must be specifically enabled with the `--experiments="moons"` option on both coderd and the workspace proxy.

Workspace proxies provide low-latency experiences for geo-distributed teams.

Coder's networking does a best effort to make direct connections to a workspace. In situations where this is not possible, such as [dashboard connections](../networking/README.md#dashboard-connections), workspace proxies are able to reduce the amount of distance the network traffic needs to travel.

A workspace proxy is a relay connection a developer can choose to use when connecting with their workspace over ssh, a workspace app, port forwarding, etc.

<!-- TODO: Might want to modify this diagram? -->

![ProxyDiagram](../images/workspaceproxy/proxydiagram.png)

# Deploy a workspace proxy

Each workspace proxy should be a unique instance. At no point should 2 workspace proxy instances share the same authentication token. They only require port 443 to be open and are expected to have network connectivity to the coderd dashboard. Workspace proxies **do not** make any database connections.

Workspace proxies can be used in the browser by navigating to the user `Account -> Workspace Proxy`

## Requirements

- The [Coder CLI](../cli.md) must be installed and authenticated as a user with the Owner role.

## Step 1: Create the proxy

Create the workspace proxy and make sure to save the returned authentication token for said proxy. This is the token the workspace proxy will use to authenticate back to primary coderd.

```bash
$ coder proxy create --name=newyork --display-name="USA East" --icon="/emojis/2194.png"
Workspace Proxy "newyork" created successfully. Save this token, it will not be shown again.
Token: 2fb6500b-bb47-4783-a0db-dedde895b865:05271b4ef9432bac14c02b3c56b5a2d7f05453718a1f85ba7e772c0a096c7175
```

To verify it was created.

```bash
$ coder proxy ls
NAME         URL                    STATUS STATUS
newyork                             unregistered
```

## Step 2: Deploy the proxy

Deploying the workspace proxy will also register the proxy with coderd and make the workspace proxy usable. If the proxy deployment is successful, `coder proxy ls` will show an `ok` status code:

```
$ coder proxy lsPM
NAME              URL                           STATUS STATUS
brazil-saopaulo   https://brazil.example.com  ok
europe-frankfurt  https://europe.example.com  ok
sydney            https://sydney.example.com  ok
```

Other Status codes:

- `unregistered` : The workspace proxy was created, and not yet deployed
- `unreachable` : The workspace proxy was registered, but is not responding. Likely the proxy went offline.
- `unhealthy` : The workspace proxy is reachable, but has some issue that is preventing the proxy from being used. `coder proxy ls` should show the error message.
- `ok` : The workspace proxy is healthy and working properly!

### Configuration

<!--
 I am not sure the best way to present this.
 Ideally in the future we can auto sync some of the settings with coderd.
 -->

Workspace proxy configuration overlaps with a subset of the coderd configuration. To see the full list of configuration options: `coder proxy server --help`

```bash
# Proxy specific configuration. These are REQUIRED
# Example: https://coderd.example.com
CODER_PRIMARY_ACCESS_URL="https://<url_of_coderd_dashboard>"
CODER_PROXY_SESSION_TOKEN="<session_token_from_proxy_create>"

# Runtime variables for "coder start".
CODER_HTTP_ADDRESS=0.0.0.0:80
CODER_TLS_ADDRESS=0.0.0.0:443
# Example: https://east.coderd.example.com
CODER_ACCESS_URL="https://<access_url_of_proxy>"
# Example: *.east.coderd.example.com
CODER_WILDCARD_ACCESS_URL="*.<app_hostname_of_proxy>"

CODER_TLS_ENABLE=true
CODER_TLS_CLIENT_AUTH=none
CODER_TLS_CERT_FILE="<cert_file_location>"
CODER_TLS_KEY_FILE="<key_file_location>"

# Additional configuration options are available.
```

### Running on a VM

```bash
# Set configuration options via environment variables, a config file, or cmd flags
coder proxy server
```

<!-- Additional run options? -->
