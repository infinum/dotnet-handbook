We follow RESTFUL conventions in order to have a consistency between client and server communication. We are always using JSON as standard way of communicating for making the request and getting the response.

Here are some guidelines for the best practices.



### Routes

Standard route format, for an endpoint that returns customers information, should be *`api/v1/customers`*.  We achieve this by using *`Route`* attribute and .NET routing keywords such as *`{{version:apiVersion}}`*, *`[controller]`* etc.

Default route should be set ether on controller level or on base controller if controllers are inheriting it.



```c#
	[ApiController]
    [Route("api/v{version:apiVersion}/[controller]")]
    public abstract class BaseApiController : ControllerBase
    {
        
    }
```



### Versioning

In order to provide long time support and migration, it is recommended to use versioning. For details of API versioning, please refer to [Versioning](Versioning) chapter.



### Controllers

API controllers should be used to separate actions by entity. Usually controllers are able to return array of entities, therefore they should be named in plural. Giving a meaningful and consistent names to controllers will result in cleaner code and cleaner routes. The name should be followed by suffix Controller as it is a keyword in .NET that is handled in routing.

``` c#
[ApiVersion("1.0")]
public class CustomersController: BaseApiController
{
    
}
```

In the given example, we are using *`BaseApiController`* inheritance to inherit the default route. Any routes added to controller or controller methods will be prefixed by route defined in *`BaseApiController`*. The route for this controller will be *`api/v1/customers`*.



### Methods

In .NET there are attributes for each RESTFUL method. Most common that are used are *`GET`*, *`POST`*, *`PUT`* and *`DELETE`*. Methods without a route will take a default route from the controller, but we can also add a route to the method it self.

#### GET

Get is a method reserved for getting information about a resource. Get shouldn't have any side effects, meaning any number of calls should produce a same set of results.



```c#
	[HttpGet]
    public async Task<IActionResult> Get()
    {
       var customers = await _customerServices.GetAll();
       return Ok(customers);
    }
```

Get is also used to get information about a single resource.



```c#
    [HttpGet("{customerId}")]
    public async Task<IActionResult> Get(Guid customerId)
    {
       var customers = await _customerServices.Get(customerId);
       return Ok(customers);
    }
```


#### Post

Post is a method reserved for creating a new resource.

```c#
	[HttpPost]
    public async Task<IActionResult> Post(Customer customer)
    {
       var customer = await _customerServices.Create(customer);
       return Ok(customer);
    }
```

#### Put

Put is a method reserved for creating or updating a resource. Put requires a complete information about the resource. If a resource already exist, it is updated, if not than the resource is created.

```c#
	[HttpPut("{customerId}")]
    public async Task<IActionResult> Put(Guid customerId, Customer customer)
    {
       var customer = await _customerServices.CreateOrUpdate(customerId, customer);
       return Ok(customer);
    }
```

#### Patch

Patch is a method reserved for updating a resource. Unlike Put, Patch method updates only properties provided in the request and cannot create a new resource.

```c#
    [HttpPatch("{customerId}")]
    public async Task<IActionResult> Patch(Guid customerId, Customer customer)
    {
       var customer = await _customerServices.Update(customerId, customer);
       return Ok(customer);
    }
```

#### Delete

Delete is a method reserved for deleting a resource.

```c#
	[HttpDelete("{customerId}")]
    public async Task<IActionResult> Delete(Guid customerId)
    {
       var customers = await _customerServices.Delete(customerId);
       return Ok();
    }
```



### Method arguments

A controller method can get its arguments in three ways. Through route, body of the request and/or through URL parameters. Note that all arguments in .NET can be value types, like Boolean or number, or reference types, like GUID or other classes. The mapping between the HTTP request and the argument is handled automatically.

#### Route parameter argument

Route parameter is a part of the route and should be used to access a specific resource.

```c#
	[HttpGet("{customerId}")]
    public async Task<IActionResult> Get(Guid customerId)
    {
       var customer = await _customerServices.Get(customerId);
       return Ok(customer);
    }
```

The route to this endpoint, following the previous code, would be *`api/v1/customers/{customerId}`* where *`{customerId}`* is a *`Guid`*.

#### Body of the request argument

Argument can also be passed in the body of the request. By default this is JSON and is most commonly used in *`Post`* and *`Put`* methods.

```c#
	[HttpPost]
    public async Task<IActionResult> Post([FromBody]Customer customer)
    {
       var customer = await _customerServices.Create(customer);
       return Ok(customer);
    }

	[HttpPut]
    public async Task<IActionResult> Put(Customer customer)
    {
       var customer = await _customerServices.Update(customer);
       return Ok(customer);
    }
```

In *`Post`* and *`Put`* methods it is not required to put *`FromBody`* attribute as the code will work and default to it. Both of these samples would work as body arguments.

#### URL parameter argument

URL parameters are usually used for filtering a resources, example of pagination.

```c#
	[HttpGet]
    public async Task<IActionResult> Get([FromQuery]Pagination pagination)
    {
       var paginatedCustomers = await _customerServices.GetAllCustomersPaginated(pagination);
       return Ok(paginatedCustomers);
    }
```

The route to this endpoint, following the previous code, would be *`api/v1/customers?pageNumber=1&pageSize=10`* where *`pageNumber`* and *`pageSize`* are public properties of *`Pagination`* class.

#### Combined arguments

For consistency we are usually trying to avoid combined arguments, but they are also supported and there are edge cases where they can be used. In this example, we are getting a customer information together with all orders of the customer. This can potentially be a large response and we want to provide client an option to not get order information.

```c#
	[HttpGet("{customerId}")]
    public async Task<IActionResult> Get(Guid customerId, [FromQuery]bool includeOrders = false)
    {
       var customer = await _customerServices.Get(customerId, includeOrders);
       return Ok(paginatedCustomers);
    }
```

This endpoint now serves two routes: *`api/v1/customers/{customerId}`*, *`api/v1/customers/{customerId}?includeOrders={value}`* where *`{customerId}`* is GUID and *`value`* is Boolean (true/false) value.



### Custom routes

A custom route for action can be defined on action level. If the route is relative, meaning it doesn't start with *`/`* or *`~/`* than it will be relative to controller route.

```c#
	[HttpPost]
	[Route("{customerId}/activateCustomer")]
    public async Task<IActionResult> Activate(Guid customerId)
    {
       await _customerServices.Activate(customerId);
       return Ok();
    }
```

The route to this endpoint, following the previous code, would be *`api/v1/customers/{customerId}/activateCustomer`* where *`customerId`* is a GUID.

```c#
    [HttpPost]
    [Route("/api/custom/{customerId}/activateCustomer")]
    public async Task<IActionResult> Activate(Guid customerId)
    {
       await _customerServices.Activate(customerId);
       return Ok();
    }
```

The route to this endpoint, would be *`api/custom/{customerId}/activateCustomer`* where *`customerId`* is a GUID.

### Response codes

Most commonly controllers should return a *`OK (200)`* response code and other response codes should be handled in the exception handling middleware. For exception handling, please refer to [Exception Handling](Exception handling.md). 

Other codes that are commonly used are:

*`CREATED (201)`* - resource is created successfully.

*`NOT FOUND (404)`* - resource not found.

*`BAD REQUEST (400)`* - request does not contain all required arguments.

*`UNAUTHORIZED (401)`* - client has not provided correct authorization.

*`METHODNOTALLOWED (405)`* - client has not enough permissions to access this resource.

*`INTERNALSERVERERROR (500)`* - unhandled exception on server side.

### Documentation

A full documentation with Microsoft guidelines can be found [here](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md).