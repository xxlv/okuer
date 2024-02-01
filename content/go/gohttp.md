---
title: "Go net/http 分析"
date: 2024-01-01T21:09:14+08:00
draft: false
---

## 背景
- RoundTrip
- Transport 
- TLS
- ALPN
- Http2



## 如何工作的?

### Client的建立
这是一个非常简单的使用Http Client的例子

```go
	client := &http.Client{
		Timeout: 30 * time.Second,
	}
```

当上面的Client 初始化完成后, 将经过下面几个阶段:
1. Do 入口
2. Parse And Get Req 
3. Send 
4. send with transport (DefaultTransport with h2 ,impl RoundTripper interface)

``` mermaid 
graph LR
    A[Do 入口] --> B[Parse And Get Req]
    B --> C[Send]
    C --> D[send with transport]
    D --> E{DefaultTransport with h2}
    E --> F[impl RoundTripper interface]
```


这里注意，如果没有配置Transport,go将会选择一个 DefaultTansport,并且开启 ForceAttemptHTTP2 

*ForceAttemptHTTP2* 这个字段为True将会忽略你的 non-zero Dial, DialTLS, or DialContext func or TLSClientConfig 
当你提供自定义的拨号（dial）函数或 TLS 配置时，可能会有一些特别的需求或者约束，这些需求或约束可能与 HTTP/2 的某些特性不兼容。 设置为True则总是尝试使用h2。


``` go
var DefaultTransport RoundTripper = &Transport{
	Proxy: ProxyFromEnvironment,
	DialContext: defaultTransportDialContext(&net.Dialer{
		Timeout:   30 * time.Second,
		KeepAlive: 30 * time.Second,
	}),
	ForceAttemptHTTP2:     true,
	MaxIdleConns:          100,
	IdleConnTimeout:       90 * time.Second,
	TLSHandshakeTimeout:   10 * time.Second,
	ExpectContinueTimeout: 1 * time.Second,
}

```


### Transport的任务

Transport 是实现了RoundTripper 接口的一个代表Http请求模型的Struct

下面我摘录一下比较重要的几个字段,

1. idleConn 空闲的链接
2. DialTLS/Dial 拨号
3. TLSNextProto NextProto

> TLSNextProto 是一个映射（map），它指定了在 TLS ALPN（Application-Layer Protocol Negotiation，应用层协议协商）之后，Transport 如何切换到另一个协议（如 HTTP/2）。

>    ALPN 是一个 TLS 扩展，它允许客户端和服务器在握手阶段就协商出一个共同支持的应用层协议，比如 HTTP/1.1、HTTP/2 或者其他的协议。这个协议的名称会在 TLS 握手完成后被返回，比如 "http/1.1" 或者 "h2"（代表 HTTP/2）。

>    如果 Transport 通过 TLS 连接拨号并且协议名称非空，同时 TLSNextProto 包含对应的映射条目（例如 "h2"），那么将会调用与该条目关联的函数。这个函数会接收请求的权限（例如 "example.com" 或 "example.com:1234"）和 TLS 连接，然后必须返回一个 RoundTripper，由这个 RoundTripper 来处理请求。

>    如果 TLSNextProto 不为 nil，HTTP/2 支持将不会自动启用。这是因为 TLSNextProto 提供了一种方式来自定义协议协商后的处理，所以如果你设置了 TLSNextProto，Transport 将会假定你想要自己控制协议切换的过程，包括是否启用 HTTP/2。

``` go
type Transport struct {
	idleConn     map[connectMethodKey][]*persistConn // most recently used at end
	
	Dial func(network, addr string) (net.Conn, error)
	DialTLS func(network, addr string) (net.Conn, error)
	TLSNextProto map[string]func(authority string, c *tls.Conn) RoundTripper
	h2transport        h2Transport // non-nil if http2 wired up
	ForceAttemptHTTP2 bool

}
```

