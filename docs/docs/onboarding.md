# Onboarding DSLs

## Preparing Policy for onboarding:

**Step 1: Create a Policy:**

As explained in the introduction, policy is a python class which can be loaded and executed dynamically, here is a sample policy:

This sample simulates a policy that checks if an IP address is allowed based on a list fetched from an external API (using requests). It also supports management actions to update the allowlist or fetch current state.

```python
import requests
import logging

class AIOSv1PolicyRule:
    def __init__(self, rule_id, settings, parameters):
        """
        settings = {
            "allowlist_api": "https://example.com/api/allowlist"
        }
        parameters = {
            "local_allowlist": ["127.0.0.1"]
        }
        """
        self.allowlist_cache = []
        self.settings = settings
        self.parameters = parameters

    def fetch_remote_allowlist(self):
        try:
            response = requests.get(self.settings["allowlist_api"])
            response.raise_for_status()
            data = response.json()
            return data.get("allowlist", [])
        except Exception as e:
            logging.error(f"Failed to fetch remote allowlist: {e}")
            return []

    def eval(self, parameters, input_data, context):
        ip = input_data.get("ip")
        if not ip:
            return {"allowed": False, "reason": "IP missing in input_data", "input_data": input_data}

        # Merge local + remote allowlists
        local = parameters.get("local_allowlist", [])
        if not self.allowlist_cache:
            self.allowlist_cache = self.fetch_remote_allowlist()

        full_allowlist = set(local + self.allowlist_cache)

        allowed = ip in full_allowlist
        return {
            "allowed": allowed,
            "input_data": input_data,
            "reason": "allowed" if allowed else "denied"
        }

    def management(self, action: str, data: dict) -> dict:
        if action == "update_local_allowlist":
            new_ips = data.get("ips", [])
            if not isinstance(new_ips, list):
                return {"status": "error", "message": "Expected 'ips' to be a list"}
            self.parameters["local_allowlist"] = new_ips
            return {"status": "success", "updated_allowlist": new_ips}

        elif action == "fetch_remote_allowlist":
            fetched = self.fetch_remote_allowlist()
            self.allowlist_cache = fetched
            return {"status": "success", "remote_allowlist": fetched}

        elif action == "get_state":
            return {
                "status": "success",
                "local_allowlist": self.parameters.get("local_allowlist", []),
                "cached_remote_allowlist": self.allowlist_cache
            }

        else:
            return {"status": "error", "message": f"Unknown action '{action}'"}

```

requirements.txt - use this if the third party libraries are used:
```
logging
requests
```

**2. Organize the policy code:**

Follow the structure below to organize the policy code:
```
.
-- code
    -- function.py
    -- requirements.txt
```

The file which contains `AIOSv1PolicyRule` must be named `function.py`

**3. Create a tar.xz or .zip archive of the `code` directory:**
```
zip -r ip-allow-list-checker.zip code/
```

**4. Upload the code to assets registry (optional - can use any static content server with public URL)**

If you are using assets registry:
```sh
curl -X POST http://<server-url>/upload_asset \
     -H "Content-Type: multipart/form-data" \
     -F "asset=@./ip-allow-list-checker.zip" \
     -F 'asset_metadata={
           "asset_name": "IP-allow-list",
           "asset_version": { "version": "1.0", "tag": "beta" },
           "asset_metadata": { "description": "IP allow list policy code" },
           "asset_tags": ["policy", "example"]
         }'
```

The API should return a public URL of the policy code if successful.

**5. Create the policy DB entry:

Here is the DB entry of the policy which can be on-boarded into the DB:

For the structure of templates used in `policy_input_schema`, `policy_output_schema`, `policy_settings_schema`, `policy_parameters_schema` and `management_commands_schema` - refer to the **Templates** section.

