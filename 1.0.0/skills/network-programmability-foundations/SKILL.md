---
name: ccnp-network-programmability-foundations
description: >
  Use this skill when working with network programmability and automation fundamentals —
  APIs, data formats, data models and their transport protocols, and reading or writing
  the Python that drives them. Invoke when the user asks about: network programmability,
  programmatic management, CLI pros and cons, CLI does not scale, misconfiguration, human
  error outages, API, application programming interface, Northbound API, Southbound API,
  network controller, REST, RESTful API, HTTP methods, GET, POST, PUT, PATCH, DELETE,
  OPTIONS, HEAD, CRUD, CREATE READ UPDATE DELETE, HTTP status codes, 200 OK, 201 Created,
  400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, Postman, Postman
  Builder, Postman collections, Postman history, Postman environment, API testing, DevNet
  sandbox, XML, Extensible Markup Language, start tag, end tag, JSON, JavaScript Object
  Notation, key/value pairs, curly braces, data format, indentation, Cisco DNA Center API,
  Catalyst Center API, Token API, sandboxdnac.cisco.com, basic authentication, X-Auth-Token,
  auth token, Network Device API, device inventory API, limit, offset, query parameters,
  API filters, API documentation, Cisco vManage API, SD-WAN API, Authentication API,
  j_username, j_password, x-www-form-urlencoded, JSESSIONID, Java session ID, Fabric Device
  API, data models, YANG, Yet Another Next Generation, RFC 6020, YANG module, YANG tree
  structure, container, leaf, list, choice, case, enum, enumeration, type, config false,
  configuration data vs state data, NETCONF, RFC 4741, RFC 6241, NETCONF operations, get,
  get-config, edit-config, copy-config, delete-config, datastore, capabilities, RPC, Remote
  Procedure Call, NETCONF transactions, all or nothing, SNMP vs NETCONF, OIDs vs paths,
  SMI, MIB, BER encoding, RESTCONF, RFC 8040, yang-data+json, Cisco-IOS-XE-native, Cisco
  DevNet, developer.cisco.com, DevNet Documentation, DevNet Learn, DevNet learning labs,
  DevNet Technologies, DevNet Community, DevNet ambassadors, DevNet evangelists, DevNet
  Events, DevNet Express, GitHub, version control, repository, repo, project, commit,
  commit log, branch, pull request, peer review, code sharing, README.md, Python, Python
  script, Python module, import, from import, requests module, urllib3, HTTPBasicAuth,
  PrettyTable, json module, Python dictionary, key value pair, Python function, def, print
  function, Python string, multiple-line string, triple quotes, three quotation marks,
  Python comment, hash character, condition, if statement, variable, for loop, verify=False,
  disable_warnings, InsecureRequestWarning.
---

## Purpose
This is the on-ramp to the Automation domain: why the CLI stops scaling, what an API is and
how REST exposes it, the two data formats (XML and JSON) you will read all day, the data
models (YANG) and the two protocols that carry them (NETCONF and RESTCONF), plus the
practical toolchain — Postman, DevNet, GitHub, and enough Python to read someone else's
script and know what it does.

## Key Concepts

**The CLI and why it stops scaling**
- The CLI has been **the most commonly used management method for the past 30 years**, and it
  is not going away — but it has a ceiling.
- **The biggest flaw is misconfiguration.** Businesses have frequent, sometimes extremely
  complex network changes, and when complexity rises the cost of a failure rises with it,
  because troubleshooting a complex network takes longer.
- **A majority of network outages are caused by human beings**, not by a software or hardware
  component failing. Many outages come from misconfigurations caused by a lack of network
  understanding. Not all outages can be avoided, but tooling reduces the human-error share.
- The two CONs that specifically make it unscalable — and the two the exam asks about — are
  that **the CLI is prone to human error and misconfiguration** and that **it is used on a
  device-by-device basis**.

**APIs, Northbound and Southbound**
- **APIs are mechanisms used to communicate with applications and other software**, and also
  to communicate with components of the network through software. They can **configure or
  monitor** specific components.
- The architecture (Figure 28-1) is three layers: **Applications ↔ Northbound API ↔
  Controller ↔ Southbound API ↔ Data Plane**.
- **Northbound API** — used to communicate **from a network controller to its management
  software**. When a network operator logs into a controller (Cisco DNA Center's GUI, for
  example) to manage the network, the information passed from the management software is
  leveraging a **Northbound REST-based API**. **Best practice is to encrypt that traffic with
  TLS** between the software and the controller; most API types can encrypt data in flight.
- **Southbound API** — when the operator makes a change in the controller's management
  software, those changes are **pushed down to the individual devices** using a Southbound
  API. Those devices can be routers, switches, **or even wireless access points**.
- Mnemonic that survives exam pressure: **north is up toward the apps and the humans; south is
  down toward the gear.**

**REST APIs**
- An API that uses REST is a **RESTful API**. RESTful APIs **use HTTP methods to gather and
  manipulate data**. Because HTTP has a defined structure, REST **offers a consistent way to
  interact with APIs from multiple vendors** — that consistency is the whole point.
- HTTP functions are similar to the functions most applications or databases use to store or
  alter data, whether the data lives in a database or in the application. Those are the
  **CRUD** functions: **CREATE, READ, UPDATE, DELETE**. In a SQL database the CRUD functions
  are what interact with and manipulate the stored data.
- **An API call using REST is very much like an HTTP transaction.** Each API call in a RESTful
  API **maps to an individual URL for a particular function** — so **every configuration
  change or poll to retrieve data has a unique URL**, whether it is a GET, POST, PUT, PATCH,
  or DELETE.

**API tools — Postman**
- Testing is one of the most important pieces of interacting with any software using APIs; it
  ensures the code accomplishes the outcome that was intended.
- **APIs are software interfaces into an application or a controller, and many require
  authentication** — such an API is just like any other device a user must authenticate to.
  **A developer who is authenticated can make changes that impact the application. If a REST
  API call is used to delete data, that data is removed from the application or controller
  just as if a user had logged in via the CLI and deleted it.**
- **Best practice: use a test lab or the Cisco DevNet sandbox while learning or practicing**,
  to avoid accidental impact to a production or lab environment.
- **Postman** is an application that makes it possible to **interact with APIs using a
  console-based approach**, supporting various data types and formats against REST-based APIs.
- The **Builder** portion is where the work happens; the four areas that need the most focus:
  - **History** — a list of all recent API calls made using Postman. **Clear All** at the top
    of the Collection window wipes the whole list; hovering over a single call and clicking
    the **trash can** icon removes just that one.
  - **Collections** — API calls stored in **groups** that fit the user's own structure. Any
    naming convention, displayed as a **folder hierarchy** — for example a collection named
    `DNA-C` holding all the Cisco DNA Center calls. Useful during testing because calls can be
    found and sorted. A collection can be marked a **favorite** with the star icon.
  - **New Tab** — each tab holds **its own API call and parameters, completely independent of
    any other tab**. One tab can talk to Cisco DNA Center while another talks to a Cisco Nexus
    switch.
  - **URL bar** — each tab has its own, because each API call maps to its own unique URL.

**Data formats — XML and JSON**
- **XML (Extensible Markup Language)** — the same format commonly used when constructing web
  services. It is a **tag-based language**: a tag **must begin with `<` and end with `>`**, so
  a start tag named interface is `<interface>`. **A section that is started must also be
  ended**, and the **end tag is the same string preceded by `/`** — `</interface>`. Inside the
  start and end tags you can use different code and parameters.
- **A key feature of XML is that it is readable by both humans and applications, and
  indentation is part of what makes it readable. Indentation is not required, but it is a
  recommended best practice for legibility** — the chapter makes the point by showing the same
  data with and without it (Examples 28-1 and 28-2).
