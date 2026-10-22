# Policies System:

Policies system constitutes various systems, databases and services responsible for archiving the end to end functionality of all the 5 types of policy execution and the applications built to provide onboarding, discovery of policies and executors.

## Schema of the structures used in Policies system:

### PolicyRule:

This schema represents a basic policy unit:

```python
@dataclass
class PolicyRule:
    policy_rule_uri: str
    name: str
    version: str
    release_tag: str
    metadata: Dict
    tags: str
    code: str
    type: str
    policy_input_schema: Dict
    policy_output_schema: Dict
    policy_settings_schema: Dict
    policy_parameters_schema: Dict
    policy_settings: Dict
    policy_parameters: Dict
    management_commands_schema: list
    description: str
```

---

#### Field-wise Explanation:

1. **`policy_rule_uri: str`**  
   - A unique identifier for the policy rule.
   - Constructed as: `"{name}:{version}-{release_tag}"`, e.g., `"access_control:1.0-beta"`.
   - Used to identify a specific version and release of the policy.

2. **`name: str`**  
   - The name of the policy.
   - Acts as the base identifier for the policy.

3. **`version: str`**  
   - The version of the policy (e.g., `"1.0"`, `"2.3"`).
   - Helps in versioning and maintaining backward compatibility.

4. **`release_tag: str`**  
   - A tag indicating the release status of the policy (e.g., `"beta"`, `"stable"`).
   - Used in combination with version to indicate the policy's development stage.

5. **`metadata: Dict`**  
   - Arbitrary metadata related to the policy.
   - Could include author info, creation timestamps, changelogs, etc.

6. **`tags: str`**  
   - A string containing tags associated with the policy.
   - Useful for categorization, filtering, or searching.

7. **`code: str`**  
   - The actual source code of the policy as a string.
   - This code will be executed as part of the policy logic.

8. **`type: str`**  
   - Meta-information about .
   - Corresponds to one of the policy execution types defined earlier.

9. **`policy_input_schema: Dict`**  
   - JSON schema defining the structure of valid input data for the policy.
   - Ensures the policy receives the expected data format.

10. **`policy_output_schema: Dict`**  
   - JSON schema defining the structure of the expected output from the policy.
   - Useful for validating policy output.

11. **`policy_settings_schema: Dict`**  
   - JSON schema for validating the structure of the settings specific to the policy.
   - Ensures configurable settings adhere to expected formats.

12. **`policy_parameters_schema: Dict`**  
   - JSON schema for validating the structure of runtime parameters.
   - Helps ensure correctness and completeness of runtime data.

13. **`policy_settings: Dict`**  
   - Actual settings provided to the policy at configuration time.
   - These can include toggles, thresholds, or default values.

14. **`policy_parameters: Dict`**  
   - Actual runtime parameters provided during policy invocation.
   - Can include request-specific data, user context, etc.

15. **`management_commands_schema`**
   - List of management commands supported and their schemas.
   - Useful for understanding the management commands implemented by the system.

16. **`description: str`**  
   - A human-readable description of the policy.
   - Useful for documentation, display in UI, and user understanding.

---

### PolicyExecutors:

PolicyExecutors represents a remote policy executor that is deployed and registered by a developer on his/her own machine/cluster.

```python
@dataclass
class PolicyExecutors:
    executor_id: str
    executor_host_uri: str
    executor_metadata: Dict
    executor_hardware_info: Dict
    executor_status: str = field(default="healthy")
```

Here’s a field-by-field explanation for the `PolicyExecutors` data class based on its structure and methods:

---

#### Field-wise Explanation:

1. **`executor_id: str`**  
   - A unique identifier for the policy executor.  
   - Typically used to distinguish between multiple executor instances in the system.

2. **`executor_host_uri: str`**  
   - The URI or address (e.g., IP + port or DNS) where the executor service is hosted.  
   - Applications or services will use this URI to send requests to the executor.

