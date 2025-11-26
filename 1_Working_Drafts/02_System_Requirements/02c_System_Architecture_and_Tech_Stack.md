# 2.3 System Architecture and Technology Stack

To address the critical limitations of the incumbent product configurator identified in Section 2.2.2—namely its closed architecture, lack of integration capabilities, and outdated data—this thesis proposes a modern three-tier system architecture. This design is explicitly engineered to provide real-time data integration, a flexible and open API, and automated data synchronization, directly solving the previously discussed operational bottlenecks. The architectural pattern separates the system into presentation, application, and data layers, enabling modularity, scalability, and maintainability.

### 2.3.1 Architectural Design and Rationale

The system is structured into three distinct tiers, as illustrated in Figure [INSERT FIGURE NUMBER HERE: System Architecture Diagram]. This separation allows each tier to be developed, scaled, and maintained independently.

**The Presentation Tier (Frontend):** This tier is responsible for the user interface and client-side experience. It directly addresses the "B2B Exclusivity" and poor usability of the old configurator by providing a modern, responsive interface suitable for direct use in client consultations.
-   **Key Responsibilities:** Rendering product recommendations, visualizing room layouts from MagicPlan data, and capturing user inputs for configuration.
-   **Technology:** Next.js with TypeScript.

**The Application Tier (Backend):** This tier acts as the central nervous system of the application, directly solving the "No Integration Capabilities" and "Closed Architecture" problems of the incumbent system. It serves as an integration hub, decoupling the frontend from the data sources and external services.
-   **Key Responsibilities:**
    -   Exposing a secure RESTful API for the frontend client.
    -   Consuming room dimension and fixture data from the MagicPlan API.
    -   Executing the core business logic, including the rule-based and graph-based recommendation algorithms.
    -   Implementing the logic for ancillary product recommendations, such as compatible shower trays, to solve the "Lack of Shower Tray Recommendation" gap.
    -   Orchestrating automated data-sourcing tasks.
-   **Technology:** Express.js with TypeScript, with Python for specialized automation scripts.

**The Data Tier (Database):** This tier provides a single source of truth for all application data, ensuring integrity and eliminating the data fragmentation and staleness issues of the previous workflow.
-   **Key Responsibilities:** Persistently storing the full product catalog, complex component compatibility rules, processed MagicPlan scan data, and user information.
-   **Technology:** PostgreSQL.

### 2.3.2 Technology Stack Justification

The technology stack was selected to build a robust, modern, and maintainable application that directly counteracts the deficiencies of the prior system.

-   **Frontend (Next.js & TypeScript):** Next.js, a React framework, was chosen for its capabilities in server-side rendering (SSR) and static site generation (SSG), ensuring high performance and a professional user experience. Its component-based architecture is ideal for building a complex product configurator. TypeScript adds static typing, which is crucial for maintaining a large codebase and ensuring clear data contracts with the API, reducing runtime errors.

-   **Backend (Express.js & TypeScript):** Express.js, a minimalist web framework for Node.js, was selected to build the flexible and powerful RESTful API that the incumbent system lacked. Its unopinionated nature provides the freedom to design the API structure precisely to this project's needs. TypeScript ensures reliability and clear documentation for the API endpoints.

-   **Database (PostgreSQL):** A powerful open-source object-relational database system, PostgreSQL was chosen for its robustness, ACID compliance, and strong support for complex queries and data relationships. This is essential for modeling the intricate compatibility rules between different shower enclosure components (doors, side panels, etc.) and for enabling features like shower tray recommendations.

-   **Automation (Python):** Python was selected for the backend automation and data scraping tasks due to its extensive ecosystem of libraries for web scraping (e.g., BeautifulSoup, Scrapy) and data manipulation. This choice directly enables the automated jobs that solve the "Outdated Product Data" and "No Real-Time Price Visibility" problems by keeping the product catalog and pricing information current.

-   **Containerization (Docker):** Docker is used to containerize the entire application stack. This ensures a consistent development, testing, and deployment environment, mitigating risks associated with environment discrepancies and simplifying the deployment process.

### 2.3.3 Data Flow and Automation

The architecture is designed to support a seamless flow of data, from initial room scan to final product recommendation.

1.  **Data Acquisition:** A user performs a room scan using the MagicPlan app. The dimensional and fixture data is retrieved via an API call from the Application Tier.
2.  **Processing & Logic:** The Application Tier processes this spatial data. The recommendation engine then queries the PostgreSQL database for compatible products based on the room's constraints and the system's embedded compatibility rules.
3.  **Recommendation:** A curated list of suitable products is sent to the Presentation Tier and displayed to the user for final selection.

To ensure data is always current, two primary automated background jobs run on the backend:
-   **Product Ingestion Service:** A Python script that can ingest new product data from manufacturer sources, populating the PostgreSQL database and keeping the product catalog complete.
-   **Price Update Service:** A Python script that periodically scrapes partner portals to update pricing for existing products in the database, ensuring that all quotes are based on real-time data.

This automated, API-driven architecture stands in stark contrast to the manual, fragmented, and error-prone workflow of the incumbent system.