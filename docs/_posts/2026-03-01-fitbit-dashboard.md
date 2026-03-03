---
layout: post
title:  "Fitbit Dashboard Application"
date:   2026-03-01 12:00:00 -0500
tags: [fitbit,kotlin,claude-code,vibe-coding]
---

Like most software professionals, I've always maintained a set of pet projects that were not primarily meant to 
solve a problem, but to explore and learn new technologies and programming languages.
My pet projects were usually based on collecting, displaying, and managing server stats from my Linux PC, for no other 
reason than that it was an available and consistent source of real world data.  

About a decade and a half ago, I switched from using a Linux PC as my every day desktop computer to a MacBook. I don't 
want to make this a story about the pros and cons of that choice. Someone at work used a MacBook, and made it look like 
the unix-based laptop I always wanted to have, and I figured I could afford something nice by now, that I didn't have to 
go for the affordable option by default any more. 

One side effect of this choice was that I would have to get a new source of data for my pet projects. The first thing I 
tried was open source weather data. When the weather API I was using was taken off the air, I decided biometrics might 
make a fun and more dependable source of data, so I bought a fitbit. Exercise made the graphs more interesting so I 
got more and more into endurance exercise. It turned out running through the forest for hours on end was a lot of fun. 
That, combined with a series of other weak excuses such as evening classes, young children, moving to another continent, 
and needing to stay up to date on tech stacks that couldn't really be used for this project, led to me never actually 
writing the fitbit app. 

In 2021, Fitbit was acquired by Google. First they took the fitbit dashboard down, then they stopped releasing new 
models, so it seemed just a matter of time before they would take the API offline as well. In addition to that, I was 
never completely sure I could trust Google with my biometric data. Google is a data company; their business model 
revolves around your data. Competitors like Garmin, Samsung, and Apple are hardware companies. Their 
business model is selling you gadgets, not gathering data or selling ads. I'm not trusting them blindly, I'm just 
looking at incentives. I kept putting it off because I didn't want to lose a decade worth of data. 

Finally Google made the choice for me. They announced they would delete the fitbit data unless you migrated it to your 
google account. This meant I now had to make the app if I wanted to keep my data in a useful format. I decided to try to 
recreate the old Fitbit dashboard.

The goals of this project are two-fold: 
First, I wanted to be able to preserve and view my historical Fitbit data after
moving to a tracker from another brand. 
Second, I wanted to do this without writing any code by hand, using only the
latest AI tools available to me. 
A stretch goal was to convert and import the historical data to my new tracker. 

Because of the EU General Data Protection Regulation, Google was forced to allow you to download all your personal data, 
which came in the form of a large zip file. The data turned out to be a mix of relational and unstructured data. I made 
the choice to import the data into a database, and make a web application, exposing the data via GraphQL, to support 
querying the data as well as support a web UI with graphs, as a replacement for the old Fitbit dashboard.

It took some planning and correcting to make the AI stick to the architecture and not always take the easiest path to
implement a feature. We have to keep in mind that a significant part of architecture is to keep the code readable and
maintainable by humans. I can imagine that future architectures which are designed never to be touched by humans, may 
favor very different choices, but as of late 2025 we are not there yet.

### Vibe coding

I started my professional career working at an AS/400 shop. 
There I worked with a number of very nice, smart, and highly motivated people, who each had 10 to 20 years of experience 
developing a single ERP application in a forgotten programming language on a dying platform. It occurred to me that if 
that company ever went out of business, they would have a hard time finding other work. I've always kept this in the 
back of my mind. 

