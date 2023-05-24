```go
package main

import (
	"encoding/json"
	"net/http"
	"time"
)

const (
	address = "0.0.0.0:8080"
)

type Response struct {
	field1 string `example:"someValue"` //using struct tags to give some metaData about field
}

type Request struct {
	field1 string `example:"someValue"`
}

type Handler struct {
}

func NewHandler() *Handler {
	return &Handler{}
}

func (this *Handler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	switch method := r.Method; {
	case method == "GET":
		w.Header().Set("Access-Control-Allow-Origin", "*")            //indicate whether response can be shared with requesting code from given origin
		w.Header().Set("Access-Control-Allow-Header", "Content-Type") //indicates which http headers can be used during actual request
		w.Header().Set("Content-Type", "application/json; charset=UTF-8")
		//business logic here...
		//make a response
		resp, err := json.Marshal(Response{})
		if err != nil {
			http.Error(w, err.Error(), http.StatusBadRequest)
			return
		}
		w.Write(resp)
		break
	case method == "PUT":
		break
	case method == "POST":
		req := &Request{}
		err := json.NewDecoder(r.Body).Decode(req)
		if err != nil {
			http.Error(w, err.Error(), http.StatusBadRequest)
		}
		////business logic here... do something with req....
		//make a response
		resp, err := json.Marshal(Response{})
		if err != nil {
			http.Error(w, err.Error(), http.StatusBadRequest)
		}
		w.Write(resp)
		break
	case method == "DELETE":
		http.Error(w,"DELETE not implemented",http.StatusNotImplemented)
		return
		break
	case method == "PATCH":
		http.Error(w,"PATCH not implemented",http.StatusNotImplemented)
		return
		break
	case method == "OPTIONS":
		http.Error(w,"OPTIONS not implemented",http.StatusNotImplemented)
		return
		break
	}
}
func main() {
	s := &http.Server{
		Addr:           address,
		Handler:        NewHandler(),
		ReadTimeout:    time.Duration(10 * int64(time.Second)),
		WriteTimeout:   time.Duration(60 * int64(time.Second)),
		MaxHeaderBytes: 1 << 20, //1 mb
	}

	s.ListenAndServe()
}

```