3. **`executor_metadata: Dict`**  
   - Arbitrary metadata associated with the executor.  
   - Can include custom tags, descriptions, labels, version info, or deployment details.

4. **`executor_hardware_info: Dict`**  
   - Contains information about the hardware environment of the executor.  
   - Includes the hardware information of the cluster/machine the executor is deployed.

5. **`executor_status: str = field(default="healthy")`**  
   - Indicates the current health or status of the executor (e.g., `"healthy"`, `"unreachable"`, `"busy"`).  
   - Defaults to `"healthy"` if not specified.  
   - Helps in monitoring and orchestrating executions across available executors.

### Function:
Functions are used to represent the Type 4 running instances of policies .

```python
@dataclass
class Function:
    function_id: str
    function_executor_id: str
    function_executor_uri: str
    function_metadata: Dict
    function_tags: List[str]
    function_policy_rule_uri: str
    function_policy_data: Dict
```

#### Field-wise Explanation:

1. **`function_id: str`**  
   - A unique identifier for the deployed policy function.  
   - Used to reference, manage, and invoke the function.

2. **`function_executor_id: str`**  
   - The ID of the policy executor responsible for running this function.  
   - This should correspond to an existing `PolicyExecutors.executor_id`.

3. **`function_executor_uri: str`**  
   - The URI of the executor where the function is deployed and accessible.  
   - Used by the system to route function execution requests.

4. **`function_metadata: Dict`**  
   - Arbitrary metadata related to the function.  
   - Can include deployment time, version, creator info, config flags, etc.

5. **`function_tags: List[str]`**  
   - A list of tags associated with the function.  
   - Useful for categorization, searching, and filtering.

6. **`function_policy_rule_uri: str`**  
   - The unique URI of the policy rule this function is based on.  
   - Should match a `PolicyRule.policy_rule_uri` and helps trace the function to its source policy logic.

7. **`function_policy_data: Dict`**  
   - A full snapshot or reference data of the policy rule used in the function.  
   - May include input/output schemas, settings, parameters, or a copy of the policy metadata.

### Graph:

Graph represents the Type 5 policy rule

```python
@dataclass
class Graph:
    graph_uri: str
    graph_name: str
    graph_version: str
    graph_release_tag: str
    graph_metadata: str
    graph_function_ids: List[str]
    graph_connection_data: Dict
    graph_search_tags: List[str]
    graph_description: str
    graph_input_schema: Dict
    graph_output_schema: Dict

```

#### Field-wise Explanation:

1. **`graph_uri: str`**  
   - A unique identifier for the graph.  
   - Constructed as: `"{graph_name}:{graph_version}-{graph_release_tag}"`.  
   - Used to uniquely identify the versioned graph.

2. **`graph_name: str`**  
   - The human-readable name of the graph.  
   - Represents the identity or purpose of the graph (e.g., `"fraud-detection-graph"`).

3. **`graph_version: str`**  
   - The version of the graph.  
   - Used for versioning and backward compatibility.

4. **`graph_release_tag: str`**  
   - A release tag indicating the status of the graph (e.g., `"beta"`, `"prod"`).  
   - Helps in managing staged releases.

5. **`graph_metadata: str`**  
   - Arbitrary metadata for the graph, stored as a string (could be JSON-encoded).  
   - May include author info, creation date, update notes, etc.

6. **`graph_function_ids: List[str]`**  
   - A list of function IDs (from the `Function` class) that are part of the graph.  
   - Each function represents a node in the DAG.

7. **`graph_connection_data: Dict`**  
   - Defines how the functions are connected within the graph (i.e., the edges of the DAG).  
   - Likely a mapping of function IDs to their downstream function dependencies.

8. **`graph_search_tags: List[str]`**  
   - Tags used to categorize and search for graphs.  
   - Useful for filtering in registries or UIs.

9. **`graph_description: str`**  
   - A human-readable description of the graph.  
   - Explains its intent, functionality, or use cases.

