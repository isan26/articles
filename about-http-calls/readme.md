# The day I created a dependency to fix my dependencies
👉 https://www.npmjs.com/package/reqmate

## My problem with dependencies.
I'm gonna be honest and expose my opinion right at the start. I'm aware that more than one colleague is going to disagree, and that's fine.

I don't like to have dependencies in my projects. Having no dependencies at all is very impractical, but dependencies come at a cost, so every project is a constant struggle where you have to evaluate the pros and cons of using external tools. If you don't use any, you will have to deal with already solved problems over and over again, and on top of that, you cannot compete with already established solutions with years ahead of you — and there's no reason to do that anyway. But on the other hand, it's very simple to pollute your system with tons of libraries that you don't really need, that add complexity and risk. I've worked on codebases that use libraries for simple tasks like iterating arrays and other stuff that can be done easily with just the language.

More often than not, I see engineers using Axios for simple HTTP calls, and we don't need Axios to do a simple GET call. Moreover, the library is scattered across the application, making it impossible to change and making us completely dependent on a third-party tool.

## Did I created what I swore to destroy?

I'm a big fan of using the language's capabilities over libraries when possible. That means:

1. Always use `fetch` over `axios`.
1. Building an abstraction within my app to handle HTTP calls.

But here's the problem:
I found myself writing the same abstraction over and over again. I didn’t want to deal with raw fetch every time I was writing business logic — I wanted it abstracted away.

So I created a library that was:

1. Simple to use
1. Reusable

And capable of doing more than just sending HTTP requests. Now, instead of just doing simple HTTP calls, you have a lot more features justifying the install.

### Presenting Reqmate
Yes, I coded a library, but let's analyze it first.
When doing HTTP calls, I found out that I usually needed to:

1. Cache requests.
2. Poll and retry.
3. Clean up the response.

So for my own http library, I wanted to have:
1. No dependencies (lol), just use fetch API.
2. Caching, as simple as possible.
3. Polling, multiple strategies.
4. Extensibility - show me the menu but let me cook my own meal too.
5. Easy of use - no degrees required, please.

An example with caching and polling:

```typescript
// Basic example
const getExample = await reqmate.get("/product?id=666").send();

// More complex
const response = await reqmate
    .get('https://jsonplaceholder.typicode.com/todos/3')
    .setCache(30000) // Cache will store for 30 seconds
    .setRetry({
        type: 'polling',
        maxRetries: 3,
        onResponse: (response, done) => console.log({response}),
    })
    .setParser(async (response: Response) => {
        const { title, completed } = await response.json();
        return `${title} is ${completed ? "Completed" : "Not Completed"}`;
    })
    .send()
```

As simple as that — you start by declaring the HTTP call you want to perform and end with `send()`. But in the middle, you can, if you need to, add extra configuration. That way, you can tell the library to do polling, caching, and even parse the response. Note the use of callbacks and how you can tap into this and inject different types of behaviors — it's simple, and we are used to doing this.

On top of this, you can use the solutions that are already built-in, or you can create your own. In one case, I created a MongoDB integration to store the cache in a MongoDB collection on a project using NestJS.

Check the full documentation at:
https://www.npmjs.com/package/reqmate
And feel free to connect with me for any feedback you may have.


## My advice
> Abstractions should not depend on details, details should depend on abstractions.

Create a layer in your project that internally uses the library, so other parts of your code don't depend on the detail of what library you're using. This will protect your team from having to know how to use the library and from any changes you may want to make in the future.

