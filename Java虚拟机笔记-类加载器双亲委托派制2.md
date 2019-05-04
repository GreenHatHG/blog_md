---
title: Java虚拟机笔记-类加载器双亲委托派制2
date: 2019-05-04 10:33:34
categories: 读书笔记
tags:
- jvm
---

继续学习jvm类加载器双亲委派机制

<!-- more -->

# 样例1

```java
public class Main{
    public static void main(String[] args) throws Exception{
        //获取系统类加载器
        ClassLoader classLoader = ClassLoader.getSystemClassLoader();
        System.out.println(classLoader);
        System.out.println("-------------------");
        
        while(null != classLoader){
            //获取加载器的父亲
            classLoader = classLoader.getParent(); 
            System.out.println(classLoader);
        }
    }
}
/*输出：
sun.misc.Launcher$AppClassLoader@18b4aac2
-------------------
sun.misc.Launcher$ExtClassLoader@74a14482
null
*/
```

## getSystemClassLoader

```java
/**
 * Returns the system class loader for delegation.  This is the default
 * delegation parent for new <tt>ClassLoader</tt> instances, and is
 * typically the class loader used to start the application.
 *
 * <p> This method is first invoked early in the runtime's startup
 * sequence, at which point it creates the system class loader and sets it
 * as the context class loader of the invoking <tt>Thread</tt>.
 *
 * <p> The default system class loader is an implementation-dependent
 * instance of this class.
 *
 * <p> If the system property "<tt>java.system.class.loader</tt>" is defined
 * when this method is first invoked then the value of that property is
 * taken to be the name of a class that will be returned as the system
 * class loader.  The class is loaded using the default system class loader
 * and must define a public constructor that takes a single parameter of
 * type <tt>ClassLoader</tt> which is used as the delegation parent.  An
 * instance is then created using this constructor with the default system
 * class loader as the parameter.  The resulting class loader is defined
 * to be the system class loader.
 *
 * <p> If a security manager is present, and the invoker's class loader is
 * not <tt>null</tt> and the invoker's class loader is not the same as or
 * an ancestor of the system class loader, then this method invokes the
 * security manager's {@link
 * SecurityManager#checkPermission(java.security.Permission)
 * <tt>checkPermission</tt>} method with a {@link
 * RuntimePermission#RuntimePermission(String)
 * <tt>RuntimePermission("getClassLoader")</tt>} permission to verify
 * access to the system class loader.  If not, a
 * <tt>SecurityException</tt> will be thrown.  </p>
 *
 * @return  The system <tt>ClassLoader</tt> for delegation, or
 *          <tt>null</tt> if none
 *
 * @throws  SecurityException
 *          If a security manager exists and its <tt>checkPermission</tt>
 *          method doesn't allow access to the system class loader.
 *
 * @throws  IllegalStateException
 *          If invoked recursively during the construction of the class
 *          loader specified by the "<tt>java.system.class.loader</tt>"
 *          property.
 *
 * @throws  Error
 *          If the system property "<tt>java.system.class.loader</tt>"
 *          is defined but the named class could not be loaded, the
 *          provider class does not define the required constructor, or an
 *          exception is thrown by that constructor when it is invoked. The
 *          underlying cause of the error can be retrieved via the
 *          {@link Throwable#getCause()} method.
 *
 * @revised  1.4
 */
@CallerSensitive
public static ClassLoader getSystemClassLoader() {
    initSystemClassLoader();
    if (scl == null) {
        return null;
    }
    SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        checkClassLoaderPermission(scl, Reflection.getCallerClass());
    }
    return scl;
}
```

## getParent

```java
/**
 * Returns the parent class loader for delegation. Some implementations may
 * use <tt>null</tt> to represent the bootstrap class loader. This method
 * will return <tt>null</tt> in such implementations if this class loader's
 * parent is the bootstrap class loader.
 *
 * <p> If a security manager is present, and the invoker's class loader is
 * not <tt>null</tt> and is not an ancestor of this class loader, then this
 * method invokes the security manager's {@link
 * SecurityManager#checkPermission(java.security.Permission)
 * <tt>checkPermission</tt>} method with a {@link
 * RuntimePermission#RuntimePermission(String)
 * <tt>RuntimePermission("getClassLoader")</tt>} permission to verify
 * access to the parent class loader is permitted.  If not, a
 * <tt>SecurityException</tt> will be thrown.  </p>
 *
 * @return  The parent <tt>ClassLoader</tt>
 *
 * @throws  SecurityException
 *          If a security manager exists and its <tt>checkPermission</tt>
 *          method doesn't allow access to this class loader's parent class
 *          loader.
 *
 * @since  1.2
 */
@CallerSensitive
public final ClassLoader getParent() {
    if (parent == null)
        return null;
    SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        // Check access to the parent class loader
        // If the caller's class loader is same as this class loader,
        // permission check is performed.
        checkClassLoaderPermission(parent, Reflection.getCallerClass());
    }
    return parent;
}
```

