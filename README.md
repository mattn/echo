# Bolt [![GoDoc](http://img.shields.io/badge/go-documentation-blue.svg?style=flat-square)](http://godoc.org/github.com/labstack/bolt) [![Build Status](http://img.shields.io/travis/fatih/structs.svg?style=flat-square)](https://travis-ci.org/labstack/bolt) [![Coverage Status](http://img.shields.io/coveralls/labstack/bolt.svg?style=flat-square)](https://coveralls.io/r/labstack/bolt)
Bolt is a fast HTTP router (zero memory allocation) + micro web framework in Go.

### Features
- Zippy router.
- Extensible middleware / handler, supports:
	- Middleware
		- `func(bolt.HandlerFunc) bolt.HandlerFunc`
		- `http.HandlerFunc`
		- `http.Handler`
		- `func(http.Handler) http.Handler`
	- Handler
		- `func(*bolt.Context)`
		- `http.HandlerFunc`
		- `http.Handler`
- Serve static files, including index.

### Example
https://github.com/labstack/bolt/tree/master/example

```go
package main

import (
	"net/http"

	"github.com/labstack/bolt"
	mw "github.com/labstack/bolt/middleware"
	"github.com/rs/cors"
	"github.com/thoas/stats"
)

type user struct {
	ID   string `json:"id"`
	Name string `json:"name"`
}

var users map[string]user

func init() {
	users = map[string]user{
		"1": user{
			ID:   "1",
			Name: "Wreck-It Ralph",
		},
	}
}

func createUser(c *bolt.Context) {
	u := new(user)
	if c.Bind(u) {
		users[u.ID] = *u
		c.JSON(http.StatusOK, u)
	}
}

func getUsers(c *bolt.Context) {
	c.JSON(http.StatusOK, users)
}

func getUser(c *bolt.Context) {
	c.JSON(http.StatusOK, users[c.P(0)])
}

func main() {
	b := bolt.New()

	//*************************//
	//   Built-in middleware   //
	//*************************//
	b.Use(mw.Logger)

	//****************************//
	//   Third-party middleware   //
	//****************************//
	// https://github.com/rs/cors
	b.Use(cors.Default().Handler)

	// https://github.com/thoas/stats
	s := stats.New()
	b.Use(s.Handler)
	// Route
	b.Get("/stats", func(c *bolt.Context) {
		c.JSON(200, s.Data())
	})

	// Serve index file
	b.Index("public/index.html")

	// Serve static files
	b.Static("/js", "public/js")

	//************//
	//   Routes   //
	//************//
	b.Post("/users", createUser)
	b.Get("/users", getUsers)
	b.Get("/users/:id", getUser)

	// Start server
	b.Run(":8080")
}
```

### Benchmark
Based on [julienschmidt/go-http-routing-benchmark] (https://github.com/vishr/go-http-routing-benchmark)
##### [GitHub API](http://developer.github.com/v3)
```
BenchmarkAce_GithubAll          20000	     70743 ns/op	   13792 B/op	     167 allocs/op
BenchmarkBear_GithubAll         10000	    251638 ns/op	   79952 B/op	     943 allocs/op
BenchmarkBeego_GithubAll         3000	    485840 ns/op	  146272 B/op	    2092 allocs/op
BenchmarkBolt_GithubAll         30000	     49183 ns/op	       0 B/op	       0 allocs/op
BenchmarkBone_GithubAll          1000	   2167949 ns/op	  648016 B/op	    8119 allocs/op
BenchmarkDenco_GithubAll        20000	     82404 ns/op	   20224 B/op	     167 allocs/op
BenchmarkGin_GithubAll          20000	     72831 ns/op	   13792 B/op	     167 allocs/op
BenchmarkGocraftWeb_GithubAll	 5000	    385807 ns/op	  133280 B/op	    1889 allocs/op
BenchmarkGoji_GithubAll          3000	    580354 ns/op	   56113 B/op	     334 allocs/op
BenchmarkGoJsonRest_GithubAll    5000	    468481 ns/op	  135995 B/op	    2940 allocs/op
BenchmarkGoRestful_GithubAll      200	   9555198 ns/op	  649139 B/op	    7355 allocs/op
BenchmarkGorillaMux_GithubAll     200	   6906975 ns/op	  153136 B/op	    1791 allocs/op
BenchmarkHttpRouter_GithubAll	30000	     55239 ns/op	   13792 B/op	     167 allocs/op
BenchmarkHttpTreeMux_GithubAll	10000	    139673 ns/op	   56112 B/op	     334 allocs/op
BenchmarkKocha_GithubAll	    10000	    148809 ns/op	   23304 B/op	     843 allocs/op
BenchmarkMacaron_GithubAll	     2000	    713989 ns/op	  224960 B/op	    2315 allocs/op
BenchmarkMartini_GithubAll	      100	  10716312 ns/op	  237953 B/op	    2686 allocs/op
BenchmarkPat_GithubAll	          300	   4099176 ns/op	 1504101 B/op	   32222 allocs/op
BenchmarkRevel_GithubAll	     2000	   1142947 ns/op	  345553 B/op	    5918 allocs/op
BenchmarkRivet_GithubAll	    10000	    256426 ns/op	   84272 B/op	    1079 allocs/op
BenchmarkTango_GithubAll	      500	   3543693 ns/op	 1338664 B/op	   27736 allocs/op
BenchmarkTigerTonic_GithubAll	 2000	    980339 ns/op	  241088 B/op	    6052 allocs/op
BenchmarkTraffic_GithubAll	      200	   7430271 ns/op	 2664761 B/op	   22390 allocs/op
BenchmarkVulcan_GithubAll	     5000	    294640 ns/op	   19894 B/op	     609 allocs/op
BenchmarkZeus_GithubAll	         2000	    778697 ns/op	  300688 B/op	    2648 allocs/op
```
