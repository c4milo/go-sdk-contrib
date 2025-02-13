# GO Feature Flag GO Provider

GO Feature Flag provider allows you to connect to your GO Feature Flag instance.  

[GO Feature Flag](https://gofeatureflag.org) believes in simplicity and offers a simple and lightweight solution to use feature flags.  
Our focus is to avoid any complex infrastructure work to use GO Feature Flag.

This is a complete feature flagging solution with the possibility to target only a group of users, use any types of flags, store your configuration in various location and advanced rollout functionality. You can also collect usage data of your flags and be notified of configuration changes.


# GO SDK usage

## Install dependencies

The first things we will do is install the **Open Feature SDK** and the **GO Feature Flag provider**.

```shell
go get github.com/open-feature/go-sdk-contrib/providers/go-feature-flag
```

## Initialize your Open Feature provider

Despite other providers, this GO provider can be used with the **relay proxy** or used standalone
using the **GO Feature Flag module**.

### Using the relay proxy

If you want to use the provider with the **relay proxy** you should set the field `Endpoint` in the options.  
By default it will use a default `HTTPClient` with a **timeout** configured at **10000** milliseconds. You can change
this configuration by providing your own configuration of the `HTTPClient`.

#### Example
```go
options := gofeatureflag.ProviderOptions{
  Endpoint: "http://localhost:1031",
  HTTPClient: &http.Client{
    Timeout:   1 * time.Second,
  },
}
provider, _ := gofeatureflag.NewProvider(options)
```

### Using the GO module _(standalone version)_
If you want to use the provider in standalone mode using the GO module, you should set the field `GOFeatureFlagConfig`
in the options.

You can check the [GO Feature Flag documentation website](https://docs.gofeatureflag.org) to look how to configure the 
GO module. 

#### Example
```go
options := gofeatureflag.ProviderOptions{
  GOFeatureFlagConfig: &ffclient.Config{
      PollingInterval: 10 * time.Second,
      Context:         context.Background(),
      Retriever: &fileretriever.Retriever{
        Path: "../testutils/module/flags.yaml",
      },
    },
}
provider, _ := gofeatureflag.NewProvider(options)
```

## Initialize your Open Feature client

To evaluate the flags you need to have an Open Feature configured in you app.
This code block shows you how you can create a client that you can use in your application.

```go
import (
  // ...
  gofeatureflag "github.com/open-feature/go-sdk-contrib/providers/go-feature-flag/pkg"
  of "github.com/open-feature/go-sdk/pkg/openfeature"
)

// ...

options := gofeatureflag.ProviderOptions{
    Endpoint: "http://localhost:1031",
}
provider, err := gofeatureflag.NewProvider(options)
of.SetProvider(provider)
client := of.NewClient("my-app")
```

## Evaluate your flag

This code block explain how you can create an `EvaluationContext` and use it to evaluate your flag.


> In this example we are evaluating a `boolean` flag, but other types are available.
> 
> **Refer to the [Open Feature documentation](https://openfeature.dev/docs/reference/concepts/evaluation-api#basic-evaluation) to know more about it.**

```go
evaluationCtx := of.NewEvaluationContext(
    "1d1b9238-2591-4a47-94cf-d2bc080892f1",
    map[string]interface{}{
      "firstname", "john",
      "lastname", "doe",
      "email", "john.doe@gofeatureflag.org",
      "admin", true,
      "anonymous", false,
    })
adminFlag, _ := client.BoolValue(context.TODO(), "flag-only-for-admin", false, evaluationCtx)
if adminFlag {
   // flag "flag-only-for-admin" is true for the user
} else {
  // flag "flag-only-for-admin" is false for the user
}
```
