# unrefined notes

An exploration of tinygrad. It's changing every day, so I want to catch up before it reaches 1.0, which is expected soon.

Simple Program to begin with:
```python
from tinygrad import Tensor
a = Tensor([1,2])
b = Tensor([3,4])
res = a.mul(b).sum().numpy()
print(res) # 1 * 3 + 2 * 4 = 11
```

How does tinygrad run this program and gives me the result?

Line 1: 
First, I import the Tensor class from `tinygrad/tensor.py`. It's the main class I interact with.

Line 2 & 3: 
When the Tensor([1,2]) or Tensor([3,4]) is called, it does the following internally:
- since device argument is not set in the Tensor, it uses the DEFAULT, as explained in [device notes](device.md#default).
- since dtype argument is not set, it derives it from the data i.e. int here. So, dtype would be set as dtypes.int (tinygrad type)
- finally, it converts data to LazyBuffer with PYTHON? device. dtype is truncated to its actual data type range using ctypes
  this data is then converted into its equivalent binary string using the struct module
  lazybuffer has a buffer which has an allocate method to allocate the data to device memory
  at this point, data is on PYTHON device, so copy to it actual device (METAL in this case)

```python
In [1]: a = Tensor([1,2])

In [2]: a
Out[2]: <Tensor <LB METAL (2,) int (<MetaOps.COPY: 30>, None)> on METAL with grad None>

In [3]: a.lazydata
Out[3]: <LB METAL (2,) int (<MetaOps.COPY: 30>, None)>
```
Notice the <MetaOps.COPY: 30> which means that the data is copied from PYTHON to METAL
device is METAL
shape is (2,) i.e. 1d Tensor with 2 elements
dtype is int

Line 4:
