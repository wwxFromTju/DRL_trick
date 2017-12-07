# DeepRLHacks-ZH

# 声明 
这是英文的原文，觉得有用，在看ironman的时候，整理了一下
[英文](https://github.com/williamFalcon/DeepRLHacks)
希望大家觉得有用的话，给原版点个star，我的仓库以后应该会整理自己的学习DRL中的trick和精力，欢迎star与watch 
[我的](https://github.com/wwxFromTju/DRL_trick)

# Tips to debug new algorithm
# 训练新强化学习的算法
### 1. 使用低维状态空间的环境来训练网络
* John 建议使用Pendulum problem来训练agent，因为这是一个二维状态空间的问题，两个维度就是角度与速度
* 因为这样容易可视化值函数，可以通过可视化的方式来观察：算法是否应该处在对的状态，算法是怎么学习的，学习的过程对不对
* 可视化可以帮助思考为什么算法不工作，比如是value函数太光滑之类的

### 2.测试算法的合理性，对于特定的算法构造对应的问题
* 比方说你的算法想要解决一个层次强化学习问题，那么让agent学习的问题应该具有明确的层次性
* 观察agent，确定agent（在不同的层次中）做了正确的事
* 不要over fit你要学习的问题，特别是这个环境的状态空间比较小的时候
### 3.使用你最熟悉的环境来测试
* 你可以明确知道在这个环境中，大概需要多久来训练agent
* 知道rewards大概会怎么变化
* 可以明确baseline大概是在哪里，比如以前训练别的算法的数据，或者自己的启发式算法
* 比方说一个直走的腿型机器人，那么它合理的走法我们是可以想象的，所以可以观察学习过程中是否是有效的，同时可以快速地发现有哪些是奇怪的行为

# Tips to debug a new task
# 调试网络的技巧
* 从最简单的开始，直到你能看到它有学习到东西
* 应用1: 简化你的特征空间
    * 比如说你在面对一个图像输入的强化学习任务，这可能比较难构造好的特征（比如HOG并不一定好），使用CNN可能使的网络比较难学，所以一种方法就是通过对task的理解，直接编码位置x，y，编码物品的种类，然后在简化的feature中学习
    * 直到在你的简化问题中，agent有效地学习，然后再切换到最初以图片作为输入（难的）问题
* 应用2：简化reward函数
    * 设计reward能够快速地让agent学习到正确的事，这样能够快速知道算法，实现是否是对的
    * 比如：在一个task中，我们希望让agent能够碰到物体。所以reward设计为碰到物体就给一个+1的reward，那么这样对于agent学习起来会比较难，因为可能需要做一些动作才会碰到物体，所以获得reward这件事其实是比较少发生的。所以一种简化的办法就是：使用与物体距离成反函数的reward来给予agent，那么agent获得的reward就比较密集了
    
# Tips to frame a problem in RL
# 强化学习的框架性技巧
有时候不好确定feature，reward应该是怎么样的，这边有几个建议
### 1. 第一步，观察随机策略在这个问题里面是什么样的表现
* 看看随机的agent对你有什么启发
* 如果随机的agent能够有时候把你希望的task完成了，那么RL应该也会学习到正确的事。（策略梯度会根据reward，调整行为，学习到相应的task）
* 如果随机的agent根本就做不到对的事情，那么RL绝对做不到
### 2. 确定给agent的输入是有效的：
* 比如你就采用agent观察的输入来做决策，如果你自己都做不到，那么RL通常也很难做大
* 比如：这是一个图片的task，你来观察处理好的输入，看看是不是把一些重要的东西处理掉了
### 3. 确定任何东西的尺度是正取的：
* 比如：
    * observations： 均值为0，方差为1.
    * reward： 如果可能控制的话，缩放到相应的尺度
    * 通过你能得到的数据来处理
* 观察你所有的reward与observation，确定在尺度上没有特别异常
### 4. 为学习的问题设置baseline
* 因为不确定算法大概会工作的怎么样，所以通过已有的算法来做baseline比较
    * cross entropy method
    * policy gradient method
    * Q-learning

# Reproducing papers
# 重现算法
我们经常需要去重现算法，但是发现重现算法很难，这里有一些技巧可以考虑：
### 比需要的数据，使用更多的采样 
### policy是对的，但是没有那么明确
* 让agent学的更久
* 微调超参数，来得到更好的性能
* 可以考虑更大的batch size
    * batch size比较小，那么噪声会有比较大的影响
    * 比如在TRPO算法中，如果使用比较小的batch size，那么最终用了100k time step
    * 比如在DQN中，我们用了10k time step，1mm frame的replay buffer

# Guidelines on-going training process
# 训练过程的指导
确保你的训练过程是正确的
### 1. 观察每个参数的敏感性
* 如果算法非常敏感，那么这代表这个算法不够robust，这样并不好
* 我们希望算法是通用，所以有时候出现的一些有趣的行为并不是通常的，只是因为环境的动态性
### 2. 确定在优化过程中，用来观察的indicator是正常的
* 变化
* 确定value function是否是正确的：
    * 是否预测准确
    * 是否预测的return准确
    * 更新的大小有多大
* 一些用在DL中的技巧
### 3. 有一个系统的环境作为测试
* 需要有纪律（坚持做）
* 观察在所有环境中，你训练的效果：
    * 有时候只在特定的环境中有效，但是在其他的环境中无效
    * 对于一个问题，容易达成over fit
* 偶尔运行一下benchmarks，来看看当前的效果
### 4. 有时候你会认为你的算法有效，但是实际上只是随机的噪声
* 比如：在7个任务中测试3种算法，有一种算法在所有任务中都运行的很好。但是实际上，通过调整random seed，这3种算法实际上是一样的
### 5. 尝试不同的随机数种子
* 运行多次，并取平均
* 在多个任务上，分别运行多个随机数种子，如果不这么做的话，很有可能只是在一个任务上over fit
### 6. 有些增加其实是不需要的
* 很多技巧本质上就是对某些东西做normalizing，或者只是帮助优化
* 一系列的技巧，有些技巧的目的或者影响是一样的，所以可以移去重复的，让你的算法简化（这是一个key）
### 7. 让你的算法简化
能够更泛化
### 8. 自动化你的经验
* 不要花一整天的时间来观察代码的输出
* 将实验放在云服务上（服务器上？），然后分析结果
* 框架性地来追踪经验与结果：
    * 比如IPython 
    * 存在数据库

# General training strategies
# 通用的训练技巧
### 1. 白化和标准化数据（在开始前，处理所有数据）
* observation
    * 计算mean，standard deviation。然后z-transform
    * 对于所有的数据，而不是最近的数据
    * At least it'll scale down over time how fast it's changing.
    * 如果学习的过程中不断目标，那么这样会影响优化
    * 如果只是用最近的数据的mean，那么相当于每段时间的数据都在变，那么优化效果会很差（通常） 
* reward
    * 缩放，不要移动
    * 影响 agent’s will to live
    * 将会改变问题
    
* standardize targets
    * 比如通过reward来影响
* PCA 白化？
    * could help
    * 尝试一下，看看是不是对NN有效
    * 大尺度的缩放，从（-1000，1000）到（-0.001，0.001）会使的训练很慢
    
### 2. discount factors传达到信息
* 确定的给予了多长的信度分配
* 比如：factor设置为0.99，那么相当于往后面看了100个steps，这意味着你比较注重近期利益
    * 观察相关的真实时间是一个比较有效的选择
    * 直觉的，RL通长使用离散的时间表示
    * 比如：在100step中，实际上是3秒的时间，那么你就可以观察这3秒发生了什么

### 3. 通过离散的方法来解决问题
* 比如在DQN中，做的frame skip
    * 那么作为一个人类，在这种情况下，你可以控制吗？
    * 看看随机探索会发生什么
    * Discretization determines how far your browning motion goes.
    * 如果需要做一系列的动作，那么应该更偏向探索
    * 针对相应的问题，选择不同的离散尺度来工作

### 4. 仔细地看episode的return
* 不只是mean，同时也要看min，max
    * max就是你的策略能够做的更好
    * 同时确定你的policy是否是做正确的事
* 查看episode的长度
    * 输赢只能告诉你最终的信息，但是长度的变化，能够体现输赢的速率（比如一些坚持类的游戏）
    * 有时候长度的改变就代表的学习到了  

# Policy gradient diagnostics
# 策略梯度诊断
### 1. 看entropy
* 看action的entropy：状态的entropy有时候更有用，但是很难去计算
* 如果下降很快，那么policy会很快变成确定性的，然后不会去探索
* 如果不下降，那么policy就一直是随机的
* 可以通过一些手段固定entropy：
    * KL penalty
    * Keep entropy from decreasing too quickly
    * Add entropy bonus
* 如何测量entropy
    * 对于大部分policy可以通过计算，来计算entropy
    * 如果是Gaussian，那么可以计算differential entropy  
### 2. 看KL divergence
* 看KL divergence的更新大小
* 举例：
    * 如果KL是小于等于0.01，那么是小的
    * 如果KL是大于等于10，那么是大的
### 3. 用baseline来解释方差
* 来看值函数是否是好的预测
    * 如果是负数，那么就可能是过拟合了
    * 那么就要去调整超参
### 4. 初始化策略  
* 非常重要（比监督学习更重要）
* Zero or tiny final layer to maximize entropy
    * 在开始的时候，最大随机探索 

# Q-learning Strategies
### 1. 注意你的replay buffer memory的使用情况
* 你可能需要非常大的replay buffer， 需要对此调整相应的code
### 2. 调整不同的学习率的schedule
### 3. 可能需要一段学习来收敛，或者开始学习
* 需要有耐心，比如DQN就学的很慢

# Other
# 其他
一个好的特征应该能够区分出两个frame之间的区别


