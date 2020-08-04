
# Fleet

**Fleet** is High-Level API for distributed training in PaddlePaddle. The name of **Fleet** means that a large crowd of ships working together to finish a large scale job. The design of Fleet makes a trade-off between easy-to-use and algorithmic extensibility and is highly efficient. First, a user can shift from local machine paddlepaddle code to distributed code  **within ten lines of code**. Second, different algorithms can be easily defined through **distributed strategy**  through Fleet API. Finally, distributed training is **extremely fast** with Fleet and just enjoy it.

**Note: all the examples here should be replicated from develop branch of Paddle**

## Installation of Fleet-Lightning
To show how to setup distributed training with fleet, we introduce a small library call **fleet-lightning**. **fleet-lightning** helps industrial users to directly train a specific standard model such as Resnet50 without learning to write a Paddle Model. 

``` bash
pip install fleet-lightning
```

## A Distributed Resnet50 Training Example

``` python
import os
import fleet_lightning as lightning
import paddle.fluid as fluid
from paddle.fluid.incubate.fleet.collective import fleet, DistributedStrategy
import paddle.fluid.incubate.fleet.base.role_maker as role_maker

configs = lightning.parse_train_configs()

role = role_maker.PaddleCloudRoleMaker(is_collective=True)
fleet.init(role)

model = lightning.applications.Resnet50()

loader = model.load_imagenet_from_file("/pathto/imagenet/train.txt")

optimizer = fluid.optimizer.Momentum(learning_rate=configs.lr, momentum=configs.momentum)
optimizer = fleet.distributed_optimizer(optimizer)
optimizer.minimize(model.loss)

place = fluid.CUDAPlace(int(os.environ.get('FLAGS_selected_gpus', 0)))
exe = fluid.Executor(place)
exe.run(fluid.default_startup_program())

epoch = 30
for i in range(epoch):
    for data in loader():
        cost_val = exe.run(fleet.main_program, feed=data, fetch_list=[model.loss.name])
    
```
