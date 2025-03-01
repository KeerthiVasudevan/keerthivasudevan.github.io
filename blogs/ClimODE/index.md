---
layout: default
# title: ClimODE: Climate and Weather Forecasting With Physics-Informed Neural ODEs
---
# ClimODE: Merging Physics and AI for Weather and Climate Forecasting
## Table of Contents
- [Introduction](#introduction)
- [Background and Challenges](#background-and-challenges)
  - [The Traditional Approaches](#the-traditional-approaches)
  - [Why Deep Learning?](#why-deep-learning)
- [ClimODE](#climode)
  - [Overview](#overview)
  - [Core Methodology](#core-methodology)
  -   [Flow Velocity Model](#flow-velocity-model)
  -   [Advection Model](#advection-model)
  -   [Emission Model](#emission-model)
- [Other Contributions](#other-contributions)
  - [PDE-to-ODE Conversion](#pde-to-ode-conversion)
  - [Modeling Weather as a Continuous Spatio-Temporal System](#modeling-weather-as-a-continuous-spatio-temporal-system)
  - [Initial Velocity Inference](#initial-velocity-inference)
- [Experiments](#experiments)
  - [Benchmarking Models](#benchmarking-models)
  - [Evaluation Metrics](#evaluation-metrics)
  - [Dataset](#dataset)
- [Results](#results)
  - [Global Forecasting](#global-forecasting)
  - [Local Forecasting](#local-forecasting)
  - [Climate Forecasting](#climate-forecasting)
- [Ablation Studies](#ablation-studies)
  - [Effect of Emission Model](#effect-of-emission-model)
  - [Effect of Individual Components](#effect-of-individual-components)
- [Limitations and Future Work](#limitations-and-future-work)
- [Conclusion](#conclusion)
- [References](#references)

## Introduction
Weather forecasting is an essential aspect of modern life, influencing decisions on both personal and global scales. From planning a simple outdoor activity to issuing critical warnings for natural disasters like storms and heatwaves, accurate weather predictions are indispensable. Farmers rely on forecasts to optimize agricultural strategies, and industries such as aviation, logistics, and energy depend on reliable weather models for efficient operations.

Traditionally, weather forecasting has been driven by complex physical simulations, which, while accurate, require immense computational resources. In contrast, deep learning models offer a more data-driven approach but often act as black boxes, failing to incorporate fundamental physical laws that govern weather dynamics.

This blog explores **ClimODE** [1], a novel climate and weather forecasting model that bridges the gap between traditional numerical methods and deep learning. By integrating physics-based constraints with deep learning techniques, ClimODE offers a computationally efficient and physically consistent approach to long-term weather and climate prediction.

## Background and Challenges
### The Traditional Approaches

Traditional weather prediction primarily relies on Numerical Weather Prediction (NWP) models [2], which use physics-based simulations to forecast future weather conditions. By solving complex equations governing atmospheric dynamics, these models provide highly accurate short-to medium-range forecasts.

Despite their effectiveness, NWP models come with significant challenges. One of the biggest drawbacks is computational cost. High-fidelity simulations require immense computing power, often relying on supercomputers to run effectively. Additionally, sensitivity to initial conditions poses a fundamental limitation. Small errors in the initial weather state can grow exponentially, leading to inaccurate long-term forecasts. As a result, reliable forecasting is typically limited to about six days, beyond which predictions become increasingly uncertain.

<p align="center">
  <img src="assets/images/NWP.png" alt="Weather models use systems of differential equations based on physics, including fluid motion, thermodynamics, and radiative transfer. A 3D grid divides the Earth for calculations." width="500">
  <br>
  <em style="color: grey;">Figure 1: Traditional weather models simulate atmospheric dynamics using physics-based equations on a 3D grid, where complex equations are solved for each grid cell, along with calculations for interactions with neighboring cells. Source [2]</em>
</p>



### Why Deep Learning?
Deep learning has emerged as a promising alternative to traditional Numerical Weather Prediction (NWP) models by learning patterns from historical weather data rather than explicitly solving complex physics equations. This approach significantly reduces computational costs, making them more scalable and efficient. Additionally, these models excel at capturing intricate relationships between weather variables, leveraging vast datasets to uncover patterns and correlations.

Several deep learning models have been developed for weather and climate prediction, each employing different architectures to enhance forecasting capabilities. Examples of such models include:

- **FourCastNet** [4], a Fourier-based neural network model that specializes in global weather forecasting. 

- **ClimaX** [3], a transformer-based model, designed for climate modeling and general-purpose weather forecasting.

- **Pangu-Weather** [8], another deep learning-based model, that utilizes 3D Earth-specific transformer networks to achieve high-resolution predictions. 

While these models demonstrate the potential of deep learning in weather forecasting, they still suffer from key limitations. Many function as black-box models, making it difficult to interpret how they generate predictions. Additionally, most data-driven models do not explicitly incorporate physical constraints, leading to physically inconsistent or unrealistic forecasts. Another major drawback is uncertainty estimation—most deep learning approaches do not quantify uncertainty, making it difficult to assess the reliability of predictions. 

These limitations highlight the need for hybrid approaches that integrate deep learning with physics-based constraints, paving the way for **Physics-Informed Neural Networks (PINNs)**.

<p align="center">
  <img src="assets/images/PINN.png" alt="The intersection of Physics and Neural Networks gives rise to Physics-Informed Neural Networks (PINNs), combining the interpretability of physics-based models with the efficiency of neural networks." width="400">
  <br>
  <em style="color: grey;">Figure 2: Source [5]</em>
</p>


## ClimODE

### Overview

ClimODE presents a novel approach to weather forecasting by combining the strengths of deep learning with foundational principles of physics. Unlike traditional numerical weather prediction methods that rely on computationally expensive partial differential equations (PDEs), ClimODE employs a more efficient framework by converting these PDEs into ordinary differential equations (ODEs). This simplification reduces computational cost while maintaining accuracy, enabling scalable and stable long-term forecasting.

ClimODE also addresses the limitations of purely data-driven deep learning models, which often act as black boxes and lack physical consistency. By integrating real-world physics into its design, ClimODE ensures adherence to key principles such as conservation of mass and the continuity of weather variables, providing interpretable and reliable predictions. Moreover, the model captures the continuous and smooth nature of weather as a spatiotemporal process, avoiding errors that arise when weather variables are treated as discrete states.

### Core Methodology

As per statistical mechanics, weather can be seen as a spatial movement of quantities over time governed by the partial differential equation continuity equation.

<p align="center">
  <img src="assets/images/Continuity_eqn.png" alt="Continuity Equation: A foundational equation in weather modeling that describes the conservation of mass and movement of weather variables across time and space." width="400">
  <br>
  <em style="color: grey;">Figure 3: Continuity Equation</em>
</p>

ClimODE transforms the continuity equation into a physics-informed neural network by breaking it down into dedicated components, each handling a specific term of the equation:

- **Time Evolution**: This term represents how weather variables like temperature and humidity evolve over time. ClimODE captures this evolution through the Flow Velocity Model, which predicts the speed and direction of weather variable movement.
  
- **Advection**: This term describes the transport and compression of weather variables due to fluid motion, such as wind. ClimODE models this through the Advection ODE Model, ensuring the smooth spatiotemporal flow of weather variables across regions.

- **External Sources**: This term accounts for external factors like energy variations caused by the day-night cycle or seasonal effects. ClimODE handles this through the Emission Model, which incorporates bias and variance to capture systematic deviations and uncertainties.

#### Flow Velocity Model:

ClimODE uses a neural network to model the velocity field(v) and predict how weather variables (like temperature, pressure, etc.) evolve with time. The flow velocity model determines where and how fast weather variables move, essentially the speed and the direction. The model uses the current weather states, gradients that capture how the variables change spatially, and spatiotemporal embeddings, which incorporate geographical and temporal information, such as latitude, longitude, time, and seasonal context, to predict the same.

ClimODE leverages local convolutions and global attention mechanisms to capture the full spectrum of weather dynamics from local small-scale weather patterns to global-scale weather interactions. The global attention component enables the model to understand large-scale weather interactions, such as El Niño, a climate phenomenon originating in the Pacific Ocean that influences weather globally, causing droughts in Australia and heavy rainfall in South America. Simultaneously, local convolutions allow ClimODE to model small-scale patterns, such as sea breeze circulation in coastal areas, where temperature differences between land and sea drive localized wind patterns. These interactions affect rainfall, temperature, and air quality in nearby regions. By addressing both global and local phenomena, ClimODE provides a comprehensive framework for accurate weather prediction.

#### Advection Model:

At the core of the partial differential equations that model how weather evolves continuously in both time and space is the advection principle. This principle describes how weather variables, such as temperature and humidity, are transported through a fluid, like the atmosphere, due to its motion. In simpler terms, advection captures how these variables are carried by wind and flow across time and space.

Advection consists of two essential components: transport and compression. Transport tracks the movement of weather variables through the atmosphere, such as warm air shifting from one region to another due to wind. Compression, on the other hand, reflects changes in these variables caused by forces like convergence, where air flows into a region, or divergence, where it flows out. This process plays a vital role in phenomena like cloud formation and shifts in atmospheric pressure.

By explicitly integrating the physics of advection into its framework, ClimODE ensures adherence to the fundamental principles governing weather phenomena. It assumes the entire system to be a closed one that abides by the laws of conservation, meaning the total quantities of weather variables, such as temperature, remain constant and are neither created nor destroyed. This adherence to real-world physics not only enhances long-term forecasting stability but also prevents collapses in forecasts over extended time horizons, ensuring consistency with physical laws.

#### Emission Model:

If we limit weather modeling to the previously discussed components, two main problems emerge. First, the system is assumed to be deterministic, failing to account for uncertainties that can arise from various sources, such as unpredictable weather events. Second, the system is treated as closed, which does not allow for value changes like those caused by day-night cycles. To address these issues, ClimODE incorporates an additional step in its pipeline: the emission model.

The emission model outputs two key elements: bias and variance. The bias captures systematic deviations in the model, such as predictable changes during day-night cycles, and corrects for these deviations to enhance prediction accuracy. Meanwhile, the variance quantifies the uncertainty in weather predictions. This includes aleatoric uncertainty, which stems from the inherent randomness of weather systems, such as sudden storms or small-scale turbulence, as well as epistemic uncertainty, which arises from limitations in the model’s knowledge or data, such as insufficient observations. By explicitly modeling these uncertainties, ClimODE makes its predictions more robust and reliable in real-world scenarios.

<p align="center">
  <img src="assets/images/ClimODE_full_pipeline.png" alt="The full pipeline of ClimODE demonstrates its architecture, consisting of an Initial Velocity module, an Advection ODE model, and an Emission Model. The Advection ODE incorporates convolutional and attention-based mechanisms to model the weather variables. Outputs include velocity and weather states for subsequent time steps." width="800">
  <br>
  <em style="color: grey;">Figure 4: The full ClimODE pipeline. Source [1]</em>
</p>


### Other Contributions

#### PDE-to-ODE Conversion

One of the key contributions of ClimODE is its ability to simplify the traditionally complex numerical weather prediction pipeline. Conventional methods rely on solving computationally intensive partial differential equations (PDEs) to simulate weather patterns. In contrast, ClimODE applies methods discussed in prior research ([11] and [12]) to reformulate these PDEs as a system of first-order ordinary differential equations (ODEs). This transformation drastically reduces computational costs, making weather forecasting scalable and accessible without sacrificing accuracy. Using numerical solvers like Runge-Kutta [6], ClimODE handles the reduced ODE system efficiently, offering a computational advantage while staying true to the underlying physics.

#### Modeling Weather as a Continuous Spatio-Temporal System

ClimODE treats weather as a continuously evolving spatio-temporal system, aligning more closely with real-world dynamics. This approach ensures that variables such as temperature and humidity transition smoothly over time and space, avoiding abrupt, unrealistic changes that may arise from discretized modeling methods. By capturing this smooth evolution, ClimODE enhances the realism and accuracy of its forecasts.

#### Initial Velocity Inference

One critical step in ClimODE's framework is the estimation of initial velocities, which are required to start solving the system of Ordinary Differential Equations (ODEs). Traditionally, estimating velocity in dynamic systems is a challenging inverse problem that often requires complex encoders or approximations. However, ClimODE innovatively uses the continuity equation to directly infer these velocities. By framing it as an optimization problem, ClimODE ensures accurate and spatially consistent initial velocities, enabling it to effectively solve the governing equations while adhering to physical principles and maintaining computational efficiency.

## Experiments

### Benchmarking Models

ClimODE was benchmarked against state-of-the-art models, including ClimaX, a transformer-based model designed for weather and climate science, and FourCastNet, a data-driven weather forecasting model leveraging Fourier transforms. The model was also compared against a NeuralODE and the state-of-the-art Integrated Forecasting System (IFS), which is widely regarded as the gold standard in weather forecasting. While IFS is computationally expensive, it serves as a critical benchmark for measuring the accuracy of models. IFS is a numerical weather prediction (NWP) model and represents one of the most advanced global physics simulation systems.

### Evaluation Metrics

Performance comparisons were conducted using two primary evaluation metrics: Latitude-weighted Root Mean Square Error (RMSE) and Anomaly Correlation Coefficient (ACC). The Latitude-weighted RMSE, a modified version of the standard RMSE, accounts for the Earth's curvature by weighting errors based on latitude. This adjustment ensures that grids near the poles, which have a smaller surface area, do not disproportionately influence the overall result. On the other hand, ACC measures the model's ability to detect deviations from normal weather conditions. A higher ACC score reflects good performance in identifying and forecasting these deviations, which is critical for reliable and effective weather prediction.

### Dataset

ClimODE was trained and evaluated using a preprocessed ERA5 dataset, sourced from WeatherBench [9]. The dataset contains meteorological variables at a resolution of 5.625° and spans 6-hour increments for precise temporal modeling. 

The following five key meteorological variables were considered:
- Ground temperature (t2m)
- Atmospheric temperature (t)
- Geopotential (z)
- Ground wind vectors (u10, v10)

<p align="center">
  <img src="assets/images/data.png" alt="Input data structure showing latitude-longitude grid with meteorological variables." width="450">
  <br>
  <em style="color: grey;">Figure 5: Visualization of the data from WeatherBench used in ClimODE, showing weather variables (e.g., temperature) distributed across a latitude-longitude grid at a specific time step. Each channel represents a single variable at a specific vertical level. Source [9]</em>
</p>


The dataset was divided into distinct segments for training, validation, and testing purposes. Data from 2006 to 2015 was allocated for training the model, while the year 2016 was used for validation. The remaining years, 2017 and 2018, were reserved for testing to evaluate the model's performance on unseen data.

## Results

### Global Forecasting

ClimODE demonstrates remarkable performance in global weather forecasting, particularly in predicting crucial meteorological variables such as ground temperature, atmospheric temperature, and wind vectors. When benchmarked against other neural methods, ClimODE consistently outperforms them across evaluation metrics, showcasing the effectiveness of integrating a physics-informed approach into its architecture.

However, when compared to the gold-standard IFS, ClimODE falls short in overall accuracy. This is expected, as IFS is a highly advanced numerical weather prediction model with extensive computational resources dedicated to solving partial differential equations. Despite this gap, ClimODE proves its merit by offering a computationally efficient alternative while maintaining physical consistency in its predictions.


<p align="center">
  <img src="assets/images/Global_forecasting_result.png" alt="Global forecasting results comparing ClimODE against other models like NODE, ClimaX, FCN, and IFS. The metrics include RMSE and ACC for key meteorological variables such as z, v10, u10, t2m, and t, across different lead times. ClimODE demonstrates superior performance compared to neural methods while underperforming compared to IFS standards." width="800">
  <br>
  <em style="color: grey;">Figure 6: Performance comparison of ClimODE and baseline models for RMSE and ACC metrics. Source [1]</em>
</p>


### Local Forecasting

For Local Forecasting, ClimODE's capability in regional weather prediction has been evaluated over specific bounding regions, namely North America, South America, and Australia. These regions were chosen due to their diverse meteorological characteristics.

The results reveal that ClimODE performs exceptionally well in forecasting key meteorological variables such as ground temperature (t2m), atmospheric temperature (t), and geopotential height (z). Additionally, it maintains competitive performance in predicting wind vectors (u10, v10). This showcases ClimODE’s proficiency in modeling the finer-scale, localized weather dynamics that are critical for regional predictions.

### Climate Forecasting

Climate forecasting involves extended weather prediction over longer durations, typically spanning weeks or months. It aims to capture and predict average weather conditions over defined periods. ClimODE was evaluated for its climate forecasting capabilities by predicting monthly averages of key meteorological variables using the ERA5 dataset. For a fair comparison, the same dataset, variables, and hyperparameters were used.

Comparative analysis was done with FourCastNet, a data-driven model, which showed ClimODE's superior performance in predicting monthly averages, as demonstrated by metrics like Latitude-weighted RMSE and Anomaly Correlation Coefficient (ACC). ClimODE effectively captured long-term climate patterns with improved accuracy, highlighting its efficacy in climate forecasting along with short-term weather predictions.

<p align="center">
  <img src="assets/images/Monthly_forecast_results.jpg" alt="Monthly forecasting results comparing ClimODE and FCN across variables such as z, t, t2m, u10, and v10. Metrics are plotted as RMSE over lead times of 1 to 4 months, with ClimODE consistently outperforming FCN, especially for extended lead times." width="800">
  <br>
  <em style="color: grey;">Figure 9: Monthly Forecasting Results: Performance of ClimODE vs. FCN across key meteorological variables over different lead times. ClimODE outperforming FCN, especially for extended lead times. Source [1]</em>
</p>


## Ablation Studies

### Effect of Emission Model

The emission model effectively captures systematic deviations caused by the day-night cycle, as shown by the bias in Figure 7a. Positive bias is observed over the Pacific Ocean, reflecting daytime conditions, while negative bias around Europe and Africa signifies nighttime. Figure 7b further highlights that uncertainty is higher over land and in northern regions, demonstrating the model's ability to estimate variance accurately.

<p align="center">
  <img src="assets/images/ablation_emission.png" alt="Effect of Emission Model: Bias and Uncertainty Maps" width="800">
  <br>
  <em style="color: grey;">Figure 7: Model prediction for ground temperature(t2m). (a) Bias, showing systematic deviations due to the day-night cycle; and (b) Uncertainty, highlighting higher values over land and northern regions at 12:00 AM UTC. Source [1]</em>
</p>

### Effect of Individual Components

The aim of this study was to evaluate the contributions of individual components in ClimODE's architecture. The model was decomposed into the following parts:

- **NODE**: A free-form second-order neural ODE.  
- **Adv**: The advection ODE, responsible for modeling transport and compression.  
- **Att**: The global attention mechanism, designed to capture large-scale dependencies.  
- **ClimODE**: Combines all the above components, including the emission model.  
<p align="center">
  <img src="assets/images/ablation_components.png" alt="Performance comparison of NODE, NODE+Adv, NODE+Adv+Att, and ClimODE using RMSE as the evaluation metric. The graph shows that the advection and emission models contribute the most to performance improvements, while the global attention mechanism has the least impact." width="800">
  <br>
  <em style="color: grey;">Figure 8: Effect of individual components on ClimODE's performance. Source [1]</em>
</p>


The performance of **NODE**, **NODE+Adv**, **NODE+Adv+Att**, and **ClimODE** was compared using RMSE as the evaluation metric. The results showed that all components contribute significantly to overall performance. The advection and emission models had the most significant impact, while the global attention mechanism contributed the least.


## Limitations and Future Work
While ClimODE demonstrates impressive performance in weather prediction, it does have certain limitations. One notable drawback is that its accuracy still falls short compared to gold-standard numerical weather prediction models such as the Integrated Forecasting System (IFS). Additionally, ClimODE's evaluation has been geographically limited to regions like North America, South America, and Australia, leaving its performance in other areas, such as Europe and Asia, largely unexplored.

Another significant limitation is the model's lack of explicit consideration for long-term climate phenomena, such as climate change. This is critical for practical applications like disaster mitigation and policy planning. Furthermore, as mentioned in other research [10], ClimODE's performance in the subseasonal-to-seasonal (S2S) forecasting domain has been suboptimal.

For future work, the authors propose incorporating mechanisms to account for climate change and extending their methods to additional datasets, such as those mentioned by Rasp et al. (2023)[7], to better capture the dynamic nature of Earth's climate.

## Conclusion

ClimODE introduces a novel approach to weather and climate forecasting by integrating physics-based principles with deep learning. By transforming the continuity equation from Partial Differential Equations (PDEs) into a system of Ordinary Differential Equations (ODEs), ClimODE significantly reduces computational costs while preserving physical consistency. Its uncertainty quantification enhances reliability by providing confidence intervals for predictions, improving interpretability. Additionally, ClimODE effectively captures both local and global weather patterns and demonstrates robustness in both short-term and long-term forecasting. Its strong performance against competitive models like FourCastNet and ClimaX highlights its potential as a scalable and physics-informed alternative to traditional forecasting methods, paving the way for more accurate and efficient climate modeling.

Link to paper: [https://arxiv.org/abs/2404.10024](https://arxiv.org/abs/2404.10024)

## References
1. **ClimODE: Climate and weather forecasting with physics-informed neural ODEs** 
   Verma, Yogesh, Heinonen, Markus, and Garg, Vikas.  
   *arXiv preprint arXiv:2404.10024*, 2024.
   Available at: [https://arxiv.org/abs/2404.10024](https://arxiv.org/abs/2404.10024)

2. **Numerical Weather Prediction** 
   Wikipedia.  
   Available at: [https://en.wikipedia.org/wiki/Numerical_weather_prediction](https://en.wikipedia.org/wiki/Numerical_weather_prediction)

3. **ClimaX: A foundation model for weather and climate**  
   Nguyen, Tung; Brandstetter, Johannes; Kapoor, Ashish; Gupta, Jayesh K.; Grover, Aditya.  
   *arXiv preprint arXiv:2301.10343*. 2023.  
   Available at: [https://arxiv.org/abs/2301.10343](https://arxiv.org/abs/2301.10343)

4. **FourCastNet: A global data-driven high-resolution weather model using adaptive Fourier neural operators**  
   Pathak, Jaideep; Subramanian, Shashank; Harrington, Peter; Raja, Sanjeev; Chattopadhyay, Ashesh; Mardani, Morteza; Kurth, Thorsten; Hall, David; Li, Zongyi; Azizzadenesheli, Kamyar, et al.  
   *arXiv preprint arXiv:2202.11214*. 2022.  
   Available at: [https://arxiv.org/abs/2202.11214](https://arxiv.org/abs/2202.11214)

5. **Unlocking the Potential of Physics-Informed Neural Networks (PINNs)** 
   Justy Gwen.  
   *Medium*. Available at: [https://medium.com/@justygwen/unlocking-the-potential-of-physics-informed-neural-networks-pinns-68976b501415](https://medium.com/@justygwen/unlocking-the-potential-of-physics-informed-neural-networks-pinns-68976b501415)

6. **Über die numerische Auflösung von Differentialgleichungen**  
   Carl Runge.  
   *Mathematische Annalen*, 46(2):167–178, 1895.

7. **WeatherBench 2: A Benchmark for the Next Generation of Data-Driven Global Weather Models** 
   Stephan Rasp, Stephan Hoyer, Alexander Merose, Ian Langmore, Peter Battaglia, Tyler Russel, Alvaro Sanchez-Gonzalez, Vivian Yang, Rob Carver, Shreya Agrawal, et al.  
   *arXiv preprint arXiv:2308.15560*, 2023.  
   Available at: [https://arxiv.org/abs/2308.15560](https://arxiv.org/abs/2308.15560)

8. **Pangu-Weather: A 3D High-Resolution Model for Fast and Accurate Global Weather Forecast**   
   Kaifeng Bi, Lingxi Xie, Hengheng Zhang, Xin Chen, Xiaotao Gu, and Qi Tian.  
   *arXiv preprint arXiv:2211.02556*, 2022.  
   Available at: [https://arxiv.org/abs/2211.02556](https://arxiv.org/abs/2211.02556)

9. **WeatherBench: A Benchmark Data Set for Data-Driven Weather Forecasting** 
   Stephan Rasp, Peter D. Dueben, Sebastian Scher, Jonathan A. Weyn, Soukayna Mouatadid, and Nils Thuerey.  
   *Journal of Advances in Modeling Earth Systems*, Vol. 12, No. 11, e2020MS002203, 2020.  
   Available at: [https://agupubs.onlinelibrary.wiley.com/doi/full/10.1029/2020MS002203](https://agupubs.onlinelibrary.wiley.com/doi/full/10.1029/2020MS002203)

10. **Maximizing the Impact of Deep Learning on Subseasonal-to-Seasonal Climate Forecasting: The Essential Role of Optimization**  
   Yizhen Guo, Tian Zhou, Wanyi Jiang, Bo Wu, Liang Sun, and Rong Jin.  
   *arXiv preprint arXiv:2411.16728*, 2024.  
   Available at: [https://arxiv.org/abs/2411.16728](https://arxiv.org/abs/2411.16728)

11. **The Numerical Method of Lines: Integration of Partial Differential Equations**  
    William E. Schiesser.  
    *Elsevier*, 2012.

12. **Learning Continuous-Time PDEs from Sparse Data with Graph Neural Networks**  
    Valerii Iakovlev, Markus Heinonen, and Harri Lähdesmäki.  
    *arXiv preprint arXiv:2006.08956*, 2020.  
    Available at: [https://arxiv.org/abs/2006.08956](https://arxiv.org/abs/2006.08956)
