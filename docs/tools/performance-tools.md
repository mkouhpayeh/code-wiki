# 📚 Performance Tools

## Load Test - Bombardier
Bombardier is a HTTP(S) benchmarking tool. It is written in Go programming language and uses excellent **FastHTTP** instead of Go's default http library, because of its lightning fast performance.

``` bash title="Install"
go install github.com/codesenberg/bombardier@latest
```

``` bash  title="Usage (500 requests using 50 connections and latency distribution)"
bombardier [<flags>] <url>
bombardier -c 50 -n 500 -d 10s -l http://localhost:5005
```

---

## Get Process ID

``` bash 
tasklist /fi "IMAGENAME eq AsyncTest.exe"
```

---

## dotnet-counters

``` bash 
dotnet-counters monitor --process-id [36541] --counters System.Runtime[threadpool-thhread-count, threadpool-queue-length, monitor-lock-connection-count] --output counters.txt
```

---

## SQL Server Connections

``` sql
SELECT  dbid, DB_NAME(dbid) AS DatabaseName, COUNT(dbid) AS NumberOfConnections
        FROM sys.sysprocess
        WHERE DB_NAME(dbid) = 'TestDB'
        GROUP BY dbid     
```
    
--- 