- **JSON (JavaScript Object Notation)** — newer than XML but **taking the industry by storm**,
  and some say it will soon replace XML. It is arguably **much easier to work with**: simple
  to read and create, with a much cleaner structure.
  - **JSON stores all its information in key/value pairs.**
  - **JSON uses objects for its format. Each JSON object starts with a `{` and ends with a
    `}`** — curly braces.
  - As with XML it is easier to read indented, but **even without indentation JSON is
    extremely easy to read.**

**HTTP status codes**
- The response code is the first thing to read on any API call — it separates "the network is
  broken" from "you are not allowed to do that."
- `2xx` means it worked; `4xx` means the **client** did something wrong, and which `4xx` you
  get tells you whether it was the request, the credentials, or the permissions.

**Cisco DNA Center APIs**
- **The Cisco DNA Center controller expects all incoming data from the REST API to be in JSON
  format.**
- **The HTTP POST function is used to send the credentials** to the controller. Cisco DNA
  Center uses **basic authentication** to pass a username and password to the **Token API**,
  which authenticates a user so they can make additional API calls. Just as when logging into
  a device via the CLI, a properly secured system prompts for credentials — the same applies
  to using an API to authenticate to software.
- **The token** you receive is a long string. **Think of it as a hash generated from the
  supplied login credentials. The token changes every time an authentication is made.** The
  token is **usable only for the current authenticated session**; another user authenticating
  via the Token API receives their own unique token.
- **The Network Device API** retrieves the list of devices currently in the inventory managed
  by the controller. The payload per device is large — type, family, macAddress, bootDateTime,
  collectionStatus, interfaceCount, lineCardCount, managementIpAddress, memorySize,
  platformId, reachabilityStatus, series, role, hostname, upTime, softwareVersion,
  softwareType, serialNumber, instanceUuid, id, and more.
- The payoff the chapter is selling: **in the time it takes someone to log in to one device
  and issue all the relevant `show` commands, an API call can gather that data for the entire
  network.**
- **Filters and offsets** — when using APIs it is common to manipulate data with filters and
  offsets, and **this is where the API documentation becomes so valuable**. In Postman you can
  modify the Network Device API URL:
  - **`?limit=1`** on the end of the URL shows only a **single device** — "the user only wants
    to retrieve one record from the inventory."
  - **`&offset=2`** states that the one record returned should be **the second record** in the
    inventory.
  - These are **query parameters that are part of the API**, and they can be invoked from a
    client like Postman or from code.

**Cisco vManage (SD-WAN) APIs**
- Similar in spirit to the DNA Center APIs, but **the steps for connecting are different**.
  You still must provide login credentials before using any other call.
- The four things that must be right to authenticate:
  - The URL bar must have the API call targeting the **Authentication API**.
  - **The HTTP POST operation** is used to send the username and password to vManage.
  - **The Headers Content-Type key must be `application/x-www-form-urlencoded`** — *not*
    `application/json`, which is what DNA Center wants. This is the single most testable
    difference between the two.
  - The body must contain keys with the **`j_username`** `devnetuser` and the **`j_password`**
    `Cisco123!`.
- **The response delivers a Java session ID, displayed as `JSESSIONID`** — the vManage
  equivalent of the DNA Center token. **This session ID is passed to vManage for all future API
  calls for this user.** HTTP **200 OK** indicates a successful POST.
- The **Fabric Device API** (HTTP GET) returns an inventory of fabric devices in JSON, under
  the **`data`** key: **Device ID, System IP, Host name, Reachability, Status, Device type,
  Site ID**.
- Setup references: Cisco SD-WAN Postman environment steps at
  `https://developer.cisco.com/sdwan/`; Cisco DNA Center at
  `https://developer.cisco.com/learning/tracks/dnacenter-programmability/`. A Postman
  environment can be **downloaded from DevNet** so all the authentication details are
  pre-populated.

**YANG data models**
- **SNMP is widely used for fault handling and monitoring, but it is not often used for
  configuration changes** — CLI scripting is used more often than other methods.
- **YANG data models are an alternative to SNMP MIBs and are becoming the standard for data
  definition languages. YANG is defined in RFC 6020.**
- **Data models describe** whatever can be **configured** on a device, everything that can be
  **monitored** on a device, and **all the administrative actions** that can be executed on a
  device — such as resetting counters or rebooting it — **including all the notifications the
  device is capable of generating.** All of those variables can be represented in a YANG model.
- **Data models create a uniform way to describe data, which is beneficial across vendors'
  platforms**, and let operators configure, monitor, and interact with network devices
  **holistically across the entire enterprise environment**.
- Structure:
  - **YANG models use a tree structure.** Within it, models are **similar in format to XML**
    and are **constructed in modules**. Modules are **hierarchical** and contain all the
    different data and types that make up a YANG device model.
  - **YANG models make a clear distinction between configuration data and state
    information.** The tree structure represents **how to reach a specific element**, and
    elements can be **either configurable or not configurable**.
  - **Every element has a defined type.** An interface can be configured on or off, but the
    **operational** interface state cannot be changed — if the options are only up or down, it
    is either up or down and nothing else is possible.
  - **`config false` marks a leaf as non-configurable.** In the chapter's network example,
    `observed-speed` carries `config false` because it is the **auto-detected** value, not a
    configurable one — while `speed` is configurable with `enum 10m / 100m / auto`.

**NETCONF**
- **NETCONF, defined in RFC 4741 and RFC 6241, is an IETF standard protocol that uses the YANG
  data models to communicate with the various devices on the network.**
- **NETCONF runs over SSH, TLS, and — although not common — SOAP (Simple Object Access
  Protocol).**
- Two key differentiators from SNMP:
  - **SNMP cannot distinguish between configuration data and operational data; NETCONF can.**
    The chapter calls this one of the most important differences.
  - **NETCONF uses paths to describe resources, whereas SNMP uses OIDs.** A NETCONF path looks
    like `interfaces/interface/eth0` — far more descriptive than an OID.
- Common use cases: collecting the status of specific fields; changing the configuration of
  specific fields; taking administrative actions; sending event notifications; backing up and
  restoring configurations; **testing configurations before finalizing the transaction**.
- **Transactions are all or nothing.** There is **no order of operations or sequencing within a
  transaction** — no part of the configuration is done first; **the configuration is deployed
  all at the same time**. Transactions are **processed in the same order every time on every
  device**. When deployed they **run in a parallel state and do not impact each other**:
  parallel transactions touching different areas of a device's configuration do not overwrite
  or interfere with each other, and they do not impact each other if the same transaction is
  run against multiple devices.
- **NETCONF exchanges information called capabilities when the TCP connection has been made.
  Capabilities tell the client what the device it is connected to can do.**
- **Information and configurations are stored in datastores**, which are manipulated with the
  NETCONF operations. **NETCONF uses Remote Procedure Call (RPC) messages in XML format** to
  send information between hosts.

**RESTCONF**
- **RESTCONF, defined in RFC 8040**, is used to programmatically interface with data defined in
  **YANG models** while also using the **datastore concepts defined in NETCONF**.
- **There is a common misconception that RESTCONF is meant to replace NETCONF — this is not the
  case.** Both are very common methods for programmability and data manipulation. **RESTCONF
  uses the same YANG models as NETCONF and Cisco IOS XE.**
- The goal is to **provide a RESTful API experience while still leveraging the device
  abstraction capabilities provided by NETCONF**.
- **RESTCONF supports these HTTP methods and CRUD operations: GET, POST, PUT, DELETE,
  OPTIONS.** Note **OPTIONS is in the RESTCONF list but PATCH is not** in the chapter's list,
  which inverts the REST table earlier in the chapter.
