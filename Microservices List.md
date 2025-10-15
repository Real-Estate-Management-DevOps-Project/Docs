**Final Microservices List**

**User/Identity Service:**

* Responsibilities: Manages user data, roles, and permissions within your system's database.

**Audit & Compliance Service:**

* Responsibilities: Provides endpoints for other services to send audit logs to. It will be responsible for writing these logs to the database.


**Property Registration Service:**

* Responsibilities: Handles all CRUD (Create, Read, Update, Delete) operations for properties, buildings, and land.

**Lease & Tenancy Management Service:**

* Responsibilities: Manages leases, rental agreements, and connects tenants (users) to properties. This service will need to communicate with the Property service and the User service.

**Notification Service:**

* Responsibilities: Sends reminders for rent, lease expiry, etc. It will be triggered by events from the Lease service.

**Technology Allocation (.NET , Spring Boot, React and Mysql)**

Based on our team's skills, here is the most logical and effective way to divide the work. This split plays to everyone's strengths.  
—----------------------------------------------------------------------------  
**Technology Stack**  
.NET

**Microservices to Build**  
1\. User/Identity Service  
2\. Audit & Compliance Service done 

**Justification**  
.NET's ASP.NET Core Identity is an industry-leading framework for security and user management. The Audit service is a high-performance, system-level concern that fits well with .NET's strengths in building robust infrastructure services.  
—---------------------------------------------------------------------------

**Technology Stack**  
Spring Boot	

**Microservices to Build**  
1\. Property Registration Service \<br\>   
2\. Lease & Tenancy Management Service \<br\>   
3\. Notification Service	

**Justification**  
These services represent the core business logic. Spring Boot is excellent for this, with powerful data tools (JPA/Hibernate) and a vast ecosystem for handling complex business rules and integrations (like scheduling for notifications).  
—-----------------------------------------------------------------------------------------------------------  
**Technology Stack**  
React	

**Microservices to Build**  
Frontend Application	

**Justification**  
**This is confirmed.** The frontend will be a completely separate Single Page Application (SPA) that communicates with all the backend microservices via an API Gateway.

—-------------------------------------------------------------------------------------------------------------------

**User/Identity Service**  
**.NET service will NOT handle passwords or the login process. Instead, it acts as a Resource Server and a User Profile Manager.**

| Responsibility | Who Handles It? | How it Works |
| :---- | :---- | :---- |
| **Showing Login Page** | **WSO2 Asgardio** | Your React frontend will redirect the user to a login page hosted by Asgardio. |
| **Validating Passwords** | **WSO2 Asgardio** | Your services will **never** see or store a user's password. This is a major security benefit. |
| **Issuing Tokens** | **WSO2 Asgardio** | After a successful login, Asgardio will issue a secure **JSON Web Token (JWT)** to the React frontend. |
| **Validating Tokens** | **ALL your Microservices (.NET & Spring Boot)** | The React app will send this JWT with every API request. Each of microservices must be configured to validate this token to ensure the request is authentic. |
| **Managing Roles & Permissions** | **WSO2 Asgardio & Your .NET Service (Shared)** | **Asgardio** is the source of truth for high-level roles (e.g., admin, user). These roles are embedded inside the JWT. **Your .NET service** can store *additional, application-specific* profile information that links to the Asgardio user. |
| **Storing User Profiles** | **Your .NET User/Identity Service** | It stores data like a user's full name, contact info. It acts as the internal representation of a user. |

