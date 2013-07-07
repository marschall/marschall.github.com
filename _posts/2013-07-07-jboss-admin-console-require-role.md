---
layout: post
title: Require a Role for the JBoss AS 7 Admin Console 
---

The [Admin Console](https://github.com/jbossas/console) in JBoss AS 7 is no longer a WAR deployed with all the other applications. Therefore you can no longer just edit its `web.xml` if you want change how users are authorized to access the Admin Console. JBoss AS 7 supports three ways to authorize users for  the Admin Console (or remoting): a `.properties` file, LDAP and a JAAS domain. In our case we already have a JAAS domain for the applicationn and we would like the reuse the same domain for the Admin Console but in addition require a specific role.

This is not supported out of the box but can be implemented with a rather small effort. We create a new domain that delegates to the existing domain and additionally checks for the required role. This can be done using a JAAS login module. The [following blog post](http://pilhuhn.blogspot.ch/2013/05/creating-delegating-login-module-for.html) explains how to implement a login module that delegates to an other one. It requires only small changes to work in our case. 

First in the `#initialize` method we read which role is required

{% highlight java %}
    String requiredRoleName = (String) options.get("requiredRole");

    if (delegateTo == null || delegateTo.isEmpty()) {
      delegateTo = "other";
      LOG.warn("module-option 'delegateTo' was not set. Defaults to 'other'.");
    }
    this.requiredRole = new SimplePrincipal(requiredRoleName);
}
{% endhighlight %}

Then in the `#login` method we check for the role

{% highlight java %}
  private boolean isInGroup() {
    Subject otherSubject = loginContext.getSubject();
    Set<Group> groups = otherSubject.getPrincipals(Group.class);
    for (Group group : groups) {
      if (group.isMember(this.requiredRole)) {
        return true;
      }
    }
    return false;
{% endhighlight %}


All that is left is configuring our new module, "acmeDomain" is the default domain for the application and "acmeAdminDomain" is the domain for the Admin Console that requires the role "ADMIN_CONSOLE".

{% highlight xml %}
<security-domains>
  <!-- other domains -->
  <security-domain name="acmeDomain" cache-type="default">
    <authentication>
      <login-module code="Remoting" flag="optional">
        <module-option name="password-stacking" value="useFirstPass"/>
      </login-module>
      <login-module code="com.acme.security.AcmeLoginModule" flag="required" module="com.acme.jboss.security">
        <module-option name="dsJndiName" value="java:jboss/datasources/AcmeDS"/>
      </login-module>
    </authentication>
  </security-domain>
  <security-domain name="acmeAdminDomain" cache-type="default">
    <authentication>
      <login-module code="Remoting" flag="optional">
        <module-option name="password-stacking" value="useFirstPass"/>
      </login-module>
      <login-module code="com.acme.jboss.security.loginmodule.DelegatingLoginModule" flag="required" module="com.acme.jboss.security">
        <module-option name="delegateTo" value="acmeDomain"/>
        <module-option name="requiredRole" value="ADMIN_CONSOLE"/>
      </login-module>
    </authentication>
  </security-domain>
</security-domains>
{% endhighlight %}

And finally make the Admin Console use it

{% highlight xml %}
<management>
  <security-realms>
    <security-realm name="AcmeAdminRealm">
      <authentication>
        <jaas name="acmeAdminDomain"/>
      </authentication>
    </security-realm>
  </security-realms>
  <management-interfaces>
    <native-interface security-realm="AcmeAdminRealm">
      <socket-binding native="management-native"/>
    </native-interface>
    <http-interface security-realm="AcmeAdminRealm">
      <socket-binding http="management-http"/>
    </http-interface>
  </management-interfaces>
</management>
{% endhighlight %}

That's it, we're quite happy with the result, especially since we now no longer need to modify the Admin Console to require a specific role.


