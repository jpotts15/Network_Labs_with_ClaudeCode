
---

## EVE-NG MCP Server

The MCP server wraps the EVE-NG Community REST API and exposes tools Claude can call
directly in conversation:

| Tool | Description |
|------|-------------|
| `list_labs` | List all labs on the server |
| `create_lab` | Create a new lab |
| `add_node` | Add a node (template, image, CPU, RAM, position) |
| `create_network` | Create a bridge/cloud network |
| `connect_interface` | Wire a node interface to a network |
| `list_nodes` | List nodes in a lab with telnet port info |
| `list_networks` | List networks in a lab |
| `start_node` / `stop_node` | Control node power state |
| `get_node_interfaces` | Inspect interface assignments |

### Discovered API quirks

Working with EVE-NG Community Edition surfaced several non-obvious behaviours:

**Network creation always returns `{"id": 1}`**
`POST /api/labs/{lab}/networks` returns a static `id: 1` regardless of the real ID
assigned. Fix: call `list_networks` after creating all networks and build the
`name → id` map from the actual listing.

**`visibility=0` networks are silently discarded on creation**
Networks created with `visibility=0` (direct-wire appearance) are dropped without
error. Fix: create all networks with `visibility=1`, wire all interfaces, then
`PUT visibility=0` on each network.

**Cloud networks (cloud0/cloud1) cannot be created via API**
`POST` to create a cloud-type network returns error `20021`. These must be added
manually in the EVE-NG GUI and connected after the script runs.

---

## Lab Generation

Labs are defined as Python scripts that describe nodes, networks, and connections,
then execute the full build sequence through the MCP client.

### Example: Palo-Alto-Lab

Three remote sites each running a different PA-VM version, connected through a
central ISP router to a hub site with Panorama.
