Java EE 6 supports a suite of five scopes, each with its own behavior for managing the user's interaction with a Java Web application. Find out how and when to use them.
by Leonard Anghel
As you probably know, scopes are a very important aspect of a Web application. The scope notion represents the idea that an object belongs to a certain part of an application. So, you can use scopes for holding state over the duration of the user's interaction with the application. In Java Web applications, request and session scopes are the most commonly used but the Java EE 6 specification supports a suite of five scopes, any of which can accept an object.

In this article, I will present each of the Java EE 6 scopes in the context of a Java Bean and I will demonstrate its behavior through a simple JSF/XHTML page.

Java EE 6 Scopes Overview

The Java EE 6 scopes are summarized in the table below, which presents the scope annotation, scope class and a short description for each one. I will detail each of them in the sections to follow.

Scope

Scope Class

Scope Annotation

Short Description

Dependent

javax.enterprise.context.DependentScoped

@Dependent

Default scope

Request

javax.enterprise.context.RequestScoped

@Request

Single HTTP request/user

Session

javax.enterprise.context.SessionScoped

@Session

Multiple HTTP requests/user

Conversation

javax.enterprise.context.ConversationScoped

@Conversation

A short-lived/controlled session

Application

javax.enterprise.context.ApplicationScoped

@Application

All users share the same state



Java EE 6 Request Scope

The request scope is very useful in any Web application, and an object defined in the request scope usually has a short lifespan. When the container accepts an HTTP request from the client, the specified object is attached to the request scope and it is released when the container has finished transmitting the response to that request. A new HTTP request always comes in a new request scope object. In short, a request scope represents a user's interaction with a Web application in a single HTTP request.

In Java EE 6, you place a Java Bean under the request scope by annotating the Bean with @RequestScoped and importing the javax.enterprise.context.RequestScoped class. For example, this Bean generates random numbers and stores them in an ArrayList:

package rnd.beans;
import java.util.ArrayList;
import java.util.Random;
import javax.enterprise.context.RequestScoped;
import javax.inject.Named;
@Named(value = "rndBeanRequest")
@RequestScoped
public class rndBeanRequest {
private ArrayList rnds = new ArrayList();
private int rnd = 0;
/** Creates a new instance of rndBeanRequest */
public rndBeanRequest() {
}
public int getRnd() {
return rnd;
}
public void setRnd(int rnd) {
this.rnd = rnd;
}
public ArrayList getRnds() {
return rnds;
}
public void setRnds(ArrayList rnds) {
this.rnds = rnds;
}
public void newRnd() {
this.rnd = new Random().nextInt(100);
this.rnds.add(rnd);
}
}
Note: The @Named annotation is used by Java EE 6 applications to make the Bean accessible via EL (EL stands for Expression Language, used by JSPs and JSF components).

For example, you can build a simple test for the above Java Bean using JavaServer Faces (JSF):

Just generated:
<h:outputText value="#{rndBeanRequest.rnd}"/>


List of generated numbers:
<h:dataTable var="t" value="#{rndBeanRequest.rnds}">
<h:column>
<h:outputText value="#{t}"/>
</h:column>
</h:dataTable>
<h:form>
<h:commandButton value="Get Random" actionListener="#{rndBeanRequest.newRnd}" action="index.xhtml"/>
</h:form>
You can see the effect of the request scope very easily:

Each time you press the Get Random button, a new number is generated.
Most importantly, you can't keep a list of generated numbers because for every request a new Bean instance is created and a new empty list is created and filled with only the last random number generated. This makes the list useless.



Java EE 6 Session Scope

Request scope is very useful in any Web application when you need a single interaction per HTTP request. However, when you want a user's interaction with a Web application across multiple HTTP requests, that's when the session scope goes into action. Session scope allows you to create and bind objects to a session. For example, you can add the session scope to your random generator for storing a list of generated numbers across multiple requests.

In Java EE 6, you place a Java Bean under session scope by annotating the Bean with @SessionScoped and importing the javax.enterprise.context.SessionScoped class. For example, this Bean generates random numbers and stores them into an ArrayList:

