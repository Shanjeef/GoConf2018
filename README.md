# GoConf2018
Takeaways from attending GoConf2018

Day 1: Testing Workship:
-------------------------

- Internal vs external black box testing (https://www.gopherguides.com/courses/testing-in-go-gophercon-2018/modules/fundamentals-testing-module#slide-16). Can make the test file a different pkg. Ex: product pkg "hello", can have a hello_test.go, which is part of pkg "hello_test". Can only verify exposed/public methods of "hello". Can be helpful for getting around circular dependencies in test code

- Run unit tests recursively in sub pkg (go test ./...)

- Increase unit tests run time in parallel. Can leverage the "parallel" flag. Can also place the `t.Parallel()` at the start of the test to allow it to run in parallel *within* the package unit tests. Perhaps we can place in the unit test pkg's "init" method, to avoid doing this in each individual test?

- Look at the "race" flag:
	- https://www.gopherguides.com/courses/testing-in-go-gophercon-2018/modules/fundamentals-testing-module#slide-37
	- https://www.gopherguides.com/courses/testing-in-go-gophercon-2018/modules/fundamentals-testing-module#slide-39

- You can create "Example" functions which will generate go doc: 
	- https://www.gopherguides.com/courses/testing-in-go-gophercon-2018/modules/fundamentals-testing-module#slide-45

- Look at MOQ: https://github.com/matryer/moq which you can point an interface to, and it will create a mock concrete implementation of the interface for tests. We're not using dependency injection in unit tests; instead we create pkg variables assigned to functions, which are overwritten (mocked) in unit tests. Downside is in future, other packages may depend on this functionality and make these function variables public!

- Look at go:generate. Can link general go generate commands

- Demo TDD: Write an API to order Pizza

Day 2:
-------

**Talk 1: Go Scheduler:**
  - Distributes goroutines over multiple OS worker threads
  - Go has a work-stealing implementation (vs work-sharing):
    - Work-sharing: When a processor generates new threads, it attempts to migrate some of them to the other processors with the hopes of them being utilized by the idle/underutilized processors.
    - Work-stealing: An underutilized processor actively looks for other processor’s threads and “steal” some.
  - Each processor is assigned a local queue of goroutines. 
  - Each processor can be assigned one or more OS threads, but only one can be executed at a time (per processor)
  - A processor is starved if its OS threads are blocked or in a system call, and the local queue is empty
  - In the case the processor is idle, it can steal goroutines from other processor's local queues
  - At intermittent times, the processor is assigned work from the global queue as well
  - However, re-allocating goroutines across queues increases latency

**Talk 2: Macaroons**
- Macaroons are essentially cookies (tokens) with authorization info.
- They can be delegated to another entity which will act on the requester's behalf, with the requester's authority
- Caveats can be added to the macaroon (by 3rd parties as well), so it be used to restrict what services can do even with your full authority. Ex: a member's role is "admin", but caveats can be added at runtime to the macaroon, to add restrictions with what can be done with the token instance

**Talk 3: Go For Information Displays:**
- Look at SVG Go, SVG Play which will allow us to change code (data) and generate pictures, charts.
- Used rasberry-pi to run a program that loads weather and current stories onto screen. Leveraged OpenVG pkg
- Check out his GoDeck pkg which is for creating presentations

**Talk 4: Machine Learning on GoCode**
- Source{d} (company), are leveraging machine learning to have some automated code reviews, style guide enforcing.

**Talk 5: Structuring Go Apps:**
- Used a beer reviewing app example: able to add beers, review beers, list all beers, ability to add sample data
- "Layered Architecture"; group by functionality (handlers, models, storage, etc)
- Group by Module; (beers, reviews, storage)
- Domain Driven Development:
  - Establish the domain and business logic
  - Define the bounded context(s), the models within each context. Ex: "entities", "value objects", "aggregates". Value Objects are more like properties of Entities.
	- Group by context. Ex: "adding", "listing", "beers", "reviews"
- Hexagonal Architecture --> dependencies only point inwards
- References:
  - Ben Johnson on Medium re: package design
  - Go and Package Focused Design, Gopher Academy Bloc
  - Repo Structure, Peter Bourgon
  - "Building an enterprise service in Go"
  - Hexagonal Architecture, Chris Fidao

**Talk 6: Allocator Wrestling:**
- Program's allocation pattern can affect its perf, but they are opaque
- Depending on their lifetimes, objects are allocated either on stack or heap
- Efficiently satisfy allocations of a given size, but avoid fragmentations. --> Allocate like sized objects in blocks
- Avoid locking in common case -> maintain local caches
- Heap is divided into arenas and spans.
- Go's garbage collector: tricolor concurrent mark sweep collector
- Look at pprof and execution tracer
- Basically check what % of CPU was available for work, vs garbage collection
- go build -gcflags= "-m -m" tells us which variables are stored on the heap
- Readings:
  - Allocation Efficient in High Perf Go Routines 

**Talk 7: gPRC, Finite State Machine, Talk:**
- State Machine Types: Deterministic vs Non-Deterministic
- Ex: Onboarding/Login UX Flow; final output depends on sequence of defined inputs
- Shouldn't be stuck in a state, add monitoring around error threshold, and possible revert to a previous state
- Read up on protobuf protocol

Day 3:
-------

**Talk 1: Writing Accessible Go:**
- Toolset isn't written with accessibility in mind
- Products are made accessible, but development process isn't targeted towards those with disabilities
- Declare variables just prior to time of use, in order to reduce information context a reader must have
- Write code in "paragraphs" and strategically placing new lines
- Be consistent with styles, have style guides, enforce them
- Disability isn't always visible, there may be folks that don't wish to voice them
- "Curb Cut Effect" --> Something built for one audience, can have benefecial impacts on other audiences

**Talk 2: Going Serverless**
-  Its offloading processing to a black-box service provider, running your logic
- Reduces boiler plate (ex: Leveraging muxs, deployment code)
- Functions (your logic) + Events (resonse to something that happens in the cloud) + Manged Services (pay per use)
- Rather than hosting your own service, leverage managed services
- GCP will support serverless soon for Go. You can write functions, with unit tests, and url triggers
- Ensure that each func only has a single connection to the DB, in order that many concurrent requests (each being a func), don't exhaust the database connection pool
- We need instrumentation tools (ex: Open Census) to be able to obtain tracing
- Questions: How would you leverage shared helper methods across funcs?

**Talk 3: Go in Debian**
- manpages.debian.org

**Talk 4: Contributing to Go**
- Contribute error reports:
	- What were you trying to do?
	- What did you expect to see?
	- What did you see?
	- How to reproduce error? Give a reduced test case(s).
- Contribute documentation. "Example" unit tests, as people scan vs read
- Start off with small commits, review other commits in order for you to understand the changes.
- bit.ly/goscratch to test out some initial changes. Its a scratchpad for you to understand how the committing process works
- golang-dev mailing list for posting help or when you get stuck with your changes

**Talk 5: Guide to Secure Connections:**
- Look at "Kubernetes The Hard Way" which walks through setting up security keys, etc but doesn't teach you the details as to why these keys and security files are required
- TLS (was called SSL) connections: Establish identities between 2 parties, and encrypt traffic
- "Connection Refused" err msg usually mean wrong port thats not open, so TCP error, and usually not anything wrong with TLS settings
- Public keys are used to encrypt, but private keys are used to decrypt
- "Msg" + private key = signature. ("Msg" + signature) is sent to the server along with the public key. Msg + Public key = signature
- Sharing the public key to other party needs to be verified re: identity. Certificate Authority is leveraged for this.
- Certificate(X.509) contains: Subject Name, Subject's public key, Issuer (CA) name, Validity
- If you want a certificate, you need to create a Certificate Signing Request, using your private key
- CLI Tools to create the certificate: openssl, cfssl, mkcert, minica. 
- A certificate and private key is created.
- .pem files are for certificate and keys
- Can use openssl to output the contents of the certificate, which contains the public key
- When creating the server, you create a tls.Config object and set your public key and certificate file
- Clients need to be aware of the CA in order to validate the certificate. Typically you can load the system's CA pool which has a list of big CAs.
- Servers can then set (via tls.Config) that clients' certs are verified ("ClientAuth" property)
- Client can generate a cert and private key, register it in its own tls.Config
- Setting up a secure connection:
	Server: ListenAndServeTLS(cert, key)
	Client: 
		- tls.Dial
		- May need to add CA cert to TLSConfig.RootCAs
- github.com/lizrice/secure-connections

**Talk 6: MongoDB: Lessons Learned From Implementing Specs:**
- MongoDB has many drivers for different languages, so they consistent specs
- Tests are standardardized for different drivers; yaml, json and prose test
- Write clear specs that aren't open for implementation
- Share your knowledge gained during the process
- blog.cloudflare.com/exposing-go-on-the-internet

**Talk 7: Building Command Line User Interfaces:**
- bit.ly/cli-ui
Pkgs:
	- chzyer/readlines
	- manifoldco/promptui
	- gdamore/tcell
	- mattn/go-runewidth
