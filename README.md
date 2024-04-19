# Birnam-Wood by iGEM Calgary
Welcome to Birnam Wood, a program aimed to reduce the number of parameters 
required to make accurate kinetic rate constant predictions.

## Installation
The following packages must be installed prior to running Birnam Wood
```
pip install -U pandas 
pip install -U numpy 
pip install -U scikit-learn
```
## Usage
Start by cloning the reposititory by using Console or the Github Application.
Once installed, make sure the dataset files are accessible. First run all cells
within ```nested_cross_validation.ipynb``` this is a lengthy process as multiple 
models will be developed and evaluated. The outcome will be four spreadsheets,
one for dataOff and dataOn which contain each model containing the MSE value for 
each model, as well as the number of parameters. While DataOffJ and dataOnJ 
contain the resulting J score values. The first two columns of each dataset 
represent an integer key used to identify particular models.

Finally, an output will show the best models based off of the Jscore value. Use 
the model that you see fit. 

The next step is to gather data for your protein. More information regarding this
process can be found from our wiki page. 

Finally, once a csv is created with the proteins and its respected values, open 
up ```Birnam_Wood.ipynb``` and replace the name of our spreadsheet, ```lanM.csv```
and run the program to recieve a prediction. A sample has been provided for 
reproducibility. 

-------------------------------------------------------------------------------------
https://2021.igem.org/Team:Calgary/Birnam_Wood

## Small Steps to Big Leaps
With the increase in industrial applications of protein-ligand interactions, detailed modelling of these biological systems is becoming increasingly critical for pharmacological and biochemical processes. One important component of these models is the kinetic rate constant, a value that plays an important role in the modelling of our adsorber column. However, issues arise when calculating this value, as traditional methods, like simulating the association process between the protein and the ligand and characterizing the energy landscape, are prohibitively computationally expensive and time consuming [1]. Ideally, there would be a much quicker and simpler method of assessing the kinetic rate constant which would make the value more accessible to researchers accessing the viability of ideals early in the ideation phase.

### So what can be done?
An alternative to traditional methods uses feature selection algorithms to predict kinetic rate constants based on a given protein’s structural and energetic properties, calculated using its bound and unbound states [2]. Our software, Birnam Wood, builds on these existing feature selection algorithms by integrating an ensemble learning method with a resampling procedure that repeatedly evaluates the effectiveness of the model. Ultimately, this serves to reduce the runtime, number of required features, and accuracy of current industry standard predictions.

## What is a Random Forest? 
Birnam Wood uses the random forest (RF) ensemble learning method, which is an algorithm consisting of multiple decision trees. A decision tree consists of a series of branch statements which is filled up by a sample from a training dataset. Although the weakness of a decision tree is that it tends to overfit data giving poor predictions. Therefore, a random forest is used which creates multiple decision trees using random samples then averages the output values to make a prediction [3]. We decided to use this method as it presents better accuracy compared to other algorithms and runs efficiently on datasets with a large number of descriptors [4].

