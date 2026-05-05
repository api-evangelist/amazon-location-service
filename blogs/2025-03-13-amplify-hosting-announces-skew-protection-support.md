---
title: "Amplify Hosting Announces Skew Protection Support"
url: "https://aws.amazon.com/blogs/mobile/amplify-hosting-announces-skew-protection/"
date: "Thu, 13 Mar 2025 19:38:18 +0000"
author: "Matt Auerbach"
feed_url: "https://aws.amazon.com/blogs/mobile/feed/"
---
<p>A common challenge in web application development and deployment is version skew between client and server resources. Today, we’re excited to announce deployment skew protection for applications deployed to <a href="https://aws.amazon.com/amplify/hosting">AWS Amplify Hosting</a>. This feature helps verifies end users have a seamless experience during application deployments.</p> 
<h2>The Challenge</h2> 
<p>Modern web applications are complex systems comprising numerous static assets and server-side components that must all work together. In a world where its common to have multiple deployments occur hourly, version compatibility becomes a critical concern. When new deployments occur, users with cached versions of your application may attempt to fetch resources from the updated deployment, potentially resulting in 404 errors and broken functionality.</p> 
<p>This challenge is compounded by client-server version skew, which manifests in two common scenarios. First, users often keep browser tabs open for extended periods, continuing to run older versions of your application while attempting to interact with updated backend services. Second, in mobile applications, users with disabled auto-updates may continue using outdated versions indefinitely, requiring backend services to maintain compatibility with multiple client versions simultaneously.</p> 
<p>These version management challenges can significantly impact user experience and application reliability if not properly addressed. Consider the scenario:</p> 
<ol> 
 <li>A user loads your application (version A)</li> 
 <li>You deploy a new version (version B)</li> 
 <li>The user’s cached JavaScript tries to load assets that only existed in version A</li> 
 <li>Result: broken functionality and poor user experience</li> 
</ol> 
<h3><strong>How skew protection works</strong></h3> 
<p>Amplify Hosting now intelligently coordinates request resolution across multiple client sessions, ensuring precise routing to intended deployment versions.</p> 
<p><strong>1. Smart asset routing</strong></p> 
<p>When a request comes in, Amplify Hosting now:</p> 
<ul> 
 <li>Identifies the deployment version that originated the request</li> 
 <li>Routes and resolves the request to the identified version of the asset</li> 
 <li>A hard refresh will always serve the latest deployment assets in a user session</li> 
</ul> 
<p><strong>2. Consistent version serving</strong></p> 
<p>The system confirms that:</p> 
<ul> 
 <li>All assets from a single user session come from the same deployment</li> 
 <li>New user sessions always get the latest version</li> 
 <li>Existing sessions continue working with their original version until refresh</li> 
</ul> 
<h3>Advantages to skew protection</h3> 
<p>Here are some of the advantages of having skew protection:</p> 
<ul> 
 <li>Zero Configuration: Works out of the box with popular frameworks</li> 
 <li>Reliable Deployments: Eliminate 404 errors due to deployment skew</li> 
 <li>Performance Optimized: Minimal impact on response times</li> 
</ul> 
<h3>Best practices</h3> 
<p>While skew protection handles most scenarios automatically, we recommend:</p> 
<ol> 
 <li>Using atomic deployments</li> 
 <li>Testing your deployment process in a staging environment</li> 
