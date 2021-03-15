# Logging in Golang

## Native Go Logging

Go offers native logging library:

### Logging to Stdout/Stderr

```text
$ mkdir logging-frameworks && cd logging-frameworks
$ cat > native.go <<EOF
package main

import "log"

func main() {
	log.Println("Hello world!")
	log.Debug("Debugging is fun")
	log.Info("Something is interesting!")
	log.Warn("Something bad may come!")
	log.Error("Something must be wrong!")
	log.Fatal("OMG!")
}
EOF

$ go run native.go
INFO[0000] Hello world!
INFO[0000] Something is interesting!
WARN[0000] Something bad may come!
ERRO[0000] Something must be wrong!
FATA[0000] OMG!
exit status 1
```

The code above prints the text "Hello world!" to the standard error, but it also includes the date and time, which is handy for filtering log messages by date.

### Logging to a File

```text
package main

import (
	"log"
	"os"
)

func main() {
	file, err := os.OpenFile("logs.txt", os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0666)
	if err != nil {
		log.Fatal(err)
	}

	log.SetOutput(file)

	log.Println("Hello world!")
}
```

A file named `logs.txt` will be created with content:

```text
2021/03/14 22:10:32 Hello world!
```

You can basically output your logs to any destination that implements the `io.Writer` interface, so you have a lot of flexibility when deciding where to log messages in your application.

### Creating custom loggers

```text
package main

import (
	"log"
	"os"
)

var (
	WarningLogger *log.Logger
	InfoLogger    *log.Logger
	ErrorLogger   *log.Logger
)

func init() {
	file, err := os.OpenFile("logs.txt", os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0666)
	if err != nil {
		log.Fatal(err)
	}

	InfoLogger = log.New(file, "INFO: ", log.Ldate|log.Ltime|log.Lshortfile)
	WarningLogger = log.New(file, "WARNING: ", log.Ldate|log.Ltime|log.Lshortfile)
	ErrorLogger = log.New(file, "ERROR: ", log.Ldate|log.Ltime|log.Lshortfile)
}

func main() {
	InfoLogger.Println("Starting the application...")
	InfoLogger.Println("Something noteworthy happened")
	WarningLogger.Println("There is something you should know about")
	ErrorLogger.Println("Something went wrong")
}
```

Output in `logs.txt`:

```text
INFO: 2021/03/14 22:14:06 custom-logger.go:26: Starting the application...
INFO: 2021/03/14 22:14:06 custom-logger.go:27: Something noteworthy happened
WARNING: 2021/03/14 22:14:06 custom-logger.go:28: There is something you should know about
ERROR: 2021/03/14 22:14:06 custom-logger.go:29: Something went wrong
```

### Log flags

There are log flags we can use to enhance the logs:

```text
const (
    Ldate         = 1 << iota     // the date in the local time zone: 2009/01/23
    Ltime                         // the time in the local time zone: 01:23:23
    Lmicroseconds                 // microsecond resolution: 01:23:23.123123.  assumes Ltime.
    Llongfile                     // full file name and line number: /a/b/c/d.go:23
    Lshortfile                    // final file name element and line number: d.go:23. overrides Llongfile
    LUTC                          // if Ldate or Ltime is set, use UTC rather than the local time zone
    Lmsgprefix                    // move the "prefix" from the beginning of the line to before the message
    LstdFlags     = Ldate | Ltime // initial values for the standard logger
)
```

So:

```text
$ cat > native.go <<EOF
package main

import (
	"log"
	"os"
)

const (
	DEBUG = "DEBUG"
	ERROR = "ERROR"
	INFO  = "INFO"
)

func main() {
	logger := log.New(os.Stdout, "", log.Ldate|log.Ltime|log.Lshortfile)

	logger.Printf("%s: Hello world!", INFO)
	logger.Printf("%s: Hello world!", DEBUG)
	logger.Printf("%s: Hello world!", ERROR)
}
EOF

$ go run native.go
2021/03/14 22:26:34 native-logging.go:17: INFO: Hello world!
2021/03/14 22:26:34 native-logging.go:18: DEBUG: Hello world!
2021/03/14 22:26:34 native-logging.go:19: ERROR: Hello world!
```

## Community Logging Frameworks

