---
layout: post
title:  "Code Obfuscation"
date:   2018-07-02 14:37:00 +0800
categories: niagara
---

>ProGuard successfully processes any Java bytecode, ranging from small Android applications to entire run-time libraries. It primarily reduces the size of the processed code, with some potential increase in efficiency as an added bonus. The improvements obviously depend on the original code.

## ProGuard GUI

The Niagara Regisry is a database of these types which allows the Niagara Framework to correctly create instances of a given Type. SO these Niagara Types must be kept in ProGuard configuration:

```
-keep,includedescriptorclasses class * extends javax.baja.sys.BObject {
    public static final <fields>;
    public protected <methods>;
}
```

## Gradle Build

ProGuard also provides a Gradle task, so that it integrates into your Gradle build process. You can specify configurations in ProGuard's own format or embedded in the Groovy configuration.

## Maven Build

Use ___idfc-proguard-maven-plugin___ to obfuscate Maven artifacts:

```xml
<profile>
    <id>obfuscate</id>
    <build>
        <plugins>
            <plugin>
                <groupId>com.idfconnect.devtools</groupId>
                <artifactId>idfc-proguard-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>obfuscate</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <inputFile>${project.build.outputDirectory}</inputFile>
                    <libraryJarPaths>
                        <libraryJarPath>${java.home}/lib/jce.jar</libraryJarPath>
                    </libraryJarPaths>
                    <excludeManifests>false</excludeManifests>
                    <excludeMavenDescriptor>false</excludeMavenDescriptor>
                    <outputArtifacts>
                        <outputArtifact>
                            <file>${project.build.finalName}.${project.packaging}</file>
                        </outputArtifact>
                    </outputArtifacts>
                    <proguardIncludeFile>${basedir}/proguard.pro</proguardIncludeFile>
                </configuration>
            </plugin>
        </plugins>
    </build>
</profile>
```

**proguard.pro:** this is proguard configuration file. The following is the configuration sample:

```
-dontshrink
-dontoptimize
-printmapping out.map

-keepparameternames
-renamesourcefileattribute SourceFile
-keepattributes Exceptions,InnerClasses,Signature,Deprecated,
                SourceFile,LineNumberTable,*Annotation*,EnclosingMethod

-adaptresourcefilecontents **.properties,META-INF/MANIFEST.MF
-keep class sun.misc.Unsafe { *; }

# Preserve exported classes/interfaces
-keep public class com.honeywell.guomao.bootstrap.App {
    public static void main(java.lang.String[]);
}

-keepclassmembers class **.SignalRService$* {
  public protected *;
}

-keepclassmembernames class * {
    java.lang.Class class$(java.lang.String);
    java.lang.Class class$(java.lang.String, boolean);
}

-keepclasseswithmembernames,includedescriptorclasses class * {
    native <methods>;
}

-keepclassmembers,allowoptimization enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
    java.lang.Object writeReplace();
    java.lang.Object readResolve();
}
```

___NOTE:___

```
-keepclassmembers class **.SignalRService$* {
  public protected *;
}
```

As for internal automatically-generated classes, the normal proguard configuration will also obfuscate these classes. SO they will not work as expected. However, we can use the above settings to avoid obfuscating these similar classes by proguard.
