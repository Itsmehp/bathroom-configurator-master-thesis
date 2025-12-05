# 3.2 Data Acquisition and Processing Pipeline

The successful acquisition of accurate measurements is the foundational step upon which the entire recommendation system is built. This section details the complete pipeline for integrating with the MagicPlan API, from fetching raw data to processing it into a structured format suitable for the recommendation engine. The process involves identifying the correct API endpoints, analyzing the complex data structure, transforming the raw data, and classifying fixtures.

### 3.2.1 Interfacing with the MagicPlan API

The system interacts with two principal RESTful endpoints provided by the MagicPlan API to retrieve user-specific plan data \cite{MagicplanCloudAPI}:

1.  **`GET /plans`**: This endpoint retrieves a paginated list of all plans associated with a user's account. Each item in the response provides summary data, including the unique `planId`, the project it belongs to, its name, and a `thumbnailUrl`. The `planId` and `thumbnailUrl` are used in the frontend application to allow the user to visually select which bathroom plan they wish to configure.

2.  **`GET /plans/{planId}`**: Once a user selects a plan, this endpoint is called using the specific `planId`. It returns a detailed, verbose JSON object representing the entire floor plan, including all its rooms, fixtures, and structural elements. This detailed response is the primary source for all measurement and fixture data used by the recommendation algorithms.

### 3.2.2 Raw Data Transformation and Cleaning

The JSON response from the `GET /plans/{planId}` endpoint is substantial, with an average size of approximately 487 KB based on an analysis of a sample of 365 floor plans, and serves as the primary source of truth for all critical data used in the system. The system is designed to navigate this complex, nested structure to extract only the most relevant information for processing. A dedicated `extractPlanData` function handles this multi-step data extraction and transformation.

The simplified data flow, from API endpoint to the core database entities, is illustrated in Figure 3.1.

*[PLACEHOLDER: Insert new simplified Mermaid Chart as Figure here]*

**Figure 3.1:** This figure illustrates oversimplified data extraction pipeline. It shows how key metadata and structural information are sourced exclusively from the JSON payloads of the `GET /plans` and `GET /plans/{planId}` endpoints and mapped directly to the `Plan`, `Room`, and core `Fixture` database tables.

The extraction logic proceeds as follows:

1.  **Basic Plan Information Extraction:** High-level plan details are extracted from the `data.data.plan` object within the JSON response, including the `id` (as `planId`), `name`, and `thumbnail_url`.

2.  **Room Area Extraction:** For each room object in the JSON, the system extracts the value from the `area_with_interior_walls_only` property to serve as the room's area measurement.

3.  **Consolidated Fixture Processing:** A key challenge presented by the raw JSON data is that fixtures are not located in a single, unified list. Instead, they are split across two separate arrays within each room object: `furnitures` and `wall_items`. The system processes these by iterating through both arrays, mapping each object to a standardized fixture format, and concatenating them into a single, unified `fixtures` list for each room.

4.  **Note on Legacy Data Formats:** The API response also contains a secondary, embedded XML string (`magicplan_format_xml`). While this field was explored during initial development for certain values like `areaWithInteriorWallsOnly`, the final implementation relies exclusively on the more structured and reliable JSON data for all core entities. The data from the XML source is not used in the production system.

5.  **Final Data Assembly:** The extracted `planId`, `name`, `thumbnailUrl`, and the processed array of `rooms` (each now containing its calculated area and unified `fixtures` list) are assembled into a clean, standardized `CleanedPlanData` object, ready for persistence in the database.

### 3.2.3 Automated Fixture Recognition and Classification

A primary goal of the data processing pipeline is to identify specific fixtures from the raw MagicPlan data (e.g., "Eckdusche", "Badewanne") so they can be used by the recommendation engine. Rather than implementing a complex classification system during data ingestion, a more pragmatic approach was chosen due to the project's initial focus on the German market.

The chosen method is a simple and effective keyword-matching algorithm. Raw fixture names are stored in the database without modification. When a user selects a plan in the frontend application, the system logic scans the list of fixture names for specific German substrings—"dusche" for showers and "badewanne" for bathtubs. If a match is found, the system uses that fixture's dimensions to pre-fill the relevant measurement fields for the user. This approach was selected for its simplicity and has proven robust for the current scope, as raw fixture names like "Rechteckige Dusche" or "Duschtür" are correctly identified.

### 3.2.4 Data Persistence

To ensure system performance and create a persistent record for each project, the processed plan data is stored in a database. This strategy avoids the need for repeated, costly, and time-consuming API calls to MagicPlan for every user session. Before persistence, the data is organized into a standardized structure, conceptually known as `CleanedPlanData`, which ensures that all information is consistent and easily accessible.

This data model is composed of the following nested entities:

*   **CleanedPlanData**: The root object representing a single floor plan. It contains:
    *   A unique `planId`.
    *   A user-assigned `name`.
    *   A `thumbnailUrl` for a preview image.
    *   A list of all associated `Room` objects.

*   **CleanedRoom**: An object representing a single room within the plan. It contains:
    *   The calculated `area_with_interior_walls`.
    *   A list of all `Fixture` objects located within that room.

*   **CleanedFixture**: An object representing a single fixture. It contains:
    *   The original, unprocessed `name` from the API.
    *   The dimensional properties: `width`, `depth`, and `height`.

This hierarchical data model ensures that all relevant information is captured in a structured format, ready for efficient storage and subsequent retrieval by the recommendation algorithms.

### 3.2.5 System Robustness and Error Handling
To ensure reliability, the system incorporates a multi-layered error handling strategy, addressing potential failures at the API, data, and service levels.

*   **API Response Handling:** The system's own backend endpoints provide a robust error-handling layer between the client and the external MagicPlan API. As defined in the API specification, endpoints validate incoming requests and translate any upstream API errors or invalid parameters into clear HTTP status codes (e.g., 400 for a bad request, 401 for authorization failure, 409 for a resource conflict). This ensures the frontend client receives predictable and actionable feedback.

*   **Data-Level Fault Tolerance:** At the data processing level, the function responsible for parsing the embedded XML string (`extractMagicplanXmlAttributes`) is wrapped in a `try...catch` block. This is a deliberate design choice to "swallow" parsing errors. If the XML string is malformed or missing, the function returns `null` instead of halting the entire operation. This fault-tolerant approach ensures that the system can still process and save the remainder of a plan's data, even if one part is corrupted.

*   **Proactive Service Monitoring:** A dedicated health check endpoint is included in the system. This endpoint can be used to monitor the connectivity and status of critical external services like the MagicPlan API, allowing for proactive identification of upstream issues that could impact system functionality.
