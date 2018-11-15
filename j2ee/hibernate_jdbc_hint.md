1. hibernate jpa doesn't support oracle jp hint

2 resolve  em.createNativeSql() 


demo

package com.xf.test;

import java.math.BigDecimal;
import java.util.HashSet;
import java.util.Set;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;
import javax.persistence.Query;

import org.hibernate.CacheMode;

import com.xf.entity.Father;
import com.xf.entity.Son;

public class TestEm {
public static void main(String[] args) {
    EntityManagerFactory  emf = Persistence.createEntityManagerFactory("MyJpa");
    EntityManager em = emf.createEntityManager();
    em.getTransaction().begin();
    long begin =System.currentTimeMillis();
    Query  q =  em.createQuery("select   sonName  from Son");
   // q.setHint("org.hibernate.comment"," /*+FIRST_ROWS*/");
    q.setHint("javax.persistence.cache.retrieveMode","BYPASS");
    
    q.getResultList();
    long end = System.currentTimeMillis();
    System.out.println(end-begin);
    
    Query  q1 =  em.createNativeQuery("select /*+FIRST_ROWS*/  SON_NAME from Son" ,Son.class);
      q1.getResultList();
   
/*    Son s = new Son();
    Set<Son>  sss = new  HashSet();
    Son s1 = new Son();
    s1.setSonAge(new BigDecimal(11));
    s1.setSonName("s1_name");
    sss.add(s);
    sss.add(s1);
    s.setSonAge(new BigDecimal(12));
    s.setSonName("s_name");
    //s.setFather(f);
    Father f =new Father();
    f.setFatherAge(new BigDecimal(32));
    f.setFatherName("f_name");
    f.setSons(sss);
    s.setFather(f);
    em.merge(f);
    em.remove(s); */
    em.getTransaction().commit();
    
    
     
}
}


result 


Hibernate: select son0_.SON_NAME as col_0_0_ from Son son0_
156
Hibernate: select /*+FIRST_ROWS*/  SON_NAME from Son