</ol> 
<h2><strong>Enabling Skew Protection</strong></h2> 
<p>Skew protection must be enabled for each Amplify app. This is done at the branch level for every Amplify Hosting app. You can read the full <a href="https://docs.aws.amazon.com/amplify/latest/userguide/skew-protection.html">Amplify Hosting documentation</a> here.</p> 
<p>1. To enable a branch, click on <strong>App Settings</strong>, then click <strong>Branch Settings</strong>. <strong>Next</strong>, select the branch you want to enable followed by the <strong>Actions tab</strong>.</p> 
<p><img alt="Screenshot of the branches tab in Amplify Console settings" class="alignnone size-full wp-image-13908" height="309" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/03/11/Screenshot-2025-03-04-at-4.21.35 PM.png" width="1530" /></p> 
<p style="text-align: center;">FIGURE 1 – Amplify Hosting Branch Settings</p> 
<p style="text-align: left;">&nbsp;2. For skew protection to take effect, you must deploy the application once.</p> 
<p style="text-align: left;">Note: Skew protection will not be available to customers using our legacy SSRv1/WEB_DYNAMIC applications.</p> 
<p style="text-align: left;"><span style="text-decoration: underline;">Pricing:</span> There is <strong>no additional cost</strong> for this feature and it is available to all Amplify Hosting regions.</p> 
<h2>Tutorial</h2> 
<p>To get started, follow these steps to create a Next.js application and enable skew protection on it.</p> 
<h3>Prerequisites</h3> 
<p>Before you begin, make sure you have the following installed:</p> 
<ul> 
 <li>Node.js (v18.x or later)</li> 
 <li>npm or npx (v10.x or later)</li> 
 <li>Git (v2.39.5 or later)</li> 
</ul> 
<h3>Create a Next.js app</h3> 
<p>Let’s create a Next.js app to see skew protection in action.</p> 
<ol> 
 <li>Create a new Next.js 15 app with <a href="https://www.typescriptlang.org/">Typescript</a> and <a href="https://tailwindcss.com/">Tailwind CSS</a></li> 
</ol> 
<pre><code class="lang-bash">$ npx create-next-app@latest skew-protection-demo --typescript --tailwind --eslint
$ cd skew-protection-demo</code></pre> 
<p>2. Create a SkewProtectionDemo component that lists fingerprinted assets with deployment IDs. Use the following code to create the component.</p> 
<p>Note: Amplify will automatically tag the fingerprinted assets of a NextJS app with a <code>dpl</code> query parameter set to a UUID. This UUID is also available during the build via the <code>AWS_AMPLIFY_DEPLOYMENT_ID</code> environment variable for other frameworks.</p> 
<pre><code class="lang-ts">// app/components/SkewProtectionDemo.tsx

"use client";

import { useState, useEffect } from "react";
import DeploymentTester from "./DeploymentTester";

interface Asset {
  type: string;
  url: string;
  dpl: string;
}

