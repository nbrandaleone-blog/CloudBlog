---
title: "Running Hugo on Firebase"
date: 2021-03-24T14:05:15-04:00
socialshare: true
draft: false
---
## Firebase
In my last blog on this [topic](https://nicks-playground.net/posts/running-hugo-in-the-cloud/) I discussed how I switched from _jekyll_ to _Hugo_. I also talked about hosting from a static S3 bucket, which is how I did things when I was at AWS.  I tried to duplicate the same thing on Google Cloud, and it did not work out nearly as well.

### "Why not", you ask?
Well, it did technically work, but it was not optimal. Replicating the same architecture on a different cloud provider is not always a good decision. Each cloud provider has its own products, each with unique strengths and weaknesses.

The cost of running a static blog directly on GCP turned out to be too costly for me.  It was not breaking the bank at about $20/month, but that is more than I wanted to spend on a low-volume blog.

## Alternatives:
1) [blogger](blogger.com). This is an older product purchased by Google easily over 20 years ago.  However, it is **free**, and very easy to use.
2) [Firebase hosting](https://firebase.google.com/docs/hosting)

I decided to use Firebase Hosting. While not free, it is extremely inexpensive. [Firebase](https://firebase.google.com/) is extensible and powerful - aimed at teams building web/mobile apps. If I ever want to run my own database or set up something like data sync for a mobile app, it will be easy to do with Firebase.

## Use Cloud Native to your advantage
I am so happy I simply did not replicate my old architecture on GCP. After some research, I stumbled upon an excellent product at a reasonable price. Google's focus is much more Developer centric than AWS. By taking advantage of their strength in this area, I was able to leverage their platform that much more effectively.  It is a good lesson, regardless of which cloud platform you are on.
