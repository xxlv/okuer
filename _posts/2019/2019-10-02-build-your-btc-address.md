# 安全离线生成比特币的钱包地址

比特币是数字资产，受到密码学（密钥对）保护，一旦你的私钥被公开，你的资产将不在安全。传统的账号/密码受到服务提供商的保护，比如在支付宝，你的账号账号被盗，导致的财产损失，会被支付宝保险赔付。因为有一个第三方信任机构（保险/支付宝）。但是比特币是构建在区块链上的，区块链是去中心化的。这意味着没有一个类似支付宝的角色来保证你的财产安全，一切都要靠自己。
目前，有许多在线生成比特币地址的服务，这些服务大大简化了生成步骤，但同时引入了非常大的分析。一旦你的账号在网络环境被生成，它有极可能被泄漏。泄漏途径有许多，如本地机器被黑客入侵、服务提供商私自保存了密钥。 因此，如果资产金额较大，强烈建议使用本地的脚本自己生成密钥对，并且永远不要在联网的环境下暴露私钥。
本文提供一种简单的离线脚本，一步步的带你生成比特币的密钥对。（本文所有脚本均在这里，https://github.com/xxlv/btc-address-gen-scripts.git）

### 第一，初始化依赖
你需要安全python环境，同时引入包
```
sudo pip install bitcoin
sudo pip install ecdsa
```
第二步，拔掉网线或者断开wifi
非常重要的步骤，如果不是测试，一定要做！！！
第三步，运行脚本，生成密钥对
```
python key-to-address-ecc-example.py
```
### 生成的结果如下：
>
Private Key (hex) is:  14962ffc7f1c9dba79aadb608cd7088574e41a418a5349004309b722a9051b90
Private Key (decimal) is:  9311615220753377728968374325829133502017354028737325199749815733352682298256
Private Key (WIF) is:  5HyMSVcZMnBeGEoFbPqRaPHK1xVpHj6yCGnC4bvqxRUm72uuvbx
Private Key Compressed (hex) is:  14962ffc7f1c9dba79aadb608cd7088574e41a418a5349004309b722a9051b9001
Private Key (WIF-Compressed) is:  2SdMjLGhnybsEFJmA85WrTioEtQAaKc8PQjeMQRFd5Q6AKdxygTY5D
Public Key (x,y) coordinates is: (32009024601293775410666277862003376881148118697205564723468898850638045984780, 64137136342866323469645354187456635120169454941786517702226349133247186720277)
Public Key (hex) is: 0446c477454946ee7187a0b859cedf3c4799726b5d8921b22b6b366b97178d380c8dcc552e1ab8546673dcbfa05f3aa97e33df950f2020805b11e4d8489fdfd615
Compressed Public Key (hex) is: 0346c477454946ee7187a0b859cedf3c4799726b5d8921b22b6b366b97178d380c
Bitcoin Address (b58check) is: 18HWNXDVj1jSz9syiJpo2YcEofympTLQgC
Compressed Bitcoin Address (b58check) is: 1HViUwJwUofaAPKbnzNRXHqqRE6UQE7g5B

每个人的机器每次调用生成的都不是相同的。 这是btc账号的全部信息，建议冷藏。
### 第四步，生成校验签名
执行
```python signmessage.py  -s```
依此输入：

- 比特币公钥地址（上面的 Bitcoin Address (b58check) ）：18HWNXDVj1jSz9syiJpo2YcEofympTLQgC
- 签名信息如：hello
- 钱包格式的私钥（等价于私钥,Private Key (WIF)）：5HyMSVcZMnBeGEoFbPqRaPHK1xVpHj6yCGnC4bvqxRUm72uuvbx
### 输出（每个的都不同）：
```HB/9ZeoXB5zmaZq5OkC+btPcWcBly74dnJfrUmKB+EKhs+5q3stDATXXkAT2CpyovXUC6pIvD4ibfW90842f9Zw=```

### 第五步

确保信息被安全存储之后连网，在线校验签名：https://brainwalletx.github.io/#verify

此时，你已经拥有了自己的安全的比特币地址。
