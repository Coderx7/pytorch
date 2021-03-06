# Implemented functions

## Pytorch

### Import

```
import PyTorch
```

### Create view onto numpy arrays

```
A = numpy.random.rand(6).reshape(3,2).astype(numpy.float32)

tensorA = PyTorch.asFloatTensor(A)
```

This is a view, so, in theory, modifying the contents of the torch tensor, will also modify the contents
of the numpy tensor, as long as methods causing reallocation are not called on the torch tensor.  For 
example, calling `.uniform(0,1)` wont cause reallocation, but calling `.resize()` could.

### Convert into numpy array

```
A = PyTorch.DoubleTensor(5,3)
Anp = A.asNumpyTensor()
```

### Basic functions

```
A = PyTorch.DoubleTensor(5,3)
A.fill(1)
print('A', A)
print('nElement', DoubleTensor(3,2).nElement())
B = A.clone()
```

### Tensor types

```
A = PyTorch.DoubleTensor(5,3).fill(1)
A = PyTorch.FloatTensor(5,3).fill(1)
A = PyTorch.LongTensor(5,3).fill(1)
A = PyTorch.ByteTensor(5,3).fill(1)
```

### Views

```
D = PyTorch.ByteTensor(5,3).fill(1)
D.narrow(1,2,1).fill(0)
```

### Per-element

```
A = PyTorch.DoubleTensor(5,3).geometric(0.9)
A += 3
A *= 3
A -= 3
A /= 3   # note: for integer types, use //=

C = A + 5
C = A - 5
C = A * 5
C = A / 2   # note: for integer types, use //

B = PyTorch.DoubleTensor(5,3).geometric(0.9)
C = A + B
C = A - B
C = A.clone().cmul(B)
C = A / B   # for integer types, use /

A += B
A -= B
A.cmul(B)
A /= B   # Note: for integer types, use //=
```

### Resize

```
print('resize1d', PyTorch.DoubleTensor().resize1d(3).fill(1))
print('resize2d', PyTorch.DoubleTensor().resize2d(2, 3).fill(1))
size = PyTorch.LongStorage(2)
size[0] = 4
size[1] = 3
print('resize', PyTorch.DoubleTensor().resize(size).fill(1))
```

### Random distributions

```
PyTorch.manualSeed(123)
print(PyTorch.DoubleTensor(3,4).uniform())
print(PyTorch.DoubleTensor(3,4).normal())
print(PyTorch.DoubleTensor(3,4).cauchy())
print(PyTorch.DoubleTensor(3,4).exponential())
print(PyTorch.DoubleTensor(3,4).logNormal())

print(PyTorch.DoubleTensor(3,4).bernoulli())
print(PyTorch.DoubleTensor(3,4).geometric())
print(PyTorch.DoubleTensor(3,4).geometric())
print(PyTorch.DoubleTensor(3,4).geometric())
```

## Pynn

```
import PyTorch
from PyTorchAug import *

mlp = Sequential()
mlp.add(SpatialConvolutionMM(1, 16, 5, 5, 1, 1, 2, 2))
mlp.add(ReLU())
mlp.add(SpatialMaxPooling(3, 3, 3, 3))
mlp.add(Reshape(32 * 8 * 8))
mlp.add(Linear(32 * 8 * 8, 150))
mlp.add(Tanh())
mlp.add(Linear(150, 10))
mlp.add(LogSoftMax())

crit = ClassNLLCriterion()

input = PyTorch.FloatTensor(100, 1, 28, 28).uniform()
target = PyTorch.FloatTensor(100).fill(1)

output = input.forward(input)
loss = crit.forward(input, target)
gradOutput = crit.backward(input, target)
mlp.zeroGradParameters()
mlp.backward(input, gradOutput)
mlp.updateParameters(learningRate)
```

