# CloudBlog

This is a static blog, generated by Hugo.
I use it to document my learnings and experience
with various Cloud Providers, specifically AWS and GCP.

I also include personal posts from time to time.

Firebase for hosting
Docs: https://firebase.google.com/docs/cli?authuser=0#macos
GCP for DNS
DNS hostname: https://nicks-playground.net/

Example blog: https://pretired.dazwilkin.com/posts/200930/

---

Added Social media links using the following guide:
https://codingnconcepts.com/hugo/social-icons-hugo/

### Create post. Source files are located here: content/posts
hugo new posts/my-first-post.md

### Build site (with drafts)
hugo -D

### Live hosting
hugo server

### Sync with GitHub
git add .
git push origin main

# Now using Firebase
firebase deploy --only hosting

### Sync blog to Google Storage bucket (old method)
gsutil rsync -R ~/gcp/CloudBlog/public/ gs://nicks-playground.net/
