# **Cyber Security** (18 ECTS)
#CyberSec 
## **Knowledge – The student has knowledge of:**

- The conceptual difference between hashing and encryption, including when each technique is appropriate.
- The principles behind JWT-based authentication, including the structure of tokens, claims, expirations, and trust relationships.
- The purpose of HTTPS/TLS and how encrypted transport protects data in transit.
- The core differences between symmetric and asymmetric encryption and their typical use cases.
- How SQL injection attacks occur, and why parameterized queries and ORM frameworks mitigate this risk.

---
## **Skills – The student can:**

- Implement secure password storage by applying BCrypt hashing and verification within backend services.
- Configure and apply JWT authentication, including token generation, validation, and refresh token rotation.
- Use secure data-access patterns in EF Core to prevent SQL injection vulnerabilities.
- Document and propose the application of symmetric and asymmetric encryption for protecting images and geolocation data.
- Apply principles of secure communication by describing how HTTPS/TLS should be integrated into a backend architecture.

---
## **Competencies – The student can:**

- Assess security risks related to storing and transmitting GDPR-sensitive data such as license plates, images, and GPS coordinates, and propose appropriate mitigations.
- Design secure data flows that combine hashing, token-based authentication, encrypted transport, and planned at-rest encryption.
- Reflect on current security limitations in the prototype and articulate well-reasoned improvement strategies (e.g., endpoint authorization, TLS enforcement, report signing).
- Communicate security-related design decisions to stakeholders and explain how they support evidence integrity and data protection.
- Independently acquire new security techniques and apply them to project-specific challenges in backend development.

---
# **Data Storage / Databases** (12 ECTS)
#Data 
## **Knowledge – The student has knowledge of:**

- The structure of relational databases, including tables, relationships, constraints, and delete behaviors.
- The principles behind EF Core’s code-first approach and how migrations are used to manage schema evolution.
- ACID properties and how transactions support data consistency and reliability.
- Key considerations when comparing database storage of large files with external file/blob storage solutions.
- The role of repository and Unit of Work patterns in structuring data access within a modular backend architecture.

---
## **Skills – The student can:**

- Implement and configure data models in EF Core, including relationships, constraints, and entity configuration.
- Use EF Core migrations to create and evolve a relational database over the course of the project.
- Perform CRUD operations through repository patterns and maintain consistency through transaction boundaries.
- Document the data model and explain how it supports project functionality such as scans, reports, and image handling.
- Develop and justify proposed storage strategies for large files based on system requirements.

---
## **Competencies – The student can:**

- Evaluate how data-model and storage decisions impact system performance, maintainability, and data integrity.
- Design improvements to the current schema and storage approach, such as introducing audit trails, history tracking, or external file storage.
- Reflect on how EF Core and repository-based access contribute to modularity, testability, and scalability.
- Relate database design decisions to broader system needs, including documentation, evidence handling, and secure data processing.
- Independently learn new database concepts and apply them to practical backend solutions as the system scales or changes.