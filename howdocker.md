how docker works?

docker run --name nginx -p 80:80 nginx:latest -> docker cli -> containerd -> runc -> nginx container 
every thing is backed by docker daemon

wait containerd is separate program can be accessed by:

api -> containerd  (api calls)
ctr -> containerd  (debugging)
nerdctl -> contained (general purpose)

containerd follows oci standards now can be used directly in kubernetes

cri----containerd

and it can be accessed by
crictl -> cri-----------containerd (debugging)