export default function SkewProtectionDemo() {
  const [fingerprintedAssets, setFingerprintedAssets] = useState&lt;Asset[]&gt;([]);
  const [deploymentId, setDeploymentId] = useState&lt;string&gt;("Unknown");

  // Detect all assets with dpl parameters on initial load
  useEffect(() =&gt; {
    detectFingerprintedAssets();
  }, []);

  // Function to detect all assets with dpl parameters
  const detectFingerprintedAssets = () =&gt; {
    // Find all assets with dpl parameter (CSS and JS)
    const allElements = [
      ...Array.from(document.querySelectorAll('link[rel="stylesheet"]')),
      ...Array.from(document.querySelectorAll("script[src]"))
    ];
    
    const assets = allElements
      .map(element =&gt; {
        const url = element.getAttribute(element.tagName.toLowerCase() === "link" ? "href" : "src");
        if (!url || !url.includes("?dpl=")) return null;

        // Extract the dpl parameter
        let dplParam = "unknown";
        try {
          const urlObj = new URL(url, window.location.origin);
          dplParam = urlObj.searchParams.get("dpl") || "unknown";
        } catch (e) {
          console.error("Error parsing URL:", e);
        }

        return {
          type: element.tagName.toLowerCase() === "link" ? "css" : "js",
          url: url,
          dpl: dplParam,
        };
      })
      .filter(asset =&gt; asset !== null);

    setFingerprintedAssets(assets);
    
    // Set deployment ID if assets were found
    if (assets.length &gt; 0) {
      setDeploymentId(assets[0]?.dpl || "Unknown");
    }
  };

  // Function to format URL to highlight the dpl parameter
  const formatUrl = (url: string) =&gt; {
    if (!url.includes("?dpl=")) return url;
    
    const [baseUrl, params] = url.split("?");
    const dplParam = params.split("&amp;").find(p =&gt; p.startsWith("dpl="));
    
    if (!dplParam) return url;
    
    const otherParams = params.split("&amp;").filter(p =&gt; !p.startsWith("dpl=")).join("&amp;");
    
    return (
      &lt;&gt;
        &lt;span className="text-gray-400"&gt;{baseUrl}?&lt;/span&gt;
        {otherParams &amp;&amp; &lt;span className="text-gray-400"&gt;{otherParams}&amp;&lt;/span&gt;}
        &lt;span className="text-yellow-300 font-bold"&gt;{dplParam}&lt;/span&gt;
      &lt;/&gt;
    );
  };

  return (
    &lt;main className="min-h-screen p-6 bg-white"&gt;
      &lt;div className="w-full max-w-2xl mx-auto"&gt;
        &lt;h1 className="text-2xl font-bold text-gray-900 mb-6"&gt;
          Amplify Skew Protection Demo
        &lt;/h1&gt;

        &lt;div className="grid grid-cols-1 gap-4 mb-6"&gt;
          &lt;div className="flex items-center p-4 bg-white text-gray-800 rounded-md border border-gray-200 hover:bg-gray-50 transition-colors"&gt;
            &lt;div className="w-10 h-10 flex items-center justify-center bg-gray-100 rounded-lg mr-3"&gt;
              &lt;span className="text-lg"&gt;?&lt;/span&gt;
            &lt;/div&gt;
            &lt;div&gt;
              &lt;p className="font-medium"&gt;Zero-Downtime Deployments&lt;/p&gt;
              &lt;p className="text-xs text-gray-600"&gt;Assets and API routes remain accessible during deployments using deployment ID-based routing&lt;/p&gt;
            &lt;/div&gt;
          &lt;/div&gt;
          
          &lt;div className="flex items-center p-4 bg-white text-gray-800 rounded-md border border-gray-200 hover:bg-gray-50 transition-colors"&gt;
            &lt;div className="w-10 h-10 flex items-center justify-center bg-gray-100 rounded-lg mr-3"&gt;
              &lt;span className="text-lg"&gt;⚡&lt;/span&gt;
            &lt;/div&gt;
            &lt;div&gt;
              &lt;p className="font-medium"&gt;Built-in Next.js Support&lt;/p&gt;
              &lt;p className="text-xs text-gray-600"&gt;Automatic asset fingerprinting and deployment ID injection for Next.js applications&lt;/p&gt;
            &lt;/div&gt;
          &lt;/div&gt;
          
          &lt;div className="flex items-center p-4 bg-white text-gray-800 rounded-md border border-gray-200 hover:bg-gray-50 transition-colors"&gt;
            &lt;div className="w-10 h-10 flex items-center justify-center bg-gray-100 rounded-lg mr-3"&gt;
              &lt;span className="text-lg"&gt;?&lt;/span&gt;
            &lt;/div&gt;
            &lt;div&gt;
              &lt;p className="font-medium"&gt;Advanced Security&lt;/p&gt;
              &lt;p className="text-xs text-gray-600"&gt;Protect against compromised builds by calling the delete-job API to remove affected deployments&lt;/p&gt;
            &lt;/div&gt;
          &lt;/div&gt;
        &lt;/div&gt;

        &lt;div className="bg-gradient-to-r from-blue-900 to-purple-900 p-4 rounded-md mb-6"&gt;
          &lt;h2 className="text-sm font-medium text-blue-200 mb-1"&gt;
            Current Deployment ID
          &lt;/h2&gt;
          &lt;div className="p-2 bg-black bg-opacity-30 rounded-md font-mono text-lg text-center text-yellow-300"&gt;
            {deploymentId}
          &lt;/div&gt;
        &lt;/div&gt;

        {fingerprintedAssets.length &gt; 0 ? (
          &lt;&gt;
            &lt;h2 className="text-xl font-bold text-gray-900 mb-6"&gt;
              Fingerprinted Assets
            &lt;/h2&gt;
            &lt;div className="border border-gray-200 rounded-md overflow-hidden mb-6 bg-white"&gt;
              &lt;div className="max-h-48 overflow-y-auto"&gt;
                &lt;table className="min-w-full divide-y divide-gray-200"&gt;
                  &lt;thead className="bg-gray-50"&gt;
                    &lt;tr&gt;
                      &lt;th className="px-3 py-2 text-left text-xs font-medium text-gray-900 uppercase tracking-wider w-16"&gt;
                        Type
                      &lt;/th&gt;
                      &lt;th className="px-3 py-2 text-left text-xs font-medium text-gray-900 uppercase tracking-wider"&gt;
                        URL
                      &lt;/th&gt;
                    &lt;/tr&gt;
                  &lt;/thead&gt;
                  &lt;tbody className="divide-y divide-gray-200"&gt;
                    {fingerprintedAssets.map((asset, index) =&gt; (
                      &lt;tr key={index} className="bg-white hover:bg-gray-50"&gt;
                        &lt;td className="px-3 py-2 text-sm text-gray-900"&gt;
                          &lt;span
                            className={`inline-block px-2 py-0.5 rounded-full text-xs ${
                              asset.type === "css"
                                ? "bg-blue-100 text-blue-800"
                                : "bg-yellow-100 text-yellow-800"
                            }`}
                          &gt;
                            {asset.type}
                          &lt;/span&gt;
                        &lt;/td&gt;
                        &lt;td className="px-3 py-2 text-xs font-mono break-all text-gray-900"&gt;
                          {formatUrl(asset.url)}
                        &lt;/td&gt;
                      &lt;/tr&gt;
                    ))}
                  &lt;/tbody&gt;
                &lt;/table&gt;
              &lt;/div&gt;
            &lt;/div&gt;
            
            &lt;h2 className="text-xl font-bold text-gray-900 mb-6"&gt;
              Test Deployment Routing
            &lt;/h2&gt;
            &lt;DeploymentTester /&gt;
          &lt;/&gt;
        ) : (
          &lt;div className="p-6 text-center text-gray-600 border border-gray-200 rounded-md"&gt;
            &lt;p&gt;No fingerprinted assets detected&lt;/p&gt;
            &lt;p className="text-sm mt-2"&gt;
              Deploy to Amplify Hosting to see skew protection in action
            &lt;/p&gt;
          &lt;/div&gt;
        )}
      &lt;/div&gt;
    &lt;/main&gt;
  );
}</code></pre> 
<p>3. Next, create a DeploymentTester component that demonstrates how API requests maintain deployment consistency by sending the <code>X-Amplify-Dpl</code> header with each request, allowing Amplify to route to the correct API version. Use the following code to create the component.</p> 
<pre><code class="lang-ts">// app/components/DeploymentTester.tsx

