分析uniswap v3
---

### eth 合约地址
* `router` : `0xE592427A0AEce92De3Edee1F18E0157C05861564`
* `routerV2` : `0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45`
* `UniversalRouter` : `0x66a9893cc07d91d95644aedd05d03f95e1dba8af`
* `factory`: `0x1F98431c8aD98523631AE4a59f267346ea31F984`

### [核心算法](https://app.uniswap.org/whitepaper-v3.pdf)

#### 集中流动性计算
* 与uniswap v2 本质上一样
    $XY = K = L^2$
    $P = \frac{Y}{X}$
    => $X = $