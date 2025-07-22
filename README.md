# Documentation: SPS Experiment Designer & Analyzer (v1.0.4)

This document provides an overview of the `sps.html` web-based tool. This program is designed to assist researchers in designing, analyzing, and optimizing experiments based on Response Surface Methodology (RSM). This documentation is based on version 1.0.4 of sps.html, dated June 28th, 2025.

---

## 1. Overview & Workflow

The application guides the user through a structured workflow for performing a Design of Experiments (DoE) analysis for a Spark Plasma Sintering (SPS) process, or any similar process that fits the model. The primary goal is to find the optimal set of process parameters (factors) to maximize one or more desired outcomes (responses), in this particular case Yield and Compacity. Obviously, if you replace the Yield (and/or Campacity) with another response value, the optimisation will be done for that value. Note that the input values are restricted: the Yield is >0 and <100. if you use other response parameters, you need to scale those values to fit in this range. 

The workflow is as follows:
1.  **Design Experiment**: The user defines the process factors (e.g., Temperature, Pressure, Dwell and heating rate), their minimum and maximum operating values, and selects a DoE method. The tool provides default factors for Pressure, Dwell Time, Heating Rate, and Temperature. This initial step is critical as it defines the experimental space to be investigated. If the min value = max value for one or more of the parameters, those parameters will not be used in the sampling. The number of points depends on the selected method, the simplest case is factorial with 2^n sampling points, see below.

2.  **Generate Plan**: This an experimental plan, listing the specific combination of factor settings for each run. This plan is algorithmically constructed to provide the most statistically relevant information with the minimum number of experiments, ensuring efficiency.

3.  **Enter Data**: The user performs the experiments according to the generated plan and enters the measured responses (Yield and Compacity) into the data table. A feature to fill the table with simulated data based on a pre-defined mathematical model is also available for validation and demonstration purposes. This allows users to familiarize themselves with the analysis workflow before generating their own data.

4.  **Analyze & Optimize**: Once you perform the experiments at the selected points (or any other points, these values can be edited)the tool automatically performs a regression analysis to create a mathematical model linking the factors to the responses. It calculates key statistical metrics, predicts optimal conditions, and visualizes the relationships using contour plots. This is the core analytical engine of the application.

5.  **Export**: A summary report of the data, models, and predicted optima can be exported as a PDF document for archiving, sharing, or inclusion in publications.

## 2. Design of Experiments (DoE) Methods

The tool implements three standard Response Surface Methodology (RSM) designs. RSM is a collection of statistical and mathematical techniques useful for developing, improving, and optimizing processes. The key idea is to explore the relationships between several explanatory variables (factors) and one or more response variables.

The tool supports the following designs, selectable from a dropdown menu:

### 2.1. Full Factorial Design
A full factorial experiment measures the response at all possible combinations of the factor levels. For *k* factors, each at 2 levels (min and max), this design consists of $2^k$ runs.

* **Purpose**: Excellent for screening studies aimed at identifying the main effects of factors and their interactions. It provides a comprehensive, though potentially large, examination of the design space.
* **Implementation**: The `generateFactorial` function creates this plan. It is primarily suited for fitting first-order models with interaction terms. The tool uses it as a base for the more complex Central Composite Design.

### 2.2. Box-Behnken Design (BBD)
A Box-Behnken design is an efficient, three-level design used for fitting second-order (quadratic) models. It does not contain runs at the extreme vertices of the experimental space, which can be advantageous when these points represent expensive or unsafe operating conditions.

* **Purpose**: To fit a quadratic model and find optima without needing to test extreme factor combinations. It is a highly efficient choice for optimization studies.
* **Structure**: The design is constructed by combining two-level factorial designs with incomplete block designs. The `generateBoxBehnken` function implements this by creating runs where factors are at their center points while pairs of other factors are at their min/max levels. It also includes several runs at the overall center point to improve the estimate of pure error.
* **Requirement**: Requires at least 3 varying factors to be generated.

