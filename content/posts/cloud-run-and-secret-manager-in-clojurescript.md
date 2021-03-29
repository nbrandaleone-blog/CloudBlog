---
title: "Cloud Run and Secret Manager in Clojurescript"
date: 2021-03-29T09:37:16-04:00
draft: false
---
# How to use Google's Secret Manager with Cloud Run

![GCP Secret Manager](/images/gcp-secret-manager.png)

[Cloud Run](https://cloud.google.com/run) is an amazing product - a totally severless product, which
launches and manages containers on your behalf. It handles scaling
and concurrency, security and so on.  It is probably most similar to
Amazon's [Fargate](https://aws.amazon.com/fargate/) product, but also overlaps with [Lambda](https://aws.amazon.com/lambda) as well.

I was playing around with GCP [Secret Manager](https://cloud.google.com/secret-manager), and I thought I would
share how to leverage it while using Cloud Run. I also posted some good reference blog posts at the end.

The sample code uses [Clojurescript](https://clojurescript.org/).  I am a big fan of functional 
programming, and Clojure in general.  If you are not familiar with Clojure, it is a LISP derivative, that runs on top of the JVM.  Clojurescript transpiles into Javascript, and therefore can take advantage of the enormous node packages availble to the community. Check out [Shadow-CLJS](https://shadow-cljs.github.io/docs/UsersGuide.html), which makes for importing javascript libraries into Clojurescript a trivial process.

## Hello, World
We will create a simple [ExpressJS](http://expressjs.com/) web server. It will read in a secret
from secret manager. If that fails, it will look for an environmental
variable called `TARGET`. If that fails, it will simply respond with
`Hello, World!`.

Cloud Run allows for environmental variables to be passed into your
running container.  There is no direct integration with secret manager,
so we must call it via API. Interestingly enough, [Cloud Build](https://cloud.google.com/build/docs/securing-builds/use-secrets), _does_ allow for
secrets to be injected as environmental variables.  That simplify things,
although it can be argued that environmental variables are dangerous tools when it comes to sensitive data; since they can be exposed via logs or memory dumps quite easily. Still, it you want something that requires very little coding, Cloud Build's integration with Secret Manager is the way to go.

## Deploy container
I built my container locally, and pushed it up into [Google Container Registry](https://cloud.google.com/container-registry). This was simple and effective, but I would look at [Cloud Build](https://cloud.google.com/build) as a CI/CD system for anything more advanced over a simple demo.

```bash
$ gcloud run deploy hello-secret --image gcr.io/nicks-playground-3141/hello-secret:0.4 --platform managed
Deploying container to Cloud Run service [hello-secret] in project [nicks-playground-3141] region [us-central1]
✓ Deploying... Done.
  ✓ Creating Revision...
  ✓ Routing traffic...
Done.
Service [hello-secret] revision [hello-secret-00010-qit] has been deployed and is serving 100 percent of traffic.
Service URL: https://hello-secret-hrhyswd3ya-uc.a.run.app

# Test against URL
$ curl https://hello-secret-hrhyswd3ya-uc.a.run.app/
Hello, World!
```

## Deploy with an environmental variable injected into the container
```bash
$ gcloud run deploy hello-secret --set-env-vars TARGET=GALAXY \
--image gcr.io/nicks-playground-3141/hello-secret:0.4 --platform managed

# Test aginst URL
$ curl https://hello-secret-hrhyswd3ya-uc.a.run.app/
Hello, GALAXY!
```

## Create secret
Let's create a secret in secret-manager:
```bash
$ gcloud secrets create TARGET \
    --replication-policy="automatic"

$ echo -n "Universe" | \
    gcloud secrets versions add TARGET  --data-file=-

# Verify the secret
$ gcloud secrets versions access 1 --secret="TARGET"
Universe
```

## Update IAM permissions to read from Secret Manager
I did not do this the first time I tried it out, and it of course
failed.  You can create a new Service Account for your Cloud Run container, with appropriate permissions.
Or, you can expand the capabilities of the [default](https://cloud.google.com/run/docs/securing/service-identity) service account used by Cloud Run - which it what I did here.
This is **not** considered best practices, since you do not want all your
containers being allowed to read all the secrets.  However, for this project, this was an acceptable choice.

```bash
$ gcloud iam service-accounts add-iam-policy-binding nicks-playground-3141 \
--member="serviceAccount:659824402950-compute@developer.gserviceaccount.com" \
--role="roles/secretmanager.secretAccessor" 
```

## Restart service, and test
``` bash
$ curl https://hello-secret-hrhyswd3ya-uc.a.run.app
Hello, Universe!
```

## Debugging
The GCP Logging Console is amazing, with powerful eye-candy.
However, if you want to simply tail the logs since your program
emits logs to STDOUT, you can execute the following command:

```bash
gcloud beta logging tail "resource.type=cloud_run_revision AND \
resource.labels.service_name=hello-secret AND \
severity>=DEFAULT" \
--format="default(timestamp,resource["labels"]["service_name"],textPayload)"
```

## Clojurescript
For those of you who want to look what Clojurescript code looks like,
the repository is located on Github [here](https://github.com/nbrandaleone/hello-secret).

## Snippet of Clojurescript calling GCP Secret Manager
``` Clojurescript
(ns server.main
  (:require
    ["@google-cloud/secret-manager" :as Secret]
    ["express" :as express]
    [taoensso.timbre :as log]))

(defn get-word []
  "determine the TARGET value, to follow 'hello' if secret-manager fails"
  (let [t (get (env) "TARGET" "World")] t))

(defn secret-api []
  "Calls GCP Secret Manager, using a JS promise
   Use an atom to store state. Probably shouldn't, but it make things
   a bit easier to test."
  (let [client (Secret/SecretManagerServiceClient.)]
    (-> (.accessSecretVersion client (clj->js {:name mysecret}))
      (.then (fn [name]
               (let [payload (first name)] 
                 (reset! word (str (.. payload -payload -data)))))
             (fn [] (do
                      (reset! word (get-word))
                      (log/debug "Secret Manager Promise rejected")))))))
```

---

# References:
- [Google Cloud Secret manager: a better place to store your secrets](https://www.inthepocket.com/blog/google-cloud-secret-manager-a-better-place-for-your-secrets)
- [Severless Mysteries with Secret Manager Libraries on Google Cloud](https://dev.to/googlecloud/serverless-mysteries-with-secret-manager-libraries-on-google-cloud-3a1p)
- [Secret Manager: Improve Cloud Run security without changing the code](https://medium.com/google-cloud/secret-manager-improve-cloud-run-security-without-changing-the-code-634f60c541e6)
