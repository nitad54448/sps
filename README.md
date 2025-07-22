# SPS Experiment Designer & Analyzer
### Manual for Users (v1.0.4)

This document is a guide to the **SPS Experiment Designer & Analyzer** tool and it is split into two parts:

* **Part I: The User's Guide** will shwo how to use the tool, what the different options mean for your experiments, and how to interpret the results in a practical way.

* **Part II: The Technical Reference** is for understanding the inner workings of the program. This section contains the specific function names, mathematical equations, and detailed statistical formulas for the analysis.

---

# Part I: The User's Guide
## Getting Started and Interpreting Results


### **1. The Workflow: From Setup to Solution**

The tool is designed to follow the natural workflow of a scientific experiment.

1.  **Step 1: Design Your Experiment** 
    You can start by adjusting labels about your process. What can you control? This is where you define your factors names and values (like Temperature and Pressure) and the safe operating range for each (their minimum and maximum values). You'll then choose an experimental "recipe" or design that best fits your goals. There are manu options possible, only a few are included here.

2.  **Step 2: Generate the Experimental Plan** 
    The tool will generate a experimental plan for you to follow. This isn't a random set of experiments; it's a scientifically chosen set of runs designed to give you the most information about your process with the least amount of work. For best results (or any results) the runs must be reproducibible. IF some parameters suggested by the program are not always achieved: let's say the program suggests 1000 °C but you measured at 980 °C: you can edit the value suggested by the program and put 980 °C. It may not be optimal as design of experiment value but it may be used in subsequent calculations.

3.  **Step 3: Run Your Experiments and then enter data** 
    You need to perform the experiments exactly (or as much as possible) as laid out in the plan. Afterwards, you'll come back to the tool and enter your measured results (your "responses," like Yield and Compacity) into the data table. If you just want to try the tool, you can use the "Fill with Model Data" button to see how it works.

4.  **Step 4: Analyze, Optimize, and Understand** 
    The program analyzes your results and builds a mathematical model of your process if you fill (most of) the data table. It tells you how good that model is, which factors are most important, and predicts the best settings to achieve your goals. It also helps you find the perfect compromise when your are looking for two goals (if they are not completely divergent).

5.  **Step 5: Visualize and Export Your Findings** 📄
    Finally, you can explore your results with 2D contour plots to "see" how the factors affect the reaction outcome. When you're ready, you can export a PDF report of the analysis.

### **2. Setting Up Your Experiment**

A successful analysis begins with a well-defined experiment.

#### **2.1. Defining Your Process Factors**

In the first section of the tool, you define the parameters you can control.

* **Factor (Unit)**: Give your factor a descriptive name (e.g., "Temperature (°C)").
* **Min / Max**: Set the lowest and highest values you want to test for that factor. This defines your experimental "sandbox."
* **Varying vs. Fixed Factors**: If you set the Min and Max values to be the same, the tool will treat that factor as a fixed constant and will not include it in the optimization. For a factor to be studied, its Min and Max values must be different.

#### **2.2. Choosing the Right Experimental Design**

The tool offers three "recipes," or designs. Your choice depends on your goal.

* **Full Factorial Design**:
    * **When to use it**: This design is best for initial "screening" studies. Use it when you have a few factors and you want to know which ones are important and if they interact with each other. It tests every possible combination of your min/max settings.

* **Box-Behnken Design (BBD)**:
    * **When to use it**: This is a very efficient design for optimization. Choose this if you want a detailed model of your process but you need to **avoid** running experiments at extreme conditions (e.g., where both temperature and pressure are at their absolute maximum). It requires at least 3 factors.

* **Central Composite Design (CCD)**:
    * **When to use it**: This is the a powerful and popular design for optimization. It gives a complete picture of your process, including any curvature in the response, allowing you to reliably find the "peak" of your performance.
    * **CCD Type Option (Face-Centered vs. Rotatable)**: This option fine-tunes the design.
        * **Face-Centered**: Choose this if you absolutely must stay within your defined Min/Max limits for all experiments.
        * **Rotatable**: This is the default and usually the best choice. It gives you the most consistent prediction quality across your entire process space. *Note: This may suggest a few experimental runs that are slightly outside your initial Min/Max values.* You can always edit these suggested values in the data table if they are not feasible.

