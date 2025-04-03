# Hi there 👋 we are The Chiselers

- We are working on creating some hardware cores using [Chisel](https://www.chisel-lang.org/) HDL
- Additionally we have set up a simple development harness to create professional grade tools at zero cost and built it into one package. We support:
   - Generating Verilog with [Circt](https://circt.llvm.org/) & FIRRTL
   - Testing With [Treadle](https://github.com/chipsalliance/treadle) and [Verilator]
   - Synthesis with [Yosys](https://yosyshq.net/yosys/)
   - Timing Analysis with [OpenSTA](https://github.com/The-OpenROAD-Project/OpenSTA)
   - Formal Verification with [Chisel Formal](https://woset-workshop.github.io/PDFs/2021/a03.pdf) & [Z3](https://github.com/Z3Prover/z3)
   - Coverage with [scala-scoverage](https://github.com/scoverage/scalac-scoverage-plugin) for line coverage and our own custom Toggle Coverage Harness
      - See [Uart.scala](https://github.com/The-Chiselers/uart/blob/75516efe70c4a93c155d41343f3badf3a01ac8a0/src/main/scala/tech/rocksavage/chiselware/uart/hw/Uart.scala#L586C1-L586C31) and [UartTest.scala](https://github.com/The-Chiselers/uart/blob/75516efe70c4a93c155d41343f3badf3a01ac8a0/src/test/scala/tech/rocksavage/chiselware/uart/UartTest.scala#L407) for examples of how to use our toggle coverage harness   
   - Documentation and Publishing with LaTeX & Github Actions
   - And Finally Package Management with [Nix](https://nixos.org/)
 
## Development Harness Usage

Below are some examples of usage with our [Uart](https://github.com/The-Chiselers/uart) module being an example. Make sure to be in the root of the project for the following commands. 

**Note:** You will need the Nix package manager installed to use the commands below:

 - Windows: Recommend Using the [NixOS WSL](https://nix-community.github.io/NixOS-WSL/install.html) image
 - MacOS: [MacOS Nix Package Manager Setup](https://nixos.org/download/#nix-install-macos)
 - Other Linux Distributions: [Linux Nix Package Manager Setup](https://nixos.org/download/#nix-install-linux)

Once Nix is installed, enter our developement shell by running:
```sh
sh dev_shell.sh
```
or
```sh
nix develop --extra-experimental-features 'nix-command flakes' -c $SHELL
```

### Generate Verilog

```sh
make verilog
```
or
```sh
sbt "runMain tech.rocksavage.Main verilog --mode write --module tech.rocksavage.chiselware.uart.hw.Uart --config-class tech.rocksavage.chiselware.uart.UartConfig"
```

The output can be found in: `out/verilog`

### Run Tests

```sh
make test
```
or 
```sh
sbt test
```

### Run Coverage

```sh
make cov
```
or
```sh
sbt coverageOn test coverageReport
```
- Make sure that coverage is enabled for toggle coverage, our [Uart Template](https://github.com/The-Chiselers/uart/blob/75516efe70c4a93c155d41343f3badf3a01ac8a0/src/main/scala/tech/rocksavage/chiselware/uart/hw/Uart.scala#L586) kept it gated by a parameter

The output can be found in: `out/cov`

### Run Synthesis

```sh
make synth
```
or
```sh
sbt "runMain tech.rocksavage.Main synth --module tech.rocksavage.chiselware.uart.hw.Uart --config-class tech.rocksavage.chiselware.uart.UartConfig --techlib synth/stdcells.lib"
```

The output can be found in: `out/synth`

### Run Timing Analysis

```sh
make sta
```
or
```sh
sbt "runMain tech.rocksavage.Main sta --module tech.rocksavage.chiselware.uart.hw.Uart --config-class tech.rocksavage.chiselware.uart.UartConfig --techlib synth/stdcells.lib --clock-period 5.0"
```

The output can be found in: `out/sta`

## Libraries

We've created a few libraries to make Hardware Development easier:
- [Address Decoder](https://github.com/The-Chiselers/addrdecode)
- [Register Map](https://github.com/The-Chiselers/registermap)

Both of these modules together allow for simple creation of an APB interface to a module. The Uart repo provides a good example of the capabilities of both of these together.
- [Address Decoder Example](https://github.com/The-Chiselers/uart/blob/75516efe70c4a93c155d41343f3badf3a01ac8a0/src/main/scala/tech/rocksavage/chiselware/uart/hw/Uart.scala#L353)
- [Register Map Example](https://github.com/The-Chiselers/uart/blob/75516efe70c4a93c155d41343f3badf3a01ac8a0/src/main/scala/tech/rocksavage/chiselware/uart/hw/Uart.scala#L35)


## Development Harness Modification

Above are many commands that look like:
```sh
sbt "runMain tech.rocksavage.Main verilog --mode write --module tech.rocksavage.chiselware.uart.hw.Uart --config-class tech.rocksavage.chiselware.uart.UartConfig"
```

If you want to point our harness at your own module.
1. Create your own repo from the Uart Template Repo
2. In the Makefile, Change `--module tech.rocksavage.chiselware.uart.hw.Uart` to the class path of your module.
   1. For instance if your new module was an Aes module in the following package: `package abc.def.ghi`, this might be an example configuration:
```scala
// Aes.scala
package abc.def.ghi

import chisel3._
import chisel3.util._
import tech.rocksavage.chiselware.addrdecode.{AddrDecode, AddrDecodeError}
import tech.rocksavage.chiselware.addressable.RegisterMap
import tech.rocksavage.chiselware.apb.{ApbBundle, ApbParams}
import tech.rocksavage.test.TestUtils.coverAll

class Aes(val aesParams: AesParams, formal: Boolean) extends Module {
    val io = IO(new Bundle {
        val apb        = new ApbBundle(ApbParams(dataWidth, addressWidth))
        val in         = Input(Bool())
        val out        = Output(Bool())
    })
...
}
```
   3. The the new module path would be: `--module abc.def.ghi.Aes`
3. Create a Scala file with the default parameters you want to generate verilog for, synthesize, and run timing analysis on.
   1. It might look like the following for the above Aes module:
```scala
package abc.def.ghi

import tech.rocksavage.traits.ModuleConfig

class UartConfig extends ModuleConfig {
    override def getDefaultConfigs: Map[String, Any] = Map(
      "config1" -> Seq(
        AesParams(
          addressWidth = 32,
          dataWidth = 32,
          wordWidth = 8,
          otherParam = 1
        ),
        false
      ),
      "config2" -> Seq(
        UartParams(
          addressWidth = 32,
          dataWidth = 32,
          wordWidth = 8,
          otherParam = 2
        ),
        false
      ),
      "config3" -> Seq(
        UartParams(
          addressWidth = 32,
          dataWidth = 32,
          wordWidth = 8,
          otherParam = 3
        ),
        false
      )
    )
}
```
4. Modify the Makefile to point to the new config class: `--config-class tech.rocksavage.chiselware.uart.UartConfig`

Thats it, then your project should have all of the tools set up and ready to use.
