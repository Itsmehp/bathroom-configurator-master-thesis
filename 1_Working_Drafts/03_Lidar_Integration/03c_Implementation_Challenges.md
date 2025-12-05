
---

### Academic Formatting of Implementation Challenges

Below is a re-structured and academically formatted version of the challenges you outlined. This version maintains your core observations while enhancing clarity, formality, and logical flow.

According to internal company interviews, the MagicPlan application exhibits certain limitations and anomalies, although a definitive cause has not been identified. For instance, the scanner occasionally fails to generate accurate wall data for spaces with more than five walls. Additionally, niche showers are sometimes detected outside the boundaries of the room plan, and the walls in some rooms appear slightly misaligned. While these measurement inconsistencies did not inherently pose an issue due to the user's manual override capabilities (allowing adjustments based on their own floor plans), they did highlight the variability of the source data.

A further challenge involves the synchronization of data. The implementation of storing data in the local database does not dynamically update if the user manually modifies a MagicPlan floorplan after its initial addition to the database. However, a feature exists that allows users to re-import an existing plan, prompting an update rather than saving a duplicate.

Due to these MagicPlan limitations and the necessity to fully optimize quoting for the company, some features, while challenging and achievable, were intentionally left out of the project's scope. These are detailed below:

### 3.3.1 Challenge: Inconsistent and Incomplete Spatial Data

A primary obstacle to implementing reliable automated spatial analysis was the inconsistent and often incomplete nature of the spatial data provided by the MagicPlan API. Several key issues were identified:

1.  **Wall Data Ambiguity:** The API response provided coordinate data for a room's external walls but did not explicitly define the internal walls. While interior wall thickness was available, this was often set to an incorrect, standardized default of 0.12m by users, making it impossible to programmatically reconstruct an accurate interior floor plan from the exterior wall data. This affected an estimated 90% of the reviewed projects.
2.  **Fixture Positioning and Collision Detection:** Fixture data from the API included a single coordinate for the object's origin and scaling properties for its dimensions. This proved problematic for spatial analysis, as resizing a fixture from its center point caused it to either detach from a wall or extend beyond the room's boundaries. This issue was compounded by the unavailability of reliable interior wall data, which is critical for collision detection. In placement simulations, fixtures would incorrectly align with the center line of a wall's thickness instead of its interior surface. While a workaround was attempted by offsetting the fixture's origin by half of its width and depth, the unreliable wall thickness data made calculating the correct offset impossible. This level of spatial inaccuracy would gravely affect the shower selection process, which, as will be discussed in later chapters, is extremely dependent on precise measurements.
3.  **Inaccurate Object Recognition:** In some test cases, the MagicPlan scanner misidentified fixture placement. For example, showers located in an alcove were sometimes detected as standard corner showers, which would lead to incorrect automated style recommendations.

These data quality issues made it infeasible to reliably implement automated shower style selection (e.g., alcove vs. corner) based purely on spatial analysis, as the risk of generating incorrect recommendations was unacceptably high. While a user override feature was always planned, the goal of a fully automated initial suggestion was deemed out of scope.

### 3.3.2 Challenge: Uncaptured Business Logic in Cost Optimization

A further significant challenge was the system's inability to capture the complex, implicit business logic that human consultants apply during the quotation process, especially in renovation scenarios.

A common use case for the company is replacing a bathtub with a new shower. An analysis of the project database (N=37) revealed an average bathtub width of 1.67m. Insurance customers, however, often prefer a smaller, more cost-effective shower (e.g., 1.20m). A human consultant optimizes this replacement not just on the cost of the shower unit, but on the total project cost. For example, a slightly larger and more expensive shower might require fewer replacement tiles and less modification to existing plumbing, making it cheaper overall than a smaller shower that requires extensive rework.

This decision-making requires data points not available in MagicPlan, such as the location of drains and thermostats. As this nuanced, context-dependent business logic could not be algorithmically defined without further research, a fully automated quotation system that could truly optimize for total project cost was determined to be beyond the scope of this thesis.

### 3.3.3 Challenge: Data Synchronization

A final architectural challenge involves maintaining data consistency between the system's local database and the MagicPlan platform. Once a floor plan is imported into the system, any subsequent edits made within the MagicPlan application are not automatically synchronized.

To mitigate this, the system was designed to handle re-imports. If a user attempts to save a plan that already exists in the database, they are presented with an "update" prompt rather than creating a duplicate entry. This approach, however, places the responsibility for maintaining data integrity on the user and does not support real-time, dynamic synchronization.