### 2.3. Central Composite Design (CCD)
A CCD is the most common design used for building second-order models, it is efficient and flexible.

* **Purpose**: To efficiently estimate a second-order model, allowing for the detection of curvature in the response surface and the location of optimal process settings. It is considered the workhorse of response surface methodology for optimization.
* **Structure**: The `generateCentralComposite` function builds the design from three types of points:
    1.  **Factorial Points**: The corners of the cube, generated from a two-level factorial design (coded as -1 and +1). These points are the primary source of information for main effects and interactions.
    2.  **Center Points**: Replicate runs at the center of the experimental domain (coded as 0 for all factors). These are crucial for estimating pure experimental error and checking for model lack of fit and reproducibility issues.
    3.  **Axial (or Star) Points**: Points along the axes of the factors, set at a distance 'α' from the center. These points provide the ability to estimate quadratic terms, which is essential for modeling curvature.
* **CCD Type (α value)**: The tool allows for two types of CCDs by adjusting the value of $\alpha$:
    * **Face-Centered ($\alpha = 1.0$)**: The axial points are located on the "faces" of the factorial cube. This confines all runs within the specified min/max range for each factor.
    * **Rotatable ($\alpha = k^{1/4}$)**: Here, $\alpha$ is calculated based on the number of factors (*k*) to ensure that the variance of the predicted response is constant at all points equidistant from the center of the design. This property provides uniform prediction precision, which is ideal for optimization. Note that this can result in axial points that are outside the specified min/max range. This range extention might not be possible in some cases, the extended range can be adjusted by the user simply by editing the parameter is question. 


## 3. Mathematical Foundation

### 3.1. Coding of Variables
To simplify calculations and make the model coefficients directly comparable regardless of their original units, the real-world factor values (e.g., 800°C to 1000°C) are "coded" into a dimensionless range from -1 to +1. The `codeValue` function performs this transformation:
$$\text{Coded Value} = \frac{2 \times (\text{Real Value} - \text{Min Value})}{(\text{Max Value} - \text{Min Value})} - 1$$
The `uncodeValue` function reverses this process to present results in their original units.

### 3.2. The Model
The application uses a second-order polynomial model to describe the relationship between the factors and the response. For two factors, $X_1$ and $X_2$, the model is:
$$y = \beta_0 + \beta_1 X_1 + \beta_2 X_2 + \beta_{12} X_1 X_2 + \beta_{11} X_1^2 + \beta_{22} X_2^2 + \epsilon$$
Where:
* $y$ is the predicted response (e.g., Yield).
* $\beta_0$ is the intercept coefficient.
* $\beta_i$ are the linear effect coefficients.
* $\beta_{ij}$ are the interaction effect coefficients.
* $\beta_{ii}$ are the quadratic effect coefficients, which model curvature.
* $\epsilon$ is the random error term.

The `buildModel` function constructs this model, including linear, interaction, and quadratic terms for Box-Behnken and Central Composite designs.

### 3.3. Model Fitting: Method of Least Squares
The coefficients ($\beta$) of the polynomial model are determined using a simple least squares method. This is solved efficiently using matrix algebra. The equation is:
$$\mathbf{b} = (\mathbf{X}^T \mathbf{X})^{-1} \mathbf{X}^T \mathbf{y}$$
Where:
* $\mathbf{b}$ is the vector of model coefficients to be calculated.
* $\mathbf{X}$ is the design matrix, containing the coded values of the factor settings for each experimental run.
* $\mathbf{X}^T$ is the transpose of the design matrix.
* $(\mathbf{X}^T \mathbf{X})^{-1}$ is the inverse of the $\mathbf{X}^T \mathbf{X}$ matrix.
* $\mathbf{y}$ is the vector of observed experimental responses.

The application's `matrix` object contains functions for matrix multiplication (`multiply`), transposition (`transpose`), and inversion (`invert`) to perform this calculation within the `buildModel` function.

## 4. Detailed Analysis of Statistical Parameters

