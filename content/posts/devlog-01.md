+++
title = "Devlog 01: Rust Development (Ravana)"
date = "2022-06-21T21:27:52+05:30"
cover = ""
tags = ["rust"]
keywords = ["ravana"]
description = ""
showFullContent = false
readingTime = false
hideComments = false
draft = true
+++

## Deserializing Responses

Continuing development of my [Rust-based Reddit API wrapper](https://github.com/Cyanide4Dinner/ravana_reddit_api),
[`serde_json`](https://docs.serde.rs/serde_json/) effortlessly deserialized response string to map-based tree structure which I use to fill up my custom `Listing` & `Post` structures.

## Error Types

Also, refactoring made me rethink approach to errors. Firstly, if using enums (which I was) no value was added by declaring multiple error enums, if anything having multiple enums made returning errors a hassle as I had to conert between the error types. There might be some benefit on categorizing errors into groups so that the user can have more information and flexibility on handling the errors, especially for a library. However, enum variants seemed enough for one level of grouping. I'm still not able to reason for a second level of grouping by having different enum types too for errors. Hence, I've finalised on keeping a single enum as error type for the entire library.

For code quality and maintainability, the variants should not be abused which raises the question of when should we seperate out a error as a seperate enum variant?

One way is to create a variant for all those errors concerned with one functionality. Such as a type for errors based on request processing or connection and another for errors that arose due to invalid response or failure during serializing and parsing.

Another is to create different enum variants for different errors that could be *handled* differently. For instance, a developer might want on timing out on a request, that an alert pops requesting user to check their internet connection and grouping it with other errors such as those arising for server errors seems more appropriate.

## Iterating over Errors
A neat trick I came across while indulging in a problem I was facing tells us how to convert a `Vec<Result<T, E>` to `Result<Vec<T>, E>`. Credits to [this answer](https://stackoverflow.com/a/26370894/9897643).

This comes quite handy when using `map` and the closure could throw errors. If we want the pure vector and throw error if `map` throws error for any item, simply collect over `Result` and `from_iter` will do the rest for us.

```rust
let posts: Vec<Post> = Result::<Vec<Post>, Error>::from_iter::<Vec<Result<Post, Error>>>(
		values
		.into_iter()
		.map(|value| -> Result<Post, Error> { 
			value.convert_value_to_post() //Could throw error
		})
		.collect())?;
```

## Async Management
In my [`ravana`](https://github.com/Cyanide4Dinner/ravana) (Terminal-based Reddit client), I'll have HTTP
requests to fetch data from Reddit API, for which I'd like to use concurrency offered by Rust async. Since,
I'm using [Tokio](https://tokio.rs/), I was curious if I could reuse the same instatiated Tokio `Runtime` for
each async code execution, most of which will be API calls made by the `ravana_reddit_client` library.

Hence, I've resorted to storing my `Runtime` object in the `App` struct for executing my async calls inside.
This style is encouraged in [Tokio's blog](https://tokio.rs/tokio/topics/bridging).
