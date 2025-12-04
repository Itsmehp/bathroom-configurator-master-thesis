# 3.2 MagicPlan API Integration

The successful acquisition of accurate measurements is the foundational step upon which the entire recommendation system is built. This section details the process of integrating with the MagicPlan API, which serves as the primary source for LiDAR-scanned floor plans. The integration involves identifying the correct API endpoints, analyzing the complex data structure of the responses, and developing a robust process for extracting, transforming, and storing the relevant information.

### 3.2.1 API Endpoint Analysis

The system interacts with two principal RESTful endpoints provided by the MagicPlan API to retrieve user-specific plan data \cite{MagicplanCloudAPI}:

1.  **`GET /plans`**: This endpoint retrieves a paginated list of all plans associated with a user's account. Each item in the response provides summary data, including the unique `planId`, the project it belongs to, its name, and a `thumbnailUrl`. The `planId` and `thumbnailUrl` are used in the frontend application to allow the user to visually select which bathroom plan they wish to configure.

2.  **`GET /plans/{planId}`**: Once a user selects a plan, this endpoint is called using the specific `planId`. It returns a detailed, verbose JSON object representing the entire floor plan, including all its rooms, fixtures, and structural elements. This detailed response is the primary source for all measurement and fixture data used by the recommendation algorithms.

### 3.2.2 Analysis of the Plan Data Structure

The JSON response from the `GET /plans/{planId}` endpoint is substantial, with an average size of approximately 487 KB. The most critical data is nested within the `data.data` object, which contains both high-level plan details and a `plan_detail` object with the granular room-by-room breakdown. The system is designed to navigate this complex structure to extract only the most relevant information for processing.

### 3.2.3 Data Extraction and Transformation

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

4.  **Final Data Assembly:** The extracted `planId`, `name`, `thumbnailUrl`, and the processed array of `rooms` (each now containing its calculated area and unified `fixtures` list) are assembled into a clean, standardized `CleanedPlanData` object, which is then ready for persistence in the database. This multi-step, hybrid approach is essential for navigating the idiosyncrasies of the MagicPlan API and constructing a reliable data model.

---

### What to Write Next (Missing Information)

To make this section comprehensive for a Master's Thesis, you should add the following details:

1.  **Algorithmic Pseudo-code:** Provide pseudo-code for the main `extractPlanData` function. This should clearly show the sequence of operations: extracting basic info, calling the XML parser, looping through rooms, applying the area fallback logic, and processing both the `furnitures` and `wall_items` arrays.

2.  **XML Parsing Implementation:** Briefly explain the implementation of the `extractFloorAreasFromMagicplanXml` function. What library was used for XML parsing (e.g., `fast-xml-parser` in JavaScript), and how does it navigate the XML tree to locate and extract the area values?

3.  **Detailed Error Handling:** How does the extraction logic handle potential data integrity issues?
    -   What happens if the `magicplan_format_xml` string is present but malformed?
    -   How does the system behave if the `furnitures` or `wall_items` arrays are missing for a particular room? Does it create a room with no fixtures, or does it raise an error?

4.  **Data Mapping and Cleaning:** Provide a table or a more detailed description of the mapping from the raw API fields (e.g., `f.symbol.name`, `f.size.x`) to your application's fixture properties (`name`, `width`). Mention any data cleaning steps, such as trimming whitespace from names or validating that dimension values are positive numbers.
