---
title: Remote Debug Junit Tests Run From Ant
tags:
- java
aliases:
- /2013/08/08/remote-debug-junit-tests-run-from-ant.html
---
How to turn on remote debugging for junit testrunner in ant.

Sometimes you want to run a testset in an isolated environment. Be it a
different box, a virtual machine, a different user or just a console and not
an IDE. Reasons may vary, e.g. you need a special setup for the integration
tests or you have a fancy build configuration which is hard to reproduce in
the IDE. Still, no matter what your reasons are, you want to debug such tests.
In this post I'll show how to use a remote debugger to connect to a test
environment using IntelliJ Idea or Eclipse. I assume you have an ant build.

To quickly warm up - Java enables you to remotely debug a JVM instance. For
more technical details visit the [Java Platform Debugger Architecture
documentation][jpda]. To follow this post it is enough to know that you can
run JVM with the remote debugging enabled. JVM opens a port and allows
debuggers, e.g. Eclipse or Idea IDE to connect to it. By connecting an IDE to
a running JVM you can use the debugged in pretty much the same way as you use
it for apps run from the IDE itself.

[jpda]: http://docs.oracle.com/javase/6/docs/technotes/guides/jpda/architecture.html

In order to enable the debug mode pass following options to JVM:
	
	-Dagentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=8787

If you set `suspend` to `y` the JVM will start up, but will not execute any
code before you connect the debugger. This is quite useful in our case -
debugging a test set. The `address` parameter specifies which port will be
used for debugging.

Having rehashed the remote debugging topic we can carry on. The ant test
target may look like this:

	<target name="test" depends="test-compile">
		<junit showoutput="yes" fork="true">
			<classpath><path refid="classpath.test"/></classpath>
			<formatter type="plain" usefile="false"/>
			<batchtest>
				<fileset dir="${build.test.dir}" includes="**/*Test.class"/>
			</batchtest>
		</junit>
	</target>

All we need to do is to pass the arguments:

    <target name="test" depends="test-compile">
        <junit showoutput="yes" fork="true">

            <jvmarg
             value="-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=8787"
            />

            <classpath><path refid="classpath.test"/></classpath>
            <formatter type="plain" usefile="false"/>
            <batchtest>
                <fileset dir="${build.test.dir}" includes="**/*Test.class"/>
            </batchtest>
        </junit>
    </target>

The effect of `ant test` is:

	test:
    	[junit] Listening for transport dt_socket at address: 8787

It works, but it is far from perfection. Surly we do not want all our builds to
wait and rely on us connecting the debugger. The better way is to make the
remote debugger dependent on a property we can pass. Like this:

	<property name="remoteDebug" value="false"/>

    <target name="test" depends="test-compile">
        <condition property="remoteDebugJvmArgs"
          value="-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=8787"
          else="-ea">
            <istrue value="${remoteDebug}"/>
        </condition>
        <junit showoutput="yes" fork="true" forkmode="perTest">

            <jvmarg value="${remoteDebugJvmArgs}"/>

            <classpath><path refid="classpath.test"/></classpath>
            <formatter type="plain" usefile="false"/>
            <batchtest>
                <fileset dir="${build.test.dir}" includes="**/*Test.class"/>
            </batchtest>
        </junit>
    </target>

We make use of ant immutable properties. The property `remoteDebug` is set to
`false`, but if we specify it via the command line
	
	ant test -DremoteDebug=true

ant will not change it. In the test task we have a condition - if the
`remoteDebug` property is true, then use the remote debug arguments. _Et
voilà!_ To run a regular build do `ant test` and to let it wait for a remote
debugger do `ant test -DremoteDebug=true`.

There is one caveat here. If `remoteDebug` is false, we pass the `-ea` option
to JVM. The reason is that we need to pass something - otherwise JVM will
complain. `-ea` enables assertions for a package which has the benefit of
doing nothing if no package is specified. You can read more on this issue in
the [ant-users] mailing list.

[ant-users]: http://ant.1045680.n5.nabble.com/problem-with-java-task-attribute-jvmarg-when-value-is-quot-quot-td1340083.html

The other thing to keep an eye on are the `fork` and `forkmode` parameters.
For `jvmarg` to have any effect `fork` must be set to `true`. Otherwise ant
won't spawn new JVM for the tests - it means no opportunity to pass `jvmarg`s.
`forkmode` controls whether new JVMs are spawned. It's default value is
`perTest` which means a new JVM instance for each test class. If you debug
your tests one-by-one this is fine (and usually this is what you want to do -
just debug a single test). On the other hand, if you want to run a full set of
tests you may consider changing it to `once` - this way it won't ask you to
connect a debugger for every test class. Please consult the [junit task
documentation ][ant-junit-docs] for more details.

[ant-junit-docs]: http://ant.apache.org/manual/Tasks/junit.html

The only thing which remains is how to connect to this JVM using your favorite
IDE. It is quite simple actually.

In IntelliJ Idea:

1. Go to `Run` -> `Edit Configurations...`,
2. Click `Add New Configuration` and select `Remote`,
3. Modify the parameters, e.g. the port number,
4. Save and debug using this configuration.

![Remote debugging in Idea](/archive/2013-08-08-remote-idea.png)

In Eclipse:

1. Open `Debug Configuration` from `Run Menu`,
2. Select new `Remote Java Application`,
3. Enter the required parameters (esp. host and port),
4. Save and debug using this configuration.

And that's it. Please not that you need the source code in your IDE for the
debugger to be useful.

If you want a working example of an ant build with an option to remotely debug
tests you can consult [my playground project at githib][oracle-playground].
Use tag `2013-08-08-ant-remote-debug`.

[oracle-playground]: https://github.com/puszczyk/oracle-demo-project
