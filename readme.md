<!--
GENERATED FILE - DO NOT EDIT
This file was generated by [MarkdownSnippets](https://github.com/SimonCropp/MarkdownSnippets).
Source File: /readme.source.md
To change this file edit the source file and then run MarkdownSnippets.
-->

# <img src="/src/icon.png" height="30px"> XunitContext

[![Build status](https://ci.appveyor.com/api/projects/status/sdg2ni2jhe2o33le/branch/master?svg=true)](https://ci.appveyor.com/project/SimonCropp/XunitContext)
[![NuGet Status](https://img.shields.io/nuget/v/XunitContext.svg)](https://www.nuget.org/packages/XunitContext/)

Extends [xUnit](https://xunit.net/) to expose extra context and simplify logging.

Redirects [Trace.Write](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.trace.write), [Debug.Write](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.debug.write), and [Console.Write and Console.Error.Write](https://docs.microsoft.com/en-us/dotnet/api/system.console.write) to [ITestOutputHelper](https://xunit.net/docs/capturing-output). Also provides static access to the current [ITestOutputHelper](https://xunit.net/docs/capturing-output) for use within testing utility methods.

Uses [AsyncLocal](https://docs.microsoft.com/en-us/dotnet/api/system.threading.asynclocal-1) to track state.

Support is available via a [Tidelift Subscription](https://tidelift.com/subscription/pkg/nuget-xunitcontext?utm_source=nuget-xunitcontext&utm_medium=referral&utm_campaign=enterprise).

<!-- toc -->
## Contents

  * [ClassBeingTested](#classbeingtested)
  * [XunitContextBase](#xunitcontextbase)
  * [Logging](#logging)
    * [Logging Libs](#logging-libs)
  * [Filters](#filters)
  * [Context](#context)
    * [Current Test](#current-test)
    * [Test Failure](#test-failure)
    * [Counters](#counters)
    * [Base Class](#base-class)
    * [Parameters](#parameters)
    * [UniqueTestName](#uniquetestname)
  * [Global Setup](#global-setup)
  * [Security contact information](#security-contact-information)<!-- endtoc -->


## NuGet package

https://nuget.org/packages/XunitContext/


## ClassBeingTested

<!-- snippet: ClassBeingTested.cs -->
<a id='snippet-ClassBeingTested.cs'/></a>
```cs
using System;
using System.Diagnostics;

static class ClassBeingTested
{
    public static void Method()
    {
        Trace.WriteLine("From Trace");
        Console.WriteLine("From Console");
        Debug.WriteLine("From Debug");
        Console.Error.WriteLine("From Console Error");
    }
}
```
<sup><a href='/src/Tests/Snippets/ClassBeingTested.cs#L1-L13' title='File snippet `ClassBeingTested.cs` was extracted from'>snippet source</a> | <a href='#snippet-ClassBeingTested.cs' title='Navigate to start of snippet `ClassBeingTested.cs`'>anchor</a></sup>
<!-- endsnippet -->


## XunitContextBase

`XunitContextBase` is an abstract base class for tests. It exposes logging methods for use from unit tests, and handle the flushing of longs in its `Dispose` method. `XunitContextBase` is actually a thin wrapper over `XunitContext`. `XunitContext`s `Write*` methods can also be use inside a test inheriting from `XunitContextBase`.

<!-- snippet: TestBaseSample.cs -->
<a id='snippet-TestBaseSample.cs'/></a>
```cs
using Xunit;
using Xunit.Abstractions;

public class TestBaseSample  :
    XunitContextBase
{
    [Fact]
    public void Write_lines()
    {
        WriteLine("From Test");
        ClassBeingTested.Method();

        var logs = XunitContext.Logs;

        Assert.Contains("From Test", logs);
        Assert.Contains("From Trace", logs);
        Assert.Contains("From Debug", logs);
        Assert.Contains("From Console", logs);
        Assert.Contains("From Console Error", logs);
    }

    public TestBaseSample(ITestOutputHelper output) :
        base(output)
    {
    }
}
```
<sup><a href='/src/Tests/Snippets/TestBaseSample.cs#L1-L26' title='File snippet `TestBaseSample.cs` was extracted from'>snippet source</a> | <a href='#snippet-TestBaseSample.cs' title='Navigate to start of snippet `TestBaseSample.cs`'>anchor</a></sup>
<!-- endsnippet -->


## Logging

`XunitContext` provides static access to the logging state for tests. It exposes logging methods for use from unit tests, however registration of [ITestOutputHelper](https://xunit.net/docs/capturing-output) and flushing of logs must be handled explicitly.

<!-- snippet: XunitLoggerSample.cs -->
<a id='snippet-XunitLoggerSample.cs'/></a>
```cs
using System;
using Xunit;
using Xunit.Abstractions;

public class XunitLoggerSample :
    IDisposable
{
    [Fact]
    public void Usage()
    {
        XunitContext.WriteLine("From Test");

        ClassBeingTested.Method();

        var logs = XunitContext.Logs;

        Assert.Contains("From Test", logs);
        Assert.Contains("From Trace", logs);
        Assert.Contains("From Debug", logs);
        Assert.Contains("From Console", logs);
        Assert.Contains("From Console Error", logs);
    }

    public XunitLoggerSample(ITestOutputHelper testOutput)
    {
        XunitContext.Register(testOutput);
    }

    public void Dispose()
    {
        XunitContext.Flush();
    }
}
```
<sup><a href='/src/Tests/Snippets/XunitLoggerSample.cs#L1-L33' title='File snippet `XunitLoggerSample.cs` was extracted from'>snippet source</a> | <a href='#snippet-XunitLoggerSample.cs' title='Navigate to start of snippet `XunitLoggerSample.cs`'>anchor</a></sup>
<!-- endsnippet -->

`XunitContext` redirects [Trace.Write](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.trace.write), [Console.Write](https://docs.microsoft.com/en-us/dotnet/api/system.console.write), and [Debug.Write](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.debug.write) in its static constructor.

<!-- snippet: writeRedirects -->
<a id='snippet-writeredirects'/></a>
```cs
Trace.Listeners.Clear();
Trace.Listeners.Add(new TraceListener());
#if (NETSTANDARD)
DebugPoker.Overwrite(
text =>
{
    if (string.IsNullOrEmpty(text))
    {
        return;
    }

    if (text.EndsWith(Environment.NewLine))
    {
        WriteLine(text.TrimTrailingNewline());
        return;
    }

    Write(text);
});
#else
Debug.Listeners.Clear();
Debug.Listeners.Add(new TraceListener());
#endif
var writer = new TestWriter();
Console.SetOut(writer);
Console.SetError(writer);
```
<sup><a href='/src/XunitContext/XunitContext.cs#L55-L84' title='File snippet `writeredirects` was extracted from'>snippet source</a> | <a href='#snippet-writeredirects' title='Navigate to start of snippet `writeredirects`'>anchor</a></sup>
<!-- endsnippet -->

These API calls are then routed to the correct xUnit [ITestOutputHelper](https://xunit.net/docs/capturing-output) via a static [AsyncLocal](https://docs.microsoft.com/en-us/dotnet/api/system.threading.asynclocal-1).


### Logging Libs

Approaches to routing common logging libraries to Diagnostics.Trace:

 * [Serilog](https://serilog.net/) use [Serilog.Sinks.Trace](https://github.com/serilog/serilog-sinks-trace).
 * [NLog](https://github.com/NLog/NLog) use a [Trace target](https://github.com/NLog/NLog/wiki/Trace-target).


## Filters

`XunitContext.Filters` can be used to filter out unwanted lines:

<!-- snippet: FilterSample.cs -->
<a id='snippet-FilterSample.cs'/></a>
```cs
using Xunit;
using Xunit.Abstractions;

public class FilterSample :
    XunitContextBase
{
    static FilterSample()
    {
        Filters.Add(x => x != null && !x.Contains("ignored"));
    }

    [Fact]
    public void Write_lines()
    {
        WriteLine("first");
        WriteLine("with ignored string");
        WriteLine("last");
        var logs = XunitContext.Logs;

        Assert.Contains("first", logs);
        Assert.DoesNotContain("with ignored string", logs);
        Assert.Contains("last", logs);
    }

    public FilterSample(ITestOutputHelper output) :
        base(output)
    {
    }
}
```
<sup><a href='/src/Tests/Snippets/FilterSample.cs#L1-L29' title='File snippet `FilterSample.cs` was extracted from'>snippet source</a> | <a href='#snippet-FilterSample.cs' title='Navigate to start of snippet `FilterSample.cs`'>anchor</a></sup>
<!-- endsnippet -->

Filters are static and shared for all tests.


## Context

For every tests there is a contextual API to perform several operations.

 * `Context.TestOutput`: Access to [ITestOutputHelper](https://xunit.net/docs/capturing-output).
 * `Context.Write` and `Context.WriteLine`: Write to the current log.
 * `Context.LogMessages`: Access to all log message for the current test.
 * [Counters](#counters): Provide access in predicable and incrementing values for the following types: `Guid`, `Int`, `Long`, `UInt`, and `ULong`.
 * `Context.Test`: Access to the current `ITest`.
 * `Context.SourceFile`: Access to the file path for the current test.
 * `Context.SourceDirectory`: Access to the directory path for the current test.
 * `Context.SolutionDirectory`: The current solution directory. Obtained by walking up the directory tree from `SourceDirectory`.
 * `Context.TestException`: Access to the exception if the current test has failed. See [Test Failure](test-failure).

<!-- snippet: ContextSample.cs -->
<a id='snippet-ContextSample.cs'/></a>
```cs
using Xunit;
using Xunit.Abstractions;

public class ContextSample  :
    XunitContextBase
{
    [Fact]
    public void Usage()
    {
        var currentGuid = Context.CurrentGuid;

        var nextGuid = Context.NextGuid();

        Context.WriteLine("Some message");

        var currentLogMessages = Context.LogMessages;

        var testOutputHelper = Context.TestOutput;

        var currentTest = Context.Test;

        var sourceFile = Context.SourceFile;

        var sourceDirectory = Context.SourceDirectory;

        var solutionDirectory = Context.SolutionDirectory;

        var currentTestException = Context.TestException;
    }

    public ContextSample(ITestOutputHelper output) :
        base(output)
    {
    }
}
```
<sup><a href='/src/Tests/Snippets/ContextSample.cs#L1-L35' title='File snippet `ContextSample.cs` was extracted from'>snippet source</a> | <a href='#snippet-ContextSample.cs' title='Navigate to start of snippet `ContextSample.cs`'>anchor</a></sup>
<!-- endsnippet -->

Some members are pushed down to the be accessible directly from `XunitContextBase`:

<!-- snippet: ContextPushedDownSample.cs -->
<a id='snippet-ContextPushedDownSample.cs'/></a>
```cs
using Xunit;
using Xunit.Abstractions;

public class ContextPushedDownSample  :
    XunitContextBase
{
    [Fact]
    public void Usage()
    {
        WriteLine("Some message");

        var currentLogMessages = Logs;

        var testOutputHelper = Output;

        var sourceFile = SourceFile;

        var sourceDirectory = SourceDirectory;

        var solutionDirectory = SolutionDirectory;

        var currentTestException = TestException;
    }

    public ContextPushedDownSample(ITestOutputHelper output) :
        base(output)
    {
    }
}
```
<sup><a href='/src/Tests/Snippets/ContextPushedDownSample.cs#L1-L29' title='File snippet `ContextPushedDownSample.cs` was extracted from'>snippet source</a> | <a href='#snippet-ContextPushedDownSample.cs' title='Navigate to start of snippet `ContextPushedDownSample.cs`'>anchor</a></sup>
<!-- endsnippet -->

Context can accessed via a static API:

<!-- snippet: ContextStaticSample.cs -->
<a id='snippet-ContextStaticSample.cs'/></a>
```cs
using Xunit;
using Xunit.Abstractions;

public class ContextStaticSample :
    XunitContextBase
{
    [Fact]
    public void StaticUsage()
    {
        var currentGuid = XunitContext.Context.CurrentGuid;

        var nextGuid = XunitContext.Context.NextGuid();

        XunitContext.Context.WriteLine("Some message");

        var currentLogMessages = XunitContext.Context.LogMessages;

        var testOutputHelper = XunitContext.Context.TestOutput;

        var currentTest = XunitContext.Context.Test;

        var sourceFile = XunitContext.Context.SourceFile;

        var sourceDirectory = XunitContext.Context.SourceDirectory;

        var solutionDirectory = XunitContext.Context.SolutionDirectory;

        var currentTestException = XunitContext.Context.TestException;
    }

    public ContextStaticSample(ITestOutputHelper output) :
        base(output)
    {
    }
}
```
<sup><a href='/src/Tests/Snippets/ContextStaticSample.cs#L1-L35' title='File snippet `ContextStaticSample.cs` was extracted from'>snippet source</a> | <a href='#snippet-ContextStaticSample.cs' title='Navigate to start of snippet `ContextStaticSample.cs`'>anchor</a></sup>
<!-- endsnippet -->


### Current Test

There is currently no API in xUnit to retrieve information on the current test. See issues [#1359](https://github.com/xunit/xunit/issues/1359), [#416](https://github.com/xunit/xunit/issues/416), and [#398](https://github.com/xunit/xunit/issues/398).

To work around this, this project exposes the current instance of `ITest` via reflection.

Usage:

<!-- snippet: CurrentTestSample.cs -->
<a id='snippet-CurrentTestSample.cs'/></a>
```cs
using Xunit;
using Xunit.Abstractions;

public class CurrentTestSample :
    XunitContextBase
{
    [Fact]
    public void Usage()
    {
        var currentTest = Context.Test;
        // DisplayName will be 'TestNameSample.Usage'
        var displayName = currentTest.DisplayName;
    }

    [Fact]
    public void StaticUsage()
    {
        var currentTest = XunitContext.Context.Test;
        // DisplayName will be 'TestNameSample.StaticUsage'
        var displayName = currentTest.DisplayName;
    }

    public CurrentTestSample(ITestOutputHelper output) :
        base(output)
    {
    }
}
```
<sup><a href='/src/Tests/Snippets/CurrentTestSample.cs#L1-L27' title='File snippet `CurrentTestSample.cs` was extracted from'>snippet source</a> | <a href='#snippet-CurrentTestSample.cs' title='Navigate to start of snippet `CurrentTestSample.cs`'>anchor</a></sup>
<!-- endsnippet -->

Implementation:

<!-- snippet: Context_CurrentTest.cs -->
<a id='snippet-Context_CurrentTest.cs'/></a>
```cs
using System;
using System.Reflection;
using Xunit.Abstractions;
using Xunit.Sdk;

namespace Xunit
{
    public partial class Context
    {
        ITest? test;

        static FieldInfo? cachedTestMember;

        public ITest Test
        {
            get
            {
                InitTest();

                return test!;
            }
        }

        MethodInfo? methodInfo;
        public MethodInfo MethodInfo
        {
            get
            {
                InitTest();
                return methodInfo!;
            }
        }

        Type? testType;
        public Type TestType
        {
            get
            {
                InitTest();
                return testType!;
            }
        }

        void InitTest()
        {
            if (test != null)
            {
                return;
            }
            test = (ITest) GetTestMethod().GetValue(TestOutput);
            var method = (ReflectionMethodInfo) test.TestCase.TestMethod.Method;
            var type = (ReflectionTypeInfo) test.TestCase.TestMethod.TestClass.Class;
            methodInfo = method.MethodInfo;
            testType = type.Type;
        }

        public static string MissingTestOutput = "ITestOutputHelper has not been set. It is possible that the call to `XunitContext.Register()` is missing, or the current test does not inherit from `XunitContextBase`.";

        FieldInfo GetTestMethod()
        {
            if (TestOutput == null)
            {
                throw new Exception(MissingTestOutput);
            }

            if (cachedTestMember != null)
            {
                return cachedTestMember;
            }

            var testOutputType = TestOutput.GetType();
            cachedTestMember = testOutputType.GetField("test", BindingFlags.Instance | BindingFlags.NonPublic);
            if (cachedTestMember == null)
            {
                throw new Exception($"Unable to find 'test' field on {testOutputType.FullName}");
            }

            return cachedTestMember;
        }
    }
}
```
<sup><a href='/src/XunitContext/Context_CurrentTest.cs#L1-L81' title='File snippet `Context_CurrentTest.cs` was extracted from'>snippet source</a> | <a href='#snippet-Context_CurrentTest.cs' title='Navigate to start of snippet `Context_CurrentTest.cs`'>anchor</a></sup>
<!-- endsnippet -->


### Test Failure

When a test fails it is expressed as an exception. The exception can be viewed by enabling exception capture, and then accessing `Context.TestException`. The `TestException` will be null if the test has passed.

One common case is to perform some logic, based on the existence of the exception, in the `Dispose` of a test.

<!-- snippet: TestExceptionSample -->
<a id='snippet-testexceptionsample'/></a>
```cs
public class TestExceptionSample :
    XunitContextBase
{
    [Fact]
    public void Usage()
    {
        //This tests will fail
        Assert.False(true);
    }

    public TestExceptionSample(ITestOutputHelper output) :
        base(output)
    {
    }

    public override void Dispose()
    {
        var theExceptionThrownByTest = Context.TestException;
        var testDisplayName = Context.Test.DisplayName;
        var testCase = Context.Test.TestCase;
        base.Dispose();
    }
}

[GlobalSetUp]
public static class GlobalSetup
{
    public static void Setup()
    {
        XunitContext.EnableExceptionCapture();
    }
}
```
<sup><a href='/src/Tests/Snippets/TestExceptionSample.cs#L8-L44' title='File snippet `testexceptionsample` was extracted from'>snippet source</a> | <a href='#snippet-testexceptionsample' title='Navigate to start of snippet `testexceptionsample`'>anchor</a></sup>
<!-- endsnippet -->


### Counters

Provide access to predicable and incrementing values for the following types: `Guid`, `Int`, `Long`, `UInt`, and `ULong`.


#### Non Test Context usage

Counters can also be used outside of the current test context:

<!-- snippet: NonTestContextUsage -->
<a id='snippet-nontestcontextusage'/></a>
```cs
var current = Counters.CurrentGuid;

var next = Counters.NextGuid();

var counter = new GuidCounter();
var localCurrent = counter.Current;
var localNext = counter.Next();
```
<sup><a href='/src/Tests/Snippets/CountersSample.cs#L8-L16' title='File snippet `nontestcontextusage` was extracted from'>snippet source</a> | <a href='#snippet-nontestcontextusage' title='Navigate to start of snippet `nontestcontextusage`'>anchor</a></sup>
<!-- endsnippet -->


#### Implementation

<!-- snippet: Context_Counters.cs -->
<a id='snippet-Context_Counters.cs'/></a>
```cs
using System;

namespace Xunit
{
    public partial class Context
    {
        GuidCounter GuidCounter = new GuidCounter();
        IntCounter IntCounter = new IntCounter();
        LongCounter LongCounter = new LongCounter();
        UIntCounter UIntCounter = new UIntCounter();
        ULongCounter ULongCounter = new ULongCounter();
        DateTimeOffsetCounter DateTimeOffsetCounter = new DateTimeOffsetCounter();
        DateTimeCounter DateTimeCounter = new DateTimeCounter();

        public uint CurrentUInt
        {
            get => UIntCounter.Current;
        }

        public int CurrentInt
        {
            get => IntCounter.Current;
        }

        public long CurrentLong
        {
            get => LongCounter.Current;
        }

        public ulong CurrentULong
        {
            get => ULongCounter.Current;
        }

        public Guid CurrentGuid
        {
            get => GuidCounter.Current;
        }

        public DateTime CurrentDateTime
        {
            get => DateTimeCounter.Current;
        }

        public DateTimeOffset CurrentDateTimeOffset
        {
            get => DateTimeOffsetCounter.Current;
        }

        public int IntOrNext<T>(T input)
        {
            if (input is Guid guidInput)
            {
                return GuidCounter.IntOrNext(guidInput);
            }

            if (input is int intInput)
            {
                return IntCounter.IntOrNext(intInput);
            }

            if (input is uint uIntInput)
            {
                return UIntCounter.IntOrNext(uIntInput);
            }

            if (input is ulong ulongInput)
            {
                return ULongCounter.IntOrNext(ulongInput);
            }

            if (input is long longInput)
            {
                return LongCounter.IntOrNext(longInput);
            }

            if (input is DateTime dateTimeInput)
            {
                return DateTimeCounter.IntOrNext(dateTimeInput);
            }

            if (input is DateTimeOffset dateTimeOffsetInput)
            {
                return DateTimeOffsetCounter.IntOrNext(dateTimeOffsetInput);
            }

            throw new Exception($"Unknown type {typeof(T).FullName}");
        }

        public uint NextUInt()
        {
            return UIntCounter.Next();
        }

        public int NextInt()
        {
            return IntCounter.Next();
        }

        public long NextLong()
        {
            return LongCounter.Next();
        }

        public ulong NextULong()
        {
            return ULongCounter.Next();
        }

        public Guid NextGuid()
        {
            return GuidCounter.Next();
        }

        public DateTime NextDateTime()
        {
            return DateTimeCounter.Next();
        }

        public DateTimeOffset NextTimeOffset()
        {
            return DateTimeOffsetCounter.Next();
        }
    }
}
```
<sup><a href='/src/XunitContext/Context_Counters.cs#L1-L125' title='File snippet `Context_Counters.cs` was extracted from'>snippet source</a> | <a href='#snippet-Context_Counters.cs' title='Navigate to start of snippet `Context_Counters.cs`'>anchor</a></sup>
<!-- endsnippet -->

<!-- snippet: Counters.cs -->
<a id='snippet-Counters.cs'/></a>
```cs
using System;

namespace Xunit
{
    public static class Counters
    {
        static GuidCounter GuidCounter = new GuidCounter();
        static IntCounter IntCounter = new IntCounter();
        static LongCounter LongCounter = new LongCounter();
        static UIntCounter UIntCounter = new UIntCounter();
        static ULongCounter ULongCounter = new ULongCounter();

        public static uint CurrentUInt
        {
            get => UIntCounter.Current;
        }

        public static int CurrentInt
        {
            get => IntCounter.Current;
        }

        public static long CurrentLong
        {
            get => LongCounter.Current;
        }

        public static ulong CurrentULong
        {
            get => ULongCounter.Current;
        }

        public static Guid CurrentGuid
        {
            get => GuidCounter.Current;
        }

        public static uint NextUInt()
        {
            return UIntCounter.Next();
        }

        public static int NextInt()
        {
            return IntCounter.Next();
        }

        public static long NextLong()
        {
            return LongCounter.Next();
        }

        public static ulong NextULong()
        {
            return ULongCounter.Next();
        }

        public static Guid NextGuid()
        {
            return GuidCounter.Next();
        }
    }
}
```
<sup><a href='/src/XunitContext/Counters.cs#L1-L63' title='File snippet `Counters.cs` was extracted from'>snippet source</a> | <a href='#snippet-Counters.cs' title='Navigate to start of snippet `Counters.cs`'>anchor</a></sup>
<!-- endsnippet -->

<!-- snippet: GuidCounter.cs -->
<a id='snippet-GuidCounter.cs'/></a>
```cs
using System;

namespace Xunit
{
    public class GuidCounter :
        Counter<Guid>
    {
        protected override Guid Convert(int i)
        {
            var bytes = new byte[16];
            BitConverter.GetBytes(i).CopyTo(bytes, 0);
            return new Guid(bytes);
        }
    }
}
```
<sup><a href='/src/XunitContext/Counters/GuidCounter.cs#L1-L15' title='File snippet `GuidCounter.cs` was extracted from'>snippet source</a> | <a href='#snippet-GuidCounter.cs' title='Navigate to start of snippet `GuidCounter.cs`'>anchor</a></sup>
<!-- endsnippet -->

<!-- snippet: LongCounter.cs -->
<a id='snippet-LongCounter.cs'/></a>
```cs
namespace Xunit
{
    public class LongCounter :
        Counter<long>
    {
        protected override long Convert(int i)
        {
            return i;
        }
    }
}
```
<sup><a href='/src/XunitContext/Counters/LongCounter.cs#L1-L11' title='File snippet `LongCounter.cs` was extracted from'>snippet source</a> | <a href='#snippet-LongCounter.cs' title='Navigate to start of snippet `LongCounter.cs`'>anchor</a></sup>
<!-- endsnippet -->


### Base Class

When creating a custom base class for other tests, it is necessary to pass through the source file path to `XunitContextBase` via the constructor.

<!-- snippet: XunitContextCustomBase -->
<a id='snippet-xunitcontextcustombase'/></a>
```cs
public class CustomBase :
    XunitContextBase
{
    public CustomBase(
        ITestOutputHelper testOutput,
        [CallerFilePath] string sourceFile = "") :
        base(testOutput, sourceFile)
    {
    }
}
```
<sup><a href='/src/Tests/Snippets/CustomBase.cs#L5-L16' title='File snippet `xunitcontextcustombase` was extracted from'>snippet source</a> | <a href='#snippet-xunitcontextcustombase' title='Navigate to start of snippet `xunitcontextcustombase`'>anchor</a></sup>
<!-- endsnippet -->


### Parameters

Provided the parameters passed to the current test when using a `[Theory]`.

Use cases:

 * To derive the [unique test name](#uniquetestname).
 * In extensibility scenarios for example [Verify file naming](https://github.com/SimonCropp/Verify/blob/master/docs/naming.md).

Usage:

<!-- snippet: ParametersSample.cs -->
<a id='snippet-ParametersSample.cs'/></a>
```cs
using System.Collections.Generic;
using System.Linq;
using Xunit;
using Xunit.Abstractions;

public class ParametersSample :
    XunitContextBase
{
    [Theory]
    [MemberData(nameof(GetData))]
    public void Usage(string arg)
    {
        var parameter = Context.Parameters.Single();
        var parameterInfo = parameter.Info;
        Assert.Equal("arg", parameterInfo.Name);
        Assert.Equal(arg, parameter.Value);
    }

    public static IEnumerable<object[]> GetData()
    {
        yield return new object[] {"Value1"};
        yield return new object[] {"Value2"};
    }

    public ParametersSample(ITestOutputHelper output) :
        base(output)
    {
    }
}
```
<sup><a href='/src/Tests/Snippets/ParametersSample.cs#L1-L29' title='File snippet `ParametersSample.cs` was extracted from'>snippet source</a> | <a href='#snippet-ParametersSample.cs' title='Navigate to start of snippet `ParametersSample.cs`'>anchor</a></sup>
<!-- endsnippet -->

Implementation:

<!-- snippet: Parameters -->
<a id='snippet-parameters'/></a>
```cs
static List<Parameter> GetParameters(ITestCase testCase)
{
    return GetParameters(testCase, testCase.TestMethodArguments);
}

static List<Parameter> GetParameters(ITestCase testCase, object[] arguments)
{
    var method = testCase.TestMethod;
    var infos = method.Method.GetParameters().ToList();
    if (arguments == null || !arguments.Any())
    {
        if (infos.Count == 0)
        {
            return empty;
        }

        throw NewNoArgumentsDetectedException();
    }

    var items = new List<Parameter>();

    for (var index = 0; index < infos.Count; index++)
    {
        items.Add(new Parameter(infos[index], arguments[index]));
    }

    return items;
}
```
<sup><a href='/src/XunitContext/Context_Parameters.cs#L28-L57' title='File snippet `parameters` was extracted from'>snippet source</a> | <a href='#snippet-parameters' title='Navigate to start of snippet `parameters`'>anchor</a></sup>
<!-- endsnippet -->


#### Complex parameters

Only simple types (string, int, DateTime etc) can use the above automated approach. If a complex type is used the following exception will be thrown

 <!-- include: NoArgumentsDetectedException. path: /src/Tests/NoArgumentsDetectedException.include.md -->
> No arguments detected for method with parameters.
> This is most likely caused by using a parameter that Xunit cannot serialize.
> Instead pass in a simple type as a parameter and construct the complex object inside the test.
 <!-- end include: NoArgumentsDetectedException. path: /src/Tests/NoArgumentsDetectedException.include.md -->

To use complex types override the parameter resolution using `XunitContextBase.UseParameters`:

<!-- snippet: ComplexParameterSample.cs -->
<a id='snippet-ComplexParameterSample.cs'/></a>
```cs
using System.Collections.Generic;
using System.Linq;
using Xunit;
using Xunit.Abstractions;

public class ComplexParameterSample :
    XunitContextBase
{
    [Theory]
    [MemberData(nameof(GetData))]
    public void UseComplexMemberData(ComplexClass arg)
    {
        UseParameters(arg);
        var parameter = Context.Parameters.Single();
        var parameterInfo = parameter.Info;
        Assert.Equal("arg", parameterInfo.Name);
        Assert.Equal(arg, parameter.Value);
    }

    public static IEnumerable<object[]> GetData()
    {
        yield return new object[] {new ComplexClass("Value1")};
        yield return new object[] {new ComplexClass("Value2")};
    }

    public ComplexParameterSample(ITestOutputHelper output) :
        base(output)
    {
    }

    public class ComplexClass
    {
        public string Value { get; }

        public ComplexClass(string value)
        {
            Value = value;
        }
    }
}
```
<sup><a href='/src/Tests/Snippets/ComplexParameterSample.cs#L1-L40' title='File snippet `ComplexParameterSample.cs` was extracted from'>snippet source</a> | <a href='#snippet-ComplexParameterSample.cs' title='Navigate to start of snippet `ComplexParameterSample.cs`'>anchor</a></sup>
<!-- endsnippet -->


### UniqueTestName

Provided a string that uniquely identifies a test case.

Usage:

<!-- snippet: UniqueTestNameSample.cs -->
<a id='snippet-UniqueTestNameSample.cs'/></a>
```cs
using Xunit;
using Xunit.Abstractions;

public class UniqueTestNameSample :
    XunitContextBase
{
    [Fact]
    public void Usage()
    {
        var currentGuid = Context.UniqueTestName;

        Context.WriteLine(currentGuid);
    }

    public UniqueTestNameSample(ITestOutputHelper output) :
        base(output)
    {
    }
}
```
<sup><a href='/src/Tests/Snippets/UniqueTestNameSample.cs#L1-L19' title='File snippet `UniqueTestNameSample.cs` was extracted from'>snippet source</a> | <a href='#snippet-UniqueTestNameSample.cs' title='Navigate to start of snippet `UniqueTestNameSample.cs`'>anchor</a></sup>
<!-- endsnippet -->

Implementation:

<!-- snippet: UniqueTestName -->
<a id='snippet-uniquetestname'/></a>
```cs
string GetUniqueTestName(ITestCase testCase)
{
    var method = testCase.TestMethod;
    var name = $"{method.TestClass.Class.ClassName()}.{method.Method.Name}";
    if (!Parameters.Any())
    {
        return name;
    }

    var builder = new StringBuilder($"{name}_");
    foreach (var parameter in Parameters)
    {
        builder.Append($"{parameter.Info.Name}=");
        if (parameter.Value == null)
        {
            builder.Append("null_");
            continue;
        }

        builder.Append($"{parameter.Value}_");
    }

    builder.Length -= 1;

    return builder.ToString();
}
```
<sup><a href='/src/XunitContext/Context_TestName.cs#L26-L53' title='File snippet `uniquetestname` was extracted from'>snippet source</a> | <a href='#snippet-uniquetestname' title='Navigate to start of snippet `uniquetestname`'>anchor</a></sup>
<!-- endsnippet -->


## Global Setup

Xunit has no way to run code once prior to any tests executing. XUnitContext adds this feature via an attribute.

<!-- snippet: GlobalSetup.cs -->
<a id='snippet-GlobalSetup.cs'/></a>
```cs
using Xunit;

[GlobalSetUp]
public static class GlobalSetup
{
    public static void Setup()
    {
        Called = true;
    }

    public static bool Called;
}
```
<sup><a href='/src/Tests/GlobalSetup/GlobalSetup.cs#L1-L12' title='File snippet `GlobalSetup.cs` was extracted from'>snippet source</a> | <a href='#snippet-GlobalSetup.cs' title='Navigate to start of snippet `GlobalSetup.cs`'>anchor</a></sup>
<!-- endsnippet -->

Multiple setups can be defined as nested classes and classes in namespaces are supported.

Alternatives to this approach:

 * Using a [module initializer ](https://github.com/Fody/ModuleInit).
 * Having a single base class that all tests inherit from, and place any configuration code in the static constructor of that type.


## Security contact information

To report a security vulnerability, use the [Tidelift security contact](https://tidelift.com/security). Tidelift will coordinate the fix and disclosure.


## Icon

[Wolverine](https://thenounproject.com/term/wolverine/18415/) designed by [Mike Rowe](https://thenounproject.com/itsmikerowe/) from [The Noun Project](https://thenounproject.com/).
