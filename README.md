# Payroll REST API

Payroll is a small Spring Boot REST API that models two common backend resources: employees and customer orders. It was built as a personal learning project to practice creating production-style API endpoints with persistence, hypermedia links, and clear resource state transitions.

The application starts with seed data, stores records through Spring Data JPA, and exposes JSON endpoints for creating, reading, updating, deleting, cancelling, and completing records.

## What this project demonstrates

- Built a Java 21 Spring Boot application with Maven.
- Designed RESTful controllers for `Employee` and `Order` resources.
- Used Spring Data JPA repositories to persist entities without writing manual SQL.
- Modeled domain objects with Jakarta Persistence annotations such as `@Entity`, `@Id`, `@GeneratedValue`, and `@Table`.
- Added HATEOAS responses with self links and collection links so API clients can discover related actions.
- Implemented order workflow rules with statuses: `IN_PROGRESS`, `COMPLETED`, and `CANCELLED`.
- Returned proper HTTP responses for resource creation, deletion, and disallowed state transitions.
- Loaded initial sample data at startup with a `CommandLineRunner`.
- Added centralized exception handling for missing employees.

## Tech stack

- Java 21
- Spring Boot 4
- Spring Web MVC
- Spring Data JPA
- Spring HATEOAS
- H2 in-memory database
- Maven Wrapper

## Project structure

```text
src/main/java/com/example/payroll
├── PayrollApplication.java          # Spring Boot entry point
├── LoadDatabase.java                # Seeds sample employees and orders
├── Employee.java                    # Employee JPA entity
├── EmployeeController.java          # Employee REST endpoints
├── EmployeeRepository.java          # Employee persistence layer
├── EmployeeModelAssembler.java      # Employee HATEOAS response links
├── Order.java                       # Order JPA entity
├── OrderController.java             # Order REST endpoints and workflow actions
├── OrderRepository.java             # Order persistence layer
├── OrderModelAssembler.java         # Order HATEOAS response links
└── Status.java                      # Order status enum
```

## Getting started

### Prerequisites

- Java 21 installed
- Git installed

You do not need to install Maven separately because this project includes the Maven Wrapper.

### Clone and run

```bash
git clone <your-repository-url>
cd payroll
./mvnw spring-boot:run
```

On Windows:

```bat
mvnw.cmd spring-boot:run
```

The API runs at:

```text
http://localhost:8080
```

### Run tests

```bash
./mvnw test
```

### Build the project

```bash
./mvnw clean package
```

### Run the packaged application

```bash
java -jar target/payroll-0.0.1-SNAPSHOT.jar
```

## Seed data

When the app starts, it preloads:

### Employees

| Name | Role |
| --- | --- |
| Bilbo Baggins | burglar |
| Frodo Baggins | thief |

### Orders

| Description | Status |
| --- | --- |
| Macbook Pro | COMPLETED |
| iPhone | IN_PROGRESS |

## API documentation

Base URL:

```text
http://localhost:8080
```

Responses use Spring HATEOAS, so records include `_links` with URLs for related resources and available actions.

## Employee endpoints

### Get all employees

```bash
curl http://localhost:8080/employees
```

Returns a collection of employees with links to each employee and the employee collection.

### Get one employee

```bash
curl http://localhost:8080/employees/1
```

Returns one employee by ID.

### Create an employee

```bash
curl -X POST http://localhost:8080/employees \
  -H "Content-Type: application/json" \
  -d '{"firstName":"Samwise","lastName":"Gamgee","role":"gardener"}'
```

Creates a new employee and returns `201 Created` with a `Location` header pointing to the new resource.

### Replace or create an employee by ID

```bash
curl -X PUT http://localhost:8080/employees/1 \
  -H "Content-Type: application/json" \
  -d '{"name":"Bilbo Baggins","role":"ring bearer"}'
```

Updates an existing employee if the ID exists. If the ID does not exist, the app saves the request body as a new employee.

### Delete an employee

```bash
curl -X DELETE http://localhost:8080/employees/1
```

Deletes the employee and returns `204 No Content`.

## Order endpoints

### Get all orders

```bash
curl http://localhost:8080/orders
```

Returns all orders. Orders with `IN_PROGRESS` status include `cancel` and `complete` links.

### Get one order

```bash
curl http://localhost:8080/orders/1
```

Returns one order by ID.

### Create an order

```bash
curl -X POST http://localhost:8080/orders \
  -H "Content-Type: application/json" \
  -d '{"description":"Mechanical keyboard"}'
```

Creates an order. New orders are automatically assigned `IN_PROGRESS` status by the API.

### Cancel an order

```bash
curl -X DELETE http://localhost:8080/orders/2/cancel
```

Cancels an order only when its current status is `IN_PROGRESS`. If the order has already been completed or cancelled, the API returns `405 Method Not Allowed` with a problem-details response.

### Complete an order

```bash
curl -X PUT http://localhost:8080/orders/2/complete
```

Completes an order only when its current status is `IN_PROGRESS`. If the order has already been completed or cancelled, the API returns `405 Method Not Allowed` with a problem-details response.

## Example response

```json
{
  "id": 2,
  "description": "iPhone",
  "status": "IN_PROGRESS",
  "_links": {
    "self": {
      "href": "http://localhost:8080/orders/2"
    },
    "orders": {
      "href": "http://localhost:8080/orders"
    },
    "cancel": {
      "href": "http://localhost:8080/orders/2/cancel"
    },
    "complete": {
      "href": "http://localhost:8080/orders/2/complete"
    }
  }
}
```

## Learning outcomes

This project helped me practice building a backend service beyond simple CRUD. I implemented REST endpoints, persistence with repositories, startup data loading, response modeling, and business rules around order status changes. I also learned how HATEOAS can make an API more self-descriptive by returning links to the actions a client can take next.

## Future improvements

- Add more complete tests for controller behavior and order workflow rules.
- Add validation for request bodies.
- Add centralized exception handling for missing orders.
- Add a persistent database profile for local development outside of H2.
- Add OpenAPI/Swagger documentation for interactive API exploration.
