# Devskiller programming task sample - C#/.NET Core

## Introduction

With [Devskiller.com](https://devskiller.com) you can assess your candidates'
programming skills as a part of your recruitment process. We have found that
programming tasks are the best way to do this and have built our tests
accordingly. The way our test works is your candidate is asked to modify the
source code of an existing project.

During the test, your candidates have the option of using our browser-based
code editor and can build the project inside the browser at any time. If they
would prefer to use an IDE they are more comfortable with, they can also
download the project code or clone the project’s Git repository and work
locally.

You can check out this short video to see the test from the [candidate's
perspective](https://devskiller.zendesk.com/hc/en-us/articles/360019534639-How-the-TalentScore-test-looks-like-from-the-candidate-perspective).

This repo contains a sample project for C#/.NET Core and below you can
find a detailed guide for creating your own programming project.

**Please make sure to read our [Getting started with programming
projects](https://devskiller.zendesk.com/hc/en-us/articles/360019531059-Getting-started-with-Programming-Tasks) guide first**

## Automatic assessment
It is possible to automatically assess a solution posted by the candidate.
Automatic assessment is based on Tests results and Code Quality measurements. 

All unit tests that are executed during the build will be detected by the Devskiller platform. 

There are two kinds of unit tests:

1. **Candidate tests** - unit tests that are visible for the candidate during the test. These should be used to do only basic verification and are designed to help the candidate understand the requirements. Candidate tests **WILL NOT** be used to calculate the final score.
2. **Verification tests** - unit tests that are hidden from the candidate during the assessment. Files containing verification tests will be added to the project after the candidate finishes the test and will be executed during verification phase. Verification test results **WILL** be used to calculate the final score.
After the candidate finishes the test, our platform builds the project posted by the candidate and executes the verification tests and static code analysis.

## Technical details for .NET Core support
Any language of .NET platform can be used **(C#, F#, VisualBasic)**, though this article focus on C# only. Currently supported .NET Core version by DevsSkiller platform can be checked [here](https://devskiller.com/runtime-info/).
`Dotnet command` will be used to build, restore packages and test your code. You can use any unit-testing framework like **NUnit, XUnit or MSTest**. 
Please refer to [msdn](https://docs.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-dotnet-test) for details about using testing frameworks.
Don't forget to add reference to `Microsoft.NET.Test.Sdk` in your test projects.


## DevSkiller project descriptor 

Programming task can be configured with the DevSkiller project descriptor file. Just create a `devskiller.json` file and place it in the root directory of your project. Here is an example project descriptor:
```
{
  "readOnlyFiles" : [ "CalculatorSample.sln" ],
  "verification" : {
    "testNamePatterns" : [".*VerifyTests.*"],
    "pathPatterns" : ["CalculatorSample.VerifyTests/**"],
    "overwrite" : {
	 "CalculatorSample.VerifyTests/CalculatorSample.sln" : "CalculatorSample.sln"
    }
  }
}
```
You can find more details about devskiller.json descriptor in our [documentation](https://devskiller.zendesk.com/hc/en-us/articles/360019530419-Programming-task-project-descriptor)

In example above, by setting `readOnlyFiles` field with a solution file, we make sure candidate won't be able to edit it. **It's important during phase of verification tests execution, don't forget to add it!**
- `testNamePatterns` - an array of RegEx patterns which should match all the test names of verification tests. Test names should contain: `[namespace_name].[Class_name].[method_name]` . In our sample project, all verification tests are inside VerifyTests  class, so the following pattern will be sufficient:
```
"testNamePatterns"  : [".*VerifyTests.*"]
```
- `pathPatterns` - an array of GLOB patterns which should match all the files containing verification tests. All the files that match defined patterns will be deleted from candidates projects and will be added to the projects during the verification phase. 
```
"pathPatterns" : ["CalculatorSample.VerifyTests/**"]
```

Because files with verification tests will be deleted from candidates projects, you need to make sure, that during final solution build, Devskiller platform will be aware of them.
To make that happen, you must point which solution file should be overwritten - you want solution from **CalculatorTask\CalculatorSample.VerifyTests** folder to overwrite the **root** solution file
```
"CalculatorSample.VerifyTests/CalculatorSample.sln" : "CalculatorSample.sln"
```


## Hints

1. Remember that you aren't bounded to unit tests. You can do some integration tests, in example .net core mvc apps are easy to instantiate within single test scope, try to use that.
2. Make sure, each test is self-runnable and independent of external components like: native system libraries, external databases, etc.
3. Avoid using external libraries in your source. You will never know on what operating system your code will be executed. If you need some external libraries please reference them as NuGet packages. This will make sure, you're code will behave on Devskiller platform in the same way it behaves on your environment.
4. When needed and applicable, consider usage of in-memory database engine emulation when aplicable. It's fast to run and easy to use.
5. Don't forget that you can utilize .NET framework references in .NET Core projects if needed - do it with caution though (maybe only in verification tests? - its up to you), you want to test candidate against .NET Core... don't you? Otherwise consider using [MSBuild](https://devskiller.zendesk.com/hc/en-us/articles/360019469800-C-NET-with-MSBuild)
6. Try to make test names clear and as short as possible. Long names will be harder to read for recruiters and can be confusing. Some test-parameter-injection methods tend to generate complex text output when executing tests. Make sure to check, if the test output looks good.
7. Remember to describe clearly the task instructions in `README.md` file.
8. When leaving gaps in code to be filled by candidate, consider throwing `NotImplementedException`, rather than in ex. returning `null`, `false`, `0`, etc. in your methods. As those returned values, dependent on logic, could be some edge cases, besides, the unit tests execution will instantly fail even before checking assertions, so the candidadte will be 100% sure where he is expected to make code changes.
10. You want candidate to start working on the task as soon as possible, not to struggle with configuration. Project delivered for candidate should be compilable and working without any configuration needed. It should only fail tests on execution.
