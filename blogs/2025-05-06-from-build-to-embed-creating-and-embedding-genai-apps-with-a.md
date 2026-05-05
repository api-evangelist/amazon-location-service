---
title: "From Build to Embed: Creating and Embedding GenAI Apps with AWS Amplify, CDK, and Amazon Q Business"
url: "https://aws.amazon.com/blogs/mobile/from-build-to-embed-creating-and-embedding-genai-apps-with-aws-amplify-cdk-and-amazon-q-business/"
date: "Tue, 06 May 2025 21:41:20 +0000"
author: "Ben-Amin York Jr."
feed_url: "https://aws.amazon.com/blogs/mobile/feed/"
---
<p>In a enterprise landscape, custom applications play a critical role in improving operations, enhancing productivity, and centralizing knowledge within the organization. However, these tools often lack intelligent, conversational interfaces that help users access relevant information faster and more intuitively. Traditional dashboards and search bars fall short when it comes to interpreting complex queries or surfacing contextual insights from vast organizational data.</p> 
<p>Generative AI offers a powerful solution to this challenge. By embedding conversational experiences directly into developer-controlled applications, organizations can enable users to ask questions in natural language and receive precise, actionable responses. <a href="https://aws.amazon.com/q/business/">Amazon Q Business</a> delivers this capability via a secure, <a href="https://docs.aws.amazon.com/amazonq/latest/qbusiness-ug/embed-amazon-q-business.html">embeddable HTML inline frame</a> (iframe)—without the burden of managing large language model infrastructure.</p> 
<p>This blog is aimed at developers building custom or enterprise-owned applications—whether knowledge portals, support dashboards, or internal web tools. It shows how to embed a generative AI-powered conversational experience using Amazon Q Business, <a href="https://docs.amplify.aws/react/how-amplify-works/concepts/">AWS Amplify Gen 2</a>, and the <a href="https://aws.amazon.com/cdk/">AWS Cloud Development Kit</a> (CDK). Embedding Amazon Q Business into applications requires access to the application’s source code and is not supported in third-party SaaS platforms where embedding custom code is not possible.</p> 
<p><b>The presented approach enables</b>:</p> 
<ul> 
 <li>Conversational access to internal documents and knowledge bases.</li> 
 <li>Secure integration with enterprise identity management systems.</li> 
 <li>Scalable, AI-driven search without complex backend implementations.</li> 
 <li>Rapid deployment using AWS Amplify’s capabilities for frontend and backend development.</li> 
</ul> 
<p>With Amazon Q Business and AWS Amplify, you can quickly add generative AI to your apps—to boost productivity, reduce manual effort, and accelerate decision-making.</p> 
<h6><img alt="Opening code in VSCode." class="alignnone size-full wp-image-12994" height="1200" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/02/app1.gif" width="1430" /><em>Figure 1 Submitting a prompt to Amazon Q Business iframe.</em></h6> 
<p>To embed a generative AI assistant into your internal application, you’ll leverage the following AWS services:</p> 
<ul> 
 <li><strong><a href="https://docs.amplify.aws/">AWS Amplify</a>:</strong> A comprehensive set of tools and services that help developers build, deploy, and manage secure full-stack applications. It simplifies frontend and backend development by integrating tightly with services like Amazon Cognito for authentication, Amazon S3 for storage, and the CDK for building additional AWS infrastructure.</li> 
 <li><strong><a href="https://aws.amazon.com/cognito/">Amazon Cognito</a>:</strong> A managed service for adding authentication and authorization to your app. Cognito supports user sign-up, sign-in, and access control, and can be federated through an external identity provider (IdP) for enterprise access management.</li> 
 <li><strong><a href="https://aws.amazon.com/iam/identity-center/">AWS IAM Identity Center</a>:</strong> IAM Identity Center allows secure, centralized access management for your internal users. It supports identity federation with enterprise providers like Okta, Microsoft Entra ID, Ping Identity, and others. This enables your organization to enforce unified authentication policies and ensure only authorized users can interact with the embedded AI assistant.</li> 
 <li><strong><a href="https://aws.amazon.com/q/business/">Amazon Q Business</a>:</strong> A managed generative AI service that can be embedded via iframe into internal applications. Q Business connects to enterprise data sources, such as Amazon S3, and enables natural language querying through an intelligent assistant interface. It supports secure access and integrates with IAM Identity Center for federated enterprise use.</li> 
 <li><strong><a href="https://aws.amazon.com/s3/">Amazon Simple Storage Service (S3)</a>:</strong> A durable and scalable object storage service used to store internal documents, PDFs, manuals, or any unstructured content. These files act as the knowledge base that powers the Amazon Q Business assistant, enabling contextual responses to employee queries.</li> 
