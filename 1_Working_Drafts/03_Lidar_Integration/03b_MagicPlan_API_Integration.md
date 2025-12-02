# 3.2 MagicPlan API Integration

The successful acquisition of accurate spatial data is the foundational step upon which the entire recommendation system is built. This section details the process of integrating with the MagicPlan API, which serves as the primary source for LiDAR-scanned floor plans. The integration involves identifying the correct API endpoints, analyzing the complex data structure of the responses, and developing a robust process for extracting, transforming, and storing the relevant information.

### 3.2.1 API Endpoint Analysis

The system interacts with two principal RESTful endpoints provided by the MagicPlan API to retrieve user-specific plan data:

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

Due to the large size and complexity of the JSON response, a multi-step data extraction and transformation process was implemented to parse the raw data and map it to the system's database schema.

The process is as follows:
1.  **Request and Parse:** The backend sends a request to the `GET /plans/{planId}` endpoint and parses the resulting JSON string into a programmable object.
2.  **Iterate and Filter:** The system iterates through the `rooms` array and, for each room, subsequently iterates through its `fixtures` array. During this stage, only fixtures relevant to the bathroom renovation context (e.g., doors, windows, existing shower structures) are selected, while irrelevant objects (e.g., decorative furniture) are ignored.
3.  **Map to Database Schema:** The attributes from each relevant fixture object are then mapped to the corresponding columns in the database. For instance, the `symbolName` from the API is mapped to the `name` column in the `Fixtures` table.
4.  **Store Spatial Data:** The precise positional and dimensional data (e.g., `x`, `y`, `width`, `height`, `depth`, `angle`) is crucial for future spatial analysis and potential 3D visualization. To maintain flexibility, this granular data is collected into a JSON object and stored in a single `extrainfo` column in the `Fixtures` table. This approach avoids cluttering the main table with numerous columns and allows the data structure to evolve without requiring schema migrations.

This structured extraction process ensures that only necessary and clean data is persisted, making subsequent queries by the recommendation engine efficient and reliable.

---

### What to Write Next (Missing Information)

To make this section comprehensive for a Master's Thesis, you should add the following details:

1.  **Algorithmic Pseudo-code:** Provide a pseudo-code block that formally describes the data extraction and filtering algorithm. This will clearly illustrate the logic for iterating through the JSON and selecting relevant fixtures.

2.  **Detailed Error Handling:** How does the system handle potential issues?
    -   What happens if the API call fails or returns a non-200 status code?
    -   How does the parser handle a malformed or incomplete JSON response?
    *   What if the `exploded` or `fixtures` keys are missing from the data?

3.  **Data Cleaning Specifics:** You mentioned that the data requires "extensive cleaning." Provide concrete examples.
    *   Are there inconsistencies in `symbolName` that need to be normalized?
    *   Do you perform any validation on the dimension values (e.g., ensuring they are positive numbers)?

4.  **Performance Considerations:** Discuss the performance implications of processing such large JSON files.
    *   Did you evaluate different JSON parsing libraries for efficiency?
    *   How long does the extraction process take on average, and does it impact user experience?

5.  **A Visual Diagram:** A simple flow chart illustrating the data flow from `MagicPlan API` -> `Backend Extraction Service` -> `PostgreSQL Database` would significantly improve clarity.
