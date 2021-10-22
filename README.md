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
