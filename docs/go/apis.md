---
sidebar_position: 3
---

# Compiler APIs

This guide provides a comprehensive overview of the core APIs offered by the compiler.

## Root Package

To integrate the compiler into your Go project, include the following import statement in your code:

```go
import "github.com/PolyhedraZK/ExpanderCompilerCollection/ecgo"
```

### Compile Function

The `ecgo.Compile` function serves as the primary interface to the compiler. It accepts a `frontend.Circuit` as input and returns a `CompileResult`.

### CompileResult Structure

The `ecgo.CompileResult` structure encapsulates the compilation process outputs, including both the layered circuit and the intermediate representation (IR).

The `CompileResult` offers three methods to access the data:

1. `GetCircuitIr()`: Retrieves the IR.
2. `GetLayeredCircuit()`: Fetches the layered circuit.
3. `GetInputSolver()`: Obtains the compiled witness solver of the circuit.

### Builder API

The `ecgo.API` serves as the interface for the builder API. It extends the `frontend.API` interface from gnark, providing additional features such as support for sub-circuits and utility functions.

## Sub-Circuit API

Sub-circuit functionality is enabled through the following methods:

```go
type SubCircuitSimpleFunc func(api frontend.API, input []frontend.Variable) []frontend.Variable
type SubCircuitFunc interface{}

type SubCircuitAPI interface {
	MemorizedSimpleCall(SubCircuitSimpleFunc, []frontend.Variable) []frontend.Variable
	MemorizedCall(SubCircuitFunc, ...interface{}) interface{}
}
```

`SubCircuitFunc` is designed to accommodate any function that takes simple types and `frontend.Variable` as inputs and returns `frontend.Variable` as output.

For example, a function with the signature `func(frontend.API, int, uint8, [][]frontend.Variable, string, ...[]frontend.Variable) [][][]frontend.Variable` would be considered a valid `SubCircuitFunc`.

A key requirement for `SubCircuitFunc` is determinism: for any given set of identical simple-type inputs, the output must always be the same. This is crucial because the compiler memorizes the wiring of sub-circuit calls and reuses these memorized results for identical inputs, enhancing efficiency.

## Additional APIs

```go
type API interface {
	Output(frontend.Variable)
	GetRandomValue() frontend.Variable
	CustomGate(gateType uint64, inputs ...frontend.Variable) frontend.Variable
}
```

- `Output`: Appends a variable to the circuit's output. Typically used to designate certain variables as public outputs of the circuit.
- `GetRandomValue`: Retrieves a random value directly, offering a more efficient approach than generating pseudo-random numbers using a hash function. This direct access to random numbers is facilitated by the Libra proving process.
- `CustomGate`: Similar to Gnark's `NewHint`, this method essentially calls a hint function to compute a result. In the resulting layered circuit, it will be compiled into a custom gate of the specified gate type. Unlike `NewHint`, it requires pre-registering the hint function and other parameters. For specific details, refer to this example.

Several other APIs exist for compatibility with the old pure Golang Expander Compiler, but they are currently no-op.

## Builder Extensions

### Memorized Function Wrappers

```go
func MemorizedSimpleFunc(f SubCircuitSimpleFunc) SubCircuitSimpleFunc
func MemorizedVoidFunc(f SubCircuitFunc) func(frontend.API, ...interface{})
func Memorized0DFunc(f SubCircuitFunc) func(frontend.API, ...interface{}) frontend.Variable
func Memorized1DFunc(f SubCircuitFunc) func(frontend.API, ...interface{}) []frontend.Variable
func Memorized2DFunc(f SubCircuitFunc) func(frontend.API, ...interface{}) [][]frontend.Variable
func Memorized3DFunc(f SubCircuitFunc) func(frontend.API, ...interface{}) [][][]frontend.Variable
```

These function wrappers serve as syntactic conveniences for `MemorizedCall`, streamlining its usage. For instance, the following invocations are functionally equivalent:

```go
output := api.MemorizedSimpleCall(f, input)

memorizedF := builder.MemorizedSimpleFunc(f)
output := memorizedF(api, input)
```