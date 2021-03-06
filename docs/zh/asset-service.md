# Asset Service

Asset service 是 huobi-chain 的内置资产模块，负责管理链原生资产以及第三方发行资产。

## 特点

- 资产成为一等公民：Asset 模块利用 muta 框架提供的 service 能力，做为链原生提供的功能，让加密资产成为一等公民
  
- 第三方发行资产： 用户可以使用 Asset 模块发行资产，自定义资产属性和总量等

- 资产与合约交互： 未来可以打通虚拟机和资产模块，为资产的广泛使用提供支持

## 接口

Asset 模块采用类似以太坊 ERC-20 的接口设计，主要包含：

1. 发行资产

```rust
// 资产数据结构
pub struct Asset {
    pub id:     Hash,
    pub name:   String,
    pub symbol: String,
    pub supply: u64,
    pub issuer: Address,
}

// 发行资产接口
// 资产 ID 自动生成，确保唯一
fn create_asset(&mut self, ctx: ServiceContext, payload: CreateAssetPayload) -> ProtocolResult<Asset>;

// 发行资产参数
pub struct CreateAssetPayload {
    pub name:   String,
    pub symbol: String,
    pub supply: u64,
}

// Example: graphiql send tx 
mutation create_asset{
  unsafeSendTransaction(inputRaw: {
    serviceName:"asset",
    method:"create_asset",
    payload:"{\"name\":\"Test Coin\",\"symbol\":\"TC\",\"supply\":100000000}",
    timeout:"0x172",
    nonce:"0x9db2d7efe2b61a88827e4836e2775d913a442ed2f9096ca1233e479607c27cf7",
    chainId:"b6a4d7da21443f5e816e8700eea87610e6d769657d6b8ec73028457bf2ca4036",
    cyclesPrice:"0x9999",
    cyclesLimit:"0x9999"
  }, inputPrivkey: "0x30269d47fcf602b889243722b666881bf953f1213228363d34cf04ddcd51dfd2"
  )
}
```

2. 查询资产信息

```rust
// 查询接口
fn get_asset(&self, ctx: ServiceContext, payload: GetAssetPayload) -> ProtocolResult<Asset>；

// 查询参数
pub struct GetAssetPayload {
    pub id: Hash, // 资产 ID
}

// Example: graphiql send tx 
query get_asset{
  queryService(
  caller: "016cbd9ee47a255a6f68882918dcdd9e14e6bee1"
  serviceName: "asset"
  method: "get_asset"
  payload: "{\"id\": \"5f1364a8e6230f68ccc18bc9d1000cedd522d6d63cef06d0062f832bdbe1a78a\"}"
  ){
    ret,
    isError
  }
}
```

3. 转账

```rust
// 转账接口
fn transfer(&mut self, ctx: ServiceContext, payload: TransferPayload) -> ProtocolResult<()>；

// 转账参数
pub struct TransferPayload {
    pub asset_id: Hash,
    pub to:       Address,
    pub value:    u64,
}

// Example: graphiql send tx 
mutation transfer{
  unsafeSendTransaction(inputRaw: {
    serviceName:"asset",
    method:"transfer",
    payload:"{\"asset_id\":\"5f1364a8e6230f68ccc18bc9d1000cedd522d6d63cef06d0062f832bdbe1a78a\",\"to\":\"f8389d774afdad8755ef8e629e5a154fddc6325a\", \"value\":10000}",
    timeout:"0x289",
    nonce:"0x9db2d7efe2b61a28827e4836e2775d913a442ed2f9096ca1233e479607c27cf7",
    chainId:"b6a4d7da21443f5e816e8700eea87610e6d769657d6b8ec73028457bf2ca4036",
    cyclesPrice:"0x9999",
    cyclesLimit:"0x9999",
    }, inputPrivkey: "0x30269d47fcf602b889243722b666881bf953f1213228363d34cf04ddcd51dfd2"
  )
}
```

4. 查询余额