'use client';

import { useState } from 'react';

interface ApiResponse {
  message: string;
  timestamp: string;
  version: string;
  deploymentId: string;
}

export default function DeploymentTester() {
  const [testInProgress, setTestInProgress] = useState(false);
  const [testOutput, setTestOutput] = useState&lt;ApiResponse | null&gt;(null);
  const [callCount, setCallCount] = useState(0);

  const runApiTest = async () =&gt; {
    setTestInProgress(true);
    setCallCount(prev =&gt; prev + 1);
    
    try {
      const response = await fetch('/api/skew-protection', {
        headers: {
        // Amplify provides the deployment ID as an environment variable during build time
          'X-Amplify-Dpl': process.env.AWS_AMPLIFY_DEPLOYMENT_ID || '',
        }
      });
      
      if (!response.ok) {
        throw new Error(`API returned ${response.status}`);
      }
      const data = await response.json();
      setTestOutput(data);
    } catch (error) {
      console.error("API call failed", error);
      setTestOutput({
        message: `Error: ${error instanceof Error ? error.message : 'Unknown error'}`,
        timestamp: new Date().toISOString(),
        version: 'error',
        deploymentId: 'error'
      });
    } finally {
      setTestInProgress(false);
    }
  };

  return (
    &lt;div className="border border-gray-200 rounded-md overflow-hidden bg-white"&gt;
      &lt;div className="bg-gray-50 px-4 py-3 flex justify-between items-center border-b border-gray-200"&gt;
        &lt;span className="font-medium text-gray-800"&gt;Test Deployment Routing&lt;/span&gt;
        &lt;button
          onClick={runApiTest}
          disabled={testInProgress}
          className={`px-3 py-1 rounded text-sm ${
            testInProgress
              ? 'bg-gray-200 text-gray-500 cursor-not-allowed'
              : 'bg-blue-600 hover:bg-blue-700 text-white'
          }`}
        &gt;
          {testInProgress ? "Testing..." : "Test Route"}
        &lt;/button&gt;
      &lt;/div&gt;
      
      &lt;div className="p-4"&gt;
        {testOutput ? (
          &lt;div className="p-3 bg-gray-50 rounded border border-gray-200 font-mono text-sm"&gt;
            &lt;div className="text-green-600 mb-2"&gt;{testOutput.message}&lt;/div&gt;
            &lt;div className="text-gray-600 text-xs space-y-1"&gt;
              &lt;div&gt;API Version: &lt;span className="text-blue-600 font-medium"&gt;{testOutput.version}&lt;/span&gt;&lt;/div&gt;
              &lt;div&gt;Deployment ID: &lt;span className="text-purple-600 font-medium"&gt;{testOutput.deploymentId}&lt;/span&gt;&lt;/div&gt;
              &lt;div&gt;Call #: {callCount}&lt;/div&gt;
              &lt;div&gt;Time: {new Date(testOutput.timestamp).toLocaleTimeString()}&lt;/div&gt;
            &lt;/div&gt;
          &lt;/div&gt;
        ) : testInProgress ? (
          &lt;div className="p-3 bg-gray-50 rounded border border-gray-200 text-sm text-gray-600"&gt;
            Testing deployment routing...
          &lt;/div&gt;
        ) : (
          &lt;div className="p-3 bg-gray-50 rounded border border-gray-200 text-sm text-gray-600"&gt;
            Click &amp;quot;Test Route&amp;quot; to verify how requests are routed to the correct deployment version
          &lt;/div&gt;
        )}
      &lt;/div&gt;
    &lt;/div&gt;
  );
} </code></pre> 
<p>4. Now create an API route that uses the <code>X-Amplify-Dpl</code> header to identify which deployment the request is coming from, simulating how Amplify routes API requests to maintain version consistency during deployments. Use the following code to create the API route:</p> 
<pre><code class="lang-ts">// app/api/skew-protection/route.ts
import { NextResponse } from 'next/server';
import { type NextRequest } from 'next/server';

