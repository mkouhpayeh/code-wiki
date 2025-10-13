# Load Test
1.  Bombardier: is a HTTP(S) benchmarking tool. It is written in Go programming language and uses excellent **FastHTTP** instead of Go's default http library, because of its lightning fast performance.
    ``` shell title="Install"
    go install github.com/codesenberg/bombardier@latest
    ```
    ``` shell title="Usage (500 requests using 50 connections and latency distribution)"
    bombardier [<flags>] <url>
    bombardier -c 50 -n 500 -d 10s -l http://localhost:5005
    ```
2.  
