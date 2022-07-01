---
title: 'Documentation on RModelParser_ONNX.cxx'
date: 20 May 2022
permalink: /posts/2012/08/RModelParser_ONNX.cxx/
---

# Documentation on RModelParser_ONNX.cxx

Hello, everyone! The purpose of this blog post is to describe all the methods and functions used in RModelParser_ONNX.cxx and deep dive into its understanding!

Development of a fast inference system in TMVA that takes takes a trained ONNX model as input and produces compilation-ready standalone C++ scripts as output. These scripts will then provide users an easy way to deploy their deep learning models in their physics software and analysis frameworks.

ONNX represents models as an acyclic computation graph where each node is an operator. When traversing the graph the codegen function can be called for each node thereby generating model inference code. The RModelParser_ONNX parses the onnx model, and for each operator calls the corresponding codegen function.

Code generation for operators such as ReLU, Gemm, transpose, Conv, BatchNorm, InstanceNorm, RNN, LSTM, GRU are already implemented. There is a need to add code generation support for other operators to allow inference capability for more model architectures.

Note:- 
The prerequisite for running the Parse function are as follows.
- Protobuf 3.0 or higher (for input of ONNX model files)
- BLAS or Eigen (for execution of the generated code for inference)

### Include the header Files and libraries.

```C++
#include "TMVA/RModelParser_ONNX.hxx"
#include "onnx_proto3.pb.h"

#include <string>
#include <memory>
#include <cassert>
```

### Understanding Make_ROperator function
The make_ROperator function returns a unique pointer to the RModel operator which includes the nodeproto object , the graphproto object and the tensortype. std::unique_ptr is a smart pointer that owns and manages another object through a pointer and disposes of that object when the unique_ptr goes out of scope. 

The parameters of function includes the index where the node needs to be stored in the graph, the graphproto object and an unordered map of string and tensortype. Now here the string represents the type of operator and the tensor_type here comes from enum class ETensorType ( Tensor type can have data types as  UNDEFINED = 0, FLOAT = 1, UNINT8 = 2, INT8 = 3, UINT16 = 4, INT16 = 5, INT32 = 6, INT64 = 7, STRING = 8, BOOL = 9, FLOAT16 = 10, DOUBLE = 11, UINT32 = 12, UINT64 = 13, COMPLEX64 = 14, COMPLEX28 = 15, BFLOAT16 = 16)