package rnd.beans;
import java.io.Serializable;
import java.util.ArrayList;
import java.util.Random;
import javax.inject.Named;
import javax.enterprise.context.SessionScoped;
@Named(value = "rndBeanSession")
@SessionScoped
public class rndBeanSession implements Serializable{
/** Creates a new instance of rndBeanSession */
public rndBeanSession() {
}
private ArrayList rnds = new ArrayList();
private int rnd = 0;
public int getRnd() {
return rnd;
}
public void setRnd(int rnd) {
this.rnd = rnd;
}
public ArrayList getRnds() {
return rnds;
}
public void setRnds(ArrayList rnds) {
this.rnds = rnds;
}
public void newRnd() {
this.rnd = new Random().nextInt(100);
this.rnds.add(rnd);
}
}
For example, you can build a simple test for the above Java Bean using JSF:

Just generated:
<h:outputText value="#{rndBeanSession.injrnd}"/>


List of generated numbers:
<h:dataTable var="t" value="#{rndBeanSession.rnds}">
<h:column>
<h:outputText value="#{t}"/>
</h:column>
</h:dataTable>
<h:form>
<h:commandButton value="Get Random" actionListener="#{rndBeanSession.newRnd}" action="index.xhtml"/>
</h:form>
Now, when you press the Get Random button multiple times, each generated number is stored into the rnds list. This time, the list stores those numbers for real, which is proving the effect of session scope. The output shows you a list of numbers representing the user's interaction with the Web application across multiple HTTP requests.

Java EE 6 Application Scope

An application scope extends session scope with shared state across all users' interactions with a Web application. More precisely, objects settled on application scope can be accessed from any page that is part of the application (e.g. JSF, JSP, XHTML, etc.). Usually, application scope objects are used as counters. For example, application scope can be used to count how many users are online or to share that information with all users.

In Java EE 6, you place a Java Bean under the application scope by annotating the Bean with @ApplicationScoped and importing the javax.enterprise.context.ApplicationScoped class. For example, this Bean generates random numbers and stores them in an ArrayList shared with all users by using the application scope.

package rnd.beans;
import java.util.ArrayList;
import java.util.Random;
import javax.inject.Named;
import javax.enterprise.context.ApplicationScoped;
@Named(value="rndBeanApplication")
@ApplicationScoped
public class rndBeanApplication {
/** Creates a new instance of rndBeanApplication */
public rndBeanApplication() {
}
private ArrayList rnds = new ArrayList();
private int rnd = 0;
public int getRnd() {
return rnd;
}
public void setRnd(int rnd) {
this.rnd = rnd;
}
public ArrayList getRnds() {
return rnds;
}
public void setRnds(ArrayList rnds) {
this.rnds = rnds;
}
public void newRnd() {
this.rnd = new Random().nextInt(100);
this.rnds.add(rnd);
}
}
Testing this Bean with multiple machines will reveal the same list of random numbers. Here is a possible JSF code for this:

Just generated:
<h:outputText value="#{rndBeanApplication.rnd}"/>


List of generated numbers:
<h:dataTable var="t" value="#{rndBeanApplication.rnds}">
<h:column>
<h:outputText value="#{t}"/>
</h:column>
</h:dataTable>
<h:form>
<h:commandButton value="Get Random" actionListener="#{rndBeanApplication.newRnd}" action="index.xhtml"/>
</h:form>



Java EE 6 Conversation Scope

Conversation scope is dedicated to the user's interaction with JSF applications and represents a unit of work from the point of view of the user. Think of conversation scope as a developer-controlled session scope across multiple invocations of the JSF lifecycle. The developer can explicitly set the conversation scope boundaries and can start, stop or propagate the conversation scope based on business logic flow. All long-running conversations are scoped to a particular HTTP Servlet session and may not cross session boundaries. In addition, conversation scope holds the state associated with a particular Web browser tab in a JSF application.

Working with conversation scope is a little bit different from the rest of the scopes. First, you mark the Bean with @ConversationScope, represented by the javax.enterprise.context.ConversationScoped class. Second, CDI  provides a built-in Bean (javax.enterprise.context.Conversation) for controlling the lifecycle of conversations in a JSF application. This Bean may be obtained by injection, like this:

private @Inject Conversation conversation;
Now, the current request should be transformed into a long-running conversation by calling the begin() method. You also need to prepare for the destruction of the conversation by calling the end() method.

