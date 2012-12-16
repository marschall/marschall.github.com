---
layout: post
title: Memoryfilesystem Initial Release
---

Java 7 brought with it a pluggable file system API ([JSR-203](http://jcp.org/en/jsr/detail?id=203)). This allows anyone to write a file system and have the same code run on differnt file systems. I expected that several implmentations would pop up. For example a DAV file system, an FTP file system or an SSH file system that would allow you to easily manipulate files on remote systems. But most of all I wanted to see an in-memory file system that would allow to test code much like an in-memory embedded database.

However for whatever reason this didn't happen. So I went and stared implementing an in-memory file system. As I soon discovered the API is comprehensive for example you have to implement a [glob](http://en.wikipedia.org/wiki/Glob_(programming)) parser. So the implemtation took quite a bit longer than initially expected. Yet still no other implementation showed up during that time. Yesterday I released the first version and it's availble on Maven Central.

    <dependency>
        <groupId>com.github.marschall</groupId>
        <artifactId>memoryfilesystem</artifactId>
        <version>0.1.0</version>
        <scope>test</scope>
    </dependency>

As the version number suggests it's an early version that's likely to have many bugs. It's important to note that this really is only intended for testing purposes and not production use. You can find it on GitHub under (https://github.com/marschall/memoryfilesystem)[https://github.com/marschall/memoryfilesystem] along samples and instructions on how to use it.