- **RESTCONF requests and responses can use either JSON or XML** structured data formats.

**Cisco DevNet**
- Everything in this chapter is available for use and practice at **Cisco DevNet,
  `http://developer.cisco.com`**. Five menu options across the top:
  - **Documentation** — a single place to get **API documentation** for solutions such as Cisco
    DNA Center, Cisco SD-WAN, IoT, and Collaboration, including how to programmatically
    interact with them. **A great place to start** when learning to interact with devices and
    software-defined controllers.
  - **Learn** — where you navigate DevNet's offerings, with subsections for **guided learning
    tracks** that walk through technologies and their API labs — Programming the Cisco Digital
    Network Architecture (DNA), ACI Programmability, Getting Started with Cisco WebEx Teams
    APIs, Introduction to DevNet. **The site tracks your progress** through a module so you can
    leave and resume, which helps across multiple days or weeks.
  - **Technologies** — pick relevant content **based on the technology** you want to study and
    dive straight into the associated labs and training.
  - **Community** — **perhaps one of the most important sections.** Access to people at various
    stages of learning; **DevNet ambassadors and evangelists** are available to help. Latest
    events and news, blogs, developer forums, social media. **A safe zone for asking questions,
    simple or complex** — the place to start for all things Cisco and network programmability.
  - **Events** — all past and future events, including **DevNet Express** events and conferences
    where DevNet will be presenting.

**GitHub**
- **GitHub is a hosted web-based repository for code**, and one of the most efficient and
  commonly adopted ways of using **version control**. It also has capabilities for **bug
  tracking and task management**.
- It is one of the easiest ways to **track changes in your files, collaborate with other
  developers, and share code with the online community** — and a great place to look for code
  to get started on programmability, because other engineers are often solving similar problems
  and have already written and tested the code.
- **One of the most powerful features is the ability to rate and provide feedback on other
  developers' code. Peer review is encouraged in the coding community.**
- **Projects are repositories that contain code files.** GitHub gives a single pane to create,
  edit, and share them, plus **a summary of commit logs** on the main repository page whenever
  a file is saved or created.
- GitHub provides a guide covering **how to create a repository, start a branch, add comments,
  and open a pull request.**
- The **pencil** icon opens editing mode, which works like any text editor — type directly or
  paste code in. Once a file is in the repository, **other users can contribute, add, or delete
  lines based on the original code** — "this is the true power of sharing code."

