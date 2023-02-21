---
title: "Go with wasm"
date: 2023-02-21T11:52:51+08:00
---


# WASM


WebAssembly（缩写为 wasm）是一种低级的、面向栈的虚拟机，用于在现代 Web 浏览器中执行二进制代码。它被设计成一个可移植、高效和安全的执行环境，可以被用于许多不同的场景，包括：

在 Web 浏览器中提供比 JavaScript 更高效的编程语言。wasm 可以使得高性能的编程语言如 C/C++、Rust 和 Go 等运行在 Web 平台上，从而为 Web 应用提供更快的执行速度和更好的用户体验。

在 Web 应用中使用第三方库。由于 wasm 的二进制格式可以被多种语言所编译生成，因此可以在 Web 应用中使用其他语言编写的库，而不需要对库的源代码进行修改。

跨平台开发。wasm 可以在多个平台上运行，因此可以在不同的硬件和操作系统上共享代码，简化跨平台开发的难度和成本。

# In Go 

Go 语言的 WebAssembly（wasm）支持允许开发人员编写 Go 语言代码，并将其编译为 wasm 模块，以便在 Web 平台上运行。Go 语言的 wasm 支持是 Go 1.11 版本中引入的，并在后续版本中得到了进一步的改进和优化。

使用 Go 语言的 wasm 支持，可以将现有的 Go 语言代码直接编译为 wasm 模块，然后在 Web 平台上运行。这为开发人员提供了许多好处，包括：

更高的性能。由于 wasm 是一种比 JavaScript 更低级别的虚拟机，因此可以提供更高的性能和更低的内存消耗。

更好的安全性。由于 wasm 运行在 Web 浏览器的安全沙箱中，因此可以提供更好的安全性，避免 Web 应用被恶意代码攻击。

更高的可移植性。由于 wasm 的二进制格式可以在多个平台上运行，因此可以使得 Go 语言代码在不同的平台上共享，从而简化了跨平台开发的难度。



# Demo 

## 原生go 

### code
```go
package main

import "syscall/js"

func Add(his js.Value, args []js.Value) any {
	return "ok "
}

func main() {
	alert := js.Global().Get("alert")
    js.Global().Set("Add", js.FuncOf(Add))
	alert.Invoke("Hello World!")

	<-make(chan int, 0)
}
```

> 注意  js.Global().Set("Add",js.FuncOf(Add)) 这里是需要将go的方法注册到js中

### 编译成为wasm 

`GOOS=js GOARCH=wasm go build -o add.wasm`


### js调用

> wasm_exec.js 位于 `$GOROOT/misc/wasm/wasm_exec.js` 属于js 补丁

```js 
	<script src="wasm_exec.js"></script>
	<script>
		if (!WebAssembly.instantiateStreaming) { // polyfill
			WebAssembly.instantiateStreaming = async (resp, importObject) => {
				const source = await (await resp).arrayBuffer();
				return await WebAssembly.instantiate(source, importObject);
			};
		}

		const go = new Go();
		let mod, inst;
		WebAssembly.instantiateStreaming(fetch("add.wasm"), go.importObject).then((result) => {
			mod = result.module;
			inst = result.instance;
			document.getElementById("runButton").disabled = false;
		}).catch((err) => {
			console.error(err);
		});

		async function run() {
			console.clear();
			await go.run(inst);
			inst = await WebAssembly.instantiate(mod, go.importObject); // reset instance

		}
	</script>
```




## tinygo 
`brew tap tinygo-org/tools && brew install tinygo`

### code 

```go 
package main

// This calls a JS function from Go.
func main() {
    println("adding two numbers:", add(2, 3)) // expecting 5
}

// This function is imported from JavaScript, as it doesn't define a body.
// You should define a function named 'add' in the WebAssembly 'env'
// module from JavaScript.
//
//export add
func add(x, y int) int

// This function is exported to JavaScript, so can be called using
// exports.multiply() in JavaScript.
//export multiply
func multiply(x, y int) int {
    return x * y;
}

```


### 运行
`tinygo build -o wasm.wasm -target wasm ./main.go`

> 注意 需要执行 `cp $(tinygo env TINYGOROOT)/targets/wasm_exec.js .` 来下载tinygo的 wasm_exec.js


### js 访问

```js 
instance.exports.multiply()

```



### tinygo 特色
TinyGo 是一种针对嵌入式设备的 Go 编译器，它可以将 Go 代码编译成非常小的、快速运行的二进制文件，以适应嵌入式设备的资源限制。以下是一些 TinyGo 的场景：

物联网设备：由于嵌入式设备通常具有较小的存储和内存，TinyGo 可以帮助将 Go 代码编译为更小、更快的二进制文件，以满足 IoT 设备的资源限制。

机器人和无人机：TinyGo 可以使机器人和无人机的控制系统更加灵活和可靠，提高其自主决策和反应速度。

传感器：TinyGo 可以编写用于读取和处理传感器数据的代码，并将其编译为小型二进制文件，以在资源受限的设备上运行。

嵌入式系统：TinyGo 也可用于构建嵌入式系统的固件和驱动程序，从而提高系统的效率和可靠性。

游戏控制器：TinyGo 可以用于编写游戏控制器代码，以实现更快的响应时间和更低的延迟，从而提高游戏体验。



## 差异
tinygo 生成的wasm提交较小,推荐用`tinygo`
