# Realty Flats Service

Flats catalog for the [realty agency platform](https://github.com/foliageh/realty-agency-cloud):
a Jakarta EE 11 REST API on Payara with dynamic filtering, multi-field sorting, paging and a
cancellable background job. The agency service and the Mule ESB layer call it over HTTPS.

**Stack:** Java 25 · Jakarta EE 11 (JAX-RS, CDI, Bean Validation, JPA) · Jakarta Data · Payara 7 · PostgreSQL · OpenAPI 3 · Maven

## API

| Method | Path | What it does |
|---|---|---|
| `POST` | `/flats` | Create a flat (validated DTO) |
| `GET` `PUT` `DELETE` | `/flats/{id}` | Read, update, delete |
| `POST` | `/flats/filter?sortBy=price,area&sortDirection=asc,desc&pageNumber=0&pageSize=20` | Filter by name, area, rooms, price, furnish, transport and balcony; sort by several fields; page |
| `GET` | `/flats/rooms-greater-than/{rooms}` | Flats with more rooms than given |
| `DELETE` | `/flats/by-furnish/{furnish}` | Bulk delete by furnish type |
| `POST` `GET` `DELETE` | `/flats/unique-living-spaces` | Start, poll and cancel a background job that collects unique living spaces |

The full contract is in [`api-docs.yaml`](api-docs.yaml) (OpenAPI 3).

## Design notes

- **Jakarta Data repository.** `FlatRepository` extends `BasicRepository` and declares queries with
  `@Query`. One query handles every optional filter, and `PageRequest` plus `Order` give paging and
  sorting by any number of fields.
- **Validation and errors.** Bean Validation on DTOs and entities, plus exception mappers for
  validation, not found, wrong method and unexpected errors, all returning one JSON error format.
- **Background job.** The long query runs in a `CompletableFuture` on a dedicated executor. Only
  one job runs at a time. The client polls for the result (`202 Accepted` until it is ready) and
  can cancel the job.
- **HTTPS.** Payara listens on 8182 with a self-signed certificate. `configure-payara.bat` sets up
  the JDBC pool, the resource and the TLS listener.
- **API tests.** `test-api.ps1` and `test-api.bat` run the main scenarios against a live server.

## Branch `lab3`

The [`lab3`](../../tree/lab3) branch splits the service into an EJB module with remote business
interfaces and a thin web module, configures the stateless bean pool, and registers the service in
Consul for discovery.

## Run locally

```bash
mvn package
configure-payara.bat C:\payara7          # JDBC pool, resource, HTTPS listener
asadmin deploy target/itmo-soa-lab2-1.0.war
```

---
Built as coursework for the Service-Oriented Architecture course at ITMO University, 2025.
