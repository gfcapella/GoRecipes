# Defer free

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

An alternative would be to return not only the file and error but to also include a free function (in this case calling Close).

Here is the example of the pattern ([playground](https://play.golang.org/p/dldi98bGyRP)):

    package main

    import (
	    "fmt"	
	    "os"	
    )

    func main() {
	    f, free, err := open()		
	    if err != nil {	
		    fmt.Println("Error: ", err.Error())
	    }
	    defer free()
	    fmt.Println("Using ", f)
    }

    // here the df function implements the defer free pattern
    type df func()

    func open() (*os.File, df, error) {
	    fmt.Println("Opening file...")
	    f, err := os.Open("filename")
	    return f, func() {	
		    if err == nil {
			    f.Close()			
		    }		
		    fmt.Println("Closing file, if open")
	    }, err
    }

This example feels a bit forced, given we all know how to use files, but it becomes very interesting when applied to heavy setup operations, or even to object locking (see [playground](https://play.golang.org/p/ffXnVzTmibl)):

	package main

    import (
	    "fmt"	
	    "os"	
    )

	type T struct {}
	type unlock func()
	
	func (t T) lock() (unlock, error) {
		fmt.Println("Locking")
		return func() {
			fmt.Println("Unlocking")
		}, nil
	}
	
    func main() {
	    unlock, err := T{}.lock()		
	    if err != nil {	
		    fmt.Println("Error: ", err.Error())
	    }
	    defer unlock()
    }