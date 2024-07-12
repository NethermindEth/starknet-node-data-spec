# Node Data API

Each of the nodes (Juno, Madara, Papyrus, Pathfinder, etc.) **MUST** implement standard peer-to-peer (p2p) communication and **MAY** implement node data telemetry.

### **Network Information Collection:**

- **Crawlers**: Collect network information by connecting to nodes via p2p.
- **Telemetry Servers**: Have a public endpoint open for nodes to report data via WebSocket.

### **Node Data API**:

- Exposed by crawlers and telemetry servers.
- Standardized using JSON-RPC.

### **Integration**:

- Network monitoring tools, visual dashboards (e.g., network dashboards, block explorers), etc., connect to node data API endpoints to retrieve information.
- These tools **MAY** aggregate data as needed.

The diagram below illustrates the interaction between nodes, crawlers, telemetry servers, and various network monitoring tools:

1. **Nodes** communicate with each other via p2p.
2. **Crawlers** connect to nodes to collect network information.
3. **Telemetry Servers** receive data reports from nodes.
4. **Node Data API** endpoints are provided by crawlers and telemetry servers.
5. Network tools like **block explorers**, **network monitors**, and **telemetry dashboards** use these endpoints to access and display node data.

![Node Data API Diagram](images/node-data-api-diagram.png)

### REST API:

Endpoint: `/api/nodes`

Method: `GET`

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Node Data API Response",
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "nodeId": {
        "type": "string"
      },
      "nodeIp": {
        "type": "string",
        "format": "ipv4"
      },
      "nodeName": {
        "type": "string"
      },
      "nodeClientVersion": {
        "type": "string"
      },
      "nodeClientType": {
        "type": "string"
      },
      "cpuUsage": {
        "type": "number"
      },
      "memoryUsage": {
        "type": "number"
      },
      "networkTraffic": {
        "type": "number"
      },
      "blockHeight": {
        "type": "number"
      }
    },
    "required": ["nodeId"]
  }
}
```
