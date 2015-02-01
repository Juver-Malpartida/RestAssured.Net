# RestAssured.Net
A simple testing suite for REST based interfaces.  The goal is to provide a tool that is simple to use and produces repeatable results.

## Download
### Nuget
[Nuget RestAssured.Net Info](https://www.nuget.org/packages/RestAssured/)
```Text
PM> Install-Package RestAssured
```

## Example
### Response
```JSON
{
  "ip":"2605:6000:f705:ab00:78e3:1959:78d4:bd76",
  "about":"/about",
  "Pro!":"http://getjsonip.com"
}
```
### Test
```C#
//Create a new test suite
new RestAssured()
  .Given()
    //Optional, set the name of this suite
    .Name("JsonIP Test Suite")
    //Optional, set the header parameters.  
    //Defaults will be set to application/json if none is given
    .Header("Content-Type", "application/json")
    .Header("Accept-Encoding", "gzip,deflate")
  .When()
    //url
    .Get("http://jsonip.com")
  .Then()
    //Give the name of the test and a lambda expression to test with
    //The lambda expression keys off of 'x' which represents the json blob as a dynamic.
    .TestBody("test a", x => x.about != null)
    //Throw an AssertException if the test case is false.
    .Assert("test a");
```

## Structure
The call chains are structured around 4 main parts.  
  1. Newing up RestAssured gives you a chainable object to work with
  2. All commands under Given() are setup procedures
    - Setup Headers
    - Setup Parameters
    - Setup Host and Uri Information
  3. All commands under When() are used for configuration the call environment
    - eg: GET vs POST
  4. All commands under Then() are used for setting up test rules and assertions

## Commands

### Given
```C#
  .Given()
    //Set up the name for a test suite
    .Name("string")
    //Set up Http Header
    //eg: "Accept-Encoding", "gzip, deflate"
    .Header("string", "string")
    //Add query string parameters for any HTTP verb
    .Query("string", "string")
    //Add parameters for the body of the request if it is POST, PUT or DELETE
    .Param("string", "string")
    //Allows for a string content such as JSON or XML.  Used for POST, PUT and DELETE.  The body is overriden if Param() is used
    .Body("string")
    //Allows for an object to be serialized to JSON or XML.  Used for POST, PUT and DELETE.  The body is overriden if Param() is used
    .Body<T>(T object)
    //Set the host of the target server
    //Useful when you want to reuse a test suite between multiple Uris
    .Host("string")
    //Set the uri for the target endpoint
    .Uri("string")
    //Debug the settings
    .Debug()
```

### When
```C#
  .When()
    //Add a load test to the mix.  Set the # of threads and # of secs to run
    .Load(int, int)
    //Use the GET verb with a url.  This will override the above section.
    .Get("string")
    //Use the GET verb without and rely on url settings from Given() section
    .Get()
    //Use the POST verb with a url.  This will override the above section.
    .Post("string")
    //Use the POST verb without and rely on the url settings from Given() section.
    .Post()
    //Use the PUT verb with a url.  This will override the above section.
    .Put("string")
    //Use the PUT verb without and rely on the url settings from Given() section.
    .Put()
    //Use the DELETE verb with a url.  This will override the above section.
    .Delete("string")
    //Use the DELETE verb without and rely on the url settings from Given() section.
    .Delete()
    //Debug the settings
    .Debug()
```

### Then
```C#
  .Then()
    //Write a test to make an assertion against the response body
    //The response body is the json blob that is returned from a REST call
    //eg: "test A", x => x.id != null
    .TestBody("string", Func<dynamic, bool>)
    //Write a test to make an assertion against the response header
    //eg: "test B", "content-type", x => x.Contains("json")
    .TestHeader("string", "string", Func<string, bool>)
    //Write a test to make an assertion against load test results.  Refer to the Reference section below for complete list of keys.
    //eg: "test C", "average-ttl-ms", x => x < 1000 && x > 0
    .TestLoad("string", "string", Func<double, bool>)
    //Write a test to make an assertion against the response status code
    //eg: "test D", x => x == 200
    .TestStatus("string", Func<int, bool>)
    //Assert your test.  Failed test will throw an AssertException
    .Assert("string")
    //Apply a v3 or v4 json schema to the response.  Corrupted schemas will throw an ArgumentException
    .Schema("string")
    //Assert your schema against the response.  Failed test will throw an AssertException
    .AssertSchema()
    //Assert all your test.  Failed test will throw an AssertException
    .AssertAll()
    //Output all of the test results
    .WriteAssertions()
    //Debug the response
    .Debug()
```

##Reference

###Keys for TestLoad() Command
There are predefined keys for these test.
```
total-call
total-succeeded
total-lost
average-ttl-ms
maximum-ttl-ms
minimum-ttl-ms
```


## JSON Schema Support
ResAssured leverages Newtonsoft.Json for its JSON parsing and JSON schema validation support.  
Newtonsoft.Json on supports draft 3 and draft 4 of the JSON scheam specification, so we can too!

Utilize these tools to validate your JSON schema

[v3 Schema Validator](http://jsonschema.net/previous/)

[v4 Schema Validator](http://jsonschema.net)

## More Examples

### Breaking up a call chain
```C#
//Create a new test suite
var endpoint1 = new RestAssured()
  .Given()
    //Optional, set the name of this suite
    .Name("JsonIP Test Suite")
    //Optional, set the header parameters.  
    //Defaults will be set to application/json if none is given
    .Header("Content-Type", "application/json")
    .Header("Accept-Encoding", "gzip,deflate");
    .Host("jsonip.com")
    .Uri("/endpoint1");

//Make a copy of the settings from above, but adjust the endpoint.
var endpoint2 = endpoint1.Clone().Uri("/endpoint2");

//Do a GET action with the first endpoint configuration
endpoint1.When().Get().Then().TestBody("test 1", x => x.id != null).Assert("test 1");

//Do a POSt action with the second endpoint configuration
endpoint2.When().Post().Then().TestBody("test 1", x => x.id != null).Assert("test 1");
```

### Load Test
```C#
new RestAssured()
    .Given()
        .Name("JsonIP multi thread")
    .When()
        //Configure a load test with
        //6 threads that runs for 30 seconds
        .Load(6, 30)
        //Using this address
        .Get("http://yourremote.com")
    .Then()
        //Assert Load Test and Print Results
        .Debug
        .TestLoad("good-average", "average-ttl-ms", x => x > 100 && x < 400)
        .Assert("good-average");
        
```
