# Consul Tutorial

If you are a microservice rookie developer looking for tutorials on how to use consul as a service discovery and a distributed storage, then you have come to right place. I compiled enough information you need to know to get you well started.

## Download

You can download Consul binary executable file directly from the official website and then put it in a PATH directory to make it available.

## Put Consul in PATH

```bash
echo $PATH
```

Output is gonna be something like the following:

```
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/go/bin:/Users/yourUsername/go/bin
```

I put **consul** binary executable file in **GOPATH/bin/**, not a conventional way to do this. And I recommend you put it in directory **/usr/local/bin**.

Once you no longer need consul, just delete **consul** binary from the directory.

## Start Consul

```bash
consul agent -dev -node localmachine -ui
```

"agent" means you want to start a Consul agent, which is the essential component of consul service discovery.

"-dev" means that you want to start the Consul agent in development mode.

"-node localmachine" names the Consul agent. In Macos, this is required because it is likely your hostname contains dash ("-") and Consul has a problem with hostname having dash. To check out whether your hostname contains dash, run the following command:

```bash
hostname
```

"-ui" enables the built-in static web UI server, which you can access at **http://localhost:8500/ui**

## Useful Commands

For quick development, I have tried to save you the trouble to go through consul documentations without purpose by sharing the following commands, which are enough for your microservices development.

### List all the services registered to Consul

```bash
consul catalog services
```

### Put KV in Consul

```bash
consul kv put micro/config/log '{"key":"value", "key_1":"value_1"}'
```

**micro/config/log** represents the location where you want to put these kv pairs.

The following JSON is the data.

If the JSON contains environment variables to be parsed, refer to [this Stack Overflow link](https://stackoverflow.com/questions/56980951/how-to-parse-an-environment-value-in-a-string-in-a-command).

### Get KV in Consul

```bash
consul kv get micro/config/log
```

You can also access KV in the UI.

### Check out Registered Services

```bash
curl http://127.0.0.1:8500/v1/catalog/services
```

**httpie** is a great simple substitute of curl. To get pretty-formatted json, do the following:

```bash
sudo yum install httpie
http :8500/v1/catalog/services
```

The output is something like the following:

```json
HTTP/1.1 200 OK
Content-Encoding: gzip
Content-Length: 612
Content-Type: application/json
Date: Mon, 29 Jul 2019 06:21:56 GMT
Vary: Accept-Encoding
X-Consul-Effective-Consistency: leader
X-Consul-Index: 2589856
X-Consul-Knownleader: true
X-Consul-Lastcontact: 0

{
    "consul": [], 
    ...
}
```

### Check out A Specific Service

```bash
http :8500/v1/catalog/service/go.micro.srv.hello
```

Sample output:

```json
HTTP/1.1 200 OK
Content-Encoding: gzip
Content-Length: 814
Content-Type: application/json
Date: Mon, 29 Jul 2019 06:26:26 GMT
Vary: Accept-Encoding
X-Consul-Effective-Consistency: leader
X-Consul-Index: 2589907
X-Consul-Knownleader: true
X-Consul-Lastcontact: 0

[
    {
        "Address": "8.8.8.8", 
    }
]
```

### Deregister A Service

Occasionaly, you may want to deregister a service manually. For me, I wrote my microservices initially without specifying register **TTL** and **Interval**, thus making microservices still registered to Consul after killing microservices. You should always specify those two properties in a way like the following.

```go
// New Service
service := micro.NewService(
    micro.Registry(microReg),
    micro.RegisterTTL(time.Second*30),
    micro.RegisterInterval(time.Second*10),
    micro.Name("go.micro.srv.hello"),
    micro.Version("latest"),
)
```

At that time, I did not know much about Consul. And its great power with its many commands is daunting for beginners like you and me. I understand. I thought about restarting Consul since it would definitely solve everything easily. But I was on a production environment and thus restarting is not an option. My supervisor would probably have been wondering what went wrong with whatever services there are if I had gone down that path.

Instead, I played around Consul and found that there is deregister command. Notice that you can not just use, let's say, "go.micro.srv.hello" as "-id." You have to use the full name of the registered services.

```bash
consul services deregister -id=go.micro.srv.hello-b0dh2c1b-9330-472e-a085-01a32e755a3f
```

All right, all right, all right. You are now well on your way to conquer microservices from the service discovery part.