10. **`graph_input_schema: Dict`**  
    - JSON schema defining the structure of the input data accepted by the graph.  
    - Ensures input validation at execution time.

11. **`graph_output_schema: Dict`**  
    - JSON schema defining the expected structure of the output from the graph.  
    - Useful for validation and interoperability.

## Policy system APIs:

### Policy Rule CURD APIs:

#### **1. Create a Policy**
**POST** `/policy`  
**Description:** Creates a new policy rule by passing the full `PolicyRule` JSON.

```bash
curl -X POST http://<policy-system-url>/policy \
  -H "Content-Type: application/json" \
  -d @policy.json
```
> `policy.json` should contain the full `PolicyRule` object.


####  **2. Read/Get a Policy**
**GET** `/policy/<policy_rule_uri>`  
**Description:** Retrieves a policy by its unique `policy_rule_uri`.

```bash
curl http://<policy-system-url>/policy/ip_allowlist:1.0-beta
```

---

#### **3. Update a Policy**
**PUT** `/policy/<policy_rule_uri>`  
**Description:** Updates an existing policy. The payload must contain the full updated policy.

- Accepts Mongo-style content in the body (though full policy replacement is expected).

```bash
curl -X PUT http://<server-url>:5000/policy/ip_allowlist:1.0-beta \
  -H "Content-Type: application/json" \
  -d '{
    "$set": {
      "policy_parameters.local_allowlist": ["10.0.0.1", "8.8.8.8"],
      "description": "Updated policy to allow new internal IP ranges."
    }
  }'
```


#### **4. Delete a Policy**
**DELETE** `/policy/<policy_rule_uri>`  
**Description:** Deletes the policy associated with the given `policy_rule_uri`.

```bash
curl -X DELETE http://<policy-system-url>/policy/ip_allowlist:1.0-beta
```

#### **5. Query Policies**
**POST** `/policy/query`  
**Description:** Performs a query using MongoDB-style filters to search for matching policies.

```bash
curl -X POST http://<server-url>:5000/policy/query \
  -H "Content-Type: application/json" \
  -d '{
    "release_tag": "beta",
    "tags": { "$regex": "security" },
    "version": { "$gte": "1.0" }
  }'
```

> You can use Mongo-style filters like `$regex`, `$in`, `$gt`, etc.

---

### Executor APIs (Type 2 policy execution):
Any developer or user can set up their own executor and register it in the policies system to make it available for use by applications and other users.

Refer to the Infra setup notes to learn how to set up an executor.

Here is a JSON document representing a sample executor registration payload (all the memory units are in `MB`):

```json
{
  "executor_id": "executor-001",
  "executor_metadata": {
    "author_name": "Prasanna",
    "author_email": "openvision.ai",
    "organization": "openvision.ai",
    "country": "US",
    "tags": ["amd_64", "kubernetes-cluster", "NIC"],
    "description": "My executor deployed on kubernetes"
  },
  "executor_hardware_info": {
    "nodes": {
      "count": 2,
      "nodeData": [
        {
          "id": "node-001",
          "vcpus": {
            "count": 12
          },
          "memory": 32005,
          "swap": 975,
          "storage": {
            "disks": 53,
            "size": 488742
          },
          "network": {
            "interfaces": 10,
            "txBandwidth": 0,
            "rxBandwidth": 0
          }
        },
        {
          "id": "node-002",
          "vcpus": {
            "count": 12
          },
          "memory": 32005,
          "swap": 975,
          "storage": {
            "disks": 53,
            "size": 488742
          },
          "network": {
            "interfaces": 10,
            "txBandwidth": 0,
            "rxBandwidth": 0
          }
        }
      ]
    },
    "vcpus": {
      "count": 24
    },
    "memory": 64010,
    "swap": 1950,
    "storage": {
      "disks": 106,
      "size": 977484
    },
    "network": {
      "interfaces": 20,
      "txBandwidth": 0,
      "rxBandwidth": 0
    },
    "id": "cluster-123"
  },
}
```

