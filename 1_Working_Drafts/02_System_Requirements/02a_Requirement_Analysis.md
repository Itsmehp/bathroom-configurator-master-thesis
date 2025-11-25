# 2. Requirement Analysis

This chapter outlines the analysis of the existing system's shortcomings and defines the functional and non-functional requirements for the new product recommendation system.

## 2.1 Problem Statement and Background

An analysis of the company's internal processes revealed significant inefficiencies in the existing workflow for generating quotes for shower enclosures. These challenges stem from the limitations of previously used software solutions.(footnotes reference from company)

### 2.1.1 Challenges with Legacy Commercial Software

The primary software used for quote generation was "HERO Software"[^1], a comprehensive but overly complex tool for the company's specific needs. Based on feedback from the sales department, two main issues were identified:

1.  **High Usability Barriers**: The software's interface was not intuitive for sales personnel, often requiring intervention from the IT department to configure quote templates and calculation parameters.(add footnote reference)
2.  **Requirement of Expert Knowledge**: Effective use of the configurator demanded deep expertise in construction details (e.g., gutter placement, faucet location). This dependency led to a steep learning curve and a high probability of errors, especially for new employees.

Furthermore, the software's high licensing costs posed a significant financial burden, especially since many of its features, designed for general construction, were rarely used by the company.(footnote reference)

### 2.1.3 Limitations of the Subsequent In-House Solution

To address these issues, a simpler, more user-friendly application was developed internally using JavaScript. While this application improved usability, it introduced a different problem: it lacked intelligent filtering capabilities. The sales team still could not automatically select or recommend shower enclosures based on critical parameters such as:

-   Dimensions (width, depth, height)
-   Shower construction style (e.g., niche, corner entry, walk-in)
-   Door type (e.g., folding, sliding, pivot)
-   Orientation (left, right, or universal)

This lack of intelligent logic meant the process was still manual and prone to selection errors.

### 2.1.4 Limitations of the Partner's database

The partner's database functioned as a business-to-business (B2B) website, supplying the company with discounted products across the home renovation spectrum, including shower enclosures and other bathroom-related items. A critical deficiency was the lack of an Application Programming Interface (API), which prevented programmatic access to product data for shower enclosures or other fixtures. Although the website offered a configurator that could suggest shower enclosures based on width and depth, its usability was hampered by a cumbersome navigation requiring multiple clicks to specify the desired shower type and opening mechanism. Moreover, the configurator presented an incomplete range of options, omitting several configurations that were explicitly detailed in the product catalog. While a wide array of style choices was available—such as various glass types, profile colors, anti-water coatings, automatic close features, and profile styles—internal company reviews highlighted a preference for the most basic designs, a preference driven primarily by the insurance-funded customer base.
## 2.2 System Requirements

Based on the analysis of the existing problems, the following functional and non-functional requirements have been defined for the new system.

### 2.2.1 Functional Requirements

The system must provide the following functionalities:

| ID    | Requirement                                                                                                       | Priority     |
| :---- | :------------------------------------------------------------------------------------------------------------------ | :----------- |
| FR-1  | The system shall allow users to input shower dimensions (width, depth, height) to filter compatible products.       | Must-have    |
| FR-2  | The system shall enable filtering of shower enclosures by construction style (e.g., niche, corner, U-shape).        | Must-have    |
| FR-3  | The system shall allow filtering of products by door type (e.g., sliding, pivot, folding).                          | Must-have    |
| FR-4  | The system shall allow filtering by shower door orientation (left, right, or universal).                            | Should-have  |
| FR-5  | The system shall generate a preliminary quote based on the selected product and configuration.                      | Should-have  |
| FR-6  | The system shall be fully compatible with the macOS operating system.                                               | Must-have    |
| FR-7  | The system shall provide a documented API to allow for programmatic searching of compatible products.                 | Must-have    |
| FR-8  | The system shall include an administrative back-end for managing the product catalogue (CRUD).                      | Must-have    |
| FR-9  | The system shall include an administrative back-end for managing user accounts (CRUD).                              | Must-have    |
| FR-10 | The system shall provide a dashboard displaying key usage statistics.                                                 | Nice-to-have |
| FR-11 | The system shall automatically synchronize product pricing from the partner's data source.                            | Nice-to-have |

### 2.2.2 Non-Functional Requirements

The system must adhere to the following quality attributes:

| ID    | Requirement                                                                                                | Priority     |
| :---- | :--------------------------------------------------------------------------------------------------------- | :----------- |
| NFR-1 | The user interface shall be highly intuitive, requiring minimal training for non-technical sales staff.    | Must-have    |
| NFR-2 | The system shall generate product recommendations in under 3 seconds to enable real-time consultations.    | Must-have    |
| NFR-3 | The system's UI shall be responsive and fully functional on both desktop and tablet devices.                 | Should-have  |
| NFR-4 | The total cost of ownership (TCO) shall be significantly lower than the previously licensed software.      | Must-have    |
| NFR-5 | The system shall be architected to support multi-lingual user interfaces (internationalization, i18n).       | Should-have  |
| NFR-6 | The system's database shall be backed up automatically on a daily schedule.                                  | Nice-to-have |