</ul> 
<h6><img alt="Opening code in VSCode." class="alignnone size-full wp-image-12994" height="1200" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/02/archit.png" width="1430" /><em>Figure 2 From Build to Embed Architecture Diagram</em></h6> 
<h2>Prerequisites</h2> 
<ul> 
 <li><strong>An <a href="https://signin.aws.amazon.com/signup?request_type=register">AWS account</a>:</strong> Note that AWS Amplify is part of the <a href="https://aws.amazon.com/amplify/pricing/">AWS Free Tier</a>.</li> 
 <li><strong>Install:</strong> <a href="https://www.npmjs.com/">npm</a> (v9 or later), and <a href="https://git-scm.com/">git</a> (v2.14.1 or later).</li> 
 <li><strong>A Text Editor:</strong> For this guide we will use VSCode, but you can use your preferred IDE.</li> 
 <li><strong>Sample Dataset:</strong> Upload any PDF or explore sample data sets available on <a>Kaggle</a>.</li> 
 <li><strong>IAM Identity Center:</strong> You must <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/enable-identity-center.html#to-enable-identity-center-instance">enable an IAM Identity Center instance</a> and <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/addusers.html">add a user</a> to your Identity Center directory.</li> 
</ul> 
<h2>Cloning the Repo</h2> 
<p><strong>Step 1:</strong> Navigate to the <a href="https://github.com/aws-samples/sample-build-and-embed-genai-apps">repository</a> on AWS Samples and fork it to your GitHub repositories.</p> 
<p><strong>Step 2:</strong> Clone the app by running the command below in your terminal.</p> 
<pre><code class="lang-bash">git clone https://github.com/&lt;YOUR_GITHUB_USERNAME&gt;/sample-build-and-embed-genai-apps.git</code></pre> 
<p><strong>Step 3:</strong> Access the newly cloned repository in VSCode by executing the commands below in your terminal.</p> 
<pre><code class="lang-bash">cd sample-build-and-embed-genai-apps 
code . -r </code></pre> 
<p>VSCode will open the repository folder, including the Amplify folder, which contains the app code that you’ll review in the next section.</p> 
<h6><img alt="Opening code in VSCode." class="alignnone size-full wp-image-12994" height="1200" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/02/ide-1.png" width="1430" /><em>Figure 3 Opening code in VSCode.</em></h6> 
<p><strong>Step 4:</strong> Install the required packages, including the Amplify Gen 2 packages, by running the following command:</p> 
<pre><code class="lang-bash">npm install</code></pre> 
<h2>The Amplify Backend</h2> 
<p>In the final app (as seen in the gif at the beginning of the post), users log into the application, click the chatbot icon, authenticate with federated access (via the <a href="https://docs.aws.amazon.com/amazonq/latest/qbusiness-ug/embed-web-experience.html">iframe</a> to <a href="https://docs.aws.amazon.com/amazonq/latest/qbusiness-ug/create-application-iam.html">access Q Business web experience</a>), and finally are able to begin asking questions to Amazon Q Business. The code for this is in the repository you cloned. Here, you’ll go over the key steps for creating your Amplify-developed and hosted search engine app.</p> 
<h6 style="text-align: center;"><img alt="Amplify Gen 2 Project Folder Structure." class="alignnone size-full wp-image-12994" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/02/ampbackend.png" style="width: 35%; height: auto;" /><br /> <em>Figure 4 Amplify Gen 2 Project Folder Structure.</em></h6> 
<p>In the <span style="color: red; font-weight: bold;"><code>amplify/auth/resource.ts</code></span> file (Figure 5), authentication is configured to require users to log in with their email to access the application and upload files. By enabling email-based login, you ensure only verified users can interact with sensitive data and functionalities.</p> 
<pre><code class="lang-ts">import { defineAuth } from '@aws-amplify/backend';
 
