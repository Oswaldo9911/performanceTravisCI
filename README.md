# JMeter - TravisCI

## Dependencies

 - Python 2.7 to install JMeter 5.1
 - Java 1.7+ to run JMeter

## Install JMeter and JMeter plugins

```
$ python bin/JMeterInstaller.py
```

The installer will also install several JMeter Plugins, which can be used directly or within a continuous integration server such as Jenkins

## Open Tests in JMeter

```
apache-jmeter-5.1/bin/jmeter.sh -t tests/test.jmx
```