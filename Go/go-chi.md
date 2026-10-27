`chi` is a lightweight router for building Go HTTP services.
- built on the new `context` package (handles singnaling, canceling & request-scoped values across handler chain)

import: `go get github.com/go-chi/chi/v5`
```running a simple server
package main

import (
	"net/http"
	
	"github.com/go-chi/chi/v5"
	"github.com/go-chi/chi/v5/middleware"
)

func main() {
	r := chi.NewRouter()
	r.Use(middleware.Logger)
	r.Get("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("Hello World!"))
	})
	
	http.ListenAndServe(":3000", r)
}
```

get param from request: `dateParam := chi.URLParam(r, "date")`
