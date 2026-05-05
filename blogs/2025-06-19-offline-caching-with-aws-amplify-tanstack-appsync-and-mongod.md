---
title: "Offline caching with AWS Amplify, Tanstack, AppSync and MongoDB Atlas"
url: "https://aws.amazon.com/blogs/mobile/offline-caching-with-aws-amplify-tanstack-appsync-and-mongodb-atlas/"
date: "Thu, 19 Jun 2025 20:59:54 +0000"
author: "Dan Kiuna"
feed_url: "https://aws.amazon.com/blogs/mobile/feed/"
---
<p>In this blog we demonstrate how to create an offline-first application with optimistic UI using <a href="https://aws.amazon.com/amplify/">AWS Amplify</a>, <a href="https://aws.amazon.com/appsync/">AWS AppSync</a>, and <a href="https://www.mongodb.com/products/platform/atlas-cloud-providers/aws">MongoDB Atlas</a>. Developers design offline first applications to work without requiring an active internet connection. Optimistic UI then builds on top of the offline first approach by updating the UI with expected data changes, without depending on a response from the server. This approach typically utilizes a local cache strategy.</p> 
<p>Applications that use offline first with optimistic UI provide a number of improvements for users. These include reducing the need to implement loading screens, better performance due to faster data access, reliability of data when an application is offline, and cost efficiency. While implementing offline capabilities manually can take sizable effort, you can use tools that simplify the process.</p> 
<p>We provide a <a href="https://github.com/mongodb-partners/amplify-mongodb-tanstack-offline">sample to-do application</a> that renders results of MongoDB Atlas CRUD operations immediately on the UI before the request roundtrip has completed, improving the user experience. In other words, we implement optimistic UI that makes it easy to render loading and error states, while allowing developers to rollback changes on the UI when API calls are unsuccessful. The implementation leverages <a href="https://tanstack.com/query/latest/docs/react/overview">TanStack Query</a> to handle the optimistic UI updates along with <a href="https://docs.amplify.aws/react/build-a-backend/data/optimistic-ui/">AWS Amplify</a>. The diagram in Figure 1 illustrates the interaction between the UI and the backend.</p> 
<p>TanStack Query is an asynchronous state management library for TypeScript/JavaScript, React, Solid, Vue, Svelte, and Angular. It simplifies fetching, caching, synchronizing, and updating server state in web applications. By leveraging TanStack Query’s caching mechanisms, the app ensures data availability even without an active network connection. AWS Amplify streamlines the development process, while AWS AppSync provides a robust GraphQL API layer, and MongoDB Atlas offers a scalable database solution. This integration showcases how TanStack Query’s offline caching can be effectively utilized within a full-stack application architecture.</p> 
<p><img alt="Amplify Tanstack Optimistic UI - Interaction Diagram" class="alignnone wp-image-14195 size-large" height="1024" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/06/10/TanstackWithAtlas-793x1024.png" width="793" /></p> 
<p><strong>Figure 1. Interaction Diagram</strong></p> 
<p>The sample application implements a classic to-do functionality and the exact app architecture is shown in <strong>Figure 2</strong>. The stack consists of:</p> 
<ul> 
 <li>MongoDB Atlas for database services.</li> 
 <li>AWS Amplify the full-stack application framework.</li> 
 <li>AWS AppSync for GraphQL API management.</li> 
 <li>AWS Lambda Resolver for serverless computing.</li> 
 <li>Amazon Cognito for user management and authentication.</li> 
</ul> 
<p><img alt="Amplify Tanstack Optimistic UI - Architecture Diagram" class="alignnone wp-image-14197 size-large" height="557" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/06/10/architecture-1024x557.png" width="1024" /></p> 
<p><strong>Figure 2. Architecture</strong></p> 
<h2>Deploy the Application</h2> 
<p>To deploy the app in your AWS account, follow the steps below. Once deployed you can create a user, authenticate yourself, and create to-do entries – see Figure 8.</p> 
<h3>Set up the MongoDB Atlas cluster</h3> 
<ol> 
 <li>Follow the <a href="https://www.mongodb.com/docs/atlas/tutorial/create-atlas-account/">link</a> to the setup the <a href="https://www.mongodb.com/docs/atlas/tutorial/deploy-free-tier-cluster/">MongoDB Atlas cluster</a>, Database , <a href="https://www.mongodb.com/docs/atlas/tutorial/create-mongodb-user-for-cluster/">User</a> and <a href="https://www.mongodb.com/docs/atlas/security/add-ip-address-to-list/">Network access</a></li> 
 <li>Set up the user 
  <ol> 
   <li><a href="https://www.mongodb.com/docs/atlas/security-add-mongodb-users/">Configure User</a></li> 
  </ol> </li> 