export const auth = defineAuth({
    loginWith: {
        email: true,
    },
});</code></pre> 
<h6><em>Figure 5 defineAuth in amplify/auth/resource.ts</em></h6> 
<p>In the <span style="color: red; font-weight: bold;"><code>amplify/storage/resource.ts</code></span> file (Figure 6), <a href="https://docs.amplify.aws/react/build-a-backend/storage/">Amplify storage</a> is configured to enable secure, user-scoped file management. The <code>defineStorage</code> function instantiates the storage resource with a user-friendly name <code>q-datasource-bucket</code> and applies access control to the <code>protected/{entity_id}/*</code> path. This configuration allows authenticated users to read files within their own scoped directory, while granting the file owner permissions to read, write, and delete their content.</p> 
<pre><code class="lang-ts">import { defineStorage } from "@aws-amplify/backend";
 
export const storage = defineStorage({
    name: "q-datasource-bucket",
    access: (allow) =&gt; ({
        'protected/{entity_id}/*': [
            allow.authenticated.to(['read']),
            allow.entity('identity').to(['read', 'write', 'delete'])
        ]
    })
});</code></pre> 
<h6><em>Figure 6 defineStorage in amplify/storage/resource.ts</em></h6> 
<p>In the <span style="color: red; font-weight: bold;"><code>amplify/backend.ts</code></span> (Figure 7) file, you import the CDK libraries to configure key aspects of your application. The <code>aws-iam</code> module is used to manage permissions, <code>aws-kms</code> handles encryption and key management, and <code>aws-qbusiness</code> integrates Amazon Q Business into your stack. Each library plays a specific role in ensuring your application is secure and properly integrated with AWS services.</p> 
<pre><code class="lang-ts">import * as iam from 'aws-cdk-lib/aws-iam';
import * as kms from 'aws-cdk-lib/aws-kms';
import * as q from 'aws-cdk-lib/aws-qbusiness';</code></pre> 
<h6><em>Figure 7 import CDK libraries in amplify/backend.ts</em></h6> 
<p>Next, use the <span style="color: red; font-weight: bold;"><code>backend.createStack()</code></span> (Figure 8) method to direct the backend to generate a new CloudFormation Stack to house custom resources. With AWS Amplify Gen 2, you can create custom resources using the CDK, enabling the use of services beyond the Amplify library, with stacks backed by <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html">CloudFormation</a> templates for scalability. For example, you could create a Generative AI stack for AI-related services, ensuring logical organization before adding custom AWS resources. Now, you can begin defining custom AWS resources!</p> 
<pre><code class="lang-ts">export const customResource = backend.createStack("CustomResourceStack");</code></pre> 
<h6><em>Figure 8 define backend stack for custom resources in amplify/backend.ts</em></h6> 
<p>During the <span style="color: red; font-weight: bold;"><code>CustomResourceStack</code></span> deployment, you’ll see several <b>IAM roles</b>, <b>trust policies</b>, and <b>inline access policies</b> being created—all of which are critical for enabling Amazon Q Business to function securely and interact with other AWS services. These include:</p> 
<ul> 
 <li><b>Service Access Role</b> (QApplicationServiceAccessRole) for Amazon Q Business to emit logs and metrics via CloudWatch.</li> 
 <li><b>Web Experience Roles</b> (QWebExperienceRole and QWebExperienceRole) for enabling the embedded assistant to function with full application context and user interactions.</li> 
 <li><b>Data Source Role</b> (QBusinessS3Role) specifically for allowing Amazon Q Business to read documents from the S3 bucket provisioned by Amplify Storage.</li> 
</ul> 
<p>Each role includes a trust policy defining <b>who can assume the role</b>, and a series of fine-grained permissions granting access to specific actions and resources. You can view the full required policy structure for Amazon Q Business data sources in the <a href="https://docs.aws.amazon.com/amazonq/latest/qbusiness-ug/iam-roles-ds.html#create-s3-datasource-iam-role">IAM roles documentation for Amazon Q Business connectors</a>.</p> 
<h2>Setting Up Amazon Q Business</h2> 
<p>With all foundational IAM roles and policies in place, you’re ready to define the Amazon Q Business application, the core component that powers the embedded generative AI assistant. In your <span style="color: red; font-weight: bold;"><code>amplify/backend.ts</code></span> (Figure 9) file, using the CDK, you can instantiate it declaratively with the necessary configuration, subscription plan, and IAM integration:</p> 
<pre><code class="lang-ts">export const qapp = new q.CfnApplication(customResource, "Qapp", {
  displayName: "Qapp",
  description: "CDK instantiated Amazon Q Business App",
  autoSubscriptionConfiguration: {
    autoSubscribe: "ENABLED",
    defaultSubscriptionType: "Q_LITE"
  },
  identityType: "AWS_IAM_IDC",
  roleArn: `arn:aws:iam::${customResource.account}:role/aws-service-role/qbusiness.amazonaws.com/AWSServiceRoleForQBusiness`,
  /* REPLACE WITH YOUR IAM IDENTITY CENTER ARN */
  // identityCenterInstanceArn: "arn:aws:sso:::instance/",
});</code></pre> 
<h6><em>Figure 9 defining Q Business Application in amplify/backend.ts</em></h6> 
<p>This step formally creates the Q Business application and associates it with the service-linked role (<b>AWSServiceRoleForQBusiness</b>). Be sure to substitute your IAM Identity Center ARN before deployment to enable federated access for users.</p> 
<h2>Creating an Index</h2> 
<p>Indexes in Amazon Q Business are used to store, organize, and retrieve enterprise data efficiently. You need an index to enable structured querying of stored documents, allowing the chatbot to retrieve relevant responses based on user queries. The following CDK code in your <span style="color: red; font-weight: bold;"><code>amplify/backend.ts</code></span> (Figure 10) sets up an index within your Amazon Q Business application:</p> 
<pre><code class="lang-ts">export const qIndex = new q.CfnIndex(customResource, "QIndex", {
  displayName: "QIndex",
  description: "CDK instantiated Amazon Q Business App index",
  applicationId: qapp.attrApplicationId,
  capacityConfiguration: {
    units: 1,
  },
  type: "STARTER",
});</code></pre> 
<h6><em>Figure 10 defining Q Business Index in amplify/backend.ts</em></h6> 
<p>Here, the STARTER index type is selected, which is a basic configuration suitable for testing and small-scale deployments. For production use, a higher tier like <b>ENTERPRISE</b> would be necessary to ensure scalability, availability, and support for advanced features—see the <a href="https://docs.aws.amazon.com/amazonq/latest/qbusiness-ug/tiers.html">Amazon Q Business pricing and tiers</a> for guidance.</p> 
<h2>Creating a Retriever</h2> 
<p>Retrievers enhance the search capabilities by fetching relevant documents from the index. In Amazon Q Business, a retriever connects to the index and ensures that queries return meaningful responses. The following CDK code in your <span style="color: red; font-weight: bold;"><code>amplify/backend.ts</code></span> (Figure 11) sets up a retriever within your Amazon Q Business application:</p> 
<pre><code class="lang-ts">export const qRetriever = new q.CfnRetriever(customResource, "QRetriever", {
  displayName: "QRetriever",
  applicationId: qapp.attrApplicationId,
  type: "NATIVE_INDEX",
  configuration: {
    nativeIndexConfiguration: {
      indexId: qIndex.attrIndexId,
    },
  }
});</code></pre> 
<h6><em>Figure 11 defining Q Business Retriever in amplify/backend.ts</em></h6> 
<p>This retriever is configured to work with the previously created <span style="color: red; font-weight: bold;"><code>qIndex</code></span>, ensuring efficient retrieval of indexed content for Amazon Q Business.</p> 
<h2>Defining a Data Source</h2> 
<p>A data source is where the Amazon Q Business application retrieves data for its knowledge base. In this case, the data source is an Amazon S3 bucket defined earlier in the Amplify Storage configuration <span style="color: red; font-weight: bold;"><code>amplify/storage/resource.ts</code></span> and referenced in the <span style="color: red; font-weight: bold;"><code>amplify/backend.ts</code></span> (Figure 12) file. This bucket stores documents that Amazon Q Business will index and analyze.</p> 
<pre><code class="lang-ts">export const qDataSource = new q.CfnDataSource(customResource, "QDataSource", {
  displayName: `${backend.storage.resources.bucket.bucketName}`,
  applicationId: qapp.attrApplicationId,
  indexId: qIndex.attrIndexId,
  configuration: {
    type: "S3",
    syncMode: "FULL_CRAWL",
    connectionConfiguration: {
      repositoryEndpointMetadata: {
        BucketName: backend.storage.resources.bucket.bucketName
      }
    },
    repositoryConfigurations: {
      document: {
        fieldMappings: [
          {
            indexFieldName: "s3_document_id",
            indexFieldType: "STRING",
            dataSourceFieldName: "s3_document_id"
          }
        ]
      }
    },
  },
  roleArn: qBusinessS3Role.roleArn
});</code></pre> 
<h6><em>Figure 12 defining Q Business Data Source in amplify/backend.ts</em></h6> 
<p>The key elements in this setup:</p> 
<ul> 
 <li><b>syncMode</b>: “FULL_CRAWL” ensures that all documents in the S3 bucket are indexed.</li> 
 <li>Field mappings define how document metadata is structured for indexing.</li> 