#### 1. **Create Executor**
**Endpoint:** `POST /executor`  
**Description:** Creates a new executor using the provided payload. The payload must follow the `PolicyExecutors` schema.

```bash
curl -X POST http://<policy-system-url>/executor \
  -H "Content-Type: application/json" \
  -d @executor.json
```

####  **Read Executor**
**Endpoint:** `GET /executor/<executor_id>`  
**Description:** Retrieves executor details for a specific `executor_id`.

```bash
curl http://<policy-system-url>/executor/executor-001
```

####  **Update Executor**
**Endpoint:** `PUT /executor/<executor_id>`  
**Description:** Updates an existing executor. The payload must contain the full updated executor object.

```bash
curl -X PUT http://<policy-system-url>/executor/executor-001 \
  -H "Content-Type: application/json" \
  -d {
    "$set": {
      "metadata.tags": ["amd_64", "zero-downtime", "ai-worker"],
      "metadata.description": "Updated policy to allow new internal IP ranges."
    }
  }
```

#### 4. **Delete Executor**
**Endpoint:** `DELETE /executor/<executor_id>`  
**Description:** Deletes the executor associated with the given `executor_id`.

```bash
curl -X DELETE http://<policy-system-url>/executor/executor-001
```

#### 5. **Query Executors**
**Endpoint:** `POST /executor/query`  
**Description:** Queries executors using MongoDB-style filter criteria.

```bash
curl -X POST http://<server-url>:5000/executor/query \
  -H "Content-Type: application/json" \
  -d '{
        "executor_metadata.tags": { "$in": ["kubernetes-cluster"] },
        "executor_hardware_info.vcpus.count": { "$gte": 16 }
      }'
```

> This will find executors tagged as `kubernetes-cluster` with 16 or more total vCPUs.


#### **Execute Policy on Executor**
**Endpoint:** `POST /executor/<executor_id>/execute_policy`  
**Description:** Executes a policy on the specified executor. You must provide the `policy_rule_uri`, `input_data`, and optional `parameters`. The request is proxied to the executor.

```bash
curl -X POST http://<policy-system-url>/executor/executor-001/execute_policy \
  -H "Content-Type: application/json" \
  -d '{
    "policy_rule_uri": "ip_allowlist:1.0-beta",
    "input_data": {
      "ip": "10.0.0.1"
    },
    "parameters": {
      "local_allowlist": ["10.0.0.1"]
    }
  }'
```

#### **Create Executor Infrastructure**
**Endpoint:** `POST /executor/<executor_id>/create-infra`  
**Description:** Provisions infrastructure for the executor on kubernetes on the target cluster. `cluster_config` includes the JSON data of the kubernetes `config` file of the target cluster.

```bash
curl -X POST http://<policy-system-url>/executor/executor-001/create-infra \
  -H "Content-Type: application/json" \
  -d '{
    "cluster_config": <json-data of the kubeconfig file>
  }'
```

#### **Remove Executor Infrastructure**
**Endpoint:** `DELETE /executor/<executor_id>/remove-infra`  
**Description:** Decommissions or tears down the infrastructure associated with the executor on the target cluster. `cluster_config` includes the JSON data of the kubernetes `config` file of the target cluster.
```bash
curl -X DELETE http://<policy-system-url>/executor/executor-001/remove-infra \
  -H "Content-Type: application/json" \
  -d '{
    "cluster_config": {
      "provider": "kubernetes",
      "region": "us-west"
    }
  }'
```
---

### Jobs APIs (Type 3 policy execution):

#### **Submit Job to Executor**
**Endpoint:** `POST /jobs/submit/<executor_id>`  
**Description:** Submits a one-time job to a specified executor. If `executor_id` is empty, a resource allocator policy is used to select one dynamically.

