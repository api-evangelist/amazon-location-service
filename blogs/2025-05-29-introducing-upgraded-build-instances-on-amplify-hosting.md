---
title: "Introducing Upgraded Build Instances on Amplify Hosting"
url: "https://aws.amazon.com/blogs/mobile/introducing-upgraded-build-instances-on-amplify-hosting/"
date: "Thu, 29 May 2025 18:14:26 +0000"
author: "Matt Auerbach"
feed_url: "https://aws.amazon.com/blogs/mobile/feed/"
---
<p><a href="https://aws.amazon.com/amplify/hosting/">AWS Amplify Hosting</a> has built web applications using a fixed instance environment. As applications grow more complex and require intensive build processes for dependency management, asset optimization, and comprehensive testing, developers need more powerful build environments to maintain productivity and deployment speed.</p> 
<p>Today, we’re excited to introduce customizable build instances for Amplify Hosting. This update adds two new instance sizes: Large and XLarge, both offering increased memory and CPU resources. Development teams can now scale their build resources to match their specific needs. This is especially useful when handling large dependency trees, processing numerous static assets, or running memory-intensive operations like TypeScript compilation and parallel testing frameworks.</p> 
<h3>Challenges with Fixed Size Build Instance</h3> 
<p>Customers building large applications and running heavier workloads experience <a href="https://github.com/aws-amplify/amplify-hosting/issues/654">increased build times, slow CI/CD and build failures due to memory errors</a>. As development teams adopt Amplify Hosting, they expect scalable build environments that can match a variety of workloads. While build time optimizations exist, applications cannot grow within a fixed instance size container configuration of 8GiB memory and 4 vCPUs. Virtual memory (swap space) introduces significant performance penalties compared to physical memory, and no effective alternatives exist for expanding compute capacity or storage within a fixed environment. These hardware limitations create fundamental barriers that software optimizations alone cannot overcome.</p> 
<p><strong>Build Instance Specifications</strong></p> 
<table> 
 <thead> 
  <tr> 
   <th><strong>Build Instance</strong></th> 
   <th><strong>&nbsp; &nbsp; &nbsp;vCPUs</strong></th> 
   <th><strong>&nbsp; &nbsp; Memory</strong></th> 
   <th><strong>&nbsp; &nbsp;Disk space</strong></th> 
  </tr> 
 </thead> 
 <tbody> 
  <tr> 
   <td>Standard</td> 
   <td>&nbsp; &nbsp; &nbsp;4</td> 
   <td>&nbsp; 8 GiB</td> 
   <td>&nbsp; &nbsp; 128 GB</td> 
  </tr> 
  <tr> 
   <td>Large</td> 
   <td>&nbsp; &nbsp; 8</td> 
   <td>&nbsp; 16 GiB</td> 
   <td>&nbsp; &nbsp; 128 GB</td> 
  </tr> 
  <tr> 
   <td>XLarge</td> 
   <td>&nbsp; &nbsp; 36</td> 
   <td>&nbsp; 72 GiB</td> 
   <td>&nbsp; &nbsp; 256 GB</td> 
  </tr> 
 </tbody> 
</table> 
<h2></h2> 
<h2>Introducing Customizable Build Instances</h2> 
<p>Amplify Hosting’s new customizable build instances allow developers to select the computing resources that best match their application needs. This allows achieving predictable build performance by eliminating resource constrains. In addition to our Standard build instance, we are introducing Large and XLarge instances. The following table summarizes the specifications of all of Amplify Hosting’s build instance offerings:</p> 
<p>You can configure build instances at the application level either during the app creation or later udpating your existing applications. Updated configuration will extend to all of your app’s branches and change an app’s build instance size across builds. Any build running for the app, irrespective of the branch, will use the build instance size that you configured at the app level when you triggered the build. You can configure build instance size when creating a new app or updating an existing app.</p> 
<ul> 
 <li><span style="text-decoration: underline;">To customize build instance size for a new app via Amplify console:</span> During the <strong>App Settings</strong> step, scroll down and click on <strong>Advanced settings</strong>. From the drop down list, select the Build Instance type you would like to configure for your app.</li> 
</ul> 
<div class="wp-caption aligncenter" id="attachment_14137" style="width: 1012px;">
 <img alt="Screenshot of amplify hosting console with a dropdown to select build instances" class="wp-image-14137 " height="329" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/29/Screenshot-2025-05-28-at-12.07.22 AM.png" width="1002" />
 <p class="wp-caption-text" id="caption-attachment-14137">Figure 1 – Configure build instance size for a new app</p>
</div> 
<ul> 
 <li><span style="text-decoration: underline;">To customize build instance size for an existing app via Amplify console:</span> Navigate to your app, open the <strong>Hosting tab</strong>, and select <strong>Build Settings</strong>. Scroll down to <strong>Advanced settings</strong> and click <strong>Edit</strong>. From the drop down, select the <strong>Build Instance type</strong> you would like to update your app with.</li> 
