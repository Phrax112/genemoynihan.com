---
title: AWS Amplify
author: Gene
layout: post
---

I've just migrated this site from being self-hosted to running on AWS using the Amplify service. Apaert from when studying for the AWS certifications I haven't used AWS for many personal or professional projects (GCP being the normal goto). However, I decided I'd try both AWS and GCP for the purposes of hosting this static website, originally with the concept of using Bucket storage on GCP or s3 on AWS. Within about 5 minutes or so however, it became quite clear to me that AWS was the
superior solution. Setting up my third party domain name (I use Hover as a management service) with Route 53 was almost completely handled by the Amplify service which then also issued me certifcates in the same pipeline. In contrast to GCP where I seemingly couldn't use my personal account for registering a domain name (the G Suite process is long and dark, at least for the 10 minutes I spent googling it).

With the domain name setup I allowed OAuth acces to my Github account for the Amplify service, select my Jekyll project and master branch and everything worked perfectly. With the auto-check enabled any new commit to master triggers a new website deployment making updating new posts no more difficult than writing the post and pushing to master.

I think the lesson I need to learn from this is to explore and docuement any new cloud project across providers