```bash
curl -X POST http://<policy-system-url>/jobs/submit/executor-001 \
  -H "Content-Type: application/json" \
  -d '{
    "name": "run-ip-check-job",
    "policy_rule_uri": "ip_allowlist:1.0-beta",
    "policy_rule_parameters": {
      "local_allowlist": ["192.168.1.1"]
    },
    "node_selector": {
      "zone": "us-central1-a"
    },
    "inputs": {
      "ip": "192.168.1.1"
    }
  }'
```

#### **Get Job by ID**
**Endpoint:** `GET /jobs/<job_id>`  
**Description:** Retrieves the job status and results using the job's unique identifier.

```bash
curl http://<policy-system-url>/jobs/1a2b3c4d-5678-9012-efgh-1234567890ab
```

#### **Query Jobs**
**Endpoint:** `POST /jobs/query`  
**Description:** Queries job records using MongoDB-style filters (e.g., by executor, policy, job status, etc.).

```bash
curl -X POST http://<policy-system-url>/jobs/query \
  -H "Content-Type: application/json" \
  -d '{
    "policy_rule_uri": "ip_allowlist:1.0-beta",
    "inputs.ip": "192.168.1.1"
  }'
```

---

### Function APIs (Type 4 policy execution):

#### **Create Function Deployment**
**Endpoint:** `POST /function/deployments/create/<executor_id>`  
**Description:** Deploys a function on a specific executor. You must specify the `name`, `policy_rule_uri`, and optionally `policy_rule_parameters`, `replicas`, `autoscaling`, and function metadata/tags. If `executor_id` is an empty string (`""`), dynamic resource allocation will be used via a policy.

```bash
curl -X POST http://<policy-system-url>/function/deployments/create/executor-001 \
  -H "Content-Type: application/json" \
  -d '{
    "name": "check-ip-fn",
    "policy_rule_uri": "ip_allowlist:1.0-beta",
    "policy_rule_parameters": {
      "local_allowlist": ["10.0.0.1"]
    },
    "replicas": 2,
    "autoscaling": {
      "enabled": true,
      "min_replicas": 1,
      "max_replicas": 5,
      "target_cpu_utilization_percentage": 10
    },
    "function_metadata": {
      "maintainer": "network-team",
      "description": "Checks IP allowlist"
    },
    "function_tags": ["network", "security"]
  }'
```

#### **Remove Function Deployment**
**Endpoint:** `DELETE /function/deployments/remove/<name>`  
**Description:** Removes a deployed function by name. Also deletes the function entry from the database.

```bash
curl -X DELETE http://<policy-system-url>/function/deployments/remove/check-ip-fn
```


#### **Call a Deployed Function**
**Endpoint:** `POST /function/call_function/<name>`  
**Description:** Invokes a deployed function with input data. Returns the result of the policy execution.

```bash
curl -X POST http://<policy-system-url>/function/call_function/check-ip-fn \
  -H "Content-Type: application/json" \
  -d '{
    "ip": "10.0.0.1"
  }'
```


#### **Read Function Metadata**
**Endpoint:** `GET /function/<function_id>`  
**Description:** Retrieves metadata and configuration details for the given function ID.

```bash
curl http://<policy-system-url>/function/check-ip-fn
```

#### **Query Functions**
**Endpoint:** `POST /function/query`  
**Description:** Queries functions using MongoDB-style filters. Useful for filtering by tags, policy, executor, etc.

```bash
curl -X POST http://<policy-system-url>/function/query \
  -H "Content-Type: application/json" \
  -d '{
    "function_tags": { "$in": ["network"] },
    "function_policy_rule_uri": "ip_allowlist:1.0-beta"
  }'
```
---

### Graph APIs (Type 5 policy execution)

The graph structure can be defined as follows. Consider the given graph:

```
input_data
    ↓
  funcA
  /    \
funcB  funcC
   \   /
   funcD
    ↓
 final output
```