// This version identifier can be changed between deployments to demonstrate skew protection
const CURRENT_API_VERSION = "v2.0";

export async function GET(request: NextRequest) {
  // Get the deployment ID from the X-Amplify-Dpl header
  // This is how Amplify routes API requests to the correct deployment version
  const deploymentId = request.headers.get('x-amplify-dpl') || '';
  
  // Determine which version to serve based on deployment ID
  const apiVersion = CURRENT_API_VERSION;
  const message = `Hello from API ${apiVersion}! ?`;
  
  // Return the response with deployment information
  return NextResponse.json({
    message,
    version: apiVersion,
    deploymentId: deploymentId || 'none',
    timestamp: new Date().toISOString()
  });
} 
</code></pre> 
<p>5. Add the Amplify deployment ID environment variable to make it accessible to client code</p> 
<pre><code class="lang-ts">// next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  env: {
    AWS_AMPLIFY_DEPLOYMENT_ID: process.env.AWS_AMPLIFY_DEPLOYMENT_ID || '',
  }
};
export default nextConfig;
</code></pre> 
<p>6. Push the changes to a GitHub repository</p> 
<ul> 
 <li>Create a new GitHub repository</li> 
 <li>Add and commit the changes to the Git branch</li> 
 <li>Add remote origin and push the changes upstream</li> 
