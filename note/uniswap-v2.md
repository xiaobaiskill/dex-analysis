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
    *  当添加流动性时的已知的三个条件
        *  添加前的liquidity token 的计算公式

            $\large L_{0} = \sqrt{\small x_{0} * y_{0}}$
        
        * 当有用户添加了 $d_{x}$ & $d_{y}$ 的数量时

            $\large L_{1} = \sqrt{\small(x_{0}+d_{x})(y_{0}+d_{y})}$
            
        * 按照流动性份额等比例原则,添加流动性时,要求按照当前池的比例提供代币,故

            $\frac{y_{0}}{x_{0}} = \frac{(y_{0} + d_{y})}{(x_{0} + d_{x})} = \frac{d_{y}}{d_{x}}$

            => $d_{x} = d_{y} * \frac{x_{0}}{y_{0}}$

            => $d_{y} = d_{x} * \frac{y_{0}}{x_{0}}$

    * 进入可以推导出如下公式(添加$d_{x}$ & $d_{y}$后 可以获得多少的LP token) 
        
        设: $d_{x}$ & $d_{y}$ 可以获取到 $\large S$ 个LP Token
            $\large T$ 为 total liquidity token (加池子之前)

        => $\large \frac{S}{T} = \frac{L_{1} - L_{0}}{L_{0}}$

        将 $d_{y} = d_{x} \frac{y_{0}}{x_{0}}$ 带入至 $L_{1}$

        => $L_{1} = \sqrt{(x_{0}+d_{x})(y_{0}+d_{x}\frac{y_{0}}{x_{0}})}$

        => $L_{1} = \sqrt{x_{0}y_{0} + d_{x}y_{0} + d_{x}y_{0} + d_{x}^2\frac{y_{0}}{x_{0}}}$

        => $L_{1} = \sqrt{x_{0}y_{0}(1 + 2\frac{d_{x}}{x_{0}} + \frac{d_{x}^2}{x_{0}^2})}$

        => $L_{1} = \sqrt{x_{0}y_{0}(1+\frac{d_{x}}{x_{0}})^2}$

        => $L_{1} = L_{0}(1 + \frac{d_{x}}{x_{0}})$

        => $S = \frac{L_{0}(1+\frac{d_{x}}{x_{0}}) - L_{0}}{L_{0}} * T$

        => $S = \frac{d_{x}}{x_{0}} * T$

        将 $d_{x} = d_{y} * \frac{x_{0}}{y_{0}}$ 带入至 S 中

        => $S = \frac{d_{y} * \frac{x_{0}}{y_{0}}}{x_0} * T$
        
        => $S = \frac{d_{y}}{y_{0}} *T = \frac{d_{x}}{x_{0}} * T$

    *  结论: ​新增的 LP Token 比例 = ​用户提供的代币比例
    
        $\frac{S}{T} = \frac{d_{y}}{y_{0}} = \frac{d_{x}}{x_{0}}$
    

