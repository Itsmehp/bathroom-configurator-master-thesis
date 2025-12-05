# 3.2 Data Acquisition and Processing Pipeline

The successful acquisition of accurate measurements is the foundational step upon which the entire recommendation system is built. This section details the complete pipeline for integrating with the MagicPlan API, from fetching raw data to processing it into a structured format suitable for the recommendation engine. The process involves identifying the correct API endpoints, analyzing the complex data structure, transforming the raw data, and classifying fixtures.

### 3.2.1 Interfacing with the MagicPlan API

The system interacts with two principal RESTful endpoints provided by the MagicPlan API to retrieve user-specific plan data \cite{MagicplanCloudAPI}:

1.  **`GET /plans`**: This endpoint retrieves a paginated list of all plans associated with a user's account. Each item in the response provides summary data, including the unique `planId`, the project it belongs to, its name, and a `thumbnailUrl`. The `planId` and `thumbnailUrl` are used in the frontend application to allow the user to visually select which bathroom plan they wish to configure.

2.  **`GET /plans/{planId}`**: Once a user selects a plan, this endpoint is called using the specific `planId`. It returns a detailed, verbose JSON object representing the entire floor plan, including all its rooms, fixtures, and structural elements. This detailed response is the primary source for all measurement and fixture data used by the recommendation algorithms.

### 3.2.2 Raw Data Transformation and Cleaning

The JSON response from the `GET /plans/{planId}` endpoint is substantial, with an average size of approximately 487 KB. The most critical data is nested within the `data.data` object, which contains both high-level plan details and a `plan_detail` object with the granular room-by-room breakdown. The system is designed to navigate this complex structure to extract only the most relevant information for processing.

Due to the size and complexity of the API response, a multi-step data extraction and transformation process was implemented to parse the raw data and map it to the system's database schema. This process is handled by a dedicated `extractPlanData` function.

The extraction logic proceeds as follows:

1.  **Basic Plan Information Extraction:** First, high-level plan details are extracted from the `data.data.plan` object, including the `id` (as `planId`), `name`, and `thumbnail_url`.

2.  **Hybrid Room Area Processing:** The process for determining room area employs a robust hybrid strategy that combines data from two different sources within the API response for maximum accuracy.
    *   **Primary Source (XML):** The system's primary source for area data is an embedded XML string located at `data.data.plan_detail.magicplan_format_xml`. This XML is parsed by a helper function, `extractFloorAreasFromMagicplanXml`, which returns an indexed array of the most accurate floor area values.
    *   **Fallback Mechanism (JSON):** In cases where the XML data is missing or invalid for a specific room, the system implements a fallback. It retrieves the area from the `area_with_interior_walls_only` property within the standard room object, located at `data.data.plan_detail.plan.floors[0].rooms`. This ensures that an area value is always available if possible.

3.  **Consolidated Fixture Processing:** A significant complexity in the raw data is that fixtures are not stored in a single list. Instead, they are split across two separate arrays within each room object: `furnitures` and `wall_items`. The system processes these as follows:
    *   It iterates through the `furnitures` array, mapping each object to a standardized fixture format. For example, the fixture's name is sourced from `f.symbol.name` and its width from `f.size.x`.
    *   It then performs the same iteration and mapping process for the `wall_items` array.
    *   Finally, the two resulting lists of mapped objects are concatenated into a single, unified `fixtures` array for the room.

4.  **Final Data Assembly:** The extracted `planId`, `name`, `thumbnailUrl`, and the processed array of `rooms` (each now containing its calculated area and unified `fixtures` list) are assembled into a clean, standardized `CleanedPlanData` object, which is then ready for persistence in the database. It is important to note that a deprecated `extraInfo` column previously existed in the database table for plans. This column stored spatial data in JSON format, including fixture size, position, rotation, and exterior wall coordinates (e.g., `[x1,y1]` and `[x2,y2]`). While initially used for testing spatial analysis, this column is no longer utilized due to limitations that will be discussed later in the system implementation phase. This multi-step, hybrid approach is essential for navigating the idiosyncrasies of the MagicPlan API and constructing a reliable data model.

### 3.2.3 Automated Fixture Recognition and Classification

A primary goal of the data processing pipeline is to identify specific fixtures from the raw MagicPlan data (e.g., "Eckdusche", "Badewanne") so they can be used by the recommendation engine. Rather than implementing a complex classification system during data ingestion, a more pragmatic approach was chosen due to the project's initial focus on the German market.

The chosen method is a simple and effective keyword-matching algorithm. Raw fixture names are stored in the database without modification. When a user selects a plan in the frontend application, the system logic scans the list of fixture names for specific German substrings—"dusche" for showers and "badewanne" for bathtubs. If a match is found, the system uses that fixture's dimensions to pre-fill the relevant measurement fields for the user. This approach was selected for its simplicity and has proven robust for the current scope, as raw fixture names like "Rechteckige Dusche" or "Duschtür" are correctly identified.

### 3.2.4 Data Persistence

To ensure system performance and create a persistent record for each project, the processed plan data is stored in a database. This strategy avoids the need for repeated, costly API calls to MagicPlan for every user session. Before persistence, the data is organized into a standardized structure, conceptually known as `CleanedPlanData`, which ensures that all information is consistent and easily accessible.

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


---

### What to Write Next (Missing Information)

This section has a strong conceptual foundation. To make it comprehensive for a Master's Thesis, you should now add the following implementation details:

1.  **XML Parsing Implementation:** You mention using a helper function, `extractFloorAreasFromMagicplanXml`. Based on the code provided, this function uses Regular Expressions instead of a formal XML parsing library. You should briefly explain this implementation:
    *   Justify the choice to use regex. Was it for performance, simplicity, or to handle the specific, non-standard nature of the embedded XML string?
    *   Describe how the regex patterns (`/<floor.../g` and `/<plan.../i`) are designed to specifically target and extract the `areaWithInteriorWallsOnly` and `interiorWallWidth` attributes while ignoring other XML content.

2.  **Detailed Error Handling Strategy:** Your system uses a multi-layered approach to handling errors. You should describe these layers:
    *   **Proactive API Health Check:** Explain how the system's health check endpoint monitors the MagicPlan API's status to preemptively flag connection issues.
    *   **Robust API Response Handling:** Describe how your backend routes (e.g., `/api/magicplan/:planId`) validate input and translate upstream errors from the MagicPlan API or invalid client requests into meaningful HTTP status codes (like 400, 401, 404, 409) for the frontend.
    *   **Data-Level Fault Tolerance:** The `extractMagicplanXmlAttributes` function includes a `try...catch` block that intentionally "swallows" parsing errors. Justify this design choice. Does this ensure the data pipeline can continue processing a plan even if parts of the XML are malformed, by defaulting to `null` values?

***Note on Completed Items:***
*   **Algorithmic Pseudo-code:** As you noted, this will be covered in Chapter 5 (System Implementation). This point is now resolved for this section.
*   **Data Mapping:** The Mermaid flowchart you provided is an excellent way to present the data mapping from the API to your database schema. It clearly and visually documents the process and is suitable for inclusion in your thesis. This point is now resolved.
