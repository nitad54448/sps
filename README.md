# Documentation: SPS Experiment Designer & Analyzer (v1.0.3)

This document provides a comprehensive overview of the `sps.html` web-based tool. The tool is designed to assist researchers in designing, analyzing, and optimizing experiments based on Response Surface Methodology (RSM). This documentation is based on version 1.0.3, dated June 25th, 2025.

---

## 1. Overview & Workflow

The application guides the user through a structured workflow for performing a Design of Experiments (DoE) analysis for a Spark Plasma Sintering (SPS) process, or any similar process that fits the model. The primary goal is to find the optimal set of process parameters (factors) to maximize one or more desired outcomes (responses), such as Yield and Compacity.

The user workflow is as follows:
1.  **Design Experiment**: The user defines the process factors (e.g., Temperature, Pressure), their minimum and maximum operating values, and selects a DoE method. The tool provides default factors for Pressure, Dwell Time, Heating Rate, and Temperature.
2.  **Generate Plan**: The tool generates a detailed experimental plan, listing the specific combination of factor settings for each run.
3.  **Enter Data**: The user performs the experiments and enters the measured responses (Yield and Compacity) into the data table. A feature to fill the table with simulated data based on a pre-defined mathematical model is also available for validation and demonstration purposes.
4.  **Analyze & Optimize**: The tool automatically performs a regression analysis to create a mathematical model linking the factors to the responses. It calculates key statistical metrics, predicts optimal conditions, and visualizes the relationships using contour plots.
5.  **Export**: A summary report of the data, models, and predicted optima can be exported as a PDF document.

## 2. Design of Experiments (DoE) Methods

The tool implements three standard Response Surface Methodology (RSM) designs. RSM is a collection of statistical and mathematical techniques useful for developing, improving, and optimizing processes. The key idea is to explore the relationships between several explanatory variables (factors) and one or more response variables.

The tool supports the following designs, selectable from a dropdown menu:

### 2.1. Full Factorial Design
A full factorial experiment measures the response at all possible combinations of the factor levels. For *k* factors, each at 2 levels (min and max), this design consists of $2^k$ runs.

* **Purpose**: Excellent for identifying the main effects of factors and their interactions.
* **Implementation**: The `generateFactorial` function creates this plan. It is primarily suited for fitting first-order models with interaction terms. The tool uses it as a base for the more complex Central Composite Design.

### 2.2. Box-Behnken Design (BBD)
A Box-Behnken design is an efficient, three-level design used for fitting second-order (quadratic) models. It does not contain runs at the extreme vertices of the experimental space, which can be advantageous when these points represent expensive or unsafe operating conditions.

* **Purpose**: To fit a quadratic model and find optima without needing to test extreme factor combinations.
* **Structure**: The design is constructed by combining two-level factorial designs with incomplete block designs. The `generateBoxBehnken` function implements this by creating runs where factors are at their center points while pairs of other factors are at their min/max levels. It also includes several runs at the overall center point.
* **Requirement**: Requires at least 3 varying factors to be generated.

### 2.3. Central Composite Design (CCD)
A CCD is the most common design used for building second-order models. It is highly efficient and flexible.

* **Purpose**: To efficiently estimate a second-order model, allowing for the detection of curvature in the response surface and the location of optimal process settings.
* **Structure**: The `generateCentralComposite` function builds the design from three types of points:
    1.  **Factorial Points**: The corners of the cube, generated from a two-level factorial design (coded as -1 and +1).
    2.  **Center Points**: Replicate runs at the center of the experimental domain (coded as 0 for all factors). These are crucial for estimating pure experimental error and checking for model lack of fit.
    3.  **Axial (or Star) Points**: Points along the axes of the factors, set at a distance 'α' from the center. These points provide the ability to estimate quadratic terms.
