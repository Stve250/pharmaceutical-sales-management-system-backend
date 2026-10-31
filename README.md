Pharmaceutical Sales Management System — Backend
A REST API for managing pharmaceutical wholesale sales to pharmacies, including sales representatives, territories, product lines, orders, delivery notes, and reporting.
Academic information
- Student: Steven Ndayizeye
- Student ID: 27165
- Course: INSY 8322 — Web Technology and Internet
- Institution: Adventist University of Central Africa (AUCA)
- Project: Mid-Term Examination, Academic Year 2026–2027
- Java 27 (as configured in pom.xml)
- Spring Boot 4.1.1 (as configured in pom.xml)
- Spring Web MVC
- Spring Data JPA / Hibernate
- PostgreSQL
- Maven Wrapper
- Postman for API testing
Architecture
The application follows a layered structure:
Postman / HTTP client
        ↓
Controller — REST endpoints and HTTP responses
        ↓
Service — application logic and business rules
        ↓
Repository — Spring Data JPA and JPQL queries
        ↓
PostgreSQL database
Java source code is under src/main/java/auca/pharma/rw/, with controller, service, repository, and domain packages.
Domain model
Entity	Purpose
ProductLine	Categories of pharmaceutical products
Territory	Geographic areas assigned to sales representatives
SalesRepresentative	Staff handling orders, assigned to a territory and multiple product lines
Pharmacy	Pharmacy customers, identified by a unique licence number
SalesOrder	Orders with dates, amounts, pharmacy, representative, and status
DeliveryNote	Records of products delivered for a delivered sales order


The SalesOrderStatus enum defines PLACED, APPROVED, DELIVERED, and CANCELLED.

Relationships include SalesRepresentative → Territory (many-to-one), SalesRepresentative ↔ ProductLine (many-to-many), SalesOrder → Pharmacy and SalesOrder → SalesRepresentative (many-to-one), and DeliveryNote → SalesOrder (many-to-one). Entity IDs are generated UUIDs.
Setup and run
Prerequisites
1. Install a JDK compatible with the Java version configured in pom.xml (Java 27).
2. Install and start PostgreSQL.
3. Use VS Code, IntelliJ IDEA, or another Java IDE.
4. Install Postman to test the API.
Database
Create a PostgreSQL database named:
CREATE DATABASE pharmaceutical_sales_db;
Configure src/main/resources/application.properties with credentials for your own local PostgreSQL installation. Do not commit real passwords to GitHub.
spring.application.name=pharmaceutical-sales-management-system
spring.datasource.url=jdbc:postgresql://localhost:5432/pharmaceutical_sales_db
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
server.port=8080
The example above uses environment variables for credentials. Set DB_USERNAME and DB_PASSWORD before starting the app. The source project currently contains locally configured database credentials; replace them before publishing.
Start the API
From the project root:

Windows:
.\mvnw.cmd spring-boot:run
macOS / Linux:
./mvnw spring-boot:run
The API runs at http://localhost:8080 if the configured port is available.
API endpoints
Standard CRUD
The following paths support POST /save, GET /all, GET /{id}, PUT /update/{id}, and DELETE /delete/{id}:
Resource	Base path
Product lines	/api/product-lines
Territories	/api/territories
Sales representatives	/api/sales-representatives
Pharmacies	/api/pharmacies
Sales orders	/api/sales-orders


Searches and business operations
Method	Endpoint	Description
GET	/api/pharmacies/by-district?district=gasabo	Pharmacies by district (case-insensitive)
GET	/api/sales-orders/by-status?status=PLACED	Orders by status
GET	/api/sales-orders/between?start=2026-10-01&end=2026-10-31	Orders in inclusive date range
GET	/api/sales-representatives/by-product-line?name=antibiotics	Representatives promoting a product line
GET	/api/territories/free	Unassigned territories
GET	/api/product-lines/unused	Unassigned product lines
GET	/api/pharmacies/of-sales-representative/{salesRepresentativeId}	Pharmacies served by a representative
PATCH	/api/sales-orders/{id}/status?status=APPROVED	Change order status
PUT	/api/sales-representatives/{salesRepresentativeId}/product-lines/{productLineId}	Add a product line
PATCH	/api/sales-representatives/{salesRepresentativeId}/territory?territoryNumber=4	Change territory
POST	/api/delivery-notes/save?salesOrderId={id}	Create delivery note
GET	/api/delivery-notes/all	List delivery notes
GET	/api/delivery-notes/pharmacy/{licenceNumber}	Delivery history by pharmacy
DELETE	/api/delivery-notes/delete/{id}	Delete delivery note
GET	/api/sales-orders/stats/by-status	Order counts by status
PATCH	/api/sales-orders/cancel-day?salesRepresentativeId={id}&date=2026-10-20	Bulk-cancel eligible orders for a day
GET	/api/dashboard/summary	Dashboard summary
GET	/api/sales-orders/page?page=0&size=5&sort=orderDate,desc	Paginated sales orders


The date values shown in GET requests are examples for querying existing records, not recommended dates for creating new orders.
Business rules
- New sales orders cannot be dated before today.
- A representative cannot have another non-cancelled order on the same date when creating an order.
- New orders start with PLACED status regardless of the supplied status.
- Allowed status changes: PLACED → APPROVED or CANCELLED; APPROVED → DELIVERED or CANCELLED.
- DELIVERED and CANCELLED are final statuses.
- A delivery note can be created only for a DELIVERED order; its delivery date is set by the service.
- An order with delivery notes cannot be deleted until those notes are deleted.
- A sales representative can hold multiple product lines, but adding the same one again is rejected.
- Territory changes reject a territory already assigned to another representative.
The API uses HTTP statuses such as 200, 201, 400, 404, and 409 to distinguish successful operations, invalid input, missing resources, and conflicts.
Postman testing
Import the collection in the project root:

pharmaceutical-sales-management-system-midterm.postman_collection.json

Set the base URL to http://localhost:8080, start the backend, and create test data in this order:
1. Product lines
2. Territories
3. Pharmacies
4. Sales representatives
5. Sales orders
6. Delivery notes (after the relevant orders reach DELIVERED)
