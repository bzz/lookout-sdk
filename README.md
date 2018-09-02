lookout SDK [![GoDoc](https://godoc.org/gopkg.in/src-d/lookout-sdk.v0?status.svg)](https://godoc.org/github.com/src-d/lookout-sdk) [![PyPI version](https://badge.fury.io/py/lookout-sdk.svg)](https://pypi.org/project/lookout-sdk/) [![Build Status](https://travis-ci.org/src-d/src-d/lookout-sdk.svg)](https://travis-ci.org/src-d/src-d/lookout-sdk)
-----------

What is lookout SDK?
===================
Lookout SDK is toolkit for writing new analyzers for the [lookout](https://github.com/src-d/lookout/).


What does it provide
====================
It provides to analyzer an easy access to the DataService API though gRPC service.

DataService abstracts all data access and details of dealing witn actual Git repositories, UAST extraction, programming language detection, etc. This way analyzer can only focus on source code analysis logic.

Architecture of lookout and it's componetes are described in [src-d/lookout/docs](https://github.com/src-d/lookout/tree/master/docs#lookout)

SDK includes:
 - proto definitinos
 - pre-generate libraries code for Golang and Python
 - quickstart documentation on how to write an anlyzer


How to create a new Analyzer
============================

Essentially, every analyzer is a [gRCP server](https://grpc.io/docs/guides/#overview) that implements [Analyzer service](./proto/service_analyzer.proto#L30). Lookout itself act as a gRCP client and will push your analyzer whenever a new PR is ready for anlysis.

### Golang
 - using pre-generated code `gopkg.in/src-d/lookout-sdk.v0/pb`
 - implement Analyzer service interface. Example:
   ```go
   NotifyReviewEvent(ctx context.Context, review *pb.ReviewEvent) (*pb.EventResponse, error)
   NotifyPushEvent(context.Context, *pb.PushEvent) (*pb.EventResponse, error)
   ```
   - analyzer should request [a stream of](https://grpc.io/docs/tutorials/basic/go.html#server-side-streaming-rpc-1) files and UASTs from [DataService](./proto/service_data.proto#L27) that lookout exposed, by default, on `localhost:10301`
   - analyzer have [options](./proto/service_data.proto#L61) to ask either for all files, or just a changed ones, as well as UASTs, language, full file content and/or exclude some paths: by regexp, or just all [vendored paths](https://github.com/github/linguist/blob/master/lib/linguist/vendor.yml)
   - analyzer have to return a list of [Comment](./proto/service_analyzer.proto#L42) messages
 - run gRPC server to listen for requests from the lookout

 SDK contains a quickstart exmaple of the Analyzer that detects language and number of functions for every file [language-analyzer.go](./language-analyzer.go):
  - `go get -u .`
  - `go run language-analyzer.go`


### Python

 - `pip install lookout-sdk`
 - using pre-generated code from (TK python import)
 - implement Analyzer class that extends [AnalyzerServicer](./python/service_analyzer_pb2_grpc.py#34). Example:
   ```python
   def NotifyReviewEvent(self, request, context):
   def NotifyPushEvent(self, request, context):
   ```
 - start [grpc server](https://grpc.io/docs/tutorials/basic/python.html#starting-the-server) and add Analyzer instance to it

SDK conatains a quickstart example of the Analyzer that detects language and number of functions for every file [language-analyzer.py](./language-analyzer.py):
 - `pip3 install -r analyzer-requirements.txt`
 - `python3 language-analyzer.py`


How to test analyzer
====================
 - get `lookout-sdk` binary from [src-d/lookout releases](https://github.com/src-d/lookout/releases)
 - run [`bblfshd`](https://doc.bblf.sh/using-babelfish/getting-started.html)
 - build and start analyzer e.g. Golang
   - `go get -u .`
   - `go run language-analyzer.go` ,
   or Python
   - `pip3 install -r analyzer-requirements.txt`
   - `python3 language-analyzer.py`
 - test **without** Github access, on the latest commit in some Git repository in local FS
```
/lookout review \
  --log-level=debug \
  --git-dir="$GOPATH/src/gopkg.in/src-d/lookout-sdk.v0" \
  "ipv4://localhost:2020"
```

 this will create a "fake" Review event and notify the analyzer, as if you were creating a PR
 from `HEAD~1`.

(TK link to `lookout-sdk` binary desctiption + it's options)


Cavets
======
 - client: disable secure connection on dialing with `grpc.WithInsecure()`
 - client: turn off gRCP [fail-fast](TK link)
 - client/server: set max gRCP message size


How to update SDK
=================
 - re-generate all the code using (TK link to .sh), commit
 - tag a realease


 # License
[Apache License v2.0](./LICENSE)
