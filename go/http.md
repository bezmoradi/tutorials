# Go > HTTP

Create an HTTP server in Go is as simple as this:

```go
package main

import (
	"fmt"
	"net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprint(w, "Hello World")
}

func main() {
	http.HandleFunc("/", handler)
	http.ListenAndServe(":8080", nil)
}
```

The `HandleFunc` function sets up a handler function for a specific URL pattern (`/` in this case). When a request is made to the root path (`/`), the provided handler function is executed. The handler function takes two arguments: `w` is a `ResponseWriter` that allows you to construct an HTTP response, and `r` is an `Request` that contains information about the incoming request.

## Why Is `Request` A Pointer?

This `Request` holds information about the incoming HTTP request, like the URL, headers, and other details. Since this struct can potentially be **large** and complex OR is intended to be **mutable** (You'll see how next), it is more efficient to pass it by pointer. When you pass a pointer, you're not making a copy of the entire struct, you just send a reference to it. One common scenario where you might want to mutate the `Request` is when you're working with a middleware:

```go
package main

import (
	"context"
	"fmt"
	"net/http"
)

const validApiKey = "jlkhlds8783742%klkjlfd"

type user struct {
	name string
}

func authMiddleware(next http.HandlerFunc) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		apiKey := r.Header.Get("X-API-Key")
		if apiKey != validApiKey {
			http.Error(w, "Unauthorized", http.StatusUnauthorized)
			return
		}

		authenticatedUser := user{name: "John Doe"}
		r = r.WithContext(context.WithValue(r.Context(), "user", authenticatedUser))
		next(w, r)
	}
}

func handler(w http.ResponseWriter, r *http.Request) {
	authenticatedUser, ok := r.Context().Value("user").(user)
	if !ok {
		http.Error(w, "Internal Server Error", http.StatusInternalServerError)
		return
	}
	fmt.Fprintf(w, "Hello, %s! You are authenticated.", authenticatedUser.name)
}

func main() {
	http.HandleFunc("/", authMiddleware(handler))
	http.ListenAndServe(":8080", nil)
}
```

In this example, the `AuthMiddleware` checks for the presence of the `X-API-Key` header in the request. If the header is present and has the correct value, it considers the request authenticated. It then attaches information about the authenticated user to the request context using `WithContext`.

```text
curl --location 'http://localhost:8080' \
--header 'X-API-Key: jlkhlds8783742%klkjlfd'
```

By sending the above curl request, we would get `Hello, John Doe! You are authenticated.` but if we get rid of out custom header while sending the request like so:

```text
curl --location 'http://localhost:8080'
```

We will get `Unauthorized` response.

## Why Isn't `ResponseWriter` A Pointer?

`ResponseWriter` is designed to be used by multiple parts of your code at the same time. If it were a pointer, each user would have to worry about whether they're modifying the same underlying data. On the other hand, by using a non-pointer, it's easier to make it thread-safe because each part using it gets its own independent copy.

# How to Send HTTP Requests in Go?

In order to send an HTTP request to a remote server, we can use the following approaches using the built-in `http` package:

```go
package main

import (
	"fmt"
	"io"
	"log"
	"net/http"
)

func main() {
	client := http.Client{}
	res, err := client.Get("https://go.dev")
	if err != nil {
		log.Fatal(err)
	}

	bytes, err := io.ReadAll(res.Body)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(string(bytes))
}
```

We are using `client.Get` method to directly make a GET request. In this case, the `http.Client` automatically creates an HTTP GET request and sends it to the specified URL. This is a convenient way to make simple GET requests. The other way is follows:

```go
func main() {
	client := http.Client{}
	request, err := http.NewRequest("GET", "https://go.dev", nil)
	if err != nil {
		log.Fatal(err)
	}

	res, err := client.Do(request)
	if err != nil {
		log.Fatal(err)
	}

	bytes, err := io.ReadAll(res.Body)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(string(bytes))
}
```

This approach though allows you to have more control over the request, such as adding headers or using other HTTP methods.