Now we have factoryMethodMap defined under RModelParser_ONNX.hxx. We make an object of factoryMethodMap named mapOptypeOperator which contains the list of operators implemented and their respective make_ROperator functions.
```C++
using factoryMethodMap = std::unordered_map<std::string, std::unique_ptr<ROperator> (*)(const onnx::NodeProto&, const onnx::GraphProto&, std::unordered_map<std::string, ETensorType>&)>;
const factoryMethodMap mapOptypeOperator = {
   {"Gemm", &make_ROperator_Gemm},
   {"Transpose", &make_ROperator_Transpose},
   {"Relu", &make_ROperator_Relu},
   {"LeakyRelu", &make_ROperator_LeakyRelu},
   {"Conv", &make_ROperator_Conv},
   {"RNN", &make_ROperator_RNN},
   {"Selu", &make_ROperator_Selu},
   {"Sigmoid", &make_ROperator_Sigmoid},
   {"LSTM", &make_ROperator_LSTM},
   {"GRU", &make_ROperator_GRU},
   {"BatchNormalization", &make_ROperator_BatchNormalization},
   {"AveragePool", &make_ROperator_Pool},
   {"GlobalAveragePool", &make_ROperator_Pool},
   {"MaxPool", &make_ROperator_Pool},
   {"Add", &make_ROperator_Add},
   {"Reshape", &make_ROperator_Reshape},
   {"Flatten", &make_ROperator_Reshape},
   {"Slice", &make_ROperator_Slice},
   {"Squeeze", &make_ROperator_Reshape},
   {"Unsqueeze", &make_ROperator_Reshape},
   {"Flatten", &make_ROperator_Reshape}
};
```
We map through the mapOptypeOperator to find the operator type. If till the end we don't get the operator type we throw an error.
If we got the operator type we return the nodeproto object, graphproto object and the tensortype of operator.
```C++
std::unique_ptr<ROperator> make_ROperator(size_t idx, const onnx::GraphProto& graphproto, std::unordered_map<std::string, ETensorType>& tensor_type){
   const auto& nodeproto = graphproto.node(idx);
   auto find = mapOptypeOperator.find(nodeproto.op_type());
   if (find == mapOptypeOperator.end()){
      throw std::runtime_error("TMVA::SOFIE - Operator type " + nodeproto.op_type() + " is not yet supported");
      // std::unique_ptr<ROperator> op;
      // return op;
   } else {
      //std::cout << "create operator " << nodeproto.op_type() << std::endl;
      return (find->second)(nodeproto, graphproto, tensor_type);
   }
}
```
### Understanding the basic working of operators 
We will not go into the detailed understanding of all the operators here but this blog will just throw light on basic implementation of operators in RModelParser_ONNX.cxx
According to the code snippet mentioned below for the Relu Operator. The parameters and the return type of Relu operator is same as the make_ROperator function. We define a inputtype of enum class ETensortype and declare input name from the nodeproto object which we provided as input.Here we iterate over the map tensor_type to find the input_name its present as the values in the unordered map. If we don't find the particular input type, we throw an error! If we found the input_name then we assign it to the input type
```C++
 std::unique_ptr<ROperator> make_ROperator_Relu(const onnx::NodeProto& nodeproto, const onnx::GraphProto& /*graphproto */, std::unordered_map<std::string, ETensorType>& tensor_type){

   ETensorType input_type;

   auto input_name =  nodeproto.input(0);
   auto it = tensor_type.find(input_name);
   if (it != tensor_type.end()){
      input_type = it->second;
   }else{
      throw std::runtime_error("TMVA::SOFIE ONNX Parser relu op has input tensor" + input_name + " but its type is not yet registered");
   }
   ```
Now we declare an operator op and use the switch statement to check whether the input_type is valid or not. Now if we found the input_type to a valid ETensortype which is mentioned in switch case then we reset the operator with the provided input and output otherwise we throw an error stating input type is not supported.
   ```C++
   std::unique_ptr<ROperator> op;


   switch(input_type){
   case ETensorType::FLOAT:
      op.reset(new ROperator_Relu<float>(nodeproto.input(0), nodeproto.output(0)));
      break;
   default:
      throw std::runtime_error("TMVA::SOFIE - Unsupported - Operator Relu does not yet support input type " + std::to_string(static_cast<int>(input_type)));
   }
```
Moving on to the output ,we declare the output type and assign it the value by calling TypeInference function reference to the operator. The TypeInference function checks the type of input and returns the value, its defined for each operators indivitually in their header files. We add the output to the map tensor_type if it does not exist in it.

We add the output of the operator to the unordered map tensor_type so that it can be used as an input_type in other operators. So basically the output of an operator serves as an input for some other operator.

