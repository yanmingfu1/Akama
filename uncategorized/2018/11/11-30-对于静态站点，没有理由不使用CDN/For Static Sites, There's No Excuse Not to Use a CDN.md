## For Static Sites, There’s No Excuse Not to Use a CDN

Are you getting the most out of your static site? If you’re not hosting your site on a CDN, you definitely aren’t.

CDNs, or Content Delivery Networks, are commonly used to distribute static resources like images, videos, and client-side code (CSS and JavaScript.) Since a static site is composed entirely of static resources (including the HTML pages,) it is possible to serve the entire website through a CDN!

In this article, we will explore why you should be using a CDN to host your static site, and how you can do it with Netlify. Netlify makes it so easy to deploy and host your static site with their CDN, you have no excuse not to do it!

### The Ping Problem

One metric that is used when evaluating the performance of a web page is the time to first byte, or TTFB. This measures how long it takes for the client to begin receiving data from the server.

There are two main problems that increase TTFB:

1. We have to wait for data to travel across the physical wires that make up the internet.
2. We have to wait for the server to process the request after receiving it, and prepare the response before sending it.

For a dynamic site like WordPress, problem #2 is a frequent bottleneck. The server has a lot of WordPress code to run in order to prepare the response to be sent to the client. However, with static sites, this is nearly instantaneous, since the pages are all generated when the site is deployed. Problem #2 is not really a problem for us!

So, is there anything we can do about problem #1?

The time it takes for data to travel across a network is referred to as latency. Latency is primarily a physical limitation: a signal needs to travel through a cable; the longer the cable, the longer it will take the signal to move through it. Theoretically, we could transmit this signal at the speed of light, but in reality data moves much slower than that.

#### Testing Latency

You might have heard network latency referred to as ping. This name comes from the ```ping``` utility which can be used to measure it. You most likely have this utility on your computer.

You can use ```ping``` yourself to understand how latency works. Open up a terminal and type the following command:

```
ping ec2.us-east-1.amazonaws.com
```

This command will send a request to a server in Amazon’s us-east-1 region, and wait for a response. The us-east-1 region is located in Northern Virginia in the United States. When I run this command from my office in Maryland, I get the following response:

```
PING ec2.us-east-1.amazonaws.com (54.239.28.176) 56(84) bytes of data. 
64 bytes from 54.239.28.176: icmp_seq=1 ttl=232 time=50.3 ms
```

The ```ping``` utility reported that it took 50.3 milliseconds for data to travel from my computer to Amazon’s server and back.

By changing the hostname in my command from ```ec2.us-east-1.amazonaws.com``` to ```ec2.ap-southeast-2.amazonaws.com```, I can ping their Sydney, Australia region instead:

```
PING ec2.ap-southeast-2.amazonaws.com (54.240.195.243) 56(84) bytes of data. 
64 bytes from 54.240.195.243: icmp_seq=1 ttl=233 time=301 ms
```

Communicating with the more distant server caused a 500% increase in round-trip latency. 250 milliseconds may sound like a short amount of time, but this time penalty applies to every request the user makes to that server: CSS, JavaScript, images, etc. Even small amounts of latency can negatively impact your site. If there were an easy way to eliminate this latency, wouldn’t you want to do it?

> If you want to experiment with pinging other regions, take a look at this chart of EC2 regions. You can ping the IP address directly, or use a hostname following the convention ec2.$REGION.amazonaws.com.
>

### Improving Latency With a CDN

There are plenty of technologies that rely on real-time peer-to-peer communication. A static site is not one of them! One way we can solve this latency problem is by replicating our website to multiple servers distributed across the globe. Imagine that I could deploy two copies of my site to AWS: one copy to the us-east-1 region, and another to the ap-southeast-2 region, and somehow direct my users to whichever server was closer to them.

This is exactly what a CDN can do for us! CDN stands for Content Delivery Network, and its purpose is to replicate resources to a network of edge nodes distributed all over the world. When a user requests a resource hosted by the CDN, they are routed to the edge node closest to them.

To demonstrate this, I enlisted the help of my friend Chris (IG: @chrismaustphotos) who lives on the other side of the USA in San Diego, California. For this experiment, I had us both run the ping command against two hosts. The first host was the same ec2.us-east-1.amazonaws.com from the previous example, and the second was forestry.io. Forestry is hosted on AWS’ CloudFront CDN, so I was expecting it to take him longer to reach the us-east-1 server, but for us both to reach forestry.io quickly. The results confirm this hypothesis

#### Ping Response Times

Now, there several factors that affect ping, but here’s the takeaway: we both received a response from forestry.io faster than we received a response from the server in Northern Virginia, despite being on opposite ends of the country.

We can better understand what’s going on here by checking the PTR record for forestry.io’s IP address. To do this, run the following command:

```
dig -x $IPADDRESS
```

Where $IPADDRESS is the resolved IP that was reported by the ping command. Your output will look something like this:

```
dig -x 216.137.41.157 

; <<>> DiG 9.10.3-P4-Ubuntu <<>> -x 216.137.41.157 
;; global options: +cmd 
;; Got answer: 
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 25470 
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1 

;; OPT PSEUDOSECTION: 
; EDNS: version: 0, flags:; udp: 1280 
;; QUESTION SECTION: 
;157.41.137.216.in-addr.arpa.   IN      PTR 

;; ANSWER SECTION: 
157.41.137.216.in-addr.arpa. 68401 IN   PTR     server-216-137-41-157.ewr2.r.cloudfront.net. 

;; Query time: 38 msec 
;; SERVER: 127.0.1.1#53(127.0.1.1) 
;; WHEN: Thu Jun 07 12:50:18 EDT 2018 
;; MSG SIZE  rcvd: 113
```
Make note of the PTR record in the answer section. This IP points to server-216-137-41-157.ewr2.r.cloudfront.net. This is one of CloudFront’s edge nodes, and the ewr2 is a hint that this node is located in Newark, New Jersey (AWS is using a naming convention based on airport codes, and EWR is the code for Newark Liberty International Airport.)

When Chris ran his ping command, forestry.io resolved to a different IP address. Running the same command on this IP address, we see server-52-85-83-253.lax1.r.cloudfront.net. Once again we can follow AWS’ naming convention to deduce that the lax subdomain means that this node is located in Los Angeles.

TL;DR
Using a CDN allows us to host copies of our website all over the world. When a user requests our site, the CDN directs them to the nearest copy to minimize latency

### Hosting Your Static Site on a CDN

If the previous section has you thinking that putting your static site on a CDN is complicated, worry not. Netlify makes it easy! Their service takes care of deploying and hosting your static site, and setup takes just a few steps. The video below shows you how easy it is to get started with Netlify:

Checkout Netlify’s documentation for more information on deploying and hosting with Netlify, or head to their signup page to get your site deployed in minutes.

> Netlify not only makes set up easy: their deployment process automatically takes care of some really tricky stuff like atomic deploys and instant cache invalidation. Take a look at their features page to learn more.
>


### The Future is Static

Thanks to static sites, it has never been easier to make your website fast and highly available to a global audience. Choosing to #gostatic gives you great performance out of the gate, but using a CDN to deliver your site will ensure your site stays fast for everyone. Thanks to the simplicity of Netlify, there’s no excuse not to give it a shot!

