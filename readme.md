# US Colleges API

This is an open API for accessing the names and locations of accredited colleges and universities in the United States.

## Local Installation

### Prerequisites

- Docker and Docker Compose
- NPM/NodeJS 6+

### Local Setup

- Clone this repository.
- Install the composer packages: `npm run -s composer:install`.
- Copy `.env.example` to `.env` and add your credentials.
- Build the dockerfile: `npm run -s app:local:build`.
- Bring up the containers: `npm run -s app:local:up`. It may take a few seconds for the database to start up the first time.
- Run the migrations: `npm run -s db:migrate`.
- Run the seeder: `npm run -s db:seed`.

The application's documentation will be reachable at `localhost:8100/`.

## Usage

### API Documentation

- Once your web application is running, visit `localhost:8100/` to view the documentation.
- By default, all requests are made on the first page of 100, ordered by ID. You can change that though.
- Examples:

#### Get the second 100 colleges sorted by name and transformed:

``` 
http://localhost:8100/colleges?order=name&page=2,100&transform=1
```

#### Get the second page of records for colleges in Memphis, TN:

``` 
http://localhost:8100/colleges?order=id&page=2,20&filter=location,eq,Memphis, TN
```

#### Get a single college by uuid
```
http://localhost:8100/colleges/a3379d4e-bc71-4a7d-b7d9-c954c8b1d300
```

Currently only GET requests are supported. POST, PUT, DELETE and others will be available with API key authorization in a later release.

## Server Deployment
This project is intended to be hosted on [Hyper.sh](https://hyper.sh/), but you can set it up anywhere you host docker containers.

### First time deployment
- Upload local data to volume: `hyper run -v $(pwd)/.data:/.data ubuntu`
- Remove the original container: `hyper rm <CID>`
- Create snapshot: `hyper snapshot create --volume <VID> --name colleges`
- Create named volume from snapshot: `hyper volume create --snapshot=colleges --name=colleges`
- Pull the latest: `hyper pull karllhughes/colleges:latest`
- Bring up the containers: `hyper compose up -f docker/compose.hyper.yml -p colleges$RANDOM -d`
- Attach your floating IP: `hyper fip attach <FIP> <CID>`

### Updating code

Updating production code:

- Pull the latest: `hyper pull karllhughes/colleges:latest`
- Recreate the containers: `hyper compose up -f docker/compose.hyper.yml -d --force-recreate -p <PID>`
- Attach your floating IP: `hyper fip attach <FIP> <PID>`

Or the single command/container:

- Add a `docker/deployer/.env` file with your Hyper.sh credentials.
- Run `npm run -s app:prod:deploy`.

## Technical Details

- The list of colleges is retrieved from [ope.ed.gov](https://ope.ed.gov/accreditation/GetDownloadFile.aspx), and the last update was in March 2016.
- I standardized fields based on [Schema.org's CollegeOrUniversity object](https://schema.org/CollegeOrUniversity). Not all fields are currently supported and I had to add a couple special ones, so check the migrations files for a list of supported fields.

