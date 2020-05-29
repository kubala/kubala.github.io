---
title: Setting up Goma
description: Setting up goma server
next-link: /docs/building-chromium-with-goma
next-name: Building Chromium with Goma
prev-link: /docs/setting-up-rbe
prev-name: Setting Up RBE
---

# Setting Up Goma

This section will describe steps needed to run a dummy Goma server. At the end ot this section our Goma server will be ready to schedule remote jobs using the BuildGrid backend.

## Setting up Server

### Creating Service Account

Goma server needs a Google Cloud service account in order to authenticate the clients. Therefore, you'll need to create a service account in GCP and download its key.

- Follow [this guide](https://cloud.google.com/iam/docs/creating-managing-service-accounts) to create a service account. Fill the requested fields with anything you like. This does not matter for now.

- Once the service account is ready, create and download the credentials in JSON format. Save the file somewhere - make sure it will be readable by the user running Goma server. [This guide](https://cloud.google.com/iam/docs/creating-managing-service-account-keys) has all that you need to complete this step.

### Getting the Code

Go and checkout the goma server with:

{% highlight shell %}
git clone https://chromium.googlesource.com/infra/goma/server
cd server
git checkout v0.0.13
{% endhighlight %}

### Patching Goma

As mentioned previously BuildGrid worker properties must match the one requested by the Goma server or BuildGrid scheduler won't be able to schedule an actual job. Goma server requests the `container-image` property to be set on the worker but BuildGrid does not allow such property to be set. Therefore either BuildGrid or Goma server needs to be patched. We'll do the latter as in this particular case patching Goma will be easier.

In Goma server repo, edit `cmd/remoteexec_proxy/main.go` and apply following patch:

{% highlight diff %}
@@ -408,9 +408,6 @@ func main() {
 					RbeInstanceBasename: path.Base(*remoteInstanceName),
 					Properties: []*cmdpb.RemoteexecPlatform_Property{
 						{
-							Name:  "container-image",
-							Value: *platformContainerImage,
-						}, {
 							Name:  "OSFamily",
 							Value: "Linux",
 						},
{% endhighlight %}

This simply removes the unsupported property so the build requests won't be flagged with it making BuildGrid schedule the remote build.

Once patched, we'll run the Goma server:

{% highlight shell %}
go run cmd/remoteexec_proxy/main.go -port 5050 -remoteexec-addr localhost:50051 -remote-instance-name goma-test -insecure-remoteexec -service-account-json <path_to_service_account_json> -whitelisted-users <your_gmail_email_address>
{% endhighlight %}

The above command will:

- make Goma server listen for incoming client connections on port `5050`
- connect to BuildGrid at `localhost:50051`
- use `goma-test` instance
- use insecure connection to RBE backend

If everything went well you should get similar output:

{% highlight plaintext %}
INFO	remoteexec_proxy/main.go:277	allow access for ["kuba.lason@gmail.com"] / domains []
INFO	remoteexec_proxy/main.go:290	using service account: /home/kubal/src/key.json
INFO	acl/checker.go:73	service account key: update
INFO	acl/checker.go:82	acl updated
WARN	remoteexec_proxy/main.go:360	use insecrure remoteexec API
WARN	remoteexec_proxy/main.go:372	redis disabled for gomafile-digest: no REDISHOST environment
INFO	exec/inventory.go:190	configure platform config: target:{addr:"localhost:50051"}  build_info:{}  remoteexec_platform:{properties:{name:"OSFamily"  value:"Linux"}  rbe_instance_basename:"goma-test"}  dimensions:"os:linux"
{% endhighlight %}

## Getting the Goma Client

We won't be building the Goma client from sources (although we could) - we'll use the precompiled binaries that are available through CIPD (Chrome Infrastructure Package Deployment). For that purpose we'll install chromium's `depot_tools` (we'll need it later anyway). Follow the [depot_tools install guide](https://chromium.googlesource.com/chromium/src/+/master/docs/linux/build_instructions.md#install) to setup the required tools (do not checkout the chromium code yet - we just need `depot_tools` at this stage). Once ready, run:

{% highlight shell %}
cipd install infra/goma/client/linux-amd64 -root ~/goma
{% endhighlight %}

After that Goma client should be available in `~/goma`. Feel free to use a different install path - just remember to adjust all the commands copied from here so that the correct install path is used.