In early 2025, Andrej Karpathy made quite a stir when he introduced the term 
[Vibe Coding](https://x.com/karpathy/status/1886192184808149383?lang=en) to the world. 

> There's a new kind of coding I call "vibe coding", where you fully give in to the vibes, embrace exponentials, and 
> forget that the code even exists.

When he wrote this, the tools I had access to were not capable of that yet, but if that was the direction software 
development was going to take, I had no intention of being left behind. 

There was a significant leap in capabilities of the AI tools between when the project was started in early 2025 and when
it was finished. By spring 2025 the tools were just starting to become useful, and by the end of the year it was clear
that being a "programmer", either as a profession or as an identity, is going the way of weavers and blacksmiths. It is 
no use to be nostalgic about this. In a previous blog post, I wrote about learning 6502 assembly as a teenager. This is 
not a skill I'm ever likely to use again, nor is anyone ever likely to pay me for writing C or x86 assembly. I have 
always known the same would one day be true for Java, though until recently I would not have predicted it would be 
replaced by prompting an AI. 

Who could have predicted it? Well, Andrej Karpathy did, in his 
[2017 essay on Software 2.0](https://karpathy.medium.com/software-2-0-a64152b37c35). 
He [summarized](https://x.com/karpathy/status/893576281375219712?lang=en) his point as follows: 

> Gradient descent can write better code than you. I'm sorry. 

In this essay, he predicted that future software would be trained rather than programmed. 
Reinterpreted with a couple of years of hindsight, that summary is more accurate now than he might have suspected 
at the time. 

## Disclaimer

If you decide to run this application for yourself, do not expose it to the internet. This is your personal data.
Fitbit records when you sleep, when you exercise, your GPS locations (though we are not importing that data). I think
it should be obvious why you need to keep this private. The security / authentication serves only to support multiple
users, it is not designed to protect your data when exposed to the internet. The registration
endpoint is open and anyone with network access can create an account.

I accept no responsibility for any consequences of using this software or the information in this documentation.

## Obtaining Your Fitbit Data

To use this application, you first need to download your data from Fitbit:

1. Log in to your Fitbit account at [fitbit.com](https://www.fitbit.com)
2. Go to **Settings** (gear icon) → **Data Export**
3. Or navigate directly to: https://www.fitbit.com/settings/data/export
4. Click **Request Data** to export your complete Fitbit history
5. Fitbit will email you when your data is ready (this can take a few hours to days)
6. Download the ZIP file.

### Import Your Fitbit Data

You can either:
- **Upload it directly** via the dashboard (Import Data) or the REST API. Click on the avatar in the top right
- **Extract it** to a `data` directory for CLI import. See the project's README for the steps involved. 

### 1. Run the Server

```bash
mvn -pl server spring-boot:run
```

The server will start on http://localhost:8080

### 2. Create an Account

Open the dashboard and click "Create one" on the login page to register a new account. After registering, you'll be automatically logged in and prompted to import your Fitbit data.

Username must be 3-50 characters (letters, numbers, hyphens, underscores). Password must be at least 8 characters.

### 3. Access the Application

- **GraphiQL** (GraphQL IDE): http://localhost:8080/graphiql
- **REST API**: http://localhost:8080/api

### 4. Run the Dashboard (Optional)

```bash
cd dashboard
npm install
npm run dev
```

The dashboard runs on http://localhost:3000 and proxies API requests to the server.

## Docker Deployment

The application is packaged as a single Docker image containing PostgreSQL 17, the Spring Boot backend, and the
SvelteKit dashboard. The image is stored as a private package on GitHub Container Registry (GHCR).

### Prerequisites

- Docker installed on both your development machine and the target server
- A GitHub account with access to the package
- A GitHub Personal Access Token (PAT) — see below

### Creating a GitHub Personal Access Token

You need a token to push the image from your machine and to pull it on the server.

1. Go to **https://github.com/settings/tokens**
2. Click **Generate new token → Classic**
3. Give it a descriptive name (e.g. `fitbit-deploy`)
4. Select scopes:
    - **`write:packages`** — to build and push from your dev machine
    - **`read:packages`** — to pull on the server (a separate read-only token is safer for the server)
5. Click **Generate token** and save it somewhere safe — GitHub only shows it once

### Setup

Both scripts read your GitHub username from the `GITHUB_USER` environment variable. Add it to your shell profile (`.zshrc`, `.bashrc`, etc.) so you don't have to set it every time:

```bash
export GITHUB_USER=your-github-username
```

Log in to GHCR on each machine you'll use:

```bash
echo YOUR_PAT | docker login ghcr.io -u $GITHUB_USER --password-stdin
```

### Building and Pushing the Image

Run from the project root on your development machine:

```bash
./docker/build-and-push.sh           # builds and pushes :latest
./docker/build-and-push.sh v1.2      # builds and pushes a specific tag
```

This creates a multi-platform image (`linux/amd64` and `linux/arm64`) using Docker BuildKit.

### Running on the Server

Copy `docker/run-on-server.sh` to the server, then:

```bash
POSTGRES_PASSWORD=changeme ./run-on-server.sh           # pulls and starts :latest
POSTGRES_PASSWORD=changeme ./run-on-server.sh v1.2      # specific tag
```

The container binds to port 8080 on all interfaces by default.

## GraphQL API

The application exposes a GraphQL API at `/graphql` for querying Fitbit data.

### Example Queries

#### Heart Rate Data

```graphql
# Get heart rate aggregated by time interval
query {
  heartRatesPerInterval(range: {from: "2024-01-01T00:00:00Z", to: "2024-01-01T23:59:59Z"}) {
    timeInterval
    bpmSum
  }
}
```

#### Steps Data

```graphql
# Get daily step totals
query {
  dailyStepsSum(range: {from: "2024-01-01T00:00:00Z", to: "2024-01-31T23:59:59Z"}) {
    date
    totalSteps
  }
}

# Get weekly step averages
query {
  weeklyStepsAverage(range: {from: "2024-01-01T00:00:00Z", to: "2024-03-31T23:59:59Z"}) {
    weekNumber
    averageSteps
  }
}
```

## Data Export

Only Apple Health XML format is currently supported.

The Apple Health API, also known as HealthKit, is only available on iOS, so we can't directly upload the data. This
application can export the stats to files in the Apple Health XML format, which you can import into Apple Health using
one of several available (paid) apps in the app store. I'm not affiliated with any of them. They're fine, the ones I
tested all worked. 

### Via Dashboard

Log in to the dashboard, click your profile avatar, and select **Export to Apple Health**. Choose a data type and date
range, then click Export to download the XML file.

### Via CLI

```bash
# Export heart rate data
curl -u user:password "http://localhost:8080/api/export/heartrate?from=2024-01-01T00:00:00&to=2024-12-31T23:59:59" -o heartrate.xml

# Export steps data
curl -u user:password "http://localhost:8080/api/export/steps?from=2024-01-01T00:00:00&to=2024-12-31T23:59:59" -o steps.xml
```

Available export types: `heartrate`, `steps`, `calories`, `distance`, `sleep`

The amount of heart rate data for an entire year can be too large for Apple Health to import in one go, so you may have
to divide the export data into chunks. See the script `export.sh` for an example.

## Architecture

The project consists of five modules:

- **model** - Shared JPA entities and Spring Data repositories
- **importer** - Importer library: base classes and domain importers (no `@SpringBootApplication`, plain jar)
- **importer-cli** - CLI runner for the importer (`@SpringBootApplication`, `ImportRunner`, executable jar)
- **server** - REST API, GraphQL server, resolvers, exporters, and REST import endpoint (depends on importer library)
- **dashboard** - SvelteKit web dashboard for visualizing Fitbit data

Dependencies:
- **importer** depends on model
- **importer-cli** depends on importer
- **server** depends on model and importer

## Tech Stack

- Kotlin 2.3.0 / JVM 25 / Spring Boot 3.4.4
- PostgreSQL 17 with JPA/Hibernate
- GraphQL + REST (custom controllers)
- SvelteKit 2 + Svelte 5 + TypeScript + URQL + TailwindCSS

## Link to the project

- <https://github.com/RikEnde/fitbit-kotlin>