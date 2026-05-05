---
title: "IAM Compute Roles for Server-Side Rendering with AWS Amplify Hosting"
url: "https://aws.amazon.com/blogs/mobile/iam-compute-roles-for-server-side-rendering-with-aws-amplify-hosting/"
date: "Tue, 18 Feb 2025 21:17:34 +0000"
author: "Jay Raval"
feed_url: "https://aws.amazon.com/blogs/mobile/feed/"
---
<p>Today, <a href="https://aws.amazon.com/amplify/hosting/">AWS Amplify Hosting</a> is introducing compute roles for AWS Amplify applications, enabling you to extend server-side rendering capabilities with secure access to AWS services from the compute runtime. With compute roles, developers can attach specific permissions to their server-side rendered apps, allowing Amplify to make authorized calls to other AWS services. This new capability helps streamline the development process while maintaining security best practices for your applications.</p> 
<h2><span style="text-decoration: underline;">Key capabilities</span></h2> 
<p>With compute roles, you can now:</p> 
<ol> 
 <li>Access sensitive configuration data at runtime within Next.js API routes using <a href="https://aws.amazon.com/secrets-manager/">AWS Secrets Manager</a> and <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html">AWS Systems Manager Parameter Store</a></li> 
 <li>Connect your SSR apps directly to databases like <a href="https://aws.amazon.com/rds/">Amazon RDS</a>, <a href="https://aws.amazon.com/dynamodb/">Amazon DynamoDB</a>. and other AWS databases</li> 
 <li>Define fine-grained permissions for your compute environment using AWS Identity and Access Management (IAM) policies</li> 
 <li>Make secure, authenticated calls to any AWS service from your server-side code</li> 
</ol> 
<h2><span style="text-decoration: underline;">Tutorial</span></h2> 
<p>To get started, follow these steps to create an IAM compute role and associate it with an Amplify Next.js application:</p> 
<h3><span style="text-decoration: underline;">Prerequisites</span></h3> 
<p>Before you begin, make sure you have the following installed:</p> 
<ul> 
 <li><a href="https://nodejs.org/en">Node.js</a> (v18.x or later)</li> 
 <li><a href="https://docs.npmjs.com/getting-started">npm</a> or <a href="https://docs.npmjs.com/cli/v11/commands/npx">npx</a> (v10.x or later)</li> 
 <li><a href="https://git-scm.com/">git</a> (v2.39.5 or later)</li> 
 <li><a href="https://aws.amazon.com/cli/">AWS CLI</a></li> 
</ul> 
<h3><span style="text-decoration: underline;">Create an IAM compute role</span></h3> 
<p>Let’s create an IAM role that allows Amplify Hosting’s SSR compute service to securely access AWS resources based on the role’s permission and trust relationship. In our example, we’ll securely access an Amazon Simple Storage Service (Amazon S3) bucket.</p> 
<ol> 
 <li>Sign in to the AWS Management console and navigate to the <a href="https://us-east-1.console.aws.amazon.com/iamv2/home?">IAM console</a></li> 
 <li>Under the Access Management tab, choose Roles, then select <strong>Create role</strong></li> 
 <li>Select Custom trust policy and enter the following policy to allow Amplify to assume the role:</li> 
</ol> 
<pre><code class="lang-json">{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": [
                    "amplify.amazonaws.com"
                ]
            },
            "Action": "sts:AssumeRole"
        }
    ]
}</code></pre> 
<ol start="4"> 
 <li>Add the AWS managed <code>AmazonS3ReadOnlyAccess</code> permission policy to the role and choose <strong>Next</strong></li> 
 <li>Name the role <code>amplify-compute-role</code> and choose <strong>Create role</strong></li> 
</ol> 
<p>You have successfully created a compute role that Amplify can assume to connect to other AWS services.</p> 
<p><span style="text-decoration: underline;"><strong>Security Best Practice</strong>:</span> Implement the principle of least privilege by granting only essential permissions for your specific use case.</p> 
<h3><span style="text-decoration: underline;">Create an Amazon S3 bucket</span></h3> 
<p>We will now setup a private Amazon S3 bucket that the Next.js app can securely access using the compute role. Instead of exposing Amazon S3 objects publicly or managing access keys, we’ll use Amplify Hosting’s compute role to securely retrieve private content from Amazon S3.</p> 
<ol> 
 <li>Sign in to the AWS Management console and navigate to the <a href="https://us-east-1.console.aws.amazon.com/s3/home?">Amazon S3 console</a></li> 
 <li>Choose <strong>Create bucket</strong> and enter a unique bucket name</li> 
 <li>Keep <strong>ACLs disabled</strong> and <strong>Block all public access</strong> enabled</li> 