</ul> 
<h2>Creating the Amazon Q Business Web Experience</h2> 
<p>The Web Experience in amazon Q business is a frontend that allows users to interact with Amazon Q Business embedded in a customized chatbot. This component is defined in your <span style="color: red; font-weight: bold;"><code>amplify/backend.ts</code></span> (Figure 13) file and defines how the chatbot interface appears and functions, including allowed website origins, branding, and authentication.</p> 
<pre><code class="lang-ts">export const qWebExperience = new q.CfnWebExperience(customResource, "QWebExperience", {
  applicationId: qapp.attrApplicationId,
  origins: [
    /* REPLACE WITH YOUR AMPLIFY DOMAIN URL */
    "https://main..amplifyapp.com",
  ],
  samplePromptsControlMode: "ENABLED",
  subtitle: "AnyCompany Generative AI Assistant",
  title: "AnyCompany Q App",
  welcomeMessage: "Welcome to your Amazon Q Business Application!",
  roleArn: qWebExperienceRole.roleArn
});</code></pre> 
<h6><em>Figure 13 defining Q Business Web Experience in amplify/backend.ts</em></h6> 
<h2>Web Experience Configuration</h2> 
<ul> 
 <li><b>Origins</b>: Specifies the website domains permitted to embed the chatbot via an &lt;iframe&gt;. Ensure that your Amplify app’s deployed URL is correctly listed here.</li> 
 <li><b>samplePromptsControlMode</b>: Enables storing predefined sample prompts within the chatbot UI.</li> 
 <li><b>title &amp; subtitle</b>: Sets the chatbot’s display name and additional descriptions.</li> 
 <li><b>welcomeMessage</b>: The first message users see when they open the chat window.</li> 