* **CCD Type (α value)**: The tool allows for two types of CCDs by adjusting the value of $\alpha$:
    * **Face-Centered ($\alpha = 1.0$)**: The axial points are located on the "faces" of the factorial cube. This confines all runs within the specified min/max range for each factor.
    * **Rotatable ($\alpha = k^{1/4}$)**: Here, $\alpha$ is calculated based on the number of factors (*k*) to ensure that the variance of the predicted response is constant at all points equidistant from the center of the design. This can result in axial points that are outside the specified min/max range.

## 3. Mathematical Foundation

### 3.1. Coding of Variables
To simplify calculations and make the model coefficients comparable, the real-world factor values (e.g., 800°C to 1000°C) are "coded" into a dimensionless range from -1 to +1. The `codeValue` function performs this transformation:
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
The coefficients ($\beta$) of the polynomial model are determined using the method of least squares. This is solved efficiently using matrix algebra. The equation is:
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
* **Formula Used**: $R^2 = 1 - \frac{SS_{Error}}{SS_{Total}}$. Where $SS_{Error}$ is the sum of squared residuals and $SS_{Total}$ is the total sum of squares.
* **Interpretation**:
    * The value ranges from 0 to 1 (or 0% to 100%).
    * An R² of 0.90 means that 90% of the response's variability is accounted for by the model's factors.
    * **A higher R² is generally better**. However, it should not be the sole indicator of a model's quality, as it can be misleadingly high.
* **Role**: Provides a quick assessment of the model's overall goodness-of-fit.

### 4.2. Adjusted R-Squared (Adj R²)

* **Definition**: A modified version of R-Squared that has been adjusted for the number of predictors (terms) in the model. It is more suitable for comparing models with different numbers of terms.
* **Formula Used**: $\text{Adj } R^2 = 1 - \left[ \frac{(1 - R^2)(n-1)}{n - p - 1} \right]$. Where *n* is the number of experimental runs and *p* is the number of model terms.
* **Interpretation**:
    * The Adjusted R² increases only if the new term improves the model more than would be expected by chance. It decreases if a non-significant term is added.
    * It is always lower than R². This value is more reliable than R² for evaluating model fit.
* **Role**: Helps prevent "overfitting" by penalizing the inclusion of unnecessary terms.

### 4.3. Predicted R-Squared (Pred R²)

* **Definition**: This statistic measures how well the model is expected to predict responses for new observations that were not used in the model fitting.
* **Formula Used**: $\text{Pred } R^2 = 1 - \frac{\text{PRESS}}{SS_{Total}}$. Where PRESS is the Predicted Residual Sum of Squares.
* **Interpretation**:
    * A high Predicted R² indicates good predictive power.
    * It should be in "reasonable agreement" with the Adjusted R². A large difference (e.g., > 0.2) between Adj R² and Pred R² suggests the model may be overfit.
* **Role**: Provides the most rigorous test of the model's predictive ability on new data.

### 4.4. PRESS (Predicted Residual Sum of Squares)

* **Definition**: A form of cross-validation used to calculate Predicted R². The tool uses an efficient mathematical equivalent that does not require refitting the model *n* times. It is calculated by summing the squares of the predicted residuals.
* **Interpretation**:
    * **A lower PRESS value is better**, indicating smaller prediction errors.
    * It is primarily used as an intermediate step to calculate the Predicted R².
* **Role**: Serves as the foundation for assessing the model's predictive performance.

### 4.5. Adequate Precision (Adeq. Precision)

* **Definition**: This statistic measures the signal-to-noise ratio. It compares the range of the predicted values at the design points to the average prediction error.
* **Formula Used**: $\text{Adeq. Precision} = \frac{\max(\hat{y}) - \min(\hat{y})}{\sqrt{\frac{p \times MS_{Error}}{n}}}$. Where $\hat{y}$ are the predicted values, *p* is the number of model terms, *n* is the number of runs, and $MS_{Error}$ is the Mean Square Error.
* **Interpretation**:
    * It indicates whether the model provides sufficient signal to be used for navigating the design space.
    * **A ratio greater than 4 is generally considered desirable**. A value below 4 suggests the model should not be used for prediction or optimization.