![image](https://github.com/wcjona/Birnam-Wood/assets/46095400/d3e52e65-af83-42d1-b40d-fcd3e645b25a)

*Figure 1. Diagram of a random decision forest*

### How will it work?
Because structural and energetic features of the protein-ligand interaction are calculated using the complex’s bound and unbound states [5]. Past models have used many different features to make predictions on the kinetic rate constant; however, we aimed to minimize the number of required features by reducing multicollinearity in the model, thereby reducing the dimensionality of the data. Multicollinearity occurs when independent variables have a high correlation with other variables, meaning that multiple features of a dataset could potentially describe similar aspects of the protein [6]. In turn, our model could overbias these descriptors, decreasing its accuracy. By reducing irrelevant and overlapping features, we found the smallest combination of parameters that yields the best results.

## Nested Cross-validation
To increase precision and reduce multicollinearity, we used nested cross-validation, which ideally would have required us to produce a unique machine learning model for every possible set of features. Since our dataset started off with 200 features, this would have required us to make 200 factorial, a number so large it is not worth typing out, machine learning models, which was deemed to be too resource and time expensive for our application. Additionally, this method would be entirely unscalable for larger datasets and difficult to reproduce.


Thus, to go around the brute force method, we developed our own interpretation to nested cross-validation using 200 nests. Similar to an upside down pyramid, the top of each nest begins with all 200 parameters, and at each level a parameter is pseudorandomly removed. In every level, we trained each model using our random forest model, then evaluated the model and gathered the importance values from each parameter. At each iteration, the program will inverse the importance values from the previous level and uses monte carlo sampling to pseudo-randomly remove a feature. Essentially this creates a random draw with a bias for removing unimportant parameters. The program then runs a new RF model with the remaining features, repeating this process until the number of features in the nest reaches zero also known as the bottom of the nest. Feature selection via a biased draw removes many of the ineffective models we would have seen through the initial brute force method. This way we were able to significantly reduce the runtime and the number of machine learning models to 40 thousand.

![image](https://github.com/wcjona/Birnam-Wood/assets/46095400/f86058e2-9768-4773-882c-81e269ef0e96)
*Figure 2. Nested Cross-Validation: where each Where each dot is a random decision forest*

## A New Evaluation Criterion
To evaluate the accuracy of each model, we planned on using mean squared error (MSE). Although we quickly realized that this evaluation neglects the number of features present in each model. Therefore, we decided to create our own evaluation criterion called J Score to evaluate models based on both MSE and the number of features. J Score evaluates each model by comparing its performance to the standard model with all 200 features, using MSE and the number of features.

![image](https://github.com/wcjona/Birnam-Wood/assets/46095400/d870ccd1-7a7f-4d8e-a492-86110caa61ca)
*Figure 3. Where “std” is the standard model, and “cur” is the current model and “m” is the mean squared error, while “n” is the number of features*

Using the formula for J Score shown in (1), the models which were less accurate (had a larger MSE value) than the original were assigned a negative score. While those which were both more accurate and had fewer features would result in a larger number with the most ideal model having a maximum value of 1. Therefore by looking at the largest J score values, we were able to find and filter out the best models from the monte carlo based nested cross-validation algorithm.

## Cutting Down Runtime
Additionally, to polish off our program, we decided to implement a lookup table (LUT) to our nested-cross validation algorithm which checks to see if a model with the same parameters has already been run. This used a numerical approach to feature tokenization in which each parameter was given a specific number. Using the sum and numerical concatenation of all the parameters we created two keys in which combined creates a unique key. A numerical representation for each parameter reduced the computational time since strings would be larger in memory. Although reducing computational time to this extent is not essential, it will significantly reduce our runtime when scaling our program to future larger datasets, features and nests.

## Dataset
Our training data, which was obtained from [7], consisted of kinetic rate constants for 44 complexes and a set of 200 molecular descriptors. These descriptors were calculated using a multitude of bioinformatic programs, using energetic models, hydrophobic burial, can see walls terms, and four-body or two body statistical potential [7]. These parameter values we’re used due to their correlation to binding affinity and kinetic rate constants.

## Results
Our program was able to produce results of all models for both kon and koff predictors. The final models selected due to their exceptional J Scores resulted in just two parameters for the kon prediction, and ten parameters for the koff prediction. The kon parameters were ‘Change in Atomic Surface Area upon binding’ (DASA), and the ‘General Four-Body Potential’ (GEN_4_BODY), with DASA coming from the NACCESS software suite (Hubbard & Thornton, 1993), and GEN_4_BODY coming from the Potentials’R’Us webserver [8]. The referenced kon model received a J Score of 0.936 including a MSE value of 0.0225. These results are a significant improvement compared to our standard model which received an MSE of 0.412. Our best koff predictor had a J Score of 0.882 and an MSE of 0.00748. Unfortunately, because of the precision of this model, we deemed it as overfitted and decided against using it for our official results for lanmodulin.

### So what does it all mean?
Using a kon predictor with accessible features, we were able to develop a kinetic rate constant prediction for lanmodulin and lanthanides, a protein-ligand interaction and the primary focus of iGEM Calgary 2021’s project. Using the value, we were able to calculate the estimated koff value by using the complex’s binding affinity from past literature (Cotruvo et al., 2018). Finally, these values were then used for simulation of the packed absorber column performance thus contributing to industrial-scale kinetic rate modeling and the characterization of lanmoduin.

## Reflection and Future Direction
Our models were successful in significantly reducing the computational burden of getting kinetic rate constants, showing a promising future for our nested cross-validation and scoring function. The next step in validating these findings is for the industrial application of the achieved models to obtain kon and koff parameters for our protein of interest, lanmodulin. At this step, however, we ran into some limitations for this specific data set and a challenge that faces the field of bioinformatics as a whole; reproducibility. For instance, NACCESS was only accessible by directly emailing its author, and the Potentials’R’Us web-server was no longer active, and its authors did not reply to inquiries into other methods of accessing the software. In situations like this, aside from attempting to rebuild the entire software from the original paper, there is no convenient way of using these discontinued programs, making older research very difficult to work with and validate. Without many of our parameters being inaccessible due to discontinued software, using our model as intended was not feasible with this data set. In the future, we would want to either find a more recent dataset that uses programs that are still currently available, or choose our own parameters from a well-established software suite, like Pyrosetta (Chaudhury et al., 2010), that will have a greater chance of being accessible in the future.

## References
Tiwary P, Limongelli V, Salvalaglio M, Parrinello M. Kinetics of protein ligand unbinding: Predicting. Proceedings of the National Academy of Sciences of the USA (PNAS). 2014 Dec 22 [accessed 2021 Oct 21]. https://www.pnas.org/content/pnas/early/2015/01/15/1424461112.full.pdf

Moal IH, Bates PA. Kinetic rate constant prediction supports the conformational selection mechanism of protein binding. PLOS Computational Biology. 2012 Jan 12 [accessed 2021 Oct 21].

Chaudhury S, Lyskov S, Gray JJ. PyRosetta: a script-based interface for implementing molecular modeling algorithms using Rosetta. International Society for Computational Biology. 2010 Jan 7 [accessed 2021 Oct 21]. https://academic.oup.com/bioinformatics/article/26/5/689/212442

Couronné R, Probst P, Boulesteix A-L. Random Forest versus logistic regression: A large-scale benchmark experiment. BMC bioinformatics. 2018 Jul 17 [accessed 2021 Oct 21]. https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6050737/

Moal IH, Agius R, Bates PA. Protein–protein binding affinity prediction on a diverse set of structures . International Society for Computational Biology. 2011 Sep 7 [accessed 2021 Oct 21]. https://academic.oup.com/bioinformatics/article/27/21/3002/218162

Brownlee J. Nested Cross-Validation for Machine Learning with Python. Machine Learning Mastery. 2021 [accessed 2021 Oct 21]. https://machinelearningmastery.com/nested-cross-validation-for-machine-learning-with-python/

Moal IH, Bates PA. Kinetic rate constant prediction supports the conformational selection mechanism of protein binding. PLOS Computational Biology. 2012 Jan 12 [accessed 2021 Oct 21]. https://journals.plos.org/ploscompbiol/article?id=10.1371%2Fjournal.pcbi.1002351

Feng Y, Kloczkowski A, Jernigan RL. Potentials 'r'us web-server for protein energy estimations with coarse-grained knowledge-based potentials. BMC Bioinformatics. 2010 Feb 17 [accessed 2021 Oct 21]. https://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-11-92
