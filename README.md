# Overview
Its 2026 and while I could generate artisanal EVE-NG labs by hand and with a loving heart I felt like a good time to use some AI for a functional puprose. 

Without first checking I had claude create a MCP for EVE-NG (after running into some headaches a quick google shows this is a solved problem but what fun is that) and it mostly worked, it took the EVE-NG API and wrapped it into a MCP and could boot strap labs. Claude does still have a lot of silly behavior where you still have to treat it like a junior network admin, for instance it still won't conf t to generate RSA keys on cisco images, I've manaually fixed it and fixed its scripts just for it to forget again, I assume at some point it could do it from the global but for whatever reason it just keeps running against that. Otherwise its great and I've generated a few labs of decdent size and complexity this way 

Claude regularly defaults to just running powershell/ python scripts to do things so there is room for improvement but with the goal in mind of being able to generate baseline eve-ng labs I don't really care about the underlying mechanics (e.g. eve-ng has a inbuilt method to import configs but its problematic so claude just shortcut to using the telnet console to push configs, which I would knock if I hadn't done the same in the past). 

A longer term idea is to build a baseline automation system to generate lab topologies for self study, then refine on those and potentially build a chaos monkey to break things to get the troubleshooting aspect.

Bottom line right now its pretty useful, a lot of folks are instantly anti-AI which is a fine prerogative but for me I've built so many baseline labs and spent so many hours overcoming minor issues that become time sinks that if I can have claude act like a 1000 monkeys beating against it until it gets to 75% then I'm happy because I can hand hold it remotely while playing with my kids and then spend the few hours I have doing the actual part of the lab I set out to do. Don't get me wrong, doing that upfront and initial work is very vaulable if you never have but this project is about automating that pain point and getting to the meat of the labs.   

## Goal
The goal of this project is to simplify the lab creation for (human) network engineers, smooth over pain points and shorten the time between learning and lab'ing. 

## Lab Examples
## Multi-Site Data Center Example
<img width="1461" height="871" alt="image" src="https://github.com/user-attachments/assets/95c7bd42-a683-4c46-948d-f8c306393af9" />


### Palo Alto Lab
<img width="1431" height="882" alt="image" src="https://github.com/user-attachments/assets/ca681554-3498-4235-96ce-459aed11d5f0" />


## Connectors
Along with the EVE-NG MCP/API I also have claude connected into a netbox instance for IPAM/DCIM and a bookstacks instance for wiki. This works well and eventually will build MCPs for it or use already built ones for the purpose.

## Homelab Setup 
The basic diagram of the setup below
<img width="773" height="873" alt="homelab drawio" src="https://github.com/user-attachments/assets/1733b8a5-df2a-4975-a99f-935c8907826c" />


## Lab Generation
In short I tell claude code to generate a lab with some parameters like a 3 tier clos like topology using cisco nexus virtual switches, give it a baseline config w/ a IGP and build a vxlan overlay, document in bookstack and use netbox as an IPAM

### Lab Generation Example
Prompt: "generate a lab a 3 tier clos like topology using cisco nexus virtual switches, give it a baseline config w/ a IGP and build a vxlan overlay, document in bookstack and use netbox as an IPAM, use lightway 7200 routers as test endpoints hanging off the clos and have 2 different encapsulated vlans with test endpoints"

Had to allow a few scripts to run as it used powershell to bootstrap configs and then had to have it use vrfs to seperate labs out in netbox

Result: 
<img width="966" height="754" alt="image" src="https://github.com/user-attachments/assets/a9035e6c-3724-429a-8331-d444f7097044" />
<img width="1521" height="512" alt="image" src="https://github.com/user-attachments/assets/28b98cc2-e23f-4e2c-838e-95d29b2ee778" />
<img width="1720" height="1181" alt="image" src="https://github.com/user-attachments/assets/82745862-3544-4d77-997b-c7779076a5a1" />

In the EVE-NG lab all nodes were off which made me suspicious that it didn't config them, pulling up a device showed the would you like to run startup config so it likely means claude either didn't push the configs so had to enter another prompt

prompt2: "can you do an initial configuration of all lab devices? I just turned on the eveng  lab and noticed devices prompting for initial config"

Claude found that whatever method it was using in python to push console telnet configs didn't work so it fixed that and pushed 

### Config Example

#### Verdict of this example
Not bad but still not great, two main hinderances in turning something like this into a bigger project/ package:
1. Many things are still gated, e.g. getting images (there are tools projects for this but enough of a gray area that I wouldn't incorporate them), getting fremium (CML free at the time of writing this) and similar
2. Claude needs to be treated like a jr network/ system/ developer that happens to know some very advanced things and can google fairly well 

## ToDo
1. Build a improved lab manager (rpi) and document so I don't end up redoing this next year
2. Integrate containerlabs, CML and GNS3 into seperate MCPs
3. Integrate a secrets manager
4. Integrate a ansible/ automation orchestration platform
5. Automated diagram generation workflow
6. Choas Monkey integration
7. Create an internal git
8. Create an internal jira/ project management type thingy
9. Create a grand unified model for lab generation that wraps all the elements together to have abstract and infinite reproducability

## known issues in the backlog
1. Claude just can't bootstrap ssh for cisco devices, always in global context
2. Claude generates each link in the lab with a network connection, it works and it hides them but its unnecessary
3. Claude doesn't check resource requirements, probably should fix on the eveng template side or just tell claude to double check resource requirements before finalizing
4. Claude doesn't really consider resource constraints, mostly fine but if this ever moved beyond just my lab would instantly become an issue for resource constrained envs

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
