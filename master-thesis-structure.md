# Front Matter

- [x] Title Page, Confidentiality Clause, Executive Summary (2–3 pages), Table of Contents
- [ ] List of Figures
- [ ] List of Tables
- [ ] List of Abbreviations
- [ ] Preface (Acknowledgment to the company for providing the topic, data, and required business information)

# Introduction ✅

## Problem Statement ✅

- [x] Traditional Quotation Challenges
- [x] Industry Relevance (Growth / Increasing customers)
- [x] Availability of LiDAR (iPhone and Laser measurements)

## Overview and Other Options

- [x] Lidar Scanning Techniques and Apps (MagicPlan, etc.)
- [x] Product Recommendation Systems
- [x] Quotation Systems
- [x] There is a lack of solutions that integrate LiDAR scanning with product recommendations for bathroom renovations. Other configurators do not offer this combined functionality.

## Purpose of the Master’s Thesis ✅

- [x] Primary goal: Develop a functional prototype to select the best shower enclosures for a given space, using exact measurements provided by the user or captured by the MagicPlan app from the database.
- [x] Design and implement a MagicPlan LiDAR data integration pipeline.
- [x] Develop intelligent product-matching algorithms for multiple shower configurations.
- [x] Create a comprehensive product database with compatibility relationships.
- [x] Generate a JSON request or PDF export format for further use.

## Scope ✅

- [x] What is the timeframe?
- [x] Focus on the German market
- [x] Product Categories: Changed from complete bathroom renovations to shower upgrades (such as converting a bathtub to a shower or replacing an existing shower).
- [x] Technical Details/Constraints: Full-stack web application (Next.js, TypeScript, Express, Postgres), iOS LiDAR devices, partner database integration.

## Preview of the overall structure (thesis structure) 

# System Requirements and Architecture Designing ✅

## Requirement Analysis: ✅

- [x] Core features: plan selection, measurement extraction, product filtering, rule-based recommendation, manual override, and quote generation (material cost).

## Non-functional requirements:

- [ ] JWT Authentication
- [ ] Usability features include an intuitive user interface, easy addition of new combinations, AI-powered search using large language models (LLM), a Selenium-based price scraper for updated pricing, a CSV upload feature for multiple products, and a model number-to-product mapping feature in the database utilizing the Selenium scraper.
- [ ] Scalability (ability to add more combinations)
- [ ] Performance

## Tech Stack Justification: ✅

- [ ] Next.js, Express, TypeScript, PostgreSQL, Docker

## System Architecture Overview

- [ ] 3-tier architecture (Presentation, API, Data)
- [ ] Are component diagrams needed? (Frontend modules, backend services, database schema overview)
- [ ] MagicPlan integration using API.

## Data Model Design ✅

- [ ] Relationships Between Data Tables
- [ ] Schema Design Justification
- [ ] Key Relationships
- [ ] Scalability support for new product styles within similar products.

# Lidar Integration ✅

## About Lidar ✅

- [ ] How it works, including mobile LiDAR systems.
- [ ] MagicPlan API integration: data structure, fixture detection, measurement extraction, plan retrieval, and more.
- [ ] Floor plan data processing pipeline: analysis of JSON structure, fixture detection algorithms, and data storage methods.
- [ ] Autofill for fixture recognition (also user-overridable).
- [ ] Implementation challenges: poor-quality scans, missing fixtures, and outdated JSON files.

# Intelligent Product Recommendation Algorithm ✅

## Product Database Structure

- [ ] Product Types (Scalable)
- [ ] Product styles (scalable)
- [ ] Orientation support (for showers)
- [ ] Opening Types (also scalable for showers)

## Product Relationship Graphs

- [ ] Type of Relationship
- [ ] Algorithm for Finding Compatible Side Panels and Trays Given a Door Selection

## Niche, corner entry, U-shape, walk-in (recommendation algorithms)

- [ ] Problem Definition, Compatibility Requirements, and Algorithm Logic and Selection for Each

## Scalable to accommodate more styles in the future.

- [ ] Added a door type table with styles.

## Optimization Strategies

# System Implementation ✅

## Backend Implementation

- [ ] API Architecture
- [ ] Route Organization (all endpoints, including scrapers)
- [ ] Services Organization
- [ ] Middleware Architecture (RBAC)
- [ ] Swagger Documentation

## Database Implementation

- [ ] Prisma Schema Design
- [ ] Migration
- [ ] Seeding Admin
- [ ] Data Integrity

## Frontend Implementation

- [ ] Page Components
- [ ] Reusable UI components
- [ ] State management using the Zustand store
- [ ] Internationalization: Support for English and German languages

## UI explanation