At the end we return the operator.
```C++
   ETensorType output_type = (op->TypeInference({input_type}))[0];
   auto it2 = tensor_type.find(nodeproto.output(0));
   if (it2 == tensor_type.end()){
      tensor_type[nodeproto.output(0)] = output_type;
   }

   return op;
}
```
### Understanding of Parse Function
The Parse function returns the RModel object and takes the filename as the parameter.
We use the rfind method to find the last occurance of '/' or '\\\\' according to operating system and get the filename in a variable.
The std::string::rfind is a string class member function that is used to search the last occurrence of any character in the string. If the character is present in the string then it returns the index of the last occurrence of that character in the string else it will return string::npos which denotes the pointer is at the end of the string.
We also record the time using some standard c++ functions.
```C++
RModel RModelParser_ONNX::Parse(std::string filename){
   char sep = '/';
   #ifdef _WIN32
      sep = '\\';
   #endif
   size_t isep = filename.rfind(sep, filename.length());
   std::string filename_nodir = filename;
   if (isep != std::string::npos){
      filename_nodir = (filename.substr(isep+1, filename.length() - isep));
   }

   std::time_t ttime = std::time(0);
   std::tm* gmt_time = std::gmtime(&ttime);
   std::string parsetime (std::asctime(gmt_time));
```
Now we declare a ONNX model and define an object of RModel class. We declare an unordered map tensor_type which takes input a string and the type of tensor from enum class ETensorType.
Now we read the input file using fstream function in input or binary format. If the input file is unable to parse from the Istream which is checked by function ParseFromIstream then we throw an error!
Now we declare a graph object and shut down the protobuf Library as we don't require the method anymore so we clear the stream.
 ```C++
 GOOGLE_PROTOBUF_VERIFY_VERSION;
   //model I/O
   onnx::ModelProto model;
   RModel rmodel(filename_nodir, parsetime);

   std::unordered_map<std::string, ETensorType> tensor_type;

   std::fstream input(filename, std::ios::in | std::ios::binary);
   if (!model.ParseFromIstream(&input)){
      throw std::runtime_error("TMVA::SOFIE - Failed to parse onnx file");
   }

   const onnx::GraphProto& graph = model.graph(); //not a memory leak. model freed automatically at the end.
   google::protobuf::ShutdownProtobufLibrary();

   // ONNX version is ir_version()  - model_version() returns 0
   // std::cout << "ONNX Version " << model.ir_version() << std::endl;
```
### What are initializers and why are they used?
The Initialisers are basically the initialized tensors  with data that are stored in ONNX and they are typically the weights of the model.
Initializer is a list of named tensor values. When an initializer has the same name as a graph input, it specifies a default value for that input. When an initializer has a name different from all graph inputs, it specifies a constant value. The order of the list is unspecified.

Here we define an unordered set of initializer names and insert the names of initialisers into it.
```C++
std::unordered_set<std::string> initializer_names;
   for (int i=0; i < graph.initializer_size(); i++){
      initializer_names.insert(graph.initializer(i).name());
   }
```
Here we loop through all the input values, add the input name as a value and the type of input i.e the ETensortype as key in the map. We should note that the input datanode is not the weight node of the model i.e it does not consist of an initializer.
We define an object of class valueinfoproto and assign the input from the graph to it. Then we create a variable to store the name of input and now the name can be accessed by object of class ValueInfoProto.

Now we check whether the Datatype of input is supported by SOFIE, i.e it should be either INT64,INT32 or FLOAT, if anything else occurs it throws an error!
Now we declare a vector of structure dimensions named fShape and boolean variable to check the existing parameters. If the input type does not have any shape we throw an error!
```C++

   for (int i=0; i < graph.input_size(); i++){

      tensor_type[graph.input(i).name()] = static_cast<ETensorType>(graph.input(i).type().tensor_type().elem_type());

      if (initializer_names.find(graph.input(i).name()) != initializer_names.end())  continue;

      //input datanode is not a weight node (has no initializer)
      const onnx::ValueInfoProto& valueinfoproto = graph.input(i);
      std::string input_name = valueinfoproto.name();

      ETensorType type = static_cast<ETensorType>(valueinfoproto.type().tensor_type().elem_type());
      if (type != ETensorType::FLOAT && type != ETensorType::INT32 && type != ETensorType::INT64) {
         throw std::runtime_error("TMVA::SOFIE Data type in input tensor " + input_name + " not supported!\n");
      }

      std::vector<Dim> fShape;
      bool existParam = false;
      if (!valueinfoproto.type().tensor_type().has_shape()) throw std::runtime_error("TMVA::SOFIE datanode with no shape restrictions is not supported yet");
```
Now we iterate through the Valueinfoproto object for all the values until the iterator becomes equal to dimension size. 
Now, we declare an object of structure Dim, it has 3 members:
- isParam which is set to false by default (bool)
- dim (size_t)
- param(string)

