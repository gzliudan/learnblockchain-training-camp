# 第 7 周第 1 课作业

## 第 1 题：设计一个看涨期权 Token

- 创建期权 Token 时，确认标的的价格与行权日期；
- 发行方法（项目方⻆色）：根据转入的标的（ETH）发行期权 Token；
- （可选）：可以用期权 Token 与 USDT 以一个较低的价格创建交易对，模拟用户购买期权。
- 行权方法（用户⻆色）：在到期日当天，可通过指定的价格兑换出标的资产，并销毁期权 Token
- 过期销毁（项目方⻆色）：销毁所有期权 Token 赎回标的，USDT 资金。