# 样例2

```java
import java.net.URL;
import java.util.Enumeration;

public class Main{
    public static void main(String[] args) throws Exception{
        //获取当前线程上下文的类加载器
        ClassLoader classLoader = Thread.currentThread().getContextClassLoader();

        String res = "Main.class";
        //获取给定名称的所有资源
        Enumeration<URL> urls = classLoader.getResources(res);

        //遍历输出所有资源
        while(urls.hasMoreElements()){
            URL url = urls.nextElement();
            System.out.println(url);
        }
    }
}
/*输出：
file:/home/cc/IdeaProjects/test/out/production/test/Main.class
*/
```

## getContextClassLoader()

```java
/**
     * Returns the context ClassLoader for this Thread. The context
     * ClassLoader is provided by the creator of the thread for use
     * by code running in this thread when loading classes and resources.
     * If not {@linkplain #setContextClassLoader set}, the default is the
     * ClassLoader context of the parent Thread. The context ClassLoader of the
     * primordial thread is typically set to the class loader used to load the
     * application.
     *
     * <p>If a security manager is present, and the invoker's class loader is not
     * {@code null} and is not the same as or an ancestor of the context class
     * loader, then this method invokes the security manager's {@link
     * SecurityManager#checkPermission(java.security.Permission) checkPermission}
     * method with a {@link RuntimePermission RuntimePermission}{@code
     * ("getClassLoader")} permission to verify that retrieval of the context
     * class loader is permitted.
     *
     * @return  the context ClassLoader for this Thread, or {@code null}
     *          indicating the system class loader (or, failing that, the
     *          bootstrap class loader)
     *
     * @throws  SecurityException
     *          if the current thread cannot get the context ClassLoader
     *
     * @since 1.2
     */
    @CallerSensitive
    public ClassLoader getContextClassLoader() {
        if (contextClassLoader == null)
            return null;
        SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            ClassLoader.checkClassLoaderPermission(contextClassLoader,
                                                   Reflection.getCallerClass());
        }
        return contextClassLoader;
    }
```

## getResources()

```java
/**
     * Finds all the resources with the given name. A resource is some data
     * (images, audio, text, etc) that can be accessed by class code in a way
     * that is independent of the location of the code.
     *
     * <p>The name of a resource is a <tt>/</tt>-separated path name that
     * identifies the resource.
     *
     * <p> The search order is described in the documentation for {@link
     * #getResource(String)}.  </p>
     *
     * @apiNote When overriding this method it is recommended that an
     * implementation ensures that any delegation is consistent with the {@link
     * #getResource(java.lang.String) getResource(String)} method. This should
     * ensure that the first element returned by the Enumeration's
     * {@code nextElement} method is the same resource that the
     * {@code getResource(String)} method would return.
     *
     * @param  name
     *         The resource name
     *
     * @return  An enumeration of {@link java.net.URL <tt>URL</tt>} objects for
     *          the resource.  If no resources could  be found, the enumeration
     *          will be empty.  Resources that the class loader doesn't have
     *          access to will not be in the enumeration.
     *
     * @throws  IOException
     *          If I/O errors occur
     *
     * @see  #findResources(String)
     *
     * @since  1.2
     */
    public Enumeration<URL> getResources(String name) throws IOException {
        @SuppressWarnings("unchecked")
        Enumeration<URL>[] tmp = (Enumeration<URL>[]) new Enumeration<?>[2];
        if (parent != null) {
            tmp[0] = parent.getResources(name);
        } else {
            tmp[0] = getBootstrapResources(name);
        }
        tmp[1] = findResources(name);

        return new CompoundEnumeration<>(tmp);
    }
```

# 获取ClassLoader的途径

1. 获取当前类的ClassLoader

```java
class<?> clazz
clazz.getClassLoader();
```

2. 获取当前线程上下文的ClassLoader

```java
Thread.currentThread.getContextClassLoader();
```

3. 获得系统的ClassLoader

```java
ClassLoader.getSystemClassLoader();
```

4. 获得调用者的ClassLoader

```java
DriverManager.getCallerClassLoader();
```