</ol> 
<p><img alt="Object Ownership and Public Access settings for the bucket" class="alignleft wp-image-13806 size-full" height="1450" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/02/18/Screenshot-2025-02-14-at-2.54.30 PM.png" width="1682" /></p> 
<ol start="4"> 
 <li>Keep the default settings and choose <strong>Create bucket</strong></li> 
 <li>Navigate to the bucket and choose <strong>Upload</strong></li> 
 <li>Upload an image (e.g. amplify-logo.png)</li> 
</ol> 
<p><span style="text-decoration: underline;"><strong>Security Verification</strong>:</span> To confirm that your S3 bucket is completely locked down, run the following <a href="https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3api/get-public-access-block.html">get-public-access-block</a> AWS CLI command:</p> 
<pre><code class="lang-bash">// Note: Replace amplify-compute-role-demo with your specific bucket name
aws s3api get-public-access-block --bucket amplify-compute-role-demo</code></pre> 
<p>The expected output should show:</p> 
<pre><code class="lang-json">{
    "PublicAccessBlockConfiguration": {
        "BlockPublicAcls": true,
        "IgnorePublicAcls": true,
        "BlockPublicPolicy": true,
        "RestrictPublicBuckets": true
    }
}</code></pre> 
<p>These settings ensure:</p> 
<ul> 
 <li>No public ACLs can be created</li> 
 <li>Existing public ACLs are ignored</li> 
 <li>Public bucket policies are blocked</li> 
 <li>Public access to the bucket is restricted</li> 
</ul> 
<h3><span style="text-decoration: underline;">Create a Next.js app</span></h3> 
<p>Next, we will create a Next.js app to securely access private content (an image) from an Amazon S3 bucket using an API route.</p> 
<ol> 
 <li>Create a new Next.js 15 app with Typescript and Tailwind CSS</li> 
</ol> 
<pre><code class="lang-bash">npx create-next-app@latest compute-role-demo --typescript --tailwind --eslint
cd compute-role-demo</code></pre> 
<ol start="2"> 
 <li>Install the AWS SDK for JavaScript S3 Client package:</li> 
</ol> 
<pre><code class="lang-bash">npm install @aws-sdk/client-s3</code></pre> 
<ol start="3"> 
 <li>Create an Amazon S3 Image API route:</li> 
</ol> 
<pre><code class="lang-ts">// app/api/image/route.ts

import { S3Client, GetObjectCommand, S3ServiceException } from "@aws-sdk/client-s3";
import { NextResponse } from 'next/server';

const BUCKET_NAME = 'amplify-compute-role-demo';
const IMAGE_KEY = 'amplify-logo.png';

export async function GET() {
  console.log(`[S3 Image Request] Starting - Bucket: ${BUCKET_NAME}, Key: ${IMAGE_KEY}`);
  const s3Client = new S3Client({});
  
  try {
    console.log('[S3 Image Request] Creating GetObjectCommand...');
    const command = new GetObjectCommand({
      Bucket: BUCKET_NAME,
      Key: IMAGE_KEY
    });

    console.log('[S3 Image Request] Sending request to S3...');
    const response = await s3Client.send(command);
    console.log('[S3 Image Request] Received S3 response:', {
      contentType: response.ContentType,
      contentLength: response.ContentLength,
      metadata: response.Metadata
    });

    console.log('[S3 Image Request] Converting response to byte array...');
    const buffer = await response.Body?.transformToByteArray();

    if (!buffer) {
      console.error('[S3 Image Request] No buffer received from S3');
      throw new Error('No image data received from S3');
    }

    console.log('[S3 Image Request] Successfully processed image:', {
      bufferSize: buffer.length,
      contentType: response.ContentType
    });

    return new NextResponse(buffer, {
      headers: {
        'Content-Type': response.ContentType || 'image/png',
        'Cache-Control': 'public, max-age=31536000, immutable'
      }
    });

  } catch (error) {
    if (error instanceof S3ServiceException) {
      console.error('[S3 Image Request] AWS S3 Error:', {
        message: error.message,
        code: error.name,
        requestId: error.$metadata?.requestId,
        statusCode: error.$metadata?.httpStatusCode
      });
    } else {
      console.error('[S3 Image Request] Unexpected error:', error);
    }

    return NextResponse.json({
      success: false,
      error: 'Failed to load content'
    }, { 
      status: 500 
    });
  } finally {
    console.log('[S3 Image Request] Request completed');
  }
}</code></pre> 
<ol start="4"> 
 <li>Create a client component:</li> 
</ol> 
<pre><code class="lang-ts">// app/components/S3Component.tsx

'use client';

import { useState } from 'react';
import Image from 'next/image';

