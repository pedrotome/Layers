## Authors

* Layers is written in C++ by **Roberto Paredes**
* Layers provides a prototype and script language in order to ease the networks definition and experimentation. This parser is written by **Jose Miguel Benedí**

## Basic

* There are 4 main parts in a layer program:
	* constants
	* data
	* networks
	* scripts



## Constants

* There are three different constants to define (default values):
	* threads: number of threads for parallelization (4)	* batch size: size of the batch for the network (100)	* log: log file where some messages are saved (netparser.log)

~~~c
const{
  threads=4
  batch=10
  log="mnist.log"
}
~~~

## Data

* Data can be read in ascii or binary. Data format is the following:

>  samples dim output
> 
>  sample1\_1 ... sample1\_dim out1\_1 .. out1\_c
> 
>  ...
> 
>  samplen\_1 ... samplen\_dim outn\_1 .. outn\_c

* _samples_ is the number of samples (int)
* _dim_ is the dimensionality of the samples (int) 
* _out_ is ths number of output targets, classes in a classification problem (int)


* each row is a sample (float values)

**Note**: when samples are color images the channels appear separated, firts R, then G and then B and _dim_ must be $3\times rows\times cols$


* An example of data definition:

~~~c
data {
  D1 [filename="../training", binary]
  D2 [filename="../test", binary]
  Dval [filename="../valid", ascii]
}
~~~

* _D1_, _D2_ and _Dval_ are the name of the data variables
* Full path to file can be used

## Networks

* Networks are the main part of a Layer program
* Networks are composed by three main parts:
	* Data
	* Layers
	* Connections
* The networks are defined like this:

~~~c
network N1 {  
   ...
}
~~~

* where _N1_ is the name of the network

## Networks - Data

* The Networks have to defined at least a training data set
* Test and validation data sets are optional:

~~~c
network N1 {  
  //data
  data tr D1  //mandatory
  data ts D2 //optional
  data va Dval //optional
  ...
}
~~~

* When test or validation data are provided the error function of the net will be also evaluated for that data sets

## Networks - Layers

* There are different layers that can be created:
	* **FI**:  Input Fully Connected layer
	* **CI**:  Input Covolutional layer
	* **F**:  Fully Connected layer
	* **FO**:  Ouput layer
	* **C**:  Convolutional layer
	* **MP**:  MaxPooling layer
	* **CA**:  Cat layer

### Networks - Layers - FI

* FI has not parameters just serve as a connection with the data

~~~c
//NETWORK
network N1 {  
  //data
  data tr D1  

  // Fully Connected Input 
  FI in 
  ...
}
~~~
* _in_ is the name of the layer
* The number of units is the dimensionality representation of the data

### Networks - Layers - CI

* Convolutional input has three mandatory parameters that indicate how the raw data has to be mapped into an input Map
* The parameters are: 
	* nz: number of channels
	* nr: image rows
	* nc: image cols

~~~c
//NETWORK
network N1 { 
  //data
  data tr D1 

  // Convolutional Input for MNIST
  CI in [nz=1, nr=28, nc=28]
  ...
}
~~~
* _in_ is the name of the layer

###  Networks - Layers -CI

* Convolutional input has optional parameters:
	* cr: crop rows
	* cc: crop cols
* Images will be randomly cropped on training phase
* Images will be center-cropped in test phase

~~~c
//NETWORK
network N1 { 
  //data
  data tr D1 

  // Convolutional Input for MNIST
  // Images will be randomly cropped to 24x24
  CI in [nz=1, nr=28, nc=28, cr=24, cc=24]
  ...
}
~~~


### Networks - Layers - F

* A fully connected layer

~~~c
//NETWORK
network N1 {  
  //data
  ...
  
  // Fully connected
  F  f1 [numnodes=1024]
  ...
}
~~~
* _f1_ is the name of the layer
* _numnodes_ is a mandatory parameter

### Networks - Layers - FO

* FO has a mandatory parameter _{classification,regression}_ that define the cost error: cross-entropy or mse respectivelly
*  For regression, optionally, we can define an _autoencoder_

~~~c
network N1 {  
  ...
  // Targets are the output in data (one-hot) 
  FO  out1 [classification]  // Cross-entropy

  // Targets are the output in data (real values) 
  FO  out2 [regression]  // mse over output

  // Targets are the same input in data
  FO  out3 [regression,autoencoder]	// mse over input
 ...
}
~~~

### Networks - Layers - C

* Convolutional layers have three mandatory parameters:
	* nk: number of kernels
	* kr: height of kernel
	* kc: width of kernel

~~~c
network N1 { 
  ...
  // Convolutional Layer
  C c1 [nk=32, kr=3, kc=3]	  
  ...
}
~~~

* _c1_ is the name of the layer
* Convolutional layers have the following optional parameters:
	* rpad: indicates if padding is done in the rows (0)
	* cpad: indicates if padding is done in the cols (0)
	* stride: stride value (1)