* 添加池子的代码解析
    * `factory`
        ```
         // this low-level function should be called from a contract which performs important safety checks
            function mint(address to) external lock returns (uint liquidity) {
                (uint112 _reserve0, uint112 _reserve1,) = getReserves(); // gas savings
                uint balance0 = IERC20(token0).balanceOf(address(this));
                uint balance1 = IERC20(token1).balanceOf(address(this));
                // 获取到 加入 支持的 token0 和 token 1 的 数量
                uint amount0 = balance0.sub(_reserve0);
                uint amount1 = balance1.sub(_reserve1);

                // _mintFee 看下面
                bool feeOn = _mintFee(_reserve0, _reserve1);
                uint _totalSupply = totalSupply; // gas savings, must be defined here since totalSupply can update in _mintFee

                if (_totalSupply == 0) {
                    // 初次添加 时 会减去10^3 , 方便后期撤池子后,任留有一点余额, 保证计算安全
                    liquidity = Math.sqrt(amount0.mul(amount1)).sub(MINIMUM_LIQUIDITY);
                _mint(address(0), MINIMUM_LIQUIDITY); // permanently lock the first MINIMUM_LIQUIDITY tokens
                } else {
                    // 这里S的计算,选择了其中小的那一个, 主要还是为了保证安全
                    liquidity = Math.min(amount0.mul(_totalSupply) / _reserve0, amount1.mul(_totalSupply) / _reserve1);
                }
                require(liquidity > 0, 'UniswapV2: INSUFFICIENT_LIQUIDITY_MINTED');

                // mint LP token
                _mint(to, liquidity);

                _update(balance0, balance1, _reserve0, _reserve1);
                // 更新k 
                if (feeOn) kLast = uint(reserve0).mul(reserve1); // reserve0 and reserve1 are up-to-date
                emit Mint(msg.sender, amount0, amount1);
            }
        ```

        * `router`
        ```
        function _addLiquidity(
            address tokenA,
            address tokenB,
            uint amountADesired,
            uint amountBDesired,
            uint amountAMin,
            uint amountBMin
        ) private returns (uint amountA, uint amountB) {
            // 检查池子是否存在
            if (IUniswapV2Factory(factory).getPair(tokenA, tokenB) == address(0)) {
                // 创建池子
                IUniswapV2Factory(factory).createPair(tokenA, tokenB);
            }
            // 获取当前池子的tokenA和tokenB的存储量
            (uint reserveA, uint reserveB) = UniswapV2Library.getReserves(factory, tokenA, tokenB);
            if (reserveA == 0 && reserveB == 0) {
                (amountA, amountB) = (amountADesired, amountBDesired);
            } else {
                // 根据用户 给定的tokenA 和tokenB 的数量, 来计算 当前如果给定tokenA 则需要多少tokenB 来加池子. 如果tokenB 不够,则根据tokenB 来计算tokenA 的数量. 进而得出用户能够加多少tokenA和tokenB进池子
                uint amountBOptimal = UniswapV2Library.quote(amountADesired, reserveA, reserveB);
                if (amountBOptimal <= amountBDesired) {
                    require(amountBOptimal >= amountBMin, 'UniswapV2Router: INSUFFICIENT_B_AMOUNT');
                    (amountA, amountB) = (amountADesired, amountBOptimal);
                } else {
                    uint amountAOptimal = UniswapV2Library.quote(amountBDesired, reserveB, reserveA);
                    assert(amountAOptimal <= amountADesired);
                    require(amountAOptimal >= amountAMin, 'UniswapV2Router: INSUFFICIENT_A_AMOUNT');
                    (amountA, amountB) = (amountAOptimal, amountBDesired);
                }
            }
        }
        ```

        ```
        function quote(uint amountA, uint reserveA, uint reserveB) internal pure returns (uint amountB) {
            require(amountA > 0, 'UniswapV2Library: INSUFFICIENT_AMOUNT');
            require(reserveA > 0 && reserveB > 0, 'UniswapV2Library: INSUFFICIENT_LIQUIDITY');

            // 加入池子的数量必须与储备量的比例相同
            // dy/dx = y0/x0
            // => dy = dx*(y0/x0)
            amountB = amountA.mul(reserveB) / reserveA;
        }
        ```

        ```
        function addLiquidity(
            address tokenA,
            address tokenB,
            uint amountADesired,
            uint amountBDesired,
            uint amountAMin,
            uint amountBMin,
            address to,
            uint deadline
        ) external override ensure(deadline) returns (uint amountA, uint amountB, uint liquidity) {
            // 得到用户提供的资金能够加入多少进池子
            (amountA, amountB) = _addLiquidity(tokenA, tokenB, amountADesired, amountBDesired, amountAMin, amountBMin);
            address pair = UniswapV2Library.pairFor(factory, tokenA, tokenB);
            // 转入池子
            TransferHelper.safeTransferFrom(tokenA, msg.sender, pair, amountA);
            TransferHelper.safeTransferFrom(tokenB, msg.sender, pair, amountB);
            // 调用pair 的mint 方法,获得LP token
            liquidity = IUniswapV2Pair(pair).mint(to);
        }
        function addLiquidityETH(
            address token,
            uint amountTokenDesired,
            uint amountTokenMin,
            uint amountETHMin,
            address to,
            uint deadline
        ) external override payable ensure(deadline) returns (uint amountToken, uint amountETH, uint liquidity) {
            // 得到用户提供的资金能够加入多少进池子
            (amountToken, amountETH) = _addLiquidity(
                token,
                WETH,
                amountTokenDesired,
                msg.value,
                amountTokenMin,
                amountETHMin
            );
            
            address pair = UniswapV2Library.pairFor(factory, token, WETH);
            // 将token 加入池子
            TransferHelper.safeTransferFrom(token, msg.sender, pair, amountToken);
            // eth 转WETH , pair 只支持erc20 token, 原生币需要转成wrapper eth. 转成weth 后, 加入池子
            IWETH(WETH).deposit{value: amountETH}();
            assert(IWETH(WETH).transfer(pair, amountETH));
            // 调用pair 的mint 方法,获得LP token
            liquidity = IUniswapV2Pair(pair).mint(to);
            // 剩余的eth 需要退还给用户
            if (msg.value > amountETH) TransferHelper.safeTransferETH(msg.sender, msg.value - amountETH); // refund dust eth, if any
        }
        ```
  
### 撤销池子
* 添加池子的流程
![添加支持流程](../image/add-liquidity-process.png)

