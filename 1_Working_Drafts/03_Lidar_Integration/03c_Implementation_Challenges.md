# 3.3 Implementation Challenges

*This section will discuss the challenges encountered during implementation, such as handling poor-quality scans, missing fixtures, and outdated JSON files.*

According to internal company interviews, the MagicPlan application exhibits certain limitations and anomalies, although no definitive cause has been identified. For instance, the scanner occasionally fails to generate accurate room plans for spaces with multiple walls (exceeding five walls). Additionally, niche showers are sometimes detected outside the boundaries of the room plan, and the walls in some rooms appear slightly misaligned.


implementation of storing data in self database will not always be updated if the user manually decides to edit the magicplan floorplan after adding it into database. it is not dynamic. but there is a feature for user to update the same plan by adding it again and update prompt is given instead of plan is saved by default if there was no plan. 