[根目录](../../CLAUDE.md) > **strategies**

# Strategies - 投资策略模块

> 模块类型：投资策略实现
> 更新时间：2025-12-18 10:30:05
> 文档覆盖率：100%

## 模块职责

该模块实现了 5 种核心的投资组合优化策略，每种策略都采用不同的数学方法来优化投资组合的权重分配，以适应不同风险偏好的投资者需求。

## 策略文件清单

| 策略名称 | 文件路径 | 核心方法 | 数学基础 | 适用场景 |
|---------|---------|---------|---------|---------|
| **特征投资组合** | `eigen_portfolio_strategy.py` | `generate_portfolio` | 主成分分析（PCA） | 市场中性策略 |
| **遗传算法** | `genetic_algo_strategy.py` | `generate_portfolio` | 进化算法 | 全局优化 |
| **最大夏普比率** | `maximum_sharpe_ratio_strategy.py` | `generate_portfolio` | 二次优化 | 风险调整收益最大化 |
| **最小方差** | `minimum_variance_strategy.py` | `generate_portfolio` | 凸优化 | 风险最小化 |
| **辅助函数** | `strategy_helper_functions.py` | `denoise_covariance` | 随机矩阵理论 | 协方差矩阵降噪 |

## 策略详解

### 1. 特征投资组合策略 (Eigen Portfolio Strategy)

#### 核心思想
基于主成分分析（PCA）将股票收益分解为正交的特征向量，每个特征向量代表一个独立的投资组合。

#### 实现细节
```python
def generate_portfolio(self, symbols, return_matrix, covariance_matrix):
    # 获取特征值和特征向量
    eigenvalues, eigenvectors = np.linalg.eig(covariance_matrix)

    # 选择第n个特征向量作为权重
    weights = eigenvectors[:, self.eigen_portfolio_number-1]

    # 处理负权重
    weights = self.remove_negative_weights(weights, covariance_matrix)
```

#### 特点
- 第一个特征组合：代表市场组合，与指数高度相关
- 第二个特征组合：与市场正交，风险收益最高
- 后续特征组合：风险收益逐级降低

### 2. 遗传算法策略 (Genetic Algorithm Strategy)

#### 参数配置
- **初始基因数**：100
- **迭代次数**：50
- **选择数量**：25
- **变异迭代**：50
- **交叉概率**：0.05
- **权重更新因子**：0.1

#### 算法流程
1. **初始化**：随机生成100个基因（权重组合）
2. **选择**：根据夏普比率选择前25个最优基因
3. **变异**：对选中基因进行随机变异
4. **交叉**：以5%概率进行基因交叉
5. **迭代**：重复50轮进化过程

#### 适应度函数
```python
def fitness_score(self, returns):
    sharpe_returns = np.mean(returns) / np.std(returns)
    return sharpe_returns
```

### 3. 最大夏普比率策略 (Maximum Sharpe Ratio Strategy)

#### 数学原理
基于二次规划求解最大夏普比率问题：
```
max w'μ / sqrt(w'Σw)
s.t. w'1 = 1
```

#### 实现方法
使用矩阵运算直接求解：
```python
def generate_portfolio(self, symbols, covariance_matrix, returns_vector):
    inverse_cov_matrix = np.linalg.pinv(covariance_matrix)
    ones = np.ones(len(inverse_cov_matrix))

    numerator = np.dot(inverse_cov_matrix, returns_vector)
    denominator = np.dot(np.dot(ones.transpose(), inverse_cov_matrix), returns_vector)
    msr_portfolio_weights = numerator / denominator
```

### 4. 最小方差策略 (Minimum Variance Strategy)

#### 优化目标
```
min w'Σw
s.t. w'1 = 1
```

#### 实现方法
```python
def generate_portfolio(self, symbols, covariance_matrix):
    inverse_cov_matrix = np.linalg.pinv(covariance_matrix)
    ones = np.ones(len(inverse_cov_matrix))
    inverse_dot_ones = np.dot(inverse_cov_matrix, ones)
    min_var_weights = inverse_dot_ones / np.dot(inverse_dot_ones, ones)
```

### 5. 策略辅助函数 (Strategy Helper Functions)

#### 主要功能
- **协方差矩阵降噪**：使用随机矩阵理论过滤噪声
- **负权重处理**：转换为只做多策略
- **数据标准化**：确保数据一致性

#### RMT 降噪方法
```python
def denoise_covariance(covariance_matrix):
    # 计算理论最大特征值
    q = return_matrix.shape[0] / return_matrix.shape[1]
    theoretical_max_eigenvalue = (1 + np.sqrt(1/q))**2

    # 过滤超过理论值的特征值
    filtered_eigenvalues = np.minimum(eigenvalues, theoretical_max_eigenvalue)
```

## 使用示例

### 单独使用策略
```python
from strategies.genetic_algo_strategy import GeneticAlgoStrategy

# 创建策略实例
strategy = GeneticAlgoStrategy()

# 生成投资组合
weights = strategy.generate_portfolio(symbols, return_matrix)
```

### 在策略管理器中使用
```python
from strategy_manager import StrategyManager

manager = StrategyManager()
portfolios = manager.generate_all_portfolios(symbols, returns, covariance)
```

## 性能对比

| 策略 | 计算复杂度 | 稳定性 | 收益潜力 | 风险水平 |
|------|-----------|--------|---------|---------|
| Eigen Portfolio | O(n³) | 中等 | 高 | 高 |
| Genetic Algorithm | O(g×n²) | 高 | 高 | 中等 |
| Maximum Sharpe | O(n³) | 中等 | 高 | 中等 |
| Minimum Variance | O(n³) | 高 | 低 | 低 |

## 优化建议

### 性能优化
1. **缓存计算**：保存特征值分解结果
2. **并行计算**：遗传算法可并行评估
3. **增量更新**：协方差矩阵增量计算

### 策略改进
1. **动态参数**：根据市场条件调整策略参数
2. **因子模型**：加入行业、风格因子
3. **约束条件**：添加换手率、集中度约束

## 常见问题

### Q: 如何选择特征投资组合编号？
A:
- 1号：市场组合，适合跟踪指数
- 2号：最高风险收益，适合激进投资者
- 3-5号：平衡型，适合中等风险偏好

### Q: 遗传算法为什么需要变异？
A: 变异引入新的基因组合，避免算法陷入局部最优，提高全局搜索能力。

### Q: RMT降噪的作用是什么？
A: 过滤协方差矩阵中的随机噪声，提取真实的市场结构信息，提高投资组合的稳定性。

## 变更记录

### 2025-12-18 10:30:05
- ✨ 完成所有5个策略的深度分析
- 📊 详细解析了遗传算法的参数和实现
- 🔍 解释了每种策略的数学原理
- 📝 添加了性能对比和优化建议
- 🎯 提供了实际使用示例

---

*返回[根目录](../../CLAUDE.md)查看项目整体文档。*