This graph can be represented in JSON format as follows:

```json
{
    "funcA": ["funcB", "funcC"],
    "funcB": ["funcD"],
    "funcC": ["funcD"]
}
```

Here is the sample graph onboarding payload:
```json
{
  "graph_name": "ip_risk_analysis",
  "graph_version": "1.0",
  "graph_release_tag": "stable",
  "graph_metadata": {
    "author_name": "Prasanna",
    "author_email": "prasanna@opencvision.ai",
    "organization": "OpenVision AI",
    "country": "US",
    "license": "MIT",
    "use_case": "End-to-end evaluation of IP risk using threat intel and activity logs.",
    "category": "security_pipeline",
    "tags": ["ip", "risk", "pipeline", "security", "graph"]
  },
  "graph_function_ids": ["funcA", "funcB", "funcC", "funcD"],
  "graph_connection_data": {
    "funcA": ["funcB", "funcC"],
    "funcB": ["funcD"],
    "funcC": ["funcD"]
  },
  "graph_search_tags": ["ip", "pipeline", "threat", "analysis"],
  "graph_description": "A multi-function policy graph that evaluates the risk level of an incoming IP using allowlist check, threat intelligence, and activity logs before making a final decision.",
  "graph_input_schema": {
    "ip": {
      "type": "string",
      "description": "IP address to be analyzed",
      "pattern": "^\\d{1,3}(\\.\\d{1,3}){3}$"
    }
  },
  "graph_output_schema": {
    "decision": {
      "type": "string",
      "description": "Final access decision",
      "choices": ["allow", "block", "review"]
    },
    "reason": {
      "type": "string",
      "description": "Explanation of decision"
    }
  }
}
```

#### **Create Graph**
**Endpoint:** `POST /graphs`  
**Description:** Creates a new policy graph. The payload must contain a full graph definition as per the `Graph` schema.

```bash
curl -X POST http://<policy-system-url>/graphs \
  -H "Content-Type: application/json" \
  -d @graph.json
```

#### **Get Graph by URI**
**Endpoint:** `GET /graphs/<graph_uri>`  
**Description:** Retrieves a policy graph by its unique `graph_uri`.

```bash
curl http://<policy-system-url>/graphs/ip_risk_analysis:1.0-stable
```

#### **Update Graph**
**Endpoint:** `PUT /graphs/<graph_uri>`  
**Description:** Updates an existing graph. The request body must contain the full updated graph object.

```bash
curl -X PUT http://<policy-system-url>/graphs/ip_risk_analysis:1.0-stable \
  -H "Content-Type: application/json" \
  -d '{
         "$set": {
            "graph_version": "2.0",
            "graph_search_tags": ["ip", "network-analysis", "ip-access-check"]
         }
  }'
```

#### **Delete Graph**
**Endpoint:** `DELETE /graphs/<graph_uri>`  
**Description:** Deletes a policy graph using its `graph_uri`.

```bash
curl -X DELETE http://<policy-system-url>/graphs/ip_risk_analysis:1.0-stable
```

#### **Query Graphs**
**Endpoint:** `POST /graphs/query`  
**Description:** Queries graphs using MongoDB-style filters.

```bash
curl -X POST http://<policy-system-url>/graphs/query \
  -H "Content-Type: application/json" \
  -d '{
    "graph_search_tags": { "$in": ["security"] },
    "graph_version": "1.0"
  }'
```

#### **Execute Graph**
**Endpoint:** `POST /graph/execute_graph`  
**Description:** Executes a graph end-to-end using the provided `graph_uri` and input data. Input must match the starting policy's input schema.

```bash
curl -X POST http://<policy-system-url>/graph/execute_graph \
  -H "Content-Type: application/json" \
  -d '{
    "graph_uri": "ip_risk_analysis:1.0-stable",
    "input_data": {
      "ip": "192.168.1.1"
    }
  }'
```