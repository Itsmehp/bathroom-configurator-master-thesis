# Master Thesis

**Rule-based shower selector for Bathroom Renovations Using LiDAR Room Scans**

This is the main index for the Master Thesis.

Executive Summary:

Traditional bathroom renovation quotation processes are time-consuming, error-prone, and require extensive manual work by sales teams to measure spaces, select compatible products, verify installability, and calculate pricing. This inefficiency creates delays for customers and limits business scalability. The challenge was to develop an AI-powered system that automatically generates complete, accurate bathroom renovation quotes from LiDAR room scans captured via smartphone, eliminating manual measurement errors and reducing quote generation time from hours to minutes.

Overall Structure and Procedure
The developed prototype is a full-stack web application built on a modern technology stack: a Next.js 14 frontend with TypeScript for the user interface, an Express.js backend with TypeScript for business logic, and PostgreSQL with Prisma ORM for data management. The system architecture follows a three-tier approach:

Data Acquisition Layer: Integration with MagicPlan API enables the system to import LiDAR-scanned bathroom floor plans captured using iOS devices. The raw scan data contains precise 3D measurements of room dimensions and existing fixtures (bathtubs, showers, toilets, sinks).

Product Intelligence Layer: A comprehensive database manages over 500+ sanitary products with detailed specifications including dimensions, prices, installation constraints (min/max  construction width ranges), styles (NICHE, ECKEINSTIEG, U-FORM, WALKIN), orientations (LEFT, RIGHT, UNIVERSAL), and compatibility relationships. The system employs sophisticated rule-based algorithms to recommend optimal product combinations.

Recommendation Engine: Four specialized search algorithms handle different shower cabin configurations:

NICHE Best Algorithm: Finds optimal single-door solutions for alcove installations
ECKEINSTIEG Best Algorithm: Matches corner entry systems with compatible side panels
U-Form Best Algorithm: Handles complex U-shaped installations requiring multiple doors and panels
Walk-In Best Algorithm: Recommends frameless walk-in shower solutions
Each algorithm implements multi-stage filtering: strict width matching (minEinbau/maxEinbau ranges), extended tolerance matching (±50mm fallback), price range validation, and compatibility verification through a relationship graph connecting doors, side panels, and shower trays.

Relationship mapping: The heart of the selection process, finds products that are related to door for when an algorithm needs two or three products to complete the configuration. The product mapping can be door to door or door to side panel with multiple types of relationship type.

Quote Generation Workflow: Users select a scanned floor plan, which auto-fills measurements from detected fixtures. The system displays all available products matching the dimensional constraints, while the Rule-Based recommendation engine suggests the optimal combination within the specified price range. The final selection generates a structured bill of materials with itemized pricing, total calculations, and product specifications exportable as JSON or can be also called using APIs.

Findings and Results
Technical Achievement: The prototype successfully demonstrates automated selection of shower enclosure for bathroom renovations based on LiDAR scans. The system processes MagicPlan APIs for floor plans with precision also cleaning and saving it to database, validates product installability against real-world constraints, and generates complete shower enclosure configuration based on the representatives selection of shower type in under 30 seconds—a 95% reduction compared to manual processes.(Ask)

Algorithm Performance: The recommendation algorithms achieve high accuracy through a cascading fallback strategy. The ECKEINSTIEG algorithm, for example, searches through 1,500+ product combinations per width iteration, testing compatibility relationships and dimensional constraints. When strict matching fails, the system automatically extends tolerance ranges and clearly communicates deviations to users, maintaining transparency in product selection.

Real-World Validation: Testing with 25+ actual bathroom floor plans demonstrated the system's ability to handle diverse scenarios including compact bathrooms (700mm x 900mm), standard installations (1200mm x 900mm), and large custom designs (1800mm x 1200mm). The system successfully identified optimal product combinations in 100% of test cases, with clear "no match found" messages for edge cases outside product catalog coverage.

Database Integrity: The implemented data model includes comprehensive validation through Prisma schema constraints and Zod middleware, ensuring data consistency across 8 core entities (Users, Plans, Products, ProductTypes, ProductRelationships, ShowerCabinDetails, DoorDetails, UFormConfigurations). Cross-field validation prevents invalid combinations such as minEinbau exceeding maxEinbau. The data model also boasts fully scalable model in case the company wants to expand to toilets, washbasins and more in the future. The possibility of adding more products and configurations from different companies products is also available.

User Experience: The multilingual interface (English/German) provides intuitive navigation through a multi-step configuration wizard. Real-time product filtering, visual product cards with images, dimensional tolerances, and price calculations create a professional quote generation experience. Product management dashboards enable administrators to maintain the catalog through CSV uploads and manual editing with full i18n support.

Scalability and Extensibility: The modular architecture supports easy addition of new product categories, shower styles, and recommendation algorithms. RESTful API design with Swagger documentation enables future integration with ERP systems, e-commerce platforms, and mobile applications. Docker containerization facilitates deployment across development, staging, and production environments.

Limitations and Future Work: The current prototype relies on rule-based algorithms rather than machine learning models. Future enhancements could incorporate AI/ML techniques for predicting customer preferences based on historical selections, automated price optimization, and visual similarity matching. Integration with 3D visualization tools would enable customers to preview their bathroom design before finalizing the quote.

The delivered prototype validates the feasibility of AI-assisted quote generation for bathroom renovations, providing a solid foundation for production deployment and demonstrating significant efficiency gains over traditional manual quotation processes.
## Table of Contents

-   [[01_Introduction]]
-   [[02_System_Requirements_and_Architecture_Designing]]
-   [[03_Lidar_Integration]]
-   [[04_Intelligent_Product_Recommendation_Algorithm]]
-   [[05_System_Implementation]]
-   [[06_Testing_and_Evaluation]]
-   [[07_Discussion_of_Findings]]
-   [[08_Conclusion]]
-   [[09_Bibliography]]
-   [[10_Appendix]]
