package com.xf.test.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class MyProxy implements InvocationHandler {
    Object obj;
    public MyProxy(Object o) {
        obj = o;
    }
    // in this case obj is the class ruby says 
    @Override  
    public Object invoke(Object proxy, Method m, Object[] args) throws Throwable {
        
        Object result = null;
       
        try {
            System.out.println("Proxy Class is called before method invocation");
            System.out.println("args value"+args);
            result = m.invoke(obj, args);
        }
        catch (InvocationTargetException e) {
            System.out.println("Invocation Target Exception: " + e);
        }
        catch (Exception e) {
            System.out.println("Invocation Target Exception: " + e);
        }
        finally {
            System.out.println("Proxy Class is called after method invocation");
        }
        return result;
        
    }
    

}

package com.xf.test.proxy;

class MyInterfaceImpl implements MyInterface {
    @Override
    public void method(String name) {
        System.out.println("Plain method is invoked" + name);
    }
}



package com.xf.test.proxy;

import java.lang.reflect.Proxy;

public class CreateProxyObject {

    public static void main(String[] args) {

        MyInterface myintf = (MyInterface) Proxy.newProxyInstance(
                MyInterface.class.getClassLoader(),
                new Class[] { MyInterface.class }, new MyProxy(
                        new MyInterfaceImpl()));

        myintf.method( new String("ded"));

    }

}