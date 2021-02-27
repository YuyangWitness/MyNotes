# Kubernetes

## 容器

容器其实是一种沙盒技术。顾名思义，沙盒就是能够像一个集装箱一样，把你的应用“装”起来的技术。这样，应用与应用之间，就因为有了边界而不至于相互干扰；而被装进集装箱的应用，也可以被方便地搬来搬去

### 容器是如何实现隔离的？

在Linux中可以创建一个新的线程，而在新线程中是可以去隔离宿主机的线程，容器就是这个新线程，用户仿佛进入了一个新的主机中，其实不是，这只是Linux Namespace给的一个障眼法，在容器中看到的Monut, Network以及User这些都是通过Namespace来实现的隔离。