#Overview
Its 2026 and while I could generate artisanal EVE-NG labs it felt like a good time to use some AI for a functional puprose. 

Without first checking I had claude create a MCP for EVE-NG (after running into some headaches a quick google shows this is a solved problem but what fun is that) and it mostly worked, it took the EVE-NG API and wrapped it into a MCP and could boot strap labs. Claude does still have a lot of silly behavior where you still have to treat it like a junior network admin, for instance it still won't conf t to generate RSA keys on cisco images, I've manaually fixed it and fixed its scripts just for it to forget again, I assume at some point it could do it from the global but for whatever reason it just keeps running against that. Otherwise its great and I've generated a few labs of decdent size and complexity this way 

##Lab Examples


##Connectors
Along with the EVE-NG MCP/API I also have claude connected into a netbox instance for IPAM/DCIM and a bookstacks instance for wiki. This works well and eventually will build MCPs for it or use already built ones for the purpose.

##Homelab Setup 
The basic diagram of the setup below
<img width="771" height="871" alt="homelab drawio" src="https://github.com/user-attachments/assets/849ad28f-59c4-4d3a-9490-8ba2407cb7d5" />


##Claude Generated Overview
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