The `buildModel` function calculates several key statistics to evaluate the quality, significance, and predictive capability of the fitted second-order polynomial model. These are displayed in the "Statistics" section of the tool.

### 4.1. R-Squared (R²)
* **Definition**: Also known as the coefficient of determination, R-Squared measures the proportion of the total variation in the observed response variable that is explained by the model.
* **Interpretation**: An R² of 0.90 means that 90% of the response's variability is accounted for by the model's factors. While a higher R² is generally better, it should not be the sole indicator, as adding terms to a model will always increase R², regardless of the term's statistical significance. With 

### 4.2. Adjusted R-Squared (Adj R²)
* **Definition**: A modified version of R-Squared that has been adjusted for the number of predictors (terms) in the model. It is more suitable for comparing models with different numbers of terms.
* **Interpretation**: The Adjusted R² increases only if the new term improves the model more than would be expected by chance. It is a more reliable indicator of model adequacy than R² (remember von Neumann's elephant: "With four parameters I can fit an elephant, and with five I can make him wiggle his trunk.")

### 4.3. Predicted R-Squared (Pred R²)
* **Definition**: This statistic measures how well the model is expected to predict responses for new observations. It is calculated from the PRESS statistic (Predicted Residual Sum of Squares).
* **Interpretation**: A high Predicted R² indicates good predictive power. It should be in "reasonable agreement" with the Adjusted R². A large difference (e.g., > 0.2) between Adj R² and Pred R² suggests the model is overfitting and has poor predictive capability.

### 4.4. PRESS (Predicted Residual Sum of Squares)
* **Definition**: A cross-validation used to calculate Predicted R². It is calculated by summing the squares of the predicted residuals, where each residual is calculated from a model that was not fitted with that specific data point.
* **Interpretation**: A lower PRESS value is better, indicating smaller prediction errors. It is primarily an intermediate step for calculating Predicted R².

### 4.5. Adequate Precision (Adeq. Precision)
* **Definition**: This statistic measures the signal-to-noise ratio. It compares the range of the predicted values at the design points to the average prediction error.
* **Interpretation**: It indicates whether the model provides sufficient signal to be used for navigating the design space. **A ratio greater than 4 is generally considered desirable**. A value below 4 suggests the model should not be used for prediction or optimization.

### 4.6. Standard Deviation (Std. Dev.)
* **Definition**: The standard deviation of the residuals, representing the typical distance that the observed values fall from the fitted regression line. It is the square root of the Mean Square Error.
* **Interpretation**: A lower standard deviation is better, indicating a closer fit of the model to the data. It is an absolute measure of model error in the original units of the response.

### 4.7. Coefficient of Variation (C.V. %)
* **Definition**: The Coefficient of Variation is the standard deviation expressed as a percentage of the mean of the observed responses.
* **Interpretation**: It measures the relative amount of variation. A lower C.V. % is better. A value below 10-15% is often considered to indicate good model reproducibility.

### 4.8. Coefficient Statistics (Std. Error & 95% CI)
* **Definition**: These statistics assess the significance of each individual term (e.g., Pressure, Temperature*Temperature) in the model.
* **Interpretation**: The **95% Confidence Interval (CI)** is the key output here. **If the 95% CI range for a term contains zero, the term is generally considered not statistically significant.** This means its effect on the response cannot be distinguished from random noise at the 5% significance level.

### 4.9. Physical Interpretation of Model Coefficients
Beyond statistical significance, the sign and magnitude of the coefficients have important physical interpretations for process optimization.

* **Linear Terms (e.g., `Pressure`)**: These represent the primary slope of the response surface. A positive coefficient means the response increases as the factor's level increases. A negative coefficient means the response decreases. This indicates the primary direction for improvement.
* **Interaction Terms (e.g., `Pressure*Temperature`)**: These indicate that the effect of one factor is dependent on the level of another.
    * A **positive** coefficient implies **synergy**: the factors work together to enhance the response.
    * A **negative** coefficient implies **antagonism**: the factors work against each other.
* **Quadratic Terms (e.g., `Pressure*Pressure`)**: These terms model the curvature of the response surface. Their interpretation is crucial for finding optima.
    * A **NEGATIVE** coefficient is often **DESIRABLE** in a maximization problem. It indicates the response surface is curved downwards, like an inverted bowl. This curvature is what creates a peak, or a maximum point. A significant negative quadratic term is strong evidence that an optimal "sweet spot" for that factor exists.
    * A **POSITIVE** coefficient is often **UNDESIRABLE** in a maximization problem. It indicates the response surface is curved upwards, like a trough or valley. This suggests the optimal settings for that factor lie at the boundaries of the experimental range. This might suggest, if reasonable or physically possible, to expand the parameters' range.

## 5. Optimization

### 5.1. Single-Response Optimization
For individual responses (Yield or Compacity), the `findGlobalOptimum` function seeks to find the combination of factor settings that results in the maximum predicted response. This is achieved using a numerical optimization algorithm called `findOptimumRandomWalk`. The naumber of steps can be selected by the user; in normal conditions this does not change the output of the fit. For some shallow or strange systems you can go up to 100k steps.

### 5.2. Multi-Response Optimization (Weighted Score)
The tool handles the optimization of multiple responses simultaneously using a weighted sum approach. The user assigns an "importance" weight to Yield and Compacity using sliders. The `findCombinedOptimum` function then maximizes a combined score:
$$\text{Combined Score} = (w_{yield} \times \text{Predicted Yield}) + (w_{compacity} \times \text{Predicted Compacity})$$
This allows finding a "compromise" setting that provides the best overall outcome based on the user's specified preferences.

### 5.3. Analysis and Interpretation of the Combined Model
When the user assigns weights to both Yield and Compacity, a "Combined Model" is generated. It is critical to understand that this is **not** a specialized multi-response model but a standard model fitted to a synthesized response.

* **Derivation of the `weightedResponse`**: The foundation of this analysis is a new response variable calculated by the tool *before* any model is fitted. For each experimental run in your data table, the tool computes a `weightedResponse` value using the formula:
    `weightedResponse = (Weight_Yield * Observed_Yield) + (Weight_Compacity * Observed_Compacity)`
    This creates a new column of data that represents the user's explicit preference for a trade-off between the two original responses.
* **Model Fitting**: The application treats this `weightedResponse` vector as a single response variable. It fits a standard second-order polynomial model to this new data, using the experimental factors as the predictors. The resulting model, including all its coefficients and statistics, is what is presented under the "Combined Model" heading.
* **Interpretation of Combined Model Statistics**: The statistics for this model (R², Adj R², etc.) quantify how well the process factors predict this synthesized `weightedResponse` score.
    * A high **Adj R²** for the Combined Model indicates that a predictable relationship exists between the factor settings and your desired trade-off. It implies that a robust compromise solution can be modeled and found.
    * The **coefficients** of the Combined Model describe how factors influence your desired trade-off. For example, a significant negative quadratic term (e.g., `Pressure*Pressure`) in this model means that there is an optimal level of Pressure for achieving the best **compromise** between Yield and Compacity, according to your weights. It points to a peak on the "compromise surface".

## 6. Visualization
The tool generates 2D contour plots to help visualize the response surface.
* **Functionality**: The `draw2DContourPlot` function displays the relationship between two factors and the predicted response, with color gradients indicating the response value.
* **Interpretation**: The user can visually identify which factor settings lead to higher or lower responses. A star symbol (★) on the plot marks the location of the predicted mathematical optimum for that response.
* **Fixed Factors**: When creating a 2D plot for a design with more than two factors, the other factors are held constant at their center point (average) value to create a 2D "slice" of the multi-dimensional surface.

## 7. About the File
* **Filename**: sps.html
* **Version**: 1.0.4
* **Date**: June 28th, 2025
* **Author**: NitaD, Univ. Paris-Saclay