</ul> 
<pre><code class="lang-git">git add .
git commit -m "initial commit"
git remote add origin https://github.com/&lt;OWNER&gt;/amplify-skew-protection-demo.git
git push -u origin main</code></pre> 
<h3>Deploy the application to Amplify Hosting</h3> 
<p>Use the following steps to deploy your newly constructed application to AWS Amplify Hosting:</p> 
<ol> 
 <li>Sign in to the <a href="https://console.aws.amazon.com/amplify/">AWS Amplify console</a>.</li> 
 <li>Choose <strong>Create new app</strong> and select <strong>GitHub</strong> as the repository source</li> 
 <li>Authorize Amplify to access your GitHub account</li> 
 <li>Choose the repository and branch you created</li> 
 <li>Review the <strong>App</strong> settings and then choose <strong>Next</strong></li> 
 <li>Review the overall settings and choose <strong>Save and deploy</strong></li> 
</ol> 
<h3>Enable skew protection</h3> 
<p>In the Amplify console, navigate to <strong>App settings</strong> and then <strong>Branch settings</strong>. Select the <strong>Branch</strong>, and from the Actions dropdown menu choose <strong>Enable skew protection</strong>.</p> 
<p><img alt="Screenshot of the branch settings in the AWS Console" class="alignnone size-full wp-image-13913" height="588" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/03/11/Screenshot-2025-03-06-at-3.45.58 PM.png" width="1754" /></p> 
<p style="text-align: center;">FIGURE 2 – Amplify Hosting Branch Settings</p> 
<p>Next, navigate to the deployments page and redeploy your application. When skew protection is enabled for an application, AWS Amplify must update its CDN cache conﬁguration. Therefore, you should expect your ﬁrst deployment after enabling skew protection to take up to ten minutes.</p> 
<p><img alt="Screenshot of the AWS Amplify Console. Shows a deployed app" class="alignnone size-full wp-image-13914" height="830" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/03/11/Screenshot-2025-03-06-at-3.46.44 PM.png" width="1752" /></p> 
<p style="text-align: center;">FIGURE 3 – Amplify Hosting App Deployments</p> 
<h3>Access the deployed Next.js app</h3> 
<p>Navigate to the <strong>Overview</strong> tab in the Amplify console and open the default Amplify generated URL in the browser. You should now observe a list of fingerprinted assets for your app along with the deployment ID.</p> 
<p><img alt="Screenshot of Amplify Hosting settings " class="alignnone size-full wp-image-13915" height="776" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/03/11/Screenshot-2025-03-06-at-3.45.04 PM.png" width="1752" /></p> 
<p style="text-align: center;">FIGURE 4 – Amplify Hosting App Settings – Branch level</p> 
<p><img alt="Screenshot of the deployed app to see skew protection demo" class="alignnone size-full wp-image-13916" height="1924" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/03/11/Screenshot-2025-03-07-at-4.02.28 PM.png" width="2156" /></p> 
<p style="text-align: center;">FIGURE 5 – Demo App Homepage</p> 
<h3>Testing skew protection</h3> 
<p>When you deploy your Next.js application to Amplify, each deployment gets assigned a unique deployment ID. This ID is automatically injected into your static assets (JS, CSS) and API routes to ensure version consistency. Let’s see it in action:</p> 
<ol> 
 <li>Asset Fingerprinting: Notice how each static asset URL includes a ?dpl= parameter with your current deployment ID. This ensures browsers always fetch the correct version of your assets.</li> 
 <li>API Routing: The <code>Test Route</code> button demonstrates how Amplify routes API requests. When clicked, it makes a request to the <code>/api/skew-protection</code> endpoint. Since the request utilizes the <code>X-Amplify-Dpl</code> header to match your current deployment ID, it ensures routing to the correct API version.</li> 