Surprisingly, there is a long list of [community logging frameworks](https://github.com/avelino/awesome-go#logging), with different design philosophies. Let's name some of the popular ones

* [glog](https://github.com/golang/glog) - Leveled execution logs for Go.
* [logrus](https://github.com/Sirupsen/logrus) - Structured logger for Go.
* [zap](https://github.com/uber-go/zap) - Fast, structured, leveled logging in Go.

Let's try out `logrus`.

### logrus

#### Get Started

As logrus is fully compliant to Golang's standard logging APIs, it can be very easy to get started:

```text
$ cat > logrus-getstarted.go <<EOF
package main

import (
  log "github.com/sirupsen/logrus"
)

func main() {
    log.Println("Hello world!")
}
EOF

# Starting from golang v1.16, let's enable module if not running in GOPATH
$ go mod init github.com/brightzheng100/logging-frameworks
$ go mod tidy
go: finding module for package github.com/sirupsen/logrus
go: found github.com/sirupsen/logrus in github.com/sirupsen/logrus v1.8.1
go: downloading golang.org/x/sys v0.0.0-20191026070338-33540a1f6037

$ go run logrus-getstarted.go
INFO[0000] Hello world!
```

#### Logging in JSON

```text
$ cat > logrus-getstarted.go <<EOF
package main

import (
	log "github.com/sirupsen/logrus"
)

func main() {
	log.SetFormatter(&log.JSONFormatter{})
	log.WithFields(
		log.Fields{
			"foo": "foo",
			"bar": "bar",
		},
	).Info("Something happened")
}
EOF

$ go run logrus-json.go
{"bar":"bar","foo":"foo","level":"info","msg":"Something happened","time":"2021-03-14T22:43:24+08:00"}
```

#### Log levels

Unlike the standard log package, logrus supports log levels.

```text
...
	log.Trace("Something very low level.")
	log.Debug("Useful debugging information.")
	log.Info("Something noteworthy happened!")
	log.Warn("You should probably take a look at this.")
	log.Error("Something failed but I'm not quitting.")
	// Calls os.Exit(1) after logging
	log.Fatal("Bye.")
	// Calls panic() after logging
	log.Panic("I'm bailing.")
}
```

We will get:

```text
{"level":"info","msg":"Something noteworthy happened!","time":"2021-03-14T22:49:08+08:00"}
{"level":"warning","msg":"You should probably take a look at this.","time":"2021-03-14T22:49:08+08:00"}
{"level":"error","msg":"Something failed but I'm not quitting.","time":"2021-03-14T22:49:08+08:00"}
{"level":"fatal","msg":"Bye.","time":"2021-03-14T22:49:08+08:00"}
exit status 1
```

Notice that the Debug level message was not printed. To include it in the logs, set log.Level to equal log.DebugLevel:

```text
log.SetLevel(log.DebugLevel)
```

#### Logging Method Name

```text
log.SetReportCaller(true)
```

#### Using Fields

```text
log.WithFields(log.Fields{
  "event": event,
  "topic": topic,
  "key": key,
}).Fatal("Failed to send event")
```

#### Default Fields

Often it's helpful to have fields always attached to log statements in an application or parts of one. For example, you may want to always log the `request_id` and `user_ip` in the context of a request. Instead of writing `log.WithFields(log.Fields{"request_id": request_id, "user_ip": user_ip})` on every line, you can create a logrus.Entry to pass around instead:

```text
requestLogger := log.WithFields(log.Fields{
    "user_id": "u1234",
    "tx_id":   "t0001",
})
requestLogger.Info("user activity")

log.Warn("You should probably take a look at this.")
log.Error("Something failed but I'm not quitting.")
```

We got this -- note that only `user activity` logs logged with extra fields:

```text
{"level":"info","msg":"user activity","time":"2021-03-15T12:48:09+08:00","tx_id":"t0001","user_id":"u1234"}
{"level":"warning","msg":"You should probably take a look at this.","time":"2021-03-15T12:48:09+08:00"}
{"level":"error","msg":"Something failed but I'm not quitting.","time":"2021-03-15T12:48:09+08:00"}
```





## References

{% embed url="https://www.honeybadger.io/blog/golang-logging/" %}



