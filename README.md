[![ci.appveyor.com](https://ci.appveyor.com/api/projects/status/github/blushingpenguin/Jolt.Net?branch=master&svg=true)](https://ci.appveyor.com/api/projects/status/github/blushingpenguin/Jolt.Net?branch=master&svg=true)
[![codecov.io](https://codecov.io/gh/blushingpenguin/Jolt.Net/coverage.svg?branch=master)](https://codecov.io/gh/blushingpenguin/Jolt.Net?branch=master)

Jolt.Net
========

JSON to JSON transformation library written in c# where the "specification" for the transform is itself a JSON document.
This is a c# port of the original Java version from https://github.com/bazaarvoice/jolt.  JSON handling is provided
by [Newtonsoft Json](https://www.newtonsoft.com/json)

### Useful For

1. Transforming JSON data from ElasticSearch, MongoDb, Cassandra, etc before sending it off to the world
1. Extracting data from a large JSON documents for your own consumption

## Table of Contents

   1. [Overview](#Overview)
   2. [Documentation](#Documentation)
   3. [Shiftr Transform DSL](#Shiftr_Transform_DSL)
   4. [Demo](#Demo)
   5. [Getting Started](#Getting_Started)
   6. [Getting Transform Help](#Getting_Transform_Help)
   7. [Why Jolt Exists](#Why_Jolt_Exists)
   8. [Alternatives](#Alternatives)
   9. [Performance](#Performance)

## <a name="Overview"></a> Overview

Jolt :

* provides a set of transforms, that can be "chained" together to form the overall JSON to JSON transform.
* focuses on transforming the *structure* of your JSON data, not manipulating specific values
    * The idea being: use Jolt to get most of the structure right, then write code to fix values
    * consumes and produces Newtonsoft.Json objects (JToken, JArray, JObject, ...)

### Stock Transforms

The Stock transforms are:

    shift       : copy data from the input tree and put it the output tree
    default     : apply default values to the tree
    remove      : remove data from the tree
    sort        : sort the Map key values alphabetically ( for debugging and human readability )
    cardinality : "fix" the cardinality of input data.  Eg, the "urls" element is usually a List, but if there is only one, then it is a String

Each transform has it's own DSL (Domain Specific Language) in order to facilitate it's narrow job.

Currently, all the Stock transforms just effect the "structure" of the data.
To do data manipulation, you will need to write c# code.   If you write your c# "data manipulation" code to implement the Transform interface, then you can insert your code in the transform chain.

The out-of-the-box Jolt transforms should be able to do most of your structural transformation, with custom c# implementing your data manipulation.

## <a name="Documentation"></a> Documentation

Jolt [Slide Deck](https://docs.google.com/presentation/d/1sAiuiFC4Lzz4-064sg1p8EQt2ev0o442MfEbvrpD1ls/edit?usp=sharing) : covers motivation, development, and transforms.

Javadoc explaining each transform DSL :

* [shift](https://github.com/blushingpenguin/Jolt.Net/blob/master/Jolt.Net/Shiftr.cs)
* [default](https://github.com/blushingpenguin/Jolt.Net/blob/master/Jolt.Net/Defaultr.cs)
* [remove](https://github.com/blushingpenguin/Jolt.Net/blob/master/Jolt.Net/Removr.cs)
* [cardinality](https://github.com/blushingpenguin/Jolt.Net/blob/master/Jolt.Net/CardinalityTransform.cs)
* [sort](https://github.com/blushingpenguin/Jolt.Net/blob/master/Jolt.Net/Sortr.cs)
* fully qualified c# type name: Class implements the ITransform or IContextualTransform interfaces, and can optionally be SpecDriven (abstract base class)
    * [Transform](https://github.com/blushingpenguin/Jolt.Net/blob/master/Jolt.Net/Transform.cs) interface
    * [SpecDriven](https://github.com/blushingpenguin/Jolt.Net/blob/master/Jolt.Net/SpecDriven.cs)
        * where the "input" is the JSON data as a JToken

Running a Jolt transform means creating an instance of [Chainr](https://github.com/blushingpenguin/Jolt.Net/blob/master/Jolt.Net/Chainr.cs)  with a list of transforms.

The JSON spec for Chainr looks like : [unit test](https://github.com/blushingpenguin/Jolt.Net/blob/master/Jolt.Net.Test/json/chainr/integration/firstSample.json).

The Java side looks like :

``` csharp
var json = JObject.Parse(File.ReadAllText("/path/to/chainr/spec.json"));
var output = chainr.Transform(input);

return output;
```

### <a name="Shiftr_Transform_DSL"></a> Shiftr Transform DSL

The Shiftr transform generally does most of the "heavy lifting" in the transform chain.
To see the Shiftr DSL in action, please look at our unit tests ([shiftr tests](https://github.com/blushingpenguin/Jolt.Net/tree/master/Jolt.Net.Test/json/shiftr)) for nice bite sized transform examples, and read the extensive Shiftr [javadoc](https://github.com/blushingpenguin/Jolt.Net/blob/master/Jolt.Net/Shiftr.cs).

Our unit tests follow the pattern :

``` json
{
    "input": {
        // sample input
    },

    "spec": {
        // transform spec
    },

    "expected": {
        // what the output of the transform looks like
    }
}
```

We read in "input", apply the "spec", and check it against the "expected" using FluentAssertions.Json to handle the comparison.

To learn the Shiftr DSL, examine "input" and "output" json, get an understanding of how data is moving, and *then* look at the transform spec to see how it facilitates the transform.

For reference, [this](https://github.com/blushingpenguin/Jolt.Net/blob/master/Jolt.Net.Test/json/shiftr/firstSample.json) was the very first test we wrote.


## <a name="Demo"></a> Demo

There is a demo available at [jolt-demo.appspot.com](http://jolt-demo.appspot.com/).
You can paste in JSON input data and a Spec, and it will post the data to server and run the transform.

Note

* it is hosted on a free Google App Engine instance, so it may take a minute to spin up.
* it validates in input JSON and spec client side.

## <a name="Getting_Started"></a> Getting Started

Getting started code wise has it's [own doc](gettingStarted.md).

## <a name="Getting_Transform_Help"></a> Getting Transform Help

If you can't get a transform working and you need help, create and Issue in Jolt (for now).

Make sure you include what your "input" is, and what you want your "output" to be.

## <a name="Why_Jolt_Exists"></a> Why Jolt Exists

Aside from writing your own custom code to do a transform, there are two general approaches to doing a JSON to JSON transforms in Java.

1) JSON -> XML -> XSLT or STX -> XML -> JSON

Aside from being a Rube Goldberg approach, XSLT is more complicated than Jolt because it is trying to do the whole transform with a single DSL.

2) Write a Template (Velocity, FreeMarker, etc) that take hydrated JSON input and write textual JSON output

With this approach you are working from the output format backwards to the input, which is complex for any non-trivial transform.
Eg, the structure of your template will be dictated by the output JSON format, and you will end up coding a parallel tree walk of the input data and the output format in your template.
Jolt works forward from the input data to the output format which is simpler, and it does the parallel tree walk for you.

## <a name="Alternatives"></a> Alternatives

Being in the Java JSON processing "space", here are some other interesting JSON manipulation tools to look at / consider :

* [jq](https://stedolan.github.io/jq) - Awesome command line tool to extract data from JSON files (use it all the time, available via brew)
* [JsonPath](https://github.com/jayway/JsonPath) - Java : Extract data from JSON using XPATH like syntax.
* [JsonSurfer](https://github.com/jsurfer/JsonSurfer) - Java : Streaming JsonPath processor dedicated to processing big and complicated JSON data.

## <a name="Performance"></a> Performance

The primary goal of Jolt was to improve "developer speed" by providing the ability to have a declarative rather than imperative transforms.
That said, Jolt should have a better runtime than the alternatives listed above.

Work has been done to make the stock Jolt transforms fast:

1. Transforms can be initialized once with their spec, and re-used many times in a multi-threaded environment.
    * We reuse initialized Jolt transforms to service multiple web requests from a DropWizard service.
2. "*" wildcard logic was redone to reduce the use of Regex in the common case, which was a dramatic speed improvement.
3. The parallel tree walk performed by Shiftr was optimized.

Two things to be aware of :

1. Jolt is not "stream" based, so if you have a very large Json document to transform you need to have enough memory to hold it.
2. The transform process will create and discard a lot of objects, so the garbage collector will have work to do.
