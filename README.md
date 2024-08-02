# go modbus

modbus write in pure go, support rtu,ascii,tcp master library,also support tcp slave.

[![GoDoc](https://godoc.org/github.com/JasonVanCode/modubsprotocal?status.svg)](https://godoc.org/github.com/JasonVanCode/modubsprotocal)
[![Go.Dev reference](https://img.shields.io/badge/go.dev-reference-blue?logo=go&logoColor=white)](https://pkg.go.dev/github.com/JasonVanCode/modubsprotocal/v2?tab=doc)
[![codecov](https://codecov.io/gh/things-go/go-modbus/branch/master/graph/badge.svg)](https://codecov.io/gh/things-go/go-modbus)
![Action Status](https://github.com/JasonVanCode/modubsprotocal/workflows/Go/badge.svg)
[![Go Report Card](https://goreportcard.com/badge/github.com/JasonVanCode/modubsprotocal)](https://goreportcard.com/report/github.com/JasonVanCode/modubsprotocal)
[![Licence](https://img.shields.io/github/license/things-go/go-modbus)](https://raw.githubusercontent.com/things-go/go-modbus/master/LICENSE)
[![Tag](https://img.shields.io/github/v/tag/things-go/go-modbus)](https://github.com/JasonVanCode/modubsprotocal/tags)
[![Sourcegraph](https://sourcegraph.com/github.com/JasonVanCode/modubsprotocal/-/badge.svg)](https://sourcegraph.com/github.com/JasonVanCode/modubsprotocal?badge)


### Supported formats

- modbus Serial(RTU,ASCII) Client
- modbus TCP Client
- modbus TCP Server

### Features

- object pool design,reduce memory allocation
- fast encode and decode
- interface design
- simple API and support raw data api

### Installation

Use go get.
```bash
    go get github.com/JasonVanCode/modubsprotocal
```

Then import the package into your own code.
```bash
    import modbus "github.com/JasonVanCode/modubsprotocal"
```

### Supported functions

---

bit access:
*   Read Discrete Inputs
*   Read Coils
*   Write Single Coil
*   Write Multiple Coils

16-bit access:
*   Read Input Registers
*   Read Holding Registers
*   Write Single Register
*   Write Multiple Registers
*   Read/Write Multiple Registers
*   Mask Write Register
*   Read FIFO Queue

### Example

---

modbus RTU/ASCII client see [example](_examples/client_rtu_ascii)

[embedmd]:# (_examples/client_rtu_ascii/main.go go)
```go
package main

import (
	"fmt"
	"time"

	"github.com/goburrow/serial"

	modbus "github.com/JasonVanCode/modubsprotocal"
)

func main() {
	p := modbus.NewRTUClientProvider(modbus.WithEnableLogger(),
		modbus.WithSerialConfig(serial.Config{
			Address:  "/dev/ttyUSB0",
			BaudRate: 115200,
			DataBits: 8,
			StopBits: 1,
			Parity:   "N",
			Timeout:  modbus.SerialDefaultTimeout,
		}))

	client := modbus.NewClient(p)
	err := client.Connect()
	if err != nil {
		fmt.Println("connect failed, ", err)
		return
	}
	defer client.Close()

	fmt.Println("starting")
	for {
		_, err := client.ReadCoils(3, 0, 10)
		if err != nil {
			fmt.Println(err.Error())
		}

		//	fmt.Printf("ReadDiscreteInputs %#v\r\n", results)

		time.Sleep(time.Second * 2)
	}
}
```


modbus TCP client see [example](_examples/client_tcp)

[embedmd]:# (_examples/client_tcp/main.go go)
```go
package main

import (
	"fmt"
	"time"

	modbus "github.com/JasonVanCode/modubsprotocal"
)

func main() {
	p := modbus.NewTCPClientProvider("192.168.199.188:502", modbus.WithEnableLogger())
	client := modbus.NewClient(p)
	err := client.Connect()
	if err != nil {
		fmt.Println("connect failed, ", err)
		return
	}
	defer client.Close()

	fmt.Println("starting")
	for {
		_, err := client.ReadCoils(1, 0, 10)
		if err != nil {
			fmt.Println(err.Error())
		}

		//	fmt.Printf("ReadDiscreteInputs %#v\r\n", results)

		time.Sleep(time.Second * 2)
	}
}
```

modbus TCP server see [example](_examples/server_tcp)

[embedmd]:# (_examples/server_tcp/main.go go)
```go
package main

import (
	modbus "github.com/JasonVanCode/modubsprotocal"
)

func main() {
	srv := modbus.NewTCPServer()
	srv.LogMode(true)
	srv.AddNodes(
		modbus.NewNodeRegister(
			1,
			0, 10, 0, 10,
			0, 10, 0, 10),
		modbus.NewNodeRegister(
			2,
			0, 10, 0, 10,
			0, 10, 0, 10),
		modbus.NewNodeRegister(
			3,
			0, 10, 0, 10,
			0, 10, 0, 10))

	err := srv.ListenAndServe(":502")
	if err != nil {
		panic(err)
	}
}
```

### References

---

- [Modbus Specifications and Implementation Guides](http://www.modbus.org/specs.php)
- [goburrow](https://github.com/goburrow/modbus)


