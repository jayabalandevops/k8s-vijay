# Core Concepts (13%)

## Creating a Pod and Inspecting it

1. Create the namespace `cka-facebook`.
2. In the namespace `cka-facebook` create a new Pod named `mypod` with the image `nginx:2.3.5`. Expose the port 80.
3. Identify the issue with creating the container. Write down the root cause of issue in a file named `pod-error.txt`.
4. Change the image of the Pod to `nginx:1.15.12`.
5. List the Pod and ensure that the container is running.
6. Log into the container and run the `ls` command. Write down the output. Log out of the container.
7. Retrieve the IP address of the Pod `mypod`.
8. Run a temporary Pod using the image `busybox`, shell into it and run a `wget` command against the `nginx` Pod using port 80.
9. Render the logs of Pod `mypod`.
10. Delete the Pod and the namespace.

<details><summary>Output</summary>
<p>

First, create the namespace.

```bash
$ kubectl create namespace cka-facebook
```

Next, create the Pod in the new namespace.

```bash
$ kubectl run mypod --image=nginx:2.3.5 --restart=Never --port=80 --namespace=cka-facebook
pod/mypod created
```

You will see that the image cannot be pulled as it doesn't exist with this tag.

```bash
$ kubectl get pod -n cka-facebook
NAME    READY   STATUS             RESTARTS   AGE
mypod   0/1     ImagePullBackOff   0          1m
```

The list of events can give you a deeper insight.

```bash
$ kubectl describe pod -n cka-facebook
...
Events:
  Type     Reason                 Age                 From                         Message
  ----     ------                 ----                ----                         -------
  Normal   Scheduled              3m3s                default-scheduler            Successfully assigned mypod to docker-for-desktop
  Normal   SuccessfulMountVolume  3m2s                kubelet, docker-for-desktop  MountVolume.SetUp succeeded for volume "default-token-jbcl6"
  Normal   Pulling                84s (x4 over 3m2s)  kubelet, docker-for-desktop  pulling image "nginx:2.3.5"
  Warning  Failed                 83s (x4 over 3m1s)  kubelet, docker-for-desktop  Failed to pull image "nginx:2.3.5": rpc error: code = Unknown desc = Error response from daemon: manifest for nginx:2.3.5 not found
  Warning  Failed                 83s (x4 over 3m1s)  kubelet, docker-for-desktop  Error: ErrImagePull
  Normal   BackOff                69s (x6 over 3m)    kubelet, docker-for-desktop  Back-off pulling image "nginx:2.3.5"
  Warning  Failed                 69s (x6 over 3m)    kubelet, docker-for-desktop  Error: ImagePullBackOff
```

Go ahead and edit the existing Pod. Alternatively, you could also just use the `kubectl set image pod mypod mypod=nginx --namespace=cka-facebook` command.

```bash
$ kubectl edit pod mypod --namespace=cka-facebook
```

After setting an image that does exist, the Pod should render the status `Running`.

```bash
$ kubectl get pod -n cka-facebook
NAME    READY   STATUS    RESTARTS   AGE
mypod   1/1     Running   0          14m
```

You can shell into the container and run the `ls` command.

```bash
$ kubectl exec mypod -it --namespace=cka-facebook  -- /bin/sh
/ # ls
bin  boot  dev	etc  home  lib	lib64  media  mnt  opt	proc  root  run  sbin  srv  sys  tmp  usr  var
/ # exit
```

Retrieve the IP address of the Pod with the `-o wide` command line option.

```bash
$ kubectl get pods -o wide -n cka-facebook
NAME    READY   STATUS    RESTARTS   AGE   IP               NODE
mypod   1/1     Running   0          12m   192.168.60.149   docker-for-desktop
```

Remember to use the `--rm` to create a temporary Pod.

```bash
$ kubectl run busybox --image=busybox --rm -it --restart=Never -n cka-facebook -- /bin/sh
If you don't see a command prompt, try pressing enter.
/ # wget -O- 192.168.60.149:80
Connecting to 192.168.60.149:80 (192.168.60.149:80)
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
-                    100% |**********************************************************************|   612  0:00:00 ETA
/ # exit
```

The logs of the Pod should show a single line indicating our request.

```bash
$ kubectl logs mypod -n cka-facebook
192.168.60.162 - - [17/May/2019:13:35:59 +0000] "GET / HTTP/1.1" 200 612 "-" "Wget" "-"
```

Delete the Pod and namespace after you are done.

```bash
$ kubectl delete pod mypod --namespace=cka-facebook
pod "mypod" deleted
$ kubectl delete namespace cka-facebook
namespace "cka-facebook" deleted
```

</p>
</details>


<details><summary>History Output</summary>
<p>

   1  create namespace cka-facebook
   2  kubectl create namespace cka-facebook
   3  kubectl get ns
   4  kubectl run mypod --image=nginx:2.3.4 --port=80 --restart=Never --namespace=cka-facebook
   5  kubectl get pods -n cka-facebook
   6  kubectl describe pod -n cka-facebook
   7  kubectl edit pod mypod --namespace=cka-facebook
   8  kubectl get pods -n cka-facebook
   9  kubectl exec mypod -it -n cka-facebook -- /bin/sh
   10  kubectl get pods -o wide
   11  kubectl get pods -o wide -n cka-facebook
   12  kubectl run busybox --image=busybox --rm -it --restart=Never -n cka-facebook -- /bin/sh
   13  kubectl logs mypod -n cka-facebook
   14  kubectl delete pod mypod --namespace=cka-facebook
   15  kubectl delete namespace cka-facebook
   16  history
</p>
</details>