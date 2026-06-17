# Questions

Here we have 3 questions related to the code base for you to answer. It is not about right or wrong, but more about what's the reasoning behind your decisions.

1. In this code base, we have some different implementation strategies when it comes to database access layer and manipulation. If you would maintain this code base, would you refactor any of those? Why?

**Answer:**
```
Yes, I would absolutely refactor the database access layer to enforce consistency and strict architectural boundaries. Currently, the codebase exhibits architectural leakage: the API adapter layer (WarehouseResourceImpl) bypasses the domain use-case boundary by interacting directly with the infrastructure layer (WarehouseRepository) to fetch and validate database entities (DbWarehouse). 

To fix this, I would implement the following refactoring strategy:
1. Enforce Domain Isolation: The API layer should interact exclusively with Domain Ports (interfaces). It should never know about "DbWarehouse" or the repository.
2. Unify Access Patterns via Repository Ports: I would introduce a domain-level repository interface (e.g., WarehouseRepositoryPort) and have the infrastructure Panache repository implement it.
3. Decouple Data Models: I would formalize the data mapping step. The database entity (DbWarehouse) should be strictly isolated to the infrastructure adapter. Data fetched from the database should be mapped immediately into a clean Domain Model (Warehouse) before being passed back through the ports to the domain or API layer. This eliminates mixed data manipulation responsibilities and ensures the core domain logic remains decoupled from the database framework.
```
----
2. When it comes to API spec and endpoints handlers, we have an Open API yaml file for the `Warehouse` API from which we generate code, but for the other endpoints - `Product` and `Store` - we just coded directly everything. What would be your thoughts about what are the pros and cons of each approach and what would be your choice?

**Answer:**
```
Both approaches have distinct trade-offs depending on project maturity and team structure:

Approach A: API-First / Code Generation (Warehouse API)
- Pros: Single source of truth. Enforces strong interface contracts between backend and frontend/external clients before a single line of code is written. Parallel development is easier, and client SDKs/documentation are updated automatically.
- Cons: Overhead in maintaining the YAML file, rigid compilation steps, and the code-gen scaffolding can sometimes introduce friction when rapidly prototyping small field changes.

Approach B: Code-First / Direct Implementation (Product and Store APIs)
- Pros: Maximum developer velocity and flexibility during early execution phases. There is no code-generation sync friction, and refactoring endpoints utilizing JAX-RS/Jakarta annotations is fast and intuitive.
- Cons: Documentation (Swagger) easily drifts from reality if not manually synchronized. Harder to coordinate changes across cross-functional teams, and API breaking changes are easier to accidentally introduce.

My Choice:
For a production-grade, collaborative monolith or microservice system, I would strictly choose the API-First (Code Generation) approach. While Code-First is tempting for initial speed, an explicit OpenAPI specification acts as a binding contract. It drastically reduces integration bugs, guarantees accurate documentation, and forces deliberate thought into API design changes before they break downstream clients.

```
----
3. Given the need to balance thorough testing with time and resource constraints, how would you prioritize and implement tests for this project? Which types of tests would you focus on, and how would you ensure test coverage remains effective over time?

**Answer:**
```
To optimize confidence while respecting strict time and resource constraints, I would follow a pragmatic testing pyramid adapted for Hexagonal Architecture:

1. Prioritization Strategy:
   - High Priority (Integration Tests): Since this is a Quarkus/Panache codebase, I would prioritize API/Slice Integration Tests using @QuarkusTest and REST-Assured. Testing the endpoints directly allows us to validate the entire request-response cycle, database transaction boundaries (@Transactional), and data mapping layers simultaneously. This catches the highest percentage of critical regressions per line of test code written.
   - Medium Priority (Domain Unit Tests): Core domain logic, boundary validations (e.g., capacity requirements), and business use-case invariants should be isolated and unit-tested using JUnit and Mockito to fake the outbound ports (like LocationResolver).
   - Lower Priority (Pure Component Unit Tests): I would avoid writing extensive unit tests for boilerplate code or pass-through services that contain no branching logic.

2. Ensuring Long-Term Effectiveness:
   - Automated Gatekeeping: Integrate a code coverage tool (like Jacoco) into the CI/CD pipeline, configuring a reasonable minimum coverage threshold (e.g., 75-80%) that fails builds if new code drops beneath it.
   - Focus on Behavior over Implementation: Test against the interface behavior (API endpoints and domain ports) rather than private implementation details. This ensures that future refactorings (such as replacing the database layer or changing repository strategies) can be performed safely without breaking the existing test suite.

```