</ul> 
<h2>Amazon Q Business and Frontend Integration</h2> 
<p>After configuring the web experience, you can now <a href="https://docs.aws.amazon.com/amazonq/latest/qbusiness-ug/embed-web-experience.html">embed your Amazon Q Business web experience</a> using an iframe. In <span style="color: red; font-weight: bold;"><code>src/components/qframe.tsx</code></span> (Figure 14), the src is dynamically populated from the <span style="color: red; font-weight: bold;"><code>amplify_outputs.json</code></span> file, which contains the deployed URL of your Q Web Experience exported by the backend. Users are prompted to authenticate in a new browser tab, after which the chat session continues within the embedded iframe.</p> 
<pre><code class="lang-ts">import React from 'react';
import rawOutputs from '../../amplify_outputs.json';
 
const outputs = rawOutputs as unknown as {
    custom: {
        q_business_url: string;
    };
};
 
const QFrame: React.FC = () =&gt; {
    const qBusinessDeployedURL = outputs.custom.q_business_url;
 
    return (
        &lt;iframe
            src={qBusinessDeployedURL}
            title="Chatbot"
            className="QFrame"
        /&gt;
    );
};
 
export default QFrame;</code></pre> 
<h6><em>Figure 14 Embeds Amazon Q Business via iframe using config URL.</em></h6> 
<h2>Running the App</h2> 
<p><strong>Step 1:</strong> Amplify provides each developer with a personal <a href="https://docs.amplify.aws/react/deploy-and-host/sandbox-environments/setup/">cloud sandbox environment</a>, offering isolated development spaces for rapid building, testing, and iteration. To initiate a cloud sandbox environment, open a new terminal window and execute the following command:</p> 
<pre><code class="lang-bash">npx ampx sandbox</code></pre> 
<p><strong>Step 2:</strong> Execute the following command to start a localhost development server.</p> 
<pre><code class="lang-bash">npm run dev</code></pre> 
<p>After running the previous command to start the application, use the ‘<b>Create Account</b>‘ feature of the <a href="https://ui.docs.amplify.aws/react/connected-components/authenticator/configuration">Amplify Authenticator component</a>, and provide an email address and password. After finalizing user setup via a confirmation email, log in to gain access to the application. (Figure 15)</p> 
<h6><img alt="Logging in using the Amplify Authenticator component during local development testing." class="alignnone size-full wp-image-12994" height="1200" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/02/localtest.gif" width="1430" /><em>Figure 15 Logging in using the Amplify Authenticator component during local development testing.</em></h6> 
<p>After interacting with the app on the development server, stop the sandbox environment by pressing <b>Ctrl + C</b> in the terminal. Then enter “<b>npx ampx sandbox delete</b>” and confirm by entering ‘<b>Y</b>‘ when prompted to delete resources in the sandbox environment. (Figure 16)</p> 
<h6><img alt="Deleting all resources in the Amplify sandbox environment" class="alignnone size-full wp-image-12994" height="1200" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/02/sandbox.png" width="1430" /><em>Figure 16 Deleting all resources in the Amplify sandbox environment</em></h6> 
<h2>Deploying backend resources</h2> 
<p>With the app functioning as expected, deploy your backend resources by following the steps in “<a href="https://docs.aws.amazon.com/amplify/latest/userguide/getting-started.html">Getting Started with Deploying an App to Amplify Hosting</a>“. Ensure that GitHub is selected as the repository for deployment.</p> 
<h2>Update Amplify Domain Endpoint</h2> 
<p>In your backend CDK code, navigate to <span style="color: red; font-weight: bold;"><code>amplify/backend.ts</code></span> and replace the placeholder in the origins field with your actual Amplify domain. You can find this domain in the Amplify console under the app name you created in the previous section. (Figure 17)</p> 
<pre><code class="lang-ts">origins: [
  "https://main.&lt;AMPLIFY_APP_ID&gt;.amplifyapp.com",
],</code></pre> 
<h6><em>Figure 17 Updating the origins field in the CDK backend code with your Amplify app’s deployed domain.</em></h6> 
<h2>Upload Sample Data</h2> 
<p>After deploying and signing into the application, <b>upload sample data</b> by accessing the app domain URL generated by Amplify. After uploading files, confirm the upload by checking the public/ folder within the Storage section of the AWS Amplify console. (Figure 18)</p> 
<h6><img alt="Uploading sample data in deployed app." class="alignnone size-full wp-image-12994" height="1200" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/02/upload.gif" width="1430" /><em>Figure 18 Uploading sample data in deployed app.</em></h6> 
<h2>Configure User Access to Q Business Web Experience</h2> 
<p>Next, for your Qapp, you need to assign a user who can log in to the embedded web experience. This user should have been created during the prerequisites using <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/addusers.html">IAM Identity Center</a>. To add a new or existing user, in the <a href="https://us-east-1.console.aws.amazon.com/amazonq/business/applications?">Amazon Q Business</a> console, select your <b>Qapp</b>, then navigate to the <b>User access</b> section and click Manage user access, followed by <b>Add groups and users</b>. (Figure 19)</p> 
<h6><img alt="Assigning user access via IAM Identity Center." class="alignnone size-full wp-image-12994" height="1200" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/02/assignUser.png" width="1430" /><em>Figure 19 Assigning user access via IAM Identity Center.</em></h6> 
<p>If the user already exists, choose <b>Assign existing users and groups</b> and search for the user you want to assign. Once added, <b>confirm</b> your selection to grant access to the Amazon Q Business web experience. (Figure 20)</p> 
<h6><img alt="Granting access to existing IAM Identity Center users." class="alignnone size-full wp-image-12994" height="1200" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/02/confirmUser.png" width="1430" /><em>Figure 20 Granting access to existing IAM Identity Center users.</em></h6> 
<h2>Sync S3 Data to Q Business Index</h2> 
<p>After uploading your sample document, sync the Amazon S3 bucket with the Amazon Q Business index to store and retrieve your data. From the <a href="https://us-east-1.console.aws.amazon.com/amazonq/business/applications?">Amazon Q Business</a> console, navigate to <b>Data Sources</b>, select <b>amplify-your-unique-bucket-name</b>. Then, select <b>Sync now</b> (Figure 21).</p> 
<h6><img alt="Syncing uploaded S3 data to Q Business index." class="alignnone size-full wp-image-12994" height="1200" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/02/sync.png" width="1430" /><em>Figure 21 Syncing uploaded S3 data to Q Business index.</em></h6> 
<p>Once the data source sync completes and the last sync status shows “<b>completed</b>“, you’re almost ready to begin using your Amazon Q Business web experience. <b>Before proceeding, make sure to sign in. After a successful sign-in</b>, you’ll see a “<b>Sign in Complete</b>” message. At that point, you can return to your application—authentication is now complete. (Figure 22)</p> 
<h6><img alt="Confirming sign-in completion for authentication." class="alignnone size-full wp-image-12994" height="1200" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/02/signin.gif" width="1430" /><em>Figure 22 Confirming sign-in completion for authentication.</em></h6> 
<p>Now that you are signed in, start querying your Amazon Q Business web experience directly through your application! (Figure 23)</p> 
<h6><img alt="Querying Amazon Q Business in your application." class="alignnone size-full wp-image-12994" height="1200" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/02/app2.gif" width="1430" /><em>Figure 23 Querying Amazon Q Business in your application.</em></h6> 
<h2>Cleaning Up</h2> 
<p>To remove the <a href="https://console.aws.amazon.com/singlesignon/home">AWS Identity Center console</a> instance, go to “<b>Settings</b>“, and open the “<b>Management</b>” tab. Under the “<b>Delete IAM Identity Center instance</b>” section, select “<b>Delete</b>” to remove it.</p> 
<p>Lastly, within the <a href="https://console.aws.amazon.com/amplify/home#/home">AWS Amplify console</a>, select “<b>View app</b>” for the application you created following this blog. Then, select “<b>App Settings</b>” followed by “<b>General Settings</b>.” Finally, select “<b>Delete app</b>” to remove the application and associated backend resources. Note, Amplify will delete <i>all backend resources</i> created as a part of your project.</p> 
<h2>Conclusion</h2> 
<p>In this blog, we explained the key steps for integrating Amazon Q Business into a custom application: setting up the frontend with AWS Amplify, configuring the Q Business application using the CDK, and embedding the chatbot via an iframe. By leveraging these AWS Services and tools, you can create an AI-powered search experience that improves user interaction and knowledge accessibility.</p> 
<p>Now it’s your turn! Embed conversational AI into your applications and unlock new levels of productivity and decision-making with Amazon Q Business. Then, supercharge your development workflow with <a href="https://aws.amazon.com/codewhisperer/">Amazon Q Developer</a>. Harness the power of generative AI and cloud computing to accelerate your development and quickly build modern applications that drive innovation.</p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Photo of author" class="wp-image-11636 alignleft" height="134" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2024/10/19/ybenamin.jpg" width="101" />
  </div> 
  <h3 class="lb-h4">Ben-Amin York Jr.</h3> 
  <p>Ben-Amin is an AWS Solutions Architect specializing in Frontend Web &amp; Mobile technologies, supports Automotive &amp; Manufacturing enterprises drive digital transformation.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Photo of author" class="wp-image-11636 alignleft" height="134" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/04/dianne.jpg" width="101" />
  </div> 
  <h3 class="lb-h4">Dianne Eldridge</h3> 
  <p>Dianne Eldridge leads global business development for industrial AI at AWS, driving strategy and growth since 2021. Previously, she spent 20 years at Emerson overseeing a global manufacturing portfolio across the US, China, and Italy.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Photo of author" class="wp-image-11636 alignleft" height="134" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/04/Vaidehi-1.png" width="101" />
  </div> 
  <h3 class="lb-h4">Vaidehi Patel</h3> 
  <p>Vaidehi Patel is a Solutions Architect at AWS, supporting Automotive and Manufacturing enterprises. She specializes in Serverless and Amazon Connect, helping customers design and scale AI/ML and Generative AI workloads to drive innovation and digital transformation.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Photo of author" class="wp-image-11636 alignleft" height="134" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/05/04/dextpham.jpg" width="101" />
  </div> 
  <h3 class="lb-h4">Dexter Pham</h3> 
  <p>Dexter Pham is a Solutions Architect at AWS, supporting enterprise customers in the Automotive &amp; Manufacturing sector to realize business and technology goals. He supports customers on their cloud journey covering infrastructure, data, AI/ML,and specializes in VMware and SAP workloads.</p> 
 </div> 
</footer>
