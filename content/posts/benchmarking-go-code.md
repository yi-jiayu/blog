---
title: "Benchmarking Go code"
subtitle: "Calculating statistical significance with benchstat"
date: 2019-05-06T09:55:06+08:00
---

I recently attended GopherCon Singapore 2019, and one of the talks was about optimising Go code, by
Daniel Martí. I managed to find a set of his slides from a previous conference that he gave the same
talk at:

{{< linkpreview title="Optimizing Go code without a blindfold"
description="Daniel Martí at dotGo 2019"
url="https://www.dotconferences.com/2019/03/daniel-marti-optimizing-go-code-without-a-blindfold" >}}

My main takeaway from his talk the usage of
[`benchstat`](https://godoc.org/golang.org/x/perf/cmd/benchstat) for computing statistics from
multiple runs of `go test -bench`, which helps to remove noise and identify statistically
significant results:

{{< linkpreview title="Command benchstat"
description="Benchstat computes and compares statistics about benchmarks."
url="https://godoc.org/golang.org/x/perf/cmd/benchstat" >}}

This weekend, I had the chance to try it out while working on my own [Chip-8 emulator implementation
in Go](https://github.com/yi-jiayu/chip8), which I was inspired to attempt after seeing [my
colleague's implementation in Haskell](https://github.com/gableh/chip8).

## The code

The code I was trying to benchmark was an implementation of the [`ADD Vx,
Vy`](http://devernay.free.fr/hacks/chip8/C8TECH10.HTM#8xy4) instruction, which also sets a register
if the addition produced a carry out. My original implementation simply did a comparison to
determine if the carry bit should be set:

```go
// 8xy4 - ADD Vx, Vy
// Set Vx = Vx + Vy, set VF = carry.
//
// The values of Vx and Vy are added together. If the result is greater than 8 bits (i.e., > 255,) VF is set to 1,
// otherwise 0. Only the lowest 8 bits of the result are kept, and stored in Vx.
func ADD_8xy4(ip *Interpreter, instr instruction) {
	x := instr.x()
	sum := uint16(ip.registers[x]) + uint16(ip.registers[instr.y()])
	ip.registers[x] = uint8(sum)
	if sum > 255 {
		ip.registers[VF] = 1
	} else {
		ip.registers[VF] = 0
	}
	ip.pc++
}
```

Some time later, I discovered the existence of the [`math/bits`](https://golang.org/pkg/math/bits/)
package with optimised functions for bit manipulation, [released in Go
1.9](https://golang.org/doc/go1.9#math-bits). It includes an [addition function which returns the carry out](https://golang.org/pkg/math/bits/#Add):

```go
func Add(x, y, carry uint) (sum, carryOut uint)
```

I then rewrote my `ADD Vx, Vy` implementation using this instead:

```go
func ADD_8xy4(ip *Interpreter, instr instruction) {
	x := instr.x()
	sum, carry := bits.Add(uint(ip.registers[x]), uint(ip.registers[instr.y()]), 0)
	ip.registers[x] = uint8(sum)
	ip.registers[VF] = uint8(carry)
	ip.pc++
}
```

But in order to determine if the new implementation was actually faster, we need to benchmark!

## The benchmark

I wrote the following benchmark to exercise adding all possible combinations of values for registers
Vx and Vy:

```go
func BenchmarkADD_8xy4(b *testing.B) {
	ip := new(Interpreter)
	instr := instruction{0x01, 0x10}
	for i := 0; i < b.N; i++ {
		for x := uint8(0); x < 0xFF; x++ {
			for y := uint8(0); y < 0xFF; y++ {
				ip.registers[0] = x
				ip.registers[1] = y
				ADD_8xy4(ip, instr)
			}
		}
	}
}
```

## Benchmarking

When I started, `benchstat` wasn't installed, so I `cd`ed to my home directory to `go get` it
globally in order to not add it to my project's `go.mod`:

```console
$ benchstat
-bash: benchstat: No such file or directory
$ cd
$ go get golang.org/x/perf/cmd/benchstat
```

Following that, I closed my CPU-hungry web browser, then ran `go test -bench` for the old code 20
times:

```console
$ for i in {1..20}; do go test -bench=^BenchmarkADD_8xy4$ >> old.txt; done
```

And then another 20 times for the new code:

```console
$ for i in {1..20}; do go test -bench=^BenchmarkADD_8xy4$ >> new.txt; done
```

(The first time I did this, I forgot to change the output filename, and ended up overwriting my
results from benchmarking the original code, and had to run it again 🤦)

## The results

Finally, we can run `benchstat` on the concatenated `go test -bench` outputs:

```console
$ benchstat new.txt old.txt 
name         old time/op  new time/op  delta
ADD_8xy4-12  96.0µs ± 2%  82.6µs ± 2%  -13.95%  (p=0.000 n=40+41)
```

Nice! So using the `Add` function from the `math/bits` package provided a 14% speedup, with a
p-value of 0!

## Further work

Besides benchmarking, I also learnt about using compiler flags to show information such as whether
variables are allocated on the stack or the heap, whether bounds checks are elided or whether
functions are inlined at GopherCon Singapore 2019.

I'm looking forward to trying these out as well, but maybe I should get something working before
diving into optimisation 😅
