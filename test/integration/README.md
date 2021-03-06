# Integration Tests

This document contains information about NSM-NSE integration tests for VPP/VPP-Agent.

Table of contents:
- [Running Tests](#running-tests)
- [Writing Tests](#writing-tests)

The integration tests are implemented as Go tests and connect to Docker client to manage containers for testing. 

Tests follow a pattern that is very similar to the [e2e tests in vpp-agent](https://docs.ligato.io/en/latest/testing/end-to-end-tests).

## Running Tests

### Prerequisites

- [Docker 18.04+](https://docs.docker.com/engine/install)
- [Go 1.14+](https://golang.org/doc/install)

The easiest way to run the entire integration test suite is to execute `test/integration/run.sh` script.

Tests are executed via [gotestsum](https://github.com/gotestyourself/gotestsum) in a Docker container.

### Run With Latest VPP-Agent

```sh
# Run using ligato/vpp-agent:latest image
./test/integration/run.sh
```

### Run With Specific VPP-Agent Version

Before running the tests, set the `VPP_AGENT` variable to the image you want to test against.

```sh
# Run with VPP 20.09
VPP_AGENT=ligato/vpp-agent:v3.2.0 ./test/integration/run.sh
```

### Run Specific Test Case

This script supports the addition of extra arguments for the test run.

```sh
# Run test cases containing AfPacket in name
./test/integration/run.sh -test.run=AfPacketVNF
```

### Run with standard verbose mode

To use different output format use env var `GOTESTSUM_FORMAT`.

```sh
# Run tests in verbose mode
GOTESTSUM_FORMAT=standard-verbose ./test/integration/run.sh
```

More output format can be found here: https://github.com/gotestyourself/gotestsum#output-format

## Writing Tests

When writing a new test case, call `Setup()` as first thing in each test. The `Setup()` will take care of setting up testing environment. 

```go
func TestSomething(t *testing.T) {
    // Setup test environment
    test := Setup(t)
    
    // testing..
}
```

The `Setup()` returns `TestContext` that provides methods to manage resources used in the test, run various actions or assert expected conditions.

```go
func TestSomething(t *testing.T) {
    test := Setup(t)
    
    // Start VPP-Agent container
    agent := test.SetupVPPAgent("agent")
    
    // Start Microservice container
    ms := test.StartMicroservice("microservice")
    
    // testing..
}
```

### Test Examples

Check out test `TestInterfaceAfPacket` in the [`010_interface_test.go`](010_interface_test.go) for simple test example that uses one agent and one microservice. For more advanced example, see test `TestInterfaceAfPacketVNF` in the [`010_interface_test.go`](010_interface_test.go) which runs multi-agent setup.

Here's diagram showing topology of test case `TestInterfaceAfPacketVNF`.

![](./test-topology-diagram1.png)

Link to diagram: https://drive.google.com/file/d/11kjtv62tFUliipl14w_CNBZtUL1Klj5Y/view?usp=sharing

The Gomega test framework is also ready to use for each test case, but not mandatory to use. See [Gomega online documentation](https://onsi.github.io/gomega/#making-assertions) for more info.

## References

- vpp-agent API: https://github.com/ligato/vpp-agent/tree/master/proto
  - VPP: https://github.com/ligato/vpp-agent/blob/master/proto/ligato/vpp/vpp.proto
  - Linux: https://github.com/ligato/vpp-agent/blob/master/proto/ligato/linux/linux.proto
- Ligato documentation: https://docs.ligato.io/en/latest/
