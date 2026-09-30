---
name: ccnp-automation-tools
description: >
  Use this skill when choosing, configuring, or troubleshooting network automation and
  configuration management tools — on-box EEM applets and Tcl on IOS-XE, and the
  off-box agent-based and agentless tools. Invoke when the user asks about: automation
  tools, configuration management, config management tool comparison, agent-based,
  agentless, push model, pull model, EEM, Embedded Event Manager, EEM applet, event
  manager applet, event detector, EEM event detectors, syslog event detector, CLI event
  detector, event none, event syslog pattern, event cli pattern, sync yes, action cli
  command, action syslog msg, action mail, action policy, event manager environment, event manager run,
  event manager session cli username, _email_server, _email_to, _email_from, _email_cc,
  $_cli_result, file prompt quiet, debug event manager, debug event manager all, debug
  event manager action cli, debug event manager action mail, HA_EM_6_LOG, Tcl, tclsh,
  Tcl script, ping.tcl, more flash, on-box automation, proactive monitoring, Puppet,
  puppet server, puppet agent, puppet console, PuppetDB, puppet database, server replica,
  Puppet modules, manifests, templates, .pp file, cisco_ios module, Puppet DSL, Puppet
  Forge, forge.puppet.com, monolithic installation, compile servers, server of servers,
  SoS, PE-PostgreSQL, Chef, Chef server, Chef client, Chef workstation, cookbook, recipe,
  .rb file, knife, knife upload, OHAI, Chef Solo, Hosted Chef, Private Chef, kitchen, test
  kitchen, BATS, Bash Automated Testing System, Minitest, RSpec, Serverspec, Ruby, Erlang,
  SaltStack, Salt, salt master, minion, MinionID, salt-minion daemon, 0MQ, ZeroMQ, reactor,
  beacon, remote execution system, jobs, pillars, grains, Salt formulas, SynDic, master of
  masters, globbing, module.function, cmd.run, network.interfaces, targets commands
  arguments, Ansible, agentless automation, control station, WinRM, sudo, playbook, play,
  task, Ansible module, ios_config, ios_command, inventory file, host inventory, ansible,
  ansible-playbook, ansible-docs, ansible-pull, ansible-vault, PLAY RECAP, ok changed
  unreachable failed, gather_facts, connection local, YAML, Yet Another Markup Language,
  three dashes, three periods, YAML list, YAML dictionary, YAML Lint, yamllint, RedHat,
  PPDIOO, Prepare Plan Design Implement Operate Optimize, Puppet Bolt, bolt, bolt command
  run, bolt script run, bolt task run, bolt plan run, bolt file upload, bolt task show,
  orchestrator-driven tasks, standalone tasks, modulename::taskfilename, task metadata
  file, Salt SSH, SaltStack SSH, server-only mode, roster file, lightweight SaltStack,
  human error, misconfiguration, reduced opex, increased agility.
---

## Purpose
Chapter 28 covered how to talk to devices programmatically; this one covers the tools that
actually do the talking. Two families: **on-box** automation with EEM applets and Tcl, where
the device watches itself and reacts, and **off-box** configuration management with Puppet,
Chef, SaltStack, and Ansible — split into **agent-based** (something installed on the node)
and **agentless** (nothing installed, SSH does the work). The exam tests which tool is which
and what each one calls its own parts.

## Key Concepts

**Embedded Event Manager (EEM)**
- **EEM is a very flexible and powerful Cisco IOS tool** that lets engineers build **software
  applets** to automate many tasks, and it derives further power from being able to **build
  custom scripts using Tcl**. Scripts can automatically execute **based on the output of an
  action or an event on a device**.
- **The main benefit: it is all contained within the local device. There is no need to rely on
  an external scripting engine or monitoring device in most cases.** That is what "on-box"
  buys you — the device does not need anything else to be alive for the automation to work.
- Architecture (Figure 29-1): **Cisco IOS EEM Applet Policy** and **Cisco IOS EEM Tcl Policy**
  both *subscribe to receive events and implement policy actions*; they sit above the
  **Policy Director**, which sits above the **Cisco IOS EEM Server**, which sits above the
  **event detectors**.
- **Event detectors**: None, Syslog, SNMP, Timer, Counter, Interface, CLI, OIR, RF,
  IOSWDSYSMON, GOLD, APPL, Process, WDSYSMON, SNMP-Notification, RPC, Track. Underneath them
  sit the subsystems they watch — SNMP agent CPU, Cisco IOS Interface Descriptor Blocks
  (IDBs), counters and memory, the Cisco IOS CLI, diagnostics, OIR, syslog, Cisco IOS
  processes, and HA.

**EEM applets**
- Applets are composed of multiple building blocks; the two primary ones are **events** and
  **actions**.
- **EEM applets use logic similar to the `if-then` statements** of common programming
  languages — *if* an event happens, *then* an action is taken.
- Syslog events are **matched using regular expressions**, which is a very powerful and
  granular way of matching patterns.
- **CLI patterns can also be matched as an event.** When certain commands are entered into the
  router at the CLI, they can trigger an EEM event — the chapter's example matches
  `"write mem.*"` and backs the config up to TFTP as a result.
