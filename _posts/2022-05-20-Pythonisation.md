---
title: 'Python Tutorials of TMVA'
date: 20 May 2022
permalink: /posts/2014/08/Pythonisation/
---
# Writing Python Tutorials for various C files of Tutorials/TMVA

Hello, everyone! The purpose of this blog post is to describe you all through the journey of writing the python tutorial notebook for training, testing, evaluating and Drawing the ROC curve for all the C tutorials present in TMVA.

First we will start with fundamentals, So that even a newbie who is not familier with ROOT can get familiarize to it.

![image](https://user-images.githubusercontent.com/84740927/160606348-beaffd14-605c-48a9-bc4a-d425a6abdc2f.png)

## Installation Of Root 
Build ROOT from source following [this](https://root.cern/install/build_from_source/) tutorial. 
- Make sure TMVA is built with support for SOFIE (configure option -Dtmva-sofie=On) and requiring a Google protobuffer already installed in your system. 
- Make sure also support for Keras and/or PyTorch is enabled. For this you need to have Python installed with recent versions of Numpy, Tensorflow and/or  PyTorch
- The cmake command for the above implementation and tests is: 
```
cmake -Dtesting=ON -Droottest=ON -Dtmva-sofie=ON -Dbuiltin_xrootd=ON -DCMAKE_INSTALL_PREFIX=../install ../root_src
```
- Note: To import root through the python interpreter we need to make sure to run this command
 ```
cmake -DCMAKE_INSTALL_PREFIX=<installdir> -DPython3_EXECUTABLE=<python3interpreter> -DPython2_EXECUTABLE=<python2interpreter> <sourcedir>
 ```
 - You should be able to import root in your python interpreter
 ![image](https://user-images.githubusercontent.com/84740927/160602909-d7898138-2517-41cf-a7a6-37e81f846a4b.png)
 
 **Note:-**
 While building Root if you get the following error , then remove the example_outputs argument from PyRunString function over [here](https://github.com/root-project/root/blob/master/tmva/pymva/src/RModelParser_PyTorch.cxx#L422). It is described in the following [Pull request](https://github.com/root-project/root/pull/10223)
![image](https://user-images.githubusercontent.com/84740927/160625770-a06dbfef-6703-4f79-b550-1170163da469.png)


##  Importing ROOT in Jupyter Notebook using Python3

- If you don't have Jupyter Notebook installed on your local machine, you can do the same following the tutorial [here](https://docs.jupyter.org/en/latest/install/notebook-classic.html).
- Once the Jupyter Notebook is installed, you have to create a virtual environment by running the following commands
```
virtualenv jupyterenv
source jupyterenv/bin/activate
```
- Now move to the directory where you want the jupyter notebook to open and type the following command.
```
jupyter notebook
```
## Understanding How to use ROOT in a Jupyter notebook
You can refer the [this](https://root.cern.ch/notebooks/HowTos/HowTo_ROOT-Notebooks.html#:~:text=ROOT%20is%20integrated%20with%20the,with%20the%20%25%25cpp%20magic.) notebook on how to import Root and start working with Root in Jupyter Notebook.

## Description of my Task
Given a C++ tutorial, implement the corresponding tutorial in Python. 
Some example tutorial to convert to Python from C++ are (all available in the [root/tutorials/tmva](https://github.com/root-project/root/tree/master/tutorials/tmva) directory):
- TMVA_Higgs_Classification.C 
- TMVA_CNN_CLassification.C
- TMVA_RNN_Classification.C

## Running the C tutorials using ROOT
Go to the build directory where you have build the Root, then run the below command to activate ROOT
```
cd <builddir>
source ../install/bin/thisroot.sh
```
Now you can go to the directory where tutorials files are present in the root repository and run the following command
```
cd cd root/tutorials/tmva/
root <filename>
```
## Proposed Workflow for implementing the tutorials in Python
### Import the necessary modules
We start with importing the necessary modules required for the tutorial. Here we imported ROOT and TMVA(Toolkit for Multivariate Data Analysis). If you want to know more about TMVA, you can refer the documentation [here](https://root.cern.ch/download/doc/tmva/TMVAUsersGuide.pdf).

```python
import ROOT
from ROOT import TMVA 
```

![image](https://user-images.githubusercontent.com/84740927/160621521-1d8c7db2-1050-4f0d-bc30-be1bad636046.png)

### Setting up TMVA
TMVA requires initialization the PyMVA to utilize PyTorch. PyMVA is the interface for third-party MVA tools based on Python. It is created to make powerful external libraries easily accessible with a direct integration into the TMVA workflow. All PyMVA methods provide the same plug-and-play mechanisms as the TMVA methods. Because the base method of PyMVA is inherited from the TMVA base method, all options of internal TMVA methods apply for PyMVA methods as well.
```python
ROOT.TMVA.Tools.Instance()

## For PYMVA methods
TMVA.PyMethodBase.PyInitialize();

outputFile = ROOT.TFile.Open("CNN_ClassificationOutput.root", "RECREATE")

factory = ROOT.TMVA.Factory("TMVA_CNN_Classification", outputFile,
                      "!V:ROC:!Silent:Color:!DrawProgressBar:AnalysisType=Classification" )

```

### Define the input datafile
Take the input of the .root file in a variable if file exists. If the file doesn't exist, then download it through Cern box.

```python
inputFileName = "Higgs_data.root"

inputFile = ROOT.TFile.Open( inputFileName )
inputFileLink = "http://root.cern.ch/files/" + inputFileName

if (ROOT.gSystem.AccessPathName(inputFileName)!=None):
    # file exists
    inputFile = ROOT.TFile.Open( inputFileName )
if(inputFile == None): 
# download file from Cernbox location
    ROOT.Info("TMVA_Higgs_Classification","Download Higgs_data.root file")
    ROOT.TFile.SetCacheFileDir(".")
    inputFile = ROOT.TFile.Open(inputFileLink, "CACHEREAD")
    if (inputFile == NULL):
        Error("TMVA_Higgs_Classification","Input file cannot be downloaded - exit")

```

### Setting up the Signal and Background Trees
Here we are setting up the Training and testing Trees.
```python
# --- Register the training and test trees

signalTree = inputFile.Get("sig_tree")
backgroundTree = inputFile.Get("bkg_tree")

signalTree.Print()

```


**Note:-** Here, i was stuck with an error which took me a lot of time to find the correct solution to it. My mentor (**Lorenzo Moneta**) helped me finding the solution.

![image](https://user-images.githubusercontent.com/84740927/160624026-d652231b-8266-42b0-bbf2-3a3f6503fe5d.png)

I was not able to print the vars branch in the signal or background tree, also i couldn't find the vector<float> argument printed in the TTree. So, i was even not able to figure out that the error was in the input tree, i released the error in the train step. 
![image](https://user-images.githubusercontent.com/84740927/160624989-1b047d44-cc0a-4f96-8563-ffa9f8fecc6b.png)

This was the error i faced and it is also discussed in the root forum [here](https://root-forum.cern.ch/t/error-while-training-cnn-in-tmva/38228)
 
**Solution:**
The problem was in generating the TTree from python as input. It can be resolved using RDataFrame or after looking to the tutorials in [tutorials/tree](https://github.com/root-project/root/tree/master/tutorials/tree) directory or [tutorials/dataframe](https://github.com/root-project/root/tree/master/tutorials/dataframe). 
 
### Declare DataLoader
The next step is to declare the DataLoader class that deals with input variables

Define the input variables that shall be used for the MVA training note that you may also use variable expressions, which can be parsed by TTree::Draw( "expression" )]
```python
loader = TMVA.DataLoader("dataset")

loader.AddVariable("m_jj")
loader.AddVariable("m_jjj")
loader.AddVariable("m_lv")
loader.AddVariable("m_jlv")
loader.AddVariable("m_bb")
loader.AddVariable("m_wbb")
loader.AddVariable("m_wwbb")

### We set now the input data trees in the TMVA DataLoader class

# global event weights per tree (see below for setting event-wise weights)
signalWeight = 1.0
backgroundWeight = 1.0

#  You can add an arbitrary number of signal or background trees
loader.AddSignalTree    ( signalTree,     signalWeight     )
loader.AddBackgroundTree( backgroundTree, backgroundWeight )
```

### Applying additional cuts and setting up the Train and test dataset
Set individual event weights (the variables must exist in the original TTree)
for signal    : factory.SetSignalWeightExpression    ("weight1*weight2")
for background: factory.SetBackgroundWeightExpression("weight1*weight2")
loader.SetBackgroundWeightExpression( "weight" )
Tell the factory how to use the training and testing events. If no numbers of events are given, half of the events in the tree are used for training, and the other half for testing.


```python
# Apply additional cuts on the signal and background samples (can be different)
mycuts = ROOT.TCut("") # for example: TCut mycuts = "abs(var1)<0.5 && abs(var2-0.5)<1"
mycutb = ROOT.TCut("") # for example: TCut mycutb = "abs(var1)<0.5"

loader.PrepareTrainingAndTestTree( mycuts, mycutb, "nTrain_Signal=7000:nTrain_Background=7000:SplitMode=Random:NormMode=NumEvents:!V" )
```
 
### Booking Methods:-
There are various types of Booking Methods implemented in different tutorials. You can refer to the booking methods of other tutorials in the notebooks attached below. Here, i am mentioning the Booking methods of TMVA_Higgs_Classification.C
Here we book the TMVA methods. We book first a Likelihood based on KDE (Kernel Density Estimation), a Fischer discriminant, a BDT and a shallow neural network.

```python
 # Likelihood ("naive Bayes estimator")
if (useLikelihood):
   factory.BookMethod(loader, ROOT.TMVA.Types.kLikelihood, "Likelihood",
                           "H:!V:TransformOutput:PDFInterpol=Spline2:NSmoothSig[0]=20:NSmoothBkg[0]=20:NSmoothBkg[1]=10:NSmooth=1:NAvEvtPerBin=50" )

# Use a kernel density estimator to approximate the PDFs
if (useLikelihoodKDE):
   factory.BookMethod(loader, ROOT.TMVA.Types.kLikelihood, "LikelihoodKDE",
                      "!H:!V:!TransformOutput:PDFInterpol=KDE:KDEtype=Gauss:KDEiter=Adaptive:KDEFineFactor=0.3:KDEborder=None:NAvEvtPerBin=50" )

# Fisher discriminant (same as LD)
if (useFischer):
   factory.BookMethod(loader, ROOT.TMVA.Types.kFisher, "Fisher", "H:!V:Fisher:VarTransform=None:CreateMVAPdfs:PDFInterpolMVAPdf=Spline2:NbinsMVAPdf=50:NsmoothMVAPdf=10" )


# Boosted Decision Trees
if (useBDT):
   factory.BookMethod(loader,ROOT.TMVA.Types.kBDT, "BDT",
                      "!V:NTrees=200:MinNodeSize=2.5%:MaxDepth=2:BoostType=AdaBoost:AdaBoostBeta=0.5:UseBaggedBoost:BaggedSampleFraction=0.5:SeparationType=GiniIndex:nCuts=20" )


# Multi-Layer Perceptron (Neural Network)
if (useMLP):
   factory.BookMethod(loader, ROOT.TMVA.Types.kMLP, "MLP",
                      "!H:!V:NeuronType=tanh:VarTransform=N:NCycles=100:HiddenLayers=N+5:TestRate=5:!UseRegulator" )

```
### Booking a Deep Neural Network
Here we book the TMVA methods. We can book a DNN and a CNN .Here we are booking DNN, you can refer the example of CNN in other notebooks.
Here we book the new DNN of TMVA. If using master version you can use the new DL method.

Here we define the option string for building the Deep Neural network model.

   #### 1. Define DNN layout

   The DNN configuration is defined using a string. Note that whitespaces between characters are not allowed.

   We define first the DNN layout:

   - **input layout** :   this defines the input data format for the DNN as  ``input depth | height | width``.
      In case of a dense layer as first layer the input layout should be  ``1 | 1 | number of input variables`` (features)
   - **batch layout**  : this defines how are the input batch. It is related to input layout but not the same.
      If the first layer is dense it should be ``1 | batch size ! number of variables`` (features)

      *(note the use of the character `|` as  separator of  input parameters for DNN layout)*

   note that in case of only dense layer the input layout could be omitted but it is required when defining more
   complex architectures

   - **layer layout** string defining the layer architecture. The syntax is
      - layer type (e.g. DENSE, CONV, RNN)
      - layer parameters (e.g. number of units)
      - activation function (e.g  TANH, RELU,...)

      *the different layers are separated by the ``","`` *

   #### 2. Define Training Strategy

   We define here the training strategy parameters for the DNN. The parameters are separated by the ``","`` separator.
   One can then concatenate different training strategy with different parameters. The training strategy are separated by
   the ``"|"`` separator.

   - Optimizer
   - Learning rate
   - Momentum (valid for SGD and RMSPROP)
   - Regularization and Weight Decay
   - Dropout
   - Max number of epochs
   - Convergence steps. if the test error will not decrease after that value the training will stop
   - Batch size (This value must be the same specified in the input layout)
   - Test Repetitions (the interval when the test error will be computed)


   #### 3. Define general DNN options

   We define the general DNN options concatenating in the final string the previously defined layout and training strategy.
   Note we use the ``":"`` separator to separate the different higher level options, as in the other TMVA methods.
   In addition to input layout, batch layout and training strategy we add now:

   - Type of Loss function (e.g. CROSSENTROPY)
   - Weight Initizalization (e.g XAVIER, XAVIERUNIFORM, NORMAL )
   - Variable Transformation
   - Type of Architecture (e.g. CPU, GPU, Standard)

   We can then book the DL method using the built option string
 
```python
# Define DNN layout
inputLayoutString = "InputLayout=1|1|7"
batchLayoutString= "BatchLayout=1|32|7"
layoutString= "Layout=DENSE|64|TANH,DENSE|64|TANH,DENSE|64|TANH,DENSE|64|TANH,DENSE|1|LINEAR"
# Define Training strategies
# one can catenate several training strategies
training1  = "Optimizer=ADAM,LearningRate=1e-3,Momentum=0.,Regularization=None,WeightDecay=1e-4,"
training1 += "DropConfig=0.+0.+0.+0.,MaxEpochs=30,ConvergenceSteps=10,BatchSize=32,TestRepetitions=1"
#      training2 = ROOT.TString("LearningRate=1e-3,Momentum=0.9"
#                       "ConvergenceSteps=10,BatchSize=128,TestRepetitions=1,"
#                       "MaxEpochs=20,WeightDecay=1e-4,Regularization=None,"
#                       "Optimizer=SGD,DropConfig=0.0+0.0+0.0+0.");

trainingStrategyString = "TrainingStrategy="
trainingStrategyString += training1 # + "|" + training2
```
```python
 #  General Options.

dnnOptions = "!H:V:ErrorStrategy=CROSSENTROPY:VarTransform=G:"+"WeightInitialization=XAVIER"

dnnOptions +=  ":" + inputLayoutString
dnnOptions +=  ":" + batchLayoutString
dnnOptions +=  ":" + layoutString
dnnOptions +=  ":" + trainingStrategyString

dnnMethodName = "DNN_CPU"
if (useDLGPU):
 dnnOptions += ":Architecture=CPU"
 dnnMethodName = "DNN_CPU"
else:
 dnnOptions += ":Architecture=CPU
```
### Training All Methods
Finally you reached to the stage for which you were waiting! 
Here we train all the previously booked methods.
```python
factory.TrainAllMethods()                                                                  
```                                                                   
### Test  all methods
Now we test and evaluate all methods using the test data set     
```python
factory.TestAllMethods()
factory.EvaluateAllMethods()                                                                  
```   
### Plot the ROC curve
Here we plot the ROC curve and display it                                                                   
```python
c1 = factory.GetROCCurve(loader)
c1.Draw()                                                                   
```  
### Close the Output File
At the end we close the output file which contains the evaluation result of all methods and it can be used by TMVAGUI
to display additional plots                                                                    
```python
outputFile.Close()                                                                
```  
## Summary
![Untitled Diagram(5)](https://user-images.githubusercontent.com/84740927/167205398-bd860bde-d2ce-4d45-936b-b88f3adddfcb.jpg)

                                                                 
## Link to the Notebooks:
https://github.com/Neel-Shah-29/root-1/tree/Neel-Shah-Notebook/tutorials/tmva/Notebooks                                                                   
### Concluding from my side                                                                 
This blog describes my Journey to Pre-Gsoc work done for the Pythonization Project.At this point of time,irrespective of the outcome I am pretty happy that i learned a lot by contributing to this Root project. Now, i would definitely like to thank my mentor(**Lorenzo Moneta**) for guiding me throughout the process. If i get an opportunity to work with CERN, i would love to work on any of the 2 projects which i applied in the Root. Hope you all enjoy reading this Blog and learnt something from it!

So for the time being goodbye everyone! Will soon meet in my next blog. Until then 
                                                                   
Best Regards,
                                                                   
**Neel Shah**
