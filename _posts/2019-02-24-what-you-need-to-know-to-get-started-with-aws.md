---
layout: post
title:  What you need to know to get started with AWS
date:   2019-02-24 19:19:19 +0200
tags:   aws, devops, infrastructure, software engineering, data science
cover_image: https://images.unsplash.com/photo-1540103711724-ebf833bde8d1?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=2255&q=80
---

[AWS](https://aws.amazon.com/) is one of the most popular cloud computing services. AWS is short for Amazon Web Services and boy, they have lots of services. 

Lots of application we use every day are hosted on AWS. They include Netflix, Airbnb, Slack to name a few. 

Maybe you're working in a company that uses AWS and you want to know more about it. Or maybe you want to deploy your application to AWS. Or you’re just heard AWS somewhere and can’t get it out of your head. Anyways, here's what you'll need to know. 

# Servers 

![](https://images.unsplash.com/photo-1484557052118-f32bd25b45b5?ixlib=rb-1.2.1&auto=format&fit=crop&w=2250&q=80)

Your application, whether it’s your own or your it belongs to the company you work for, is probably running on a server. While you develop your application you're running it on your computer. But when you deploy the application you want to run it on someone else's computer. 

AWS can offer that computer. It is called [EC2](https://aws.amazon.com/ec2/) **or Elastic Compute Cloud**. 

Also, if you’re into Data Science you can [run your Jupyter notebooks](https://docs.aws.amazon.com/dlami/latest/devguide/what-is-dlami.html) there. 

# Database 

![](https://images.unsplash.com/photo-1507842217343-583bb7270b66?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=2653&q=80)

Your application probably uses a database. PostgreSQL, MySQL, even Oracle, it doesn’t matter. What you need to know is that AWS has a service for databases and it's called [RDS](https://aws.amazon.com/rds/) **(Relational Database Service)**. 

You could run your database on another EC2 instance like you’re running on your personal computer but RDS has lots of advantages. Here are a couple: 

- Cost-efficiency 
- Automatic software patching 
- Easy scaling 
- Automated backups 
- Monitoring 

You can get familiar with the full list of features [here](https://aws.amazon.com/rds/features/). 

# Deployment 

![](https://images.unsplash.com/photo-1544849250-82efaa1cd029?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=882&q=80)

But how do you deploy your application? Well, there are many ways to do that on AWS. But if you're just getting started I would recommend using [EB](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/Welcome.html) **(Elastic Beanstalk)**. 

It allows you to easily: 

- [create environments](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.environments.html) with all configuration you need (EC2, RDS) and some that you maybe don’t even need at the start but it’s still good to have (load balancers, monitoring, notifications) 
- [deploy](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.deploy-existing-version.html) to that environment 
- [check logs and ssh to the server](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli-troubleshooting.html)

EB supports deploying different platforms starting from [Docker](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker.html) to [Java](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_Java.html) to [Ruby](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_Ruby.html). You can check out the full list of supported platforms [here](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/concepts.platforms.html). 

If you've used [Heroku](https://www.heroku.com/) in the past, EB is pretty similar to it. EB is pretty easy to set up compared to configuring each AWS service you need separately. But it is still not as easy as Heroku is. 

If you would like to start deploying after you push a commit to git you can check out AWS [CodeDeploy](https://aws.amazon.com/codedeploy/)**.** 

# HTTP routing 

![Road photo](https://images.unsplash.com/photo-1486674776040-2aeda9228e76?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=2603&q=80)

Once your application is deployed you need to route your domain traffic to it. You can use [Route53](https://aws.amazon.com/route53/) to do that. 

Although when you deploy with EB you can get a pretty decent domain like `http://your-application.elasticbeanstalk.com/` it still has that feeling that it's not yours. You can buy a domain for your application directly on Route53. You can also delegate it to Route53 if you've bought it on another service like GoDaddy or NameCheap or whatever. You can use a `dig` console command to check whether your domain is migrated to AWS. 

```bash 
dig your-domain.com
```

Once you have your domain on AWS you need to add an A alias record to either 

- the domain EB gave you (`[your-application.elasticbeanstalk.com](http://your-application.elasticbeanstalk.com/)`) 
- or your EC2’s Elastic IP 
- or your load balancer DNS name 

If the term "A record" does not sound familiar to you I recommend reading [this article about DNS](https://dev.to/swyx/networking-essentials-dns-1dl7). 

# Handling more traffic 

![Roads photo](https://images.unsplash.com/photo-1530044426743-4b7125613d93?ixlib=rb-1.2.1&auto=format&fit=crop&w=1934&q=80)

At some point, your application will probably gain some traffic and your one EC2 instance will not be able to handle all the incoming requests. [ELBs](https://aws.amazon.com/elasticloadbalancing/) **(Elastic Load Balancers)** are already on AWS to rescue you. 

Load balancers are routing incoming traffic to different servers. They are doing this using the “round-robin” algorithm. Round-robin algorithm is very simple — when a request comes in load balancer sends it to one server, a second request goes into a second server. And when all the servers received a request, load balancer starts from the beginning and sends a request to the first one again. 

# Making it secure (HTTPS) 

![Locks photo](https://images.unsplash.com/photo-1506967726964-da9127fdec36?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=2251&q=80)

If you want to add HTTPS to your website you would need to get a certificate. You can get certificates in [ACM](https://aws.amazon.com/certificate-manager/) **(AWS Certificate Manager)**. Once you get them you need to use them, they won’t automatically apply themselves. You can [add the certificate to your EC2 instance](https://aws.amazon.com/premiumsupport/knowledge-center/connect-http-https-ec2/). Or if you’ve deployed using EB you should’ve gotten a load balancer. In that case, you can apply the certificate to the load balancer and you now can serve requests via HTTPS. 

However, in this simple case, people could still visit your application over HTTP. But you can force them to use HTTPS by redirecting them if they came from HTTP. Previously, you would need to [set up NGINX or Apache](https://www.keycdn.com/blog/http-to-https#5-add-301-redirects-to-new-https-urls) redirection. But in some recent time, AWS released Application Load Balancers and they include an option to redirect using some custom rules. You can read how to set up HTTPS redirection using Application Load Balancers [here](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environments-cfg-alb.html#environments-cfg-alb-console). 

# Storing files 

![Files photo](https://images.unsplash.com/photo-1544396821-4dd40b938ad3?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=2253&q=80)

You often need to save files in your application. But storing them in the database may not be the most efficient solution. AWS has a service called [S3](https://aws.amazon.com/s3/) which stands for **Simple Storage Service**. It does what the name suggests — it simply stores data. You can write there anything you can think of. It's like a Google Drive or iCloud. 

To store files on S3 you can [use AWS SDK](https://aws.amazon.com/s3/getting-started/) for your language. You can also look for other libraries because I’m sure there are more. 

# Sending emails 

![Envelopes photo](https://images.unsplash.com/photo-1526554850534-7c78330d5f90?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=2250&q=80)

Another thing most applications do is sending emails. Emails have lost of uses: newsletters, system notifications just to name a few. 

To send emails with AWS you need to use [SES](https://aws.amazon.com/ses/) **(Simple Email Service)**. You could use your personal email or register a new one for your application and send emails from those emails. And that's a pretty good starting point. 

However, in the long run, sending emails from a personal email address is not a good choice. A better way would be to get a domain and send emails from it. To do that you need to [verify your domain in SES](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/verify-domain-procedure.html). And after that, you can send [emails using AWS SDK](https://docs.aws.amazon.com/ses/latest/DeveloperGuide/send-an-email-using-sdk-programmatically.html) which supports multiple programming languages. Or you can find a library on your own or ever create one. 

# Conclusion 

In this article, I've talked about the most popular AWS services and why and how they are used. Now you should know that you should 

- Run your servers on EC2 
- Run your database on RDS 
- Deploy to AWS using EB 
- Manage your domains in Route53 
- Get SSL certificates in ACM 
- Handle more traffic with ELB 
- Store files on S3 
- Send emails with SES 

Of course, this list is just the tip of the AWS services iceberg but it should be enough to get you started. 

If you liked this article be sure to leave your comments down below and follow me. 