```json
{
    "name": "ip_allowlist",
    "version": "1.0",
    "release_tag": "beta",
    "metadata": {
      "author_name": "Prasanna",
      "author_email": "openvision.ai",
      "organization": "openvision.ai",
      "country": "US",
      "license": "MIT",
      "category": "network_access_control",
      "use_case": "Control access to internal systems by validating request IPs against a dynamic and local allowlist.",
      "geographic_scope": "Global",
      "audience": ["network_admins", "security_teams"],
      "integration_notes": "Ensure the allowlist API returns a JSON object with a top-level 'allowlist' key containing a list of IPs. ",
      "tested_environments": ["Ubuntu 22.04", "Amazon Linux 2"],
      "execution_environment": "Python 3.10+",
      "compliance_tags": ["ISO27001", "SOC2"]
   },
    "tags": "network,security,ip,access,allowlist",
    "code": "https://example.com/code/ip_allowlist_policy.zip",
    "type": "python_class_v1",
    "policy_input_schema": {
        "ip": {
            "type": "string",
            "description": "IP address of the incoming request",
            "pattern": "^\\d{1,3}(\\.\\d{1,3}){3}$"
        }
    },
    "policy_output_schema": {
        "allowed": {
            "type": "boolean",
            "description": "Whether access is allowed"
        },
        "input_data": {
            "type": "any",
            "description": "Original input data"
        },
        "reason": {
            "type": "string",
            "description": "Reason for the decision"
        }
    },
    "policy_settings_schema": {
        "allowlist_api": {
            "type": "string",
            "description": "HTTP API endpoint that returns a JSON allowlist",
            "pattern": "^https?://.*"
        }
    },
    "policy_parameters_schema": {
        "local_allowlist": {
            "type": "array",
            "description": "List of IPs locally allowed",
            "max_length": 100,
            "items": {
                "type": "string",
                "pattern": "^\\d{1,3}(\\.\\d{1,3}){3}$"
            }
        }
    },
    "policy_settings": {
        "allowlist_api": "https://example.com/api/allowlist"
    },
    "policy_parameters": {
        "local_allowlist": ["127.0.0.1", "192.168.0.1"]
    },
    "management_commands_schema": [
        {
            "name": "update_local_allowlist",
            "description": "Replace the local allowlist with a new set of IPs",
            "schema": {
                "ips": {
                    "type": "array",
                    "items": {"type": "string", "pattern": "^\\d{1,3}(\\.\\d{1,3}){3}$"}
                }
            }
        },
        {
            "name": "fetch_remote_allowlist",
            "description": "Refresh the allowlist from the configured remote API",
            "schema": {}
        },
        {
            "name": "get_state",
            "description": "Return current local and cached remote allowlist",
            "schema": {}
        }
    ],
    "description": "Policy that checks whether a given IP is allowed based on a local list and a remote allowlist API."
}
```

Now use the policies system API to onboard the policy:

```
curl -X POST -H "Content-Type: application/json" -d @./data.json http://<policies-system-url>/policy
```

`policy_rule_uri` will be:
```
ip_allowlist:1.0-beta
```

In general, `policy_rule_uri` will be inferred from `name`, `version` and `release_tag` fields:
```
{name}:{version}-{release_tag}
```

## Using the policy:

Once the policy is on-boarded, it can be loaded and executed using the `aios_policy_sandbox` python library.

Here is an example, go to the `services/policies_system/policies_local_sdk` from project root and build the python package:

```
pip3 install -e .
```

Now you can import and use the `aios_policy_sandbox` to execute the code locally:

```python
from aios_policy_sandbox import LocalPolicyEvaluator

settings={} # override the settings if needed
parameters={} # override the parameters if needed

policy_rule_uri="ip_allowlist:1.0-beta"

executor = LocalPolicyEvaluator(
    policy_rule_uri=policy_rule_uri,
    parameters=parameters,
    settings=settings,
    mode="local"
)

output = executor.execute_policy_rule({
   "ip": "192.168.0.109"
})

print(output)

```

To execute the code using a remote executor:

```python
from aios_policy_sandbox import LocalPolicyEvaluator

settings={} # override the settings if needed
parameters={} # override the parameters if needed

policy_rule_uri="ip_allowlist:1.0-beta"

executor = LocalPolicyEvaluator(
    policy_rule_uri=policy_rule_uri,
    parameters=parameters,
    settings=settings,
    mode="remote",
    executor_id="executor-123"
)

output = executor.execute_policy_rule({
   "ip": "192.168.0.109"
})

print(output)
```

Execute the management command:
```python
output = executor.execute_mgmt_command("update_local_allowlist", {
   "ips": ["192.168.0.100", "192.168.0.102"]
})

print(output)
```
---
