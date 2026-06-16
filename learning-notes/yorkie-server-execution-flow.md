# Yorkie Server Execution Flow

## Goal

This note summarizes the execution flow of `./bin/yorkie server` that I explored while reading the Yorkie source code.

## Overall Flow

./bin/yorkie server
→ cmd/yorkie/main.go
→ main()
→ Run()
→ rootCmd.Execute()
→ newServerCmd()
→ server.New(conf)
→ backend.New(...)
→ mongo.Dial(...)
→ ensureIndexes()

What I Found
1. Entry Point

cmd/yorkie/main.go is the entry point of the Yorkie binary.

func main() {
    os.Exit(Run())
}

The main() function does not start the server directly.
It delegates execution to Run().

2. CLI Execution

Run() executes the root Cobra command.

rootCmd.Execute()

This means Yorkie is structured as a CLI application with subcommands such as:

yorkie server
yorkie project
yorkie document
yorkie version

3. Server Command

The server command is defined by newServerCmd().

The command prepares configuration values and then creates and starts the Yorkie server.

newServerCmd()
→ server.New(conf)
→ y.Start()

4. Yorkie Server Creation

server.New(conf) creates the main Yorkie server object.

It initializes:

metrics
backend
RPC server
profiling server

5. Backend Creation

backend.New(...) creates the internal backend components.

It initializes components such as:

cache
pubsub
lockers
background tasks
membership
housekeeping
channel manager
database

If MongoDB configuration exists, Yorkie uses MongoDB.

mongoConf != nil
→ mongo.Dial(mongoConf)

If MongoDB configuration does not exist, Yorkie can use an in-memory database.

6. MongoDB Connection

mongo.Dial(...) connects Yorkie to MongoDB.

It performs:

mongo.Connect()
client.Ping()
ensureIndexes()
cache initialization

7. MongoDB Index Initialization

ensureIndexes() creates required indexes for MongoDB collections based on collectionInfos.

Important collections include:

ColUsers
ColProjects
ColClients
ColDocuments
ColChanges
ColSnapshots
ColVersionVectors

8. Change Storage Insight

CreateChangeInfos() shows that operation changes are stored in ColChanges.

It also updates document metadata such as server_seq in ColDocuments.

Summary

By following the execution flow, I learned that ./bin/yorkie server starts from the CLI entry point, builds server configuration, creates backend components, connects to MongoDB when configured, prepares indexes, and stores document changes through dedicated MongoDB collections.
