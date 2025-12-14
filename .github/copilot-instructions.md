# Copilot Instructions for vProfile Project

## Architecture Overview
This is a Spring MVC web application for user account management, packaged as a WAR file deployed on Tomcat. It integrates with MySQL (accounts DB), RabbitMQ (messaging), Elasticsearch (search), and Memcached (caching). The app follows a layered architecture: Controllers handle HTTP requests, Services encapsulate business logic, Repositories manage data access via JPA/Hibernate.

Key components:
- **User Management**: Registration, login, profile updates (see `UserController.java`, `UserService.java`)
- **Messaging**: Publishes user events to RabbitMQ fanout exchange "messages" (see `ProducerServiceImpl.java`)
- **Search**: Indexes users in Elasticsearch "users" index (see `ElasticSearchController.java`)
- **Caching**: Uses Memcached for session/data caching (see `MemcachedUtils.java`)

Data flows: User actions trigger DB updates, message publishing, and search indexing. Configurations in `application.properties` use hostnames like `vprodb`, `vpromq01` for containerized deployments.

## Developer Workflows
- **Build**: `mvn clean install` (skips tests by default; use `-DskipTests=false` for full build)
- **Run Locally**: `mvn jetty:run` (starts embedded Jetty on port 8080)
- **Test**: `mvn test` (unit tests), `mvn verify` (integration tests with JaCoCo coverage)
- **Code Quality**: `mvn checkstyle:checkstyle` (enforces style rules)
- **Deploy**: Multi-stage Docker build (`Dockerfile`), Jenkins pipeline (`Jenkinsfile`) for CI/CD, Kubernetes manifests (`kubernetes/`), Ansible playbooks (`ansible/`)

Import DB schema from `src/main/resources/db_backup.sql` before running.

## Conventions and Patterns
- **Package Structure**: `com.visualpathit.account.{controller,service,repository,model,utils,validator}` – group related classes logically
- **Dependency Injection**: Use `@Autowired` for services in controllers (e.g., `UserController.java`)
- **Validation**: Custom validators like `UserValidator.java` for form validation
- **Configuration**: Externalize to `application.properties`; use utils like `RabbitMqUtil.java` for config access
- **Error Handling**: Basic try-catch in services (e.g., `ProducerServiceImpl.java`); no global exception handlers
- **Testing**: JUnit 4 with Mockito for mocking (see test classes in `src/test/java/`)

Avoid direct DB queries; use JPA repositories (e.g., `UserRepository.java`). For messaging, use fanout exchange for broadcast. Elasticsearch operations use transport client API.

## Integration Points
- **Database**: MySQL with JPA; connection pooling via Commons DBCP
- **Messaging**: RabbitMQ via Spring AMQP (but direct client in `ProducerServiceImpl.java`)
- **Search**: Elasticsearch 5.6.4 transport client for indexing/searching users
- **Caching**: Memcached via spymemcached library
- **Deployment**: Containerized with Docker/Tomcat; orchestrated via K8s/Helm/Ansible

Reference `pom.xml` for dependencies, `application.properties` for configs, and deployment files for infrastructure setup.