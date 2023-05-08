# Producer-Consumer Scenario

The producer reads in tweets from a mockstream and a consumer is processing the data to find out whether someone has tweeted about golang or not. The task is to modify the code inside `main.go` so that producer and consumer can run concurrently to increase the throughput of this program.

## Expected results:
Before: 
```shell
davecheney      tweets about golang
beertocode      does not tweet about golang
ironzeb         tweets about golang
beertocode      tweets about golang
vampirewalk666  tweets about golang
Process took 3.580866005s
```

After:
```shell
davecheney      tweets about golang
beertocode      does not tweet about golang
ironzeb         tweets about golang
beertocode      tweets about golang
vampirewalk666  tweets about golang
Process took 1.977756255s
```

## Solution:

### Changes:

#### Producer

**Before:**
The tweets read from the mock stream one at a time were added to a *slice* and wait to be returned only when there is a EOF.
```go
func producer(stream Stream) (tweets []*Tweet) {
	for {
		tweet, err := stream.Next()
		if err == ErrEOF {
			return tweets
		}

		tweets = append(tweets, tweet)
	}
}
```
**After:**
The tweets read from the mock stream one at a time were added to a *channel* which **doesn't wait** to be returned when there is a EOF. With the introduction of channel, they are *readily avaiable to be consumed* by the producer.

```go
func producer(stream Stream, tweets chan *Tweet){
	for {
		tweet, err := stream.Next()
		if err == ErrEOF {
			close(tweets)
			return
		}

		tweets <- tweet
	}
}
```

The producer could now be invoked as a gorotine passing a channel of type `*Tweet`.

```go
tweets := make(chan *Tweet)

// Producer
go producer(stream, tweets)
```

#### Consumer

**Before:**
The consumer was accepting a slice of all tweets being produced.
```go
func consumer(tweets []*Tweet) {
```
**After:**
The consumer is accepting a channel of one tweet at a time being produced.

```go
func consumer(tweets chan *Tweet) {
```