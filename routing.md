---
description: >-
  Routing refers to how an application's endpoints (URIs) respond to client
  requests.
---

# 🔌 Routing

## Paths

Route paths, combined with a request method, define the endpoints at which requests can be made. Route paths can be **strings** or **string patterns**.

**Examples of route paths based on strings**

```go
// This route path will match requests to the root route, "/":
app.Get("/", func(c *fiber.Ctx) error {
      return c.SendString("root")
})

// This route path will match requests to "/about":
app.Get("/about", func(c *fiber.Ctx) error {
    return c.SendString("about")
})

// This route path will match requests to "/random.txt":
app.Get("/random.txt", func(c *fiber.Ctx) error {
    return c.SendString("random.txt")
})
```

## Parameters

Route parameters are **named URL segments** that are used to capture the values specified at their position in the URL. The obtained values can be retrieved using the [Params](https://fiber.wiki/context#params) function, with the name of the route parameter specified in the path as their respective keys.

{% hint style="info" %}
The name of the route parameter must be made up of **characters** \(`[A-Za-z0-9_]`\).
{% endhint %}

**Example of define routes with route parameters**

```go
// Parameters
app.Get("/user/:name/books/:title", func(c *fiber.Ctx) error {
    fmt.Fprintf(c, "%s\n", c.Params("name"))
	  fmt.Fprintf(c, "%s\n", c.Params("title"))
    return nil
})
// Wildcard
app.Get("/user/*", func(c *fiber.Ctx) error {
    return c.SendString(c.Params("*"))
})
// Optional parameter
app.Get("/user/:name?", func(c *fiber.Ctx) error {
    return c.SendString(c.Params("name"))
})
```

{% hint style="info" %}
Since the hyphen \(`-`\) and the dot \(`.`\) are interpreted literally, they can be used along with route parameters for useful purposes.
{% endhint %}

```go
// http://localhost:3000/plantae/prunus.persica
app.Get("/plantae/:genus.:species", func(c *fiber.Ctx) error {
    fmt.Fprintf(c, "%s.%s\n", c.Params("genus"), c.Params("species"))
    return nil // prunus.persica
})
```

```go
// http://localhost:3000/flights/LAX-SFO
app.Get("/flights/:from-:to", func(c *fiber.Ctx) error {
    fmt.Fprintf(c, "%s-%s\n", c.Params("from"), c.Params("to"))
    return nil // LAX-SFO
})
```

## Middleware

Functions that are designed to make changes to the request or response are called **middleware functions**. The [Next](https://github.com/gofiber/docs/tree/34729974f7d6c1d8363076e7e88cd71edc34a2ac/context/README.md#next) is a **Fiber** router function, when called, executes the **next** function that **matches** the current route.

**Example of a middleware function**

```go
app.Use(func(c *fiber.Ctx) error {
  // Set some security headers:
  c.Set("X-XSS-Protection", "1; mode=block")
  c.Set("X-Content-Type-Options", "nosniff")
  c.Set("X-Download-Options", "noopen")
  c.Set("Strict-Transport-Security", "max-age=5184000")
  c.Set("X-Frame-Options", "SAMEORIGIN")
  c.Set("X-DNS-Prefetch-Control", "off")

  // Go to next middleware:
  return c.Next()
})

app.Get("/", func(c *fiber.Ctx) error {
  return c.SendString("Hello, World!")
})
```

`Use` method path is a **mount**, or **prefix** path, and limits middleware to only apply to any paths requested that begin with it.

## Grouping

If you have many endpoints, you can organize your routes using `Group`.

```go
func main() {
  app := fiber.New()

  api := app.Group("/api", cors())  // /api

  v1 := api.Group("/v1", mysql())   // /api/v1
  v1.Get("/list", handler)          // /api/v1/list
  v1.Get("/user", handler)          // /api/v1/user

  v2 := api.Group("/v2", mongodb()) // /api/v2
  v2.Get("/list", handler)          // /api/v2/list
  v2.Get("/user", handler)          // /api/v2/user

  app.Listen(3000)
}
```