* **Role**: A critical check to ensure the model is not just fitting noise and has actual discriminative power.

### 4.6. Standard Deviation (Std. Dev.)

* **Definition**: The standard deviation of the residuals, representing the typical distance that the observed values fall from the fitted regression line.
* **Formula Used**: It is the square root of the Mean Square Error ($MS_{Error}$).
* **Interpretation**:
    * **A lower standard deviation is better**, indicating a closer fit of the model to the data.
    * This value is in the same units as the response variable, making it an intuitive measure of the average model error.
* **Role**: Provides an absolute measure of the model's typical error.

### 4.7. Coefficient of Variation (C.V. %)

* **Definition**: The Coefficient of Variation is the standard deviation expressed as a percentage of the mean of the observed responses.
* **Formula Used**: $\text{C.V. \%} = \frac{\text{Std. Dev.}}{\text{Mean}} \times 100$. The calculation is performed using the mean of the observed responses.
* **Interpretation**:
    * It measures the relative amount of variation.
    * **A lower C.V. % is better**. A value below 10-15% is often considered to indicate good model reproducibility.
* **Role**: Provides a standardized, relative measure of model error, independent of the response units.

### 4.8. Coefficient Statistics (Std. Error & 95% CI)

* **Definition**: These statistics assess the significance of each individual term (e.g., Pressure, Temperature*Temperature) in the model.
* **Formulas Used**:
    * **Standard Error**: The standard deviation of a coefficient's estimate, calculated as $\sqrt{MS_{Error} \times c_{ii}}$.
    * **95% Confidence Interval (CI)**: The range within which the true value of the coefficient is likely to fall. It is calculated as $\text{Coefficient} \pm (1.96 \times \text{Std. Error})$. The tool uses a fixed t-critical value of 1.96 as an approximation.
* **Interpretation**:
    * **Standard Error**: A smaller standard error indicates a more precise estimate for the coefficient.
    * **95% CI**: **If the 95% CI range for a term contains zero, the term is generally considered not statistically significant.** This means its effect on the response cannot be distinguished from random noise.
* **Role**: Allows for model reduction by identifying and potentially removing non-significant terms to create a simpler, more robust model.

## 5. Optimization

### 5.1. Single-Response Optimization
For individual responses (Yield or Compacity), the `findGlobalOptimum` function seeks to find the combination of factor settings that results in the maximum predicted response. This is achieved using a numerical optimization algorithm called `findOptimumRandomWalk`. The user can control the number of points and iterations via the "Search Precision" setting (Coarse, Fine, Extensive).

### 5.2. Multi-Response Optimization (Weighted Score)
The tool handles the optimization of multiple responses simultaneously using a weighted sum approach. The user assigns an "importance" weight to Yield and Compacity using sliders. The `findCombinedOptimum` function then maximizes a combined score:
$$\text{Combined Score} = (w_{yield} \times \text{Predicted Yield}) + (w_{compacity} \times \text{Predicted Compacity})$$
This allows finding a "compromise" setting that provides the best overall outcome based on the user's specified preferences.

## 6. Visualization
The tool generates 2D contour plots to help visualize the response surface.
* **Functionality**: The `draw2DContourPlot` function displays the relationship between two factors and the predicted response, with color gradients indicating the response value.
* **Interpretation**: The user can visually identify which factor settings lead to higher or lower responses. A star symbol (★) on the plot marks the location of the predicted mathematical optimum for that response.
* **Fixed Factors**: When creating a 2D plot for a design with more than two factors, the other factors are held constant at their center point (average) value.

## 7. About the File
* **Filename**: sps.html
* **Version**: 1.0.3
* **Date**: June 25th, 2025
* **Author**: NitaD, Univ. Paris-Saclay