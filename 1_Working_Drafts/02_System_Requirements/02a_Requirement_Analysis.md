# 2. Requirement Analysis

This chapter outlines the analysis of the existing system's shortcomings and defines the functional and non-functional requirements for the new product recommendation system.

## 2.1 Problem Statement and Background

An analysis of the company's internal processes revealed significant inefficiencies in the existing workflow for generating quotes for shower enclosures. These challenges stem from the limitations of previously used software solutions.

### 2.1.1 Challenges with Legacy Commercial Software

The primary software used for quote generation was "HERO Software"[^1], a comprehensive but overly complex tool for the company's specific needs. Based on feedback from the sales department, two main issues were identified:

1.  **High Usability Barriers**: The software's interface was not intuitive for sales personnel, often requiring intervention from the IT department to configure quote templates and calculation parameters.
2.  **Requirement of Expert Knowledge**: Effective use of the configurator demanded deep expertise in construction details (e.g., gutter placement, faucet location). This dependency led to a steep learning curve and a high probability of errors, especially for new employees.

Furthermore, the software's high licensing costs posed a significant financial burden, especially since many of its features, designed for general construction, were rarely used by the company.

### 2.1.3 Limitations of the Subsequent In-House Solution

To address these issues, a simpler, more user-friendly application was developed internally using JavaScript. While this application improved usability, it introduced a different problem: it lacked intelligent filtering capabilities. The sales team still could not automatically select or recommend shower enclosures based on critical parameters such as:

-   Dimensions (width, depth, height)
-   Shower construction style (e.g., niche, corner entry, walk-in)
-   Door type (e.g., folding, sliding, pivot)
-   Orientation (left, right, or universal)

This lack of intelligent logic meant the process was still manual and prone to selection errors.

## 2.2 System Requirements

Based on the analysis of the existing problems, the following functional and non-functional requirements have been defined for the new system.

### 2.2.1 Functional Requirements

The system must provide the following functionalities:

| ID   | Requirement                                                                                                   | Priority    |
| :--- | :------------------------------------------------------------------------------------------------------------ | :---------- |
| FR-1 | The system shall allow users to input shower dimensions (width, depth, height) to filter compatible products. | Must-have   |
| FR-2 | The system shall enable filtering of shower enclosures by construction style (e.g., niche, corner, U-shape).  | Must-have   |
| FR-3 | The system shall provide options to select products based on door type (e.g., sliding, pivot, folding).       | Must-have   |
| FR-4 | The system shall allow filtering based on the orientation of the shower door (left, right, or universal).     | Should-have |
| FR-5 | The system shall generate a preliminary quote based on the selected product and configuration.                | Should-have |
| FR-6 | The system must have full Mac Support.                                                                        |             |

### 2.2.2 Non-Functional Requirements

The system must adhere to the following quality attributes:

| ID    | Requirement                                                                                                            | Priority    |
| :---- | :--------------------------------------------------------------------------------------------------------------------- | :---------- |
| NFR-1 | The user interface must be highly intuitive and require minimal training for non-technical sales staff.                | Must-have   |
| NFR-2 | The system must perform quickly, allowing for real-time quote generation during a customer consultation.               | Must-have   |
| NFR-3 | The system must be accessible on both desktop and tablet devices.                                                      | Should-have |
| NFR-4 | The total cost of ownership (development and maintenance) must be significantly lower than previous licensed software. | Must-have   |
| NFR-5 | The user interface must be multi-lingual.                                                                              | Must-have   |