In the following example, a conversation-scoped Bean controls the conversation with which it is associated. Our Bean will store the generated random numbers for only the length of the conversation:

package rnd.beans;
import java.io.Serializable;
import java.util.ArrayList;
import java.util.Random;
import javax.enterprise.context.Conversation;
import javax.inject.Named;
import javax.enterprise.context.ConversationScoped;
import javax.inject.Inject;
@Named(value="rndBeanConversation")
@ConversationScoped
public class rndBeanConversation implements Serializable{
private @Inject Conversation conversation;
/** Creates a new instance of rndBeanConversation */
public rndBeanConversation() {
}
private ArrayList rnds = new ArrayList();
private int rnd = 0;
public int getRnd() {
return rnd;
}
public void setRnd(int rnd) {
this.rnd = rnd;
}
public ArrayList getRnds() {
return rnds;
}
public void setRnds(ArrayList rnds) {
this.rnds = rnds;
}
public void newRnd() {
this.rnd = new Random().nextInt(100);
this.rnds.add(rnd);
}
public void startRnd(){
conversation.begin();
}
public void stopRnd(){
conversation.end();
}
}
Moreover, the user can control when to start/stop the conversation through a set of JSF buttons:

Just generated:
<h:outputText value="#{rndBeanConversation.rnd}"/>


List of generated numbers:
<h:dataTable var="t" value="#{rndBeanConversation.rnds}">
<h:column>
<h:outputText value="#{t}"/>
</h:column>
</h:dataTable>
<h:form>
<h:commandButton value="Start Conversation" actionListener="#{rndBeanConversation.startRnd}" action="index.xhtml"/>
<h:commandButton value="Get Random" actionListener="#{rndBeanConversation.newRnd}" action="index.xhtml"/>
<h:commandButton value="Stop Conversation" actionListener="#{rndBeanConversation.stopRnd}" action="index.xhtml"/>
</h:form>
In this example, the conversation context automatically propagates with any JSF faces request or redirection (this facilitates the implementation of the common POST-then-redirect pattern), but it does not automatically propagate with non-faces requests such as links. In this case, you need to include the unique identifier of the conversation as a request parameter. The CDI specification reserves the request parameter cid for this use. The below example will propagate the conversation context over a link:

 <h:link outcome="/link.xhtml" value="Conversation Propagation">
<f:param name="cid" value="#{conversation.id}"/>
</h:link>
Java EE 6 Dependent Scope

This is the default scope when none is specified. In this case, an object exists to serve exactly one Bean and has the same lifecycle as that Bean. It also can be explicitly specified by annotating the Bean with the @Dependent annotation and import javax.enterprise.context.Dependent. The Bean below shows this:

 package rnd.beans;
import java.util.ArrayList;
import java.util.Random;
import javax.enterprise.context.Dependent;
import javax.inject.Named;
@Named(value = "rndBeanDependent")
@Dependent
public class rndBeanDependent {
private ArrayList rnds = new ArrayList();
private int rnd = 0;
/** Creates a new instance of rndBeanDependent */
public rndBeanDependent() {
}
public int getRnd() {
return rnd;
}
public void setRnd(int rnd) {
this.rnd = rnd;
}
public ArrayList getRnds() {
return rnds;
}
public void setRnds(ArrayList rnds) {
this.rnds = rnds;
}
public void newRnd() {
this.rnd = new Random().nextInt(100);
this.rnds.add(rnd);
}
}
The JSF code for testing the above Bean is:

Just generated:
<h:outputText value="#{rndBeanDependent.rnd}"/>


List of generated numbers:
<h:dataTable var="t" value="#{rndBeanDependent.rnds}">
<h:column>
<h:outputText value="#{t}"/>
</h:column>
</h:dataTable>
<h:form>
<h:commandButton value="Get Random" actionListener="#{rndBeanDependent.newRnd}" action="index.xhtml"/>
</h:form>
Comparing Java EE 6 Scopes

Grouping the above Beans into a single application will allow you to test and see the differences between the Java EE 6 scopes. Figure 1 below is a screenshot test that reveals each scope's advantages and disadvantages.
How Java EE 6 Scopes Affect User Interactions - 龙二 - xiangfei_2011的博客