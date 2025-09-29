---
title: Creating GraphQl in Go
date: 2025-09-12
description: Creating a schema first graphql api in go with sqlite3 db
draft: true
category: Go GraphQl
---

# Note: Creating a GraphQL Server in Go with SQLite3

This note summarizes the key steps for building a GraphQL API, [address-api](https://github.com/xaaha/address-api), in Go with a SQLite database. The goal was to take scraped JSON data, migrate it to a database, and expose it through a GraphQL API using a schema-first approach with [gqlgen](github.com/99designs/gqlgen).

### 1. Initial Project Structure

Organize the repo however, but I went with typical enterprise structure because I wanted to understand this structure better. Heck, for this project, all the things could be contained in couple of files. 

```bash
├── cmd               # Main application entry points
│   ├── server        # For the long-running API server
│   └── setup         # For one-time setup scripts (like DB migration)
├── db                # Database-related files
│   └── migrations    # SQL files for schema changes
├── internal          # Private application logic (e.g., database access)
└── graph             # GraphQL generated code and resolvers
````

### 2. Database Migration

Move data from its source (e.g., JSON files) into your database.

  - **Create a Schema:** Define your table structure in an `.sql` file (e.g., `db/migrations/001_create_tables.sql`).
  - **Write a Go Script:** Create a one-time script (`cmd/setup/main.go`) that:
    1.  Reads the source data (e.g., JSON files).
    2.  Connects to the SQLite database file (`sql.Open("sqlite3", "...")`).
    3.  Executes the SQL schema to create the tables.
    4.  Loops through the data and runs `INSERT` statements to populate the tables.
  - **Run the Migration:** Execute the script as a separate step before starting your server: `go run ./cmd/setup`.

  >[!Note]
  > The address-api repo also has a one-time script file, `internal/data/cleanup.go`, that cleans up each file. I ran it once, hence it does not have tests associated with it.

### 3. GraphQL Schema-First Design with gqlgen

Define your API contract first, then generate the Go boilerplate.

1.  **Initialize gqlgen:**

    ```bash
    go run [github.com/99designs/gqlgen](https://github.com/99designs/gqlgen) init
    ```

2.  **Define Your Schema:** Edit `graph/schema.graphqls` to define your types and queries. Use `camelCase` for field names as it's the API convention.
      - **Types:** Mirror your database structure (e.g., `type Address {...}` in this case).
      - **Queries:** Define the entry points for fetching data (e.g., `addressesByCountryCode(countryCode: String!): [Address!]!`).
3.  **Generate Go Code:** Run the generate command to create Go models and resolver stubs.
    ```bash
    go run github.com/99designs/gqlgen generate

    ```

    >[!Tip]
    > Define the generate in Make file. So, when we add mutation or query in the future, we can simply run make generate


```Makefile 
# generate the resolver code form the schema.graphqls file
generate:
	@go run github.com/99designs/gqlgen generate
        
```

### 4. Implementing Resolvers

This is where we write the logic to connect API to the database.

1.  **Dependency Injection:**
      - Add database connection (`*sql.DB`) to the `Resolver` struct in `graph/resolver.go`.
      - In the `cmd/server/main.go`, open the database connection **once**.
      - Inject this connection when you create the resolver instance: `resolver := &graph.Resolver{DB: db}`.

2. **Define Schema:**
      - Define your schema in `graph/schema.graphqls`.
      - Then run generate command to generate the code for the schema 

3.  **Write Resolver Logic:** Open `graph/schema.resolvers.go` and implement the generated functions. This is the main file where we write our implementation
      - Use the injected `r.Resolver.DB` to access the database.
      - **Always** use parameterized queries (`?`) to prevent SQL injection.
      - Use `db.QueryContext(ctx, ...)` to support request cancellation.
      - **Always** use `defer rows.Close()` to release database resources.

### 5. Starting the Server

Configure and run the main web server in `cmd/server/main.go`.

  - **Handlers:** Set up two handlers:
    1.  The GraphQL API endpoint (e.g., `/query` or `/graphql`).
    2.  The GraphQL Playground developer tool (`/`), which points to your API endpoint.
  - **Port:** Use environment variables (`os.Getenv("PORT")`) with a default fallback (e.g., "8080") to make your server deployable.
  - **Start and Handle Errors:** Use the `log.Fatal(http.ListenAndServe(...))` idiom to start the server and exit immediately if it fails to launch.

