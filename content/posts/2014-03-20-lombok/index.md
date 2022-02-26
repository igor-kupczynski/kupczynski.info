---
title: Project Lombok
tags:
- java
aliases:
- /2014/03/20/lombok.html
---
Never write your getters again!

My college recently introduced
[Project Lombok](http://projectlombok.org/) to our java code
base. This is a great tool, which aims to make java source code less
verbose by using a compile time
[annotation processor](http://deors.wordpress.com/2011/10/08/annotation-processors/). In
this post I give you a basic example of usage of Lombok and a
description on how to configure IntelliJ Idea to use it.

# Basic usage

To give a quick example ([src](http://jnb.ociweb.com/jnb/jnbJan2010.html#data))

This piece of code (notice `@Data` annotation) with Project Lombok

    @Data(staticConstructor="of")
    public class Company {
        private final Person founder;
        private String name;
        private List<Person> employees;
    }

Will result in the same byte code as this plain vanilla java

    public class Company {
        private final Person founder;
        private String name;
        private List<Person> employees;

        private Company(final Person founder) {
            this.founder = founder;
        }

        public static Company of(final Person founder) {
            return new Company(founder);
        }

        public Person getFounder() {
            return founder;
        }

        public String getName() {
            return name;
        }

        public void setName(final String name) {
            this.name = name;
        }

        public List<Person> getEmployees() {
            return employees;
        }

        public void setEmployees(final List<Person> employees) {
            this.employees = employees;
        }

        @java.lang.Override
        public boolean equals(final java.lang.Object o) {
            if (o == this) return true;
            if (o == null) return false;
            if (o.getClass() != this.getClass()) return false;
            final Company other = (Company)o;
            if (this.founder == null ? other.founder != null : !this.founder.equals(other.founder)) return false;
            if (this.name == null ? other.name != null : !this.name.equals(other.name)) return false;
            if (this.employees == null ? other.employees != null : !this.employees.equals(other.employees)) return false;
            return true;
        }

        @java.lang.Override
        public int hashCode() {
            final int PRIME = 31;
            int result = 1;
            result = result * PRIME + (this.founder == null ? 0 : this.founder.hashCode());
            result = result * PRIME + (this.name == null ? 0 : this.name.hashCode());
            result = result * PRIME + (this.employees == null ? 0 : this.employees.hashCode());
            return result;
        }

        @java.lang.Override
        public java.lang.String toString() {
            return "Company(founder=" + founder + ", name=" + name + ", employees=" + employees + ")";
        }
    }


With the `@Data` annotation Lombok will generate getter, setters,
constructor, static factory method, `.hashCode` + `.equals` and even
`.toString`. Isn't it great? Never write your getters again!

# Configuring Idea

The only caveat is that you need to configure your IDE and the build
process. The latter is easy as you need only to include `lombok.jar`
in your classpath. The former is a bit trickier if you use IntelliJ
Idea (eclipse is already described on the
[project page](http://jnb.ociweb.com/jnb/jnbJan2010.html#installation)).

To use Lombok with Idea you need to:

1. Install [lombok-intellij-plugin](https://github.com/mplushnikov/lombok-intellij-plugin):

   a. Go to:  Settings -> Plugins -> Browse repositories...

   b. Select `Lombok Plugin`, install & restart ide

2. Add `lombok.jar` as a global library:

   a. Go to: Project Structure -> Global Libraries -> '+'

   b. Select the `lombok.jar`

   c. Add it to the modules you want to have Lombok enabled (
      Project Structure -> Modules -> *Your Module* -> '+' -> Library)

3. Enable the plugin for the project + enable annotation processing:

   a. Settings -> Lombok Plugin -> Enable Lombok support for this project

   b. Settings -> Compiler -> Annotation Processors -> Enable annotation processing

4. Restart your ide again and you are good to go!

# Further reading

There is more to Lombok. Read on:
- [Reducing Boilerplate Code with Project Lombok](http://jnb.ociweb.com/jnb/jnbJan2010.html)
- [Features](http://projectlombok.org/features/index.html)
