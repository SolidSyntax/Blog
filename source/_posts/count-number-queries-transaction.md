title: Count the number of queries in a transaction
tags:
  - Java
  - JDBC
  - Performance
id: 213
categories:
  - Thoughts
date: 2015-01-28 16:18:54
---

Some time ago we upgraded our connection pool from [Apache Commons DBCP](http://commons.apache.org/dbcp/) to [The Tomcat JDBC Connection Pool](http://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html). We did so mostly because of the improved concurrency handling but soon found out that the addition of [JDBC interceptors](http://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html#JDBC_interceptors) was a useful resource in locating performance issues.
[SlowQueryReport](http://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html#org.apache.tomcat.jdbc.pool.interceptor.SlowQueryReport) is one of the included interceptors. It logs queries which take longer to execute then the specified threshold. 
While this information can be very valuable it doesn’t quite tell the complete story.

<!-- more-->

It could be that a certain unit of work (a transaction, a method call) doesn't log a single slow query and still takes a long time to complete because it executes a lot of queries. To count the number of queries in a transaction I've created a CountQueryReport extension for the Tomcat JDBC Connection Pool. It is a very simple extension which will count the number of statements (per thread) executed between a call to 'resetCount()' and 'numberOfStatements()'.

Configuration example (for a Spring environment):
{% codeblock lang:xml %}
<bean id="myDataSource" class="org.apache.tomcat.jdbc.pool.DataSource">
    ...
    <!-- Log queries -->
    <property name="jdbcInterceptors" value="net.solidsyntax.jdbc.pool.interceptor.CountQueryReport" />
</bean>
{% endcodeblock %}

Usage in a basic Servlet filter:
{% codeblock lang:java %}
public class CountQueryFilter implements Filter {

    public void init(FilterConfig filterConfig) throws ServletException {}

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {

        //reset the statement counter for the current thread
        CountQueryReport.resetCount();

        //go down the filter chain
        chain.doFilter(request,response);

        //display the number of statement executed
        System.out.println("Number of statements executed: " + CountQueryReport.numberOfStatements());
    }

    public void destroy() {}

}
{% endcodeblock %}
You can download the distribution zip [here](/2015/01/28/count-number-queries-transaction/
/CountQueryReportDist.zip) (which contains the binaries, sources and Javadocs).
Or have a look at the code on [GitHub](https://github.com/SolidSyntax/CountQueryReport).