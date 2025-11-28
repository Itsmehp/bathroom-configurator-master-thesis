# 2.3 System Architecture and Technology Stack

To address the critical limitations of the incumbent product configurator identified in Section 2.2.2—namely its closed architecture, lack of integration capabilities, and outdated data—this thesis proposes a modern three-tier system architecture. This design is explicitly engineered to provide real-time data integration, a flexible and open API, and automated data synchronization, directly solving the previously discussed operational bottlenecks. The architectural pattern separates the system into presentation, application, and data layers, enabling modularity, scalability, and maintainability.

### 2.3.1 Architectural Design and Rationale

The system is structured into three distinct tiers, as illustrated in Figure [INSERT FIGURE NUMBER HERE: System Architecture Diagram]. This separation allows each tier to be developed, scaled, and maintained independently.

**The Presentation Tier (Frontend):** This tier is responsible for the user interface and client-side experience. It directly addresses the "B2B Exclusivity" and poor usability of the old configurator by providing a modern, responsive interface suitable for direct use in client consultations.
-   **Key Responsibilities:** Rendering product recommendations, visualizing room layouts from MagicPlan data, and capturing user inputs for configuration.
-   **Technology:** Next.js with TypeScript.

**The Application Tier (Backend):** This tier acts as the central nervous system of the application, directly solving the "No Integration Capabilities" and "Closed Architecture" problems of the incumbent system. It serves as an integration hub, decoupling the frontend from the data sources and external services.
-   **Key Responsibilities:**
    -   Exposing a secure, well-documented RESTful API to be consumed by the frontend client and other authorized third-party applications.
    -   Handling user authentication and authorization to support multi-user access.
    -   Consuming room dimension and fixture data from the MagicPlan API.
    -   Executing the core business logic for product matching, which primarily employs rule-based and graph-based recommendation algorithms (rather than machine learning models), complemented by advanced search logic, such as suggesting dimensionally-adjusted alternatives when no products meet an initial price constraint.
    -   Implementing the logic for ancillary product recommendations, such as compatible shower trays, to solve the "Lack of Shower Tray Recommendation" gap.
    -   Orchestrating automated data-sourcing tasks.
    -   Use of NLP search, where the user just need to write the type of shower they need with or without dimensions and price range, and they are shown a product based on the input.
-   **Technology:** Express.js with TypeScript, with Python for specialized automation scripts.

**The Data Tier (Database):** This tier provides a single source of truth for all application data, ensuring integrity and eliminating the data fragmentation and staleness issues of the previous workflow.
-   **Key Responsibilities:** Persistently storing the full product catalog, complex component compatibility rules, processed MagicPlan scan data, and user information.
-   **Technology:** PostgreSQL.

### 2.3.2 Technology Stack Justification

The technological stack was chosen to provide a reliable, up-to-date, and maintainable application that directly addresses the shortcomings of the previous system.

-   **Frontend (Next.js & TypeScript):** Next.js, a React framework, was chosen for its capabilities in server-side rendering (SSR) and static site generation (SSG), ensuring high performance and a professional user experience. Its component-based architecture is ideal for building a complex product configurator. TypeScript adds static typing, which is crucial for maintaining a large codebase and ensuring clear data contracts with the API, reducing runtime errors.

-   **Backend (Express.js & TypeScript):** To provide the adaptable and potent RESTful API that the existing system lacked, Express.js, a simple web framework for Node.js, was chosen. Because of its unbiased nature, the API structure can be specifically designed to meet the objectives of this project. For the API endpoints, TypeScript guarantees consistency and concise documentation.

-   **Database (PostgreSQL):** PostgreSQL, an open-source object-relational database system (ORDBMS), was selected due to its resilience, ACID compliance, and dependable support for complex queries and data interactions [makrisMongoDBVsPostgreSQL2021]. This is necessary to enable features like shower tray recommendations and to mimic the complex compatibility requirements between various shower enclosure components (doors, side panels, etc.).

-   **Automation (Python):** Because Python has a large ecosystem of libraries for data manipulation and web scraping (like Selenium), it was chosen for the backend automation and data scraping chores. By maintaining the product catalog and pricing data up to date, this option immediately supports the automated jobs that address the "Outdated Product Data" and "No Real-Time Price Visibility" issues.

-   **Containerization (Docker):** The entire application stack is containerized using Docker. This reduces the risks associated with inconsistent environments and streamlines the deployment process by maintaining a constant environment throughout development, testing, and deployment.

### 2.3.3 Data Flow and Automation

The architecture is designed to support a seamless flow of data, from initial room scan to final product recommendation.

1.  **Data Acquisition:** A user performs a room scan using the MagicPlan app. The dimensional and fixture data is retrieved via an API call from the Application Tier.
2.  **Processing & Logic:** The Application Tier extracts key measurements from the processed spatial data, such as the dimensions of existing fixtures. The recommendation engine then queries the PostgreSQL database for compatible products based on the room's constraints and the system's embedded compatibility rules.
3.  **Recommendation:** A curated list of suitable products is sent to the Presentation Tier and displayed to the user for final selection. This API-centric approach has already proven its value, as the backend is currently being used by the company's proprietary quote-generation software to retrieve curated product lists, demonstrating the flexibility that was absent in the previous system.

To ensure data is always current, two primary automated background jobs run on the backend:
-   **Product Ingestion Service:** A Python script that can ingest new product data from manufacturer sources, populating the PostgreSQL database and keeping the product catalog complete.
-   **Price Update Service:** A Python script that periodically scrapes partner portals to update pricing for existing products in the database, ensuring that all quotes are based on real-time data.

This automated, API-driven architecture stands in stark contrast to the manual, fragmented, and error-prone workflow of the incumbent system.