~~~c

network N1 { 
  ...
  // Convolutional Layer with padding
  // To keep the same size of Input Maps 
  // use rpad=cpad=1
  C c1 [nk=32, kr=3, kc=3,rpad=1,cpad=1]	  

  // Stride=2 and only padding in cols
  C c2 [nk=32, kr=3, kc=3,cpad=1,stride=2]	  
  ...
}
~~~

###  Networks - Layers - MP

* MaxPool layers have two mandatory parameters:
	* sizer: height of the pooling region
	* sizec: width of the pooling region

~~~c
network N1 { 
  ...
  // Maxpool layer sizer=sizec=2
  MP p1[sizer=2,sizec=2]
  ...
}
~~~

###  Networks - Layers - CA

* Cat layers does not require any parameter

~~~c
network N1 { 
  ...
  // CAT Layer
  CA cat
  ...
}
~~~

## Networks - Connections

* To connect two layers use the symbols "->"

~~~c
network N1 { 
  ...
  c1->c2
}
~~~

## Networks - Examples

#### Example 1 - MLP MNIST
~~~c
// A typical MLP for MNIST
// 784->1024->1024->1024->10

//CONSTANTS
const{
  threads=8
  batch=10
  log="mnist.log"
}

// DATA FILES
data {
  D1 [filename="../training", binary]
  D2 [filename="../test", binary]
}

//NETWORK
network N1 {  
  //data
  data tr D1  //mandatory
  data ts D2 //optional

  // Fully Connected Input 
  FI in 
  
  // Fully connected
  F  f1 [numnodes=1024]
  F  f2 [numnodes=1024]
  F  f3 [numnodes=1024]
  
  // Fully Connected Output
  FO  out [classification]  

  // Links
  in->f1
  f1->f2
  f2->f3
  f3->out
}
~~~

* layers generates a _dot_ file with the name of each network
* Use " dot -T pdf N1.dot > N1.pdf" to generate a PDF with the network:

![](./figs/MNIST_MLP.pdf)

* but yes, more complicated networks can be defined


#### Example 2 - Hybrid MLP MNIST

~~~c
//NETWORK
network N1 {
  //data
  data tr D1  
  data ts D2 

  // Fully Connected Input
  FI in

  // Fully connected
  F  f1 [numnodes=1024]
  F  f2 [numnodes=1024]
  F  f3 [numnodes=512]

  // Fully connected
  F  f4 [numnodes=1024]
  F  f5 [numnodes=512]
  F  f6 [numnodes=512]


  // Output
  FO  out [regression,autoencoder]
  FO  out2 [classification]

  // Connections
  in->f1
  f1->f2
  f2->f3
  f3->out

  in->f4
  f4->f2
  f4->out2

  f2->f5
  f5->f6
  f6->out

}
~~~

![](./figs/MNIST_MLP2.pdf)

#### Example 3 - Convolutional CIFAR

~~~c
network N1 {
  data tr D1
  data ts D2

  CI in [nz=3, nr=32, nc=32]

  C c0 [nk=16, kr=3, kc=3]
  MP p0[sizer=2,sizec=2]
  C c1 [nk=32, kr=3, kc=3]
  MP p1 [sizer=2,sizec=2]
  C c2 [nk=64, kr=3, kc=3]
  MP p2 [sizer=2,sizec=2]

  // FC reshape
  F   f0 []

  // FC hidden
  F  f1 [numnodes=128]

  // FC output
  FO f2 [classification]

  // Conecctions
  in->c0
  c0->p0
  p0->c1
  c1->p1
  p1->c2
  c2->p2
  //reshape
  p2->f0
  f0->f1
  f1->f2

}
~~~

![](./figs/CIFAR_CNN.pdf)

#### Example 4 - CAT layers 
~~~c
network N1 { 
  data tr D1 
  data ts D2

  CI in [nz=1, nr=39, nc=50]

  C c11 [nk=32, kr=1, kc=50,rpad=1]       
  C c12 [nk=32, kr=2, kc=50,rpad=1]       
  C c13 [nk=32, kr=3, kc=50,rpad=1]       
  C c14 [nk=32, kr=4, kc=50,rpad=1]       
  C c15 [nk=32, kr=5, kc=50,rpad=1]       
  MP p1 [sizer=2,sizec=1]        

  CA cat 
  
  // FC reshape
  F   fr1 []    

  // FC hidden
  F  fh1 [numnodes=512]
  F  fh2 [numnodes=256]

  // FC output
  FO fo1 [regression]

  // Connections
  in->c11
  in->c12       
  in->c13
  in->c14
  in->c15

  c11->cat
  c12->cat
  c13->cat
  c14->cat
  c15->cat

  cat->p1

  p1->fr1
  fr1->fh1
  fh1->fh2
  fh2->fo1

}

~~~


![](./figs/CAT.pdf)



to be continued...


