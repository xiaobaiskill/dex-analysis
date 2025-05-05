分析uniswap v2
---

### eth 合约地址
* `router` : `0x7a250d5630b4cf539739df2c5dacb4c659f2488d`
* `factory`: `0x5c69bee701ef814a2b6a3edd4b1652cb9cc5aa6f`

### [核心算法](https://docs.uniswap.org/whitepaper.pdf)
* 公式
$\large X * Y = K = L^2$

`X、Y`: 分别表示X代币和Y代币
`L`: 表示 LP token 的数量

* 矢量图


### 添加池子

* 添加池子的流程
![添加支持流程](../image/add-liquidity-process.png)


* LP 的计算推导公式
    * 当添加流动性时的已知的三个条件
      * 1 添加前的liquidity token 的计算公式
$\large L_{0} = \sqrt{\small x_{0} * y_{0}}$
      
      * 当有用户添加了 $d_{x}$ & $d_{y}$ 的数量时
$\large L_{1} = \sqrt{\small(x_{0}+d_{x})(y_{0}+d_{y})}$
        
      * 添加流动性时,要求按照当前池的比例提供代币,故
$\frac{y_{0}}{x_{0}} = \frac{(y_{0} + d_{y})}{(x_{0} + d_{x})} = \frac{d_{y}}{d_{x}}$
=> $d_{x} = d_{y} * \frac{x_{0}}{y_{0}}$
=> $d_{y} = d_{x} * \frac{y_{0}}{x_{0}}$

    * 进入可以推导出如下公式(添加$d_{x}$ & $d_{y}$后 可以获得多少的LP token) 
    
    设: $d_{x}$ & $d_{y}$ 可以获取到 $\large S$ 个LP Token
        $\large T$ 为 total liquidity token


    => $\large \frac{S}{T} = \frac{L_{1} - L_{0}}{L_{0}}$

    将 $d_{y} = d_{x} \frac{y_{0}}{x_{0}}$ 带入至 $L_{1}$
    => $L_{1} = \sqrt{(x_{0}+d_{x})(y_{0}+d_{x}\frac{y_{0}}{x_{0}})}$
    => $L_{1} = \sqrt{x_{0}y_{0} + d_{x}y_{0} + d_{x}y_{0} + d_{x}^2\frac{y_{0}}{x_{0}}}$
    =>$L_{1} = \sqrt{x_{0}y_{0}(1 + 2\frac{d_{x}}{x_{0}} + \frac{d_{x}^2}{x_{0}^2})}$
    =>$L_{1} = \sqrt{x_{0}y_{0}(1+\frac{d_{x}}{x_{0}})^2}$
    =>$L_{1} = L_{0}(1 + \frac{d_{x}}{x_{0}})$

    => $S = \frac{L_{0}(1+\frac{d_{x}}{x_{0}}) - L_{0}}{L_{0}} * T$
    => $S = \frac{d_{x}}{x_{0}} * T$

    将 $d_{x} = d_{y} * \frac{x_{0}}{y_{0}}$ 带入至 S 中
    => $S = \frac{d_{y} * \frac{x_{0}}{y_{0}}}{x_0} * T$
    => $S = \frac{d_{y}}{y_{0}} *T = \frac{d_{x}}{x_{0}} * T$

    * ​新增的 LP Token 比例 = ​用户提供的代币比例
    $\frac{S}{T} = \frac{d_{y}}{y_{0}} = \frac{d_{x}}{x_{0}}$
    

* 添加池子的代码解析
  
  
### 撤销池子


### swap 