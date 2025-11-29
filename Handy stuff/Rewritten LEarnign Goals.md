# **Rewritten Learning Goals — Cyber Security**

_(Centered on what you implemented + what you explored via AES/RSA demos)_

## **Knowledge — The student has knowledge of:**

- The difference between **hashing** and **encryption**, and why hashing (BCrypt) is required for secure password storage.
    
- The structure and purpose of **JWT authentication**, including access tokens, refresh tokens, claims, and token rotation.
    
- How **symmetric encryption (AES)** and **asymmetric encryption (RSA)** work in C#, including the strengths and limitations of each.
    
- How **TLS/HTTPS** protects API communication and why it is required for transmitting sensitive data such as GPS coordinates and images.
    
- How **SQL injection works**, and why EF Core’s parameterized LINQ-based queries mitigate this risk.
    

---

## **Skills — The student can:**

- Implement secure password hashing and verification using **BCrypt** within C# backend services.
    
- Integrate and manage **JWT authentication flows**, including issuing, validating, and rotating tokens stored in the database.
    
- Demonstrate encryption concepts in C# using **AES for symmetric encryption** and **RSA for asymmetric encryption**, showing understanding of real-world usage.
    
- Apply secure coding practices in database interactions using EF Core, preventing SQL injection and unsafe data access patterns.
    
- Evaluate when encryption-at-rest or encryption-in-transit is required and design appropriate approaches for protecting sensitive app data.
    

---

## **Competencies — The student can:**

- Assess security risks related to storing and sending sensitive data (images, plates, GPS) and choose appropriate mitigations such as HTTPS, JWT, hashing, and encryption.
    
- Design improvements for production-grade security, including enforcing HTTPS, securing endpoints, and introducing encryption for sensitive content.
    
- Reflect on limitations of the current prototype and outline how to correctly implement encryption, access control, and secure storage in the next iteration.
    
- Communicate security decisions to stakeholders clearly and relate them to practical project needs, such as evidence integrity and data protection.
    
- Independently acquire new security techniques (e.g., encryption, digital signatures) and apply them in real backend development scenarios.
    

---

# ✅ **Rewritten Learning Goals — Data Storage / Databases**

_(Purely based on your EF Core implementation + schema + migrations)_

## **Knowledge — The student has knowledge of:**

- Fundamental relational database concepts as implemented with **EF Core**, including entities, relationships, constraints, and delete behaviors.
    
- How **code-first EF Core migrations** are used to evolve and maintain database schema over the lifetime of a project.
    
- The basics of **transactional behavior** in EF Core, including how SaveChanges creates transactional boundaries and ensures data integrity.
    
- Trade-offs between storing binary data (images) directly in a relational database versus external storage such as file systems or blob storage.
    
- How the **Repository + Unit of Work** patterns create cleaner architecture for modular backend systems.
    

---

## **Skills — The student can:**

- Build and refine a relational data model using EF Core, including configuring relationships, owned types, and delete rules.
    
- Use EF Core migrations to create and update tables that support domain features such as scans, reports, users, images, and refresh tokens.
    
- Implement CRUD operations using repositories backed by EF Core while maintaining consistent transactional behavior.
    
- Evaluate storage strategies for large media files and justify the chosen approach for the prototype versus production needs.
    
- Document database structure and explain how it supports the parking-app workflow (scanning, storing, exporting, auditing).
    

---

## **Competencies — The student can:**

- Evaluate how database design choices affect long-term maintainability, performance, and integrity of critical data such as scan evidence.
    
- Plan future improvements to schema design, such as separating large media storage, adding audit histories, or optimizing data retrieval.
    
- Justify the use of EF Core within a modularized backend architecture and relate the chosen patterns (Repository/UoW) to testability and system clarity.
    
- Reflect on how database design interacts with application features like PDF generation, exporting evidence, and secure data flow.
    
- Learn and apply new data-storage techniques as project requirements scale (e.g., blob storage, indexing, partitioning).