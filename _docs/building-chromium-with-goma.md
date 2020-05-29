---
title: Building Chromium with Goma
description: Building Chromium with Goma
prev-link: /docs/setting-up-goma
prev-name: Setting Up Goma
---

# Building Chromium with Goma

### Configuring Chromium

Follow build instructions from [here](https://chromium.googlesource.com/chromium/src/+/master/docs/linux/build_instructions.md). Remember that you have depot tools already installed so you can skip this part. When asked to run `gn`, run it as follows:

{% highlight shell %}
gn args out/Default
{% endhighlight %}

and add following gn args:

{% highlight shell %}
use_goma=true
goma_dir="~/goma"
is_debug=false
{% endhighlight %}

This will generate build files with Goma being used as the compiler tool.

### Setting up Goma Client

Setup environment variables so that Goma client knows the location and configuration of the Goma server.

{% highlight shell %}
export GOMA_SERVER_HOST="localhost"
export GOMA_SERVER_PORT="5050"
export GOMA_USE_SSL="false"
export GOMA_ARBITRARY_TOOLCHAIN_SUPPORT=true
export GOMA_USE_LOCAL=false
{% endhighlight %}

### Logging in into Goma

Before you can use Goma you need to login into the server. You'll have to login using the Gmail account you have whitelisted when running the server.

To login run:

{% highlight shell %}
python3 ~/goma/goma_auth.py login
{% endhighlight %}

and follow on screen directions to obtain the access code. When asked to login into a gmail account use the one you have whitelisted when running goma server. Paste the access code back to the terminal and you should see similar output:

{% highlight plaintext %}
Login as kuba.lason@gmail.com
Ready to use Goma service at http://localhost:5050
{% endhighlight %}

{% include alert.html type="info" title="Notice" content="You don't need to login every time you restart client/server. The client saves the obtained token on the host system so you can login only once and use the existing token until it expires." %}

### Starting Goma Client

Run Goma client:

{% highlight shell %}
python3 ~/goma/goma_ctl.py ensure_start
{% endhighlight %}

The output should be similar to the following one:

{% highlight plaintext %}
using /run/user/1000/goma_kubal as tmpdir

GOMA version 80401d72a6476494ab19ac9bed48a6cccbd6f8e0@1590968656
compiler proxy (pid=69032) status: http://127.0.0.1:8088 ok

Now goma is ready!
{% endhighlight %}

You can point your browser to the URL returned by goma client. This is the client status page where you can track all client actions.

### Building Chromium with Goma

To test goma run:

{% highlight shell %}
ninja -j1 -C out/Default obj/base/base/base64.o
{% endhighlight %}

If everything went well your goma status page should list the job in the `Finished Tasks` section with a `RETRY` result (this is good - don't worry about that). To see the cache in action run:

{% highlight shell %}
ninja -j1 -C out/Default -t clean
ninja -j1 -C out/Default obj/base/base/base64.o
{% endhighlight %}

The finished section of the status page should list a new job - this time with `CACHEHIT` status. 