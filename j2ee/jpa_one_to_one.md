@OneToOne: This is a type of annotation that can be used for one-to-one relationship. I contains two types of attributes: 1. cascade 2. fetch.

1. cascade: It allows you to which operations performed with its related to child. Theare are different types of cascade:

ALL: If you use this then you are able to do all of the following operations with child table.

PERSIST: It only allows to insertion with child table.

MERGE: It allows you to update records to related child table.

REMOVE: If you want to remove records from database table then you use this facility.

2. featch: The fetch attribute allows you to which policy is used by user when loading a relation. We loading a relation two types eithor LAZY or EAGER types.

LAZY: This property to load data later on when you needed.

EAGER: This property to load data immediately when you retrive data from databse table.

@JoinColumn: This annotation has same attributes as the @Column. It takes a field which is used in databse as a foreign key into your database and don't allow null value (nullable = false).



package com.xf.jpa.onotoone;
import java.io.Serializable;

import javax.persistence.CascadeType;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.FetchType;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.OneToOne;
import javax.persistence.Table;

@Entity
@Table(name="parent")
public class Parent implements Serializable{
  @Id
  @GeneratedValue
  @Column(name="id")
  private int id;
  @Column(name="pfirstName")
  private String pfirstName;
  @Column(name="plastName")
  private String plastName;
  @Column(name="parentAge")
  private int parentAge;
  @OneToOne(cascade=CascadeType.ALL,
 fetch=FetchType.EAGER)
  @JoinColumn(name="cId")  //cId is constraint in Parent
  private Child child;
  
  public int getId() {
    return id;
  }
  public void setId(int id) {
    this.id = id;
  }
  public String getPfirstName() {
    return pfirstName;
  }
  public void setPfirstName(String pfirstName) {
    this.pfirstName = pfirstName;
  }
  public String getPlastName() {
    return plastName;
  }
  public void setPlastName(String plastName) {
    this.plastName = plastName;
  }
  public int getParentAge() {
    return parentAge;
  }
  public void setParentAge(int parentAge) {
    this.parentAge = parentAge;
  }
  public Child getChild() {
    return child;
  }
  public void setChild(Child child) {
    this.child = child;
  }
}


package com.xf.jpa.onotoone;
import java.io.Serializable;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.OneToOne;
import javax.persistence.Table;

@Entity
@Table(name="Child")
public class Child implements Serializable{
  @Id
  @GeneratedValue
  @Column(name="id")
  private int id;
  @Column(name="cfirstName")
  private String cfirstName;
  @Column(name="clastName")
  private String clastName;
  @Column(name="childAge")
  private int childAge;  
  
  public int getId() {
    return id;
  }
  public void setId(int id) {
    this.id = id;
  }
  public String getCfirstName() {
    return cfirstName;
  }
  public void setCfirstName(String cfirstName) {
    this.cfirstName = cfirstName;
  }
  public String getClastName() {
    return clastName;
  }
  public void setClastName(String clastName) {
    this.clastName = clastName;
  }
  public int getChildAge() {
    return childAge;
  }
  public void setChildAge(int childAge) {
    this.childAge = childAge;
  }

}




CREATE TABLE `parent` (
   `id` int(11) NOT NULL auto_increment,
   `parentAge` int(11) default NULL,
   `pfirstName` varchar(255) default NULL,
   `plastName` varchar(255) default NULL,
   `cId` int(11) default NULL,
   PRIMARY KEY  (`id`),
   KEY `FKC4AB08AACD2DFD9A` (`cId`),
   CONSTRAINT `FKC4AB08AACD2DFD9A` 
FOREIGN KEY (`cId`) REFERENCES `child` (`id`)
 ) ENGINE=InnoDB DEFAULT CHARSET=latin1
 

Table: Child

CREATE TABLE `child` (
   `id` int(11) NOT NULL auto_increment,
   `cfirstName` varchar(255) default NULL,
   `childAge` int(11) default NULL,
   `clastName` varchar(255) default NULL,
   PRIMARY KEY  (`id`)
 ) ENGINE=InnoDB DEFAULT CHARSET=latin1