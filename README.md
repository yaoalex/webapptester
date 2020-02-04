# webapptester
Command line tool that generates boilerplate unit tests for your http handlers. Get a head start in writing your unit tests with these auto generated (Table Driven) tests. Now this doesn't completely unit test your code but you can easily add more test cases and variables to your test structure. This even parses your file for Mux router variables, and sets them up in the table for you! Each test for the handler is created separately, so it is a true unit test.  

## motivation
At one of my previous internships, I built a lot of endpoints in Go. So naturally this meant I wrote a lot of unit tests for these endpoints. What I realized when writing these unit tests was that there is a lot of repetitive actions in building these tests. Tasks like creating a http request for your test case, setting up all the necessary variables to run your test, and just structuring the test cases in general.. seemed very repetitive.  

## how-to 
This project has no outside dependencies (other than Go)

Just run

`go get github.com/yaoalex/webapptester`

and

`go install`

To use the tool simply run 

`webapptester <file you want to test>`

It will parse the file and look for any http handler functions.  
If there are testable functions found, it will try creating the test file in yourfile_test.go and if that file already exists it will prompt to store the file in a new location.  

## example

Here is a file containing a simple Get request

```
package exampleapi

import (
	"encoding/json"
	"net/http"

	"github.com/gorilla/mux"
)

// Get does nothing
func Get(w http.ResponseWriter, r *http.Request) {
	params := mux.Vars(r)
	status := params["return_status"]
	if status == "200" {
		w.WriteHeader(http.StatusOK)
		json.NewEncoder(w).Encode("OK")
	} else {
		w.WriteHeader(http.StatusBadRequest)
	}
}
```

Here is what's generated by webapptester
```
package exampleapi

import (
	"net/http"
	"net/http/httptest"
	"testing"

	"github.com/gorilla/mux"
)

// THIS IS GENERATED CODE BY WEBAPPTESTER
// you will need to edit this code to suit your needs

func TestGet(t *testing.T) {
	testCases := []struct {
		Name           string
		ExpectedStatus int
		MuxVars        map[string]string
	}{
		{
			Name:           "Get: valid test case",
			ExpectedStatus: http.StatusOK,
			MuxVars: map[string]string{
				"return_status": "valid_value",
			},
		},
		{
			Name:           "Get: invalid test case",
			ExpectedStatus: http.StatusBadRequest,
			MuxVars: map[string]string{
				"return_status": "invalid_value",
			},
		},
	}

	for _, tc := range testCases {
		t.Run(tc.Name, func(t *testing.T) {
			req, err := http.NewRequest("GET", "/test", nil)
			if err != nil {
				t.Fatal(err)
			}

			rr := httptest.NewRecorder()
			handler := http.HandlerFunc(Get)

			req = mux.SetURLVars(req, tc.MuxVars)

			handler.ServeHTTP(rr, req)
			if status := rr.Code; status != tc.ExpectedStatus {
				t.Errorf("handler returned wrong status code: got %v want %v",
					status, tc.ExpectedStatus)
			}
		})
	}
}
```

So much time saved :) 