**Basic Python components**
- **Python has by a longshot become one of the most common programming languages in terms of
  network programmability**, and **it is one of the easier languages to get started with and
  interpret.** (The quiz's first question exists purely to make you say this out loud.)
- The building blocks the chapter names, in the order it introduces them:
  - **String** — **one or more alphanumeric characters**; can comprise many numbers or letters
    depending on the Python version in use.
  - **Multiple-line string** — **three quotation marks in a row begin and end a multiple-line
    string.** Scripts often use one at the top to hold overall comments and licensing text.
  - **Comment** — **the `#` character indicates a comment.** Comments usually describe the
    **intent** of an action, and **often appear right above the action they describe**. Some
    scripts comment every action; some are barely documented at all.
  - **Variable** — a named value, such as `ENVIRONMENT_IN_USE = "sandbox"`.
  - **Dictionary** — **the structure used to hold key/value pairs**, named in the chapter's
    script `dnac`. **A dictionary contains multiple key/value pairs and starts and ends with
    curly braces `{}`.** The resemblance to JSON is the point — JSON also uses key/value pairs.
    **Dictionaries can be written multi-line (readable) or as a single line** — same object.
  - **Condition** — **a logical `if` question is asked, and depending on the answer, an action
    happens.** `if ENVIRONMENT_IN_USE == "sandbox":` is the chapter's example.
  - **Module** — **a collection of actions and instructions.** The first section of a script
    tells the interpreter which modules it will use. **Modules help Python understand what it
    is capable of** — without the Requests module imported, it would be difficult for Python to
    interpret an HTTP GET. Other ways of doing HTTP calls exist, but Requests greatly simplifies
    the process.
  - **Function** — **blocks of code built to perform specific actions. Functions are very
    structured in nature and can often be reused later within a script. Some functions are
    built into Python and do not have to be created** — **`print` is the chapter's example of a
    built-in function.** User-defined ones start with `def`.
- The three lab-environment options in `Env_Lab.py`:
  - **`sandbox`** — the DevNet **always-on and reserved sandboxes** accessed through
    `http://developer.cisco.com`. This is what the chapter uses throughout.
  - **`express`** — the back end used for **DevNet Express Events** held globally at various
    locations and Cisco offices.
  - **`custom`** — used when **a Cisco DNA Center is already installed** in a lab or another
    facility and needs to be accessed by the script.

**Securing JSON with JSON Web Tokens (JWT):**
- **JWT is an IETF open standard defined in RFC 7519.** Its purpose is **secure transmission
  of JSON-formatted information between parties**. JWTs **can be encrypted** (confidentiality)
  and **digitally signed** (integrity); a JWT **signed using PKI also provides
  nonrepudiation**.
- **A completed JWT is three parts separated by dots: `header.payload.signature`.** Each part
  is **Base64URL-encoded** before the dots are added. **The dot delimiters carry no
  information** — they are pure separators.
- **Header — this is the component that defines the signing algorithm.** A JSON object with
  the **token type** and the **algorithm used to sign the JWT**:
  ```json
  { "alg": "RS256", "typ": "JWT" }
  ```
- **Payload — a JSON object containing claims.** A claim carries information about the sender
  and the information being transmitted. **Three types of claim:**
  - **Registered** — predefined **three-character** names. **Not mandatory, but
    recommended.** `exp` (expiration time), `iss` (issuer), `sub` (subject), `aud`
    (audience).
  - **Public** — defined at will by the JWT generator, but **RFC 7519 recommends registering
    new names with IANA or using a Public Name** (a value unlikely to collide with others').
    IANA-listed examples: `name`, `given_name`, `middle_name`, `nickname`, registered to the
    OpenID Foundation Artifact Binding Working Group.
  - **Private** — **custom-created by the generator, not registered with IANA**, so it
    **risks colliding** with a claim already in use elsewhere.
- **Signature — the product of signing, not the declaration of how.** It is built from the
  Base64URL-encoded header and payload joined by a period, then signed with a **secret key**
  using **the algorithm named in the header**:
  ```
  RSASHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
  ```
- **Exam framing:** asked which JWT component *defines* the signing algorithm, the answer is
  the **header**. The signature merely *uses* that algorithm; the payload and the delimiter
  are unrelated to it.

## Procedure

**Authenticating to Cisco DNA Center with Postman (Token API), 7 steps:**
1. In the URL bar, enter **`https://sandboxdnac.cisco.com/api/system/v1/auth/token`** to target
   the Token API.
2. Select the **HTTP POST** operation from the dropdown box.
3. Under the **Authorization** tab, ensure the type is set to **Basic Auth**.
4. Enter **`devnetuser`** as the username and **`Cisco123!`** as the password.
5. Select the **Headers** tab and enter **`Content-Type`** as the key.
6. Select **`application/json`** as the value.
7. Click **Send** to pass the credentials to the Cisco DNA Center controller via the Token API.

> A successful call returns **200 OK** and a token string. That token is good only for this
> authenticated session, and a new one is issued on every authentication.

**Retrieving the device inventory with the Network Device API, 8 steps:**
1. **Copy the token** you received earlier and click a **new tab** in Postman.
2. In the URL bar enter **`https://sandboxdnac.cisco.com/api/v1/network-device`** to target the
   Network Device API.
3. Select the **HTTP GET** operation from the dropdown box.
4. Select the **Headers** tab and enter **`Content-Type`** as the key.
5. Select **`application/json`** as the value.
6. Add another key and enter **`X-Auth-Token`**.
7. **Paste the token in as the value.**
8. Click **Send** to pass the token to the controller and perform an HTTP GET to retrieve the
   device inventory list.

**Narrowing the result with query parameters:**
1. Consult the **API documentation** for the parameters that API supports — this is the step
   people skip.
2. Append **`?limit=1`** to the Network Device API URL to return **only one record**.
3. Append **`&offset=2`** to state that the one record returned should be **the second record**
   in the inventory.
4. Full form: `https://sandboxdnac.cisco.com/api/v1/network-device?limit=1&offset=2`.

**Authenticating to Cisco vManage (SD-WAN):**
1. Target the **Authentication API** in the URL bar.
2. Use the **HTTP POST** operation to send the username and password.
3. Set the Headers **Content-Type** key to **`application/x-www-form-urlencoded`** — **not**
   `application/json`.
4. Put **`j_username`** = `devnetuser` and **`j_password`** = `Cisco123!` in the **body**.
5. Send. A **200 OK** and a **`JSESSIONID`** come back; that session ID is passed on all future
   API calls for this user.

**Reading an unfamiliar Python script (the chapter's five-section method):**
1. **Modules** — read the imports first. They tell you what the script is capable of: `requests`
   means HTTP, `json` means key/value data, `HTTPBasicAuth` means basic auth, `PrettyTable`
   means formatted table output. `#! /usr/bin/env python3` specifies the Python version.
2. **Setup** — the objects and constants built before any function runs: the PrettyTable columns,
   the warning suppression, the `headers` dictionary with an empty `x-auth-token` waiting to be
   filled.
3. **Authentication function** — find the `def` that gets a token or session ID. Here
   `dnac_login()` POSTs to the Token API and returns the token from the JSON response.
4. **Work function** — the `def` that does the actual job. Here `network_device_list()` injects
   the token into `headers["x-auth-token"]`, GETs the Network Device API, and loops over
   `data['response']` adding a row per device.
5. **Execution** — the unindented lines at the bottom that actually call everything in order,
   ending in `print()`. **Read this section second, right after the imports** — it tells you
   what the script does in three lines.

## Reference Tables

**Table 28-2 — CLI PROs and CONs**

| PROs | CONs |
|---|---|
| Well known and documented | **Difficult to scale** |
| Commonly used method | Large number of commands |
| Commands can be scripted | Must know IOS command syntax |
| Syntax help available on each command | Executing commands can be slow |
| Connection to CLI can be encrypted (using SSH) | Not intuitive |
| | **Can execute only one command at a time** |
| | CLI and commands can change between software versions and platforms |
| | Using the CLI can pose a security threat if using Telnet (plaintext) |

**Table 28-3 — HTTP functions and use cases**

| HTTP Function | Action | Use Case |
|---|---|---|
| **GET** | Requests data from a destination | Viewing a website |
| **POST** | Submits data to a specific destination | Submitting login credentials |
| **PUT** | **Replaces** data in a specific destination | Updating an NTP server |
| **PATCH** | **Appends** data to a specific destination | Adding an NTP server |
| **DELETE** | Removes data from a specific destination | Removing an NTP server |

> **PUT replaces, PATCH appends.** That one-word difference is the whole distinction, and the
> NTP examples are the clearest way to hold it: *updating* the NTP server vs *adding* one.

**Table 28-4 — CRUD functions and use cases**

| CRUD Function | Action | Use Case |
|---|---|---|
| **CREATE** | Inserts data in a database or application | Updating a customer's home address in a database |
| **READ** | Retrieves data from a database or application | Pulling up a customer's home address from a database |
| **UPDATE** | Modifies or replaces data in a database or application | Changing a street address stored in a database |
| **DELETE** | Removes data from a database or application | Removing a customer from a database |

**Table 28-5 — HTTP status codes**

| Code | Result | Common reason for response code |
|---|---|---|
| **200** | OK | Using GET or POST to exchange data with an API |
| **201** | Created | Creating resources by using a REST API call |
| **400** | Bad Request | Request failed due to **client-side issue** |
| **401** | **Unauthorized** | **Client not authenticated** to access site or API call |
| **403** | **Forbidden** | **Access not granted based on supplied credentials** |
| **404** | Not Found | Page at HTTP URL location does not exist or is hidden |

> **401 vs 403 is the pair that gets missed.** 401 = *we do not know who you are* (not
> authenticated). 403 = *we know who you are and you still cannot* (authenticated, not
> authorized).

**XML vs JSON**

| | XML | JSON |
|---|---|---|
| Full name | Extensible Markup Language | JavaScript Object Notation |
| Structure | **Tag-based** — `<tag>` … `</tag>` | **Key/value pairs** inside **objects** |
| Delimiters | Tags begin `<` end `>`; end tag adds `/` | Object starts `{` ends `}` (curly braces) |
| Every opened section | **Must be closed** | Closed by the matching brace |
| Indentation | **Not required, recommended best practice** for legibility | Easier read indented, but **extremely easy to read even without** |
| Age / trajectory | Longer established; also used for web services | Newer, **"taking the industry by storm"** — some say it will replace XML |
| Used by | NETCONF (RPC messages are XML), RESTCONF | DNA Center REST API (**required**), vManage responses, RESTCONF |

**Table 28-6 — Differences between SNMP and NETCONF**

| Feature | SNMP | NETCONF |
|---|---|---|
| Resources | **OIDs** | **Paths** |
| Data models | Defined in **MIBs** | **YANG core models** |
| Data modeling language | **SMI** | **YANG** |
| Management operations | SNMP | NETCONF |
| Encoding | **BER** | **either XML or JSON** |
| Transport stack | **UDP** | **SSH/TCP** |

**Table 28-7 — NETCONF operations**

| Operation | Description |
|---|---|
| `<get>` | Requests **running configuration and state information** of the device |
| `<get-config>` | Requests some or all of the configuration **from a datastore** |
| `<edit-config>` | Edits a configuration datastore **by using CRUD operations** |
| `<copy-config>` | Copies the configuration **to another datastore** |
| `<delete-config>` | Deletes the configuration |

**NETCONF vs RESTCONF**

| | NETCONF | RESTCONF |
|---|---|---|
| RFC | **4741 and 6241** | **8040** |
| Data models | YANG | **The same YANG models** as NETCONF and Cisco IOS XE |
| Datastores | Defines them | **Uses the datastore concepts defined in NETCONF** |
| Transport | SSH, TLS, (rarely) SOAP | HTTP(S) |
| Operations | `<get>`, `<get-config>`, `<edit-config>`, `<copy-config>`, `<delete-config>` | **GET, POST, PUT, DELETE, OPTIONS** |
| Encoding | XML RPC messages | **JSON or XML** |
| Relationship | — | **Does NOT replace NETCONF** — both are common |

**Cisco DNA Center vs Cisco vManage API authentication**

| | Cisco DNA Center | Cisco vManage (SD-WAN) |
|---|---|---|
| API targeted | **Token API** (`/api/system/v1/auth/token`) | **Authentication API** |
| HTTP method | **POST** | **POST** |
| Auth method | **Basic authentication** | Credentials in the **body** |
| Content-Type | **`application/json`** | **`application/x-www-form-urlencoded`** |
| Credential fields | Username/password in Basic Auth | **`j_username`** / **`j_password`** |
| What you get back | A **token** (unique per authenticated session) | A **Java session ID — `JSESSIONID`** |
| Passed on later calls as | **`X-Auth-Token`** header | The session ID |

**DevNet menu pages**

| Page | What it is for |
|---|---|
| **Documentation** | **API documentation** — DNA Center, SD-WAN, IoT, Collaboration. Where to start for programmatic interaction |
| **Learn** | Guided **learning tracks and labs**; **progress is tracked** so you can resume |
| **Technologies** | Pick content **by technology**, dive into its labs and training |
| **Community** | Ambassadors and evangelists, blogs, forums, social. **A safe zone for asking questions** |
| **Events** | Past and future events, **DevNet Express**, conferences |

## Config Patterns

> **Provenance:** every block below is **transcribed from the chapter's own examples**
> (28-1 through 28-21), with example numbers noted. They are book-accurate. **Two of them
> contain patterns you should not carry into production** — see the annotations and the
> Design Baseline.

**XML — the same data, indented and not (Examples 28-1, 28-2)**
```xml
<users>
   <user>
     <name>root</name>
   </user>
   <user>
     <name>Jason</name>
   </user>
   <user>
     <name>Jamie</name>
   </user>
   <user>
     <name>Luke</name>
   </user>
</users>
```
```xml
<interfaces>
<interface>
<name>GigabitEthernet1</name>
</interface>
<interface>
<name>GigabitEthernet11</name>
</interface>
<interface>
<name>Loopback100</name>
</interface>
<interface>
<name>Loopback101</name>
</interface>
</interfaces>
```

**JSON — the same four users as key/value pairs (Example 28-3)**
```json
{
  "user": "root",
  "father": "Jason",
  "mother": "Jamie",
  "friend": "Luke"
}
```

**YANG — the RFC 6020 "food" model and a network-oriented one (Examples 28-5, 28-6)**
```yang
container food {
  choice snack {
      case sports-arena {
          leaf pretzel {
              type empty;
          }
          leaf popcorn {
              type empty;
          }
      }
      case late-night {
          leaf chocolate {
              type enumeration {
                  enum dark;
                  enum milk;
                  enum first-available;
              }
          }
      }
  }
}
```
```yang
list interface {
    key "name";

    leaf name {
        type string;
    }
    leaf speed {
        type enumeration {
            enum 10m;
            enum 100m;
            enum auto;
        }
    }
    leaf observed-speed {
        type uint32;
        config false;
    }
}
```
> Read the second one as: a list of interfaces; each has a configurable `speed` of 10m, 100m,
> or auto; and an `observed-speed` that **`config false`** makes read-only, because it is the
> auto-detected value rather than a setting.

**NETCONF — an RPC reply and a save-config RPC (Examples 28-7, 28-10)**
```xml
<rpc-reply message-id="101"
      xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <data>
    <top xmlns="http://example.com/schema/1.2/config">
      <users>
        <user>
          <name>Dave</name>
        </user>
        <user>
          <name>Rafael</name>
        </user>
        <user>
          <name>Dirk</name>
        </user>
      </users>
    </top>
  </data>
</rpc-reply>
```
```xml
<?xml version="1.0" encoding="utf-8"?>
<rpc xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" message-id="">
  <cisco-ia:save-config xmlns:cisco-ia="http://cisco.com/yang/cisco-ia"/>
</rpc>
```

**NETCONF — OSPF configuration of an IOS XE device (Example 28-9, abridged)**
```xml
<rpc-reply message-id="urn:uuid:0e2c04cf-9119-4e6a-8c05-238ee7f25208"
xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <data>
    <native xmlns="http://cisco.com/ns/yang/ned/ios">
      <router>
        <ospf>
          <id>100</id>
          <redistribute>
            <connected>
              <redist-options>
                <subnets/>
              </redist-options>
            </connected>
          </redistribute>
          <network>
            <ip>10.10.0.0</ip>
            <mask>0.0.255.255</mask>
            <area>0</area>
          </network>
        </ospf>
      </router>
    </native>
  </data>
</rpc-reply>
```
> This is the same `router ospf 100` configuration you would read in the CLI — just structured
> as XML. That equivalence is the point of the example.

**RESTCONF — GET the configured logging severity (Example 28-11)**
```
RESTCONF GET
------------------------
URL: https://10.85.116.59:443/restconf/data/Cisco-IOS-XE-native:native/logging/
monitor/severity

Headers: {'Accept-Encoding': 'gzip, deflate',  'Accept': 'application/
yang-data+json, application/yang-data.errors+json'}

Body:

RESTCONF RESPONSE
----------------------------
200
{
  "Cisco-IOS-XE-native:severity": "critical"
}
```

**Python — the environment file (Example 28-12, abridged; Examples 28-13 to 28-15)**
```python
"""Set the Environment Information Needed to Access Your Lab!
...
"""

# User Input

# Please select the lab environment that you will be using today
#     sandbox - Cisco DevNet Always-On / Reserved Sandboxes
#     express - Cisco DevNet Express Lab Backend
#     custom  - Your Own "Custom" Lab Backend
ENVIRONMENT_IN_USE = "sandbox"

# Set the 'Environment Variables' based on the lab environment in use
if ENVIRONMENT_IN_USE == "sandbox":
    dnac = {
        "host": "sandboxdnac.cisco.com",
        "port": 443,
        "username": "devnetuser",
        "password": "Cisco123!"
    }
```
```python
# The same dictionary written as a single line (Example 28-14)
dnac = {"host": "sandboxdnac.cisco.com", "port": 443, "username": "devnetuser", "password": "Cisco123!"}
```
> **Hardcoded credentials.** These are published DevNet sandbox credentials so nothing is at
> risk here, but the *pattern* is the thing to unlearn — real credentials belong in environment
> variables or a gitignored secrets file, never in a script that goes to a repo.

**Python — the full `get_dnac_devices.py` script (Example 28-16)**
```python
#! /usr/bin/env python3

from env_lab import dnac
import json
import requests
import urllib3
from requests.auth import HTTPBasicAuth
from prettytable import PrettyTable

dnac_devices = PrettyTable(['Hostname','Platform Id','Software Type','Software Version','Up Time' ])
dnac_devices.padding_width = 1

# Silence the insecure warning due to SSL Certificate
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

headers = {
            'content-type': "application/json",
            'x-auth-token': ""
        }

def dnac_login(host, username, password):
    url = "https://{}/api/system/v1/auth/token".format(host)
    response = requests.request("POST", url, auth=HTTPBasicAuth(username, password),
                                headers=headers, verify=False)
    return response.json()["Token"]

def network_device_list(dnac, token):
    url = "https://{}/api/v1/network-device".format(dnac['host'])
    headers["x-auth-token"] = token
    response = requests.get(url, headers=headers, verify=False)
    data = response.json()
    for item in data['response']:
        dnac_devices.add_row([item["hostname"],item["platformId"],item["softwareType"],
        item["softwareVersion"],item["upTime"]])

login = dnac_login(dnac["host"], dnac["username"], dnac["password"])
network_device_list(dnac, login)

print(dnac_devices)
```
> Two things to flag. **`verify=False` disables TLS certificate validation**, and
> `urllib3.disable_warnings(...)` hides the warning that says so — acceptable against a sandbox
> with a self-signed certificate, **not acceptable against production**, where it removes the
> protection TLS was there to provide. Also note **Example 28-16 returns
> `response.json()["Token"]` while Example 28-19 returns `response.json()["token"]`** —
> one of them is wrong, and JSON keys are case-sensitive.

## Design Baseline

Rows sourced to the ENCOR 350-401 OCG Chapter 28 with page numbers. Two rows are marked as
**observations by this skill** rather than chapter recommendations — the chapter's code does
these things without comment, and they would not survive a code review. **A deviation is a
question for the operator, not automatically a finding.**

| Baseline practice | Why | Legitimate reasons to deviate | Source |
|---|---|---|---|
| **Practice against a test lab or the Cisco DevNet sandbox, never production** | **An API DELETE removes the data exactly as if you had logged into the CLI and deleted it.** There is no "are you sure" between you and a controller's inventory | None while learning. Production API work happens against production — but deliberately, with change control, not while exploring | Ch. 28, p. 857 |
| **Encrypt Northbound API traffic with TLS** between the management software and the controller | It carries credentials and full device inventory. Most API types can encrypt data in flight, so there is no reason not to | None stated | Ch. 28, p. 855 |
| **Do not manage devices over Telnet** | Table 28-2 names it directly: using the CLI **poses a security threat if using Telnet (plaintext)**. SSH is listed as the PRO that fixes it | None — see the `network-device-access-control` skill for the SSHv2 baseline | Ch. 28, p. 855 (Table 28-2) |
| **Indent XML and JSON** even though neither requires it | **Recommended best practice from a legibility perspective** — the chapter proves it by printing the same interface list unindented and letting you compare | Machine-to-machine payloads where size matters; humans will not read those anyway | Ch. 28, p. 861 |
| **Read the API documentation before pulling data** — use the query parameters the API provides | "**This is where the API documentation becomes so valuable.**" `?limit=` and `&offset=` exist so you do not have to pull and parse an entire inventory to look at one device | Small inventories where the full pull is cheap | Ch. 28, p. 866 |
| **Prefer YANG/NETCONF over SNMP for configuration changes** | **SNMP is widely used for fault handling and monitoring but is not often used for configuration changes**, and it **cannot distinguish configuration data from operational data** — NETCONF can | Monitoring and fault handling, which is what SNMP is genuinely good at; platforms without NETCONF support | Ch. 28, pp. 870, 872 |
| **Use NETCONF's test-before-commit rather than pushing blind** | "**Testing configurations before finalizing the transaction**" is a listed NETCONF use case, and transactions are **all or nothing** — a bad transaction fails whole, but only if you let it be tested | None stated | Ch. 28, p. 872 |
| **Keep code in version control and invite peer review** | GitHub tracks changes, enables collaboration, and **the ability to rate and provide feedback on other developers' code is called out as one of its most powerful features. Peer review is encouraged in the coding community** | None | Ch. 28, p. 880 |
| **Do not disable TLS certificate validation** (`verify=False`) or suppress the warning that reports it | It removes the authentication half of TLS — you get encryption to *somebody*, with no proof of who. Suppressing `InsecureRequestWarning` hides the only signal that this is happening | A sandbox or lab with a self-signed certificate, which is exactly the chapter's case. Production needs a real or internally trusted CA | **Observation by this skill** — Ch. 28 Examples 28-16 and 28-19 do this **without flagging it** |
| **Do not hardcode credentials in scripts** — use environment variables or a gitignored secrets file | `Env_Lab.py` is published sandbox credentials so nothing is exposed there, but the pattern reproduced with real credentials puts them in git history permanently | None for real credentials. Published demo credentials in published demo code are fine | **Observation by this skill** — Ch. 28 Examples 28-12 to 28-14 |

## Verification Commands

> Chapter 28 has no device CLI. The checks below are the API- and code-side equivalents — how
> to confirm each concept on a real system. The `curl` and Python forms are **added by this
> skill** as terminal equivalents of the chapter's Postman steps; the URLs and headers are the
> chapter's own.

| Command / check | What to look for |
|---|---|
| Postman response pane, top right | **Status code, time, and size.** The chapter's successful POST shows `200 OK` in 980 ms — read the code before reading the body |
| `curl -k -X POST -u devnetuser:'Cisco123!' -H "Content-Type: application/json" https://sandboxdnac.cisco.com/api/system/v1/auth/token` | The **token** in the JSON response. A `401` here means credentials; a `403` means credentials were fine but access is not granted |
| `curl -k -H "X-Auth-Token: <token>" -H "Content-Type: application/json" https://sandboxdnac.cisco.com/api/v1/network-device` | The device inventory under the **`response`** key. `401` here means the token is missing, mistyped, or from an expired session |
| Append `?limit=1&offset=2` to the same URL | **Exactly one device, the second in the inventory** — confirms the query parameters are being honored |
| `python3 -c "import json,sys; json.load(sys.stdin)" < file.json` | **Validates JSON** — silence means well-formed, a traceback names the line and column that broke |
| `python3 -m json.tool file.json` | Pretty-prints and validates in one step — the fastest way to make an unindented API response readable |
| `xmllint --noout file.xml` | **Validates XML** — catches the unclosed tag, which is the single most common XML error |
| `xmllint --format file.xml` | Re-indents XML so you can actually read it |
| `ssh -s <device> netconf -p 830` | Opens a raw NETCONF session; the device immediately sends its **`<hello>` with its capabilities** — the list of what it can do |
| `show netconf-yang status` *(IOS-XE)* | Whether the NETCONF-YANG process is enabled and running on the device |
| `show platform software yang-management process` *(IOS-XE)* | The state of each YANG management process (confd, nesd, syncfd, ncsshd …) — verify against your release's command reference |
| `curl -k -u user:pass -H "Accept: application/yang-data+json" https://<device>/restconf/data/Cisco-IOS-XE-native:native/logging/monitor/severity` | The RESTCONF GET from Example 28-11, as a terminal command. A **200** plus the JSON body confirms RESTCONF is up and the YANG path is right |
| `git log --oneline` / `git status` | The commit summary GitHub shows on the repo page, locally. Run `git status` before every commit to confirm no secrets are staged |
| `git diff` | What you are about to commit, line by line — the last gate before credentials reach permanent history |

## Intent Questions

- **Is this a read or a write, and against what?** A GET against a sandbox and a DELETE against
  a production controller are the same three keystrokes in Postman. Know which one you are
  about to send before you click Send.
- **Which authentication scheme does this specific controller expect?** DNA Center wants Basic
  Auth with `application/json` and hands back a token; vManage wants
  `x-www-form-urlencoded` with `j_username`/`j_password` and hands back a `JSESSIONID`. Guessing
  costs you a 401 and twenty minutes.
- **Is this a configuration change or a state read?** YANG makes the distinction explicit
  (`config false`), NETCONF respects it, and SNMP cannot express it at all. If the model says a
  leaf is not configurable, no amount of retrying will write to it.
- **What is the smallest request that answers the question?** Full-inventory pulls are the
  default because they are the easiest to write, not because they are right. `limit` and
  `offset` exist for a reason.
- **Where will this code live, and who else will read it?** That decides whether credentials can
  be inline, whether `verify=False` is acceptable, and whether the script needs comments. A
  throwaway in a sandbox and a script in a shared repo have different rules.

## Troubleshooting Checklist

0. **State intent vs. observed.** Write the one-line symptom ("the inventory GET should return
   the device list, it is returning 401") — before changing anything.
1. **Read the status code first.** It partitions the problem: `2xx` means the call worked and
   the problem is in your parsing; `4xx` means **you** sent something wrong; nothing at all
   means you never reached the controller.
2. **`401 Unauthorized` — you are not authenticated.** Missing token, mistyped token, token from
   an expired session, or the header name is wrong. Remember the token is **valid only for the
   current authenticated session** and **changes on every authentication**.
3. **`403 Forbidden` — you are authenticated but not allowed.** Credentials were accepted;
   access is not granted for this call. This is a permissions problem on the controller, not a
   token problem, and re-authenticating will not help.
4. **`400 Bad Request` — client-side.** Malformed JSON body, wrong Content-Type, or a query
   parameter the API does not accept. Validate the body with `python3 -m json.tool` before
   blaming the controller.
5. **`404 Not Found` — the URL is wrong.** Every REST call maps to its own unique URL, so a
   typo in the path is a 404, not an error message about the resource. Check the API version
   segment (`/api/v1/` vs `/api/system/v1/`) — the Token API and the Network Device API use
   different ones.
6. **Wrong Content-Type is the classic cross-platform failure.** DNA Center expects
   **`application/json`**; vManage's Authentication API expects
   **`application/x-www-form-urlencoded`**. Sending one to the other fails in ways the error
   message rarely makes obvious.
7. **Token not being applied?** Confirm it is in the **`X-Auth-Token`** header, not the
   Authorization header, and that nothing overwrote the headers dictionary between
   authentication and the call.
8. **JSON key errors in your own code.** `KeyError: 'token'` usually means case — JSON keys are
   case-sensitive, and the chapter's own examples disagree about `"Token"` vs `"token"`. Print
   the raw `response.json()` and read the actual keys.
9. **Empty or partial results?** Check for a `limit`/`offset` left on the URL from an earlier
   test, and check whether the API paginates by default.
10. **TLS errors.** A certificate warning is information, not noise. Before reaching for
    `verify=False`, ask whether this is a sandbox with a self-signed cert (fine) or production
    (fix the trust chain instead).
11. **NETCONF session will not establish.** Confirm the process is enabled on the device
    (`show netconf-yang status`), that you are reaching **TCP 830** over SSH, and read the
    **`<hello>` capabilities** the device sends — if the capability you need is not listed, the
    device cannot do it and no amount of RPC will change that.
12. **NETCONF transaction rejected?** Transactions are **all or nothing** — there is no partial
    apply and no sequencing inside one. One bad element fails the whole thing, which is the
    designed behavior, not a bug. Use the test-before-commit path.
13. **RESTCONF 404 on a YANG path.** The path must match the model the device actually
    implements. Confirm the module name is right (`Cisco-IOS-XE-native:native/...`) and that
    the `Accept` header asks for `application/yang-data+json`.
14. **Script fails on import.** A missing module is the most common Python failure in this
    chapter's scripts — `requests`, `urllib3`, and `prettytable` are not in the standard
    library and must be installed.
15. **Cannot reproduce someone else's script result.** Check `Env_Lab.py`'s
    `ENVIRONMENT_IN_USE` — `sandbox`, `express`, and `custom` point at three completely
    different backends.

## Common Pitfalls

- **PUT replaces, PATCH appends.** Updating an NTP server is PUT; adding one is PATCH. Reversing
  them silently wipes the entry you meant to keep.
- **401 and 403 are not interchangeable.** 401 = not authenticated. 403 = authenticated, not
  authorized. Chasing a 403 by re-authenticating wastes the whole troubleshooting window.
- **An API DELETE is a real delete.** There is no confirmation prompt between a REST call and a
  controller's data. This is why the chapter says to practice in the DevNet sandbox.
- **Cisco DNA Center requires JSON — it is not optional.** The controller **expects all incoming
  REST data in JSON format**.
- **DNA Center and vManage authenticate differently.** `application/json` + Basic Auth + a
  **token** vs `application/x-www-form-urlencoded` + `j_username`/`j_password` + a
  **`JSESSIONID`**. This is the chapter's most reliably tested distinction (quiz Q4 and Q10 are
  the same fact from both sides).
- **The DNA Center token is per-session and changes on every authentication.** Caching one in a
  script and reusing it tomorrow produces a 401 that looks like a permissions change. (That
  token is itself a JWT — its `exp` registered claim is readable without the secret, so a
  script can check expiry instead of discovering it as a 401.)
- **Picking the signature as "the component that defines the signing algorithm."** The
  signature is what the algorithm *produces*; the **header** is where `alg` is declared. The
  signing step reads the algorithm out of the header.
- **Reading Base64URL encoding as encryption.** Header and payload are *encoded*, not
  encrypted — anyone holding the token can decode and read every claim. Signing protects
  integrity, not confidentiality. Never put a secret in a claim.
- **Assuming registered claims are mandatory.** RFC 7519 makes `exp`, `iss`, `sub`, and `aud`
  **recommended, not required** — a valid JWT can omit all of them.
- **`limit` and `offset` mean different things.** `limit` = how many records; `offset` = which
  one to start from. `?limit=1&offset=2` means "one record, the second one" — not "two records."
- **RESTCONF does not replace NETCONF.** The chapter calls this out as a **common
  misconception**. Both are current, and **RESTCONF uses the same YANG models**.
- **RESTCONF's method list includes OPTIONS but not PATCH** — which inverts the earlier REST
  table in the same chapter. GET, POST, PUT, DELETE, OPTIONS.
- **SNMP cannot distinguish configuration data from operational data.** NETCONF can. This is the
  single most important difference and the reason YANG models exist.
- **SNMP uses OIDs; NETCONF uses paths.** `interfaces/interface/eth0` vs a dotted numeric OID.
- **SNMP encodes with BER over UDP; NETCONF uses XML or JSON over SSH/TCP.** Four differences in
  one row of Table 28-6, and all four are fair game.
- **NETCONF RPC messages are XML** — even though NETCONF *encoding* can be XML or JSON, the RPC
  envelope itself is XML.
- **NETCONF transactions have no internal ordering.** "Do this part first" is not expressible.
  If you need sequencing, that is multiple transactions.
- **`config false` means read-only.** A YANG leaf carrying it — like `observed-speed` — reflects
  something the device detected, not something you can set.
- **YANG is RFC 6020; NETCONF is RFC 4741 and 6241; RESTCONF is RFC 8040.** Four RFC numbers,
  three technologies, and NETCONF is the one with two.
- **XML tags must be closed, and the end tag repeats the name with a leading `/`.** The
  unclosed tag is the most common XML failure, and `xmllint --noout` finds it instantly.
- **Indentation is legibility, not syntax** — for XML and JSON. It **is** syntax in Python.
- **Three quotation marks both open and close a multiple-line string** — that is why quiz Q7
  asks you to choose two answers.
- **`print` is a built-in Python function**, which is why quiz Q9's answer includes
  `print(dnac_devices)` alongside the `def`-defined one. A function does not have to start with
  `def` to be a function.
- **A Python dictionary and a JSON object look identical** — key/value pairs in curly braces.
  That resemblance is useful, but a dictionary is a live Python object and JSON is text.
- **A dictionary written on one line is the same dictionary.** Formatting is not structure.
- **`verify=False` disables certificate validation**, and `urllib3.disable_warnings()` hides the
  warning that tells you so. The chapter's scripts do both without comment because they target a
  sandbox. Do not copy that pattern into anything real.
- **Hardcoded credentials in a script reach git history permanently.** `Env_Lab.py` is a
  teaching artifact with published sandbox credentials, not a template.
- **The chapter's own scripts disagree on the token key** — `response.json()["Token"]` in
  Example 28-16 vs `response.json()["token"]` in Example 28-19. **JSON keys are case-sensitive**,
  so one of them raises `KeyError`. Print the response and check.
- **Postman's Clear All wipes the entire History**, not just the visible page. Individual calls
  are removed with the trash can icon on hover.
- **DevNet's Community page is the place for questions**, not Documentation — though note the
  printed answer key for quiz Q11 disagrees with the chapter text on this point (see below).

## Exam Preparation Tasks

### Key topics coverage map

Table 28-9, mapped to where each element lives in this skill. Only six rows — this chapter's
Key Topic icons are concentrated almost entirely on the API mechanics, not on the data models
or the Python, which is worth noticing in itself.

| Key topic element | Description | Page | Where it lives in this skill |
|---|---|---|---|
| Table 28-3 | HTTP Functions and Use Cases | 856 | Reference Tables → "Table 28-3 — HTTP functions and use cases", with the PUT-replaces / PATCH-appends note; Common Pitfalls |
| Table 28-4 | CRUD Functions and Use Cases | 856 | Reference Tables → "Table 28-4 — CRUD functions and use cases"; Key Concepts → "REST APIs" |
| Table 28-5 | HTTP Status Codes | 862 | Reference Tables → "Table 28-5 — HTTP status codes", with the 401 vs 403 note; Troubleshooting steps 1–5; Common Pitfalls |
| List | Steps to authenticate to Cisco DNA Center using a POST operation and basic authentication | 862 | Procedure → "Authenticating to Cisco DNA Center with Postman (Token API), 7 steps"; Reference Tables → DNA Center vs vManage comparison; Verification Commands (the `curl` equivalent) |
| List | Steps to leverage the Network Device API to retrieve a device inventory from Cisco DNA Center | 864 | Procedure → "Retrieving the device inventory with the Network Device API, 8 steps"; Verification Commands |
| Paragraph | Using the offset and limit filters with the Network Device API when gathering device inventory | 866 | Procedure → "Narrowing the result with query parameters"; Design Baseline row 5; Common Pitfalls (limit ≠ offset) |

**Coverage note.** **All 6 rows of Table 28-9 map to content here — no gaps, no partials.** The
whole chapter (pp. 849–890) was transcribed, including the quiz and the p. 856 answer key, and
every code example (28-1 through 28-21) is reproduced in Config Patterns from the book.
**Verification Commands are the exception**: the chapter has no CLI, so the `curl`, `xmllint`,
`json.tool`, NETCONF and RESTCONF commands there were **added by this skill** and are **not
gear-validated** — the two IOS-XE `show` commands in particular should be checked against your
release's command reference.

Worth flagging: the Key Topics table covers **only** the API mechanics. **YANG, NETCONF,
RESTCONF, DevNet, GitHub and all of the Python carry no Key Topic icon**, despite being most of
the chapter and all of the "Define Key Terms" list. Do not read the short table as permission to
skip them — the quiz tests Python (Q1, Q7, Q8, Q9), YANG (Q14), DevNet (Q11) and GitHub (Q12),
which is 7 of 14 questions.

### "Do I Know This Already?" question analysis

Answer key as printed on p. 856: **1** B · **2** D · **3** B · **4** D · **5** A · **6** C ·
**7** A, D · **8** A · **9** C, D · **10** D · **11** A, D · **12** A, C, D · **13** A, D ·
**14** B, C.

| Q | What it's really testing | Answer | Pitfall the distractors expose |
|---|---|---|---|
| 1 | Whether you absorbed the chapter's framing of Python | **B** — False | **Pure recall, no trap.** The chapter says Python is "one of the easier languages to get started with and interpret." The only way to miss it is to answer from your own experience rather than the text |
| 2 | Which HTTP method authenticates to DNA Center | **D** — POST | GET (c) is the trap for anyone thinking "I'm retrieving a token." You are **submitting credentials**, and Table 28-3 says POST **submits data** — with "submitting login credentials" as its literal use case. PATCH and PUT are there to punish guessing |
| 3 | The CRUD expansion, exactly | **B** — CREATE, READ, UPDATE, DELETE | **(c) CREATE, RETRIEVE, UPDATE, DELETE is the high-value distractor** — "retrieve" is a perfectly sensible word for what READ does, and Table 28-4's own Action column says READ "**retrieves** data." The acronym is fixed even though the description is not |
| 4 | The vManage Content-Type, against the DNA Center one | **D** — x-www-form-urlencoded | **(e) JSON is the trap**, because JSON is correct for **DNA Center** and the whole chapter trains you on DNA Center first. **(b) X-Auth-Token is also real but is a header key, not a Content-Type** — a category error made of a real term. Promoted to Common Pitfalls |
| 5 | Recognizing JSON by shape | **A** — the curly-brace key/value object | Distractors are **XML** (b), **plain text** (c) and a bracketed list (d). The one-glance discriminator: **JSON objects open `{` and close `}` with key/value pairs**; XML has `<tags>` |
| 6 | The Unauthorized status code | **C** — 401 | **(d) 403 Forbidden is the real trap** — both are authentication-adjacent 4xx codes. 401 = not authenticated; 403 = authenticated but not permitted. 400 and 404 are there for anyone who only remembers "404 is the error one." Promoted to Common Pitfalls |
| 7 | Python multiple-line strings | **A, D** — to begin and to end a multiple-line string | The "choose two" **is the hint** — the same token does both jobs, which is the actual fact being tested. (b) function, (c) logical OR, (e) reusable code are all real Python concepts attached to the wrong syntax |
| 8 | Recognizing a dictionary | **A** — `dnac = { ... }` | (c) is a **function**, (d) is a **function call** — so Q8 and Q9 are the same four options asked twice, testing whether you can tell a data structure from a block of code. Curly braces plus key/value pairs = dictionary |
| 9 | Recognizing functions — both kinds | **C, D** — the `def dnac_login(...)` block **and** `print(dnac_devices)` | **The trap is answering C alone.** `print` is a **built-in Python function**, which the chapter states explicitly. If you think "function" means "starts with `def`," you get this half right. Promoted to Common Pitfalls |
| 10 | The DNA Center Token API's auth method | **D** — Basic authentication | Mirror of Q4. **(b) X-Auth-Token is the sharpest distractor** — it is real, it is DNA Center, and it is the header used on **subsequent** calls. But it is not how you authenticate *to the Token API*; Basic Auth is. Sequence matters |
| 11 | What DevNet's Documentation page is for | **A, D** as printed | (d) "to access API information" matches the chapter exactly. **(a) "to ask questions" does not** — p. 879 attributes question-asking to the **Community** page, "a safe zone for asking questions." **The printed key and the chapter text appear to conflict here.** Learn the chapter's mapping (Documentation = API docs, Community = questions, Learn = labs, Events = events) and treat this key as suspect |
| 12 | What a GitHub repository is for | **A, C, D** — store a developer's code, share it with other users, provide documentation on code examples | **(e) "offers a sandbox to test custom code" is the trap** — a real concept from earlier in the same chapter, but that is the **DevNet sandbox**, not GitHub. A right answer borrowed from the wrong section. (b) music and photos is filler |
| 13 | Why the CLI does not scale | **A, D** — prone to human error and misconfiguration; used on a device-by-device basis | **(b) and (e) are inverted statements** — the CLI being "quick and efficient for many devices simultaneously" and APIs being "legacy" are the opposite of the chapter. (c) "Telnet is best practice" is actively wrong and contradicts Table 28-2 |
| 14 | YANG structural elements | **B, C** — Leaf, Container | **The weakest question in the chapter.** (a) Type and (d) String both *literally appear* in Example 28-6 as `type string;`. The intended distinction is that **leaf and container are YANG node types** (structural elements of the tree) while type is a statement and string is a datatype — but the wording does not carry that. Know the node types: **module, container, list, leaf, leaf-list, choice, case** |

**Pattern worth noting.** This quiz is unusually **paired**: Q2/Q10 both test DNA Center
authentication from different angles, Q4/Q10 contrast vManage against DNA Center, and **Q8/Q9
reuse the identical four code snippets** to test data structure vs function. The distractors are
overwhelmingly **real things from the wrong slot** — X-Auth-Token as a Content-Type, JSON as
vManage's encoding, the DevNet sandbox as a GitHub feature, RETRIEVE for READ. Two questions are
worth treating with suspicion rather than memorizing: **Q11's printed key conflicts with the
chapter text**, and **Q14's distractors appear verbatim in the chapter's own YANG example**.
