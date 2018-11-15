public class InitTestBean {
    private String prop1;
    private String prop2;
    private int i ;
    public InitTestBean(String prop1, String prop2) {
        System.out.println("Instantiating InitTestBean");
        this.prop1 = prop1;
        this.prop2 = prop2;
         i=0;
    }
/*
    public void setProp1(String prop1) {
        System.out.println("In setProp1");
        this.prop1 = prop1;
    }

    public void setProp2(String prop2) {
        System.out.println("In setProp2");
        this.prop2 = prop2;
    }

    public String getProp1() {
        return prop1;
    }

    public String getProp2() {
        return prop2;
    }*/

    public void display() {
        System.out.println("Prop1 is " + prop1);
        System.out.println("Prop2 is " + prop2);
    }

    public void initialize() {
        System.out.println("In initialize");
        this.prop1 = "init-prop1";
        this.prop2 = "init-prop2";
    }

    @PostConstruct
    public void preConstruct() {

        System.out.println("invoke after a bean is initialize");

    }
  
    @Override
    public String toString() {
        i=i+1;
        return "InitTestBean [prop1=" + prop1 + ", prop2=" + prop2 + "]"+ i ;
    }

    public void teardown() {
        System.out.println("In teardown");
        this.prop1 = null;
        this.prop2 = null;
    }

    public static void main(String[] args) {
        ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext(
                "InitTestContext.xml");
        InitTestBean bean = (InitTestBean) ctx.getBean("InitTestBean");
        System.out.println(bean);
        InitTestBean bean2 = (InitTestBean) ctx.getBean("InitTestBean");
        System.out.println(bean2);
        bean.display();
        bean2.display();
        ctx.close();

    }
}


result 

Instantiating InitTestBean
invoke after a bean is initialize
InitTestBean [prop1=Prop1, prop2=Prop2]1
InitTestBean [prop1=Prop1, prop2=Prop2]2
Prop1 is Prop1
Prop2 is Prop2

Prop1 is Prop1
Prop2 is Prop2

In teardown


in above demo , to_string method when a bean is  initialize  it is not execute , while  postconstruct  is invoke when a bean is initialize (intiazlie  once)