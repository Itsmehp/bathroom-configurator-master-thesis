# System Implementation

## Backend Implementation

1.      API Architecture

2.      Route Organization (all endpoints, including scrapers)

3.      Services Organization

4.      Middleware Architecture (RBAC)

5.      Swagger Documentation

## Database Implementation

1.      Prisma Schema Design

2.      Migration

3.      Seeding Admin

4.      Data Integrity

## Frontend Implementation

1.      Page Components

2.      Reusable UI components

3.      State management using the Zustand store

4.      Internationalization: Support for English and German languages

## UI explanation

1.      Step-by-step configuration wizard

2.      Product management and product relationship dashboard (CSV upload, bulk addition of relationships)

3.      Responsive design

4.      Quote generation and export:

a.      Selected products, quantities, prices, and customer information (from plan metadata).

b.      Bill of Materials Structure

c.      Export Formats

5.      Authentication (JWT, bcrypt hashing, RBAC, session management)

6.      Product Management Features

a.      CSV Bulk Upload: Parsing Product Data, Validation, Error Reporting, and Partial Success Handling

b.      Individual Product Editing: Inline Editing, Modal Dialog Forms, and Optimistic Updates

c.      Product Search and Filtering: Server-side pagination, debounced search, and multi-criteria filtering

d.      Product Relationship Management: User interface for defining compatible products and selecting relationship types.


  Under `Backend Implementation`, add a new subsection:

   * `5.1.X MagicPlan Data Integration Service`
       * Connect to Chapter 3: Start by referencing your design chapter. "As conceptually outlined in
         Section 3.2, a dedicated backend service was implemented to manage the data pipeline from the
         MagicPlan API."
       * Show the Code: This is the perfect place to include a code snippet of your extractPlanData
         function.
       * Detail the Fixture Recognition: Write: "The keyword-based fixture classification algorithm
         (Section 3.2.3) was implemented in TypeScript using a Map for efficient lookups. The following
         code snippet shows the mapping from raw strings to system-defined enums:" Then, show the code
         for your mapping object.
       * Justify Technology Choices: Mention specific libraries. "The embedded XML in the API response
         was parsed using the fast-xml-parser library, chosen for its high performance and robustness
         against malformed data."

  Under `Database Implementation`, you can now elaborate:

   * `5.2.1 Prisma Schema Design`
       * Connect to Chapter 3: "The conceptual data model described in Section 3.2.4 was implemented
         using a PostgreSQL database with Prisma as the Object-Relational Mapper (ORM)."
       * Show the Schema: This is where you present the actual code from your schema.prisma file for
         the Plan, Room, and Fixture models.
   * `5.2.4 Data Integrity`
       * Discuss the `extraInfo` column: You can elaborate on the limitations you mentioned.
         "Initially, a JSON field named extraInfo was included on the Plan model for experimental
         spatial analysis (as noted in Section 3.2.2). However, this approach was deprecated because
         [here you explain the limitations, e.g., 'the coordinate system was inconsistent,' or 'it
         proved too inflexible for the graph-based algorithm']. The final, cleaner schema reflects this
         implementation decision."