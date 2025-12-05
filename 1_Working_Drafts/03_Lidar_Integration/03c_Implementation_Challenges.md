# 3.3 Implementation Challenges

*This section will discuss the challenges encountered during implementation, such as handling poor-quality scans, missing fixtures, and outdated JSON files.*

According to internal company interviews, the MagicPlan application exhibits certain limitations and anomalies, although no definitive cause has been identified. For instance, the scanner occasionally fails to generate accurate wall data for spaces with multiple walls (exceeding five walls). Additionally, niche showers are sometimes detected outside the boundaries of the room plan, and the walls in some rooms appear slightly misaligned. Since the measurements remained the same this was not an issue and the user always has manual override where they can change the dimensions freely if they found they dont need inaccurate magicplan data since sometimes the customer has a floor plan of his own which is better.  

implementation of storing data in self database will not always be updated if the user manually decides to edit the magicplan floorplan after adding it into database. it is not dynamic. but there is a feature for user to update the same plan by adding it again and update prompt is given instead of plan is saved by default if there was no plan. 

Due to certain magicplan limitations and the nessasity to fully optimize quoting for the company some features were left out of the scope of the project which were challenging but achievable goals. these are the following

## 3.3.1 Automatic quotation.

The company was using magicplan to do all the 3d room scanning and it was alredy their primary workflow. So it was nessary for the project to utilize data from magicplan to help generate an automatic quoation system. but there were certain errors for this implementation. 

1. the way magicplan stores data in a json format. each json response is on average 487 KBs. this request also include full XML format which holds spartial data nessesary for spatial analysis and fixture data with positions. Magicplan provides coordinates for each wall with meters as mesurement and id which corresponds to additional data in json format. eg. wall length, height, etc. But these wall data was only for external walls. internal walls were not included in this request. also magicplan gives interior wall thickness but it was standardized to 0.12m which was an error on the employees part where they can manually change it to desired thickness. 90% of the projects were affected by this issue. This created a mismatch when trying to recreate it in spatial analysis when trying to generate coordinates for inernal wall from external wall. This became more of an issue when the fixture data position and rotation were observed. while the app stored wall data in two coordinates start and end. fixture data only one coordinate of origin from there it was sccaled . this created another mismatch when trying to reduce or increase the size of shower for spatial analysis as now we cannot find accurate interiror wall due to incorrect data and the scaling was done from the center meaning placing anything smaller and it wont touch the wall and if bigger  it will go outside the wall. the only way spatial analysis would work was if the shower was already present and the customer wanted to remove old and add new shower in the same space. another was alcove locations where spatial analysis shined but only 2or 3 plans had alcove shower or bathtub where the sidewall was detected correclty. there were other plans which had alcove shower but the magicplan failed to recognize an alcove and placed it as a corner shower. this was a reason automatic shower style selection was not used as it only worked on 3% of the plans and also the customer had a choice for a shower type.
2. Another reason, the company always did bathtub to shower or shower to shower replacements. this meant everytime they had bathtub to shower which was most of the time. they always find themselves reducing the shower size since bathtub are generally long. The average bathtub size amound 37 added plans in db was 1.67 meters a quick sql query was used to find this : SELECT AVG(width) as average_badewanne_width FROM "Fixture" WHERE name ILIKE '%badewanne%' AND width IS NOT NULL; . There were shower in database that were 180cm width but they are considered premium products and are too expensive for insurance customers. the company often reduced the size of the shower to fit the customers preference. (1) Now the new location of the shower was only known to the consultant who would select appropriate quotes. the consultant would try to minimize cost by selecting the best possible shower that would fit the space. this would need heavy business logic and manually going through multiple offers to see all the business logic. Eg. of a business logic: Make the shower atleast 120 cm because then it would reduce the amount of replacement tiles that would be required and they also have to only extend the drain and the thermostat by only 5 cm which would be reletively cheap than selecting a more cheaper smaller shower eg 90 cm but now it would need more replacement tiles and also more thermostat piping and drilling to move the drain even more by 30 cm. There were many more business logics like these but were not made available. This would also not be updated on magicplan eventhough they have this feature because of time consumption. So the spacial analysis would now also need drain location and thermostat location for such quotations, which were not present in magicplan and not even detected by magicplan . so this was kept out of the scope of the project. (2) this would also affect the automatic door type selection. since the shower can be replaced either side dpending upon the drain and thermostat location, either side can have blocking or no items in front of it. when no items are front of it the company can select pivot doors which were generally cheaper rather than something like folding doors or sliding doors which were exponentially more expensive than pivot doors especially folding doors they always were too expensive for an insurance customer. the blocking could be calculated by finding the door of the shower that is the frontside converting the single coordinate system for fixtures into 4 point coordinate system and checking the parallel distance or the radius from either side of the door equal to the width of the door to see if anything is blocking it. if yes then we can change doors to 2 part pivot door which opens both inside and out. if both of them cant fit the next options would be sliding doors or folding doors the most expensive ones. since the company never edited magicplan for selecing a shower or manually checking if its viable or not from magicplan this was also out of the scope of the project. 