* 撤池子的推导公式
    * 当前已知的三个条件
      * 1、目前的liquidity token
        $L_{0} = \sqrt{x_0y_0}$
      * 2、撤销流动性后的liquidity token
        $L_{1} = \sqrt{(x_0 - d_x)(y_0 - d_y)}$
      * 3、流动性份额等比例原则
        $\frac{x_0}{y_0} = \frac{d_x}{d_y}$

        => $d_x = d_y \frac{x_0}{y_0}$

        => $d_y = d_x \frac{y_0}{y_0}$

    * 推导公式
        * 设: $S$ 为用户持有的LP token 数量. T为 当前池子中LP token 数量
          $\frac{S}{T} = \frac{L_0 - L_1}{L_0}$

          将 $d_y = d_x \frac{y_0}{x_0}$ 带入至 $L_1$

          => $L_1 = \sqrt{(x_0 - d_x)(y_0 - d_x\frac{y_0}{x_0})}$

          =>  $L_1 = \sqrt{x_0y_0 - 2d_xy_0+d_x^2\frac{y_0}{x_0}}$

          => $L_1 = \sqrt{x_0y_0(1 - 2\frac{d_x}{x_0} + \frac{d_x^2}{x_0^2})}$

          => $L_{1} = \sqrt{x_0y_0(1-\frac{d_x}{x_0})^2}$

          => $\frac{S}{T} = \frac{\sqrt{x_0y_0} - \sqrt{x_0y_0(1-\frac{d_x}{x_0})^2}}{\sqrt{x_0y_0}}$


          => $\frac{S}{T} = 1 - 1 - \frac{d_x}{x_0}$

          => $\frac{S}{T} = \frac{d_x}{x_0}$
          
          => $d_x = x_0 \frac{S}{T}$

          将 $d_x = d_y \frac{x_0}{y_0}$ 代入

          => $d_y \frac{x_0}{y_0} = x_0 \frac{S}{T} $

          => $d_y = y_0 \frac{S}{T}$

          故用户撤池子的时候,应该提取$d_x$$d_y$ 的数量分别如下
          $d_x = x_0 \frac{S}{T}$

          $d_y = y_0 \frac{S}{T}$


* 撤池子的代码解析
    `factory`
    ```
    function burn(address to) external lock returns (uint amount0, uint amount1) {
        // 获取当前记录的token 储备量
        (uint112 _reserve0, uint112 _reserve1,) = getReserves(); // gas savings
        address _token0 = token0;                                // gas savings
        address _token1 = token1;                                // gas savings

        // 获取池子的balance 
        uint balance0 = IERC20(_token0).balanceOf(address(this));
        uint balance1 = IERC20(_token1).balanceOf(address(this));

        // 当前地址是不存储 LP token 的, 如果有LP token, 说明有人在做撤销池子, 需要把LP token 对应的token0 和token1 转给用户
        uint liquidity = balanceOf[address(this)];

        bool feeOn = _mintFee(_reserve0, _reserve1);
        uint _totalSupply = totalSupply; // gas savings, must be defined here since totalSupply can update in _mintFee
        // 计算出dx,dy 的 金额, 需转给用户 
        amount0 = liquidity.mul(balance0) / _totalSupply; // using balances ensures pro-rata distribution
        amount1 = liquidity.mul(balance1) / _totalSupply; // using balances ensures pro-rata distribution
        require(amount0 > 0 && amount1 > 0, 'UniswapV2: INSUFFICIENT_LIQUIDITY_BURNED');
        
        // 销毁当前地址的LP token 
        _burn(address(this), liquidity);

        // 转给用户
        _safeTransfer(_token0, to, amount0);
        _safeTransfer(_token1, to, amount1);
        // 转出后,得到新的池子数据,并更新
        balance0 = IERC20(_token0).balanceOf(address(this));
        balance1 = IERC20(_token1).balanceOf(address(this));

        // 更新池子的信息
        _update(balance0, balance1, _reserve0, _reserve1);
        if (feeOn) kLast = uint(reserve0).mul(reserve1); // reserve0 and reserve1 are up-to-date
        emit Burn(msg.sender, amount0, amount1, to);
    }
    ```

    `router`
    ```
    function removeLiquidity(
        address tokenA,
        address tokenB,
        uint liquidity,
        uint amountAMin,
        uint amountBMin,
        address to,
        uint deadline
    ) public virtual override ensure(deadline) returns (uint amountA, uint amountB) {
        // 获取池子地址
        address pair = UniswapV2Library.pairFor(factory, tokenA, tokenB);
        // 将 LP token 转给 池子
        IUniswapV2Pair(pair).transferFrom(msg.sender, pair, liquidity); // send liquidity to pair
        // 调用池子的burn 方法
        (uint amount0, uint amount1) = IUniswapV2Pair(pair).burn(to);
        (address token0,) = UniswapV2Library.sortTokens(tokenA, tokenB);
        // 检查撤池子的token 数据,不能低于用户想用的值, 防夹子
        (amountA, amountB) = tokenA == token0 ? (amount0, amount1) : (amount1, amount0);
        require(amountA >= amountAMin, 'UniswapV2Router: INSUFFICIENT_A_AMOUNT');
        require(amountB >= amountBMin, 'UniswapV2Router: INSUFFICIENT_B_AMOUNT');
    }
    ```


### swap 