</ol> 
<h3>Clone the GitHub Repository</h3> 
<ol> 
 <li>Clone the sample application with the following command</li> 
</ol> 
<p><code> git clone https://github.com/mongodb-partners/amplify-mongodb-tanstack-offline</code></p> 
<h3>Setup the AWS CLI credentials (optional if you need to debug your application locally)</h3> 
<ol> 
 <li>If you would like to test the application locally using a <a href="https://docs.amplify.aws/react/deploy-and-host/sandbox-environments/setup/">sandbox environment</a>, you can setup temporary AWS credentials locally:</li> 
</ol> 
<pre><code class="lang-bash">export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=
export AWS_SESSION_TOKEN=</code></pre> 
<h3>Deploy the Todo Application in AWS Amplify</h3> 
<ol> 
 <li>Open the AWS Amplify console and Select the Github Option</li> 
</ol> 
<p><img alt="Amplify Tanstack Optimistic UI - Select Github Option" class="alignnone wp-image-14201 size-large" height="317" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/06/10/fig3-github-1024x317.png" width="1024" /></p> 
<p><strong>Figure 3. Select Github option</strong></p> 
<p>2. Configure the GitHub Repository</p> 
<p><img alt="Amplify Tanstack Optimistic UI - Configure Repository Permissions" class="alignnone wp-image-14203 size-large" height="254" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/06/10/fig4-permissions-1024x254.png" width="1024" /></p> 
<p><strong>Figure 4. Configure repository permissions</strong></p> 
<p>3. Select the GitHub Repository and click Next</p> 
<p><img alt="Amplify Tanstack Optimistic UI - Select Repository and Branch" class="alignnone wp-image-14204 size-large" height="472" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/06/10/fig5-branch-1024x472.png" width="1024" /><br /> <strong>Figure 5. Select repository and branch</strong></p> 
<p>4. Set all other options to default and deploy</p> 
<p><img alt="Amplify Tanstack Optimistic UI - Deploy Application" class="alignnone wp-image-14205 size-large" height="362" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/06/10/fig6-deploy-1024x362.png" width="1024" /></p> 
<p><strong>Figure 6. Deploy application</strong></p> 
<h3>Configure the Environment Variables</h3> 
<p>Configure the Environment variables after the successful deployment</p> 
<p><img alt="Amplify Tanstack Optimistic UI - Environment Variables" class="alignnone wp-image-14206 size-full" height="396" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/06/10/fig7-envs.png" width="957" /><br /> <strong>Figure 7. Configure environment variables</strong></p> 
<h3>Open the application and test</h3> 
<p>Open the application through the URL provided and test the application.</p> 
<p><img alt="Amplify Tanstack Optimistic UI - Sample Todo entries" class="alignnone wp-image-14207 size-large" height="521" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/06/10/fig8-test-1024x521.png" width="1024" /><br /> <strong>Figure 8. Sample todo entries</strong></p> 
<p>MongoDB Atlas Output<br /> <img alt="Amplify Tanstack Optimistic UI - Data in Mongo" class="alignnone wp-image-14208 size-large" height="1024" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/06/10/fig9-mongo-914x1024.png" width="914" /><br /> <strong>Figure 9. Data in Mongo</strong></p> 
<h2>Review the Application</h2> 
<p>Now that the application is deployed, let’s discuss what happens under the hood and what was configured for us. We utilized Amplify’s git-based workflow to host our full-stack, serverless web application with continuous deployment. Amplify supports various frameworks, including server side rendered (SSR) frameworks like Next.js and Nuxt, single page application (SPA) frameworks like React and Angular, and static site generators (SSG) like Gatsby and Hugo. In this case, we deployed a SPA React based application. We can include feature branches, custom domains, pull request previews, end-to-end testing, and redirects/rewrites. Amplify Hosting provides a Git-based workflow enables atomic deployments ensuring that updates are only applied after the entire deployment is complete.</p> 
<p>To deploy our application we used AWS Amplify Gen 2, which is a tool designed to simplify the development and deployment of full-stack applications using TypeScript. It leverages the <a href="https://aws.amazon.com/cdk/">AWS Cloud Development Kit</a> (CDK) to manage cloud resources, ensuring scalability and ease of use.</p> 
<p>Before we conclude, it is important to understand our application’s updates concurrency. We implemented a simple optimistic first-come first-served conflict resolution mechanism. The MongoDB Atlas cluster persists updates in the order it receives them. In case of conflicting updates, the latest arriving update will override previous updates. This mechanism works well in applications where update conflicts are rare. It is important to evaluate how this may or may not suit your production needs, requiring more sophisticated approaches. TanStack provides capabilities for more complex mechanisms to handle various connectivity scenarios. By default, TanStack Query provides an “online” network mode, where Queries and Mutations will not be triggered unless you have network connection. If a query runs because you are online, but you go offline while the fetch is still happening, TanStack Query will also pause the retry mechanism. Paused queries will then continue to run once you re-gain network connection. In order to optimistically update the UI with new or changed values, we can also update the local cache with what we expect the response to be. This is approach works well together with TanStack’s “online” network mode, where if the application has no network connectivity, the mutations will not fire, but will be added to the queue, but our local cache can be used to update the UI. Below is a key example of how our sample application optimistically updates the UI with the expected mutation.</p> 
<pre><code class="lang-ts">const createMutation = useMutation({
    mutationFn: async (input: { content: string }) =&gt; {
    // Use the Amplify client to make the request to AppSync
      const { data } = await amplifyClient.mutations.addTodo(input);
      return data;
    },
    // When mutate is called:
    onMutate: async (newTodo) =&gt; {
      // Cancel any outgoing refetches
      // so they don't overwrite our optimistic update
      await tanstackClient.cancelQueries({ queryKey: ["listTodo"] });

      // Snapshot the previous value
      const previousTodoList = tanstackClient.getQueryData(["listTodo"]);

      // Optimistically update to the new value
      if (previousTodoList) {
        tanstackClient.setQueryData(["listTodo"], (old: Todo[]) =&gt; [
          ...old,
          newTodo,
        ]);
      }

      // Return a context object with the snapshotted value
      return { previousTodoList };
    },
    // If the mutation fails,
    // use the context returned from onMutate to rollback
    onError: (err, newTodo, context) =&gt; {
      console.error("Error saving record:", err, newTodo);
      if (context?.previousTodoList) {
        tanstackClient.setQueryData(["listTodo"], context.previousTodoList);
      }
    },
    // Always refetch after error or success:
    onSettled: () =&gt; {
      tanstackClient.invalidateQueries({ queryKey: ["listTodo"] });
    },
    onSuccess: () =&gt; {
      tanstackClient.invalidateQueries({ queryKey: ["listTodo"] });
    },
  });</code></pre> 
