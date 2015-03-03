---
title: RMI avec des SocketFactories pour compresser les flux
layout: post
tags : [ java, rmi ]
---
{% include JB/setup %}

Chez un client, je me retrouve face à une application qui fait un usage intensif des échanges de données dans des volumes importants via RMI. Examinant la bande passante consommée, mon habitude des applications web m'amène à me demander s'il n'est pas possible de paramétrer une compression à la volée sur le canal RMI à l'instar de ce qui se trouve usuellement sur le connecteur HTTP d'un conteneur de servlets.

Visiblement, rien de fourni à ce sujet dans le JDK et mes recherches me guident jusqu'à la [documentation d'Oracle](http://docs.oracle.com/javase/8/docs/technotes/guides/rmi/socketfactory/) qui détaille comment utiliser des `SocketFactory` personnalisées. L'exemple pris pour illustrer le concept est un chiffrement/déchiffrement basique des flux à la volée. Et là je me dis bon plan, mon cas est bien plus simple, je décore les `(Input|Output)Stream` dans ma sous classe de `java.net.Socket` avec une des implémentations de `java.util.zip` apportant la gestion des données compressées et le tour est joué. Il ne me restera plus qu'à mesurer les gains obtenus:

```java
public class CompressedSocket extends Socket {

    private InputStream is = null;

    private OutputStream os = null;


    public CompressedSocket(String address, int port) throws IOException {
        super(address, port);
    }

    public CompressedSocket() {
        super();
    }

    @Override
    public synchronized InputStream getInputStream() throws IOException {
        if (is == null) {
            is = new InflaterInputStream(super.getInputStream());
        }
        return is;
    }

    @Override
    public synchronized OutputStream getOutputStream() throws IOException {
        if (os == null) {
            os = new DeflaterOutputStream(super.getOutputStream());
        }
        return os;
    }
}
```

Je code un composant pour tester ... 

```java
public class MyRemoteServiceImpl implements MyRemoteService {
    
    @Override
    public String sayHello() {
        return "bonjour";
    }

}
```

... et monte la publication et la consommation dans un test unitaire (je vous fais grâce de la tuyauterie des `(Client|Server)SocketsFactory` qui ne sont que des dérivations de celles fournies dans la documentation) :

```java
public class MyRemoteServiceImplTest {

    private static final String SERVICE_NAME = "myService";

    @Before
    public void setUp() throws Exception {
        final MyRemoteServiceImpl myRemoteService = new MyRemoteServiceImpl();
        final Remote remote = UnicastRemoteObject.exportObject(myRemoteService, 0, new CompressedClientSocketFactory(),
                new CompressedServerSocketFactory());

        LocateRegistry.createRegistry(2002);
        Registry registry = LocateRegistry.getRegistry(2002);
        registry.rebind(SERVICE_NAME, remote);
    }

    @Test
    public void testRemote() throws Exception {
        Registry registry = LocateRegistry.getRegistry(2002);
        final MyRemoteService lookup = (MyRemoteService) registry.lookup(SERVICE_NAME);
        System.out.println("lookup = " + lookup.sayHello());

    }
}
```

Je m'apprêtais déjà à aller me faire couler un café en me racontant que j'étais vraiment trop fort, mais je fus stoppé net dans mon élan dès la première exécution:

```
java.rmi.ConnectIOException: error during JRMP connection establishment; nested exception is: 
	java.net.SocketTimeoutException: Read timed out
	at sun.rmi.transport.tcp.TCPChannel.createConnection(TCPChannel.java:304)
	at sun.rmi.transport.tcp.TCPChannel.newConnection(TCPChannel.java:202)
	at sun.rmi.server.UnicastRef.invoke(UnicastRef.java:130)
	at java.rmi.server.RemoteObjectInvocationHandler.invokeRemoteMethod(RemoteObjectInvocationHandler.java:194)
	at java.rmi.server.RemoteObjectInvocationHandler.invoke(RemoteObjectInvocationHandler.java:148)
	at com.sun.proxy.$Proxy3.sayHello(Unknown Source)
	at io.github.tbt.rmi.zip.MyRemoteServiceImplTest.testRemote(MyRemoteServiceImplTest.java:33)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	
[... blablabla plein d'étages dans la pile d'appel blablabla ...]

Caused by: java.net.SocketTimeoutException: Read timed out
	at java.net.SocketInputStream.socketRead0(Native Method)
	at java.net.SocketInputStream.read(SocketInputStream.java:150)
```

Par dichotomie, j'arrive à déduire que la cause de mon problème est la compression... Et comme tout bon développeur fainéant qui se respecte, j'interroge Google dans l'espoir de trouver une réponse ficelée à mon problème. Je butine sur une collection d'articles au mieux extrêmement complexes, mais la plupart du temps complètement buggés (à se demander si les auteurs ont testé avant de les publier), ou encore avec des questions posées toujours non résolues. Evidemment tout ce contenu affiche un âge moyen assez élevé, il ne traite pas d'AngularJS...

N'étant pas plus avancé par la communauté, je me vois contraint de réfléchir... Un timeout à cause de la compression? Je commence à imaginer que le `DeflaterOutputStream` doit bufferiser des octets avant de procéder à la compression, ce qui pourrait poser problème dans une communication distante, mais difficile de m'appuyer sur cette supputation par manque de faits... En revanche, elle me permet d'orienter mes recherches et je tombe sur ce [billet sur le blog d'Oracle](https://blogs.oracle.com/xuemingshen/entry/the_flushable_deflateroutputstream) qui explique qu'un [bug préhistorique](http://bugs.java.com/view_bug.do?bug_id=4206909) (répertorié en janvier 1999!!) du JDK vient enfin d'être résolu par le JDK 7, au passage mes lectures se rapprochent du présent, ce qui est rassurant. 

Pour résumer, lors du `flush()`, le `DeflaterOutputStream` déclenche bien le `flush()` du flux délégué mais sans consommer le buffer du compresseur au préalable et il ne le fait qu'en fin d'opération. Pour zipper un fichier, ce n'est évidemment pas un problème, mais pour une communication distante les choses se gâtent et le client reste en attente de données coincées dans le compresseur...  
A partir de la version 7, il est possible de spécifier la méthode de synchronisation du buffer grâce à un [nouveau constructeur] (http://docs.oracle.com/javase/7/docs/api/java/util/zip/DeflaterOutputStream.html#DeflaterOutputStream(java.io.OutputStream,%20boolean)). La création de mon flux sortant devient donc:

```java
    @Override
    public synchronized OutputStream getOutputStream() throws IOException {
        if (os == null) {
            os = new DeflaterOutputStream(super.getOutputStream(),true);
        }
        return os;
    }
```

La magie opère et mon client arrive enfin à échanger avec le serveur.
      
Comme Internet est parsemé d'exemples inopérants et de contournements hasardeux dus à un problème historique, je vous livre le mien: il est simple et fonctionnel, vous le trouverez sur [Github](https://github.com/bleporini/rmi-zip).
    
Pour conclure, les efforts fournis furent payants puisque l'application en question a bénéficié d'une couche de transport qui est devenue jusqu'à 12 fois plus rapide dans les cas les plus flagrants.


