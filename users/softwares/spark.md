## Use pyspark in jupyter notebook

Firstly, please refer to the conda workflow in python section.

`pip install findspark` in your conda virtual env. 

`spack load jdk` and `spack load spark`. It is necessary to load jdk for the exsitence of `JAVA_HOME`, otherwise spark context cannot be created, see [so](https://stackoverflow.com/questions/31841509/pyspark-exception-java-gateway-process-exited-before-sending-the-driver-its-po).

And open jupyter notebook use `jupyter notebook` as usual.

```python
import findspark
findspark.init("/home/ubuntu/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.4.0/spark-2.3.0-ovs6bpfx4hfqncvdjdsoiuw2aoxbkuvb")
import pyspark
## below is an demo example
import random
sc = pyspark.SparkContext(appName="Pi") ## create a spark context
num_samples = 100000000
def inside(p):     
  x, y = random.random(), random.random()
  return x*x + y*y < 1
count = sc.parallelize(range(0, num_samples)).filter(inside).count()
pi = 4 * count / num_samples
print(pi)
```

