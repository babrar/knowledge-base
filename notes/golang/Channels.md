---
tags: [Notebooks/Golang/Tour Notes]
title: Channels
created: '2021-01-10T02:05:22.342Z'
modified: '2021-01-12T06:38:24.991Z'
---

# Channels
Acts a means of communication between multiple goroutines (or threads).
## Unbuffered
For unbuffered channels, the sender blocks until the receiver has received the value. The receiver blocks until there is data to be received.
Therefore, the following piece of code does not send and receive the data in an infinite loop:
```go
go func() {
    for {
        select {
        case ch1 <- 1:
            println("sent 1 ")
        case c := <-ch1:
            println(" received ", c)
        }
        time.Sleep(time.Second)
    }
}()
```
This has the subtle implication that sending and receiving over an unbuffered channle has to take place on separate threads (goroutines). Full discussion [here](https://stackoverflow.com/questions/47525250/in-the-go-select-construct-can-i-have-send-and-receive-to-unbuffered-channel-in).

Note: if sender s1 and s2 write to the channel, and only s1's value was received then, s2 will still be blocked.  

## Buffered
For buffered channels, the sender only blocks until the value is copied to the buffer. if the buffer is at max capacity, the sender blocks till there is space. By contrast, receiver is blocked until there are items in the buffer. This behaviour is analogous to the producer-consumer buffer from ECE 252.


### Closing a channel
Channles normally don't require closing. The only scenario in which it is necessary to close a channel is when the receiver must be told that there are no more values incoming. Only the sender may close the channel.

If a channel is closed, then `ok` in the following code will evaluate to `false`, else it will be `true` if channel is open.
```go
v, ok := <-ch
``` 
Here, `ok` is an additional param that can be passed while receiving a value out of a channel.

## Solution to Exercise: Equivalent Binary Trees
```go
package main

import (
	"fmt"
	"golang.org/x/tour/tree"
)

// Walk walks the tree t sending all values
// from the tree to the channel ch.
func Walk(t *tree.Tree, ch chan int) {
	if t == nil {
		return
	}
	// peform inorder traversal
	Walk(t.Left, ch)
	ch <- t.Value
	Walk(t.Right, ch)
}

// Same determines whether the trees
// t1 and t2 contain the same values.
func Same(t1, t2 *tree.Tree) bool {
	if t1 == nil && t2 == nil {
		return true
	}
	ch1 := make(chan int)
	ch2 := make(chan int)

	go Walk(t1, ch1)
	go Walk(t2, ch2)

	for i := 0; i < 10; i++ {
		if <-ch1 != <-ch2 {
			return false
		}
	}

	return true
}

func main() {
	res:= Same(tree.New(1), tree.New(2))
	fmt.Println(res)
}
```


