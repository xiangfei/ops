premain方式
在Java SE5时代，Instrument只提供了premain一种方式，即在真正的应用程序（包含main方法的程序）main方法启动前启动一个代理程序。例如使用如下命令：

java -javaagent:agent_jar_path[=options] java_app_name
构造agent类
premain方式的agent类必须提供premain方法，代码如下：

package test;

import java.lang.instrument.Instrumentation;

public class Agent {

    public static void premain(String args, Instrumentation inst){
        System.out.println("Hi, I'm agent!");
        inst.addTransformer(new TestTransformer());
    }
}
premain有两个参数，args为自定义传入的代理类参数，inst为VM自动传入的Instrumentation实例。 premain方法的内容很简单，除了标准输出外，只有

inst.addTransformer(new TestTransformer());
这行代码的意思是向inst中添加一个类的转换器。用于转换类的行为。

构造Transformer
下面来实现上述过程中的TestTransformer来完成打印调用方法的类定义转换。

package test;

import java.lang.instrument.ClassFileTransformer;
import java.lang.instrument.IllegalClassFormatException;
import java.security.ProtectionDomain;

import org.objectweb.asm.ClassReader;
import org.objectweb.asm.ClassWriter;
import org.objectweb.asm.Opcodes;
import org.objectweb.asm.tree.ClassNode;
import org.objectweb.asm.tree.FieldInsnNode;
import org.objectweb.asm.tree.InsnList;
import org.objectweb.asm.tree.LdcInsnNode;
import org.objectweb.asm.tree.MethodInsnNode;
import org.objectweb.asm.tree.MethodNode;

public class TestTransformer implements ClassFileTransformer {

    @Override
    public byte[] transform(ClassLoader arg0, String arg1, Class<?> arg2,
            ProtectionDomain arg3, byte[] arg4)
            throws IllegalClassFormatException {
        ClassReader cr = new ClassReader(arg4);
        ClassNode cn = new ClassNode();
        cr.accept(cn, 0);
        for (Object obj : cn.methods) {
            MethodNode md = (MethodNode) obj;
            if ("<init>".endsWith(md.name) || "<clinit>".equals(md.name)) {
                continue;
            }
            InsnList insns = md.instructions;
            InsnList il = new InsnList();
            il.add(new FieldInsnNode(Opcodes.GETSTATIC, "java/lang/System",
                    "out", "Ljava/io/PrintStream;"));
            il.add(new LdcInsnNode("Enter method-> " + cn.name+"."+md.name));
            il.add(new MethodInsnNode(Opcodes.INVOKEVIRTUAL,
                    "java/io/PrintStream", "println", "(Ljava/lang/String;)V"));
            insns.insert(il);
            md.maxStack += 3;

        }
        ClassWriter cw = new ClassWriter(0);
        cn.accept(cw);
        return cw.toByteArray();
    }

}
TestTransformer实现了ClassFileTransformer接口，该接口只有一个transform方法，参数传入包括该类的类加载器，类名，原字节码字节流等，返回被转换后的字节码字节流。 TestTransformer主要使用ASM实现在所有的类定义的方法中，在方法开始出添加了一段打印该类名和方法名的字节码。在转换完成后返回新的字节码字节流。详细的ASM使用请参考ASM手册。

设置MANIFEST.MF
设置MANIFEST.MF文件中的属性，文件内容如下：

Manifest-Version: 1.0
Premain-Class: test.Agent


agentmain方式
premain时Java SE5开始就提供的代理方式，给了开发者诸多惊喜，不过也有些须不变，由于其必须在命令行指定代理jar，并且代理类必须在main方法前启动。因此，要求开发者在应用前就必须确认代理的处理逻辑和参数内容等等，在有些场合下，这是比较苦难的。比如正常的生产环境下，一般不会开启代理功能，但是在发生问题时，我们不希望停止应用就能够动态的去修改一些类的行为，以帮助排查问题，这在应用启动前是无法确定的。 为解决运行时启动代理类的问题，Java SE6开始，提供了在应用程序的VM启动后在动态添加代理的方式，即agentmain方式。 与Permain类似，agent方式同样需要提供一个agent jar，并且这个jar需要满足：

在manifest中指定Agent-Class属性，值为代理类全路径
代理类需要提供public static void agentmain(String args, Instrumentation inst)或public static void agentmain(String args)方法。并且再二者同时存在时以前者优先。args和inst和premain中的一致。
不过如此设计的再运行时进行代理有个问题——如何在应用程序启动之后再开启代理程序呢？ JDK6中提供了Java Tools API，其中Attach API可以满足这个需求。

Attach API中的VirtualMachine代表一个运行中的VM。其提供了loadAgent()方法，可以在运行时动态加载一个代理jar。具体需要参考《Attach API》

agentmain实例-打印当前已加载的类
构造agent类
agentmain方式的代理类必须提供agentmain方法：

package loaded;

import java.lang.instrument.Instrumentation;

public class LoadedAgent {
    @SuppressWarnings("rawtypes")
    public static void agentmain(String args, Instrumentation inst){
        Class[] classes = inst.getAllLoadedClasses();
        for(Class cls :classes){
            System.out.println(cls.getName());
        }
    }
}
agentmain方法通过传入的Instrumentation实例获取当前系统中已加载的类。

设置MANNIFEST.MF
设置MANIFEST.MF文件，指定Agent-Class:

Manifest-Version: 1.0
Agent-Class: loaded.LoadedAgent
Created-By: 1.6.0_29
绑定到目标VM
将agent类和MANIFEST.MF文件编译打成loadagent.jar后，由于agent main方式无法向pre main方式那样在命令行指定代理jar，因此需要借助Attach Tools API。

package attach;

import java.io.IOException;

import com.sun.tools.attach.AgentInitializationException;
import com.sun.tools.attach.AgentLoadException;
import com.sun.tools.attach.AttachNotSupportedException;
import com.sun.tools.attach.VirtualMachine;

public class Test {
    public static void main(String[] args) throws AttachNotSupportedException,
            IOException, AgentLoadException, AgentInitializationException {
        VirtualMachine vm = VirtualMachine.attach(args[0]);
        vm.loadAgent("/Users/jiangbo/Workspace/code/java/javaagent/loadagent.jar");

    }

}
该程序接受一个参数为目标应用程序的进程id，通过Attach Tools API的VirtualMachine.attach方法绑定到目标VM，并向其中加载代理jar。