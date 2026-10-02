## PHASE 1

### Issue 1

#### Log output 
orbis-backend  | prisma:warn Prisma failed to detect the libssl/openssl version to use, and may not work as expected. Defaulting to "openssl-1.1.x".

orbis-backend  | Please manually install OpenSSL and try installing Prisma again.

orbis-backend  | Database connection attempt 1 failed: PrismaClientInitializationError: Unable to require(`/app/node_modules/.prisma/client/libquery_engine-linux-musl.so.node`).

orbis-backend  | The Prisma engines do not seem to be compatible with your system. Please refer to the documentation about Prisma's system requirements: https://pris.ly/d/system-requirements

orbis-backend  | 
orbis-backend  | Details: Error loading shared library libssl.so.1.1: No such file or directory (needed by /app/node_modules/.prisma/client/libquery_engine-linux-musl.so.node)


### Diagnosis and FIX

**Diagnosis**
	Basically Prisma is used for the translation of JS code to SQL queries. It requires OPENSSL to establish a secure network connections.
	Since the image is built on the alpine versions therefore it is missing the shared library libssl.so.1.1 
**FIX**
	 In the /backend/Dockerfile added
		  RUN apk add --no-cache openssl


### ISSUE 2

#### Log output
prisma:info Starting a postgresql pool with 25 connections.

orbis-frontend  |  INFO  Accepting connections at http://localhost:80

orbis-backend   | Database connection attempt 1 failed: PrismaClientInitializationError: Can't reach database server at `database:5432`

orbis-backend   | 
orbis-backend   | Please make sure your database server is running at `database:5432`.

orbis-backend   |     at t (/app/node_modules/@prisma/client/runtime/library.js:112:2488)

orbis-backend   |     at async connectWithRetry (file:///app/src/config/database.js:20:7) {
orbis-backend   |   clientVersion: '6.0.1',
orbis-backend   |   errorCode: 'P1001'
orbis-backend   | }


#### Diagnosis and Fix

**Diagnosis**
	So the error is that there is a problem in establishing a connection with the database looking into the docker-compose file the database is attached to the backend-network whereas the backernd is attached to the frontend-network only  because it utilises prisma and translates JS code to SQL Queries therefore it also requires to communicate with the backend network
**FIX**
	 In the docker compose file 
	 add in
	backend:
		networks:
			- backend-network


### ISSUE 3 

#### log output
orbis-frontend  |  INFO  Accepting connections at http://localhost:80
#### ISSUE DETAIL
when we click on this url it gives us a blank page, on inspecting the console
Uncaught Error: supabaseUrl is required.

    GB http://localhost/assets/index-CMXlV9Ng.js:119  

    VB http://localhost/assets/index-CMXlV9Ng.js:119  

    <anonymous> http://localhost/assets/index-CMXlV9Ng.js:119
    
#### Diagnosis and FIX
**Diagnosis**
Since our frontend is built on vite, it requires environment variables  like VITE_SUPABASE_URL, to be present at build time.  The Dockerfile in the frontend was executing npm run build without passing any environment variables. Therefore Supabase client tried to initialize with undefined value, thereby crashing React before it could render.

**FIX**
In the frontend/Dockerfile added these:

ARG VITE_SUPABASE_URL=https://dummy.supabase.co
ARG VITE_SUPABASE_ANON_KEY=dummy-key
ARG VITE_SUPABASE_SERVICE_ROLE_KEY=dummy-service-key

ENV VITE_SUPABASE_URL=$VITE_SUPABASE_URL
ENV VITE_SUPABASE_ANON_KEY=$VITE_SUPABASE_ANON_KEY
ENV VITE_SUPABASE_SERVICE_ROLE_KEY=$VITE_SUPABASE_SERVICE_ROLE_KEY

And then in the Dockercompose file added these
args:
	VITE_SUPABASE_URL: "https://dummy.supabase.co"
	VITE_SUPABASE_ANON_KEY: "dummy-key
	VITE_SUPABASE_SERVICE_ROLE_KEY: "dummy-service-key"

