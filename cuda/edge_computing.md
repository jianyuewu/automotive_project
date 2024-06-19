# Edge computing

Embeded system is suitable for edge computing, because:  
1. Latency is smaller than central computing.  
2. Many cars and devices are used, so need huge central computing resource.  
3. Embeded system SoCs are more and more powerful recently.  

TensorFlow Lite, Caffe2, and NCNN is open source computing framework.  
MobileNet is suitable for light weight embeded system training. Based on depthwise separable convolution, which has depthwise convolution and pointwise convolution.   
Mace can use OpenCL to speedup via GPU.  
Forward propagation is calculated from the input end, the goal is to obtain the output result of the network, and then compare the output result with the target value to obtain the result error. Then, the result error is backpropagated along the network structure and disassembled to each node to obtain the error of each node, and then the weight of the node is adjusted according to the error of each node. The purpose of Forward Propagation is to get the output, and the purpose of Backpropagation is to adjust the weights by the backpropagation error. By alternating between the two, it is hoped that the output will be consistent with the target value.  
Optimize:  
1. Accurate train data.  
2. Better algorithm.  
3. Train method: i.e. loss, dropout structure, gradient descent algorithm.  


https://github.com/Tencent/ncnn
https://github.com/XiaoMi/mace
https://arxiv.org/abs/1704.04861
https://gitcode.com/Zehaos/MobileNet/blob/master/nets/mobilenet.py?utm_source=csdn_github_accelerator&isLogin=1
