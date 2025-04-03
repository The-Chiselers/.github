# Hi there 👋 we are The Chiselers

- We are working on creating some hardware cores using [Chisel](https://www.chisel-lang.org/) HDL
- Additionally we have set up a simple development harness to create professional grade tools at zero cost and built it into one package. We support:
   - Generating Verilog with [Circt](https://circt.llvm.org/) & FIRRTL
   - Testing With [Treadle](https://github.com/chipsalliance/treadle) and [Verilator](https://verilator.org/guide/latest/)
   - Synthesis with [Yosys](https://yosyshq.net/yosys/)
   - Timing Analysis with [OpenSTA](https://github.com/The-OpenROAD-Project/OpenSTA)
   - Formal Verification with [Chisel Formal](https://woset-workshop.github.io/PDFs/2021/a03.pdf) & [Z3](https://github.com/Z3Prover/z3)
   - Coverage with [scala-scoverage](https://github.com/scoverage/scalac-scoverage-plugin) for line coverage and our own custom Toggle Coverage Harness
      - See [Uart.scala](https://github.com/The-Chiselers/uart/blob/75516efe70c4a93c155d41343f3badf3a01ac8a0/src/main/scala/tech/rocksavage/chiselware/uart/hw/Uart.scala#L586C1-L586C31) and [UartTest.scala](https://github.com/The-Chiselers/uart/blob/75516efe70c4a93c155d41343f3badf3a01ac8a0/src/test/scala/tech/rocksavage/chiselware/uart/UartTest.scala#L407) for examples of how to use our toggle coverage harness   
   - Documentation and Publishing with LaTeX & Github Actions
   - And Finally Package Management with [Nix](https://nixos.org/)
 
## More Information

- See our document on How to use our tools: [Usage](Usage.md)
