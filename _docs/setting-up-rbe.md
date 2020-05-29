---
title: Setting up RBE
description: Setting up RBE service
next-link: /docs/setting-up-goma
next-name: Setting Up Goma
prev-link: /docs/prerequisites
prev-name: Prerequisites
---

# Setting Up RBE

This section will describe steps needed to run a Remote Build Execution backend and make Goma server use it.

### Choosing the RBE Backend

There are few free RBE backends available. You can find some of them being listed in [Bazel Docs](https://docs.bazel.build/versions/master/remote-execution.html). All of them should work with Goma however almost in all cases some fixes/patches will be needed in order to align them with Goma.

For purposes of this guide we'll use [BuildGrid](https://buildgrid.build). Personally I think it is the easiest RBE backend to setup, it is written in Python (for now this is an advantage) and more or less implements full RBE spec so it can serve as a good reference implementation.

### Setting up BuildGrid

Run:

{% highlight shell %}
git clone https://gitlab.com/BuildGrid/buildgrid.git
cd buildgrid
git checkout 0.0.2
python3 -m venv env
env/bin/python -m pip install --upgrade setuptools pip wheel
env/bin/python -m pip install --editable .
{% endhighlight %}

You can check if your server works by running following command:

{% highlight shell %}
env/bin/bgd server start data/config/default.conf -vvv
{% endhighlight %}

It should start the BuildGrid server in a default setup. If everything is ok we can start preparing BuildGrid to serve requests from Goma server.

### Configuring the BuildGrid for Goma

BuildGrid comes with a convenient default configuration. It'll only need a few tweaks to make it suitable for the purposes of this guide.

BuildGrid configurations are present in the `data/config` directory. Go there and make a copy of `default.conf`. You can name it as you like, for example `goma.conf`. Now, open the `goma.conf` and patch it as follows:

{% highlight diff %}
@@ -8,7 +8,6 @@ description: >
     - Unauthenticated plain HTTP at :50051
     - Single instance: [unnamed]
     - In-memory data, max. 2Gio
-    - DataStore: sqlite:///./example.db
     - Hosted services:
        - ActionCache
        - Execute
@@ -22,33 +21,24 @@ monitoring:
   enabled: false

 instances:
-  - name: ''
+  - name: goma-test
     description: |
-      The unique '' instance.
+      The unique goma-test instance.

     storages:
       - !lru-storage &cas-storage
         size: 2048M


-    schedulers:
-      - !sql-scheduler &state-database
-        storage: *cas-storage
-        connection_string: sqlite:///./example.db
-        automigrate: yes
-        connection_timeout: 15
-        poll_interval: 0.5
-
     services:
       - !action-cache &build-cache
         storage: *cas-storage
         max-cached-refs: 256
-        cache-failed-actions: true
+        cache-failed-actions: false
         allow-updates: true

       - !execution
         storage: *cas-storage
         action-cache: *build-cache
-        scheduler: *state-database
         max-execution-timeout: 7200

       - !cas

{% endhighlight %}

We won't go deep into the BuildGrid configuration but the above patch:

- sets name for RBE instance - we'll use that same name in goma server
- removes persistent scheduler - BuildGrid can save scheduled tasks' state into a persistent storage making the task states survive BuildGrid restart. This is not needed for our setup so we've removed it making BuildGrid use default non-persistent in-memory scheduler.
- disables caching of failed actions

You can now run BuildGrid server with:

{% highlight shell %}
env/bin/bgd server start data/config/goma.conf -vvv
{% endhighlight %}

and move on to the worker setup.

### Configuring BuildGrid Worker

BuildGrid provides several types of workers (also called bots) but we'll use a host tools worker that simply runs the received command in a temporary directory created on a system actually hosting the worker. While this is not the safest method it is the most suitable for us now.

Run the worker with following command:
{% highlight shell %}
env/bin/bgd bot --parent=goma-test --remote=http://localhost:50051 -w OSFamily Linux -vvv host-tools
{% endhighlight %}

and worker should output something like this:

{% highlight plaintext %}
[DEBUG][mainthread]: Creating bot session
[INFO][mainthread]: Created bot session with name: [goma-test/4d8ba0b0-7e99-40a2-98cd-39b4d359dea0][debug][MainThread]: Updating bot session: [goma-test.kubal-Virtual-Machine.26402][debug][MainThread]: Updating bot session: [goma-test.kubal-Virtual-Machine.26402]
{% endhighlight %}

You just started a worker (bot) asking it to connect to the server pointed by the `--remote` parameter and use the RBE instance as set by the `--parent` parameter. The `-w` parameter sets the worker properties. Worker properties are simple key/value pairs that describe worker capabilities. Goma server will file a request with certain properties set and BuildGrid will schedule such a job to one of its workers only if it finds one with the same exact set of properties set. Therefore, we must configure the worker to meet the Goma server expectations.
