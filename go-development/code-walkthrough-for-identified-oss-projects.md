# Code Walkthrough for Identified OSS Projects

## graceful

Git Repo: [https://github.com/kuangchanglang/graceful](https://github.com/kuangchanglang/graceful)

Description: Inspired by [overseer](https://github.com/jpillora/overseer) and [endless](https://github.com/fvbock/endless), with minimum codes and handy api to make http server graceful.

### options or optional configs

```go
var (
	defaultWatchInterval = time.Second
	defaultStopTimeout   = 20 * time.Second
	defaultReloadSignals = []syscall.Signal{syscall.SIGHUP, syscall.SIGUSR1}
	defaultStopSignals   = []syscall.Signal{syscall.SIGKILL, syscall.SIGTERM, syscall.SIGINT}

	StartedAt time.Time
)

type option struct {
	reloadSignals []syscall.Signal
	stopSignals   []syscall.Signal
	watchInterval time.Duration
	stopTimeout   time.Duration
}

type Option func(o *option)

// WithReloadSignals set reload signals, otherwise, default ones are used
func WithReloadSignals(sigs []syscall.Signal) Option {
	return func(o *option) {
		o.reloadSignals = sigs
	}
}

// WithStopSignals set stop signals, otherwise, default ones are used
func WithStopSignals(sigs []syscall.Signal) Option {
	return func(o *option) {
		o.stopSignals = sigs
	}
}

// WithStopTimeout set stop timeout for graceful shutdown
//  if timeout occurs, running connections will be discard violently.
func WithStopTimeout(timeout time.Duration) Option {
	return func(o *option) {
		o.stopTimeout = timeout
	}
}

// WithWatchInterval set watch interval for worker checking master process state
func WithWatchInterval(timeout time.Duration) Option {
	return func(o *option) {
		o.watchInterval = timeout
	}
}

...

func NewServer(opts ...Option) *Server {
	option := &option{
		reloadSignals: defaultReloadSignals,
		stopSignals:   defaultStopSignals,
		watchInterval: defaultWatchInterval,
		stopTimeout:   defaultStopTimeout,
	}
	for _, opt := range opts {
		opt(option)
	}
	return &Server{
		addrs:    make([]address, 0),
		handlers: make([]http.Handler, 0),
		opt:      option,
	}
}
```

### Fork

```go
func (m *master) forkWorker() (int, error) {
	path := os.Args[0]
	var args []string
	if len(os.Args) > 1 {
		args = os.Args[1:]
	}

	env := append(os.Environ(), fmt.Sprintf("%s=%s", EnvWorker, ValWorker), fmt.Sprintf("%s=%d", EnvNumFD, len(m.extraFiles)), fmt.Sprintf("%s=%d", EnvParentPid, os.Getpid()), fmt.Sprintf("%s=%d", EnvOldWorkerPid, m.workerPid))

	cmd := exec.Command(path, args...)
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr
	cmd.ExtraFiles = m.extraFiles
	cmd.Env = env
	err := cmd.Start()
	if err != nil {
		return 0, err
	}
	atomic.AddInt32(&m.livingWorkerNum, 1)
	go func() {
		m.workerExit <- cmd.Wait()
	}()
	return cmd.Process.Pid, nil
}
```

## endless

Git Repo: [https://github.com/fvbock/endless](https://github.com/fvbock/endless)

Description: Zero downtime restarts for golang HTTP and HTTPS servers. \(for golang 1.3+\)

Similar to graceful but have a **fetcher** to fetch the updated binary to "upgrade" itself.

## Golang-LRU

Git Repo: [https://github.com/hashicorp/golang-lru.git](https://github.com/hashicorp/golang-lru.git)

Description: This provides the lru package which implements a fixed-size thread safe LRU cache. It is based on the cache in Groupcache.

A map + a doubly LinkedList is the best way to implement LRU cache.

In Golang, it's `map[interface{}]*list.Element` + `container/list.List`:

```go
// LRU implements a non-thread safe fixed size LRU cache
type LRU struct {
	size      int
	evictList *list.List
	items     map[interface{}]*list.Element
	onEvict   EvictCallback
}

// entry is used to hold a value in the evictList
type entry struct {
	key   interface{}
	value interface{}
}
```