Fixture data from the API included a single coordinate for the
      object's origin and scaling properties for its dimensions. This proved problematic for spatial
      analysis, as resizing a fixture from its center point caused it to either detach from a wall or
      extend beyond the room's boundaries, rather than scaling naturally along the wall. - Can you also add : unavailability of interior walls affected collisions and when trying to predict collisions it would attach itself to middle  of the wall instead of on the border. half width and depth were also added to see if it work but the distance to wall would still be inacurate because of wrong wall thickness and would gravely affect the shower selection process which we will see in the coming chapters is extremely dependent on accurate measurements.



  Based on internal stakeholder interviews and an in-depth analysis of the MagicPlan API, several
  implementation challenges were identified. These challenges ultimately influenced the final scope of
  the project, particularly regarding the feasibility of fully automated spatial analysis and quotation
  generation.

  3.3.1 Challenge: Inconsistent and Incomplete Spatial Data

  A primary obstacle to implementing reliable automated spatial analysis was the quality and structure
  of the data provided by the MagicPlan API. Several key issues were identified:

   1. Wall Data Ambiguity: The API response provided coordinate data for a room's external walls but
      did not explicitly define the internal walls. While interior wall thickness was available, this
      was often set to an incorrect, standardized default of 0.12m by users, making it impossible to
      programmatically reconstruct an accurate interior floor plan from the exterior wall data. This
      affected an estimated 90% of the reviewed projects.
   2.  **Fixture Positioning and Collision Detection:** Fixture data from the API included a single coordinate for the object's origin and scaling properties for its dimensions. This proved problematic for spatial analysis, as resizing a fixture from its center point caused it to either detach from a wall or extend beyond the room's boundaries. This issue was compounded by the unavailability of reliable interior wall data, which is critical for collision detection. In placement simulations, fixtures would incorrectly align with the center line of a wall's thickness instead of its interior surface. While a workaround was attempted by offsetting the fixture's origin by half of its width and depth, the unreliable wall thickness data made calculating the correct offset impossible. This level of spatial inaccuracy would gravely affect the shower selection process, which, as will be discussed in later chapters, is extremely dependent on precise measurements.   3. Inaccurate Object Recognition: In some test cases, the MagicPlan scanner misidentified fixture
      placement. For example, showers located in an alcove were sometimes detected as standard corner
      showers, which would lead to incorrect automated style recommendations.

  These data quality issues made it infeasible to reliably implement automated shower style selection
  (e.g., alcove vs. corner) based purely on spatial analysis, as the risk of generating incorrect
  recommendations was unacceptably high. While a user override feature was always planned, the goal of
  a fully automated initial suggestion was deemed out of scope.

  3.3.2 Challenge: Uncaptured Business Logic in Cost Optimization

  A further significant challenge was the system's inability to capture the complex, implicit business
  logic that human consultants apply during the quotation process, especially in renovation scenarios.

  A common use case for the company is replacing a bathtub with a new shower. An analysis of the
  project database (N=37) revealed an average bathtub width of 1.67m. Insurance customers, however,
  often prefer a smaller, more cost-effective shower (e.g., 1.20m). A human consultant optimizes this
  replacement not just on the cost of the shower unit, but on the total project cost. For example, a
  slightly larger and more expensive shower might require fewer replacement tiles and less modification
  to existing plumbing, making it cheaper overall than a smaller shower that requires extensive rework.

  This decision-making requires data points not available in MagicPlan, such as the location of drains
  and thermostats. As this nuanced, context-dependent business logic could not be algorithmically
  defined without further research, a fully automated quotation system that could truly optimize for
  total project cost was determined to be beyond the scope of this thesis.

  3.3.3 Challenge: Data Synchronization

  A final architectural challenge involves maintaining data consistency between the system's local
  database and the MagicPlan platform. Once a floor plan is imported into the system, any subsequent
  edits made within the MagicPlan application are not automatically synchronized.

  To mitigate this, the system was designed to handle re-imports. If a user attempts to save a plan
  that already exists in the database, they are presented with an "update" prompt rather than creating
  a duplicate entry. This approach, however, places the responsibility for maintaining data integrity
  on the user and does not support real-time, dynamic synchronization.