- [ ] Step-by-step configuration wizard
- [ ] Product management and product relationship dashboard (CSV upload, bulk addition of relationships)
- [ ] Responsive design
- [ ] Quote generation and export:
- [ ] Selected products, quantities, prices, and customer information (from plan metadata).
- [ ] Bill of Materials Structure
- [ ] Export Formats
- [ ] Authentication (JWT, bcrypt hashing, RBAC, session management)
- [ ] Product Management Features
- [ ] CSV Bulk Upload: Parsing Product Data, Validation, Error Reporting, and Partial Success Handling
- [ ] Individual Product Editing: Inline Editing, Modal Dialog Forms, and Optimistic Updates
- [ ] Product Search and Filtering: Server-side pagination, debounced search, and multi-criteria filtering
- [ ] Product Relationship Management: User interface for defining compatible products and selecting relationship types.

# Testing and Evaluation ✅

## Testing Methodology

- [ ] Testing all routes using unit tests.
- [ ] API testing for middleware, error handling, validation, and database operations.
- [ ] End-to-end workflow tests
- [ ] Test data: More than 20 real bathroom floor plans

## Algorithm Accuracy

- [ ] Test scenarios: compact installations, standard installations, large installations, custom installation sizes.
- [ ] Success Metrics
- [ ] Failure Analysis

## Performance benchmark? (Response time, concurrent user load?)

## Data Quality and Consistency

- [ ] Added products are always good. If no minimum or maximum construction range is found, it always shows the width with 5 mm lower- and 10-mm higher construction ranges, respectively.
- [ ] Price Data Accuracy
- [ ] Relationship Graph

## Usability Evaluation

- [ ] User Feedback
- [ ] What is lacking?
- [ ] Accessibility?

## Comparison with Manual Process

- [ ] Time savings
- [ ] Error Reduction
- [ ] Consistency
- [ ] Scalability

# Discussion of Findings ✅

## Achievement of Objectives

- [ ] Deliverables Confirmation
- [ ] Project Deviations
- [ ] How is it better than the ML approach?

## Technical Challenges and Solutions

- [ ] Limitations of the Current Implementation
- [ ] Product catalog dependency and manufacturer availability
- [ ] Lack of 3D rendering
- [ ] LiDAR Device Limitation (iOS Only)
- [ ] Magicplan API Limitations
- [ ] No API available due to partner database limitations.
- [ ] Too many business logic constraints.
- [ ] No images of the user's bathroom; only a 2D floor plan is provided.

## Comparison with Alternative Approaches:

- [ ] Hassmann VIGOR Duschabtrennungskonfigurator: This configurator displays only one configuration at a time and does not show prices. The UI workflow is tedious, as prices are only shown when adding products to the cart, which is limited to 13 items in total. Additionally, the partner database contains all the data but the configurator is incomplete. There is no planned integration.
- [ ] Vigour 3D configurator only provides a general overview of products without displaying model numbers. It prompts users to contact the company or request a quote. (Pros: 3D visualization; Cons: no pricing information and no plan integration.)
- [ ] The DUKA 3D configurator only provides a high-level overview of products and prompts users to contact the company or request a quote. It includes 3D visualization but lacks pricing information and plan integration.

## Generalization to Other Domains

# Conclusion ✅

## Summary of Results:

- [ ] The effectiveness of rule-based recommendation algorithms remains relevant when accurate results are required.
- [ ] Business value: significant time savings and error elimination
- [ ] Positive reviews indicate that the UI workflow is superior to alternative approaches.

## Contributions

- [ ] First documented system combining LiDAR scanning with intelligent bathroom product recommendations.
- [ ] Graph-Based Compatibility Model: A reusable pattern for modeling complex product relationships in shower enclosures.
- [ ] Multi-configuration algorithm

# Bibliography ✅

# Appendix ✅

## a) Lidar raw data sample from the Magicplan API.

- [ ] Floor plan showing fixture measurements

## b) Algorithm Pseudocode

- [ ] Provide detailed pseudocode for the NICHE, Eckeinstieg, and U-form algorithms.
- [ ] Complexity Analysis and Optimization Notes

## c) Database schema documentation

- [ ] Complete the Prisma schema with field descriptions.
- [ ] Entity-Relationship Diagram

## d) API Documentation

- [ ] Swagger
- [ ] Sample Request/Response

## e) Test Case Documentation

- [ ] Table with bathroom scenarios including dimensions, expected results, and actual results.
- [ ] Screenshot of successful recommendations.

## f) User Interface Screenshots

- [ ] Complete walkthrough of the configuration wizard
- [ ] Admin Dashboard
- [ ] Sample Generated Quotes

## g) Code Repository Structure

- [ ] Directory Tree Explanation
- [ ] Key File Descriptions and Responsibilities

## h) Scraper Scripts

## i) Deployment Guide

- [ ] Environment Setup
- [ ] Docker Configuration
- [ ] Production Deployment Checklist