#### 关于Conn
Conn 本质上是一个TCP链接(net.Conn) 
persistConn/*tls.Conn 是一个包装的net.Conn 

net.Conn 叫plainConn, 建立在在其之上的连接都通过各种数据协议完成升级.比如TLS的Conn 可以完成数据加密通讯.
##### TLS Conn
TLS Conn 主要完成

1. 加密通信：TLS 通过对数据进行加密来保护通信内容的隐私和安全。这意味着，即使数据在传输过程中被拦截，攻击者也无法读取或理解被加密的数据。

2. 身份验证：TLS 使用证书来验证服务器（有时也用于验证客户端）的身份，保证数据被发送到正确的服务器，而不是被发送到冒充者的服务器。



### RoundTripper的流程(DefaultTransport)
DefaultTransport 是支持H2协议的。 但是服务端可能并不支持h2.因此 Client需要明确将自己支持的协议告知Server

这是如何做的呢？ 
答案很简单,商量着来。

我们首先注册一下 onceSetNextProtoDefaults 设置好 http2Transport 并且注册 UpgradeFn

此时我们获取一个Conn并且升级成为TLS，在Tls Conn完成后，通过ALPN层获取到服务器支持H2,因此调用UpgradeFn
将TLS Conn 升级成为H2.

> 因为TLS(https)已经被广泛接受了，因此ALPN 协议在 TLS上实现减少开销 



### ALPN 

简单说说ALPN(Application-Layer Protocol Negotiation)。
Client 需要跟 Server 进行友好协商，确定究竟这个TCP(TLS连接)之上的数据交换采取什么协议。
因此，Client 需要提供自己的支持的协议列表，Server 从中选择能够支持一个.

这就是ALPN
#### 如何做的呢？
既然是建立在TSL 基础上的，流程就是在建立TLS之后,通过发送 `ClientHello` message 在
alpnProtocols 字段上设置支持的待协商协议列表.

Server将返回 `ServerHello` message. 在这个message 中将返回Server 确定的alpnProtocols.例如 `h2`

## Upgrade to h2
在TLS 上确定了支持`h2` 之后，就需要进行UpgradeFn 来使得整个协议升级为`http2`

## Http2

http2 的核心是一个二进制分帧层。可以reuse Conn.

想象在一个通道上传输(Client Send) 和 接受(Client Accept) 货物, Client和Server建立连接之后，就像是存在一个传输的通道。
我们需要一个编号来区分到底是什么货物。当Client 将一个苹果发送给服务端的时候。Server针对这个苹果进行加工处理并返回的时候，同样要告知，处理的苹果正式Client 发送的那个。 因此需要一个叫 `StreamID` 的标记来区分每一个数据帧。 

在 `h2_bundle.go` 中的 `cc.addStreamLocked(cs) // assigns stream ID` 负责产生这个ID 
这是`h2` 协议能reuse Conn并且保证数据准确的基础。




Http2 roundTrip 的核心如下(go v1.21)

可以看到，已经升级为h2的Conn 在处理Request 的流程在一个retry逻辑中实现

如下，第一次访问的时候，connPool内不存在，因此需要返回 `http2noCachedConnError`

这个错误会被转换成一个sentenal error `ErrSkipAltProtocol` 因此会执行创建Conn 升级TLS并且进行H2升级的全过程.

```go

// RoundTripOpt is like RoundTrip, but takes options.
func (t *http2Transport) RoundTripOpt(req *Request, opt http2RoundTripOpt) (*Response, error) {
	if !(req.URL.Scheme == "https" || (req.URL.Scheme == "http" && t.AllowHTTP)) {
		return nil, errors.New("http2: unsupported scheme")
	}

	addr := http2authorityAddr(req.URL.Scheme, req.URL.Host)
	for retry := 0; ; retry++ {
		cc, err := t.connPool().GetClientConn(req, addr)
		if err != nil {
			t.vlogf("http2: Transport failed to get client conn for %s: %v", addr, err)
			return nil, err
		}
		reused := !atomic.CompareAndSwapUint32(&cc.reused, 0, 1)
		http2traceGotConn(req, cc, reused)
		res, err := cc.RoundTrip(req)
		if err != nil && retry <= 6 {
			roundTripErr := err
			if req, err = http2shouldRetryRequest(req, err); err == nil {
				// After the first retry, do exponential backoff with 10% jitter.
				if retry == 0 {
					t.vlogf("RoundTrip retrying after failure: %v", roundTripErr)
					continue
				}
				backoff := float64(uint(1) << (uint(retry) - 1))
				backoff += backoff * (0.1 * mathrand.Float64())
				d := time.Second * time.Duration(backoff)
				timer := http2backoffNewTimer(d)
				select {
				case <-timer.C:
					t.vlogf("RoundTrip retrying after failure: %v", roundTripErr)
					continue
				case <-req.Context().Done():
					timer.Stop()
					err = req.Context().Err()
				}
			}
		}
		if err != nil {
			t.vlogf("RoundTrip failure: %v", err)
			return nil, err
		}
		return res, nil
	}
}
```


## 再次回到Conn

这里在讨论一些细节，关于Conn

在上面H2的获得Conn的过程中，因为cache miss 导致回退到TCP连接的创建过程。TCP究竟是如何创建一个Conn的呢？

`(t *Transport) getConn` 这个方法负责创建一个PersisConn 



1. 构造 wantConn 
    wantConn 可能创建新的Conn 也可能从idle获取
2. 新起一个Goroutine 跑 dial
    - 如有自定义的dial hasCustomTLSDialer 跑自定义
    - 否则跑 `t.dial(ctx, "tcp", cm.addr())` 

    在内部会走到`DialContext` 方法进行Dial


    Shadow the nettrace (if any) during resolve so Connect events don't fire for DNS lookups.


    ``` go
        // Shadow the nettrace (if any) during resolve so Connect events don't fire for DNS lookups.
        resolveCtx := ctx
        if trace, _ := ctx.Value(nettrace.TraceKey{}).(*nettrace.Trace); trace != nil {
            shadow := *trace
            shadow.ConnectStart = nil
            shadow.ConnectDone = nil
            resolveCtx = context.WithValue(resolveCtx, nettrace.TraceKey{}, &shadow)
        }

    ```

    注意，我们跟一个domain建立Conn，因此，需要根据domain 找到对应的Ip，这部分在 `lookup.go` 的`lookupIPAddr`方法内支持

    当查询到多条地址的时候，通过`dialParallel` 来竞赛，防止出现长尾效应。


3. 上面完成了Conn,下面就开始 将一个Pure Conn 变成TLS 通过 `addTLS` 来实现这一点

#### addTLS 
具体来讲，就是 `clientHandshake` 通过 `hello` message 跟服务端产生握手交换协议数据。 ALPN的数据也是在这里发送给Server的。 到这一步，已经完成了TLS的Conn。接下来，就是回到了H2,借助UpradeFn, TLS之上的Conn传输将通过Http2协议。

```go
	upgradeFn := func(authority string, c *tls.Conn) RoundTripper {
		addr := http2authorityAddr("https", authority)
		if used, err := connPool.addConnIfNeeded(addr, t2, c); err != nil {
			go c.Close()
			return http2erringRoundTripper{err}
		} else if !used {
			// Turns out we don't need this c.
			// For example, two goroutines made requests to the same host
			// at the same time, both kicking off TCP dials. (since protocol
			// was unknown)
			go c.Close()
		}
		return t2
	}
```

底层采用 newClientConn 来将pureConn 升级为H2 Conn
> 下面的net.Conn 已经是一个TLS Conn 了
``` go
func (t *http2Transport) newClientConn(c net.Conn, singleUse bool) (*http2ClientConn, error) {
	cc := &http2ClientConn{
		t:                     t,
		tconn:                 c,
		readerDone:            make(chan struct{}),
		nextStreamID:          1,
		maxFrameSize:          16 << 10,                         // spec default
		initialWindowSize:     65535,                            // spec default
		maxConcurrentStreams:  http2initialMaxConcurrentStreams, // "infinite", per spec. Use a smaller value until we have received server settings.
		peerMaxHeaderListSize: 0xffffffffffffffff,               // "infinite", per spec. Use 2^64-1 instead.
		streams:               make(map[uint32]*http2clientStream),
		singleUse:             singleUse,
		wantSettingsAck:       true,
		pings:                 make(map[[8]byte]chan struct{}),
		reqHeaderMu:           make(chan struct{}, 1),
	}
    // ...
}
```
## 总结
Ok,目前全部的整体流程基本整理完了。 当然还有很多细节，比如三次握手,加密套件,连接复用，监听数据的循环等等。
