# State of the Art and Approach

This chapter reviews the state-of-the-art in technologies relevant to this thesis, beginning with an analysis of LiDAR scanning applications for interior mapping. It then examines existing product recommendation and configuration systems to identify the current technological gaps, and finally, outlines the methodological approach adopted to address these gaps. The review begins with LiDAR technology, which was initially developed for terrain mapping in aeronautics and aerospace applications. It has since expanded into diverse domains such as autonomous vehicles, enabling these systems to navigate their environments effectively and generate three-dimensional imagery that can be transformed into Building Information Modeling (BIM) or Computer-Aided Design (CAD) models. 

The integration of LiDAR sensors into consumer mobile devices, exemplified by the iPhone 12 Pro and iPad Pro, has significantly broadened access to this technology. According to MagicPlan, their software utilizes augmented reality (AR) and artificial intelligence (AI) to automatically identify and categorize objects within an environment. The generated dataset, which includes fixtures, furniture, room measurements, and spatial arrangements, offers considerable value for a range of applications.\cite{geibWhyApplesLiDAR}.

These sensors enable the creation of precise three-dimensional visualizations through point cloud data. MagicPlan’s proprietary processing software employs advanced algorithms and AI models to analyze these visualizations, converting them into editable CAD models. Although the specific technical details and proprietary algorithms—such as the application of AI models or rule-based techniques—are not publicly disclosed, the software’s integration with Apple’s RoomPlan API indicates a hybrid approach combining AI-driven spatial recognition with sensor data processing to classify and map interior objects \cite{geibWhyApplesLiDAR}.  

### Analysis of the Incumbent Product Configurator

In addition to MagicPlan for spatial data acquisition, the company's established workflow for product selection relies on a manufacturer-provided, web-based configurator. This tool, hosted on a B2B wholesale portal, is the primary instrument for specifying and ordering compatible shower enclosure components.

The configurator guides users through a hierarchical selection process to ensure component compatibility. It allows for the definition of several key parameters, such as:
*   **Entry Type:** Including corner entry (*Eckeinstieg*), niche (*Nische*), walk-in, and U-form configurations.
*   **Shower Style:** Such as a corner cabin (*Eckkabine*) or a corner entry with a full or shortened side wall.
*   **Door Type:** Options include swing doors (*Pendeltür*), folding swing doors (*Faltpendeltür*), and sliding doors (*Gleittür*).
*   **Installation Details:** Including orientation and mounting type (floor-level or on a shower tray).

Based on these initial selections, the system subsequently recommends compatible components like side panels or doors. For these components, it provides critical dimensional data, including the minimum (*minEinbau*) and maximum (*maxEinbau*) construction range.

Despite its function in ensuring hardware compatibility, a critical analysis of the configurator reveals significant operational and technical limitations that justify the development of a new, integrated solution:

1.  **Lack of Real-Time Price Visibility:** The system does not display component pricing during the configuration process. Prices are only revealed after a product is added to the shopping cart. This limitation severely hampers the sales team, who must select products to fit a customer's budget after a partial quote has already been generated. The issue is exacerbated by a functional limit of thirteen shopping carts per user, which are reportedly always at full capacity.
2.  **Limited Product Options:** For any given selection, the configurator presents only a single product line, precluding the comparison of alternative products or styles that might be available.
3.  **B2B Exclusivity and Price Opacity:** As a wholesale portal, the platform is inaccessible to end-customers and displays the company's net acquisition costs. This lack of transparency makes it unsuitable for direct use in client consultations.
4.  **Outdated Product Data:** The configurator frequently omits new product configurations that are present in the manufacturer's official product catalog and even on the consumer-facing website. Accessing these newer options depends on an employee's specialized knowledge of the print catalog, creating a significant knowledge barrier for new hires and increasing the likelihood of suboptimal product selections.
5.  **No Integration Capabilities:** The tool is a closed, standalone system with no capacity for integration with external applications like MagicPlan. This results in a fragmented workflow that requires manual and error-prone data transfer from the room scan to the product configurator.
6.  **Closed Architecture:** The platform does not offer Application Programming Interfaces (APIs) for accessing its product database, configuration logic, or cart system. This fundamentally prevents automation, data synchronization, or integration with other business-critical software.
7.  **Lack of Shower Tray Recommendation:** While the configurator prompts the user to select an installation type (with or without a shower tray), it does not subsequently recommend a compatible shower tray product. This omission creates an incomplete solution for a full shower replacement, forcing sales staff to manually identify and verify a suitable tray from a separate catalog. This represents a significant gap in providing a comprehensive and seamless customer solution.

In summary, while the incumbent configurator enforces basic compatibility, its closed architecture, outdated data, and inefficient workflow underscore the critical need for a modern, integrated, and intelligent product recommendation system that this thesis aims to deliver.

The preceding analysis demonstrates that while powerful tools for spatial scanning and product configuration exist, they operate in isolation. This thesis addresses this critical gap.
Based on an evaluation of the available technologies, this project will utilize the MagicPlan application and its API for data acquisition and take data, configurator from the wholesale website. Subsequent chapters will provide a detailed account of the data processing methods and their application within the product recommendation algorithm.
