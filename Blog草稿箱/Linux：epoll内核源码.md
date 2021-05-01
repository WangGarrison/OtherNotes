![image-20210226152603460](img/Linux%EF%BC%9Aepoll%E5%86%85%E6%A0%B8%E6%BA%90%E7%A0%81.img/image-20210226152603460.png)

因为et模式只提醒一次，所以要在提醒的这一次就把数据处理完，就要循环读取，所以et模式要使用非阻塞

sys_epoll_create创建了一个内核事件表，内核事件表底层数据结构是一颗红黑树

![image-20210226153627341](img/Linux%EF%BC%9Aepoll%E5%86%85%E6%A0%B8%E6%BA%90%E7%A0%81.img/image-20210226153627341.png)

 <img src="img/Linux%EF%BC%9Aepoll%E5%86%85%E6%A0%B8%E6%BA%90%E7%A0%81.img/image-20210226153808457.png" alt="image-20210226153808457" style="zoom:50%;" />

![image-20210226201348479](img/Linux%EF%BC%9Aepoll%E5%86%85%E6%A0%B8%E6%BA%90%E7%A0%81.img/image-20210226201348479.png)