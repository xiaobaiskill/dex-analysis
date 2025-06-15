bit math
---

### 简介
```
    该文件 只有两个方法
    mostSignificantBit: 找二进制数的​​最高有效位 所在位置
    leastSignificantBit: 找二进制数的​​最低有效位 所在位置
```


###  找最高有效位
```
// 找二进制最高有效位
function mostSignificantBit(uint256 x) internal pure returns (uint8 r) {
        require(x > 0, "BitMath: ZERO");
        if (x >= 2**128) { x >>= 128; r += 128; }
        if (x >= 2**64)  { x >>= 64;  r += 64;  }
        if (x >= 2**32)  { x >>= 32;  r += 32;  }
        if (x >= 2**16)  { x >>= 16;  r += 16;  }
        if (x >= 2**8)   { x >>= 8;   r += 8;   }
        if (x >= 2**4)   { x >>= 4;   r += 4;   }
        if (x >= 2**2)   { x >>= 2;   r += 2;   }
        if (x >= 2**1)   { r += 1; }
    }

通过不断的 右移, 最终确定,最高位
```


###  找最低有效位
```
// 找二进制最低有效位
function leastSignificantBit(uint256 x) internal pure returns (uint8 r) {
        require(x > 0);

        r = 255;
        // 与操作, 确定在 128 位的左边还是右边
        if (x & type(uint128).max > 0) {
            // 在右边
            r -= 128;
        } else {
            // 右移
            x >>= 128;
        }
        // 与操作, 确定在 64 位的左边还是右边
        if (x & type(uint64).max > 0) {
             // 在右边
            r -= 64;
        } else {
            // 右移
            x >>= 64;
        }
        if (x & type(uint32).max > 0) {
            r -= 32;
        } else {
            x >>= 32;
        }
        if (x & type(uint16).max > 0) {
            r -= 16;
        } else {
            x >>= 16;
        }
        if (x & type(uint8).max > 0) {
            r -= 8;
        } else {
            x >>= 8;
        }
        if (x & 0xf > 0) {
            r -= 4;
        } else {
            x >>= 4;
        }
        if (x & 0x3 > 0) {
            r -= 2;
        } else {
            x >>= 2;
        }
        if (x & 0x1 > 0) r -= 1;
    }

通过不断的判断, 与 一半位的左右 是否有值, 来确定,是否要右移, 或进行下一个一半位的左右 判断
```