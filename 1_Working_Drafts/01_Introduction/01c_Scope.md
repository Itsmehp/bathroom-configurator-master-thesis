
The scope of this master's thesis is precisely delineated to facilitate the successful development of a functional prototype within a constrained timeframe of 5 months. The following boundaries define the project's focus:

Market and Geographical Scope  
This study is exclusively oriented towards the German market. This emphasis guides the selection of products as well as the business logic underlying pricing and quotation processes. Although the system architecture is designed to be adaptable, the initial product database, supplier integration, and rule-based algorithms are specifically developed to align with the context of bathroom renovation practices in Germany. The adaptation of the system for other international markets is explicitly considered beyond the scope of this research.

Product and Functional Scope  
The primary function of the system is to provide recommendations for common upgrade scenarios, including:  
- Replacement of existing shower enclosures  
- Conversion of bathtubs into standalone showers  

Accordingly, the product database and recommendation algorithms concentrate on shower doors, shower side panels, and shower trays. Other components typically involved in comprehensive bathroom renovations, such as lighting, tiling, sinks, or toilets, are deliberately excluded from this study; however, the underlying database structure is designed to allow these categories to be added with minimal effort.

Technical and Implementation Constraints  
The project is scoped as the development of a full-stack web application prototype. The technical implementation is limited to the following technology stack:  
- Frontend: Next.js with TypeScript  
- Backend: Express.js with TypeScript  
- Database: PostgreSQL  with Prisma ORM

Regarding data acquisition, the system is designed to operate exclusively with room scan data obtained from iOS devices equipped with LiDAR sensors or iOS devices connected to a Bluetooth LiDAR sensor via the MagicPlan API. The development of native applications for Android or other platforms falls outside the scope of this thesis. Although access to the partner’s product database was provided, it lacked integration capabilities or usable APIs. Consequently, the thesis includes the creation of a dedicated product catalog and database through approved data extraction from the partner’s system. To support maintainability and ensure up-to-date pricing and product information, additional automated scripts were developed to periodically scrape price data and update the database, as well as a script capable of adding new products directly by retrieving all relevant information from the partner’s website using a model number as input.