- **`event none`** means the applet has **no event detector of its own**. It never
  self-triggers, but it can still run **two ways**:
  1. **Manually:** **`event manager run applet-name`** (a **privileged EXEC** command,
     `Router#`, not global config, despite what Boson's explanation says).
  2. **Called by another applet:** a second applet that *does* have an event detector runs it
     with **`action <label> policy applet-name`**. The event belongs to the caller, not to the
     `event none` applet.
  ```
  event manager applet myapplet2
   event syslog pattern "LINK-3-UPDOWN"
   action 1 policy myapplet          ! runs the event-none applet when myapplet2 fires
  ```
  **Exam wording:** Boson's correct answer is "can be run **manually or when triggered by an
  event**". "Only manually" is the trap, because it misses the `action policy` path.
- Three rules the chapter states as NOTEs, all of which are the kind of thing that silently
  breaks an applet:
  - **Include `enable` and `configure terminal` at the beginning of the actions.** The applet
    **assumes the user is in exec mode, not privileged exec or config mode.**
  - **If AAA command authorization is being used, include `event manager session cli username
    <username>`. Otherwise the CLI commands in the applet will fail.**
  - **Use decimal labels like 1.0, 2.0** so new actions can be inserted later (a 1.5 between
    1.0 and 2.0). **Labels are parsed as strings, which means 10.0 comes after 1.0, not after
    9.0.**
- **`$_cli_result`** includes the output of any CLI commands issued in the applet — in the
  chapter's example, the `show interface loopback0` output ends up in both the debug output
  and the email body.
- **`file prompt quiet`** disables the IOS confirmation mechanism that asks to confirm a user's
  actions — which is why the backup applet turns it on before `copy start` and off afterward.
- **The priority and facility of syslog messages can be changed** to fit any environment's
  alerting structure (the chapter's backup applet uses `informational`).
- Environment variables: rather than putting everything on one action line, **`event manager
  environment <name> <value>`** lets you statically set values callable from multiple actions.
  **Custom names and values can be arbitrary, but it is good practice to use common and
  descriptive variables.**

**EEM and Tcl**
- **Using an EEM applet to call Tcl scripts is another very powerful aspect of EEM.** The
  applet runs `tclsh flash:/<script>.tcl` as a CLI action.
- **To see the contents of a Tcl script in flash, use `more flash:<file>`.** The **`more`**
  command can view all other text-based files in local flash as well.

**Why EEM matters operationally**
- **EEM provides on-box monitoring of various components based on a series of events**, and
  once an event is detected an action can take place.
- **This makes network monitoring proactive rather than reactive**, and it **reduces the load
  on the network and improves efficiency from the monitoring system** — because **the devices
  can simply report when there is something wrong instead of the monitoring system continually
  asking the devices if there is anything wrong.** That polling-vs-reporting inversion is the
  whole argument.

**Why automate at all**
- **Much of the value is in moving more quickly than manual configuration**, and **automation
  helps ensure that the level of risk due to human error is significantly reduced through the
  use of proven and tested automation methods.** A team configuring **1000 devices manually by
  logging into each one is likely to introduce misconfigurations** — and it will be very
  time-consuming.
- The common, repetitive configurations teams automate: **device name/IP address, quality of
  service, access list entries, usernames/passwords, SNMP settings, compliance.**
- The counterweight the chapter states plainly: **automation can be dangerous if it duplicates
  a bad process or an erroneous configuration — and this applies to any tool, not just
  Ansible.** Automating a mistake just deploys the mistake faster and to more places.
- **Push vs pull**: **push models push configuration from a centralized tool or management
  server; pull models check in with the server to see if there is any change, and if there is,
  the remote devices pull the updated configuration files down to the end device.**

**Puppet (agent-based)**
- A robust configuration management and automation tool. **Cisco supports Puppet on Catalyst
  switches, Nexus switches, and the Cisco UCS server platform.** It works with many vendors and
  can be used **during the entire lifecycle of a device** — initial deployment, configuration
  management, and repurposing or removing devices.
- Components: a **puppet server** communicates with devices running the **puppet agent**
  (client) installed locally. Changes and automation tasks are executed in the **puppet
  console** and shared between server and agents, and stored in the **puppet database
  (PuppetDB)**, which can live on the same server or a separate box — **so tasks can be saved
  and pushed out to the agents later.**
- **High availability is optional**: a **server replica** acts as a backup, and **if the server
  is unreachable, communications go over the backup path to the replica.**
- **Puppet agents communicate to the puppet server using different TCP connections, and each
  TCP port uniquely represents a communications path from an agent** running on a device.
- **Puppet can periodically verify the configuration on devices**, at whatever frequency the
  ops team deems necessary. **If a configuration is changed it can be alerted on, and also
  automatically put back to the previous configuration** — which is how an organization
  standardizes configs while enforcing a critical parameter set. This is drift detection and
  remediation, and it is Puppet's signature capability in this chapter.
- **Modules** allow configuration of practically anything that can be configured manually, and
  contain three things: **Manifests, Templates, Files.**
- **Manifests are the code that configures the clients or nodes running the puppet agent.
  Manifests are pushed to the devices using SSL and require certificates to be installed to
  ensure the security of the communications** between server and agents.
- The chapter focuses on the **`cisco_ios`** module, which contains multiple manifests and
  **leverages SSH to connect to devices**. Manifests are saved as individual files with the
  extension **`.pp`**.
- **Puppet leverages a domain-specific language (DSL) as its programming language, largely
  based on Ruby**, which makes it simple for network operators to build custom manifests
  **without having to be software developers**.
- **Puppet Forge** (`https://forge.puppet.com`) is a **community where puppet modules,
  manifests, and code can be shared. There is no cost**, and it is a great place to start —
  including design and installation information the chapter itself does not cover. Much of the
  same material is also on github.com.

**Chef (agent-based)**
- **An open source configuration management tool designed to automate configurations and
  operations of a network and server environment. Chef is written in Ruby and Erlang, but when
  it comes to actually writing code within Chef, Ruby is the language used.**
- Chef is similar to Puppet in six ways: **both have free open source versions; both have paid
  enterprise versions; both manage code that needs to be updated and stored; both manage
  devices or nodes to be configured; both leverage a pull model; both function as a
  client/server model.**
- **Code is created on the Chef workstation** and stored in a file called a **recipe**. Once
  created, **it must be uploaded to the Chef server** to be used. **`knife` is the
  command-line tool used to upload cookbooks to the Chef server** — **`knife upload
  cookbookname`**.
- **Four types of Chef server deployment**: **Chef Solo** (server hosted locally on the
  workstation); **Chef Client and Server** (the typical deployment, with distributed
  components); **Hosted Chef** (server hosted in the cloud); **Private Chef** (all components
  within the same enterprise network).
- **Like the puppet server, the Chef server sits between the workstation and the nodes.** All
  cookbooks are stored on it, along with all the tools necessary to transfer node
  configurations to the Chef clients.
- **OHAI is a service installed on the nodes, used to collect the current state of a node and
  send that information back to the Chef server through the Chef client service. The Chef
  server then checks whether any new configuration is needed on the node by comparing the OHAI
  information to the cookbook or recipe.** The **Chef client service** running on the nodes is
  **responsible for all communications to the Chef server** — when a node needs a recipe, the
  client service signals that need back to the server.
- **Because nodes can be unique or identical, the recipes can be the same or different for each
  node.** Recipe files carry the extension **`.rb`**.
- **The `kitchen` is a place where all recipes and cookbooks can automatically be executed and
  tested prior to hitting any production nodes** — analogous to test kitchens in the food
  industry. It allows testing **within the enterprise environment and also across many cloud
  providers and virtualization technologies**, and it supports the common Ruby testing
  frameworks: **Bash Automated Testing System (BATS), Minitest, RSpec, Serverspec.**
- **Puppet and Chef are often seen as interchangeable because they are very similar. Which one
  you use ultimately depends on the skillset and adoption processes of your network
  operations.**

**SaltStack, agent and server mode (agent-based)**
- Another configuration management tool in the same category as Chef and Puppet, with its own
  terminology and architecture. **SaltStack is built on Python and has a Python interface, so a
  user can program directly to SaltStack using Python code.** However, **most of the
  instructions or states sent out to the nodes are written in YAML or a DSL — these are called
  Salt formulas. Formulas can be modified but are designed to work out of the box.**
- Architecture: **SaltStack uses the concept of systems** divided into categories. Where Puppet
  has a puppet server and puppet agents, **SaltStack has masters and minions**.
- **SaltStack can run remote commands to systems in a parallel fashion, which allows for very
  fast performance.** By default it leverages a distributed messaging platform called **0MQ
  (ZeroMQ)** for fast, reliable messaging throughout the networking stack.
- **SaltStack is an event-driven technology** with two components that are easy to reverse:
  - **A reactor lives on the master** and listens for any type of change in the node or device
    that differs from the desired state or configuration. Those changes include
    **command-line configuration, disk/memory/processor utilization, and status of services.**
  - **Beacons live on the minions.** (Minions are similar to the Puppet agents on nodes.) **If a
    configuration changes on a node, a beacon notifies the reactor on the master.** This
    process is called the **remote execution system**, and it helps determine whether the
    configuration is in the appropriate state on the minions. **These actions are called jobs,
    and executed jobs can be stored in an external database for future review or reuse.**
- Instead of modules and manifests, **SaltStack uses pillars and grains** — the other pair that
  is easy to reverse:
  - **Grains run on the minions to gather system information to report back to the master**,
    typically gathered by the **`salt-minion` daemon**. **This is analogous to Chef's OHAI
    service.** Grains provide specifics to the master, on request, about the host — uptime, for
    example.
  - **Pillars store data that a minion can retrieve from the master. Pillars can have certain
    minions assigned to them, and minions not assigned to a specific pillar do not have access
    to that data.** So data can be stored for a specific node or set of nodes inside a pillar,
    completely separate from any other node — **confidential or sensitive information that needs
    to be shared with only specific minions can be secured this way.**
- SaltStack scales to a very large number of devices, has an enterprise version, and has a GUI
  called **SynDic**, which **makes it possible to leverage the master of masters**.
- **Like Puppet, SaltStack has its own DSL. The SaltStack command structure contains targets,
  commands, and arguments.**
  - **The target is the desired system the command should run on.** You can target by
    **MinionID**, or very commonly target all systems with the **asterisk (`*`) wildcard**.
  - A combination such as **`Minion*`** grabs any system whose MinionID starts with "Minion" —
    **this is called globbing.**
  - **The command structure uses the `module.function` syntax followed by the argument.** An
    argument provides detail to the module and function being called.
- **`cmd.run`** executes an ad hoc CLI command across all managed nodes and returns the output
  to the master — `salt '*' cmd.run 'ls -l /etc'`. But **other commands and modules are
  specifically designed for such use cases**: **`network.interfaces`** gathers far more
  structured data from disparate systems — **MAC address, interface names, state, and IPv4 and
  IPv6 addresses**.
- **A team can easily tie the power of Python scripts into SaltStack** to create a powerful
  combination.

**Ansible (agentless)**
- **An automation tool capable of automating cloud provisioning, deployment of applications,
  and configuration management.** It has been around a while and was **catapulted further into
  the mainstream when RedHat purchased the company in 2015**. Popular because of **simplicity**
  and being **open source**.
- Created with four concepts in mind: **Consistent, Secure, Highly reliable, Minimal learning
  curve.**
- **Ansible is an agentless tool — no software or agent needs to be installed on the client
  machines to be managed.** Some consider this a major advantage over other products.
- **Ansible communicates using SSH for a majority of devices, and it can support Windows Remote
  Management (WinRM) and other transport methods.**
- **Ansible does not need an administrative account on the client. It can use built-in
  authorization escalation such as `sudo`** when it needs to raise the level of administrative
  control.
- **Ansible sends all requests from a control station**, which could be a laptop or a server in
  a data center — the computer used to run Ansible and issue changes to the remote hosts.
- **When preparing to automate a task, start with the desired outcome of the automation, then
  create a plan to achieve that outcome.** The methodology the chapter names is the **PPDIOO
  lifecycle: Prepare, Plan, Design, Implement, Operate, Optimize.**
- **Ansible uses playbooks** to deploy configuration changes or retrieve information from hosts.
  **A playbook is a structured set of instructions — much like the playbooks football players
  use to make different plays on the field. A playbook contains multiple plays, and each play
  contains the tasks that each player must accomplish for the play to be successful.**
- **Ansible playbooks are written using YAML (Yet Another Markup Language).** Structure:
  - **Ansible YAML files usually begin with a series of three dashes (`---`) and end with a
    series of three periods (`...`). This structure is optional, but it is common.**
  - **YAML files contain lists and dictionaries.**
  - **Comments begin with a pound sign (`#`).**
  - **Each line of a list can start with a dash and a space (`- `), and indentation makes the
    file readable.**
  - **YAML dictionaries are similar to JSON dictionaries in that they also use key/value pairs,
    but a YAML key/value pair does not need the quotation marks** — JSON is `"key": "value"`,
    YAML is `key: value`.
  - **Lists and dictionaries can be used together** in a single file.
- **YAML Lint (`www.yamllint.com`) is a free online tool to check the format of YAML files to
  make sure they have valid syntax** — paste the contents in, click Go, and it alerts you if
  there is an error.
- **Ansible uses an inventory file to keep track of the hosts it manages.** The inventory can be
  a **named group of hosts or a simple list of individual hosts**. **A host can belong to
  multiple groups and can be represented by either an IP address or a resolvable DNS name** —
  the chapter's example puts `192.168.10.1` in both `[routers]` and `[primary-gateway]`.
- Execution output: **PLAY, TASK, and PLAY RECAP** sections. **PLAY RECAP shows the status** —
  `ok=1` means the change was successful, `changed=1` means a single change was made. In the
  larger example, **`ok=4 changed=3`** means that of four tasks, **three modified the router and
  one saved the configuration**.

**Puppet Bolt (agentless)**
- **Puppet Bolt allows you to leverage the power of Puppet without having to install a puppet
  server or puppet agents on devices or nodes. Much like Ansible, Puppet Bolt connects to
  devices using SSH or WinRM connections. It is an open source tool based on the Ruby language
  and can be installed as a single package.**
- **Tasks can be used for pushing configuration and for managing services**, such as starting
  and stopping services and deploying applications. **Tasks are sharable** — users can visit
  **Puppet Forge** to find and share them. **Tasks are really good for solving problems that
  don't fit in the traditional client/server or puppet server and puppet agent model.**
- The distinguishing capability: **Puppet Bolt allows you to execute a change or configuration
  immediately and then validate it** — where Puppet proper periodically validates that a value
  is configured.
- **Two ways to use Puppet Bolt**:
  - **Orchestrator-driven tasks** — leverage the Puppet architecture to use services to connect
    to devices. **Meant for large-scale environments.**
  - **Standalone tasks** — connect directly to devices or nodes to execute tasks, and **do not
    require any Puppet environment or components to be set up.**
- **`bolt command run <command name>`** followed by the list of devices runs an individual
  command. Scripts can be constructed in **Python, Ruby, or any other scripting language the
  devices can interpret**, and run with **`bolt script run <script name>`** followed by the
  device list.
- **Puppet Bolt copies the script into a temporary directory on the remote device, executes it,
  captures the results, and removes the script from the remote system as if it were never
  copied there — a really clean way of executing remote commands without leaving residual
  scripts or files on the remote devices.**
- **Much as in the Cisco DNA Center and Cisco vManage APIs, Puppet Bolt tasks use an API to
  retrieve data between Puppet Bolt and the remote device**, which gives structure to the data
  Bolt expects to see.
- **Tasks are part of the Puppet modules and use the naming structure
  `modulename::taskfilename`**, invoked with **`bolt task run modulename::taskfilename`**. That
  naming structure is what allows tasks to be shared on Puppet Forge.
- **A task is commonly accompanied by a metadata file in JSON format** containing information
  about the task, how to run it, and comments about how the file is written. **Often the
  metadata file is named the same as the task script but with a JSON extension.** View that
  documentation with **`bolt task show modulename::taskfilename`**.
- **The Puppet Bolt command line is not the Cisco command line** — it can be a Linux, OS-X
  Terminal, or Windows OS. **Puppet Enterprise allows the use of a GUI to execute tasks.**

**SaltStack SSH, server-only mode (agentless)**
- **Salt SSH is SaltStack's agentless option, allowing users to run Salt commands without having
  to install a minion on the remote device or node. Similar in concept to Puppet Bolt.**
- **The main requirements: the remote system must have SSH enabled and Python installed.**
- **Salt SSH connects to a remote system and installs a lightweight version of SaltStack in a
  temporary directory, and can then optionally delete the temporary directory and all files
  upon completion, leaving the remote system clean.** Alternatively **the temporary directories
  can be left on the remote systems** so the files do not have to be reinstalled — **useful when
  time is a consideration**, and common on devices using Salt SSH more frequently than others.
- **Salt SSH can work in conjunction with the master/minion environment, or it can be used
  completely agentless across the environment.**
- **By default, Salt SSH uses roster files to store connection information for any host that
  does not have a minion installed.** Roster files (and most Salt SSH files) are **constructed
  in human-readable form**.
- **A major design consideration: Salt SSH is considerably slower than the 0MQ distributed
  messaging library. However, Salt SSH is still often considered faster than logging in to the
  system to execute the commands.** That is the honest framing — agentless costs you speed
  relative to agent-based, and still beats doing it by hand.
- Benefits of automating daily configuration tasks: **increased agility, reduced opex,
  streamlined management, reduced human error.**

**Choosing between them**
- **A majority of these tools function very similarly to one another**, and there is
  **considerable overlap in the tasks various tools can automate**. There are also **times when
  using multiple tools from different software vendors is appropriate.**
- **The most important factors in choosing a tool are how the tools are used and the skills of
  the operations staff who are adopting them.** A team fluent in **Ruby** may want **Chef**; a
  team confident at the **command line** may fit **Ansible or SaltStack**. **The best tool
  depends on the customer**, and choosing requires a thorough understanding of the differences
  **and solid knowledge of what the operations team is comfortable with and that will play to
  their strengths.**

## Procedure

**Building an EEM applet (the pattern all three examples follow):**
1. **Name the applet**: `event manager applet <NAME>`.
2. **Define the event** — the trigger. `event syslog pattern "<regex>" period <n>` for a log
   message, `event cli pattern "<regex>" sync yes` for a typed command, or **`event none`** for
   an applet you will run by hand.
3. **Start the actions with `enable` and `configure terminal`**, because **the applet assumes
   exec mode**, not privileged exec or config mode.
4. **Number actions with decimal labels** — 1.0, 2.0, 3.0 — leaving room to insert a 1.5 later.
   **Labels sort as strings, so 10.0 follows 1.0, not 9.0.**
5. **Add the CLI, syslog, and mail actions** in order. Use **`$_cli_result`** in a message body
   to include the output of the CLI commands the applet ran.
6. **If AAA command authorization is in use, add `event manager session cli username
   <username>`** — otherwise the applet's CLI commands will fail.
7. **Test it.** Trigger the event and watch `debug event manager action cli`; use **`debug event
   manager all`** for everything, or **`debug event manager action mail`** to isolate SMTP
   problems.

**Calling a Tcl script from EEM:**
1. Put the `.tcl` file in flash. Confirm its contents with **`more flash:<file>.tcl`**.
2. Build an applet with **`event none`** so it only runs on demand.
3. Action 1.0 is `cli command "enable"`; action 1.1 is `cli command "tclsh flash:/<file>.tcl"`.
4. Run it with **`event manager run <applet-name>`** and read the `HA_EM_6_LOG` debug output.

**Deploying a configuration with an Ansible playbook:**
1. **Build the inventory file** — groups in square brackets, hosts as IP addresses or resolvable
   DNS names. A host may appear in more than one group.
2. **Write the playbook in YAML**: `---` to open, `- hosts: <target>`, then `gather_facts:
   false` and `connection: local` for network devices.
3. **Add tasks under `tasks:`**, each with a `name:` and a module call — **`ios_config`** with
   `lines:` and `parents:` for configuration, **`ios_command`** with `commands:` for exec
   commands such as `write memory`.
4. **Validate the YAML** — paste it into **YAML Lint** (`www.yamllint.com`) and confirm it says
   valid before running anything.
5. **Run it: `ansible-playbook <playbook>.yaml`.**
6. **Read the output**: PLAY, then one TASK block per task, then **PLAY RECAP**. `changed=N`
   tells you how many tasks actually modified the device; `ok=N` counts tasks that succeeded;
   `unreachable` and `failed` should both be 0.
7. **Verify on the device.** The chapter does this explicitly — `show startup-config | se
   <filter>` to confirm the configuration landed and the `write memory` task saved it.

**The PPDIOO lifecycle (Figure 29-9):**
1. **Prepare**
2. **Plan**
3. **Design**
4. **Implement**
5. **Operate**
6. **Optimize** — and back to Prepare; it is a cycle, not a line.

## Reference Tables

**Table 29-2 — Common EEM email variables**

| EEM Variable | Description | Example |
|---|---|---|
| `_email_server` | SMTP server IP address or DNS name | `10.0.0.25` or `MAILSVR01` |
| `_email_to` | Email address to send email to | `neteng@yourcompany.com` |
| `_email_from` | Email address of sending party | `no-reply@yourcompany.com` |
| `_email_cc` | Email address of additional email receivers | `helpdesk@yourcompany.com` |

**Table 29-3 — Puppet installation modes**

| Installation Type | Scale |
|---|---|
| **Monolithic** *(the typical and recommended deployment)* | **Up to 4000 nodes** |
| Monolithic with compile servers | **4000 to 20,000 nodes** |
| Monolithic with compile servers and standalone PE-PostgreSQL | **More than 20,000 nodes** |

**Table 29-4 — Puppet and Chef component comparison**

| Chef Component | Puppet Component | Description |
|---|---|---|
| **Chef server** | **Puppet server** | Server functions |
| **Chef client** | **Puppet agent** | Client/agent functions |
| **Cookbook** | **Module** | Collection of code or files |
| **Recipe** | **Manifest** | Code being deployed to make configuration changes |
| **Workstation** | **Puppet console** | Where users interact with the configuration management tool and create code |

**Table 29-5 — Ansible playbook structure**

| Component | Description | Use Case |
|---|---|---|
| **Playbook** | A set of **plays** for remote systems | Enforcing configuration and/or deployment steps |
| **Play** | A set of **tasks** applied to a single host or a group of hosts | Grouping a set of hosts to apply policy or configuration to them |
| **Task** | **A call to an Ansible module** | Logging in to a device to issue a `show` command to retrieve output |

**Table 29-6 — Ansible CLI commands**

| CLI Command | Use Case |
|---|---|
| `ansible` | Runs **modules** against targeted hosts |
| `ansible-playbook` | Runs **playbooks** |
| `ansible-docs` | Provides documentation on syntax and parameters in the CLI |
| `ansible-pull` | **Changes Ansible clients from the default push model to the pull model** |
| `ansible-vault` | **Encrypts YAML files that contain sensitive data** |

> **Ansible is push by default** — `ansible-pull` is what flips it. That matters because
> Puppet and Chef are both **pull** models, so "which model?" is not answerable per-tool
> without this caveat.

**Table 29-7 — High-level tool comparison** *(the single most testable table in the chapter)*

| Factor | Puppet | Chef | Ansible | SaltStack |
|---|---|---|---|---|
| **Architecture** | Puppet servers and puppet agents | Chef server and Chef clients | **Control station** and remote hosts | Salt master and minions |
| **Language** | **Puppet DSL** | **Ruby DSL** | **YAML** | **YAML** |
| **Terminology** | **Modules and manifests** | **Cookbooks and recipes** | **Playbooks and plays** | **Pillars and grains** |
| **Support for large-scale deployments** | Yes | Yes | Yes | Yes |
| **Agentless version** | **Puppet Bolt** | **N/A** | **Yes** *(natively agentless)* | **Salt SSH** |

> **Chef is the only one with no agentless option.** That single N/A is the answer to the
> chapter's most-repeated quiz question.

**Agent-based vs agentless at a glance**

| | Agent-based | Agentless |
|---|---|---|
| Tools in this chapter | **Puppet**, **Chef**, **SaltStack** (master/minion) | **Ansible**, **Puppet Bolt**, **Salt SSH** |
| Installed on the node | Puppet agent / Chef client / salt-minion | **Nothing permanent** |
| Transport | Agent-to-server TCP connections (Puppet uses SSL + certificates); 0MQ for SaltStack | **SSH** (also WinRM for Ansible and Bolt) |
| Node-state collector | Chef **OHAI**; SaltStack **grains** | n/a |
| Speed | Faster — 0MQ is the chapter's benchmark | **Salt SSH is considerably slower than 0MQ**, but still faster than logging in by hand |
| Residue on the device | Persistent agent | Bolt **removes the script afterward**; Salt SSH's temp dir is **optionally** left in place |

**Language and terminology cheat sheet**

| Term | Belongs to |
|---|---|
| **Manifest**, **module**, **Puppet Forge**, **PuppetDB**, **puppet console**, `.pp` | **Puppet** |
| **Recipe**, **cookbook**, **knife**, **OHAI**, **kitchen**, `.rb` | **Chef** |
| **Pillar**, **grain**, **beacon**, **reactor**, **formula**, **minion**, **SynDic**, **0MQ**, **globbing**, `module.function` | **SaltStack** |
| **Playbook**, **play**, **task**, **inventory file**, **control station**, `ansible-vault` | **Ansible** |
| **Applet**, **event detector**, **Tcl**, `$_cli_result`, `event none` | **EEM** |

**EEM event detectors (Figure 29-1)**

| | | | |
|---|---|---|---|
| None | Syslog | SNMP | Timer |
| Counter | Interface | CLI | OIR |
| RF | IOSWDSYSMON | GOLD | APPL |
| Process | WDSYSMON | SNMP-Notification | RPC |
| Track | | | |

## Config Patterns

> **Provenance:** every block below is **transcribed from the chapter's own examples**
> (29-1 through 29-16), with example numbers noted. The EEM applets are real IOS-XE and are
> the most directly lab-able content in the whole Automation domain. **Not gear-validated by
> this skill.** Two blocks contain patterns flagged in the Design Baseline.

**EEM — syslog-triggered applet (Example 29-1)**
```ios-xe
event manager applet LOOP0
 event syslog pattern "Interface Loopback0.* down" period 1
 action 1.0 cli command "enable"
 action 2.0 cli command "config terminal"
 action 3.0 cli command "interface loopback0"
 action 4.0 cli command "shutdown"
 action 5.0 cli command "no shutdown"
 action 5.5 cli command "show interface loopback0"
 action 6.0 syslog msg "I've fallen, and I can't get up!"
 action 7.0 mail server 10.0.0.25 to neteng@yourcompany.com
  from no-reply@yourcompany.com subject "Loopback0 Issues!"
  body "The Loopback0 interface was bounced. Please monitor
  accordingly. "$_cli_result"
```
> Note **action 5.5** — inserted between 5.0 and 6.0, which is exactly why the chapter says to
> use decimal labels.

**EEM — CLI-pattern applet that backs up the config (Example 29-3)**
```ios-xe
event manager environment filename Router.cfg
event manager environment tftpserver tftp://10.1.200.29/
event manager applet BACKUP-CONFIG
 event cli pattern "write mem.*" sync yes
 action 1.0 cli command "enable"
 action 2.0 cli command "configure terminal"
 action 3.0 cli command "file prompt quiet"
 action 4.0 cli command "end"
 action 5.0 cli command "copy start $tftpserver$filename"
 action 6.0 cli command "configure terminal"
 action 7.0 cli command "no file prompt quiet"
 action 8.0 syslog priority informational msg "Configuration File Changed!
  TFTP backup successful."
```
> `file prompt quiet` is turned **on before** the copy and **off after** — the applet cannot
> answer a confirmation prompt, so it removes the prompt and then restores the default.

**EEM — manually triggered applet calling a Tcl script (Examples 29-4, 29-5)**
```ios-xe
event manager applet Ping
 event none
 action 1.0 cli command "enable"
 action 1.1 cli command "tclsh flash:/ping.tcl"
```
```
Router# event manager run Ping
```
```tcl
Router# more flash:ping.tcl
foreach address {
192.168.0.2
192.168.0.3
192.168.0.4
192.168.0.5
192.168.0.6
} { ping $address}
```

**Puppet — manifests (Examples 29-6, 29-7)**
```puppet
ntp_server { '1.2.3.4':
  ensure => 'present',
  key => 94,
  prefer => true,
  minpoll => 4,
  maxpoll => 14,
  source_interface => 'Vlan 42',
}
```
```puppet
banner { 'default':
  motd => 'Violators will be prosecuted',
}
```
> **`ensure => 'present'`** means the NTP server configuration **should be present in the
> running configuration** of the device the manifest runs on — and because **Puppet can run
> periodically**, this manifest becomes a recurring compliance check, not a one-shot push.

**Chef — a recipe (Example 29-8, abridged)**
```ruby
#
# Cookbook Name:: cisco-cookbook
# Recipe:: demo_install
#
Chef::Log.info('Demo cisco_command_config provider')

cisco_command_config 'loop42' do
  action :update
  command '
    interface loopback42
      description Peering for AS 42
      ip address 192.168.1.42/24
  '
end

cisco_command_config 'router_bgp_42' do
  action :update
  command '
    router bgp 42
      router-id 192.168.1.42
      address-family ipv4 unicast
        network 1.0.0.0/8
        redistribute static route-map bgp-statics
      neighbor 10.1.1.1
        remote-as 99
  '
end
```

**SaltStack — remote execution from the master (Figures 29-6, 29-7)**
```bash
salt '*' cmd.run 'ls -l /etc'
salt '*' network.interfaces
```
> `'*'` is the **target**, `cmd.run` / `network.interfaces` is the **module.function**, and the
> quoted string is the **argument**. `Minion*` instead of `*` is **globbing**.

**Ansible — host inventory file (Example 29-12)**
```ini
[routers]
192.168.10.1
192.168.20.1

[switches]
192.168.10.25
192.168.10.26

[primary-gateway]
192.168.10.1
```
> `192.168.10.1` appears in **two groups** — the chapter's point that a host can belong to
> multiple groups.

**Ansible — YAML list, dictionary, and both together (Examples 29-9, 29-10, 29-11)**
```yaml
---
# List of music genres
Music:
      - Metal
      - Rock
      - Rap
      - Country
...
```
```yaml
---
# HR Employee record
Employee1:
    Name: John Dough
    Title: Developer
    Nickname: Mr. DBug
```
```yaml
---
# HR Employee records
-  Employee1:
    Name: John Dough
    Title: Developer
    Nickname: Mr. DBug
    Skills:
      - Python
      - YAML
      - JSON
-  Employee2:
    Name: Jane Dough
    Title: Network Architect
    Nickname: Lay DBug
    Skills:
      - CLI
      - Security
      - Automation
```

**Ansible — interface playbook (Example 29-13)**
```yaml
---
- hosts: CSR1KV-1

  gather_facts: false
  connection: local

  tasks:
   - name: Configure GigabitEthernet2 Interface
     ios_config:
       lines:
          - description Configured by ANSIBLE!!!
          - ip address 10.1.1.1 255.255.255.0
          - no shutdown
       parents: interface GigabitEthernet2

       host: "{{ ansible_host }}"
       username: cisco
       password: testtest
```

**Ansible — EIGRP playbook with a save task (Example 29-14, abridged)**
```yaml
   - name: CONFIG EIGRP 100
     ios_config:
       lines:
          - router eigrp 100
          - eigrp router-id 1.1.1.1
          - no auto-summary
          - network 10.1.1.0 0.0.0.255

       host: "{{ ansible_host }}"
       username: cisco
       password: testtest

   - name: WR MEM
     ios_command:
       commands:
         - write memory

       host: "{{ ansible_host }}"
       username: cisco
       password: testtest
```
```
$ ansible-playbook EIGRP_Configuration_Example.yaml

PLAY [CSR1KV-1] ****************************************************
TASK [Configure GigabitEthernet2 Interface] ************************
changed: [CSR1KV-1]
TASK [CONFIG Gig3] *************************************************
changed: [CSR1KV-1]
TASK [CONFIG EIGRP 100] ********************************************
changed: [CSR1KV-1]
TASK [WR MEM] ******************************************************
ok: [CSR1KV-1]
PLAY RECAP *********************************************************
CSR1KV-1        : ok=4    changed=3    unreachable=0    failed=0
```
> **`ok=4 changed=3`** — four tasks ran, three modified the router, and the fourth (`WR MEM`)
> saved the config without changing it. **Plaintext `username`/`password` in the playbook is
> what `ansible-vault` exists to fix** — see the Design Baseline.

**Salt SSH — roster file (Example 29-16)**
```yaml
managed:
      host: 192.168.10.1
      user: admin
```

**Puppet Bolt — running commands, scripts, and tasks (Figure 29-14)**
```bash
bolt command run <command>          # Run a command remotely
bolt script run <script>            # Upload a local script and run it remotely
bolt task run <task> [params]       # Run a Puppet task
bolt plan run <plan> [params]       # Run a Puppet task plan
bolt file upload <src> <dest>       # Upload a local file

bolt task run modulename::taskfilename --nodes <list>
bolt task show modulename::taskfilename
```
> Options worth knowing: **`--nodes`** in URI format (`ssh://`, `winrm://`; **ssh is the
> default protocol, port 22, or 5985 for winrm**), `--concurrency` (**defaults to 100**),
> `--transport ssh|winrm|pcp`, `--format human|json`, `--sudo`, and **`-k/--insecure`**.

## Design Baseline

Rows traced to the ENCOR 350-401 OCG Chapter 29 with page numbers. Two rows are marked as
**observations by this skill** — the chapter's own code does these things without comment.
**A deviation is a question for the operator, not automatically a finding.**

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| **Start EEM applet actions with `enable` and `configure terminal`** | **The applet assumes the user is in exec mode**, not privileged exec or config mode. Without them the config actions silently fail | An applet whose actions are all exec-level (a `show` and a syslog message, say) | Ch. 29, p. 896 (NOTE) |
| **Add `event manager session cli username <username>` when AAA command authorization is in use** | **Otherwise the CLI commands in the applet will fail** — the applet has no authenticated identity to authorize against | No AAA command authorization configured. See the `network-device-access-control` skill for when that applies | Ch. 29, p. 896 (NOTE) |
| **Use decimal action labels (1.0, 2.0, …)** | It makes it possible to **insert new actions between existing ones later** — a 1.5 between 1.0 and 2.0. And **labels are parsed as strings, so 10.0 comes after 1.0, not after 9.0** | None — the cost is zero and the renumbering pain is real | Ch. 29, p. 896 (NOTE) |
| **Deploy Puppet monolithic unless scale forces otherwise** | **Monolithic is the typical and recommended deployment and supports up to 4000 nodes.** Compile servers and standalone PE-PostgreSQL exist for 4000–20,000 and 20,000+ | Genuinely exceeding 4000 nodes; large deployments may also need a **server of servers (SoS)** to manage distributed puppet servers | Ch. 29, p. 903 |
| **Protect puppet server ↔ agent communications with certificates** | **Manifests are pushed to the devices using SSL and require certificates to be installed** to ensure the security of the communications | None — this is how the transport is defined | Ch. 29, p. 903 |
| **Test Chef recipes and cookbooks in the `kitchen` before production nodes** | **The kitchen is a place where all recipes and cookbooks can automatically be executed and tested prior to hitting any production nodes**, and it supports BATS, Minitest, RSpec, and Serverspec | None stated — this is what it is for | Ch. 29, p. 908 |
| **Use SaltStack pillars to scope sensitive data to specific minions** | **Pillars can have certain minions assigned to them, and minions not assigned do not have access to that data** — so **confidential or sensitive information shared with only specific minions can be secured this way** | Non-sensitive data that every minion needs anyway | Ch. 29, p. 910 |
| **Validate YAML with YAML Lint before running a playbook** | **YAML Lint checks the format of YAML files to make sure they have valid syntax** and alerts you if there is an error. YAML is whitespace-significant, so a syntax error is easy to introduce and invisible to read | A CI pipeline already linting it; an editor with a YAML linter built in | Ch. 29, p. 916 |
| **Encrypt YAML files containing sensitive data with `ansible-vault`** | It is listed in Table 29-6 as exactly this: **"Encrypts YAML files that contain sensitive data."** Playbooks otherwise carry credentials in plaintext | None for real credentials | Ch. 29, p. 916 (Table 29-6) |
| **Start automation from the desired outcome and work through PPDIOO** | **Automation can be dangerous if it duplicates a bad process or an erroneous configuration** — and this **applies to any tool, not just Ansible**. Automating a mistake deploys it faster and wider | None — the chapter is explicit that the planning comes first | Ch. 29, p. 913 |
| **Verify the device configuration after a playbook run** | The chapter does this itself: after `ansible-playbook`, it reads `show startup-config \| se …` to **verify the configuration was correctly applied**. `changed=3` says Ansible thinks it changed three things, not that the device is in the state you wanted | None — PLAY RECAP is Ansible's view, not the device's | Ch. 29, p. 921 |
| **Choose the tool on operations-team skills, not on feature lists** | **The most important factors are how the tools are used and the skills of the operations staff adopting them.** All four support large-scale deployments, so the differentiator is the team | A hard technical constraint, such as needing agentless on devices where no agent can be installed | Ch. 29, p. 925 |
| **Do not leave credentials in plaintext in playbooks** | Examples 29-13 and 29-14 carry `username: cisco` / `password: testtest` inline in the YAML. The chapter supplies the fix — **`ansible-vault`** — but never connects it to these examples | Lab-only playbooks with lab-only credentials, which is what these are | **Observation by this skill** — Ch. 29 Examples 29-13/29-14, cross-referenced to Table 29-6 |
| **Be deliberate about leaving the Salt SSH temporary directory in place** | Salt SSH **can optionally delete the temp directory and all files on completion, leaving the remote system clean**, or leave them **so files do not have to be reinstalled** — a **speed vs. residue** tradeoff, not a default to accept unthinkingly | Devices using Salt SSH frequently, where reinstall time matters — the chapter's own stated case | Ch. 29, p. 924 |

## Verification Commands

| Command | What to look for |
|---------|-----------------|
| `show event manager policy registered` | Every registered applet and Tcl policy, its event type, and class — confirms the applet exists and is registered at all |
| `show event manager statistics policy` | Per-policy run counts — proves whether the applet has **ever actually fired** |
| `show event manager environment` | The `event manager environment` variables and their values (`$tftpserver`, `$filename`) — a typo here breaks the action silently |
| `show event manager history events` | Recent events EEM saw, with time and event type |
| `show running-config \| section event manager` | The applet as configured, including the expanded action labels in string order |
| `event manager run <applet-name>` | Manually fires an `event none` applet — the way to test without waiting for a real event. Privileged EXEC, not global config |
| `debug event manager action cli` | The `HA_EM_6_LOG` trace of each CLI action as it runs — IN/OUT lines showing exactly what the applet typed and what the device replied |
| `debug event manager all` | **Everything** the applet does, including actions the `action cli` debug does not cover |
| `debug event manager action mail` | **Filters out all other debug messages** so you can focus on SMTP errors — the chapter's specific recommendation for mail troubleshooting |
| `more flash:<file>.tcl` | The contents of a Tcl script in flash — and any other text-based file in local flash |
| `show startup-config \| se <filter>` | Post-change verification that the configuration landed **and was saved** — the chapter's own verification step after a playbook run |
| `ansible-playbook <file>.yaml` | **PLAY / TASK / PLAY RECAP.** `ok=` succeeded, `changed=` actually modified something, `unreachable=` and `failed=` should be 0 |
| `ansible-playbook <file>.yaml --check` | Dry run — reports what would change without changing it. *(Added by this skill; not in the chapter — verify against your Ansible version.)* |
| `ansible-playbook <file>.yaml -vvv` | Verbose run showing the connection and module detail behind a failure. *(Added by this skill.)* |
| `ansible-docs <module>` | Syntax and parameters for a module, from the CLI |
| YAML Lint (`www.yamllint.com`) | **"Valid YAML!"** before you run anything — paste and click Go |
| `knife upload <cookbookname>` | Uploads a cookbook from the Chef workstation to the Chef server — required before a recipe can be used |
| `salt '*' network.interfaces` | Per-minion interface data — MAC, names, state, IPv4/IPv6. Also a quick proof that the master can reach its minions |
| `salt '*' cmd.run '<command>'` | Ad hoc command across all managed nodes, output returned to the master |
| `bolt task show modulename::taskfilename` | The task's **JSON metadata** — what it does, how to run it, and comments on how it is written |
| `bolt command run '<command>' --nodes <list>` | Runs a command remotely over SSH/WinRM with no agent installed |

## Intent Questions

- **Should this automation live on the box or off it?** EEM is the answer when the device must
  react to its own events with no external dependency. A configuration management tool is the
  answer when many devices must be made consistent. Using one where the other belongs is the
  most common design error in this space.
- **Agent-based or agentless — and is that a choice or a constraint?** If an agent cannot be
  installed on the target, the decision is already made (Ansible, Puppet Bolt, Salt SSH). If it
  can, agent-based buys you speed and continuous drift detection.
- **Push or pull?** Puppet and Chef are **pull**; Ansible is **push** by default and only pulls
  with `ansible-pull`. This changes where the authority lives and what happens when the server
  is unreachable.
- **Is this tool supposed to enforce state continuously, or apply a change once?** Puppet
  **periodically verifies configuration and can automatically revert drift**. Puppet Bolt
  **executes immediately and validates**. Those are different jobs and they fail differently.
- **What does the operations team already know?** The chapter's own answer to "which tool" is
  the team's skillset — Ruby points at Chef, CLI comfort points at Ansible or SaltStack. A tool
  nobody on the team can debug at 3 a.m. is the wrong tool regardless of its feature list.
- **Has the process being automated actually been proven by hand first?** **Automation that
  duplicates a bad process deploys the bad process everywhere.**

## Troubleshooting Checklist

0. **State intent vs. observed.** Write the one-line symptom ("the backup applet should copy
   startup-config to TFTP on `write mem`, nothing is arriving on the TFTP server") — before
   changing anything.
1. **Is the applet registered at all?** `show event manager policy registered`. If it is not
   there, the configuration never took.
2. **Has it ever fired?** `show event manager statistics policy`. Zero runs means the **event**
   is wrong, not the actions — stop looking at the action list.
3. **Is the event pattern matching?** Syslog events use **regular expressions**; test the actual
   log string against the pattern. For `event cli`, confirm the pattern matches the command as
   typed (`"write mem.*"` matches `write memory`).
4. **Actions failing immediately?** The applet **starts in exec mode** — if `enable` and
   `configure terminal` are not the first actions, every config command fails.
5. **Actions failing with authorization errors?** **AAA command authorization** is rejecting
   them. Add `event manager session cli username <username>`.
6. **Actions running in the wrong order?** **Labels sort as strings.** An applet with actions
   1.0 through 10.0 runs 10.0 *second*, right after 1.0. Renumber to 01.0 … 10.0 or avoid
   double-digit labels.
7. **An action hangs?** It is probably waiting on a **confirmation prompt** the applet cannot
   answer. `file prompt quiet` before, and restore after.
8. **Mail action failing?** `debug event manager action mail` — it filters out everything else
   so the SMTP error is visible. The chapter's own example shows
   `%HA_EM-3-FMPD_SMTP: error in connecting to SMTP server` followed by
   `%HA_EM-3-FMPD_ERROR: Error executing applet ... statement 7.0`, which names the failing
   action number.
9. **Tcl script not running?** Confirm the file is in flash and readable with **`more
   flash:<file>.tcl`**, and that the applet's action is `tclsh flash:/<file>.tcl` with the
   correct path form.
10. **Ansible: playbook will not parse?** Run it through **YAML Lint**. YAML is
    whitespace-significant and the error is usually indentation, not logic.
11. **Ansible: `unreachable=1`?** That is connectivity or credentials, not configuration — SSH
    reachability, the inventory entry, and the `username`/`password` or key. Nothing on the
    device changed.
12. **Ansible: `failed=1`?** Ansible reached the device and the module rejected the change.
    Re-run with `-vvv` to see the module's own error.
13. **Ansible: `changed=0` but you expected a change?** Either the configuration was already in
    that state (which is correct, idempotent behavior) or the task targeted the wrong `parents:`
    context.
14. **Ansible: PLAY RECAP looks clean but the device is wrong?** **PLAY RECAP is Ansible's view.**
    Verify on the device with `show startup-config | se <filter>` — and confirm a `write memory`
    task actually ran, or the change is lost on reload.
15. **Puppet: configuration keeps reverting?** That is the feature, not the bug — **Puppet
    periodically verifies configuration and can automatically put it back to the previous
    configuration.** Change the manifest, not the device.
16. **Puppet: agent cannot talk to the server?** **Communications use SSL and require
    certificates** — check certificate installation and validity before anything else. Check
    whether the **server replica** has taken over.
17. **Chef: recipe changes not taking effect?** A recipe created on the workstation **must be
    uploaded to the Chef server** with **`knife upload <cookbookname>`** before it can be used.
18. **Chef: server does not think the node needs anything?** The server decides by **comparing
    OHAI's reported node state to the cookbook or recipe.** If OHAI is not reporting, nothing
    will ever be deemed out of date.
19. **SaltStack: command reaches no minions?** Check the **target** — `'*'` versus a MinionID
    versus a glob like `Minion*`. Then confirm the **salt-minion daemon** is running.
20. **SaltStack: master not learning about a node's change?** **Beacons live on the minion** and
    notify the **reactor on the master**. A missing beacon means the master never hears about it.
21. **Salt SSH: slower than expected?** Expected. **Salt SSH is considerably slower than 0MQ**
    — that is the documented tradeoff for going agentless. It is still faster than logging in.
22. **Salt SSH: host not found?** **Roster files store connection information for hosts without
    a minion installed** — the host must be in the roster.
23. **Puppet Bolt: script ran but left nothing behind?** Also expected. **Bolt copies the script
    to a temp directory, executes it, captures results, and removes it** as if it were never
    there.

## Common Pitfalls

- **EEM applets start in exec mode.** Forgetting `enable` and `configure terminal` is the single
  most common EEM failure, and the actions fail quietly.
- **AAA command authorization silently kills applet CLI commands** unless `event manager session
  cli username <username>` is present.
- **EEM action labels are strings, not numbers.** `10.0` runs after `1.0`, not after `9.0`.
- **`event none` means the applet never self-triggers**. It runs only via `event manager run
  <name>` or another applet's `action <label> policy <name>`. An applet that "never fires"
  may be working exactly as configured. On an exam, pick **"manually or when triggered by an
  event"** over "only manually".
- **`file prompt quiet` is a global setting the applet must restore.** The chapter's backup
  applet turns it off again in action 7.0 for a reason.
- **`debug event manager all` and `debug event manager action mail` are different tools.** Use
  the mail one for SMTP problems — it filters out everything that would bury the error.
- **EEM's value is the inversion of polling**: devices **report** when something is wrong instead
  of a monitoring system continually **asking**. That is the "proactive rather than reactive"
  claim, and it is also the load reduction.
- **Chef has no agentless version.** Puppet has Puppet Bolt, SaltStack has Salt SSH, Ansible is
  natively agentless — **Chef is the N/A in Table 29-7.** This is the chapter's favorite fact.
- **SaltStack and Salt SSH are not the same answer.** **SaltStack** (master/minion) is
  **agent-based**; **Salt SSH** is the **agentless** option. A question listing both is testing
  exactly this.
- **Ansible is push by default.** `ansible-pull` is what changes it. Puppet and Chef are both
  pull. "Which model?" has no single answer for Ansible without that caveat.
- **Reactors live on the master; beacons live on the minions.** Reversing them is the easiest
  SaltStack mistake to make.
- **Pillars are master-side data the minion retrieves; grains are minion-side data reported to
  the master.** Pillars = **p**ushed-down data, grains = **g**athered-up data.
- **Grains are analogous to Chef's OHAI** — both collect node state and report it upward. The
  chapter draws this parallel explicitly.
- **Manifests are Puppet; recipes are Chef.** Modules are Puppet; cookbooks are Chef. Playbooks
  and plays are Ansible; pillars and grains are SaltStack. Table 29-7's Terminology row is the
  whole exam in one line.
- **Puppet's language is the Puppet DSL (Ruby-based), Chef's is a Ruby DSL, and both Ansible and
  SaltStack use YAML.** But **SaltStack is built on Python** and **Ansible is written in
  Python** — so "which is built on Python" (SaltStack and Ansible) and "which language do you
  write in" (YAML for both) are two different questions with two different answers.
- **Chef is written in Ruby *and Erlang*, but you write Chef code in Ruby.** The question asks
  about "the language associated with Chef" — that is Ruby.
- **Puppet Bolt is Ruby-based, like Puppet** — going agentless does not change the language.
- **Salt SSH requires Python on the remote system**, plus SSH enabled. "Agentless" does not mean
  "no prerequisites."
- **YAML is Yet Another Markup Language, not TAML.** The quiz question literally invents "TAML"
  to see if you are reading.
- **YAML's `---` and `...` are optional but common.** Their absence is not a syntax error.
- **YAML key/value pairs drop the quotation marks JSON requires** — `key: value`, not
  `"key": "value"`.
- **`ansible-playbook` runs playbooks; `ansible` runs modules.** Running `ansible
  ConfigureInterface.yaml` does not execute the playbook.
- **PLAY RECAP is Ansible's opinion, not the device's state.** `changed=3` means three tasks
  reported a change. Verify on the box.
- **`ok=4 changed=3` is not a failure** — the fourth task (`write memory`) succeeded without
  modifying the configuration, which is what saving looks like.
- **Automation duplicates bad processes faithfully.** The chapter says this about Ansible and
  then immediately notes it applies to every tool.
- **Puppet automatically reverting your manual change is the designed behavior.** Edit the
  manifest, not the running config.
- **Recipes must be uploaded with `knife upload` before the Chef server can use them.** Writing
  the file on the workstation is not deployment.
- **The Puppet Bolt command line is not the Cisco command line** — it runs on Linux, macOS
  Terminal, or Windows.
- **Salt SSH trades speed for agentlessness.** Slower than 0MQ, faster than a human.

## Exam Preparation Tasks

### Key topics coverage map

Table 29-8, mapped to where each element lives in this skill. Eight rows, and notably **six of
them are whole tool sections** — Cisco is saying the tools themselves are the testable unit, not
individual mechanisms.

| Key topic element | Description | Page | Where it lives in this skill |
|---|---|---|---|
| Paragraph | EEM applets and configuration | 894 | Key Concepts → "Embedded Event Manager (EEM)" and "EEM applets"; Procedure → "Building an EEM applet"; Config Patterns → Examples 29-1, 29-3, 29-4; Design Baseline rows 1–3; seven Common Pitfalls bullets |
| Section | Puppet | 902 | Key Concepts → "Puppet (agent-based)"; Reference Tables → Table 29-3, Table 29-4, terminology cheat sheet; Config Patterns → Examples 29-6, 29-7; Design Baseline rows 4–5; Troubleshooting steps 15–16 |
| Section | Chef | 904 | Key Concepts → "Chef (agent-based)"; Reference Tables → Table 29-4; Config Patterns → Example 29-8; Design Baseline row 6; Troubleshooting steps 17–18; three Common Pitfalls bullets |
| Section | SaltStack (agent and server mode) | 909 | Key Concepts → "SaltStack, agent and server mode"; Reference Tables → terminology cheat sheet; Config Patterns → the `salt` commands; Design Baseline row 7; Troubleshooting steps 19–20; four Common Pitfalls bullets |
| Section | Ansible | 912 | Key Concepts → "Ansible (agentless)"; Procedure → "Deploying a configuration with an Ansible playbook" and PPDIOO; Reference Tables → Tables 29-5, 29-6; Config Patterns → Examples 29-9 to 29-14; Design Baseline rows 8–11, 13; Troubleshooting steps 10–14 |
| Section | Puppet Bolt | 922 | Key Concepts → "Puppet Bolt (agentless)"; Config Patterns → the `bolt` subcommands; Troubleshooting step 23; Common Pitfalls (Ruby-based, not the Cisco CLI) |
| Section | SaltStack SSH (server-only mode) | 923 | Key Concepts → "SaltStack SSH, server-only mode"; Config Patterns → Example 29-16 roster file; Design Baseline row 14; Troubleshooting steps 21–22; Common Pitfalls (SaltStack ≠ Salt SSH, Python required) |
| Table 29-7 | High-Level Configuration Management and Automation Tool Comparison | 924 | Reference Tables → "Table 29-7", plus the derived "Agent-based vs agentless at a glance" and "Language and terminology cheat sheet" tables; Common Pitfalls (Chef is the N/A) |

**Coverage note.** **All 8 rows of Table 29-8 map to content here — no gaps, no partials.** The
whole chapter (pp. 891–925) was transcribed, including the quiz and the p. 896 answer key, and
every code example (29-1 through 29-16) is reproduced in Config Patterns from the book. The
chapter states **there are no memory tables** for it. Two Verification Commands
(`ansible-playbook --check` and `-vvv`) are **added by this skill** and marked inline; everything
else in that table comes from the chapter.

### "Do I Know This Already?" question analysis

Answer key as printed on p. 896: **1** B · **2** A, B, E · **3** C, D · **4** A, D · **5** B ·
**6** C · **7** A, B, C, D · **8** B · **9** B · **10** A · **11** B, C.

| Q | What it's really testing | Answer | Pitfall the distractors expose |
|---|---|---|---|
| 1 | Whether you accept the chapter's premise that the CLI does not scale | **B** — False | **Pure recall, no trap.** Same setup as Chapter 28's Q1. The CLI is *familiar*, not *fast*, at scale — Table 29-2's CONs list "difficult to scale" and "can execute only one command at a time" |
| 2 | Which tools are agentless | **A, B, E** — Ansible, Puppet Bolt, Salt SSH | **(c) SaltStack and (d) Chef are the traps, and they fail for different reasons.** SaltStack proper is master/**minion** — agent-based; its agentless option is **Salt SSH**, listed separately as (e). **Chef has no agentless version at all** — the N/A in Table 29-7. Promoted to Common Pitfalls |
| 3 | Ansible terminology against Puppet and Chef terminology | **C, D** — Playbooks, Tasks | **(a) Manifests is Puppet and (e) Recipes is Chef** — real terms from the wrong tool, the chapter's favorite distractor design. **(b) Modules is genuinely arguable**: Ansible tasks *are* calls to Ansible modules (Table 29-5 says so), but "modules and manifests" is Puppet's Terminology row in Table 29-7. **Answer from Table 29-7's terminology row, not from how the tools actually work** |
| 4 | Which tools are built on Python | **A, D** — Ansible, SaltStack | The near-miss is confusing **what a tool is built on** with **what you write in**. SaltStack is **built on Python** but you write **YAML**; Chef and Puppet are the Ruby pair. Both halves of that get tested — see Q6 |
| 5 | Recognizing YAML by shape | **B** — the `# HR Employee record / Employee1: / Name: …` block | **(a) is JSON** (curly braces, quoted key/value pairs) — the same discriminator Chapter 28's Q5 tested in the other direction. (c) is a bare list of names with no structure, (d) is invented bracket syntax. The tell: **`key: value` with no quotes and no braces** |
| 6 | The language associated with Chef | **C** — Ruby | **(a) Python is the trap** — it is the right answer for **Ansible and SaltStack** (Q4), one question earlier. **(e) Tcl is also real** but belongs to **EEM**, from the first section of this same chapter. Every distractor is a real language from a neighboring slot |
| 7 | What version control and code-sharing communities give you | **A, B, C, D** — version tracking, developer attribution, collaboration/sharing, speed | **(e) "real-time telemetry software database" and (f) "automatically blocking malicious code" are the invented capabilities.** Neither Puppet Forge nor GitHub does either. Note this is "choose all that apply" with **four** correct — do not stop at two |
| 8 | The PPDIOO expansion | **B** — Prepare, Plan, Design, Implement, **Operate**, Optimize | The real discriminator is **Operate vs Observe** — (a) and (d) both substitute "Observe," and (d) also reverses Prepare/Plan. **(e) swaps Implement for "Integrate."** ⚠️ **Options (b) and (c) are printed identically in the book** — a typesetting error; (c) presumably meant to differ. Learn the six words, not the letter |
| 9 | Whether you are reading carefully | **B** — False | **The trap is a single invented letter.** Everything in the sentence is true — Ansible playbooks *do* start with three dashes — except the format is **YAML**, not "TAML." A question designed to catch skimming. Promoted to Common Pitfalls |
| 10 | The command that runs a playbook | **A** — `ansible-playbook ConfigureInterface.yaml` | **(b) `ansible ConfigureInterface.yaml` is the high-value distractor** — `ansible` is a **real command** that **runs modules against targeted hosts**, not playbooks (Table 29-6). (c) and (d) invent "play ansible-book," and (d) adds the fake `.taml` extension from Q9 |
| 11 | Identical to Q2 | **⚠️ Printed key says B, C — this appears to be an error** | **Q11 is word-for-word Q2 with the same five options, but the printed answer key gives A, B, E for Q2 and B, C for Q11.** They cannot both be right. **A, B, E (Ansible, Puppet Bolt, Salt SSH) matches Table 29-7 and the chapter text**; B, C would mean Puppet Bolt and **SaltStack**, and SaltStack proper is agent-based. **Treat Q2's key as correct and Q11's as a misprint** |

**Pattern worth noting.** This quiz is built almost entirely out of **cross-tool terminology
swaps** — manifests offered for Ansible (Q3), Python offered for Chef (Q6), Tcl offered for Chef
when it belongs to EEM (Q6), SaltStack offered as agentless when Salt SSH is (Q2/Q11). **Eight of
the eleven questions are answerable from Table 29-7 alone**, which is why it carries a Key Topic
icon. The chapter also has **two printing defects worth knowing about before you sit the exam**:
**Q8's options (b) and (c) are identical**, and **Q11's answer key contradicts Q2's for the same
question**. Neither changes what you need to know — agentless means **Ansible, Puppet Bolt, Salt
SSH**, and PPDIOO's fifth word is **Operate**.
