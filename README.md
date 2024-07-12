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

### Telemetry WebSocket API

The client implementations shall connect to a telemetry server via a WebSocket to stream telemetry data as a continuous stream. 

- The Clent will allow to specify the telemetry endpoint URL, which will be used to upload telementry data. 
- Clients are expected to follow the JSON data structure as `payload` structures in subsequent data updates:

```json

{
    "type": "object",
    "properties": {
        "Node": {
            "type": "object",
            "properties": {
                "name": {
                    "type": "string",
                    "description": ""
                },
                "peer_id": {
                    "type": "string"
                },
                "client": {
                    "type": "string"
                },
                "version": {
                    "type": "string"
                },
                "capability": {
                    "type": "string",
                    "enum": [
                        "FullNode",
                        "Sequencer"
                    ]
                }
            }
        },
        "Network": {
            "type": "object",
            "properties": {
                "Network": {
                    "type": "string"
                },
                "Starknet_version": {
                    "type": "string"
                },
                "JSON_RPC_version": {
                    "type": "string"
                },
                "current_block_hash_l2": {
                    "type": "string"
                },
                "current_block_number_l2": {
                    "type": "string"
                },
                "current_block_hash_l1": {
                    "type": "string"
                },
                "current_block_number_l1": {
                    "type": "string"
                },
                "latest_block_time": {
                    "type": "integer",
                    "description": "Expressed in ms"
                },
                "avg_sync_time": {
                    "type": "integer",
                    "description": "Expressed in ms"
                },
                "estimate_sync": {
                    "type": "number",
                    "description": "Expressed in ms"
                },
                "tx_number": {
                    "type": "integer"
                },
                "event_number": {
                    "type": "integer"
                },
                "message_number": {
                    "type": "integer"
                },
                "peer_count": {
                    "type": "integer"
                }
            }
        },
        "System": {
            "type": "object",
            "properties": {
                "node_uptime": {
                    "type": "number"
                },
                "operating_system": {
                    "type": "string"
                },
                "memory": {
                    "type": "integer",
                    "description": "Expressed in MB"
                },
                "memory_usage": {
                    "type": "number",
                    "description": "Expressed in %"
                },
                "CPU": {
                    "type": "integer",
                    "description": "Number of cores"
                },
                "CPU_type": {
                    "type": "string"
                },
                "CPU_usage": {
                    "type": "integer",
                    "description": "Expressed in %"
                },
                "storage": {
                    "type": "integer",
                    "description": "Expressed in MB"
                },
                "storage_usage": {
                    "type": "number",
                    "description": "Expressed in %"
                }
            }
        }
    }
}


```

**NOTE:** the fields in the schema are marked as NOT required, ie. it is possible to upload partial data for subsequent events. 
The client is only required to send fields which it wants to share. The first payload upladed after starting the client should include basic identification data (ie. `name`, `peer_id`, `client`, `version` `capability`). The subsequent payloads may only include `Network`-related fields. The telemetry server will use the unique websocket ID to match incoming payload records to respective Peer_ID.


- Example websocket data feed

(TBA)



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

# Telemetry Service Documentation

## Overview

The telemetry service of Starknet is largely inspired by the telemetry service of [Substrate](https://github.com/paritytech/substrate-telemetry). The process is quite straightforward and aims to monitor and communicate various metrics and events from the nodes.

## Client-side Implementation

On the client side, we have a dynamic payload system that sends data over a WebSocket session. This allows the client to send a set of data or overwrite existing data for a defined session. The implementation in Rust looks like this:

### Telemetry Service

The `TelemetryService` structure is defined as follows:

```rust
pub struct TelemetryService {
    no_telemetry: bool,
    telemetry_endpoints: Vec<(String, u8)>,
    telemetry_handle: TelemetryHandle,
    start_state: Option<mpsc::Receiver<TelemetryEvent>>,
}
```

### Telemetry Events

This service will send `TelemetryEvent`s as follow:

```rust
#[derive(Debug)]
pub struct TelemetryEvent {
    verbosity: VerbosityLevel,
    message: serde_json::Value,
}
```

### Telemetry Handle

Throught a `TelemetryHandle` defined as follow:

```rust
#[derive(Debug, Clone)]
pub struct TelemetryHandle(Option<Arc<mpsc::Sender<TelemetryEvent>>>);

impl TelemetryHandle {
    pub fn send(&self, verbosity: VerbosityLevel, message: serde_json::Value) {
        if message.get("msg").is_none() {
            log::warn!("Telemetry messages should have a message type");
        }
        if let Some(tx) = &self.0 {
            // Drop the message if the channel is full.
            let _ = tx.try_send(TelemetryEvent { verbosity, message });
        }
    }
}
```

### Data Format

The client communicates with the backend using the following data format for each session:

```json
{
  "Headers": {
    "connection": "upgrade",
    "upgrade": "websocket",
    "sec-websocket-key": "BAEH5i294qAwsXDO2ZirPw==",
    "sec-websocket-version": "13",
    "accept": "*/*",
    "host": "localhost:8080"
  },
  "messages": [
    {
      "payload": {
        "name": "Starknode",
        "peer_id": "12D3KooWMo65MdEXUembrdnYTdw588mREZmQrebkmJwpPbkGbony",
        "client": "Madara",
        "version": "0.1.0-8bf85f6dc0b",
        "capability": "Full",
        "network": "Starknet",
        "starknet_version": "0.13.2",
        "json_rpc_version": "0.7.1",
        "operating_system": "MacOs",
        "memory": 16,
        "cpu": 8,
        "storage": 1092
      },
      "ts": "2024-07-11T11:46:08.602688201+00:00"
    },
    {
      "payload": {
        "current_block_hash_l2": "0x47c3637b57c2b079b93c61539950c17e868a28f46cdef28f88521067f21e943",
        "current_block_number_l2": 0,
        "latest_block_time": 320000,
        "avg_sync_time": 382,
        "estimate_sync": 4.5,
        "tx_number": 245,
        "event_number": 1890,
        "message_number": 3,
        "peer_count": 43,
        "node_uptime": 2.3
      },
      "ts": "2024-07-11T11:46:08.996351122+00:00"
    },
    {
      "payload": {
        "current_block_hash_l2": "0x7aefb91c8e2d3a47abcfb5bda9c127a3ea9b1f94b5a6e761b4182e3f01d2f5d4",
        "current_block_number_l2": 1,
        "latest_block_time": 330000,
        "avg_sync_time": 410,
        "estimate_sync": 5.2,
        "tx_number": 300,
        "event_number": 2025,
        "message_number": 5,
        "peer_count": 48
      },
      "ts": "2024-07-12T09:23:11.123456789+00:00"
    },
    {
      "payload": {
        "current_block_hash_l1": "0x1f9090aae28b8a3dceadf281b0f12828e676c326",
        "current_block_number_l1": 20289357
      },
      "ts": "2024-07-11T11:46:09.035827447+00:00"
    },
    {
      "payload": {
        "memory_usage": 4.4,
        "cpu_usage": 3,
        "storage_usage": 245
      },
      "ts": "2024-07-11T11:46:09.035827447+00:00"
    }
  ]
}
```

As you can see here it doesn't matter what data you send when, if it matches our back/front end then it will be displayed in the right place for the current session.
