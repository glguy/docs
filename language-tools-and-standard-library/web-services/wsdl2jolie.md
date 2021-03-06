# wsdl2jolie

_wsdl2jolie_ \(whose executable is installed by default in Jolie standard trunk\) is a tool that takes a URL to a WSDL document and automatically downloads all the related files \(e.g., referred XML schemas\), parses them and outputs the corresponding Jolie port/interface/data type definitions.

## The syntax

The syntax of wsdl2jolie follows:

```text
wsdl2jolie wsdl_uri [output filename]
```

`wdsl_uri` can be a URL or a file path \(in case of local usage\).

As an output the tool returns a set of service declarations \(in Jolie\) needed for invoking the web service. The output can be automatically saved into a file by specifying the optional parameter `[output filename]`.

## wsdl2jolie example

Let us consider an example of a WSDL document for a service that provides some basic mathematical operations, the WSDL URL is `http://www.dneonline.com/calculator.asmx?WSDL`.

Reading the raw XML is not so easy, or at least requires some time.

If we execute the command `wsdl2jolie http://www.dneonline.com/calculator.asmx?WSDL` our output will be

```text
type NOTATIONType:any

type Add:void {
    .intB:int
    .intA:int
}

type Divide:void {
    .intB:int
    .intA:int
}

type MultiplyResponse:void {
    .MultiplyResult:int
}

type DivideResponse:void {
    .DivideResult:int
}

type SubtractResponse:void {
    .SubtractResult:int
}

type Multiply:void {
    .intB:int
    .intA:int
}

type Subtract:void {
    .intB:int
    .intA:int
}

type AddResponse:void {
    .AddResult:int
}

interface CalculatorSoap {
RequestResponse:
    Add(Add)(AddResponse),
    Subtract(Subtract)(SubtractResponse),
    Multiply(Multiply)(MultiplyResponse),
    Divide(Divide)(DivideResponse)
}

outputPort CalculatorSoap12 {
Location: "socket://localhost:80/"
Protocol: soap
Interfaces: CalculatorSoap
}

outputPort CalculatorSoap {
Location: "socket://www.dneonline.com:80/calculator.asmx"
Protocol: soap {
    .wsdl = "http://www.dneonline.com/calculator.asmx?WSDL";
    .wsdl.port = "CalculatorSoap"
}
Interfaces: CalculatorSoap
}
```

which is the Jolie equivalent of the WSDL document. Those `.wsdl` and `.wsdl.port` parameters are improvement to the SOAP protocol: when the output port is used for the first time, Jolie will read the WSDL document for processing information about the correct configuration for interacting with the service instead of forcing the user to manually insert it.

Once our interface is created, we can store it into a file, e.g., `CalculatorInterface.iol`, and use the output ports we discovered from Jolie code. As in the following:

```text
include "CalculatorInterface.iol"
include "console.iol"

main
{
    request.intA = 10;
    request.intB = 11;
    Add@CalculatorSoap( request )( response );
    println@Console( response.AddResult )()
}
```

Our little program will output `21`.

Remarkably, wsdl2jolie has two benefits: it acts as a useful tool that creates the typed interface of a Web Service from Jolie and creates a more human-readable form of a WSDL document \(i.e., its Jolie form\).

## The generated document

wdsl2jolie creates a document which contains:

* the types contained into \(or referred by\) the WSDL;
* the Jolie interface with all the operation declarations;
* the Jolie outputPort ports needed for the Web Service invocation.

### Mapping

In the following table we show the mapping between WSDL elements and Jolie elements:

| WSDL | Jolie |
| :--- | :--- |
| `<types>` | `type` |
| `<messages>` | `type` |
| `<portType>` | `interface` |
| `<binding>` | `outputPort:Protocol` |
| `<service:port>` | `outputPort` |

## SOAP outputPort

The SOAP outputPorts are generated with two parameters:

* `wsdl`, which sets the location of the WSDL document;
* `wsdl.port`, which sets the WSDL port related to the current outputPort.

Plus, another parameter can be added in order to display debug messages, which is `debug`; if set to 1, all the SOAP messages of the current outputPort are displayed on the standard output.

The `wsdl` and `wsdl.port` parameters are needed for formatting the messages to and from the web service in conformance with the WSDL document.

## Jolie Metaservice

Another feature that derives from the improvement of the SOAP protocol is that now Jolie standard library _MetaService_ can act as a transparent bridge between Web Services.

Once set the `addRedirection` operation with the right protocol configuration \(e.g., the `.wsdl` and `.wsdl.port` parameters\), MetaService automatically downloads the WSDL document - which is automatically cached -, and make it callable by clients.

Hence, it becomes really easy to use libraries such as QtJolie which requires only the location of the WSDL document to enable a client to call the Web Service of interest.

Plus, using wsdl2jolie combined with other tools, such as jolie2plasma, enables to use the aforementioned Jolie intermediate representation for transforming a Web Service interface definition into one compatible with a \(KDE\) Plasma::Service XML. In the same way, C++ generators can be written for QtJolie, introducing ease and type-safeness to Web Services invocations.

So far not all the WSDL and SOAP features are supported, which can raise compatibility problems when using them:

* SOAP 1.2, currently NOT supported;
* XML Schema Extended types, currently NOT supported;
* HTTP GET and HTTP POST, currently HALF supported as Web Service calls.

