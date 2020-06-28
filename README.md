# PARL_box2d_bipedalwalkerhardcore-v2-based-on-SAC
使用parl框架解决gym环境中的box2d的bipedalwalkerhardcore-v2游戏，应用强化学习方法sac  
# 环境
gym环境中的box2d，包含多个游戏，本例采用的是bipedalwalkerhardcore-v2游戏，游戏通过操作一个二足机器人跨越多种随机生成的障碍。  
主要包含三种障碍：  
  小方块/大方块：跨越  
  长方形台阶（上/下）：攀爬  
  掉落陷阱：跨越  
二足机器人包含4个维度的actions，为连续取值  
与环境交互的reward为-1~1之间，评估的是机器人的平衡状态，在未通关时接触地面，会得到-100的惩罚分  
通关条件：一个episode总分超过300  
该环境随机生成障碍路径，对模型的稳健性要求较高  
# SAC模型
soft actor critic模型，是DDPG的升级版，软更新actor和cirtic网络。在随机变化的环境中效果比较好。一般应用于连续动作的强化学习任务，在机器人训练中使用广泛。  
parl的cpu版本算法库中包含了sac模型，所以本例舍弃了DDPG模型改用SAC模型  
# 调参方法
参考sota榜成功通关的调参方法：  
memory size： 2e6，增加数据的利用率  
action repeat： 3每次交互时重复动作的次数，可以得到比较稳定的值  
reward scale： 5reward的缩放倍数，因为reward基本都在0.x，增加不同交互的得分差异，能够尽快收敛  
down：0当每次游戏未成功机器人摔倒时会给出一个-100分的惩罚，与普通reward的差异较大（尤其经过scale后），在训练时将其置0，防止对模型影响过大  
未使用(效果不佳)的trick技巧  
改变随机环境生成的比例： 根据模型训练结果发现对大小方块的跨越策略最不稳健，有论文中改变三种障碍（方块、台阶、陷阱）的生成比例为：3:1:1来增加方块跨越策略的稳健性  
加入action选取噪声，随机加入一个0-0.3的action噪声，增加探索的效率  
# 训练过程
分了三次训练：  
先简单训练了2小时，使用parl示例参数，发现不收敛，但会习得一些站立的策略  
调整参数，此时memory size为1e6，actor、critic网络的学习率均为1e-3，训练24小时，1e6 steps，习得跨越陷阱、台阶策略，但方块策略不稳健（掉落一般是由于方块未成功跨越），此时已可以完成通关条件  
将down后的-100reward改为-0.5，增加memory size为2e6，actor、critic学习率为1e-4，在前模型基础上继续训练，可以得到较为稳定的策略  
# 最终训练效果
受时间限制，最终提交模型训练共约4000episode，100episode的test达到150分左右的平均分，30%的通关几率，与屠榜模型还存在很大差距。因为模型训练时长比较长，之后想再尝试一下更改模型结构，或者其他的调参技巧看可不可以得到比较好的效果  
通关gif：  
![image](https://github.com/vivichloe/PARL_box2d_bipedalwalkerhardcore-v2-based-on-SAC/raw/master/BipedalWalkerHardcore_result.gif)  
aistudio版本的nootbook(不包含图形化显示和gif生成)：  
https://aistudio.baidu.com/aistudio/projectdetail/586297  
