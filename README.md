```go
package main

import (
    "log"
    "net/http"
    "os"

    "github.com/gorilla/mux"
)

// AuthenticatedHandler checks if the request is authenticated
func AuthenticatedHandler(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        // Implement authentication logic here
        // For example:
        authHeader := r.Header.Get("Authorization")
        if authHeader != "Bearer secret_token" {
            http.Error(w, "Unauthorized", http.StatusUnauthorized)
            return
        }
        next(w, r)
    }
}

func main() {
    // Load sensitive information from environment variables
    password := os.Getenv("PASSWORD")
    if password == "" {
        log.Fatal("PASSWORD environment variable is not set")
    }

    // Create a router
    r := mux.NewRouter()

    // Restrict access to the /admin/ path
    adminRouter := r.PathPrefix("/admin").Subrouter()
    adminRouter.Use(AuthenticatedHandler)
    adminRouter.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        // Only allow GET and HEAD methods
        if r.Method != http.MethodGet && r.Method != http.MethodHead {
            http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
            return
        }
        // ...
    })

    log.Fatal(http.ListenAndServe(":8080", r))
}
```