## WALL-E ([report](../master/walle_report.pdf))
* Major Contributors: Tianbing Xu (Baidu Research, CA), initiates the project, writes the most of code.
* Collaborators: Liang Zhao (Baidu Research, CA), Andrew Zhang (Stanford University), Shunan Zhang (Apple).
* An Efficient, Fast, yet Simple Reinforcement Learning Research Framework codebase with potential applications in Robotics and beyond.

## Motivations:
This is a long term Reinfocement Learning project focused on developing an efficient, yet simple RL framework to support
the ongoing RL research related to systems, methodologies, et cetera.
The first completed milestone is speding up RL with multi-process architectural support. In RL, the time to collect experience by running a policy on the environment MDP is a bottleneck, taking much more time compared to the computations of policy learning on GPU. With the multi-process support, we are able to collect experience in parallel and thus reduce the data collection time by a *near linear* factor.

## Reinforcement Learning Architecture Design
### General Reinforcement Learning Framework
* Agent is responsible for updating policy given experience generated by Sampler.
* Sampler is responsible for generating experience from updated policy by executing on environment MDP.
![rlframework](https://user-images.githubusercontent.com/3246048/45186700-3fbf6300-b1e3-11e8-83ec-19649fc29e32.png)
### RL Framework with Multi-Process Experience Collection
* Agent processor is running asynchronously and updates policy based on experience from Experience Queue 
when ready, sending policy parameters to Policy Queue.
* There are N Sampler Processors running in parallel. Each Sampler Processor generates experience based on the updated
policy read from the primed Policy Queue and sends experience to the Experience Queue.
![multi_proc_rl](https://user-images.githubusercontent.com/3246048/45186745-641b3f80-b1e3-11e8-85d6-a2d73ac6abbc.png)

## Dependencies

* Python 3.6
* The Usual Suspects: NumPy, matplotlib, scipy
* TensorFlow
* gym - [installation instructions](https://gym.openai.com/docs)
* [MuJoCo](http://www.mujoco.org/) (30-day trial available and free to students)
* Pickle

Refer to requirements.txt for more details.

If you are using `conda`
```
conda env create -f conda_walle.yml --prefix=`which conda`/../../envs/walle
source activate walle
```

### Running Command

#### Single Process, using one CPU to collect experience
```
cd ./src
CUDA_VISIBLE_DEVICES=0
python main.py HalfCheetah-v2 -it 1000 -b 10000
```

#### Multi-Process, Parallel Sampler (10 CPU Processes to collect experience)
```
cd ./src
CUDA_VISIBLE_DEVICES=0
python run_parallel_main.py HalfCheetah-v2 -it 1000 -b 1000 -n 10
```

#### Plot Curve
```
cd ./src/experiment
python plotcurve.py -x xvariable -i /path-to-log/ -o fig.png
python plotcurve_cmp.py -x xvariable -i /path-to-log/ -b /path-to-baseline-log/ -o fig.png
```

#### Results on Mujoco Tasks 
![HalfCheetah](https://user-images.githubusercontent.com/3246048/45062531-322ca080-b05e-11e8-9d97-fd29e04bb690.png)

* Running Time for collecting 20K samples per iteration.
![cmp-cheetah-time](https://user-images.githubusercontent.com/22249000/45120794-09ff7900-b114-11e8-8d5d-42c60321d8fe.png)

* Speedup for collecting 20K samples per iteration. The running time is not very accurate as it is fast, 
and there is variance from the asynchronously nature and Queue I/O. The basic conclusion is that experience
collection speedup w.r.t. CPU numbers is near linear (while not over-linear).
![cmp-cheetah-speedup](https://user-images.githubusercontent.com/22249000/45120823-1f74a300-b114-11e8-9035-913a872a45fb.png)

* As we can see, the average policy learning time per iteration is about 0.04 min and keeps almost 
the same for different number of processors.

![learn-time](https://user-images.githubusercontent.com/22249000/45246002-be83d100-b2b3-11e8-8fc3-4bc63cd38d3a.png)

* With the increasing number of processors to collect experience, the experience collection time is *near-linearly* reduced. 
Experience collection time is no longer the bottleneck. Instead, the policy learning time takes more and more perecentages and becomes the bottleneck of total running time. 

![time-percentage](https://user-images.githubusercontent.com/22249000/45247066-ccd4eb80-b2b9-11e8-98f5-f60e78ba3767.png)

## Reference
* Danijar Hafner, James Davidson, Vincent Vanhoucke, "TensorFlow Agents: Efficient Batched Reinforcement Learning in TensorFlow"
* Kevin Frans, Danijar Hafner, "Speeding Up TRPO Through Parallelization and Parameter Adaptation"
* Hao Liu, Yihao Feng, Yi Mao, Dengyong Zhou, Jian Peng, Qiang Liu,
"Action-depedent Control Variates for Policy Optimization via Stein's Identity"