</ul> 
<div class="wp-caption aligncenter" id="attachment_14139" style="width: 806px;">
 <img alt="Screenshot of build instances setting in an existing app settings" class="wp-image-14139" height="299" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/29/Screenshot-2025-05-28-at-12.10.14 AM.png" title="test" width="796" />
 <p class="wp-caption-text" id="caption-attachment-14139">Figure 2 – Modify build instance for an existing app</p>
</div> 
<p><span style="text-decoration: underline;">Note</span>: Build instance allocation process can require additional provisioning time before your build starts. For larger instances, especially XLarge, your build might experience latency before the build starts, due to this overhead time. However, you are billed only for the actual build time, not the overhead time.</p> 
<h2>Performance Testing Explanation</h2> 
<p>Now we will demonstrate the value of more powerful instances by using a Next.js application that generates more than 100 static pages. We will walk you through the deployment of this app on Amplify Hosting, show you how to debug memory-related build failures, and teach you how to upgrade the build instance to meet your application’s memory needs.</p> 
<p>Deploying an application to Amplify HostingWe will first deploy the static app using the default settings:</p> 
<div class="wp-caption aligncenter" id="attachment_14141" style="width: 881px;">
 <img alt="Screenshot of the review page in Amplify Hosting" class=" wp-image-14141" height="421" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/29/Screenshot-2025-05-28-at-12.26.23 AM.png" width="871" />
 <p class="wp-caption-text" id="caption-attachment-14141">Figure 3 – Review and create an app page</p>
</div> 
<p>Once the deployment starts, Amplify Hosting provision the resources and carry the deployment further:</p> 
<div class="wp-caption aligncenter" id="attachment_14142" style="width: 866px;">
 <img alt="Screenshot of build deploy screen" class=" wp-image-14142" height="237" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/29/Screenshot-2025-05-27-at-7.57.49 PM.png" width="856" />
 <p class="wp-caption-text" id="caption-attachment-14142">Figure 4 – Deployment in progress</p>
</div> 
<h4></h4> 
<h3>Configuring app to use large memory</h3> 
<p>After deployment starts, our app will fail its first build due to JavaScript heap out of memory issue. The runtime environment requests the heap memory, and the host operating system should allocate it. The JavaScript Node.js V8 runtime environment enforces a default heap size limit, which depends on several factors including the host memory size. That is why Standard and Large build instances use a default Node.js heap size of 2096 MB, while the XLarge instance uses a default heap size of 4144 MB.</p> 
<div class="wp-caption aligncenter" id="attachment_14143" style="width: 816px;">
 <img alt="Screenshot of a failed deployment in the amplify hosting console" class=" wp-image-14143" height="408" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/29/Screenshot-2025-05-28-at-12.30.00 AM.png" width="806" />
 <p class="wp-caption-text" id="caption-attachment-14143">Figure 5 – Deployment 1 failed due to heap size memory limit</p>
</div> 
<p>We can solve the conservative Node.js default heap memory limit issue by increasing the heap size. Since the app is using Standard build instance with 8 GiB memory, we will need to increase the heap size to 7000 MB which is ~7 GB. This leaves approximately 1 GB memory for operating system to use for other processes and avoids unintended behavior.</p> 
<p>Next, we will modify the build specification with the following command to increase Node.js heap memory limit.</p> 
<p><code>export NODE_OPTIONS='--max-old-space-size=7000'</code></p> 
<p>Alternately, <code>NODE_OPTIONS</code> can also be set as an environment variable in our app. Refer documentation for more details.</p> 
<p>We will also update the build spec file as following to accommodate these changes in our build flow:</p> 
<pre class="unlimited-height-code"><code class="lang-yaml">version: 1
frontend:
  phases:
    preBuild:
      commands:
        # Set the heap size to 7000 MB
        - export NODE_OPTIONS='--max-old-space-size=7000'
        # To check the heap size memory limit in MB
        - node -e "console.log('Total available heap size (MB):', v8.getHeapStatistics().heap_size_limit / 1024 / 1024)"
        - npm ci --cache .npm --prefer-offline
    build:
      commands:
        - npm run build
  artifacts:
    baseDirectory: .next
    files:
      - '**/*'
  cache:
    paths:
      - .next/cache/**/*
      - .npm/**/*</code></pre> 
<p>We will trigger a new deployment with the modified heap size by going to the Deployments page and clicking on Redeploy this version button</p> 
<p>However, the app still fails to build. This time, we see a log stating “Total available heap size (MB): 7048” that confirms our increased heap size. Interestingly, we also see a SIGKILL directive that indicates the operating system forcefully terminated the build process. This indicates another out of memory error.</p> 
<div class="wp-caption aligncenter" id="attachment_14144" style="width: 869px;">
 <img alt="Deployment 2 failed due to out of memory" class=" wp-image-14144" height="434" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/29/Screenshot-2025-05-28-at-12.49.25 AM.png" width="859" />
 <p class="wp-caption-text" id="caption-attachment-14144">Figure 6 – Deployment 2 failed due to out of memory</p>
