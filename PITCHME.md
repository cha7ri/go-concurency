#HSLIDE

# Golang Casablanca Meetup;
## Golang concurency 

Tarik Ghallab, Schibsted Morocco

#HSLIDE

## Why do we need Concurency?

#HSLIDE

- CPU can ran only one instruction at given moment.
- I/O operations takes time to complite (HDD and NET)
- The time unit very is valuable than gold!

#HSLIDE

### Concurency Is not parallelism!
- Concurrency is about dealing with lots of things at once.
- Parallelism is about doing lots of things at once.
- Concurrency provides a way to structure a solution to solve a problem that may (but not necessarily) be parallelizable.

#HSLIDE

### Some definitions
Lets understand a process first:
  - A simple definition is: a process is an instance of the a program in memory.
  - A process can have sub process which we call child.
  - In each process we have one or many threads and they can **share the memory**
And why do we need many threads?

#HSLIDE

#### Concurency in Golang
- Go routines
  Let's start with a boring example to understand better:
  
#HSLIDE

~~~
package main
  import (
      "fmt"
      "math/rand"
      "time"
  )
  func main() {
      boring("boring!")
  }

  func boring(msg string) {
      for i := 0; ; i++ {
          fmt.Println(msg, i)
          time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
      }
  }
~~~
#HSLIDE

#### Run on the background
- Now the same function but it will not block the caller so the main function can continue

#HSLIDE
~~~
package main
  import (
      "fmt"
      "math/rand"
      "time"
  )
  func main() {
      go boring("boring!")
  }

  func boring(msg string) {
      for i := 0; ; i++ {
          fmt.Println(msg, i)
          time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
      }
  }
~~~

#HSLIDE
- When the main function exists the Goroutine will be killed let's for it: 
~~~
package main
  import (
      "fmt"
      "math/rand"
      "time"
  )
  func main() {
      go boring("boring!")
      fmt.Println("I'm listening.")
      time.Sleep(2 * time.Second)
      fmt.Println("You're boring; I'm leaving.")      
  }

  func boring(msg string) {
      for i := 0; ; i++ {
          fmt.Println(msg, i)
          time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
      }
  }
~~~

#HSLIDE
## So what is a Goroutine
- It's an independently executing function, launched by a go statement.
- It Has its own stack, that can be allocated dynamically.
- The Golang runtime knows the few registers that should save when it sechedule a Goroutine, saw it's very sheap.
- We can launch thousdans of Goroutines 
- Goroutines are multiplexed on threads, so we can have only one thread running many goroutines


#HSLIDE
## Channels
~~~
package main
  import (
      "fmt"
      "math/rand"
      "time"
  )
  func main() {
      c := make(chan string)
      go boring("boring!", c)
      for i := 0; i < 5; i++ {
        fmt.Printf("You say: %q\n", <-c)
      }
      fmt.Println("You're boring; I'm leaving.")      
  }

  func boring(msg string, c chan string) {
    for i := 0; ; i++ {
        c <- fmt.Sprintf("%s %d", msg, i)
        time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
        
      }
  }
~~~
#HSLIDE

- When main sends data using the channel `c <-` it wait for a value to be sent. 
- Similarly, when the boring function executes `c <–` value, it waits for a receiver to be ready.
- Channels are used for comunication and synchronization.

#HSLIDE
## Select

The select statement provides another way to handle multiple channels. 
It's like a switch, but each case is a communication: 
- All channels are evaluated. 
- Selection blocks until one communication can proceed, which then does. 
- If multiple can proceed, select chooses pseudo-randomly. 
- A default clause, if present, executes immediately if no channel is ready.

#HSLIDE
~~~
package main
  import (
      "fmt"
      "math/rand"
      "time"
  )
  func main() {
	  c1 := make(chan string)
	  c2 := make(chan string)	

	  go boring("boring 1!", c1) 
	  go boring("boring 2!", c2) 
	  select {
		case v1 := <-c1:
			fmt.Printf("Received %v from c1\n", v1)
		case v2 := <-c2:
			fmt.Printf("Received %v from c2\n", v2)
		case c2 <- "You say boring":
			fmt.Printf("Sent 'You say boring' to c2\n")
		case <-time.After(time.Millisecond * 3):
			fmt.Printf("no one was ready to communicate, You're boring; I'm leaving.\n")
	}    
  }

  func boring(msg string, c chan string) {   
	c <- fmt.Sprintf("%s", msg)
	time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)		
  }
~~~
#HSLIDE

## Thank you

Questions?

