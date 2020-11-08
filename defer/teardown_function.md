# Defer a teardown function

Many examples on the web present the following pattern, when using `defer`:

    func Contents(filename string) (string, error) {
        f, err := os.Open(filename)
        if err != nil {
            return "", err
        }
        defer f.Close()  // f.Close will run when we're finished.
        
        // more things happen here to the file
    }

This specific snippet belongs to the [Effective Go](https://golang.org/doc/effective_go.html#defer) document.

A key issue with the pattern, is that the dependency between Open and Close is implicit and relies on the good memory of the developer.

An alternative would be to return not only the file and error but to include the teardown function (in this case Close).

Here is the example of the pattern ([playground](https://play.golang.org/p/dldi98bGyRP)):

    package main

    import (
	    "fmt"	
	    "os"	
    )

    func main() {
	    f, teardown, err := open()	
	    defer teardown()	
	    if err != nil {	
		    fmt.Println("Error: ", err.Error())
	    }	
	    fmt.Println("Using ", f)
    }

    // here the close function implements the teardown pattern
    type close func()

    func open() (*os.File, close, error) {
	    fmt.Println("Opening file...")
	    f, err := os.Open("filename")
	    return f, func() {	
		    if err == nil {
			    f.Close()			
		    }		
		    fmt.Println("Closing file, if open")
	    }, err
    }