### **3. Understanding the results**

After you enter your data, the tool presents some statistical parameters, here's how to make sense of it.

#### **3.1. The Model's Report Card: Key Statistics Explained**

* **R-Squared (R²)**: This is an overall score (from 0 to 1) for how well the model fits your data. A value of 0.95 means the model explains 95% of the behavior you observed. Higher is better, but don't rely on this alone.
* **Adjusted R-Squared (Adj R²)**: This is the "honest" R-squared. It's a more reliable score because it accounts for how complex your model is. This is usually the best indicator of a good fit.
* **Predicted R-Squared (Pred R²)**: This tells you how well the model is likely to predict future experiments. If this value is close to your Adjusted R², you can be confident your model is robust and not just "lucky."
* **Adequate Precision**: This is a critical check. It measures the model's signal against its background noise. **You need this value to be greater than 4**. If it's not, your model is too noisy to be used for navigating to an optimum.


#### **3.2. The Levers of Your Process: Interpreting Model Terms**

The model gives you a table of "coefficients" for each term. These numbers tell you how the "levers" of your process work.

* **Linear Terms (e.g., `Pressure`)**: This is the main effect. A positive number means increasing pressure increases your yield. A negative number means it decreases it.
* **Interaction Terms (e.g., `Pressure*Temperature`)**: This tells you if factors work together or against each other.
    * **Positive value**: Synergy: the factors boost each other's effects.
    * **Negative value**: Antagonism : the factors fight each other.
* **Quadratic Terms (e.g., `Pressure*Pressure`)**: This is the most important term for optimization because it shows the curvature of the parameter surface.
    * A **NEGATIVE** value is **GOOD** for maximization! It means that the process has a "peak" or a "sweet spot." As you increase pressure, the yield goes up, hits a maximum, and then starts to fall. This term proves an optimum exists in relation with this parameter.
    * A **POSITIVE** value is **NOT GOOD** for maximization. It indicates a "valley" or a trough. This means the best results are at the extreme low or high ends of your pressure range, not in the middle.

#### **3.3. Finding the Best Settings: Optimization**

* **Single Goal**: If you only care about Yield, the tool will show you the predicted best settings to get the highest possible Yield.
* **Competing Goals (The Importance Sliders)**: What if the best settings for Yield are bad for Compacity? The sliders let you define your priorities. By setting them (e.g., 70% importance for Yield, 30% for Compacity), you ask the tool to find the single best **compromise** that honors your preferences. This is a mathematical compromise, see below.

#### **3.4. What is the "Combined Model"?**

When you use the importance sliders, a "Combined Model" appears. This isn't a model of Yield or Compacity. It's a model of **the preference**. It's built on a "Weighted Score" that combines your actual experimental results according to your slider settings. The statistics for this model tell you how predictable your desired **compromise** is. A good Combined Model means there is a clear, reliable path to achieving a good balance between your goals.

### **4. Visualizing Your Process: Contour Plots**

The 2D plots are your maps to the optimum.
* **Reading the Map**: The colors show your response value (e.g., red is high yield, blue is low). The axes are two of your factors. You can use these maps to see how changing two factors at once affects your outcome.
* **The Star (★)**: The star shows the exact predicted location of the maximum value for that response.
* **Other Factors**: If you have more than two factors, the plots are drawn by holding the other factors steady at their middle value to create a clear 2D slice.

### **5. Exporting Your Report**

When you're finished, you can click the "Save Summary as PDF" button to get a report with all your data, models, and optimal values. It's name is timestamped to the actual calculation.

---
---

# Part II: The Technical Reference
## Algorithms and equations

This section contains some mathematical and algorithmic details of the `sps.html` tool. 

### **6. Design of Experiments: Implementation Details**

The designs are generated by specific functions within the tool's code.

