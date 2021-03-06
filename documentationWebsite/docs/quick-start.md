---
id: quick-start
title: Quick Start
sidebar_label: Quick Start
---

If you're familiar with ReScript and Apollo, there are just a few things you need to know and you're off to the races. Otherwise, I encourage you to skip to the [Queries](queries.md) section.

Other than some slightly different ergonomics, the underlying functionality is almost identical to the [official Apollo Client 3 docs](https://www.apollographql.com/docs/react/v3.0-beta/get-started/), so that should still be your primary, authoritative resource for working with Apollo.

In short, the only major difference is that the appropriate hook (`useQuery`, `useMutation`, etc.) is exposed as a `use` function on the module generated by `graphql-ppx`. Variables are always the last argument:

```reason
module TodosQuery = %graphql(`
  query Example ($userId: String!) {
    user(id: $userId) {
      id
      name
    }
  }
`)

@react.component
let make = () =>
  // The useQuery hook is automatically exposed on the module generated by graphql-ppx!
  switch ExampleQuery.use({userId: "1"}) {
    | {data: Some({users})} =>
    ...
  }
}
```

This library leverages records heavily which allows for very similar syntax to working with javascript objects and other benefits, but comes with a downside. You may need to annotate the types if you're using a record in a context where the compiler cannot infer what it is. The most common case is when using a T-last api like `Js.Promise`. For this reason we expose every type from the `ApolloClient.Types` module for convenience. Example:

```reason
  let queryResult = SomeQuery.use()

  // Destructuring works just like javascript!
  switch queryResult {
    | {loading: true} =>
      // Show loading
    | {data: Some(data), fetchMore} =>
      let onClick = _ => fetchMore()
      // Show data
  }


  apolloClient.query(~query=(module SomeQuery), ())
  |> Js.Promise.then(result => // Hover over the type and you can see it is an ApolloQueryResult.t__ok
    // Let's open the module so the record fields are accessible
    open ApolloClient.Types.ApolloQueryResult
    // ☝️ You don't have to go searching for a type, everything is accessible under ApolloClient.Types
    switch result {
    | Some(apolloQueryResult) =>
      // Now we can get at the `data` property of `apolloQueryResult`
      Js.log2("Got data!", apolloQueryResult.data)
    | Error(_) =>
      Js.log("Check out EXAMPLES/ for T-first promise solutions that don't have this problem!")
    }
  )
```

An alternative to this problem is to write some T-first promise helpers or use a promise library like `reason-promise`, `bs-future`, or `prometo`.
