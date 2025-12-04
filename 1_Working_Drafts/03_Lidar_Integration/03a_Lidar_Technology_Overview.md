# LiDAR Technology Overview

## Introduction to LiDAR Technology

LiDAR, which stands for Light Detection and Ranging, is a powerful remote
sensing method. It uses light in the form of a pulsed laser to measure
distances to a target. By emitting rapid laser pulses and measuring the time it
takes for them to bounce back, LiDAR systems can calculate the precise
distance to surrounding objects. This process generates a massive collection of
data points, known as a "point cloud." \cite{waykarLidarTechnologyComprehensive2022a}.

Each point in this cloud has a precise XYZ coordinate, forming a highly
accurate 3D digital representation of a physical space. This method is a
significant leap forward from traditional measurement techniques, offering
unparalleled speed, accuracy, and detail. The resulting 3D models provide a
complete and precise foundation for design, planning, and execution in complex
projects \cite{gelisDeepUnsupervisedLearning2023}.

## The Rise of LiDAR on Edge Devices

The processing of dense LiDAR point clouds is a computationally intensive task that traditionally required powerful, centralized servers. Research has explored a variety of techniques to improve on-device processing at the edge, as well as methods to make the transfer of intermediate data between edge devices and edge servers more efficient \cite{9843856, DBLP:journals/corr/abs-2011-02422, 10.1145/3579371.3589113}.

This miniaturization has led to LiDAR becoming smaller, more accessible, and cost-effective \cite{waykarLidarTechnologyComprehensive2022a}, culminating in its integration into consumer smartphones and tablets like the Apple iPhone Pro series. This has significantly broadened public access to sophisticated spatial scanning. However, each sensor modality has distinct limitations. While LiDAR provides accurate distance measurements, the resulting point clouds are often sparse and lack the rich information, such as colors and textures, required for robust object identification. \cite{zhaoLIFSegLiDARCamera2021} Conversely, camera images contain this dense semantic information but inherently lack reliable depth and scale. Therefore, the most powerful solutions employ sensor fusion, combining the complementary strengths of both modalities. On-device applications leverage machine learning to merge LiDAR's precise 3D structural data with the rich visual detail from camera images, enabling accurate object classification and the creation of detailed three-dimensional models \cite{geibWhyApplesLiDAR}.



Sophisticated mobile applications, such as MagicPlan, have emerged to leverage these capabilities. MagicPlan, a leading solution for as-built modeling, combines augmented reality (AR) with artificial intelligence (AI) to automatically detect and classify objects, capturing a scene's complete geometry beyond basic features like floors or walls \cite{mortazaviEvaluatingSiteScanning2024a, geibWhyApplesLiDAR}. Its proprietary processing software, potentially integrated with APIs like Apple’s RoomPlan, converts these visualizations into editable CAD models, holding significant long-term value as the information remains accessible and reusable \cite{geibWhyApplesLiDAR}.

In the next section we will discuss magicplan app which is widely popular due to its speed, accuracy and tools. 