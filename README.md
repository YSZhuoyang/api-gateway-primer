# Service Mesh Primer

Bootstrapping a tiny service mesh with istio which supports:

- Transcoding HTTP+JSON into gRPC+Protobuf
- Server push, streaming data to clients
- Handling Http/1.1, Http/2 and gRPC
- Secure services with Istio sidecar [mTLS](https://istio.io/latest/docs/concepts/security/) (similar to [Azure Container Group TLS](https://docs.microsoft.com/en-us/azure/container-instances/container-instances-container-group-ssl))

                   gateway             sidecar     service1
        http json   |--|                 |--|  grpc  |--|
       ------------>|  |---------------->|  |------->|  |
                    |--|        |        |--|        |--|
                                |
                                |      sidecar     service2
                                |        |--|  grpc  |--|
                                -------->|  |------->|  |
                                         |--|        |--|

## Build

1. Pull googleapis submodules for gRPC route annotation:

       git submodule update --init --recursive --remote --merge

2. Build source code and docker images

       make all

## Deploy locally on Kubernetes with [Istio](https://istio.io/)

1. Install Istio (with homebrew):

       brew install istioctl

2. Generate contract descriptor mounted to istio envoy sidecars (for gRPC transcoding):

       cd deploy/contracts && kubectl create configmap proto-descriptor --from-file=desc.pb

3. Launch istio & services:

       cd deploy
       istioctl install --set values.meshConfig.accessLogFile="/dev/stdout" --set values.grafana.enabled=true
       kubectl label namespace default istio-injection=enabled
       kubectl apply -f ./kube

## Test

- Test Http/1.1 & Http/2 with Curl

      chmod u+x ./test.sh && ./test.sh

- Test server streaming with Web Client

  - Cd to `web-client/public` dir, open `index.html` and click `Get Live Data`.