export default function S3Component() {
  const [imageError, setImageError] = useState(false);
  const [isRevealed, setIsRevealed] = useState(false);

  return (
    &lt;div className="absolute inset-0 top-[88px] flex items-center justify-center"&gt;
      &lt;div className="w-full max-w-2xl rounded-2xl mx-4"&gt;
        &lt;div className="flex flex-col items-center justify-center min-h-[400px] p-8"&gt;
          {!isRevealed ? (
            &lt;button
              onClick={() =&gt; setIsRevealed(true)}
              className="px-8 py-4 bg-[rgb(117,81,194)] text-white text-lg rounded-xl 
                       hover:bg-[rgb(107,71,184)] transition-colors"
            &gt;
              &lt;span className="flex items-center gap-2"&gt;
                Access Private S3 Bucket
                &lt;span className="text-xl"&gt;⚡&lt;/span&gt;
              &lt;/span&gt;
            &lt;/button&gt;
          ) : (
            &lt;div className="space-y-8 w-full"&gt;              
              {!imageError &amp;&amp; (
                &lt;div className="relative h-64 w-full"&gt;
                  &lt;Image 
                    src="/api/image"
                    alt="Amplify Logo"
                    fill
                    unoptimized
                    priority
                    onError={() =&gt; setImageError(true)}
                    className="object-contain"
                    sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
                  /&gt;
                &lt;/div&gt;
              )}
              
              &lt;div className="text-center"&gt;
                &lt;p className="text-gray-400 text-sm"&gt;
                  This content is securely served from S3 using an IAM Compute Role.
                &lt;/p&gt;
              &lt;/div&gt;
            &lt;/div&gt;
          )}
        &lt;/div&gt;
      &lt;/div&gt;
    &lt;/div&gt;
  );
}</code></pre> 
<ol start="5"> 
 <li>Update the home page to render the client component:</li> 
</ol> 
<pre><code class="lang-ts">import S3Component from './components/S3Component';

export default function HomePage() {
  return (
    &lt;div className="relative min-h-screen bg-[rgb(0,0,0)]"&gt;
      &lt;h1 className="text-2xl font-bold text-white py-8 text-center relative z-10"&gt;
        Amplify Hosting Compute Role Demo
      &lt;/h1&gt;
      &lt;S3Component /&gt;
    &lt;/div&gt;
  );
}</code></pre> 
<ol start="6"> 
 <li>Push the changes to a git repository: 
  <ol> 
   <li>Create a <a href="https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository">new</a> GitHub repository</li> 
   <li>Add and commit the changes to the git branch</li> 
   <li>Add remote origin and push the changes upstream</li> 
  </ol> </li> 
</ol> 
<pre><code class="lang-git">git add .
git commit -m "initial commit"
git remote add origin https://github.com/&lt;OWNER&gt;/amplify-compute-role-demo.git
git push -u origin main</code></pre> 
<h3><span style="text-decoration: underline;">Deploy the application to Amplify Hosting</span></h3> 
<ol> 
 <li>Sign in to the AWS Management Console and navigate to the <a href="https://us-east-1.console.aws.amazon.com/amplify/home?">Amplify console</a></li> 
 <li>Choose <strong>Create new app</strong> and select GitHub as the repository source</li> 
 <li>Authorize Amplify to access your GitHub account</li> 
 <li>Choose the repository and branch you created</li> 
 <li>Review the App settings and then choose <strong>Next</strong></li> 
 <li>Review the overall settings and choose <strong>Save and deploy</strong></li> 
</ol> 
<h3><span style="text-decoration: underline;">Attaching the IAM compute role to an Amplify app</span></h3> 
<ol> 
 <li>In the Amplify console, select your app and navigate to App settings &gt; IAM roles</li> 
</ol> 
<pre><img alt="IAM roles section displayed in the Amplify console" class="alignnone wp-image-13815 size-full" height="693" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/02/18/Screenshot-2025-02-06-at-1.00.12 PM.png" width="1273" /></pre> 
<ol start="2"> 
 <li>Choose <strong>Edit</strong> in the compute role section</li> 
</ol> 
<p><img alt="Compute role section displayed in the Amplify console" class="alignnone wp-image-13816 size-full" height="416" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/02/18/Screenshot-2025-02-06-at-1.01.29 PM.png" width="1279" /></p> 
<ol start="3"> 
 <li>From the role menu, select the <code>amplify-compute-role</code> and choose <strong>Save</strong></li> 
 <li>You can also add branch overrides to use a unique compute role per branch. This can particularly be helpful across different git environments such as dev, staging, or production.</li> 