* **Full Factorial Design**: Implemented by the `generateFactorial` function. It creates a design matrix for *k* factors with $2^k$ runs.
* **Box-Behnken Design (BBD)**: Implemented by the `generateBoxBehnken` function. This function requires a minimum of 3 factors.
* **Central Composite Design (CCD)**: Implemented by the `generateCentralComposite` function. The `α` value that determines the position of the axial points is set according to the user's selection:
    * **Face-Centered**: A static value of $\alpha = 1.0$ is used.
    * **Rotatable**: The value is calculated dynamically using the formula $\alpha = k^{1/4}$, where *k* is the number of varying factors.

### **7. Mathematics and statistics**

#### **7.1. Variable Coding and Normalization**
Factor levels are normalized to a dimensionless [-1, 1] range to ensure that the scale of different factors does not influence the regression calculation.

* The `codeValue` function applies the transformation:
    $$\text{Coded Value} = \frac{2 \times (\text{Real Value} - \text{Min Value})}{(\text{Max Value} - \text{Min Value})} - 1$$
* The `uncodeValue` function reverses this transformation for reporting results in original units.

#### **7.2. The Second-Order Polynomial Model**
The `buildModel` function constructs and fits a second-order model to the response data. The general form of the model is:
$$y = \beta_0 + \sum_{i=1}^{k} \beta_i X_i + \sum_{i=1}^{k} \beta_{ii} X_i^2 + \sum_{i<j}^{k} \beta_{ij} X_i X_j + \epsilon$$
Where $y$ is the predicted response, $X_i$ are the coded factor values, $\beta$ are the model coefficients, and $\epsilon$ is the error term.

#### **7.3. Parameter Estimation via Least Squares**
The coefficients ($\beta$) of the polynomial model are determined using the method of Ordinary Least Squares (OLS). This is solved via matrix algebra for computational efficiency. The governing equation is:
$$\mathbf{b} = (\mathbf{X}^T \mathbf{X})^{-1} \mathbf{X}^T \mathbf{y}$$
Where $\mathbf{b}$ is the vector of estimated coefficients, $\mathbf{X}$ is the design matrix, and $\mathbf{y}$ is the vector of observed responses. The tool's internal `matrix` object provides functions for matrix multiplication (`multiply`), transposition (`transpose`), and inversion (`invert`) to perform this calculation.

### **8. Detailed Statistical Parameter Definitions**

* **R-Squared (R²)**: The coefficient of determination.
* **Adjusted R-Squared (Adj R²)**: R-Squared adjusted for the number of terms in the model relative to the number of data points.
* **Predicted R-Squared (Pred R²)**: Calculated from the PRESS statistic, it measures the model's predictive accuracy.
* **PRESS (Predicted Residual Sum of Squares)**: A cross-validation statistic used to calculate Pred R².
* **Adequate Precision**: A signal-to-noise ratio for the model. The tool requires this to be > 4.
* **Standard Deviation (Std. Dev.)**: The square root of the residual mean square error.
* **Coefficient of Variation (C.V. %)**: The standard deviation as a percentage of the response mean.
* **95% Confidence Interval (CI)**: The interval calculated for each coefficient. If it contains zero, the term is not considered statistically significant at the $\alpha=0.05$ level.

### **9. Optimization Algorithms**

The optimal conditions are found using specific numerical search functions.

* **Single-Response Optimization**: The `findGlobalOptimum` function is called, which utilizes the `findOptimumRandomWalk` algorithm to search the modeled response surface for a maximum.
* **Multi-Response Optimization**: The `findCombinedOptimum` function is used. It also leverages the `findOptimumRandomWalk` algorithm, but its objective function is to maximize the weighted sum of the predicted responses from the individual models. The number of steps for the random walk algorithm can be adjusted by the user to ensure convergence for complex response surfaces.

### **10. The `weightedResponse` Calculation**

For the "Combined Model" analysis, the `analyzeData` function first computes a composite response vector. For each row `i` in the experiment table, the calculation is:
`exp[i].weightedResponse = (w_yield * exp[i].yield) + (w_compacity * exp[i].compacity)`
This `weightedResponse` vector is then passed to the `buildModel` function as the `y` variable to generate the Combined Model statistics.

### **11. About the Tool**
* **Filename**: sps.html
* **Version**: 1.0.4
* **Date**: June 28th, 2025
* **Author**: NitaD, Univ. Paris-Saclay