</ol> 
<p>This means that even during deployments, your users won’t experience version mismatches and each user’s session stays consistent with the version they initially loaded, preventing bugs that could occur when client and server versions don’t match.</p> 
<h3>Try it yourself</h3> 
<ol> 
 <li>Keep your current browser tab open and click Test Route to see the API version and deployment ID match.</li> 
 <li>Deploy a new version with a different CURRENT_API_VERSION in <code>api/skew-protection/route.ts</code></li> 
 <li>Open your application in a new incognito window 
  <ol> 
   <li>Compare the behavior: 
    <ul> 
     <li> 
      <ul> 
       <li>Your original tab will maintain the old version</li> 
       <li>The new incognito window will show the new version</li> 
       <li>Each tab’s assets and API calls will consistently match their respective versions</li> 
       <li>Try clicking <code>Test Route</code> repeatedly in both windows – each will consistently route to its respective version, demonstrating how Amplify maintains session consistency even when multiple versions are live</li> 
      </ul> </li> 
    </ul> <p><img alt="Screenshot of the demo app comparing both versions" class="size-full wp-image-13946 aligncenter" height="589" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/03/13/Screenshot-2025-03-13-at-12.28.02 PM.png" width="1002" /></p></li> 
  </ol> </li> 
</ol> 
<p style="text-align: center;">FIGURE 6 -Side by Side Comparison of Skew Protection Behavior</p> 
<p>This demonstrates how Amplify maintains version consistency for each user session, even when multiple versions of your application are running during deployments.</p> 
<p>Congratulations, you’ve successfully created and verified skew protection on your Next.js application deployments on Amplify Hosting.</p> 
<h3>Cleanup</h3> 
<p>Delete the AWS Amplify app by navigating to <strong>App settings</strong>, next go to <strong>General settings</strong>, then choose <strong>Delete app</strong>.</p> 
<h2>Next Steps</h2> 
<ol> 
 <li>Enable Skew Protection for your application.</li> 
 <li><a href="https://docs.aws.amazon.com/amplify/latest/userguide/skew-protection.html">Read our documentation</a> to learn more about this feature!</li> 
</ol> 
<h3>About the Authors</h3> 
<p><img alt="Matt Headshot" class="size-full wp-image-12567 alignleft" height="146" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2024/02/19/Screenshot-2024-02-19-at-3.13.45 PM.png" width="142" /></p> 
<p><strong>Matt Auerbach, Senior Product Manager, Amplify Hosting</strong></p> 
<p>Matt Auerbach is a NYC-based Product Manager on the AWS Amplify Team. He educates developers regarding products and offerings, and acts as the primary point of contact for assistance and feedback. Matt is a mild-mannered programmer who enjoys using technology to solve problems and making people’s lives easier. B night, however…well he does pretty much the same thing. You can find Matt on X <a href="https://x.com/mauerbac">@mauerbac</a>. He previously worked at Twitch, Optimizely and Twilio.</p> 
<p><img alt="Jay Author" class="alignleft wp-image-13182 size-thumbnail" height="150" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2024/08/13/IMG_9570-150x150.jpg" width="150" /></p> 
<p><strong>Jay Raval, Solutions Architect, Amplify Hosting</strong></p> 
<p>Jay Raval is a Solutions Architect on the AWS Amplify team. He’s passionate about solving complex customer problems in the front-end, web and mobile domain and addresses real-world architecture problems for development using front-end technologies and AWS. In his free time, he enjoys traveling and sports. You can find Jay on X <a href="https://x.com/i/flow/login?redirect_after_login=%2F_Jay_Raval_">@_Jay_Raval_</a></p>