</ol> 
<h3><span style="text-decoration: underline;">Access the deployed Next.js app</span></h3> 
<p>Navigate to the app’s <strong>Overview</strong> tab in the Amplify console and open the default <strong>Amplify generated URL</strong> in the browser.</p> 
<p><img alt="Overview tab with a Visit deployed URL button in the Amplify console" class="alignnone wp-image-13819 size-full" height="322" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/02/18/Screenshot-2025-02-17-at-5.54.53 PM.png" width="1013" /></p> 
<p>Next, choose the <strong>Access Private S3</strong> button. You should now observe the image that was uploaded to the Amazon S3 bucket.</p> 
<p><img alt="" class="alignnone wp-image-13818 size-large" height="777" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/02/18/Screenshot-2025-02-14-at-3.30.06 PM-1024x777.png" width="1024" /></p> 
<p><img alt="" class="alignnone wp-image-13817 size-large" height="828" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/02/18/Screenshot-2025-02-14-at-3.30.13 PM-1024x828.png" width="1024" /></p> 
<h3><span style="text-decoration: underline;">Review the hosting compute logs</span></h3> 
<p>From the App home page, navigate to Hosting &gt; Monitoring and then choose the <strong>Hosting compute logs</strong> tab.</p> 
<p><img alt="Monitoring section of the app displayed in the Amplify console" class="alignnone wp-image-13821 size-full" height="544" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/02/18/Screenshot-2025-02-06-at-12.49.29 PM.png" width="904" /></p> 
<p>Navigate to the Amazon CloudWatch log streams URL and review the latest log stream. The logs should contain the Amazon S3 image requests:</p> 
<p><img alt="Hosting compute logs displaying Amazon S3 image requests" class="alignnone wp-image-13822 size-full" height="1658" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/02/18/Screenshot-2025-02-14-at-3.37.04 PM-1.png" width="2898" /></p> 
<p>Congratulations! You’ve successfully created and attached an IAM compute role to your Next.js SSR application on Amplify Hosting, enabling secure content retrieval from your private Amazon S3 bucket! ?</p> 
<h2><span style="text-decoration: underline;">Cleanup</span></h2> 
<ol> 
 <li>Delete the AWS Amplify app by navigating to App settings &gt; General settings, then choose Delete app.</li> 
 <li>Delete the Amazon S3 bucket by navigating to S3 console &gt; Select bucket &gt; Select Delete &gt; Enter the bucket name to confirm deletion</li> 
 <li>Delete the IAM role by navigating to IAM console &gt; Select Roles &gt; Find the amplify-compute-role name &gt; Select Delete &gt; Enter the role name to confirm deletion</li> 
</ol> 
<h2><span style="text-decoration: underline;">Summary</span></h2> 
<p>AWS Amplify Hosting now offers compute roles for server-side rendered (SSR) applications, solving the challenge of securely accessing AWS services. Previously, developers manually managed credentials through environment variables. Now, with compute roles, developers can directly attach IAM permissions to their SSR apps, simplifying service integration and enhancing security.</p> 
<p>Add compute roles to your SSR apps today by checking out the Amplify <a href="https://docs.aws.amazon.com/amplify/latest/userguide/amplify-SSR-compute-role.html">documentation</a>.</p> 
<div class="blog-author-box"> 
 <div class="blog-author-box"> 
  <div class="blog-author-box"> 
   <h2><span style="text-decoration: underline;">About the Authors</span></h2> 
   <footer> 
    <table style="width: 100%; text-align: left;"> 
     <tbody> 
      <tr> 
       <td> 
        <div class="blog-author-box"> 
         <div class="blog-author-image">
          <img alt="Photo of author" class="wp-image-11636 alignleft" height="160" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2024/08/13/IMG_9570.jpg" width="120" />
         </div> 
         <h3 class="lb-h4">Jay Raval</h3> 
         <p>Jay Raval is a Solutions Architect on the AWS Amplify team. He’s passionate about solving complex customer problems in the front-end, web and mobile domain and addresses real-world architecture problems for development using front-end technologies and AWS. In his free time, he enjoys traveling and sports. You can find Jay on <a href="https://x.com/_Jay_Raval_">X @_Jay_Raval_</a></p> 
        </div> </td> 
      </tr> 
      <tr> 
       <td> 
        <div class="blog-author-box"> 
         <div class="blog-author-image">
          <img alt="Photo of author" class="wp-image-11636 alignleft" height="160" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2024/02/19/Screenshot-2024-02-19-at-3.13.45 PM.png" width="120" />
         </div> 
         <h3 class="lb-h4">Matt Auerbach</h3> 
         <p>Matt Auerbach is a NYC-based Product Manager on the AWS Amplify Team. He educates developers regarding products and offerings, and acts as the primary point of contact for assistance and feedback. Matt is a mild-mannered programmer who enjoys using technology to solve problems and making people’s lives easier. B night, however…well he does pretty much the same thing. You can find Matt on <a href="https://x.com/mauerbac">X @mauerbac</a>. He previously worked at Twitch, Optimizely &amp; Twilio.</p> 
        </div> </td> 
      </tr> 
     </tbody> 
    </table> 
   </footer> 
  </div> 
 </div> 
</div>
