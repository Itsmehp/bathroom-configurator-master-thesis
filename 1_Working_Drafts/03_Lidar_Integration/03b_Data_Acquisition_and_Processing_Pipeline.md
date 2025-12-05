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

*This section will explain the implementation of an autofill feature for fixture recognition, including the ability for users to override the automated suggestions. The algorithms detailed in the previous section for processing `furnitures` and `wall_items` form the basis of this automated recognition. The system maps raw names like "Bathtub" or "Shower" to internal system categories, which then autofill the relevant measurement fields in the user interface, streamlining the configuration process.*

The raw fixtures names that are recieved from magicplan api (e.g Eckdusche, Badewanne) needs to be standardized internal categories that the database queries can understand when requesting measurements for these fixtures. but such a system was not implemented instead we used this system "const prefillFromPlan = (plan: DetailedPlan) => {

    // Prefill from fixtures in the plan

    const badewanne = plan.fixtures?.find((f) =>

      f.name?.toLowerCase().includes("badewanne")

    );

    const dusche = plan.fixtures?.find((f) =>

      f.name?.toLowerCase().includes("dusche")

    );

  

    if (badewanne) {

      setMeasurements((prev) => ({

        ...prev,

        badewanne: {

          width: ((badewanne.width || 0) * 100).toString(),

          depth: ((badewanne.depth || 0) * 100).toString(),

        },

      }));

    }

  

    if (dusche) {

      setMeasurements((prev) => ({

        ...prev,

        dusche: {

          width: ((dusche.width || 0) * 100).toString(),

          depth: ((dusche.depth || 0) * 100).toString(),

        },

        wunschDusche: {

          width: ((dusche.width || 0) * 100).toString(),

          depth: ((dusche.depth || 0) * 100).toString(),

        },

      }));

    }

  };"
 to just query the database if the fixtures are present and convert them to cm and fill them up in the frontend.
this method was chosen for its simplicity since the project scope was only aimed towards german market the app configures name that contain german names for shower and bathtub that is dusche and badewanne respectively. currently the app has never found English words in all plans that were provided. the names are mostly stored as: Badewanne, Eckdusche 2,Rechteckige Dusche, Duschtür. 
### 3.2.4 Data Persistence

*This section will describe the pipeline for processing floor plan data, including the analysis of the JSON structure, the algorithms used for fixture detection, and the methods for data storage. The final step in the pipeline is storing the processed floor plan data. This involves saving the structured `CleanedPlanData` object, including all rooms and their recognized fixtures, into the PostgreSQL database. This ensures the data is readily available for the recommendation algorithms without needing to re-process the raw API response for every user session, thereby improving system performance and creating a persistent record of the project.*

To ensure system performance, speed and create a presistent record for each project, the processed plan data is stored in the database. This avoids the need for repeated and slow api calls to magicplan for every user session. structure of cleanPlanData:
export interface CleanedPlanData {

  planId: string | null;

  name: string | null;

  thumbnailUrl: string | null;

  rooms: CleanedRoom[];

}
export interface CleanedFixture {

  name: string | null;

  width: number | null;

  depth: number | null;

  height: number | null;

}

  

export interface CleanedRoom {

  area_with_interior_walls: number | null;

  fixtures: CleanedFixture[];

}

.
this storage ensures a well structured and accessible fixtures.


---

### What to Write Next (Missing Information)

To make this section comprehensive for a Master's Thesis, you should add the following details:

1.  **Algorithmic Pseudo-code:** Provide pseudo-code for the main `extractPlanData` function. This should clearly show the sequence of operations: extracting basic info, calling the XML parser, looping through rooms, applying the area fallback logic, and processing both the `furnitures` and `wall_items` arrays.

2.  **XML Parsing Implementation:** Briefly explain the implementation of the `extractFloorAreasFromMagicplanXml` function. What library was used for XML parsing (e.g., `fast-xml-parser` in JavaScript), and how does it navigate the XML tree to locate and extract the area values?

3.  **Detailed Error Handling:** How does the extraction logic handle potential data integrity issues?
    -   What happens if the `magicplan_format_xml` string is present but malformed?
    -   How does the system behave if the `furnitures` or `wall_items` arrays are missing for a particular room? Does it create a room with no fixtures, or does it raise an error?

4.  **Data Mapping and Cleaning:** Provide a table or a more detailed description of the mapping from the raw API fields (e.g., `f.symbol.name`, `f.size.x`) to your application's fixture properties (`name`, `width`). Mention any data cleaning steps, such as trimming whitespace from names or validating that dimension values are positive numbers.


implementation flowchart

in my app set magicplan api keys

user uses magicplan to generate a bathroom floor plan

magicplan handles the data and genrates a bathroom floor plan with accurate measurements.

My app gives user all the plans to select from.

user seelects a plan and saves the plan (saves fixtures, plan, room)

User uses this plan in bathroom renovation tab. where he can select the plan

currently 2 fixtures will be found using keywords dusche and bade , shower and bathtub, from the stored plan in the db.

the user will be shown measurements of bathtub and shower and will alredy have desired measurement set same as shower.
