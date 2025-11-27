
# 2.4 Data Model Design

The product recommendation system's accuracy, performance, and future scalability are all directly impacted by a solid and scalable data model [cite Improving database design through the analysis of relationship]. The PostgreSQL database schema, which is designed to effectively process and store complex spatial data from LiDAR scans, manage a multifaceted product catalog, store user data for authentication, complex product relationships, and customer data for automation, is described in detail in this section. The main objective is to develop a logical framework that makes it easier to do the intricate queries needed by the rule-based and graph-based recommendation algorithms.

PostgreSQL was selected over MongoDB primarily because of its superior response time, which is essential when querying for a combination in products database; its ability to reduce overall dataset storage size, which is crucial for lowering hosting service costs for the company; and its reliable and automated backup management, which is offered as an option with many well-known hosting services. [cite makrisMongoDBVsPostgreSQL2021]

### 2.4.1 Core Data Entities and Their Relationships

The data model is structured around a set of core entities that create a logical division between user data, spatial information from scans, and the product catalog. This architectural choice is crucial for maintaining data integrity, enabling scalability, and optimizing query performance. The foundational entities are:

*   **User:** Represents an authenticated system operator. This entity holds credentials and role-based access control information (via the `isAdmin` flag), linking users to the plans and products they create. It is also crucial for managing concurrent usage across multiple users.

*   **Plan:** The `Plan` entity is structured to encapsulate a complete project scanned via the MagicPlan app. It serves as the top-level container for a specific customer's quote request, linked to a company `User` who created it. This design is crucial for associating the customer with the `Rooms` within the plan and for storing essential customer-related metadata (such as customer name and ID) and a thumbnail URL for UI representation.

*   **Room:** Represents a distinct spatial area within a `Plan`. It stores dimensional data (e.g., `areaWithInteriorWalls`) and serves as a container for all `Fixture` objects identified within its boundaries. This entity is crucial for filtering to specific contexts, such as bathrooms.

*   **Fixture:** Models a fixed installation or obstacle (e.g., window, door, existing bathtub) within a `Room`. Each fixture's dimensions and location are critical inputs for the recommendation algorithm, as they impose physical constraints on product placement.

*   **Product:** The foundational entity of the product catalog. It represents a core, purchasable item (e.g., a specific model of shower door, tray) and contains universal attributes like `modelNumber`, price, and base dimensions. It is linked to a `ProductType` for categorization.

*   **ProductType:** Provides a categorical framework for the product catalog (e.g., "Shower Door," "Side Panel," "Shower Tray"). This entity allows for logical grouping, filtering, and ensures the system is extensible to future product categories.

*   **Product-Specific Details (`DoorDetail`, `ShowerCabinDetail`, `ShowerTrayDetail`):** These entities are linked to the `Product` table via a one-to-one relationship. This architectural pattern avoids a bloated and sparse `Product` table by abstracting type-specific attributes (like `openingType` for a door or `style` for a cabin) into separate, specialized tables.

*   **ProductRelationship:** This is the most critical entity for the recommendation logic. It functions as a "join table" that connects two `Product` entities (`sourceProduct` and `targetProduct`), defining a specific `RelationshipType` between them. This structure effectively transforms the product catalog into a directed graph, where products are nodes and compatibility rules are the edges—the essential foundation for the graph-based algorithm.

### 2.4.2 Architectural Justification and Scalability

The schema is intentionally designed to be both robust and extensible. The separation of the `Product` entity from its type-specific details (`DoorDetail`, etc.) is a key example of normalization. This pattern ensures that the core `Product` table remains lean, while allowing for immense flexibility in defining attributes for new product types without requiring disruptive schema changes.

Furthermore, the design for scalability is most evident in the `ProductRelationship` model. By representing compatibility as explicit relationships in a dedicated table, the system can:
1.  **Scale to new compatibility rules:** Adding new relationship types (e.g., `REQUIRES_SCREWS`, `ACCESSORY_FOR`) can be achieved by simply adding a new value to the `RelationshipType` enum.
2.  **Support complex, graph-based queries:** The model is optimized for algorithms that traverse the product graph to find valid combinations, which is far more efficient and powerful than embedding such logic in application code.
3.  **Accommodate new product styles:** When a new product is added, its compatibility with the existing ecosystem is defined simply by inserting new rows into the `ProductRelationship` table, seamlessly integrating it into the recommendation engine.

This graph-based data architecture, combined with the normalized structure, provides a powerful and future-proof foundation for the intelligent recommendation system.
