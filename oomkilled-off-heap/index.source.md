
This month I have encountered at 2 of our services CI build failure due to Java Unit tests.
Same tests passed locally.  

With no error in logs, the tests got OOMKilled, even though the Java heap size limit was ok relative to the Docker 
pod resources.  
No core/heap dump file generated. 

OOMKilled error message had a line mentioning netty c lib.  
On both cases, cause was from Google Cloud Apis and OpenTelemetry which having gRpc calls which 
[uses Netty](https://github.com/grpc/grpc-java?tab=readme-ov-file#transport) under the hood.  
When disabling the connections on tests, issue was resolved. 

The connections should not have been enabled on tests, but still it was not clear why it got OOMKilled, even though 
the Java heap size limit was ok relative to the Docker container.

Then I found [this](https://piotrd.hashnode.dev/containerized-jvm-oomkilled-problem):
> Netty uses ByteBuffers and Direct Memory to allocate and deallocate memory

### Solution

gRpc is widely used for libraries like Google Cloud clients, OpenTelemetry and more.  
As io.grpc uses Netty, a similar phenomena can happen at a micro-service, some adjustments can be made.  
There are multiple approaches on how to address it.

Commonly, Java heap size is configured to be a bit less than the pod memory limit.  
In this case, the margin between Java heap size and pod memory limit may not be enough, as the off-heap Direct Memory
may exceed the pod memory limit, resulting with OOMKilled by Kubernetes.

A more controlled approach, is to configure 
[Direct Memory limit](https://stackoverflow.com/questions/67068818/oom-killed-jvm-with-320-x-16mb-netty-directbytebuffer-objects) 
as well, and configure pod memory limit to be the summary of max heap and max off-heap (Direct Memory) + some margin 
for other memory like Meta space, classes and more.
