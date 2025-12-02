# 3.2 MagicPlan API Integration

The successful acquisition of accurate spatial data is the foundational step upon which the entire recommendation system is built. This section details the process of integrating with the MagicPlan API, which serves as the primary source for LiDAR-scanned floor plans. The integration involves identifying the correct API endpoints, analyzing the complex data structure of the responses, and developing a robust process for extracting, transforming, and storing the relevant information.

### 3.2.1 API Endpoint Analysis

The system interacts with two principal RESTful endpoints provided by the MagicPlan API to retrieve user-specific plan data \cite{MagicplanCloudAPI}:

1.  **`GET /plans`**: This endpoint retrieves a paginated list of all plans associated with a user's account. Each item in the response provides summary data, including the unique `planId`, the project it belongs to, its name, and a `thumbnailUrl`. The `planId` and `thumbnailUrl` are used in the frontend application to allow the user to visually select which bathroom plan they wish to configure.

2.  **`GET /plans/{planId}`**: Once a user selects a plan, this endpoint is called using the specific `planId`. It returns a detailed, verbose JSON object representing the entire floor plan, including all its rooms, fixtures, and structural elements. This detailed response is the primary source for all measurement and fixture data used by the recommendation algorithms.

### 3.2.2 Analysis of the Plan Data Structure

The JSON response from the `GET /plans/{planId}` endpoint is substantial, with an average size of approximately 487 KB and often containing over 14,000 lines of nested data. The critical information for this project is located within the `data` key. This object contains a `rooms` array, which details all the spaces within the plan.

For each room, the most crucial element is the `fixtures` array. Each object within this array represents a distinct item scanned in the room—such as a window, door, or piece of furniture—and contains a rich set of attributes essential for our analysis:

-   **Identification:** A unique identifier (`uid`) for each fixture and a `symbolName` (e.g., "Flügelfenster," "Kleiner Tisch") that provides a human-readable description.
-   **Categorization:** A numeric `type` that classifies the object, allowing the system to distinguish between structural elements like doors and windows versus other objects like furniture.
-   **Dimensions:** The physical `width`, `height`, and `depth` of the fixture, measured in meters.
-   **Positioning:** A set of coordinates (`x`, `y`) and an `angle` of rotation that precisely define the object's location and orientation on the 2D floor plan.

### 3.2.3 Data Extraction and Transformation

Due to the large size and complexity of the JSON response, a multi-step data extraction and transformation process was implemented to parse the raw data and map it to the system's database schema. The high-level process is as follows:

1.  **Request and Parse:** The backend sends a request to the `GET /plans/{planId}` endpoint and parses the resulting JSON string into a programmable object.
2.  **Iterate and Filter:** The system iterates through the `rooms` array and, for each room, subsequently iterates through its `fixtures` array. During this stage, only fixtures relevant to the bathroom renovation context are selected.
3.  **Map to Database Schema:** The attributes from each relevant fixture object are then mapped to the corresponding columns in the database.
4.  **Store Spatial Data:** The precise positional and dimensional data is collected into a JSON object and stored in a single `extrainfo` column in the `Fixtures` table for flexibility.

#### Extraction Logic Overview

A detailed review of the data extraction logic, as illustrated in the flowcharts (see Appendix X), reveals a more nuanced process:

1.  **Hybrid Data Parsing (JSON and XML):** While most plan data is parsed from the main JSON object, the most accurate room area measurements are found within an embedded XML string at `data.data.plan_detail.magicplan_format_xml`. A dedicated function, `extractFloorAreasFromMagicplanXml`, parses this XML to create an indexed array of floor areas.

2.  **Area Assignment with Fallback:** When processing each room, the system first attempts to assign the area from the XML-derived `floorAreas` array. If a valid area is not found at the corresponding index, a fallback mechanism is triggered, which uses the `area_with_interior_walls_only` value from the room's JSON object. This ensures data robustness.

3.  **Consolidated Fixture Processing:** The raw data does not contain a single, unified `fixtures` array. Instead, objects are separated into distinct `furnitures` and `wall_items` arrays. The extraction process iterates through both arrays independently, maps their respective properties (e.g., `symbol.name`, `size.x`) to a standardized fixture format, and then consolidates them into a single `fixtures` array for each cleaned room object.

This multi-step, hybrid approach is necessary to handle the specific idiosyncrasies of the MagicPlan API response and to construct a clean, reliable data model for the recommendation engine.

---

### What to Write Next (Missing Information)

To make this section comprehensive for a Master's Thesis, you should add the following details:

1.  **Algorithmic Pseudo-code:** Provide pseudo-code for the main `extractPlanData` function. This should include the logic for calling the `extractFloorAreasFromMagicplanXml` helper function and the loops for processing `furnitures` and `wall_items`.

2.  **XML Parsing Details:** Briefly explain the implementation of the `extractFloorAreasFromMagicplanXml` function. What library was used for XML parsing (e.g., `fast-xml-parser`), and how does it navigate the XML structure to find the area values?

3.  **Detailed Error Handling:** How does the system handle potential issues within the extraction logic?
    -   What happens if the `magicplan_format_xml` string is missing or malformed?
    -   How does the system react if `furnitures` or `wall_items` arrays are missing or empty?

4.  **Data Cleaning Specifics:** Provide concrete examples of data cleaning performed during the mapping process.
    -   How are `symbol.name` values normalized (e.g., trimming whitespace, standardizing capitalization)?
    -   Is there validation to ensure that `size.x`, `size.y`, and `size.z` are valid numbers before they are mapped to width, depth, and height?

5.  **A Visual Diagram:** A simple flow chart illustrating the data flow from `MagicPlan API` -> `Backend Extraction Service` -> `PostgreSQL Database` would significantly improve clarity. The flowcharts you created in `4_Reference_Material/Flowcharts/` are an excellent source for this.
