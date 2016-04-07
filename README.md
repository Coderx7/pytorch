# pytorch
Wrappers to use torch and lua from python

# What is pytorch?

- create torch tensors, call operations on them
- instantiate `nn` network modules, train them, make predictions
- create your own lua class, call methods on that

## Create torch tensors

```
import PyTorch
a = PyTorch.FloatTensor(2,3).uniform()
a += 3
print('a', a)
print('a.sum()', a.sum())
```

## Instantiate nn network modules

```
import PyTorch
from PyTorchAug import nn

net = nn.Sequential()
net.add(nn.SpatialConvolutionMM(1, 16, 5, 5, 1, 1, 2, 2))
net.add(nn.ReLU())
net.add(nn.SpatialMaxPooling(3, 3, 3, 3))

net.add(nn.SpatialConvolutionMM(16, 32, 3, 3, 1, 1, 1, 1))
net.add(nn.ReLU())
net.add(nn.SpatialMaxPooling(2, 2, 2, 2))

net.add(nn.Reshape(32 * 4 * 4))
net.add(nn.Linear(32 * 4 * 4, 150))
net.add(nn.Tanh())
net.add(nn.Linear(150, 10))
net.add(nn.LogSoftMax())
net.float()

crit = nn.ClassNLLCriterion()
crit.float()

net.zeroGradParameters()
input = PyTorch.FloatTensor(5, 1, 28, 28).uniform()
labels = PyTorch.ByteTensor(5).geometric(0.9).icmin(10)
output = net.forward(input)
loss = crit.forward(output, labels)
gradOutput = crit.backward(output, labels)
gradInput = net.backward(input, gradOutput)
net.updateParameters(0.02)
```

# Write your own lua class, call methods on it

Example lua class:
```
require 'torch'
require 'nn'

local TorchModel = torch.class('TorchModel')

function TorchModel:__init(backend, imageSize, numClasses)
  self:buildModel(backend, imageSize, numClasses)
  self.imageSize = imageSize
  self.numClasses = numClasses
  self.backend = backend
end

function TorchModel:buildModel(backend, imageSize, numClasses)
  self.net = nn.Sequential()
  local net = self.net

  net:add(nn.SpatialConvolutionMM(1, 16, 5, 5, 1, 1, 2, 2))
  net:add(nn.ReLU())
  net:add(nn.SpatialMaxPooling(3, 3, 3, 3))
  net:add(nn.SpatialConvolutionMM(16, 32, 3, 3, 1, 1, 1, 1))
  net:add(nn.ReLU())
  net:add(nn.SpatialMaxPooling(2, 2, 2, 2))
  net:add(nn.Reshape(32 * 4 * 4))
  net:add(nn.Linear(32 * 4 * 4, 150))
  net:add(nn.Tanh())
  net:add(nn.Linear(150, numClasses))
  net:add(nn.LogSoftMax())

  self.crit = nn.ClassNLLCriterion()

  self.net:float()
  self.crit:float()
end

function TorchModel:trainBatch(learningRate, input, labels)
  self.net:zeroGradParameters()
  local output = self.net:forward(input)
  local _, prediction = output:max(2)
  local numRight = labels:int():eq(prediction:int()):sum()
  local loss = self.crit:forward(output, labels)
  local gradOutput = self.crit:backward(output, labels)
  self.net:backward(input, gradOutput)
  self.net:updateParameters(learningRate)
  return {loss=loss, numRight=numRight}  -- you can return a table, it will become a python dictionary
end

function TorchModel:predict(input)
  local output = self.net:forward(input)
  local _, prediction = output:max(2)
  return prediction:byte()
end
```

Example of python script that calls this.  Assume the lua class is stored in file "torch_model.lua"
```
import PyTorch
import PyTorchHelpers
import numpy as np
from mnist import MNIST

batchSize = 32
numEpochs = 2
learningRate = 0.02

TorchModel = PyTorchHelpers.load_lua_class('torch_model.lua', 'TorchModel')
torchModel = TorchModel(backend, 28, 10)

mndata = MNIST('../../data/mnist')
imagesList, labelsList = mndata.load_training()
labels = np.array(labelsList, dtype=np.uint8)
images = np.array(imagesList, dtype=np.float32)
labels += 1  # since torch/lua labels are 1-based
N = labels.shape[0]

numBatches = N // batchSize
for epoch in range(numEpochs):
  epochLoss = 0
  epochNumRight = 0
  for b in range(numBatches):
    res = torchModel.trainBatch(
      learningRate,
      images[b * batchSize:(b+1) * batchSize],
      labels[b * batchSize:(b+1) * batchSize])
    numRight = res['numRight']
    epochNumRight += numRight
  print('epoch ' + str(epoch) + ' accuracy: ' + str(epochNumRight * 100.0 / N) + '%')
```

It's easy to modify the lua script to use CUDA, or OpenCL.

# Recent news

17 March:
* ctrl-c works now (tested on linux)

16 March:
* uses luajit on linux now (mac os x continues to use lua)

6 March:
* all classes should be usable from `nn` now, without needing to explicitly register inside `pytorch`
  * you need to upgrade to `v3.0.0` to enable this, which is a breaking change, since the `nn` classes are now in `PyTorchAug.nn`, instead of directly
in `PyTorchAug`

5 March:
* added `PyTorchHelpers.load_lua_class(lua_filename, lua_classname)` to easily import a lua class from a lua file
* can pass parameters to lua class constructors, from python
* can pass tables to lua functions, from python (pass in as python dictionaries, become lua tables)
* can return tables from lua functions, to python (returned as python dictionaries)

2 March:
* removed requirements on Cython, Jinja2 for installation

28th Februrary:
* builds ok on Mac OS X now :-)  See https://travis-ci.org/hughperkins/pytorch/builds/112292866

26th February:
* modified `/` to be the div operation for float and double tensors, and `//` for int-type tensors, such as
byte, long, int
* since the div change is incompatible with 1.0.0 div operators, jumping radically from `1.0.0` to `2.0.0-SNAPSHOT` ...
* added dependency on `numpy`
* added `.asNumpyTensor()` to convert a torch tensor to a numpy tensor

24th February:
* added support for passing strings to methods
* added `require`
* created prototype for importing your own classes, and calling methods on those
* works with Python 3 now :-)

[Older changes](doc/oldchanges.md)

