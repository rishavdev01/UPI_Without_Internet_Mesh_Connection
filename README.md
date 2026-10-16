# UPI Without Internet: Mesh Connection

A Spring Boot backend that explores one question: **can a UPI payment still go through when the sender has no internet?**

The idea: the sender's phone encrypts the payment and passes it to nearby phones over a Bluetooth-style mesh. Each phone forwards it onward until one of them gets a connection and uploads it to this server. The server decrypts it, checks that it's genuine and not a duplicate, and settles it.

There's no real Bluetooth here. The mesh is simulated in software, so you can watch the whole flow on one laptop through a web dashboard.

## Why this is interesting

Three problems have to be solved for this to be safe:

| Problem | How this project handles it |
|---|---|
| Strangers' phones carry your payment | Hybrid encryption (RSA + AES-GCM), so relays can't read or modify it |
| The same packet can reach the server through several phones | Idempotency check on the packet hash, so it settles exactly once |
| A packet could be tampered with or replayed | Rejected before it ever touches the ledger |

## Tech stack

- Java 17+
- Spring Boot 3.3.5 (REST APIs + embedded Tomcat)
- Thymeleaf (demo dashboard)
- Maven (via the included wrapper, nothing to install)

## Getting started

**Requirement:** JDK 17 or newer (`java -version` to check).

**Mac / Linux**

```bash
chmod +x mvnw      # only needed once, if you get "permission denied"
./mvnw spring-boot:run
```

**Windows**

```cmd
mvnw.cmd spring-boot:run
```

The first run downloads Maven and the dependencies, so give it a couple of minutes. When you see `Started UpiMeshApplication`, open:

**http://localhost:8080**

Stop the server with `Ctrl+C`.

## Running the tests

```bash
./mvnw test
```

The most important test is `IdempotencyConcurrencyTest`. It sends the same packet from several threads at once and checks that it settles only one time.

## Project structure

```
src/main/java/com/demo/upimesh/
├── config/        App configuration
├── controller/    REST API and dashboard controllers
├── crypto/        Hybrid encryption and server key handling
├── model/         Account, Transaction, MeshPacket and repositories
├── service/       Ingestion, idempotency, settlement, mesh simulator
└── UpiMeshApplication.java

src/main/resources/
├── application.properties
└── templates/dashboard.html
```

## Limitations

This is a demo, not a payment system. A real version would need things like real Bluetooth transport, hardware-backed keys on phones, proper authentication, and integration with actual banking rails.
