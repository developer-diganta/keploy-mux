# Contact Keeper

A sample contact keeper app to test Keploy integration capabilities using [Gorilla Mux](https://pkg.go.dev/github.com/gorilla/mux) and [Redis](https://redis.io/). 

## Installation Setup

> Note that Testcases are exported as files in the local repository by default

### MacOS 

```shell
curl --silent --location "https://github.com/keploy/keploy/releases/latest/download/keploy_darwin_all.tar.gz" | tar xz -C /tmp

sudo mv /tmp/keploy /usr/local/bin && keploy
```

### Linux

<details>
<summary>Linux</summary>

```shell
curl --silent --location "https://github.com/keploy/keploy/releases/latest/download/keploy_linux_amd64.tar.gz" | tar xz -C /tmp

sudo mv /tmp/keploy /usr/local/bin && keploy
```
</details>

<details>
<summary>Linux ARM</summary>

```shell
curl --silent --location "https://github.com/keploy/keploy/releases/latest/download/keploy_linux_arm64.tar.gz" | tar xz -C /tmp

sudo mv /tmp/keploy /usr/local/bin && keploy
```

The UI can be accessed at http://localhost:6789
</details>

### Windows

<details>
<summary>Windows</summary>


- Download the [Keploy Windows AMD64](https://github.com/keploy/keploy/releases/latest/download/keploy_windows_amd64.tar.gz), and extract the files from the zip folder.

- Run the `keploy.exe` file.

</details>

<details>
<summary>Windows ARM</summary>

- Download the [Keploy Windows ARM64](https://github.com/keploy/keploy/releases/latest/download/keploy_windows_arm64.tar.gz), and extract the files from the zip folder.

- Run the `keploy.exe` file.

</details>

### Prerequisites

1. Redis
2. Go

### Start URL shortener application 

```bash
git clone https://github.com/keploy/samples-go && cd samples-go/contact-keeper

# start Redis
redis-server

# run the sample app
go run main.go

# run the sample app in record mode
export KEPLOY_MODE=record && go run main.go
```

### Skip above steps with Gitpod

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/from-referrer)

## Generate testcases

To generate testcases we just need to make some API calls. You can use [Postman](https://www.postman.com/), [Hoppscotch](https://hoppscotch.io/), or simply `curl`

### Store the contact details

```bash
curl --request POST \
 --url http://localhost:8080/data/1 \
 --header 'content-type: application/json' \
 --data '{
 "name":"John Doe",
 "email":"johndoe@example.com"
}'
```

this will return the data that is inserted.

```
{
    "name":"John Doe",
    "email":"johndoe@example.com"
}
```

Also, you can add any id (a numeric value) to the endpoint to insert the data. Here we have used 1.

### Get back the stored data using the id

1. By using Curl Command

```bash
curl --request GET \
 --url http://localhost:8080/data/1

```


2. Or by querying through the browser `http://localhost:8080/data/1'

Now both these API calls were captured as **editable** testcases and written to `keploy/tests` folder. The keploy directory would also have `mocks` folder that contains all the outputs of redis operations. Here's what the folder structure look like:

```
.
├── README.md
├── main.go
├── go.mod
├── go.sum
├── keploy
│   ├── tests
│       ├── test-1.yaml
│       ├── test-2.yaml
│   └── mocks
│       ├── mock-1.yaml
│       └── mock-2.yaml


```

The test files should look like the sample below and the format is common for both ***http tests*** and ***mocks***. 
```yaml
version: api.keploy.io/v1beta2
kind: Http
name: test-1
spec:
    metadata: {}
    req:
        method: POST
        proto_major: 1
        proto_minor: 1
        url: /data/1
        header:
            Accept: '*/*'
            Content-Length: "54"
            Content-Type: application/json
            User-Agent: curl/7.81.0
        body: |-
            {
             "name":"John Doe",
             "email":"johndoe@example.com"
            }
        body_type: utf-8
    resp:
        status_code: 200
        header:
            Content-Type: application/json
        body: |
            {"name":"John Doe","email":"johndoe@example.com"}
        body_type: utf-8
        status_message: ""
        proto_major: 1
        proto_minor: 1
    objects:
        - type: error
          data: H4sIAAAAAAAA/wEAAP//AAAAAAAAAAA=
    mocks:
        - mock-1-0
    assertions:
        noise: []
    created: 1675929915

```

Now, let's see the magic! ✨💫

## Test mode

Now that we have our testcase captured, run the test file (in the gomux-redis directory, not the Keploy directory).

```shell
 go test -coverpkg=./... -covermode=atomic  ./...
```

output should look like

```shell
ok      sample-app 6.534s  coverage: 55.1% of statements in ./...
```

> **We got 55.1% without writing any e2e testcases or mocks for Redis!**

So no need to setup fake database/apis like Redis or write mocks for them. Keploy automatically mocks them and, **The application thinks it's talking to Redis 😄**

Go to the `Keploy Console` to get deeper insights on what testcases ran, what failed.

<details>
<summary>𝗜𝗻𝘀𝗶𝗴𝗵𝘁𝘀 𝗼𝗻 𝗞𝗲𝗽𝗹𝗼𝘆 𝗖𝗼𝗻𝘀𝗼𝗹𝗲</summary>

```shell
 <=========================================> 
  TESTRUN STARTED with id: "635ffdba-1382-48fd-8c81-8e6eebf95f29"
	For App: "my-app"
	Total tests: 2
 <=========================================> 

Testrun passed for testcase with id: "test-2"

--------------------------------------------------------------------

Testrun passed for testcase with id: "test-1"

--------------------------------------------------------------------


 <=========================================> 
  TESTRUN SUMMARY. For testrun with id: "635ffdba-1382-48fd-8c81-8e6eebf95f29"
	Total tests: 2
	Total test passed: 2
	Total test failed: 0
 <=========================================> 
 

```

</details>

---

### Make a code change

Now try changing something like commenting line numbers 115 and 116 and uncommenting line 119 in [main.go](./main.go) and running ` go test -coverpkg=./... -covermode=atomic ./...` again

```shell
starting test execution {"id": "3b20b4d9-58d4-485e-97a7-70856591bd7a", "total tests": 3}
testing 1 of 3  {"testcase id": "34ba2282-1224-4fde-89de-f337e0a4816c"}
testing 2 of 3  {"testcase id": "db7ca851-550b-4c0b-b3e4-00d5809a2f72"}
testing 3 of 3  {"testcase id": "cb32305f-1bcc-4eab-a6f3-e3e815cb9f48"}
result  {"testcase id": "cb32305f-1bcc-4eab-a6f3-e3e815cb9f48", "passed": false}
result  {"testcase id": "db7ca851-550b-4c0b-b3e4-00d5809a2f72", "passed": true}
result  {"testcase id": "34ba2282-1224-4fde-89de-f337e0a4816c", "passed": true}
test run completed      {"run id": "3b20b4d9-58d4-485e-97a7-70856591bd7a", "passed overall": false}
--- FAIL: TestKeploy (6.03s)
    keploy.go:77: Keploy test suite failed
FAIL
coverage: 51.1% of statements in ./...
FAIL    echo-psql-url-shortener 6.620s
FAIL

```


To deep dive the problem you can look at the keploy logs or goto [the UI](http://localhost:6789/testruns)
<details>
<summary>𝗞𝗲𝗽𝗹𝗼𝘆 𝗟𝗼𝗴𝘀</summary>

```shell
 <=========================================> 
  TESTRUN STARTED with id: "3b20b4d9-58d4-485e-97a7-70856591bd7a"
        For App: "sample-url-shortener"
        Total tests: 3
 <=========================================> 

Testrun failed for testcase with id: "cb32305f-1bcc-4eab-a6f3-e3e815cb9f48"
Test Result:
        Input Http Request: models.HttpReq{
  Method:     "POST",
  ProtoMajor: 1,
  ProtoMinor: 1,
  URL:        "/url",
  URLParams:  map[string]string{},
  Header:     http.Header{
    "Accept": []string{
      "*/*",
    },
    "Content-Length": []string{
      "33",
    },
    "Content-Type": []string{
      "application/json",
    },
    "User-Agent": []string{
      "curl/7.79.1",
    },
  },
  Body: "{\n  \"url\": \"https://github.com\"\n}",
}

        Expected Response: models.HttpResp{
  StatusCode: 200,
  Header:     http.Header{
    "Content-Type": []string{
      "application/json; charset=UTF-8",
    },
  },
  Body:          "{\"ts\":1664887399743163000,\"url\":\"http://localhost:8082/4KepjkTT\"}\n",
  StatusMessage: "",
  ProtoMajor:    0,
  ProtoMinor:    0,
}

        Actual Response: models.HttpResp{
  StatusCode: 200,
  Header:     http.Header{
    "Content-Type": []string{
      "application/json; charset=UTF-8",
    },
  },
  Body:          "{\"ts\":1664891977696880000,\"urls\":\"http://localhost:8082/4KepjkTT\"}\n",
  StatusMessage: "",
  ProtoMajor:    0,
  ProtoMinor:    0,
}

DIFF: 
        Response body: {
                "url": {
                        Expected value: "http://localhost:8082/4KepjkTT"
                        Actual value: nil
                }
                "urls": {
                        Expected value: nil
                        Actual value: "http://localhost:8082/4KepjkTT"
                }
        }
--------------------------------------------------------------------

Testrun passed for testcase with id: "db7ca851-550b-4c0b-b3e4-00d5809a2f72"

--------------------------------------------------------------------

Testrun passed for testcase with id: "34ba2282-1224-4fde-89de-f337e0a4816c"

--------------------------------------------------------------------


 <=========================================> 
  TESTRUN SUMMARY. For testrun with id: "3b20b4d9-58d4-485e-97a7-70856591bd7a"
        Total tests: 3
        Total test passed: 2
        Total test failed: 1
 <=========================================> 

```

![recent test runs](https://user-images.githubusercontent.com/21143531/159178101-403e9fab-f92b-4db3-87d7-1abdef0a7a7d.png)

![expected vs actual response](https://user-images.githubusercontent.com/21143531/159178125-9cffa7b5-509d-40ea-be4f-985b7b85d877.png)

</details>
