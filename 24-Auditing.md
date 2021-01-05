# Auditing

* k8s API can save audit logs
* Stages of the k8s API Request
  * `RequestReceived`
    * Events generated as soon as audit handler receives request
  * `ResponseStarted`
    * Once response headers sents, before response body sent (for long running requests)
  * `ResponseComplete`
    * response body has been completed, no more bytes sent
  * `Panic`
    * events when a panic occurs
* Create an Audit policy to log the stages of the API requests
  * `None` - don't log events
  * `Metadata` - Log request metadta(user, timestamp, resource, verb) but not request body
  * `Request` - log event metadata and request body (No Response)
  * `RequestResponse` - log event metadata, request and response bodies (Only for resource requests)

  Type  | Items Logged
  ---   | ---
  `None`     | Nothing Logged
  `Metadata` | Request Metadata (Requesting User, timestamp[, resource, verb])
  `Request`  | event metadata, request body (only resource requests)
  `RequestResponse` | event metadata, request body, response body (only resource requests)

  Sample Policy
  * policy is parsed until a match is found

 ```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages:
  - "RequestReceived"
rules:
# log no "read" actions
- level: None
  verbs: ["get", "watch", "list" ]

# log nothing regarding events
- level: None
  resources:
  - group: "" # core
    resource: [ "events" ]

# log nothing coming from some groups
- level: None
  UserGroups: ["system:nodes"]

# Note: secret data might be stored in logs
- level: RequestResponse
  resources:
  - group: ""
    resources: [ "secrets" ]

# for everything else log
- level: Metadata

```

Where to store data?

* **Audit backends**
  * JSON Files
  * Webhook (external API)
  * Dynamic Backend (AuditSink API)