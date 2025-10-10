
# Wikimedia With Kafka - Advanced Configurations

This repository contains several example modules that demonstrate how to integrate Wikimedia EventStreams with Apache Kafka. Each module focuses on a different use case or learning goal: from basic producers and consumers to stream processing and an OpenSearch sink. The documentation below explains the purpose of each module, how to build and run them, and important configuration tips.

## Table of Contents

- [Project Overview](#project-overview)
- [Repository Modules](#repository-modules)
- [Common Prerequisites](#common-prerequisites)
- [How to build and run](#how-to-build-and-run)
- [Module details and quick-start](#module-details-and-quick-start)
- [Kafka advanced configurations (summary)](#kafka-advanced-configurations-summary)
- [Troubleshooting and tips](#troubleshooting-and-tips)
- [Contributing and License](#contributing-and-license)

## Project Overview

The goal of this project is to provide hands-on examples of streaming Wikimedia change events into Apache Kafka and then processing or consuming them. The modules show practical configurations for reliability, throughput, and maintainability. You can use them for learning, demos, or as a starting point for production systems.

## Repository Modules

This repository is split into several modules and example projects. Each module lives in its own folder at the repository root:

- `kafka-beginners-course/` — Simple exercises for Kafka beginners. Contains step-by-step examples and small Java programs that explain basic Kafka concepts. Good for learning producer and consumer fundamentals.
- `kafka-basics/` — Minimal examples showing how to produce and consume messages with basic configurations. Use this module to test local Kafka connectivity and simple workflows.
- `kafka-consumer-opensearch/` — A consumer example that writes Kafka messages to OpenSearch (or Elasticsearch). Includes a `docker-compose.yml` to run Kafka and OpenSearch together for local testing.
- `kafka-producer-wikimedia/` — Producer application that connects to Wikimedia EventStreams and publishes events into Kafka topics. Shows advanced producer tuning and error handling.
- `kafka-streams-wikimedia/` — An example that uses Kafka Streams for lightweight stream processing of Wikimedia events (filtering, aggregation, or enrichment demos).
- `src/` — Shared source code and common utilities used across some modules (for example `io/conduktor/...`). This can include helper classes, serializers, or shared configuration loaders.

## Common Prerequisites

Before running the modules, make sure you have the following installed on your machine:

- Java 8+ (OpenJDK or Oracle JDK)
- Gradle (the repository includes Gradle wrapper scripts `gradlew` and `gradlew.bat` so you do not need a global Gradle install)
- Apache Kafka (you can use a cloud Kafka service or run locally via Docker)
- Docker and Docker Compose (recommended for local integration tests, especially for the OpenSearch consumer)

If you use macOS (as in this workspace), ensure Docker Desktop is installed and running before starting Docker Compose stacks.

## How to build and run

General steps that apply to most modules:

1. Clone the repository and move to the root folder:

   ```bash
   git clone https://github.com/juliocesarcs2004/Wikimedia-With-Kafka-Advanced-Configurations.git
   cd Wikimedia-With-Kafka-Advanced-Configurations
   ```

2. Build a module (from the repository root) using the Gradle wrapper. Replace `<module>` with the folder name (for example `kafka-producer-wikimedia`):

   ```bash
   ./gradlew :<module>:build
   ```

3. Run a module (common options):

   - To run with Gradle (preferred for examples):

     ```bash
     ./gradlew :<module>:run
     ```

   - Or run the generated jar: (after `build`)

     ```bash
     java -jar <module>/build/libs/<module>-all.jar
     ```

Notes:

- Some modules are simple Java programs (they use `main()`), others are Spring Boot / Kafka applications and may use `bootRun` or `run` tasks.
- Replace `<module>` with the folder name listed in "Repository Modules".

## Module details and quick-start

Below are short, clear descriptions and quick-start commands for each folder in the repository.

### kafka-beginners-course/

Purpose: Small, guided exercises that introduce Kafka concepts. Each example is short and focuses on a single topic (producer, consumer, partitioning, etc.).

Quick start:

1. Build: `./gradlew :kafka-beginners-course:build`
2. Run an example class with Gradle: `./gradlew :kafka-beginners-course:run` or run the specific class using your IDE.

What to expect: Console logs showing produced messages and consumers receiving them. Useful for step-by-step class demonstrations.

### kafka-basics/

Purpose: Minimal producer and consumer examples to test a local Kafka cluster or cloud broker quickly.

Quick start:

1. Build: `./gradlew :kafka-basics:build`
2. Run: `./gradlew :kafka-basics:run`

Notes: Change the broker address in the module's config or pass it as an environment variable when you run the examples.

### kafka-consumer-opensearch/

Purpose: Demonstrates how to consume messages from Kafka and index them into OpenSearch. Includes a Docker Compose setup for local testing.

Files to check:

- `docker-compose.yml` — Brings up Kafka, Zookeeper, and OpenSearch for development.
- Consumer source under `src/main/java` — Contains the logic to transform and index messages.

Quick start (local):

1. Start Docker Compose: `docker-compose -f kafka-consumer-opensearch/docker-compose.yml up -d`
2. Build and run consumer: `./gradlew :kafka-consumer-opensearch:run`

Expectations: Messages consumed from Kafka should appear in OpenSearch indices. Use OpenSearch Dashboards (if included) or curl to verify indexed documents.

### kafka-producer-wikimedia/

Purpose: Connects to Wikimedia EventStreams (SSE) and publishes change events to Kafka topics. Shows production-ready patterns: retries, batching, and error handling.

Main points:

- Connects to the Wikimedia EventStreams endpoint using an HTTP client.
- Parses EventStream messages and publishes to a configured Kafka topic.
- Has configurable producer settings (see `application.properties` or module config).

Quick start:

1. Ensure Kafka is running (Docker or remote broker)
2. Build: `./gradlew :kafka-producer-wikimedia:build`
3. Run: `./gradlew :kafka-producer-wikimedia:run`

Notes: Provide broker address, topic name and any authentication via environment variables or config file before running.

### kafka-streams-wikimedia/

Purpose: Example of stream processing with Kafka Streams. This module shows filtering, windowed aggregation, or simple enrichment of Wikimedia events.

Quick start:

1. Build: `./gradlew :kafka-streams-wikimedia:build`
2. Run: `./gradlew :kafka-streams-wikimedia:run`

Notes: Kafka Streams applications need unique `application.id` configuration per instance to manage state stores and consumer groups.

### src/

Purpose: Shared code and helpers. Look here for common serializers, DTOs, or utilities used by multiple modules.

When to change: If you improve a shared serializer or add a new helper, update this folder and rebuild dependent modules.

## Kafka advanced configurations (summary)

This repository contains examples of advanced Kafka settings. Here is a short summary and why they are important:

- `acks=all` — Guarantees that the leader waits for full ISR acknowledgement, increasing durability.
- `retries` and `retry.backoff.ms` — Help recover from temporary network or broker errors.
- `linger.ms` and `batch.size` — Improve throughput by batching messages.
- `compression.type` — Reduces network usage and storage size (common choices: `snappy`, `gzip`, `lz4`).
- `enable.auto.commit=false` (consumer) — Gives manual control of offsets to ensure messages are processed before committing.

Adjust these settings depending on your priorities (latency vs throughput vs durability).

## Troubleshooting and tips

- If a module cannot connect to Kafka: check broker address, firewall, and if Docker Compose is running the containers.
- For `kafka-consumer-opensearch`: watch OpenSearch logs and ensure the index templates mapping fits the consumed messages.
- Use small batches and `linger.ms` during testing to avoid high memory usage.
- When testing with local Docker stacks, allow a short time for services to become healthy before starting producers/consumers.

## Contributing and License

Contributions are welcome. If you want to add a new example or fix a bug:

1. Fork the repository
2. Create a feature branch
3. Add tests or a short README for new modules
4. Open a pull request with a clear description of changes

License: MIT License

---

Author: [juliocesarcs2004](https://github.com/juliocesarcs2004)