```rust
// 查询接口
fn get_balance(&self, ctx: ServiceContext, payload: GetBalancePayload) -> ProtocolResult<GetBalanceResponse> 

// 查询参数
pub struct GetBalancePayload {
    pub asset_id: Hash,
    pub user:     Address,
}

// 返回值
pub struct GetBalanceResponse {
    pub asset_id: Hash,
    pub user:     Address,
    pub balance:  u64,
}

// Example: graphiql send tx 
query get_balance{
  queryService(
  caller: "016cbd9ee47a255a6f68882918dcdd9e14e6bee1"
  serviceName: "asset"
  method: "get_balance"
  payload: "{\"asset_id\": \"5f1364a8e6230f68ccc18bc9d1000cedd522d6d63cef06d0062f832bdbe1a78a\", \"user\": \"016cbd9ee47a255a6f68882918dcdd9e14e6bee1\"}"
  ){
    ret,
    isError
  }
}
```

5. 批准额度

```rust
// 批准接口
fn approve(&mut self, ctx: ServiceContext, payload: ApprovePayload) -> ProtocolResult<()>;

// 批准参数
pub struct ApprovePayload {
    pub asset_id: Hash,
    pub to:       Address,
    pub value:    u64,
}

// Example: graphiql send tx 
  unsafeSendTransaction(inputRaw: {
    serviceName:"asset",
    method:"approve",
    payload:"{\"asset_id\":\"5f1364a8e6230f68ccc18bc9d1000cedd522d6d63cef06d0062f832bdbe1a78a\",\"to\":\"f8389d774afdad8755ef8e629e5a154fddc6325a\", \"value\":10000}",
    timeout:"0x378",
    nonce:"0x9db2d7efe2b61a28827e4836e2775d913a442ed2f9096ca1233e479607c27cf7",
    chainId:"b6a4d7da21443f5e816e8700eea87610e6d769657d6b8ec73028457bf2ca4036",
    cyclesPrice:"0x9999",
    cyclesLimit:"0x9999",
    }, inputPrivkey: "0x30269d47fcf602b889243722b666881bf953f1213228363d34cf04ddcd51dfd2"
  )
}
```

6. 授权转账

```rust
// 接口
fn transfer_from(&mut self, ctx: ServiceContext, payload: TransferFromPayload) -> ProtocolResult<()>；

// 参数
pub struct TransferFromPayload {
    pub asset_id:  Hash,
    pub sender:    Address,
    pub recipient: Address,
    pub value:     u64,
}

// Example: graphiql send tx 
mutation transfer_from{
  unsafeSendTransaction(inputRaw: {
    serviceName:"asset",
    method:"transfer_from",
    payload:"{\"asset_id\":\"5f1364a8e6230f68ccc18bc9d1000cedd522d6d63cef06d0062f832bdbe1a78a\",\"sender\":\"016cbd9ee47a255a6f68882918dcdd9e14e6bee1\", \"recipient\":\"fffffd774afdad8755ef8e629e5a154fddc6325a\", \"value\":5000}",
    timeout:"0x12c",
    nonce:"0x9db2d7efe2b61a28827e4836e2775d913a442ed2f9096ca1233e479607c27cf7",
    chainId:"b6a4d7da21443f5e816e8700eea87610e6d769657d6b8ec73028457bf2ca4036",
    cyclesPrice:"0x9999",
    cyclesLimit:"0x9999",
    }, inputPrivkey: "0x45c56be699dca666191ad3446897e0f480da234da896270202514a0e1a587c3f"
  )
}
```

7. 查询限额

```rust
// 查询接口
fn get_allowance(&self, ctx: ServiceContext, payload: GetAllowancePayload) -> ProtocolResult<GetAllowanceResponse>；

// 查询参数
pub struct GetAllowancePayload {
    pub asset_id: Hash,
    pub grantor:  Address,
    pub grantee:  Address,
}

// 返回值
pub struct GetAllowanceResponse {
    pub asset_id: Hash,
    pub grantor:  Address,
    pub grantee:  Address,
    pub value:    u64,
}

// Example: graphiql send tx 
query get_allowance{
  queryService(
  caller: "016cbd9ee47a255a6f68882918dcdd9e14e6bee1"
  serviceName: "asset"
  method: "get_allowance"
  payload: "{\"asset_id\": \"5f1364a8e6230f68ccc18bc9d1000cedd522d6d63cef06d0062f832bdbe1a78a\", \"grantor\": \"016cbd9ee47a255a6f68882918dcdd9e14e6bee1\", \"grantee\": \"f8389d774afdad8755ef8e629e5a154fddc6325a\"}"
  ){
    ret,
    isError
  }
}
```