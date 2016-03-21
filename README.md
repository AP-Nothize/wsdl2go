# wsdl2go

wsdl2go is a command line tool to generate [Go](https://golang.org) code
from [WSDL](https://en.wikipedia.org/wiki/Web_Services_Description_Language).

Download:

```
go get github.com/fiorix/wsdl2go
```

### Usage

Make sure you have gofmt under $GOROOT/bin, otherwise it'll fail.

```
wsdl2go < file.wsdl > hello.go
```

Use -i for files or URLs, and -o to specify an output file. WSDL
files that contain import tags are only processed after these
resources are downloaded. It tries automatically but might fail
due to authentication or bad SSL certificates. You can force it
anyway. YOLO.

Here's how to use the generated code: Let's say you generate the
Go code for the hello service, which provides an Echo method that
takes an EchoRequest and returns an EchoReply. To use it, you have
to create a SOAP client, then call the generated function Echo.

Example:

```
import (
	"/path/to/hello"

	"github.com/fiorix/wsdl2go/soap"
)

func main() {
	cli := &soap.Client{
		URL: "http://server",
		Namespace: hello.Namespace,
	}
	req := &hello.EchoRequest{Data: "echo"}
	reply, err := hello.Echo(cli, req)
	...
}
```

Only the **Document** style of SOAP is supported. If you're looking
for the RPC one, take another bite of your taco and move on. Soz.

### Status

Works for my needs, been tested with a few SOAP enterprise systems.
Not fully compliant to WSDL or SOAP specs.

Because of some [limitations](https://github.com/golang/go/issues/14407)
of XML namespaces in Go, there's only so much one can do to make
things like SOAP work properly. Although, the code generated by wsdl2go
might be sufficient for most systems.

Types supported:

- [x] int
- [x] long (int64)
- [x] float (float64)
- [x] double (float64)
- [x] boolean (bool)
- [x] string
- [x] hexBinary ([]byte)
- [x] base64Binary ([]byte)
- [x] date
- [x] time
- [x] dateTime
- [x] simpleType (w/ enum and validation)
- [x] complexType (struct)
- [x] complexContent (slices, embedded structs)
- [ ] faults
- [ ] decimal
- [ ] g{Day,Month,Year}...
- [ ] anyURI
- [ ] QName
- [ ] NOTATION

Date types are currently defined as strings, need to implement XML
Marshaler and Unmarshaler interfaces. The binary ones (hex and base64)
are also lacking marshal/unmarshal.

For simple types that have restrictions defined, such as an enumerated
list of possible values, we generate the validation function using reflect
to compare values. This and the entire API might change anytime,
be warned.
