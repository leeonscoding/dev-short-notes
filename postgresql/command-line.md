# Login
Locally using user and hostname
```bash
psql -U root -h localhost
```
Remotely
```bash
psql -U <user> -h <host address> -p <port> -d <database name>
```

# Quitting
```bash
\q
```

# List databases
```bash
\l
```

# Connect databases
```bash
\c <database name>
\c testdb
```

# Display tables
First connect with a database using the command '\c', then
```bash
\dt
```

# Display columns of a table
```bash
\d <table name>
//OR for more info
\d+ <table name>
```

# Create a database
First quit from psql if you are in the psql using '\q', then
```bash
createdb paste
```