</div> 
<h3></h3> 
<h3>Upgrading build instance</h3> 
<p><em>Standard</em> instance has a memory limit of 8 GiB and despite increasing the heap size close to this limit. Because of that, we are still running out of memory. We can solve this issue by upgrading our build instance to Large. This way we will allocate more memory to our build process.</p> 
<p>We will update the build spec file as following to deploy on <em>Large</em> instance:</p> 
<pre class="unlimited-height-code"><code class="lang-yaml">version: 1
frontend:
  phases:
    preBuild:
      commands:
        # Set the heap size to 14000 MB
        - export NODE_OPTIONS='--max-old-space-size=14000'
        # To check the heap size memory limit in MB
        - node -e "console.log('Total available heap size (MB):', v8.getHeapStatistics().heap_size_limit / 1024 / 1024)"
        - npm ci --cache .npm --prefer-offline
    build:
      commands:
        - npm run build
  artifacts:
    baseDirectory: .next
    files:
      - '**/*'
  cache:
    paths:
      - .next/cache/**/*
      - .npm/**/*

</code></pre> 
<p>We will update the build instance by opening the<strong> Hosting tab</strong>, and selecting <strong>Build settings</strong>. Afterwards, scroll down to <strong>Advanced settings</strong> and clicking on <strong>Edit.</strong> From the drop down, select the <strong>Build instance type</strong> as Large.</p> 
<div class="wp-caption aligncenter" id="attachment_14145" style="width: 928px;">
 <img alt="Screenshot of upgrading the instance to large using the dropdown in the amplify hosting console " class="wp-image-14145" height="253" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/29/Screenshot-2025-05-28-at-12.59.46 AM.png" width="918" />
 <p class="wp-caption-text" id="caption-attachment-14145">Figure 7 – Update build instance size to Large</p>
</div> 
<p>Next, on the <strong>Deployments</strong> page, we will click on <strong>Redeploy this version</strong> to create a new deployment with Large instance. We also notice the first log line in build logs showing the build instance specifications.</p> 
<div class="wp-caption aligncenter" id="attachment_14146" style="width: 867px;">
 <img alt="Deployment 3 in progress on Large build instance" class=" wp-image-14146" height="432" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/29/tt.png" width="857" />
 <p class="wp-caption-text" id="caption-attachment-14146">Figure 8 – Deployment 3 in progress on Large build instance</p>
</div> 
<p>We can see that our app is deployed successfully.</p> 
<div class="wp-caption aligncenter" id="attachment_14147" style="width: 935px;">
 <img alt="App is deployed" class=" wp-image-14147" height="309" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/29/Screenshot-2025-05-28-at-1.19.25 AM.png" width="925" />
 <p class="wp-caption-text" id="caption-attachment-14147">Figure 10 – App is deployed</p>
</div> 
<h2>Conclusion</h2> 
<p>In this blog, we have explored how you can create and update app workflows for configuring build instance sizes and demonstrated how you can solve memory issues in your projects hosted by Amplify Hosting. To explore further about build instance sizes and learn more about this feature, check out the <a href="https://docs.aws.amazon.com/amplify/latest/userguide/custom-build-instance.html">Amplify Hosting documentation</a>. For information on pricing for build instances, you can check our <a href="https://aws.amazon.com/amplify/pricing/">pricing page</a>.</p> 
<h3></h3> 
<h2>About the Authors</h2> 
<p><img alt="Photo of Sohaib Headshot" class="alignleft wp-image-14148 size-thumbnail" height="150" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/29/Sohaib-scaled-e1748544722427-150x150.jpg" width="150" /></p> 
<p><strong>Sohaib Uddin Syed, Software Engineer, Amplify Hosting</strong></p> 
<p>Sohaib Uddin Syed is a Software Development Engineer at AWS Amplify Hosting. Sohaib builds software that enables customers to build their front-end web applications using the scalability of AWS. In his free time, Sohaib enjoys watching football (soccer), cooking and spending time with family and friends.</p> 
<p><img alt="Angela Zheng headshot" class="size-thumbnail wp-image-14149 alignleft" height="150" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/29/Screenshot-2025-05-29-at-9.50.45 AM-150x150.png" width="150" /><strong>Angela Zheng, Software Engineer, Amplify Hosting</strong></p> 
<p>Angela Zheng is a Software Development Engineer at AWS Amplify Hosting. She works on creating features that make hosting front-end web applications accessible and straightforward for developers at any scale. In her free time, she enjoys dancing, playing a game of volleyball, experimenting with recipes in the kitchen, and traveling.</p> 
<p><img alt="Matt Headshot" class=" wp-image-12567 alignleft" height="153" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2024/02/19/Screenshot-2024-02-19-at-3.13.45 PM.png" width="149" /></p> 
<p><strong>Matt Auerbach, Sr Product Manager, Amplify Hosting</strong></p> 
<p>Matt Auerbach is a NYC-based Product Manager on the AWS Amplify Team. He educates developers regarding products and offerings, and acts as the primary point of contact for assistance and feedback. Matt is a mild-mannered programmer who enjoys using technology to solve problems and making people’s lives easier. B night, however…well he does pretty much the same thing. You can find Matt on&nbsp;X @mauerbac. He previously worked at Twitch, Optimizely &amp; Twilio.</p>
