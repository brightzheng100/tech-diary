# JDKs and IDEs Tips & FAQ

## Running Multiple JDKs

Ref: [https://gist.github.com/brightzheng100/20bd34c37367219340bdb359040370dd\#file-multi-java-version-support-md](https://gist.github.com/brightzheng100/20bd34c37367219340bdb359040370dd#file-multi-java-version-support-md)

To support multiple Java version, with **ZSH**:

**1. Pick The Right JDKs with Right Versions**

For example, we're going to have two JDKs, v8 and v11, with HotSpot installed.

```bash
$ export JDK8=adoptopenjdk8
$ export JDK11=adoptopenjdk11
```

Or if OpenJDK with OpenJ9 \(instead of HotSpot\)

```bash
$ export JDK8=adoptopenjdk8-openj9
$ export JDK11=adoptopenjdk11-openj9
```

> Note: check the available versions [here](https://github.com/AdoptOpenJDK/homebrew-openjdk)

**2. Install Multiple Java Versions:**

```bash
$ brew cask install ${JDK8}
$ brew cask install ${JDK11}
```

**3. Make then switchable, with a default version:**

```bash
$ cat <<EOF >> ~/.zshrc
# Multiple Java Versions
export JAVA_8_HOME=$(/usr/libexec/java_home -v1.8)
export JAVA_11_HOME=$(/usr/libexec/java_home -v11)

alias java8='export JAVA_HOME=$JAVA_8_HOME'
alias java11='export JAVA_HOME=$JAVA_11_HOME'

#default java11
export JAVA_HOME=$JAVA_11_HOME
EOF
```

Now, to switch versions, do this:

```bash
$ java8
$ java -version
openjdk version "1.8.0_232"
OpenJDK Runtime Environment (AdoptOpenJDK)(build 1.8.0_232-b09)
OpenJDK 64-Bit Server VM (AdoptOpenJDK)(build 25.232-b09, mixed mode)

$ java11
$ java -version
openjdk version "11.0.4" 2019-07-16
OpenJDK Runtime Environment AdoptOpenJDK (build 11.0.4+11)
OpenJDK 64-Bit Server VM AdoptOpenJDK (build 11.0.4+11, mixed mode)
```

> Note: as the process has nothing special, you may simply do it in `~/.bash_profile` too with pure Bash env

## IDE Failed To Start: failed to create java virtual machine.

Ref: [https://gist.github.com/brightzheng100/20bd34c37367219340bdb359040370dd\#file-sts-failed-to-start-md](https://gist.github.com/brightzheng100/20bd34c37367219340bdb359040370dd#file-sts-failed-to-start-md)

Sometimes STS/Eclipse failed to start: failed to create java virtual machine.

It might be caused by multiple JDKs installed.

So we have to specify if it helps.

```bash
$ sudo vi /Applications/STS.app/Contents/Eclipse/STS.ini
...
-vm
/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home/bin/java
-vmargs
...
```

> Note: `-vm` must be before `-vmargs`!

## IDE with `lombok` support

Project Lombok is cool, but it's not installed by default for most of the IDEs. So a simple installation process might be required.

**For STS/Eclipse**

1. We may download the `lombok.jar` from [here](https://projectlombok.org/download), or use the one in your `~/.m2/repository/org/projectlombok/` folder;
2. Run `java -jar <THE-PATH-TO-YOUR-JAR>`;
3. It will prompt up an UI for you to pick up what IDEs \(e.g. Eclipse, STS\) to install for, choose yours;
4. Click install button and it's done.

> Note: for STS/Eclipse, it will actually copy the jar file to IDE's folder and update its .ini file. For example, it adds below in `/Applications/STS.app/Contents/Eclipse/STS.ini` file:
>
> ```text
> ...
> -javaagent:/Applications/STS.app/Contents/Eclipse/lombok.jar
> ```

**For VS Code**

There is an addon we can install to make VS Code have lombok support. Search this addon by keyword `lombok` will do.

