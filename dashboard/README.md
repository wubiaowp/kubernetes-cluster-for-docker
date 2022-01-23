**一、下载Dashboard**

    1、下载dashboard
    wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.4.0/aio/deploy/recommended.yaml

    2、修改dashboard配置
    ![img.png](img.png)

    3、执行创建服务
    kubectl apply -f recommended.yaml

    4、查看执行结果
    ![img_2.png](img_2.png)
    ![img_1.png](img_1.png)

    5、访问Dashboard[Chrome安全限制可以换成Firefox]
    https://masterIP:30001

    6、获取Token方式登陆
    kubectl create serviceaccount dashboard-admin -n kube-system
    kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
    kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')
    ![img_3.png](img_3.png)