then we check whether the dimension  of Valueinfoproto object is equal to KDimValue which is set to 1, if it satisfies the condition then we assign the dimension values to member dim of structure object dim defined by us earlier.
else we check whether the dimension  of Valueinfoproto object is equal to KDimParam which is set to 2, if it satisfies the condition then we assign the dimension values to member param of structure object dim defined by us earlier and set existparam and isParam(member of struct dim) value as true.
If any of above condition don't satisfy then we throw an error that value is neither dimension nor parameter.
```C++
      for (int j = 0; j < valueinfoproto.type().tensor_type().shape().dim_size(); j++){
         Dim dim;
         if (valueinfoproto.type().tensor_type().shape().dim(j).value_case() == onnx::TensorShapeProto_Dimension::ValueCase::kDimValue){
            dim.dim = valueinfoproto.type().tensor_type().shape().dim(j).dim_value();
         }else if (valueinfoproto.type().tensor_type().shape().dim(j).value_case() == onnx::TensorShapeProto_Dimension::ValueCase::kDimParam){
            dim.isParam = true;
            existParam = true;
            dim.param = valueinfoproto.type().tensor_type().shape().dim(j).dim_param();
         }else{
            throw std::runtime_error("TMVA::SOFIE ONNX file error: Valueinfoproto " + input_name + " has neither dim_value nor dim_param! \n");
         }
         fShape.push_back(dim);
      }
```
Now we check if dimension size is 0 if its 0 then ONNX don't throw error rather ONNX IR defines this to be a scalar,then we declare an object of struct Dim and set member dim as 1 and we push the dim to the vector fShape.
Now we check if the parameters exists, if yes then we make a new vector and store the dimensions in it and add the Input Tensor to rmodel object with parameters as input name , its type and the vector of dimensions with parameters.if there are no parameters then add the input tensor with input name , its type and the vector of dimensions without parameters(fshape).
```C++
      if (valueinfoproto.type().tensor_type().shape().dim_size() == 0){
         Dim dim;
         dim.dim = 1;
         fShape.push_back(dim);
      } //in case this TensorShapeProto has no dimension message: ONNX IR defines this to be a scalar

      if (!existParam){
         std::vector<size_t> fShape_sizet;
         for (auto& j: fShape){
            fShape_sizet.push_back(j.dim);
         }

         rmodel.AddInputTensorInfo(input_name, type, fShape_sizet);
      }else{
         rmodel.AddInputTensorInfo(input_name, type, fShape);
      }

   }
```
Here we iterate through all the initializers, we make an object of class Tensorproto and in each iteration assign the value of graph initializer to that object.We declare a vector fShape and the variable fLength to store the dimensions of tensorproto object.
We store the name of initializer in variable input_name.We put the switch case according to the datatype,if the datatype is Float, We declare a shared pointer data and provide the size dynamically to it.Then we check whether the raw data i.e the weights of model is not empty, then we copy the data using std::memcpy function, it copies count bytes from the object pointed to by src to the object pointed to by destination. Both objects are reinterpreted as arrays of unsigned char.
If the datatype is not float by typecasting and other functions but if its still not converted then we throw an error!
We add the initialised tensor to rmodel object if datatype is supported.
```C++
   for (int i=0; i < graph.initializer_size(); i++){
      onnx::TensorProto* tensorproto = const_cast<onnx::TensorProto*>(&graph.initializer(i));
      std::vector<std::size_t> fShape;
      std::size_t fLength = 1;
      for (int j = 0; j < tensorproto->dims_size(); j++){
         fShape.push_back(tensorproto->dims(j));
         fLength *= tensorproto->dims(j);
      }

      std::string input_name = graph.initializer(i).name();

      switch(static_cast<ETensorType>(graph.initializer(i).data_type())){
         case ETensorType::FLOAT : {
            //void* data = malloc (fLength * sizeof(float));
            std::shared_ptr<void> data(malloc(fLength * sizeof(float)), free);

            if (tensorproto->raw_data().empty() == false){
               auto raw_data_ptr = reinterpret_cast<float*>(const_cast<char*>(tensorproto->raw_data().c_str()));
               std::memcpy(data.get(), raw_data_ptr, fLength * sizeof(float));
            }else{
               tensorproto->mutable_float_data()->ExtractSubrange(0, tensorproto->float_data_size(), static_cast<float*>(data.get()));
            }

            rmodel.AddInitializedTensor(input_name, ETensorType::FLOAT, fShape, data);
            break;
         }
         default: throw std::runtime_error("Data type in weight tensor " + graph.initializer(i).name() + " not supported!\n");
      }
   }

```
Here we iterate through all nodes of the graph and each node symbolizes an operator so we use the make_ROperator function to initialise the operator and we add the operator to rmodel object.
Now, The BLAS (Basic Linear Algebra Subprograms) are routines that provide standard building blocks for performing basic vector and matrix operations. The Level 1 BLAS perform scalar, vector and vector-vector operations, the Level 2 BLAS perform matrix-vector operations, and the Level 3 BLAS perform matrix-matrix operations. Because the BLAS are efficient, portable, and widely available, they are commonly used in the development of high quality linear algebra software.
This BLAS library provides some special frameworks and functions which works way faster then our computational algebra so for using that we need to add Blas Routines to rmodel object whichever we require for according to the onnx operator.
```C++
for (int i=0; i < graph.node_size(); i++){
      auto op = INTERNAL::make_ROperator(i, graph, tensor_type);
      if (!op) {
         break;
      }
      rmodel.AddOperator(std::move(op));
      std::string op_type = graph.node(i).op_type();
      if (op_type == "Gemm") {
         rmodel.AddBlasRoutines({"Gemm", "Gemv"});
      } else if (op_type == "Conv") {
         rmodel.AddBlasRoutines({"Gemm", "Axpy"});
      } else if (op_type == "RNN") {
         rmodel.AddBlasRoutines({"Gemm", "Axpy"});
      } else if (op_type == "Selu" || op_type == "Sigmoid") {
         rmodel.AddNeededStdLib("cmath");
      } else if (op_type == "LSTM") {
         rmodel.AddBlasRoutines({"Gemm", "Axpy"});
      } else if (op_type == "BatchNormalization") {
         rmodel.AddBlasRoutines({"Copy", "Axpy"});
      } else if (op_type == "GRU") {
         rmodel.AddBlasRoutines({"Gemm", "Axpy"});
      }
   }
```
Now we store the output names into a vector and add the output tensor to rmodel object. 
Then we finally return the rmodel object.
```C++
   std::vector<std::string> outputnames;
   for (int i=0; i < graph.output_size(); i++){
      outputnames.push_back(graph.output(i).name());
   }
   rmodel.AddOutputTensorNameList(outputnames);

   return rmodel;
}
```
### Interface of Sofie
```C++
using namespace TMVA::Experimental;
SOFIE::RModelParser_ONNX Parser;
SOFIE::RModel model = Parser.Parse(“./example_model.onnx”);
model.Generate();
model.OutputGenerated(“./example_output.hxx”);
```
The above code can be executed in the ROOT Command line for having an understanding of how SOFIE works. Executing these will generate a .hxx header file which will contain the inference code for the model.

```C++
model.PrintRequiredInputTensors();
```
The above code can be used to check the required size and type of input tensor for that particular model

```C++
model.PrintInitializedTensors();
```
The above code can be used to check the tensors (weights) already included in the model.

```C++
#include "example_output.hxx"
float input[INPUT_SIZE];
std::vector<float> out = TMVA_SOFIE_example_model::infer(input);
```
The above code can be used to use the generated inference code.

Hope so you all got a good understanding of Parse Function and operators of Sofie.
Meet you all soon with a new blog next time!

Until then, Good bye!

**Thanks and Regards,**

**Neel Shah**