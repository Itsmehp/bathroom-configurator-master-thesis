# 2.5 Future Developments and Scalability Opportunities
The application is built with technologies that scale well with changes. It is under constant development and it is possible to improve the suggested system even further. The improvements foreseen in the future of Rule based shower configuration selection are:


## 2.5.1 Intelligent Shower Type Recommendation Based on User History

Currently, the system enables users to manually select shower configurations based on their requirements. However, a more advanced version could implement automated recommendations by analyzing previous selections and applying rule-based logic [mangiliBayesianApproachConversational2020]. This enhancement would consider the following factors:

- The customer's preference to transition from old shower to new shower
- Dimensional and location consistency with previously selected showers
- Customer preference for enclosure updates without fixture replacement

Based on these criteria, the system could perform spatial analysis to evaluate the available space in front of the shower and identify any obstacles or connected fixtures such as bathtubs. This information would enable the recommendation of appropriate door opening mechanisms. For instance, if a toilet is positioned 30 cm from the shower entrance, the system could suggest sliding doors to accommodate the spatial constraints. Alternatively, for locations with limited clearance on both sides, pivot doors that open both inward and outward could be recommended to prevent collision issues while maintaining functionality.

## 2.5.2 Comprehensive Bathroom Renovation Module Using Artificial Intelligence

The current implementation focuses exclusively on shower enclosure selection within specified dimensions. However, the system can be expanded to include additional bathroom fixtures and materials, such as toilets, wash basins, grab handles, faucets, wall coverings, tiles, folding seats, and thermostats. This expansion would enhance profitability by offering more comprehensive solutions to customers.

The existing relationship model can be extended to maintain product compatibility across categories. For example, toilets can be linked with appropriate flush systems based on installation type (wall-mounted or concealed), and wash basins can be paired with compatible faucets. This prevents accidental selection of incompatible products.

Artificial intelligence could further personalize recommendations by incorporating additional customer information, such as age, insurance coverage status, and mobility requirements. This data would allow the system to filter available options accordingly. For instance, elderly customers or those with mobility limitations could be presented with grab handles, folding seats, and walk-in shower configurations designed for accessibility.

## 2.5.3 Machine Learning-Based Bathroom Layout Optimization

The current spatial analysis approach could be enhanced through machine learning techniques trained on multiple floor plans. With sufficient training data, the system could predict optimal fixture placement within bathroom spaces or employ reinforcement learning methods that reward effective spatial arrangements [diDeepReinforcementLearning2021].

However, this approach faces several limitations. Bathroom designs vary significantly across buildings due to structural differences, making training effective only for homogeneous datasets where variations are limited to room size, door position, and window location [diDeepReinforcementLearning2021]. Additionally, the data provided by floor plan APIs often presents preprocessing challenges. Specifically, the API returns coordinates for external walls but not internal walls, and fixtures are represented by a single center point and dimensions, creating significant data misalignment that requires extensive preparation before analysis can begin.

Furthermore, the floor plan software sometimes produces inaccurate results—such as placing fixtures outside the defined room boundaries without creating appropriate enclosures, or failing to correctly align walls to 90-degree angles despite the actual measurements being perpendicular. These technical limitations must be addressed before this enhancement can be reliably implemented.

## 2.5.4 Visual Previews via Generative AI and Diffusion Models

Currently there are two popular approaches for image generation and image editing: Generative Adversarial Networks [goodfellowGenerativeAdversarialNets2014] and Diffusion Models [hoDenoisingDiffusionProbabilistic2020]. Furthermore, Diffusion models for their capacity to generate high quality, photorealistic images. Their effective integration with Large Language Models (LLMs) also facilitates the straightforward translation of a user's textual ideas into visual concepts, as demonstrated by systems like VIDES [leVIDESVirtualInterior2023].

Diffusion models with editing capabilities could enable customers to visualize proposed bathroom designs by processing photographs taken at appropriate angles. However, two significant practical barriers currently prevent its deployment. First, a primary challenge lies in workflow integration. The existing system is optimized for rapid, automated quote generation. Introducing a manual step, such as requiring user input to select a specific area in an image, would add latency and hinder this core function. This suggests the feature might be better suited for optional, high-end design services rather than for immediate quoting.

Second, a more fundamental technical barrier is data scarcity. The product database contains limited visual assets—typically only one or two promotional images and basic CAD-style line drawings per product. This lack of diverse, high-fidelity imagery would severely challenge the ability of any generative model to produce varied and realistic renderings of the products within a new scene.

## 2.5.5 Intelligent Chatbot Assistant for Budget-Conscious Recommendations

A chatbot interface could provide intelligent suggestions when a recommended shower configuration exceeds the customer's budget. The system could strategically suggest modifications—such as reducing width or depth—while maintaining aesthetic coherence and functionality. This would help customers find solutions that balance their financial constraints with their design preferences [sunConversationalRecommenderSystem2018]. 