<p>We welcome any <a href="https://github.com/mongodb-partners/amplify-mongodb-tanstack-offline/pulls">PRs</a> implementing additional conflict resolution strategies.</p> 
<ul> 
 <li>Try out MongoDB Atlas on <a href="https://aws.amazon.com/marketplace/pp/prodview-pp445qepfdy34">AWS MarketPlace</a></li> 
 <li>Get familiar with <a href="https://aws.amazon.com/amplify/">AWS Amplify</a>, <a href="https://docs.amplify.aws/react/how-amplify-works/">Amplify Gen2</a> and <a href="https://aws.amazon.com/pm/appsync/">AppSync</a></li> 
 <li>For detailed instructions on deploying the application, refer to the <a href="https://docs.amplify.aws/react/start/quickstart/#deploy-a-fullstack-app-to-aws">deployment section</a> of our documentation</li> 
 <li>Submit a <a href="https://github.com/mongodb-partners/amplify-mongodb-tanstack-offline">PR</a> with your enhancements</li> 
</ul> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="wp-image-11636 alignleft" height="160" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/06/10/dankiuna.jpeg" width="120" />
  </div> 
  <h3 class="lb-h4">Dan Kiuna – Specialist Solutions Architect, AWS</h3> 
  <p style="text-align: left;">Dan is a Specialist Solutions Architect at AWS AppSync. He is well experienced at leveraging AppSync’s capabilities alongside other AWS services to create high-performance, real-time, and offline-enabled solutions.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="wp-image-11636 alignleft" height="160" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/06/10/igor-alekseev.jpeg" width="120" />
  </div> 
  <h3 class="lb-h4">Igor Alekseev – Partner Solutions Architect, AWS</h3> 
  <p style="text-align: left;">Igor works with strategic partners helping them build complex, AWS-optimized architectures. Prior joining AWS, as a Data/Solution Architect he implemented many projects in the Big Data domain. As a Data Engineer he was involved in applying AI/ML to fraud detection and office automation. Igor’s projects spanned a variety of industries including communications, finance, public safety, manufacturing, and healthcare.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="wp-image-11636 alignleft" height="160" src="https://d2908q01vomqb2.cloudfront.net/0a57cb53ba59c46fc4b692527a38a87c78d84028/2025/06/11/anuj-panchal.png" width="120" />
  </div> 
  <h3 class="lb-h4">Anuj Panchal – Partner Solutions Architect, Mongo DB</h3> 
  <p style="text-align: left;">As a Partner Solutions Architect at MongoDB, I ideate, develop, and deploy seamless integrations between MongoDB and AWS offerings. My mission is to simplify the developer experience when working with MongoDB and AWS. #LoveYourDevelopers</p> 
 </div> 
</footer>
