---
layout: page
title: C# - gRPC Demo
parent: C#
---

# gRPC Demo


## Intro

gRPC (**R**emote **P**rocedure **C**alls) is a technique to transfer data between machines, e.g. a client and a server. Compared to JSON APIs, it does this not with human readably json, but with a binary stream. 

gRPC can be used for:

* Low latency, highly scalable, distributed systems.
* Developing mobile clients who are communicating to a cloud server.
* Designing a new protocol that needs to be accurate, efficient and language independent.
* Layered design to enable extension e.g. authentication, load balancing, logging and monitoring etc.

*Check the following link for more details: [gRPC Project](https://grpc.io/) or [gRPC GitHub](https://github.com/grpc)*

Possible scenarios therefore would be:

* IoT Devices
* Microservices for quick communication besides queues
* small APIs

There are negative aspects as well: Creating and handling the proto files is quite complicated sometimes and can cause troubles while implementing. The necessity to rebuild the project to update the autogenerated files can easily be forgotten.
The proto files are in their own syntax and you have to get used to writing them not in C# types and formatting.
Not every client can consume gRPC communication, but restful APIs can be consumed way more often. Debugging and implementing gRPC can be more complicated, too, where with restful APIs you often just need to call the endpoint with a Browser to check if it works as intended.


# Proto Files

Client and Server have to agree about the data transfer with some sort of contract, the so called proto buffers or proto files. After creating a VS project you have the Protos folder with the used proto files for the API.

[![greet proto file example](/assets/images/coding/csharp/grpc-demo/greet-proto.png)](/assets/images/coding/csharp/grpc-demo/greet-proto.png)

The `syntax` describes the used schema for the file. The option csharp_namespace and package define the scope for using this file.


## Proto Messages and Service

The messages can be compared to model classes. These hold the data as properties. The name property in line 15 is set to the first order. Ordering is important in gRPC to find the data on deserializing the binary stream in a correct way.
Notice, that the property names are written lowercase in the proto files! This might come from the JavaScript background. In the C# implementation of the Methods, these get written in Pascal Case again, so with a starting upper case letter.

Messages have a request part and a reply part. The request is used to ask the server for something and deliver some data for the request, like the name. The reply then gets sent from the server, transporting a message for the processed request.

The service defines the remote procedure call and what is to be sent (request) and what will be returned (reply).


# C# Service

The GreeterService as C# class inherits from the `Greeter.GreeterBase` class. This class is a auto generated class which resides in the obj folder and comes from the proto package defined Greeter service definition in the proto file. The `Greeter.GreeterBase` class file gets autogenerated on building the project and takes care of all the wiring and work behind the scenes. This can be helpful because we don't have to write this code ourselves. But it can also be problematic if we have a bug in it and have to dig into this code.

[![service-base-comparison](/assets/images/coding/csharp/grpc-demo/service-class-base-class-comparison.png)](/assets/images/coding/csharp/grpc-demo/service-class-base-class-comparison.png)


The GreeterService has a constructor with Dependency Injection ready parameters for injecting a logger. 

The SayHello method overrides the base class method, which would just throw an unimplemented exception. The `HelloRequest request` as parameter holds the request object and its properties be accessed with `request.Name`.


# Client 

Starting the server project spins up a webserver with a address to call for the gRPC methods. This cannot be accessed through browser, like with restful APIs where you receive a json answer calling an endpoint. With gRPC we must use a gRPC client for this. The client can be a console application, or WPF or anything else, which is able to talk to gRPC.

To enable the gRPC abilities in the client, we need to install some NuGet packages:

* Google.Protobuf to use the proto buffer files
* Grpc.Net.Client to talk the gRPC protocol and use a client
* Grpc.Tools to have VS tooling

Now we need to share the proto buffer files to both projects, the server and the client. These have to be the same so that both know how to talk to each other. We add a folder named 'Protos' and add the `greet.proto` file in there by copying it from the server.
Then we need to adjust its settings to be 'client only' in the file properties:

[![service-base-comparison](/assets/images/coding/csharp/grpc-demo/proto-file-client-settings.png)](/assets/images/coding/csharp/grpc-demo/proto-file-client-settings.png)

After adding the proro buffer file, we have to rebuild the project to create the base class files in the obj folder again.

Now we can start to implement the client:

```csharp
public class Program
{
    private static async Task Main(string[] args)
    {
        Console.WriteLine("Starting the gRPC Client ...");

        // Server address http://localhost:5124 or https://localhost:7124
        var channel = GrpcChannel.ForAddress("http://localhost:5124");
        var client = new Greeter.GreeterClient(channel);

        var input = new HelloRequest { Name = "David" };
        var reply = await client.SayHelloAsync(input);

        Console.WriteLine($"gRPC Server Response: {reply.Message}");

        Console.WriteLine("gRPC Client finished.");
    }
}
```


# Start Server and Client

To test this, we have to start both the server and the client. We can do this by setting up the solution starting projects:

[![service-base-comparison](/assets/images/coding/csharp/grpc-demo/solution-starting-projects-context-menu.png)](/assets/images/coding/csharp/grpc-demo/solution-starting-projects-context-menu.png)

Move the server up to be started first and set both to 'start'.

[![service-base-comparison](/assets/images/coding/csharp/grpc-demo/solution-starting-projects-settings.png)](/assets/images/coding/csharp/grpc-demo/solution-starting-projects-settings.png)

[![service-base-comparison](/assets/images/coding/csharp/grpc-demo/solution-starting-projects-example.png)](/assets/images/coding/csharp/grpc-demo/solution-starting-projects-example.png)


# Add Customer example

To recreate a full service and proto buffer file from scratch, we create an additional customer service example. Starting with the proto file, adding it via VS GUI had a bug in my setup. I was not able to set the gRPC specific file settings, like server only. A solution was to just copy the already existing greet.proto file and change its name and content to the new customer.proto file:


## customers.proto

```csharp
syntax = "proto3";

option csharp_namespace = "GrpcServer";

service Customer {
	rpc GetCustomerInfo (CustomerLookupModel) returns (CustomerModel);
}

message CustomerLookupModel {
	int32 userId = 1;
}

message CustomerModel{
	string firstName = 1;
	string lastName = 2;
	string emailAddress = 3;
	bool isAlieve = 4;
	int32 age = 5;
}
```

Proto files do not support the C# types, but have their own types, like `int32`. `string`, `bool`, `float` and some others are the same. But to achieve something more complex like DateTime has to be researched when needed.


## CustomersService.cs

The CustomersService receives requests from the client and creates a CustomerModel for the reply. It overrides the generated method for GetCustomerInfo with a async task method. We simulate some DB lookup and return dummy data as `Task.FromResult(output);`:

```csharp
using Grpc.Core;

namespace GrpcServer.Services;

public class CustomersService : Customer.CustomerBase
{
    private readonly ILogger<CustomersService> _logger;

    public CustomersService(ILogger<CustomersService> logger)
    {
        _logger = logger;
    }

    public override Task<CustomerModel> GetCustomerInfo(CustomerLookupModel request, ServerCallContext context)
    {
        CustomerModel output = new CustomerModel();

        // Simulate DB access and return some dummy data
        if (request.UserId == 1)
        {
            output.FirstName = "David";
            output.LastName = "Halletz";
        }
        else if (request.UserId == 2)
        {
            // ...
        }

        return Task.FromResult(output);
    }
}
```


## Mapping the new endpoint

The new endpoint has to be registered or mapped somewhere to be called from clients. This happens in the Startup.cs or, with newer versions of .NET, in the Program.cs. See the line with `app.MapGrpcService<CustomersService>();`:

```csharp
using GrpcServer.Services;

namespace GrpcServer;

public class Program
{
    public static void Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);

        // Add services to the container.
        builder.Services.AddGrpc();

        var app = builder.Build();

        // Configure the HTTP request pipeline.
        app.MapGrpcService<GreeterService>();
        app.MapGrpcService<CustomersService>();

        app.MapGet("/", () => "Communication with gRPC endpoints must be made through a gRPC client. To learn how to create a client, visit: https://go.microsoft.com/fwlink/?linkid=2086909");

        app.Run();
    }
}
```


## Extending the Client

The client needs to have the same proto file again. with copying it to its Protos folder we have to set the Client Only setting again. After that we can now implement some code in the Program.cs file to have some further calls.

The channel gets reused and the customerClient variable hast to be defined for the `Customer.CustomerClient` and the channel. The CustomerModel with UserId 1 gets explicitly requested to get the dummy data for this call.

```csharp
using Grpc.Net.Client;
using GrpcServer;

namespace GrpcClient;

public class Program
{
    private static async Task Main(string[] args)
    {
        Console.WriteLine("Starting the gRPC Client ...");

        // Server address http://localhost:5124 or https://localhost:7124
        var channel = GrpcChannel.ForAddress("http://localhost:5124");
        var client = new Greeter.GreeterClient(channel);

        var helloRequest = new HelloRequest { Name = "David" };
        var reply = await client.SayHelloAsync(helloRequest);

        Console.WriteLine($"gRPC Server Response: {reply.Message}");

        var customerClient = new Customer.CustomerClient(channel);
        var customerRequest = new CustomerLookupModel { UserId = 1 };
        var customer = await customerClient.GetCustomerInfoAsync(customerRequest);

        Console.WriteLine($"gRPC Server Response: {customer.FirstName} {customer.LastName}");

        Console.WriteLine("gRPC Client finished.");
        Console.ReadKey();
    }
}
```

[![service-base-comparison](/assets/images/coding/csharp/grpc-demo/solution-starting-projects-customers-example.png)](/assets/images/coding/csharp/grpc-demo/solution-starting-projects-customers-example.png)


# Advanced services

With more advanced services you can implement something like calling the server to get all new customers as response. This needs an empty request to be sent to the server. One way of doing this is to create an empty message in the customers.proto file and a new rpc definition with an outgoing stream. Calling the new `GetNewCustomers` will then request with an empty parameter for all new customers and will return a stream of all new customers to the client. This is like chunking the response to the client.

```csharp
service Customer {
	rpc GetCustomerInfo (CustomerLookupModel) returns (CustomerModel);
	rpc GetNewCustomers (NewCustomerRequest) returns (stream CustomerModel);
}

message NewCustomerRequest {

}
```

Now the CustomersService has to get the overriding method to return this streamed data to the client:

```csharp
public override async Task GetNewCustomers(
        NewCustomerRequest request,
        IServerStreamWriter<CustomerModel> responseStream,
        ServerCallContext context)
    {
        List<CustomerModel> customers = new List<CustomerModel>
        {
            new CustomerModel
            {
                FirstName = "Tim",
                LastName = "Cors",
                EmailAddress = "tim@test.com",
                Age = 41,
                IsAlieve = true
            },
            // ...
        };

        foreach (var cust in customers)
        {
            // this writes the single customer to the stream and returns the cust to the client
            await responseStream.WriteAsync(cust);
        }
    }
```

The Client hast to get the new customers.proto file content as well. After rebuilding the solution we can start it and look at the result:

[![service-base-comparison](/assets/images/coding/csharp/grpc-demo/solution-starting-projects-advanced-example.png)](/assets/images/coding/csharp/grpc-demo/solution-starting-projects-advanced-example.png)