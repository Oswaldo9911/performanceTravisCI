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
## Running Headless Tests

Especially in a continuous integration server, you'll want to run JMeter tests "headlessly", i.e. without a graphical user interface.

```
rm -rf results && mkdir results
apache-jmeter-3.0/bin/jmeter.sh -j results/jmeter.log -p tests/jmeter.properties -t tests/my_test.jmx -n -l results/my_test.jtl
```

Let's break this down:

- `-j results/jmeter.log` indicates where JMeter should log its operations
- `-p tests/jmeter.properties` is the properties file to use when running the test (more on properties later)
- `-t tests/my_test.jmx` is the test to run
- `-n -l results/my_test.jtl` tells JMeter to run in non-gui mode and provides the path where JMeter will write the test results.

## Running tests with Parameters

JMeter provides the ability to parameterize tests, such that default values can be specified in the test, overridden in a properties file, and further overridden on the command line

Open `my_test_with_parameters.jmx`, like so:

```
$ apache-jmeter-3.0/bin/jmeter.sh -t tests/my_test_with_parameters.jmx
```

In the GUI, click on the `HTTP Request Defaults` node and look at the `Server Name or IP` Field. You'll see this: `${__P(server, localhost)}`

The `${__P()}` is a JMeter parameter. The value in quotes is the default, should no value be supplied in a properties file or command line.

If you update the `jmeter.properties` file with a value: `server=my.server`, when you reload the test that value will be used for the server to be tested.

To override that at the command line, use JMeter's `-J` flag, which takes the form `-J[param_name]=value`:

```
$ apache-jmeter-3.0/bin/jmeter.sh -t tests/my_test_with_parameters.jmx -Jserver=my.test.server
```

In the JMeter GUI, click on the `Thread Group` node. Notice a parameter is used for both Threads and Loop Count. To override those at the command line:

```
$ apache-jmeter-3.0/bin/jmeter.sh -t tests/my_test_with_parameters.jmx -Jserver=my.test.server -Jthreads=100 -Jloopcount=1000
```

This will run the test against the `my.test.server` URL, with 100 concurrent users, repeating 1000 times.

## Using jmeter-bootstrap as a Git Submodule

To use jmeter-bootstrap as a [Git submodule](http://git-scm.com/book/en/Git-Tools-Submodules) in your own test project, add the repo as a submodule.

Assuming you have a project with this structure:

```
/my-project
  /tests
  /results
```

Issue:

```git submodule add https://github.com/cfpb/jmeter-bootstrap.git```

This will pull the jmeter project as a git submodule into your own project. To run the JMeter installer:

```
$ python jmeter-bootstrap/bin/JMeterInstaller.py
```

Which will pull JMeter into your project, resulting in a directory structure like:

```
/my-project
  /apache-jmeter-3.0
  /tests
  /results
```


## Jenkins Jobs

Using a combination of plugins, you can create a Jenkins job to:

1. Run your JMeter tests
1. Fail on a certain error threshold
1. Generate a report and graphs based on the test results
1. Display those artifacts on the job results screen

### Plugins

1. [Performance Plugin](http://wiki.jenkins-ci.org/display/JENKINS/Performance+Plugin)
1. [Image Gallery Plugin](http://jenkins-ci.org/plugin/image-gallery/)

### Job Configuration

A Common pattern is to use this jmeter-bootstrap project as a git submodule inside of another repo, such as `My-Load-Tests`. If you do that, you shouldn't have any additional work to do as Jenkins will also fetch submodules by default.

In the configuration below, I'll use simple paths to the jmeter-bootstrap location... in your job configuration, simply change the paths to point to the correct location.

**Build Steps**

You can use a single `Execute Shell` step to do a one-time install of JMeter and the JMeter plugins, run your load tests, and emit graphs. It will look something like this:

```
python jmeter-bootstrap/bin/JMeterInstaller.py
rm -rf results
mkdir results

jmeter-bootstrap/apache-jmeter-3.0/bin/jmeter.sh -j results/jmeter.log -p tests/jmeter.properties -t tests/some_test.jmx -n -l results/some_test.jtl

./jmeter-bootstrap/bin/generate_files_from_jtl.sh some_test
```

**Post-build Actions**

1. Archive the artifacts.

I typically use something like `results/*.png, results/*.html, results/*.csv` as the pattern

2. Create Image Gallery

I use "Archived Image Gallery" and include `**/results/*.png` as the pattern

3. Publish performance test result report

I add a `JMeter Report`, then for Report Files, I use `results/*.jtl`


After you build, you'll see the graphs show up on the Build screen. Clicking on `Performance Report` will take you to a historical report showing you performance over time.

## Roadmap

- Windows compatibility
- Python 2.6 and 3.3 compatibility