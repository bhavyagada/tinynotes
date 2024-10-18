# [device.py](https://github.com/tinygrad/tinygrad/blob/master/tinygrad/device.py)

## Device class
### init

It creates a device list by using the `runtime/ops_*` file names.
```python
_devices = ['DSP', 'LLVM', 'PYTHON', 'CUDA', 'CLOUD', 'NV', 'QCOM', 'CLANG', 'METAL', 'NPY', 'AMD', 'DISK', 'HIP', 'GPU']
```

### canonicalize

A method to standardize device strings:
'metal' => 'METAL',
'cuda:0' => 'CUDA',
'gpu:1' => 'GPU:1', etc.

If device is None, it will return `device.DEFAULT`

### DEFAULT

It checks if the device is set using an environment variable and returns that. Otherwise, it tries to find an available device using the following method:
```python
def get_available_devices(self) -> Iterator[str]:
    for device in ["METAL", "AMD", "NV", "CUDA", "QCOM", "GPU", "CLANG", "LLVM"]:
      with contextlib.suppress(Exception): yield self[device].dname
```

The device class defines `[__getitem__](https://docs.python.org/3/reference/datamodel.html#object.__getitem__)`, which runs when `self[device]` is called.

`__getitem__` runs an internal function, which can be best explained using an example from my METAL (Mac) device:

```python
In [1]: from tinygrad.device import _Device
In [2]: import importlib, multiprocessing, inspect

# initialize the Device class
In [3]: device = _Device()

 # when this is run...
In [4]: device.DEFAULT
Out[4]: 'METAL'

# the following happens internally
# DEFAULT is a property funtion which calls all devices in this manner, 
# but only 'METAL' will return without an Error
In [5]: device["METAL"] # call __getitem__ here
Out[5]: <tinygrad.runtime.ops_metal.MetalDevice at 0x105429b80>

# how did tinygrad know I was running it on 'METAL' device?

# continue if it's the MainProcess
In [6]: multiprocessing.current_process().name
Out[6]: 'MainProcess'

In [7]: x = "METAL"; _devices = ['DSP', 'LLVM', 'PYTHON', 'CUDA', 'CLOUD', 'NV', 'QCOM', 'CLANG', 'METAL', 'NPY', 'AMD', 'DISK', 'HIP', 'GPU']

# dynamically import 'tinygrad.runtime.ops_metal' and get the class which matches the name 'Metal'
# and check if the imported file has {device}Device class
In [8]: ret = [cls for cname, cls in inspect.getmembers(importlib.import_module(f'tinygrad.runtime.ops_{x.lower()}')) if (cname.lower() == x.lower() + "device") and x in _devices][0]
Out[8]: tinygrad.runtime.ops_metal.MetalDevice

# found the main class MetalDevice in `runtime/ops_metal.py` which can now be set as the default by initializing it
In [9]: ret = ret("METAL")

# this is device.DEFAULT
In [10]: ret.dname
Out[10]: 'METAL'
```
